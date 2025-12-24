# Major cryptocurrency Address Formats and Key Features


| Coin/Blockchain | Version Name | Account/Address Model | Address Format | Address Prefix/Example | Key Features / Notes |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Bitcoin (BTC) | Native SegWit (P2WPKH) | UTXO-Based | Bech32 (BIP 173\) | Starts with bc1q | Lowest fees, native SegWit. |
| Bitcoin (BTC) | Taproot | UTXO-Based | Bech32m (BIP 350\) | Starts with bc1p | Enhanced privacy, most modern format. |
| Ethereum (ETH) | Standard | Account-Based | Hexadecimal | Starts with 0x | Used for ETH and all ERC-20 tokens. |
| Cardano (ADA) | Shelley (New) | Hybrid (UTXO-Like) | Bech32 (Custom) | Starts with addr1 | Supports staking and multi-asset capabilities. |
| Cardano (ADA) | Byron (Legacy) | UTXO-Based | Base58Check | Starts with Ae2 or DdzFF | Older, less feature-rich format. |
| Solana (SOL) | Standard | Account-Based | Base58 (Custom) | No specific prefix (32–44 chars) | Stores everything (code, data, tokens) as accounts. Case-sensitive. |
| XRP (XRP) | Classic | Account-Based | Base58Check (XRP variant) | Starts with r | Requires a Destination Tag for exchange deposits. |
| XRP (XRP) | X-Address | Account-Based | Base58Check (XRP variant) | Starts with X | Combines the classic address and the Destination Tag into one string. |
| Polkadot (DOT) | Standard | Account-Based | SS58 (Base58 variant) | No single prefix (varies by network) | Unified address format across the Polkadot ecosystem (Parachains). |
| NEAR | Named Account | Account-Based | Human-Readable | e.g., alice.near | Domain-name-like, easy-to-read addresses. |
| NEAR | Implicit Account | Account-Based | Hexadecimal | 64 characters long | Derived directly from the public key. |


# UTXO vs Account Model

UTXO (Unspent Transaction Output) Model: Used by Bitcoin, Litecoin, Dogecoin, etc. Funds are stored as discrete "outputs" which are consumed entirely and new outputs are created (like physical cash). This is where Script Types (P2PKH, P2SH, P2WPKH) are essential for locking/unlocking the funds.

Account Model: Used by Ethereum. Funds are stored in a single balance for each address, and transactions modify that balance (like a bank account). This is why there are no complex script types in the same way, as the address itself is the destination.

# Encoding Scheme

Base58Check: The traditional encoding for older addresses (starts with 1, 3, L, D, etc.). It's case-sensitive and omits confusing characters (like 0, O, I, l).

Bech32 / Bech32m (BIP 173/350): The modern, native SegWit encoding (starts with bc1q, bc1p, ltc1q). It is case-insensitive (only uses lowercase), offers better error detection, and is slightly more space-efficient

# Explaining the Different Script Types (UTXO Model)
For coins like Bitcoin and Litecoin that use the UTXO (Unspent Transaction Output) Model, the "script type" is the rule set written on the blockchain that determines how a coin can be spent.

## P2PKH (Pay-to-Public-Key-Hash)

The Rule: To unlock the funds, the spender must provide a Public Key and a valid Signature that proves they own the corresponding Private Key.

Analogy: A simple lockbox where the key holder's name (the public key hash) is on the outside, and you just need to open it with the right key (the private key/signature).

Address: Starts with 1 (Bitcoin Legacy).

## P2SH (Pay-to-Script-Hash)

The Rule: To unlock the funds, the spender must provide a Script that hashes to the value locked in the output, AND they must provide the Signature(s) that successfully run that script.

Analogy: A more complex lockbox with an internal combination lock. You don't see the combination (the full script) until you open the first lock, but you need to know the combination to successfully open the box.

Address: Starts with 3 (Bitcoin Legacy/Nested SegWit). This is commonly used for multi-signature wallets (e.g., "2-of-3") or to nest a SegWit script (P2SH-P2WPKH), which is how Nested SegWit works.

