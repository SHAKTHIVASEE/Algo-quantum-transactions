# Automate Algorand Quantum Transactions in Ubuntu

Guide for running Algorand post-quantum (Falcon-1024) transactions on Ubuntu/WSL.

Algorand supports post-quantum accounts using Falcon-1024 signatures, and `algokey pq` provides the commands for PQ key management and transaction signing. See the [official Algorand Post-Quantum Accounts documentation](https://dev.algorand.co/concepts/accounts/post-quantum/).

> ⚠️ **MainNet warning:** This guide sends real ALGO. Always verify the address, amount, transaction count, fee, and balance before running the script.

## 1. Install WSL Ubuntu

Open **PowerShell as Administrator**:

```powershell
wsl --install -d Ubuntu
```

Restart Windows if requested, then open Ubuntu.

## 2. Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

## 3. Install Python

```bash
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```

## 4. Install Algorand tools

Install the current Algorand software using the official installation instructions. `algokey` is included with the Algorand node software, and the PQ commands include `generate`, `import`, `info`, `sign`, and `check-address`.

Official documentation:

- https://dev.algorand.co/reference/algokey/
- https://dev.algorand.co/nodes/installation/manual-installation/

Verify:

```bash
algokey --version
algokey pq --help
```

## 5. Create the project

```bash
mkdir -p ~/algo-transfer
cd ~/algo-transfer
python3 -m venv venv
source venv/bin/activate
```

Install the Python SDK:

```bash
pip install py-algorand-sdk msgpack
python -c "import algosdk; print('Algorand SDK OK')"
```

## 6. Restore your Quantum account

Use your **Pera Quantum 25-word recovery phrase locally** to recreate the Falcon-1024 key.

Never send the recovery phrase to anyone and never put it in GitHub.

```bash
read -s PQ_MNEMONIC
```

Paste the new quantum wallet recovery phrase and press Enter.

Then:

```bash
algokey pq import -m "$PQ_MNEMONIC" -k ~/pera_quantum.key
unset PQ_MNEMONIC
```

Check the account:

```bash
algokey pq info -k ~/pera_quantum.key
```

Check PQ compliance:

```bash
algokey pq check-address YOUR_QUANTUM_ADDRESS
```

## 7. Create your transaction script

Create the Python file:

```bash
cd ~/algo-transfer
nano transfer_100_pq.py
```

In below script change the sender address with your public address & Paste shown **working transaction automation script** into the file.

```import json
import os
import subprocess
import time

from algosdk import encoding
from algosdk.v2client import algod
from algosdk.transaction import PaymentTxn, SignedTransaction

SENDER = "3CJRWINIM4VPA3BIO6N253VFZCWTCJQE67OP4KYKMCJGVP74K4Y7AJOPAY"
RECIPIENT = SENDER

COUNT = 500
AMOUNT_MICROALGO = 100_000   # 0.1 ALGO
FEE_MICROALGO = 3_000        # 0.003 ALGO each

ALGOD_ADDRESS = "https://mainnet-api.algonode.cloud"
KEYFILE = os.path.expanduser("~/pera_quantum.key")

client = algod.AlgodClient("", ALGOD_ADDRESS)

print("=== Quantum MainNet Transfer ===")
print("Sender   :", SENDER)
print("Recipient:", RECIPIENT)
print("Amount   : 0.1 ALGO")
print("Count    :", COUNT)
print("Fee      : 0.003 ALGO each")
print()

balance = client.account_info(SENDER)["amount"]
required = COUNT * (AMOUNT_MICROALGO + FEE_MICROALGO)

print(f"Balance  : {balance / 1_000_000:.6f} ALGO")
print(f"Required : {required / 1_000_000:.6f} ALGO")

if balance < required:
    raise SystemExit("Insufficient ALGO balance.")

confirm = input(
    "\nType START500 to send 500 transactions: "
).strip()

if confirm != "START500":
    raise SystemExit("Cancelled.")

successful = 0

with open("transactions_500.txt", "a") as log:

    for i in range(1, COUNT + 1):

        try:
            params = client.suggested_params()
            params.fee = FEE_MICROALGO
            params.flat_fee = True

            txn = PaymentTxn(
                sender=SENDER,
                sp=params,
                receiver=RECIPIENT,
                amt=AMOUNT_MICROALGO,
            )

            stxn = SignedTransaction(txn, None)

            encoded = encoding.msgpack_encode(stxn)
            raw_txn = __import__("base64").b64decode(encoded)

            with open("unsigned.txn", "wb") as f:
                f.write(raw_txn)

            subprocess.run(
                [
                    "algokey",
                    "pq",
                    "sign",
                    "-k",
                    KEYFILE,
                    "-t",
                    "unsigned.txn",
                    "-o",
                    "signed.txn",
                    "--overwrite",
                ],
                check=True,
            )

            result = subprocess.run(
                [
                    "curl",
                    "-sS",
                    "-X",
                    "POST",
                    "-H",
                    "Content-Type: application/msgpack",
                    "--data-binary",
                    "@signed.txn",
                    f"{ALGOD_ADDRESS}/v2/transactions",
                ],
                capture_output=True,
                text=True,
                check=True,
            )

            response = json.loads(result.stdout)

            if "txId" not in response:
                raise RuntimeError(
                    f"Submission failed: {result.stdout}"
                )

            txid = response["txId"]
            successful += 1

            print(
                f"[{i:03}/{COUNT}] SUCCESS | {txid}"
            )

            log.write(f"{i:03} | {txid}\n")
            log.flush()

            time.sleep(0.2)

        except Exception as e:

            print(
                f"[{i:03}/{COUNT}] ERROR: {e}"
            )

            print("STOPPING to prevent duplicate transactions.")
            break

print()
print("=== COMPLETE ===")
print(f"Successful: {successful}")
print("Transaction log: transactions_500.txt")
PY
```

Save in nano:

1. `Ctrl + O`
2. `Enter`
3. `Ctrl + X`

If your setup also uses `make_tx.py`:

```bash
nano make_tx.py
```

Paste your working helper script and save it.

Check the files:

```bash
ls -la
```

You should have something similar to:

```text
algo-transfer/
├── make_tx.py
├── transfer_100_pq.py
└── venv/
```

## 8. Configure the transaction script

For the MainNet Algod endpoint used in this guide:

```python
ALGOD_ADDRESS = "https://mainnet-api.algonode.cloud"
```

## 9. Test the script

Always run a syntax check first:

```bash
python -m py_compile transfer_100_pq.py
```

No output means the syntax check passed.

Check your Quantum account:

```bash
algokey pq info -k ~/pera_quantum.key
```

Before a real MainNet run, confirm:

- Sender address
- Recipient address
- Transaction amount
- Transaction count
- Fee
- Available ALGO balance

## 10. Run the automation

```bash
cd ~/algo-transfer
source venv/bin/activate
python transfer_100_pq.py
```

Your script can record successful transaction IDs in a log such as:

```text
transactions_100.txt
```

Make sure the account has enough balance for the total amount plus fees.

## 11. New PC quick setup

After installing WSL, Python, Algorand tools, and the project:

```bash
cd ~/algo-transfer
source venv/bin/activate
algokey pq info -k ~/pera_quantum.key
python -m py_compile transfer_100_pq.py
python transfer_100_pq.py
```

Do not copy the old `venv` directory to a new PC. Recreate it with Section 5.

## Useful commands

Check Algorand tools:

```bash
algokey --version
```

Check Quantum key:

```bash
algokey pq info -k ~/pera_quantum.key
```

Check address:

```bash
algokey pq check-address YOUR_QUANTUM_ADDRESS
```

Activate the project:

```bash
cd ~/algo-transfer
source venv/bin/activate
```

Run the script:

```bash
python transfer_100_pq.py
```

## Official Algorand documentation

- [Post-Quantum Accounts](https://dev.algorand.co/concepts/accounts/post-quantum/)
- [AlgoKey](https://dev.algorand.co/reference/algokey/)
- [Algorand Installation](https://dev.algorand.co/nodes/installation/manual-installation/)

## Repository

**SHAKTHIVASEE/Algo-quantum-transactions**
