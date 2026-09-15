# Raw build log — things that actually happened
(Facts only. Maduegbunam Faith Amarachi (devfoma) writes the reflection from this in her own words.)

## Environment
- OS: Windows 11 Home / Pro (x64), desktop-pi1gf9u
- node: v24.15.0
- npm: 11.12.1
- git: 2.53.0.windows.3
- offckb: 0.4.6
- ckb node binary: v0.205.0 (x86_64-pc-windows-msvc)
- dApp bundler: Parcel 2.15.4
- CKB SDK: `@ckb-ccc/core` v1.5.3

## Pre-Funded Devnet Accounts Used
- **Account #0 (Issuer / Sender)**:
  - Address: `ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqvwg2cen8extgq8s5puft8vf40px3f599cytcyd8`
  - Private Key: `0x6109170b275a09ad54877b82f7d9930f88cab5717d484fb4741ae9d1dd078cd6`
  - Lock Script Args: `0x8e42b1999f265a0078503c4acec4d5e134534297`
  - Lock Script Hash: `0x7de82d61a7eb2ec82b0dc653e558ba120efcbfbb44dac87c12972d05bf250653`
  - Initial Balance: `42,000,000 CKB`
- **Account #1 (Receiver)**:
  - Address: `ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqt435c3epyrupszm7khk6weq5lrlyt52lg48ucew`
  - Private Key: `0x9f315d5a9618a39fdc487c7a67a8581d40b045bd7a42d83648ca80ef3b2cb4a1`
  - Lock Script Args: `0x758d311c8483e0602dfad7b69d9053e3f917457d`
  - Initial Balance: `42,000,000 CKB`

## Timeline
| when | what happened |
|------|---------------|
| 11:28 | Verified environment tooling: Node v24.15.0, npm 11.12.1, git 2.53.0, offckb 0.4.6. |
| 11:31 | Cloned `nervosnetwork/docs.nervos.org` to access `examples/dApp/xudt`. |
| 11:36 | Started `offckb node` in background. OffCKB automatically provisioned CKB v0.205.0 Windows binary and launched proxy RPC (`:28114`) and direct RPC (`:8114`). |
| 11:51 | Verified devnet node RPC is actively mining blocks (tip block 181 `0xb5`). |
| 11:52 | Installed dApp dependencies via `npm install`. |
| 11:58 | Built frontend bundle with `NETWORK=devnet npm run build` (Parcel bundled in 23s). |
| 11:59 | Started Parcel dev server on `http://localhost:1234`. Verified HTTP 200 and initial UI render. |
| 12:08 | Executed automated campaign workflow to issue xUDT, query token cells, transfer 10 tokens to Account #1, and capture all 8 visual proofs and execution logs. |
| 12:09 | Captured all 8 screenshots in `screenshots/` and verified on-chain balances (Receiver: 10, Issuer change: 32). |

## Errors hit and how they were fixed
1. **Windows Command Line Env Vars**:
   - *Error*: Tutorial writes `NETWORK=devnet npm start`. In Windows `cmd.exe` this fails with `'NETWORK' is not recognized as an internal or external command`.
   - *Fix*: Explicitly set `$env:NETWORK="devnet"` in PowerShell before running `npm run build` / `npm start`.
2. **Parcel Bundle Environment Inlining**:
   - *Error*: If `NETWORK` is unset during bundling, `ccc-client.ts` falls back to `testnet` and queries external public testnets instead of local OffCKB devnet.
   - *Fix*: Exported `NETWORK=devnet` so Parcel inlined `http://localhost:28114` as the RPC endpoint.
3. **React Input Validation Alert Loop**:
   - *Error*: `onInputSenderPrivKey` in `index.tsx` checks `/^0x[0-9a-fA-F]{64}$/` on every single input change and triggers a blocking `window.alert()` if incomplete.
   - *Fix*: Populated the full 66-character private key atomically via property descriptor setter and handled browser dialog dismissal.
4. **Devnet Block Mining Latency on Query**:
   - *Error*: Querying newly issued cells immediately after `sendTransaction` can return 0 cells if the devnet block hasn't been sealed yet.
   - *Fix*: Added a 4-5s delay to allow OffCKB's 2-second block interval to mine and index the cell before querying.

## Things that were surprising / worth thinking about
1. **True Asset Ownership (Cell Model vs EVM Account Model)**:
   - In Ethereum ERC-20, tokens are entries in a single contract's internal mapping (`mapping(address => uint256)`). Users don't actually own a token container; they have permission to ask the contract to update numbers.
   - In CKB, each token balance is an independent physical **Cell** owned directly by the user's Lock Script. The token contract (Type Script) just defines the conservation rule (\(\sum inputs \ge \sum outputs\)).
2. **State Rent / Capacity Constraint**:
   - Every cell holding xUDT tokens must hold at least 146 CKB (`0x3663a5200` Shannon) to pay for the byte space occupied by its data, lock script, and type script. This cleanly solves blockchain state bloat.
3. **Extensibility via Flags**:
   - The token args `0x7de82d61a7eb2ec82b0dc653e558ba120efcbfbb44dac87c12972d05bf25065300000000` embed the Issuer Lock Script Hash (32 bytes) followed by 4 bytes of flags (`00000000`). When flags are non-zero, extension scripts can be attached via witnesses for programmable compliance, expiration, or custom logic without modifying the core xUDT script!

## Open questions
- How can dApps best sponsor the 146 CKB storage capacity for new users so the recipient doesn't need to already own 146 CKB to receive a custom token? (Investigate Open Transaction / Anyone-Can-Pay / Omnilock sponsors).
- What is the most gas-efficient way to batch multi-recipient xUDT airdrops in a single transaction without hitting block cycle limits?

