# Algorand Quantum Transactions — New PC Setup Guide

A practical guide for recreating the Pera Quantum / Falcon-1024 transaction environment on a new Windows PC using WSL Ubuntu.

## 1. Install Ubuntu / WSL

Open PowerShell as Administrator:

```powershell
wsl --install -d Ubuntu
```

Restart Windows if requested, then open Ubuntu.

## 2. Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

Create your Linux username and password if Ubuntu asks.

## 3. Install Python

```bash
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```

## 4. Install Algorand tools

Install the current Algorand package using the official Algorand installation instructions.

Install Algorand tools

```sudo apt update
sudo apt install -y gnupg2 curl software-properties-common

curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc

sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"

sudo apt update

sudo apt install -y algorand
```


Verify that the PQ commands are available:

```bash
algokey --version
algokey pq --help
```

You should have PQ commands such as `import`, `info`, and `sign`.

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

## 6. Restore your Pera Quantum key

The safest approach is to recreate the key from your **Pera Quantum recovery phrase** rather than copying the private key file to the new PC.

Never send your recovery phrase to anyone, including ChatGPT. Never commit it to GitHub.

Enter it locally:

```bash
read -s PQ_MNEMONIC
```

Paste the 25-word recovery phrase and press Enter.

Import it:

```bash
algokey pq import -m "$PQ_MNEMONIC" -k ~/pera_quantum.key
```

Clear the shell variable immediately:

```bash
unset PQ_MNEMONIC
```

Verify the key:

```bash
algokey pq info -k ~/pera_quantum.key
```

Confirm that the displayed Falcon-1024 PQ address matches your Pera Quantum account.

> **Security:** Do not commit `pera_quantum.key` or your recovery phrase to this repository. Keep the recovery phrase offline and secure.

## 7. Restore your scripts

Copy your Python scripts from the old PC into `~/algo-transfer`.

Example:

```text
algo-transfer/
├── make_tx.py
├── transfer_100_pq.py
└── transactions_100.txt
```

Do not copy the `venv` directory; recreate it using the commands above.

## 8. Check the transaction configuration

Typical configuration:

```python
SENDER = "YOUR_QUANTUM_ADDRESS"
RECIPIENT = SENDER

COUNT = 100
AMOUNT_MICROALGO = 100_000
FEE_MICROALGO = 3_000
```

For 500 transactions:

```python
COUNT = 500
```

For 0.1 ALGO:

```python
AMOUNT_MICROALGO = 100_000
```

MainNet Algod endpoint:

```python
ALGOD_ADDRESS = "https://mainnet-api.algonode.cloud"
```

If your script uses a delay between transactions:

```python
time.sleep(0.3)
```

## 9. Verify before sending

Check the restored account:

```bash
algokey pq info -k ~/pera_quantum.key
```

Check the account balance:

```bash
python -c "from algosdk.v2client import algod; c=algod.AlgodClient('', 'https://mainnet-api.algonode.cloud'); a='YOUR_QUANTUM_ADDRESS'; print(c.account_info(a)['amount']/1000000, 'ALGO')"
```

Check the Python script for syntax errors:

```bash
python -m py_compile transfer_100_pq.py
```

No output means the syntax check passed.

## 10. Run the transaction automation

Activate the environment:

```bash
cd ~/algo-transfer
source venv/bin/activate
```

Run the script:

```bash
python transfer_100_pq.py
```

Before starting, verify the sender, recipient, amount, count, fee, and available balance.

If your script uses a confirmation such as:

```text
START100
```

enter it only after checking all displayed transaction details.

For 500 transactions, make sure the script's count is set to 500 and use the corresponding confirmation required by your script.

## 11. Recommended backup

Keep a backup of the source scripts:

```text
algo-transfer-backup/
├── make_tx.py
└── transfer_100_pq.py
```

Do **not** put these in the backup repository:

```text
pera_quantum.key
25-word recovery phrase
private keys
```

## Quick start after the new PC is configured

For future sessions, the basic startup is:

```bash
cd ~/algo-transfer
source venv/bin/activate
python transfer_100_pq.py
```

## Security checklist

- Never publish your Quantum recovery phrase.
- Never commit `pera_quantum.key` to GitHub.
- Add sensitive key files to `.gitignore`.
- Verify the sender and recipient before every real MainNet run.
- Test with a small transaction count before increasing the count.
- Keep an offline backup of the recovery phrase.

## Repository

This guide is maintained in the `SHAKTHIVASEE/Algo-quantum-transactions` repository.
