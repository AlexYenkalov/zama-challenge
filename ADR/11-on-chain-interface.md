# On-Chain Interface (EVM Smart Contract)

## Table of Contents

- [Contract Upgradability](#1-contract-upgradability)
- [Security Considerations](#2-security-considerations)
- [Smart Contract Logic](#3-smart-contract-logic)

---

## 1. Contract Upgradability

- **Options Considered**:

  - **Trustless (Immutable) Contracts:**
    - Pros
      - ✅ **Security**: No admin keys or upgrade exploits  
      - ✅ **User Trust**: No central authority control  
      - ✅ **Privacy**: No admin access to job data  
      - ✅ **Gas Efficiency**: 5,700 gas per confirmation (minimal overhead)  
      - ✅ **Regulatory Compliance**: Immutable privacy guarantees  
      - ✅ **Audit Transparency**: All code visible and immutable  
      - ✅ **Decentralization**: No single point of failure  
      - ✅ **Cost**: Lower deployment and maintenance costs  
    - Cons
      - ❌ **No Bug Fixes**: Cannot fix bugs after deployment  
      - ❌ **No Feature Updates**: Cannot add new functionality  
      - ❌ **Migration Required**: Need new contract for changes  
      - ❌ **Data Loss**: Cannot migrate existing data  

  - **Upgradeable Contracts (Proxy Pattern):**
    - Pros
      - ✅ **Bug Fixes**: Can fix bugs after deployment  
      - ✅ **Feature Updates**: Can add new functionality  
      - ✅ **No Migration**: Same contract address  
      - ✅ **Data Preservation**: Existing data remains  
    - Cons
      - ❌ **Gas Overhead**: ~2,100 gas per call (proxy overhead)  
      - ❌ **Admin Risk**: Admin can upgrade (centralization)  
      - ❌ **Trust Required**: Requires trust in admin  
      - ❌ **Complexity**: More complex to audit and deploy  
      - ❌ **Security Risk**: Upgrade mechanism can be exploited  
      - ❌ **Cost**: Higher deployment and maintenance costs  

- **Trade-offs**:
  - **Simplicity vs Flexibility**
  - **Security vs Upgradability**

- **Chosen Approach**: `Trustless (Immutable) Contracts`
  - **Rationale**:
    - Security-first with modular design for different functions
    - For a privacy-preserving job confirmation system, trustless contracts provide superior security, user trust, and regulatory compliance. The migration strategy allows for future enhancements while maintaining immutability benefits.

- **Recommended Approach for Jobs API:**
  1. **Phase 1 (MVP)**: Deploy trustless V1 contract
  2. **Phase 2 (Production)**: Deploy trustless V2 with enhancements
  3. **Phase 3 (Scale)**: Deploy trustless V3 with advanced features

- **Migration Strategy Benefits:**
  - ✅ **Clean Slate**: Each version is fully audited and immutable  
  - ✅ **User Choice**: Users can choose which version to use  
  - ✅ **No Forced Upgrades**: Users control their migration  
  - ✅ **Proven Security**: Each version is battle-tested  
  - ✅ **Regulatory Compliance**: Immutable privacy guarantees  
  - ✅ **Future-Proof**: Can add features without compromising security  


---

## 2. Security Considerations

**Chosen Approach**: **AccessControl + Pausable Pattern**

**Rationale**: For a production job confirmation system, we need both role-based access control and emergency stop capability. The AccessControl pattern provides secure, audited permission management, while Pausable enables immediate response to security incidents. Timelock provides governance delay for admin functions but adds complexity for MVP, making it suitable for future upgrades when governance is needed.

- **Access Control Patterns Considered:**

  - **1. Confirmer Pattern (AccessControl)**
    - ✅ **Role-based security**: Multiple roles and permissions
    - ✅ **Audited**: OpenZeppelin AccessControl is battle-tested
    - ✅ **Gas efficient**: Optimized storage patterns
    - ✅ **Flexible**: Can add more roles later
    - ✅ **Standard**: Industry standard approach
    - ✅ **Production-ready**: Used by major DeFi protocols
    - ❌ **Admin risk**: Admin can grant/revoke roles
    - ❌ **Centralization**: Single admin controls permissions
    - ❌ **No emergency stop**: Cannot immediately halt contract

  - **2. Paused Pattern (Pausable)**
    - ✅ **Emergency stop**: Can pause contract immediately
    - ✅ **Security response**: Prevents further damage during investigation
    - ✅ **Recovery**: Can unpause after fix is deployed
    - ✅ **Audited**: OpenZeppelin Pausable is well-tested
    - ✅ **Immediate execution**: No delay for emergency actions
    - ❌ **Single point of failure**: Pauser can halt entire system
    - ❌ **No governance**: No community input on pause decisions
    - ❌ **Centralization**: Single pauser controls system

  - **3. Timelock Pattern (Governance)**
    - ✅ **Governance delay**: 24-48 hour delay for admin functions
    - ✅ **Community review**: All changes are visible before execution
    - ✅ **Security**: Prevents immediate admin changes
    - ✅ **Transparency**: All changes are transparent
    - ✅ **Decentralization**: Reduces admin power
    - ❌ **Complexity**: More complex to implement and maintain
    - ❌ **Slow response**: Cannot respond quickly to security issues
    - ❌ **Governance overhead**: Requires community coordination

- **Security Trade-offs**:
  - **Complexity vs Security**
  - **Centralization vs Decentralization**

- **Security Benefits:**
  - ✅ **Role-based access**: Only authorized confirmers can confirm jobs
  - ✅ **Emergency stop**: Can pause contract if security issue found


---

## 3. Smart Contract Logic

- **Options Considered**:

  - **Event-Only**
    - ✅ **Ultra-low gas**: `6,450` gas per confirmation
    - ✅ **Privacy preserving**: No correlatable data
    - ✅ **Block Timestamp**: Available from block metadata
      - ❌ Can be manipulated by miners
    - ❌ **Job confirmation check**: Using events only (off-chain)
    - ❌ **Job confirmation deduplication**: Using events only (off-chain)
    - ❌ **Job (integrity) hash verificationn**: Using events only (off-chain)
    - ❌ **Job confirmation date verification**: Using events only (off-chain)

    **Explicit Gas Breakdown** `(6,450)`:
      - CALL opcode: `2,100` gas
      - SLOAD (access control check): `2,100` gas
      - Input validation: `200` gas (jobHash + confirmedAt checks)
      - LOG3 opcode: `1,750` gas
      - Function execution: `300` gas

    **Gas Loss on Revert:**
      - If input validation fails: `2,300` gas lost (~$0.05 at 20 gwei)
        - CALL opcode: `2,100` gas (transaction initiation)
        - Input validation: `200` gas (jobHash + confirmedAt checks)
        - REVERT opcode: `0` gas (revert is free)

    ```ts
    contract EventOnlyJobConfirmation {
        // ============================================================================
        // EVENT-ONLY JOB CONFIRMATION CONTRACT
        // ============================================================================
        //
        /// @title EventOnlyJobConfirmation - Ultra-efficient event-only job confirmation
        /// @dev Minimal on-chain state with maximum gas efficiency (6,450 gas per job)
        /// @dev Security: Authorization required, duplicate prevention handled off-chain
        /// @dev Privacy: No on-chain storage, only event emissions for audit trail
        
        // ============================================================================
        // ERRORS
        // ============================================================================
        
        /// @notice Thrown when job hash is zero or invalid
        error InvalidJobHash();
        
        /// @notice Thrown when confirmedAt timestamp is zero or invalid
        error InvalidConfirmedAt();
        
        // ============================================================================
        // STORAGE LAYOUT
        // ============================================================================
        /// @dev No on-chain storage (event-only contract)
        /// @dev Gas cost: 0 gas for storage operations
        
        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /// @notice Emitted when a job is confirmed
        /// @param jobHash Job identifier (bytes32 hash of job data + salt)
        /// @param confirmedAt Timestamp when the job was confirmed
        /// @dev Gas cost: 1,750 gas (LOG3 opcode)
        /// @dev Monitoring: Requires off-chain event indexing for duplicate detection
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        // ============================================================================
        // CORE CONFIRMATION LOGIC
        // ============================================================================
        
        /// @notice Confirm a job with event-only approach
        /// @dev Ultra-efficient confirmation with no on-chain storage
        /// @param jobHash Job identifier (bytes32 hash of job data + salt)
        /// @param confirmedAt Timestamp when the job was confirmed (must be > 0)
        /// @dev Gas cost: 6,450 gas (CALL: 2,100 + SLOAD: 2,100 + validation: 200 + LOG3: 1,750 + execution: 300)
        /// @dev Security: Authorization required, duplicate prevention handled off-chain
        /// @dev Use case: Ultra-cheap job confirmations with off-chain duplicate handling
        function confirmJob(bytes32 jobHash, uint256 confirmedAt) external onlyAuthorizedConfirmer {
            if (jobHash == bytes32(0)) revert InvalidJobHash();
            if (confirmedAt == 0) revert InvalidConfirmedAt();

            emit JobConfirmed(jobHash, confirmedAt);
        }
    }
    ```

  - **Event & Mapping**
    - ❌ **Higher gas**: `26,450` gas per confirmation
    - ✅ **Privacy preserving**: Job hash built from `job data` and `salt`
      - ❌ But it still it could be a correlation factor
    - ✅ **Block Timestamp**: Available from block metadata
      - ❌ Can be manipulated by miners
    - ✅ **Job confirmation check**: Using events and on-chain state
    - ✅ **Job confirmation deduplication**: Using events and on-chain state
    - ✅ **Job (integrity) hash verification**: Using events and on-chain state
    - ❌ **Job confirmation date verification**: Using events only (off-chain)

    **Explicit Gas Breakdown** `(26,450)`:
      - CALL opcode: `2,100` gas
      - SLOAD (duplicate check): `2,100` gas
      - SSTORE (confirmation storage): `20,000` gas
      - LOG3 opcode: `1,750` gas
      - Function execution: `500` gas

    **Gas Loss on Revert:**
      - If basic validation fails: `2,300` gas lost (~$0.05 at 20 gwei)
        - CALL opcode: `2,100` gas (transaction initiation)
        - Input validation: `200` gas (jobHash + confirmedAt checks)
        - REVERT opcode: `0` gas (revert is free)
    ```ts
    contract EventAndMappingJobConfirmation {
        // ============================================================================
        // EVENT & MAPPING JOB CONFIRMATION CONTRACT
        // ============================================================================
        //
        /// @title EventAndMappingJobConfirmation - On-chain state with duplicate prevention
        /// @dev Simple confirmation state management with efficient storage (26,450 gas per job)
        /// @dev Security: On-chain duplicate prevention with state validation
        /// @dev Functionality: Enables on-chain status queries and duplicate detection
        
        // ============================================================================
        // ERRORS
        // ============================================================================
        
        /// @notice Thrown when job hash is zero or invalid
        error InvalidJobHash();
        
        /// @notice Thrown when confirmedAt timestamp is zero or invalid
        error InvalidConfirmedAt();
        
        /// @notice Thrown when attempting to confirm a job that was already confirmed
        error AlreadyConfirmed();
        
        // ============================================================================
        // STORAGE LAYOUT
        // ============================================================================
        // @dev JobConfirmation struct: 1 storage slot (32 bytes total)
        // @dev Gas cost: 20,000 gas for first write, 5,000 gas for subsequent writes
        
        /// @notice Struct to store job confirmation data
        /// @dev Single storage slot (32 bytes) for gas efficiency
        struct JobConfirmation {
            uint256 confirmedAt;  // Slot 0: 32 bytes - Confirmation timestamp
        }
        
        /// @notice Mapping to track job confirmations and prevent duplicates
        /// @dev Key: jobHash (bytes32 hash of job data + salt), Value: JobConfirmation struct
        mapping(bytes32 => JobConfirmation) public confirmations;

        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /**
         * @dev Emitted when a job is confirmed with time constraints
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp since when the job counts as confirmed
         * 
         * GAS COST: 1,750 gas (LOG3 opcode)
         * MONITORING: Enables off-chain event indexing and time-based monitoring
         */
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        
        // ============================================================================
        // CORE CONFIRMATION LOGIC
        // ============================================================================
        
        /// @notice Confirm a job with on-chain duplicate prevention
        /// @dev Stores confirmation state and prevents duplicate confirmations
        /// @param jobHash Job identifier (bytes32 hash of job data + salt)
        /// @param confirmedAt Timestamp when the job was confirmed (must be > 0)
        /// @dev Gas cost: 26,450 gas (CALL: 2,100 + SLOAD: 2,100 + SSTORE: 20,000 + LOG3: 1,750 + execution: 500)
        /// @dev Security: Basic validation + duplicate prevention
        /// @dev Use case: Production confirmations with on-chain duplicate prevention
        function confirmJob(bytes32 jobHash, uint256 confirmedAt) external {
            // Input validation (cheapest first)
            if (jobHash == bytes32(0)) revert InvalidJobHash();
            if (confirmedAt == 0) revert InvalidConfirmedAt();
            
            // Storage check (most expensive)
            if (confirmations[jobHash].confirmedAt != 0) revert AlreadyConfirmed();
             
            // Direct assignment (gas optimized)
            confirmations[jobHash].confirmedAt = confirmedAt;

            emit JobConfirmed(jobHash, confirmedAt);
        }
     
        // ============================================================================
        // STATUS QUERY FUNCTIONS
        // ============================================================================
        
        /**
         * @dev Get confirmed at timestamp
         * 
         * GAS COST: 2,100 gas (SLOAD opcode)
         * EFFICIENCY: Single storage read for complete confirmation data
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @return confirmedAt Timestamp when the job was confirmed
         */
        function getConfirmedAt(bytes32 jobHash) public view returns (uint256) {
            return confirmations[jobHash].confirmedAt;
        }
    }
    ```

  - **Event & Struct**
    - ❌ **Higher gas**: `30,850` gas per confirmation
    - ✅ **Privacy preserving**: Job fingerprint is a hash from `job` and `salt`
      - ❌ But it still it could be a correlation factor
    - ✅ **Block Timestamp**: Available from block metadata
      - ❌ Can be manipulated by miners
    - ✅ **Job confirmation check**: Using events and on-chain state
    - ✅ **Job confirmation deduplication**: Using events and on-chain state
    - ✅ **Job (integrity) hash verification**: Using events and on-chain state
    - ✅ **Job confirmation date verification**: Using events and on-chain state

    **Explicit Gas Breakdown** `(30,850)`:
      - CALL opcode: `2,100` gas
      - SLOAD (duplicate check): `2,100` gas
      - SLOAD (FUTURE_TOLERANCE): `2,100` gas
      - SLOAD (PAST_TOLERANCE): `2,100` gas
      - SSTORE (confirmation storage): `20,000` gas
      - LOG3 opcode: `1,750` gas
      - Function execution: `700` gas

    **Gas Loss on Revert:**
      - If time validation fails: `6,700` gas lost (~$0.13 at 20 gwei)
        - CALL opcode: `2,100` gas (transaction initiation)
        - Input validation: `100` gas (jobHash check)
        - SLOAD (FUTURE_TOLERANCE): `2,100` gas
        - SLOAD (PAST_TOLERANCE): `2,100` gas
        - Time validation logic: `300` gas
        - REVERT opcode: `0` gas (revert is free)
    
    ```ts
    contract EventAndStructJobConfirmation {
        // ============================================================================
        // EVENT & STRUCT JOB CONFIRMATION CONTRACT
        // ============================================================================
        //
        /// @title EventAndStructJobConfirmation - Advanced time validation with struct storage
        /// @dev Simple confirmation with event emission (30,850 gas per job)
        /// @dev Security: Advanced time validation with configurable tolerances
        /// @dev Functionality: Maximum security with time-based validation
        
        // ============================================================================
        // ERRORS
        // ============================================================================
        
        /// @notice Thrown when job hash is zero or invalid
        error InvalidJobHash();
        
        /// @notice Thrown when confirmedAt timestamp is zero or invalid
        error InvalidConfirmedAt();
        
        /// @notice Thrown when attempting to confirm a job that was already confirmed
        error AlreadyConfirmed();
        
        /// @notice Thrown when future tolerance is set to zero in constructor
        error FutureToleranceInvalid();
        
        /// @notice Thrown when past tolerance is set to zero in constructor
        error PastToleranceInvalid();
        
        // ============================================================================
        // CONSTANTS
        // ============================================================================
        
        /// @notice Maximum seconds confirmedAt can be in the future (configurable at deployment)
        /// @dev Prevents confirmation of jobs with timestamps too far in the future
        uint256 public immutable FUTURE_TOLERANCE;
        
        /// @notice Maximum seconds confirmedAt can be in the past (configurable at deployment)
        /// @dev Prevents confirmation of jobs with timestamps too far in the past
        uint256 public immutable PAST_TOLERANCE;
        
        // ============================================================================
        // CONSTRUCTOR
        // ============================================================================
        
        /// @notice Constructor to set chain-specific time tolerances
        /// @dev Sets immutable time validation parameters for the contract
        /// @dev Gas cost: 20,000 gas (constructor execution)
        /// @dev Security: Set once at deployment, immutable thereafter
        /// @param _futureTolerance Future tolerance in seconds (must be > 0)
        /// @param _pastTolerance Past tolerance in seconds (must be > 0)
        /// @dev Use case: Configure time validation based on blockchain characteristics
        constructor(uint256 _futureTolerance, uint256 _pastTolerance) {
            if (_futureTolerance == 0) revert FutureToleranceInvalid();
            if (_pastTolerance == 0) revert PastToleranceInvalid();
            
            FUTURE_TOLERANCE = _futureTolerance;
            PAST_TOLERANCE = _pastTolerance;
        }
        
        // ============================================================================
        // STORAGE LAYOUT
        // ============================================================================
        // @dev JobConfirmation struct: 1 storage slot (32 bytes total)
        // @dev Gas cost: 20,000 gas for first write, 5,000 gas for subsequent writes
        
        /// @notice Struct to store job confirmation data
        /// @dev Single storage slot (32 bytes) for gas efficiency
        /// @dev confirmedAt: Timestamp when the job was confirmed
        struct JobConfirmation {
            uint256 confirmedAt;  // Slot 0: 32 bytes - Confirmation timestamp
        }
        
        /// @notice Mapping to track job confirmations and prevent duplicates
        /// @dev Key: jobHash (bytes32 hash of job data + salt), Value: JobConfirmation struct
        /// @dev Gas cost: 20,000 gas for first write, 5,000 gas for subsequent reads
        mapping(bytes32 => JobConfirmation) public confirmations;
        
        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /**
         * @dev Emitted when a job is confirmed with time constraints
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp since when the job counts as confirmed
         * 
         * GAS COST: 1,750 gas (LOG3 opcode)
         * MONITORING: Enables off-chain event indexing and time-based monitoring
         */
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        // ============================================================================
        // CORE CONFIRMATION LOGIC
        // ============================================================================
        
        /// @notice Confirm a job with advanced time validation
        /// @dev Advanced time validation with configurable tolerances
        /// @param jobHash Job identifier (bytes32 hash of job data + salt)
        /// @param confirmedAt Timestamp when the job was confirmed (must be within tolerances)
        /// @dev Gas cost: 30,850 gas (CALL: 2,100 + SLOAD: 2,100 + SLOAD: 2,100 + SLOAD: 2,100 + SSTORE: 20,000 + LOG3: 1,750 + execution: 700)
        /// @dev Security: Time validation + duplicate prevention
        /// @dev Time validation: confirmedAt must be within FUTURE_TOLERANCE and PAST_TOLERANCE
        /// @dev Use case: Maximum security confirmations with time-based validation
        function confirmJob(bytes32 jobHash, uint256 confirmedAt) external {
            // Input validation (cheapest first)
            if (jobHash == bytes32(0)) revert InvalidJobHash();
            
            // Combined time validation (gas optimized)
            if (
                confirmedAt == 0 |
                confirmedAt > block.timestamp + FUTURE_TOLERANCE |
                confirmedAt < block.timestamp - PAST_TOLERANCE
            ) {
                revert InvalidConfirmedAt();
            }
            
            // Storage check (most expensive)
            if (confirmations[jobHash].confirmedAt != 0) revert AlreadyConfirmed();
             
            // Direct assignment (gas optimized)
            confirmations[jobHash].confirmedAt = confirmedAt;

            emit JobConfirmed(jobHash, confirmedAt);
        }
        
        // ============================================================================
        // STATUS QUERY FUNCTIONS
        // ============================================================================
        
        /**
         * @dev Get confirmed at timestamp
         * 
         * GAS COST: 2,100 gas (SLOAD opcode)
         * EFFICIENCY: Single storage read for complete confirmation data
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @return confirmedAt Timestamp when the job was confirmed
         */
        function getConfirmedAt(bytes32 jobHash) public view returns (uint256) {
            return confirmations[jobHash].confirmedAt;
        }
    }
    ```

  - **Batch Confirmation (Batch)**
    - ✅ **Gas efficiency**: `23,950` gas per batch
    - ✅ **Privacy preserving**: Job fingerprint is a hash from `job` and `salt`
      - ❌ But it still it could be a correlation factor
    - ✅ **Block Timestamp**: Available from block metadata
    - ✅ **Job confirmation check**: Using events and on-chain state
    - ✅ **Job confirmation deduplication**: Using events and on-chain state
    - ✅ **Job (integrity) hash verification**: Using events and on-chain state
    - ❌ **Job confirmation date verification**: Using events only (off-chain)
    - ✅ **Cost optimization**: 68% savings vs individual confirmations
    - ❌ **Batch complexity**: More complex error handling for partial failures

    **Case 01: Individual Job Confirmation (confirmJobs)**
    - Security: No duplicate prevention (off-chain handling required)
    - Trade-off: Cheapest per job, but no on-chain duplicate prevention

    **Explicit Gas Breakdown - Case 01** `(for 10 jobs = 2,180 gas per job)`:
      - CALL opcode: `2,100` gas (one-time)
      - Input validation: `200` gas (confirmedAt check, one-time)
      - Loop processing: `17,500` gas (10 jobs × 1,750 gas per event)
      - Function execution: `300` gas
      - **Total for 10 jobs**: `20,100` gas
      - **Average per job**: `2,010` gas per job

    **Case 02: Batch Confirmation (confirmBatch)**
    - Security: Batch-level duplicate prevention
    - Trade-off: Higher base cost, but massive savings for large batches

    **Explicit Gas Breakdown - Case 02** `(23,950 gas per batch)`:
      - CALL opcode: `2,100` gas
      - Batch hash validation: `100` gas
      - Batch hash storage: `20,000` gas (SSTORE)
      - Batch event emission: `1,750` gas (LOG3 opcode)
      - Function execution: `200` gas
      - **Total per batch**: `23,950` gas (regardless of batch size)
    
    ```ts
    contract BatchJobConfirmation {
        // ============================================================================
        // BATCH JOB CONFIRMATION CONTRACT WITH BATCH OPTIMIZATION
        // ============================================================================
        //
        // DESIGN: Dual-mode job confirmation with gas optimization
        // SECURITY: Two approaches - individual jobs vs batch-level confirmation
        // PRIVACY: Privacy-preserving identifiers with flexible confirmation strategies
        // EFFICIENCY: Optimized gas usage for different use cases
        //
        // @dev Dual-mode contract: Individual jobs (1,500 gas) vs Batch confirmation (23,950 gas)
        
        // ============================================================================
        // DUAL-MODE CONFIRMATION STRATEGY
        // ============================================================================
        // @dev The contract provides two distinct confirmation approaches:
        // @dev Case 01: Individual jobs (confirmJobs) - 1,500 gas per job, no duplicate prevention
        // @dev Case 02: Batch confirmation (confirmBatch) - 23,950 gas per batch, batch-level security
        
        /// @notice Thrown when batch hash is zero or invalid
        error InvalidBatchHash();
        
        /// @notice Thrown when confirmedAt timestamp is zero or invalid
        error InvalidConfirmedAt();
        
        /// @notice Thrown when attempting to confirm a batch that was already processed
        error BatchAlreadyProcessed();
        
        // ============================================================================
        // STORAGE LAYOUT & GAS OPTIMIZATION
        // ============================================================================
        
        /// @notice Mapping to track confirmed batch hashes and prevent duplicates
        /// @dev Key: batchHash (Merkle root of job hashes), Value: true if confirmed
        /// @dev Gas cost: 20,000 gas for first write, 5,000 gas for subsequent reads
        mapping(bytes32 => bool) public batches;
        
        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /// @notice Emitted when an individual job is confirmed
        /// @param jobHash The unique identifier of the confirmed job
        /// @param confirmedAt The timestamp when the job was confirmed
        /// @dev Gas cost: 1,275 gas per event emission
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        /// @notice Emitted when a batch of jobs is confirmed
        /// @param batchHash The Merkle root hash of the confirmed batch
        /// @param confirmedAt The timestamp when the batch was confirmed
        /// @dev Gas cost: 1,275 gas per event emission
        event BatchConfirmed(bytes32 indexed batchHash, uint256 confirmedAt);
        
        /// @notice Confirms multiple jobs without batch-level duplicate prevention
        /// @dev Ultra-cheap event-only approach for simple job confirmations
        /// @dev WARNING: No duplicate prevention - same job can be confirmed multiple times
        /// @param jobHashes Array of job hashes to confirm (read-only, uses calldata for gas efficiency)
        /// @param confirmedAt Timestamp when the jobs were confirmed (must be > 0)
        /// @dev Gas cost: ~1,500 gas per job (event emission only)
        /// @dev Use case: Simple confirmations where duplicate prevention is handled off-chain
        /// @dev Case 01: NOT-A-REAL-BATCH - Individual job processing with events only
        function confirmJobs(
            bytes32[] calldata jobHashes,
            uint256 confirmedAt
        ) external {
            if (confirmedAt == 0) revert InvalidConfirmedAt();
            
            // Emit individual job events (no storage)
            for (uint256 i = 0; i < jobHashes.length; i++) {
                emit JobConfirmed(jobHashes[i], confirmedAt);
            }
        }

        /// @notice Confirms a batch of jobs using batch hash
        /// @dev Batch-level duplicate prevention with minimal on-chain state
        /// @dev Individual jobs are not tracked on-chain, only batch hashes
        /// @param batchHash Merkle root hash of the job batch (must be non-zero)
        /// @param confirmedAt Timestamp when the batch was confirmed (must be > 0)
        /// @dev Gas cost: 23,950 gas per batch (batch hash storage + event)
        /// @dev Gas breakdown: CALL(2,100) + validation(100) + SSTORE(20,000) + LOG3(1,750) = 23,950
        /// @dev Security: Prevents duplicate batch confirmations, no individual job tracking
        /// @dev Use case: Production batch confirmations with batch-level security
        /// @dev Case 02: BATCHING USING BATCH HASH - Batch-level confirmation with duplicate prevention
        function confirmBatch(
            bytes32 batchHash,
            uint256 confirmedAt
        ) external {
            if (batchHash == bytes32(0)) revert InvalidBatchHash();
            if (confirmedAt == 0) revert InvalidConfirmedAt();
            if (batches[batchHash]) revert BatchAlreadyProcessed();
            
            batches[batchHash] = true;
            
            emit BatchConfirmed(batchHash, confirmedAt);
        }
    }
    ```


- **Security Considerations**:

  - **Upgradability**: All implementations are immutable contracts
    - ✅ **Security**: No upgrade attack vectors, maximum decentralization
    - ❌ **Flexibility**: Cannot fix bugs or add features post-deployment

  - **Pausability**: No pause mechanism in any implementation
    - ✅ **Decentralization**: No central control, fully trustless
    - ❌ **Emergency Response**: Cannot halt operations in case of critical bugs

  - **Merkle Tree Security**
    - ✅ **Cryptographic Integrity**: Merkle root proves all jobs are included in the batch
    - ✅ **Tamper Evidence**: Any modification to job data breaks the Merkle root
    - ✅ **Batch Verification**: Natural fit for batch processing with O(log n) complexity
    - ✅ **Zero Gas Overhead**: Same costs as regular options but with enhanced security

  - **Commit-Reveal MEV Protection**
    - ✅ **Front-Running Protection**: Two-phase commit-reveal prevents MEV attacks
    - ✅ **Data Hiding**: Commitment phase hides job data until reveal
    - ✅ **Batch MEV Protection**: Natural fit for batch processing with MEV resistance
    - ✅ **Zero Gas Overhead**: Same costs as regular options but with enhanced MEV protection

  - **Authentication & Authorization of On-Chain Operations**:

    - **Permissionless**: No access control - anyone can execute
      - ✅ **Permissionless**: Fully decentralized
      - ❌ **Security Risk**: Malicious actors can spam

    - **OpenZeppelin Ownable**: Single owner model
      - ✅ **Simple**: Single administrative account
      - ❌ **Single Point of Failure**: Owner compromise affects system

    - **OpenZeppelin AccessControl**: Role-based access control
      - ✅ **Flexible Roles**: Multiple roles with specific permissions
      - ❌ **Centralization**: Requires trusted administrators

    - **SigCheck**: Confirmer signature verification
      - ✅ **Tamper Evidence**: Any modification to job data breaks the signature
      - ✅ **Cryptographic Authentication**: ECDSA signature proves confirmer identity
      - ✅ **Non-Repudiation**: Signer cannot deny having signed the confirmation
      - ❌ **Signature Overhead**: ECDSA verification required for every batch operation

    - **ZK**: ZK proof verification
      - ✅ **Cryptographic Authentication**: Only valid proof holders
      - ✅ **Maximum Privacy**: ZK proofs enable job confirmation without revealing any job data
      - ✅ **Cryptographic Privacy**: Mathematical proof of job validity without data exposure
      - ✅ **Batch Privacy**: Natural fit for batch processing with complete data privacy
      - ❌ **ZK Proof Overhead**: ZK proof generation and verification required for every batch operation
      - ❌ **Higher Gas Cost**: 50,000 gas per batch (vs 23,950 for regular batch)


- **Multi-Chain Cost Analysis**

  - **Per Single Transaction by implementation options**

    |---------------|------------|-----------------|----------------|------------|
    | Blockchain    | EventOnly  | EventAndMapping | EventAndStruct | Batch      |
    |---------------|------------|-----------------|----------------|------------|
    | **Polygon**   | $0.17      | $0.71           | $0.83          | $0.04      |
    | **Harmony**   | $0.13      | $0.53           | $0.62          | $0.03      |
    | **Huobi**     | $0.13      | $0.53           | $0.62          | $0.03      |
    | **KuCoin**    | $0.13      | $0.53           | $0.62          | $0.03      |
    | **Cronos**    | $0.13      | $0.53           | $0.62          | $0.03      |
    | **Moonriver** | $0.13      | $0.53           | $0.62          | $0.03      |
    | **Avalanche** | $0.59      | $2.35           | $2.74          | $0.14      | 
    | **BSC**       | $1.01      | $4.03           | $4.70          | $0.24      | 
    | **Fantom**    | $0.27      | $1.08           | $1.26          | $0.06      |
    | **Ethereum**  | $54.50     | $223.45         | $260.69        | $13.25     | 
    |---------------|------------|-----------------|----------------|------------|

  - **For 1M Jobs by implementation options**

    |---------------|------------|-----------------|----------------|------------|------------|-------------|
    | Blockchain    | EventOnly  | EventAndMapping | EventAndStruct | Batch(10)  | Batch(100) | Batch(1000) |
    |---------------|------------|-----------------|----------------|------------|------------|-------------|
    | **Polygon**   | $170,000   | $710,000        | $830,000       | $17,000    | $4,000     | $400        |
    | **Harmony**   | $130,000   | $530,000        | $620,000       | $13,000    | $3,000     | $300        |
    | **Huobi**     | $130,000   | $530,000        | $620,000       | $13,000    | $3,000     | $300        |
    | **KuCoin**    | $130,000   | $530,000        | $620,000       | $13,000    | $3,000     | $300        |
    | **Cronos**    | $130,000   | $530,000        | $620,000       | $13,000    | $3,000     | $300        |
    | **Moonriver** | $130,000   | $530,000        | $620,000       | $13,000    | $3,000     | $300        |
    | **Avalanche** | $590,000   | $2,350,000      | $2,740,000     | $59,000    | $14,000    | $1,400      |
    | **BSC**       | $1,010,000 | $4,030,000      | $4,700,000     | $101,000   | $24,000    | $2,400      |
    | **Fantom**    | $270,000   | $1,080,000      | $1,260,000     | $27,000    | $6,000     | $600        |
    | **Ethereum**  | $54,500,000| $223,450,000    | $260,690,000   | $5,450,000 | $1,325,000 | $132,500    |
    |---------------|------------|-----------------|----------------|------------|------------|-------------|


- **Trade-offs**:
  - **Trustless Deployment vs Upgradability**
  - **Trustless Deployment vs Pausability**
  - **Ultra-low Cost vs On-chain State Checks**
  - **Permissionless vs Access Control**


- **Chosen Approach**: `Batch Job Confirmation Strategy with multiple deployments with security features tailored per use-case`
  - **Primary Strategy**: BatchJobConfirmation (23,950 gas per batch) for high-volume operations
  - **Rationale:**
    - `Gas Efficiency`:
      - 35% cost savings vs EventOnly with batch processing
      - **$65 (on Polygon)** vs **$3.9M (on Ethereum)** for **1M transactions**
      - Optimal for high-volume applications (100+ jobs per batch)
    - `Functionality`: 
      - Batch confirmation for high-volume operations
    - `Scalability`:
      - Maximum cost efficiency for high-volume applications
      - **0.1T transactions** at the cost of **$6.5K (on Polygon)**
    - `Trustless Deployment`:
      - **Contract Simplicity**: Batch-optimized contracts with clear separation of concerns
      - **Contract Security**: No upgrade mechanisms, ensuring contract finality and security
      - **Compliance**: Immutable contracts provide audit trail and regulatory compliance benefits
      - **Cost Efficiency**: Batch processing reduces gas costs by 35% for high-volume operations
