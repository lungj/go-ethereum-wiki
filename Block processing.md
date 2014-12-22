This is a brief description on how block processing works (e.g., block 20 => block 21 => etc)

Block processing starts in the `ChainManager` (https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go)

* Block(s) are inserted using `InsertChain(...)` (https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go#L269)
* Each block is then processed by the `Processor` which in our case is always the `BlockManager` https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go#L271
    * Block validation and processing all happens with the `BlockManager` https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go
    * Checked if the block already's in storage (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L171)
    * Check if we have the parent (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L175)
    * Validate the easy, less intensive things first (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L195)
        * Calculate the total difficulty and compare it to the parent's TD (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L276)
        * Verify the proof of work (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L293)
    * Transition the state (e.g., apply transactions) (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L199)
    * Give coin base block gas limit which transactions use as gas pool to buy gas off (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L88)
    * Create a new state transition object which will help us with the transition (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L117)
        * All transactional state changes happen within the `StateTransition` object and start at the `TransitionState` method (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L135)
        * Check the tx nonce: `tx.nonce == account.nonce` (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L123)
        * Check if we can buy the gas of the coin base (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L139)
        * Use TxGas (500) (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L154)
        * Create a contract if address = 0x0 * 32 (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L174)
        * OR run the code if the recipient has any code to execute (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L180)
             * Create `VmEnv` and use that as the Virtual machine's environment for creating or executing now contracts (https://github.com/ethereum/go-ethereum/blob/docbranch/core/vm_env.go)
             * Create `Execution` object which takes care of the actual Vm instantiation (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L35)
                 * Get the recipient and sender (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L39)
                 * Transfer value (err is set incase of insufficient funds) (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L42)
                 * Snapshot the state for rollback feature (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L51)
                 * `defer` checking for depth and oog errors so that we may rollback to the previous snapshot (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L52)
                 * Check for precompiled contract (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L61) see (https://github.com/ethereum/go-ethereum/blob/docbranch/vm/address.go#L23) for all pre compiled contracts.
                 * Execute code given (https://github.com/ethereum/go-ethereum/blob/docbranch/core/execution.go#L69)
                     * Enter VM run loop (https://github.com/ethereum/go-ethereum/blob/docbranch/vm/vm_debug.go#L39)
                     * Create a new closure which holds the code, gas and information about the caller (https://github.com/ethereum/go-ethereum/blob/docbranch/vm/vm_debug.go#L49)
                     * Analyse valid jump positions (https://github.com/ethereum/go-ethereum/blob/docbranch/vm/vm_debug.go#L75)
                     * it's worth noting that `jump` (https://github.com/ethereum/go-ethereum/blob/docbranch/vm/vm_debug.go#L88) uses the jump positions in order to determine whether the `JUMP` landed on a valid position.
                     * .... Run code .... (evaluation of code is a different topic)
                     * Eventually the execution will end, be it a success or a failure (stack err, OOG, depth level, etc) and return a value and whether it was successful.
    * Evaluate outcome and set the code accordingly (if action was create) (https://github.com/ethereum/go-ethereum/blob/docbranch/core/state_transition.go#L178)
    * Update the state changes made during the state transition (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L140)
    * Create a new receipt for validity checks at the end of the transaction processing (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L143)
    * Validate the receipts by creating a new bloom filter out of it and compare it against the received block's bloom (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L204)
    * Validate the transaction hash (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L210)
    * Validate the receipt hash (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L216)
    * Accumulate the reward for the miner (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L222)
        * Validate the uncle's parent (must be in chain) (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L311)
        * Validate the uncle's block number (not greater than 6) (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L316)
        * Validate the uncle's not includes multiple times (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L320)
        * Reward miner (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L338)
    * Update the state with _all_ changes and validate the merkle root (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L228)
    * Calculate the total difficulty of the block (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L234)
    * Sync the state to the db (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L236)
    * Return the total difficulty if the process was a success (https://github.com/ethereum/go-ethereum/blob/docbranch/core/block_manager.go#L245)
* Write block to the database (https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go#L292)
* Compare the total difficulty of the process block to our current known TD (https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go#L293) if greater than:
    * Set new TD (https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go#L298)
    * Insert in to the canonical chain by setting it as current block (https://github.com/ethereum/go-ethereum/blob/docbranch/core/chain_manager.go#L292)

## fin     