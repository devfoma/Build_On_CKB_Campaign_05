# Build on CKB — Campaign 05: Create a Fungible Token (xUDT)

**Author:** Maduegbunam Faith Amarachi (devfoma)  
**Campaign:** Build On CKB Campaign 05  
**Tutorial Reference:** [Create a Fungible Token (Nervos Docs)](https://docs.nervos.org/docs/dapp/create-token)  
**SDK & Tooling:** `@offckb/cli` v0.4.6 | CKB Node v0.205.0 | `@ckb-ccc/core` v1.5.3 | Node.js v24.15.0  

---

## Executive Summary & Quest Checklist

| Quest | Requirement | Status | Key Artifact / Tx Hash |
|---|---|---|---|
| **Quest 1** | Run OffCKB Devnet & Dapp Example | ✅ Complete | Live at `http://localhost:1234` ([Screenshot 04](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/04-dapp-running.png)) |
| **Quest 2** | Create a Custom xUDT Token | ✅ Complete | Tx: `0xf4d44bca1c3edbbf177d6e59ce886b0253a93b7898cf26e1d71a9d1eee6a7b6c` ([Screenshot 05](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/05-issue-token.png)) |
| **Quest 3** | Query Token Cell by Issuer Lock Hash | ✅ Complete | 42 tokens found in Cell #0 ([Screenshot 06](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/06-query-by-lockhash.png)) |
| **Quest 4** | Transfer Tokens by Replacing Lock Script | ✅ Complete | Tx: `0x6f01edf9c6a47639bd4211ac47c2fc3ff7b663ab08cb467e7e3611536970c06c` ([Screenshot 07](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/07-transfer.png)) |
| **Post-Check** | Verify Receiver & Change Cells | ✅ Complete | Recipient: 10 tokens / Sender Change: 32 tokens ([Screenshot 08](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/08-after-transfer.png)) |

---

## 1. Environment Setup & Configuration

### My Tooling Versions
- **OS:** Windows 11 (x64)
- **Node.js:** `v24.15.0`
- **npm:** `11.12.1`
- **Git:** `2.53.0.windows.3`
- **OffCKB CLI:** `0.4.6`
- **CKB Devnet Binary:** `0.205.0` (automatically provisioned by OffCKB)

### Starting My Devnet
I launched my local standalone CKB node using `offckb node`. It produced blocks every ~2000ms, exposing:
- **Proxy RPC:** `http://127.0.0.1:28114` (used by the CCC client)
- **Direct RPC:** `http://127.0.0.1:8114`

```
PS > offckb --version
0.4.6
```
[![OffCKB Version](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/01-offckb-version.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/01-offckb-version.png)

```
PS > offckb node
Launching CKB devnet Node...
CKB devnet RPC Proxy server running on http://127.0.0.1:28114
Mining block every 2000ms...
```
[![OffCKB Node Devnet](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/02-devnet-running.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/02-devnet-running.png)

### The Devnet Accounts I Used

#### Account #0 (My Issuer & Sender)
- **Address:** `ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqvwg2cen8extgq8s5puft8vf40px3f599cytcyd8`
- **Private Key:** `0x6109170b275a09ad54877b82f7d9930f88cab5717d484fb4741ae9d1dd078cd6`
- **Lock Script Code Hash:** `0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8`
- **Lock Script Args:** `0x8e42b1999f265a0078503c4acec4d5e134534297`
- **Lock Script Hash:** `0x7de82d61a7eb2ec82b0dc653e558ba120efcbfbb44dac87c12972d05bf250653`

#### Account #1 (My Token Receiver)
- **Address:** `ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqt435c3epyrupszm7khk6weq5lrlyt52lg48ucew`
- **Private Key:** `0x9f315d5a9618a39fdc487c7a67a8581d40b045bd7a42d83648ca80ef3b2cb4a1`
- **Lock Script Args:** `0x758d311c8483e0602dfad7b69d9053e3f917457d`

[![OffCKB Accounts](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/03-accounts.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/03-accounts.png)

---

## 2. Dapp Example Execution & Proofs

### Proof 1: Running the Dapp Example
I installed dependencies and launched the dApp frontend using Parcel configured with `NETWORK=devnet`. The web app connects directly to my local OffCKB devnet proxy and automatically populated Account #0's initial capacity of **42,000,000 CKB**.
- **URL:** `http://localhost:1234`

[![dApp Running](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/04-dapp-running.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/04-dapp-running.png)

---

### Proof 2: Creating My Custom xUDT Token
Using Account #0 as the issuer, I minted 42 units of my custom xUDT token.

- **Issuance Transaction Hash:**  
  `0xf4d44bca1c3edbbf177d6e59ce886b0253a93b7898cf26e1d71a9d1eee6a7b6c`
- **Token xUDT Args:**  
  `0x7de82d61a7eb2ec82b0dc653e558ba120efcbfbb44dac87c12972d05bf25065300000000`
- **Structure of My Token Args:**
  - `0x7de82d61a7eb2ec82b0dc653e558ba120efcbfbb44dac87c12972d05bf250653` (32 bytes: my Issuer Lock Script Hash)
  - `00000000` (4 bytes: 32-bit flags specifying extensions, `00000000` = standard sUDT compatibility mode)
- **Token Cell Capacity:** `146 CKB` (`0x3663a5200` Shannon)
- **Token Data:** `0x2a000000000000000000000000000000` (42 in u128 little-endian)

[![Issue Custom Token](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/05-issue-token.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/05-issue-token.png)

---

### Proof 3: Querying My Custom Token Cell by Issuer Lock Script Hash
I copied the generated xUDT args (derived from my Issuer Lock Script Hash) and queried the cell collector (`findCellsByType`):

- **Query Param (xUDT Args):**  
  `0x7de82d61a7eb2ec82b0dc653e558ba120efcbfbb44dac87c12972d05bf25065300000000`
- **Result:** I found Cell #0 on-chain:
  - **Token Amount:** `42`
  - **Holder Lock Script Args:** `0x8e42b1999f265a0078503c4acec4d5e134534297` (matches my Issuer Account #0)

[![Query Token Cell](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/06-query-by-lockhash.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/06-query-by-lockhash.png)

---

### Proof 4: Transferring Tokens to Another Account by Replacing the Lock Script
I initiated a transfer of 10 tokens from Account #0 to Account #1 (`ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqt435c3epyrupszm7khk6weq5lrlyt52lg48ucew`).

- **Transfer Transaction Hash:**  
  `0x6f01edf9c6a47639bd4211ac47c2fc3ff7b663ab08cb467e7e3611536970c06c`
- **Transaction Mechanics:**
  - **Input 0:** Consumed my 42-token Cell (`0xf4d44bca...:0`)
  - **Output 0 (Recipient Cell):** 10 tokens, where I swapped the Lock Script to Account #1 (`args: 0x758d311c8483e0602dfad7b69d9053e3f917457d`), while keeping the xUDT Type Script identical.
  - **Output 1 (My Change Cell):** 32 tokens, kept locked to Account #0 (`args: 0x8e42b1999f265a0078503c4acec4d5e134534297`), with the same xUDT Type Script.
  - **Output 2 (Capacity Change Cell):** Returned remaining CKB capacity minus the transaction fee.

[![Transfer Custom Token](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/07-transfer.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/07-transfer.png)

---

### Proof 5: Verifying Balances After Transfer
I re-queried the xUDT args in Step 2 after the transfer transaction was mined into a block. The query confirmed the split state:
- **Cell #0 (Recipient):** Token amount = **10**, Holder = `0x758d311c8483e0602dfad7b69d9053e3f917457d`
- **Cell #1 (My Change):** Token amount = **32**, Holder = `0x8e42b1999f265a0078503c4acec4d5e134534297`
- Total supply preserved: 10 + 32 = 42 tokens!

[![After Transfer Verification](https://raw.githubusercontent.com/devfoma/Build_On_CKB_Campaign_05/main/screenshots/08-after-transfer.png)](https://github.com/devfoma/Build_On_CKB_Campaign_05/blob/main/screenshots/08-after-transfer.png)

---

## 3. My Reflection

### What I Found Fascinating Building on CKB with xUDT

Coming into this campaign with an EVM mindset where everything lives in contract-centric state and global mappings like `mapping(address => uint256)`, building on Nervos CKB felt like a genuine paradigm shift.

Here are the biggest technical breakthroughs, debugging moments, and concepts that stood out to me:

#### 1. True Digital Ownership vs. "Permission Slips"
In Ethereum's ERC-20 world, I never really felt like I truly "held" my tokens. The ERC-20 smart contract owns the whole state, and my address is just a key in a database table. If that contract has an admin key, pause function, or reentrancy bug, my balance can be altered or locked without my direct signature.

On CKB, **assets are first-class citizens**. 
When I minted my 42 xUDT tokens in Step 2, the network didn't increment a balance variable in a centralized contract. Instead, it produced an independent, tangible **Cell** whose Lock Script was tied directly to my public key hash. I am the sole owner of that cell container. Not even the token issuer can reach into my wallet and move or seize that cell unless I sign a transaction that satisfies my Lock Script.

#### 2. The Clean Orthogonality of Lock Script vs. Type Script
The separation of responsibilities in CKB's Cell model is something I fell in love with:
- **Lock Script answers:** *WHO is authorized to unlock and consume this Cell?*
- **Type Script answers:** *WHAT state transition rules must be obeyed when transforming this Cell?*

When I transferred tokens in Step 4, I didn't invoke a `transfer()` method on a smart contract. I simply assembled a transaction that consumed my 42-token input cell and created two new output cells:
- One cell with 10 tokens locked to the recipient.
- One cell with 32 tokens locked back to my own address.

The **xUDT Type Script** ran inside the CKB-VM and enforced one straightforward invariant:
Sum of Input Token Amounts >= Sum of Output Token Amounts
Because 42 >= 10 + 32, the transaction was mathematically valid. I transferred ownership simply by replacing the Lock Script on the recipient's output cell while preserving the exact same Type Script!

#### 3. State Rent: Why Every Cell Needs 146 CKB Capacity
One of the most eye-opening things I learned was why my token cell required exactly **146 CKB** of capacity (`0x3663a5200` Shannon).
In CKB, 1 CKB represents 1 byte of state storage on the blockchain:
- 8 bytes for capacity
- 32 bytes for the lock script hash & structure
- 32 bytes for the type script hash & structure
- 16 bytes for the u128 token balance data
- Plus script code hashes and arguments

Totaling ~146 bytes!
This directly tackles the state bloat tragedy that plagues Ethereum, where zombie tokens and abandoned contract memory sit in node RAM forever without paying ongoing rent. On CKB, state is prepaid and scarce. If I ever decide to melt or burn my tokens in the future, I can consume the cell and **reclaim my 146 CKB back into my liquid balance**!

#### 4. The "x" in xUDT: Real-World Use Cases That Excite Me
In standard Simple UDT (sUDT), the script args are strictly the issuer's lock script hash.
In xUDT (Extensible UDT), the args include a 4-byte flag field (`00000000` in my demo).
What makes xUDT so exciting is that those flags let me attach **Extension Scripts** via transaction witnesses. This unlocks use cases that are normally clunky or expensive on other chains:
- **Regulatory Compliant Tokens:** Embedding KYC/AML whitelist checks directly into witness validation without altering the core token code.
- **Vesting & Payroll Streams:** Time-locked cell scripts that enable periodic withdrawals of xUDT tokens without complex proxy contracts.
- **Soulbound Credentials & Badges:** Extension scripts that forbid swapping the Lock Script, ensuring credentials remain permanently tied to the recipient.
- **Cross-Chain RGB++ Assets:** Using xUDT as the isomorphic binding layer for Bitcoin Ordinals and Runes, enabling Layer 2 speed on CKB with Layer 1 Bitcoin security.

#### 5. Real Windows Debugging Moments That Taught Me the Stack
Building this on Windows 11 gave me some great hands-on debugging experiences:
- **Shell Differences:** The Nervos tutorial uses Unix-style syntax (`NETWORK=devnet npm start`), which threw errors in Windows cmd and PowerShell. Setting `$env:NETWORK = "devnet"` explicitly before launching Parcel ensured the client bundle targeted my local RPC (`http://localhost:28114`) rather than public testnets.
- **Input Validation Quirks:** In `index.tsx`, the `onInputSenderPrivKey` handler validated the full 66-character private key regex on every `onChange` event and immediately threw a blocking `alert()`. Learning how to dispatch the input atomically was a great reminder of how frontends must accommodate interactive user input cleanly.
- **Mempool Block Mining:** Querying cells immediately after `sendTransaction` returned empty until the devnet miner produced the next block. Implementing a reactive polling retry loop really drove home the asynchronous reality of decentralized consensus versus local database writes.

---
*Built with passion, curiosity, and code for Build on CKB Campaign 05.*
