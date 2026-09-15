# Build on CKB — Campaign 05 Runbook (xUDT / OffCKB)

Tutorial: https://docs.nervos.org/docs/dapp/create-token
Shell: **Command Prompt (cmd.exe)** on Windows.
Home path contains spaces (`C:\Users\U S E R`) — **always quote paths**.

> Why cmd over PowerShell: npm's global shims are `.cmd` files, so `offckb` runs
> without touching PowerShell's script execution policy.

---

## cmd cheat-sheet for this run

| thing | cmd syntax |
|---|---|
| set an env var | `set NETWORK=devnet` (no quotes, no spaces around `=`) |
| home folder | `%USERPROFILE%` |
| chain commands | `&&` (a `;` does **not** chain in cmd) |
| new tab/window | `start cmd` |

---

## Step 0 — Prerequisites

```cmd
node -v
npm -v
git --version
```
Need node >= 20, git >= 2.40.

## Step 1 — Install OffCKB (need >= 0.4.0; latest is 0.4.6)

```cmd
npm install -g @offckb/cli
offckb --version
```
In cmd the `@` needs no quoting.

## Step 2 — Get the example source

```cmd
cd /d "%USERPROFILE%\Drips\Devfoma\Build_On_CKB_Campaign_05"
git clone https://github.com/nervosnetwork/docs.nervos.org.git --depth 1
cd "docs.nervos.org\examples\dApp\xudt"
```
`cd /d` is needed if it ever has to switch drive letters.

## Step 3 — Start the devnet  — TERMINAL A, leave it running

```cmd
offckb node
```
Expect: proxy RPC `http://127.0.0.1:28114`, direct RPC `http://127.0.0.1:8114`, blocks ticking.

## Step 4 — Pre-funded accounts  — TERMINAL B (`start cmd`)

```cmd
offckb accounts
```
Note down, for account #0 (issuer) and account #1 (receiver):
- private key
- lock script hash
- address

These are public devnet test keys — safe to screenshot.

## Step 5 — Run the dApp  — TERMINAL B

```cmd
cd /d "%USERPROFILE%\Drips\Devfoma\Build_On_CKB_Campaign_05\docs.nervos.org\examples\dApp\xudt"
npm install
set NETWORK=devnet
npm start
```

> **Gotcha:** the tutorial writes `NETWORK=devnet npm start`. That's bash.
> In cmd it fails with `'NETWORK' is not recognized as an internal or external command`.
> `set NETWORK=devnet` must be its own line, *before* `npm start`.
> (`set` only lasts for that cmd window — so it must be the same window that runs `npm start`.)

Open <http://localhost:1234>.

---

## Proofs to capture → save into `screenshots\`

| # | File name | Must show |
|---|-----------|-----------|
| 1 | `01-offckb-version.png`    | `offckb --version` |
| 2 | `02-devnet-running.png`    | `offckb node` producing blocks |
| 3 | `03-accounts.png`          | `offckb accounts` |
| 4 | `04-dapp-running.png`      | dApp live at localhost:1234 |
| 5 | `05-issue-token.png`       | custom xUDT issued + tx hash |
| 6 | `06-query-by-lockhash.png` | token cells found via issuer lock script hash |
| 7 | `07-transfer.png`          | transfer to account #1 + tx hash |
| 8 | `08-after-transfer.png`    | balances after transfer |

Screenshot: **Win + Shift + S** → save into
`C:\Users\U S E R\Drips\Devfoma\Build_On_CKB_Campaign_05\screenshots\`