## P2WPKH (Pay-to-Witness-Public-Key-Hash)

The Rule: Same rule as P2PKH (Public Key + Signature), but the data (the "witness") is moved to a separate part of the transaction block (the "witness" data).

Analogy: The lockbox is opened just like P2PKH, but the transaction record itself is lighter because the signature/public key data is stored in an optional, discounted area of the block.

Address: Starts with bc1q (Native SegWit/Bech32).

# The Core Concept: Locking, Not Sending
In Bitcoin, you don't actually send a currency amount to an address. Instead, you create a new transaction output that is locked with a specific Script (the rule set) and points to a previous, unspent output (UTXO).

##  1. How Funds Are Locked (The Locking Script: scriptPubKey)
When a Bitcoin user wants to "send" money, they are actually creating a new transaction output (UTXO) on the blockchain. This output contains two main parts:Value: The amount of Bitcoin being transferred.Locking Script (scriptPubKey): This is the puzzle or the set of conditions that must be satisfied to spend the funds. This script specifies the necessary public key hash (or a more complex script hash) that the spender must prove ownership of.
Analogy: The scriptPubKey is like a safe deposit box on the blockchain with a required combination (the public key hash) written on the outside. Only the person who knows the combination (the private key) can create the key to open it.For a standard P2PKH (Legacy) transaction, the scriptPubKey essentially says:"To spend this money, you must provide a valid Signature AND a Public Key that hashes to this specific Public Key Hash."

## 2. How Funds Are Unlocked (The Unlocking Script: scriptSig)
To "receive" and then spend the money, the new spender (the "receiver" from the previous transaction) creates a new transaction input that references that specific locked output (the UTXO).

The input must contain the Unlocking Script (scriptSig) which provides the solution to the locking script's puzzle.
scriptSig (The Solution): This script contains two key pieces of data:

The Signature: Created by the spender's Private Key over the details of the new transaction. This proves ownership.
The Public Key: The public key corresponding to the private key used for the signature.

Verification (The Execution): The Bitcoin network's nodes take the two scripts—the scriptSig (Solution) and the scriptPubKey (Puzzle)—and run them together in order through the Bitcoin Script virtual machine (stack-based language).

The scriptSig pushes the signature and public key onto the stack.

The scriptPubKey uses these items to perform cryptographic checks (like hashing the public key and verifying the signature).

Success Condition: If the combined script executes successfully (meaning it returns a value of "True"), the output is unlocked, and the funds can be spent in the new transaction.
Example of P2PKH Script Flow (Simplified)Here's a step-by-step look at what happens when the two scripts are run:StepScript ExecutingActionResult on StackInput (scriptSig)[Signature]Pushes the signature onto the stack.
[Signature]Input (scriptSig)[Public Key]Pushes the public key onto the stack.
[Signature], [Public Key]Output (scriptPubKey)OP_DUPDuplicates the Public Key.
[Sig], [PK], [PK]Output (scriptPubKey)OP_HASH160Hashes the top Public Key.
[Sig], [PK], [PK_Hash]Output (scriptPubKey)[PK Hash (Target)]Pushes the original required hash.
[Sig], [PK], [PK_Hash], [PK_Hash (Target)]Output (scriptPubKey)OP_EQUALVERIFYCompares the two hashes. If not equal, script fails.
[Signature], [Public Key] (if successful)Output (scriptPubKey)OP_CHECKSIGUses the Signature and Public Key to verify the transaction details. If successful, returns TRUE.
[TRUE]If the script returns TRUE, the transaction is valid, and the funds are unlocked!The SegWit Difference (P2WPKH)For Native SegWit (P2WPKH), the process is the same, but the Signature and Public Key (the Witness data) are stored in the separate, cheaper-to-send witness field of the transaction, rather than the scriptSig. 
This is what gives SegWit its fee reduction benefit.
