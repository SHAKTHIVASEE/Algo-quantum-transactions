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

Install Algorand tools:

```bash
sudo apt update
sudo apt install -y gnupg2 curl software-properties-common

curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc

sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"

sudo apt update

sudo apt install -y algorand-devtools
```

Verify that the PQ commands are available:

```bash
algokey --version
algokey pq --help
```

You should have PQ commands such as `import`, `info`, and `sign`.

## 5. Create the project

Create the project folder and enter it:

```bash
mkdir -p ~/algo-transfer
cd ~/algo-transfer
```

Create and activate the Python virtual environment:

```bash
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

## 7. Create new transaction scripts

On a new PC, you can create the Python scripts manually instead of restoring an old `venv` or expecting the scripts to appear automatically.

### 7.1 Go to the project folder

```bash
cd ~/algo-transfer
```

### 7.2 Create the main transaction script

Create a new file named `transfer_100_pq.py`:

```bash
nano transfer_100_pq.py
```

The file will initially be empty. This is normal.

Now **paste your previously working 100/500 transaction Python code** into the file.

For example, the file should contain your working code that:

- Connects to Algorand MainNet.
- Uses your Falcon-1024 / Pera Quantum signing setup.
- Sets the sender and recipient.
- Sets the transaction amount and fee.
- Builds and signs the transactions.
- Sends the transactions.
- Records successful transaction IDs in `transactions_100.txt`.

> **Important:** The README does not store your private key, recovery phrase, or secret signing material. Copy only the Python source code that you previously tested and verified.

Save the file in `nano`:

1. Press `Ctrl + O`
2. Press `Enter`
3. Press `Ctrl + X`

Check that it was created:

```bash
ls -l transfer_100_pq.py
```

Check its contents:

```bash
nano transfer_100_pq.py
```

### 7.3 Create the helper script, if your setup uses it

If your previous setup used `make_tx.py`, create it the same way:

```bash
nano make_tx.py
```

Paste your previously working `make_tx.py` code, then save:

1. `Ctrl + O`
2. `Enter`
3. `Ctrl + X`

Check both files:

```bash
ls -la ~/algo-transfer
```

You should have something similar to:

```text
algo-transfer/
├── make_tx.py
├── transfer_100_pq.py
└── venv/
```

The transaction log `transactions_100.txt` will normally be created by the transaction script after a successful run. You do not need to create an empty log file manually unless your script specifically requires one.

### 7.4 Test the new script before running it

Make sure the virtual environment is active:

```bash
cd ~/algo-transfer
source venv/bin/activate
```

Run a Python syntax check:

```bash
python -m py_compile transfer_100_pq.py
```

If there is **no output**, the Python syntax check passed.

If there is an error, open the file and correct it:

```bash
nano transfer_100_pq.py
```

Then run the syntax check again.

### 7.5 Copy the script from your old PC instead

If you already have the working script on your old PC, you can copy `transfer_100_pq.py` and `make_tx.py` into `~/algo-transfer` instead of manually retyping them.

Do **not** copy the old `venv` directory. Recreate the virtual environment using Section 5.

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
