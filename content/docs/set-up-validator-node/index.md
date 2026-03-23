---
title: "QL1 Validator Guide (Using Drive)"
description: "Step-by-step guide for running a validator node using Drive."
slug: "set-up-validator-node"
toc: true
---
## Prerequisites

- **Git** – For cloning repositories
    
- **Docker** (20.10+) – For running containers
    
- **Docker Compose** (1.29+) – For managing multi-container setups
    
- **Drive** – For running and managing QOM nodes
    

> ⚠️ QOM nodes are designed for Linux machines or servers (VPS) environments. macOS and Windows are not officially supported. Validator nodes are expected to run 24/7.

Before proceeding, follow the Drive Quick Start Guide (Until `Step 1: Clone the Repository`):  
👉 [Drive Quick Start Guide](https://github.com/deep-thought-labs/drive/blob/main/docs/quick-start.md)

---

## Step 1️⃣ Navigate to the QOM Service Directory

```bash
cd ~/drive/services/node3-qom
```

**Available Services:**

- `node0-infinite/` – Infinite Mainnet Node
    
- `node1-infinite-testnet/` – Infinite Testnet Node
    
- `node2-infinite-creative/` – Infinite Creative Network Node
    
- `node3-qom/` – QOM Network Node

---

## Step 2️⃣ Start Containers

```bash
./drive.sh up -d 

./drive.sh ps
```

If the container shows “UP,” it’s working.

---

## Step 3️⃣ Initialize the Validator via GUI

**Open GUI:**
```bash
./drive.sh exec qom node-ui
```
### ⚠️ Important: Follow the steps **in the exact order** below.

> (If you already have your mnemonic prepared, start from **step 3**.)

1. From the main menu, go to **“1 Key Management → 1 Generate Key (Dru-Run – Recommended)”**.
    
2. Create a new key:
    
    - Enter a key name (for example, `validator`).
        
    - The system will display your **mnemonic phrase** (24 words).
        
    - ⚠️ **Important:** Write down the mnemonic immediately and store it securely.
        
    - This mnemonic is the **only way** to recover your validator key.
        
3. Go to **“Main Menu → 2 Node Operations → 4 Advanced Operations → 2 Initialize with Recovery (Validator)”**.
    
4. When prompted, enter the moniker.
    
5. Follow the on-screen instructions to "**Press Enter**".
	
6. Then enter your  **mnemonic phrase** (24 words) and press enter key.
    
7. After initialization is complete, return to the **Main Menu**, select **“0 Exit”**, and close the GUI.

---
## Step 4️⃣ Configure State Sync

After initializing the node, the necessary files will be created. Edit the file to configure for State Sync. Since **completing node sync is a prerequisite for registering as a validator**, it is recommended to apply this configuration and catch up to the latest block. If you do not perform State Sync, node sync may take several days to several weeks.

```bash
cd ~/drive/services/node3-qom/persistent-data/config 

nano config.toml
```

**Edit `config.toml` as below:**
```toml
#######################################################
###         State Sync Configuration Options        ###
#######################################################
[statesync]
.
.
.
enable = true
.
.
.
rpc_servers = "rpc.foxxone.one:443,138.201.133.24:26657,51.38.37.230:26687"
trust_height = 10052000
trust_hash = "85B05855BAFACB3E0EE50BB4441705E428CF908AAAD9B4AB15F2E479DAA67898"
trust_period = "112h0m0s"
```

> Replace the latest values of `trust height` and `trust hash`.
> Please get them from  [Ping Dashboard](https://ping.qom.one/qom/statesync)

 <details>
<summary><strong>⚠️ When ping dashboard is unavailable:</strong></summary>

Retrieve trust height and trust hash manually (requires `jq`).

Trust height:
```bash
LATEST=$(curl -sk https://rpc.foxxone.one/status | jq -r '.result.sync_info.latest_block_height')
TRUST_HEIGHT=$(( ( (LATEST - 2000) / 1000 ) * 1000 ))
echo $TRUST_HEIGHT
```

Trust hash:
```bash
for RPC in \
"https://rpc.foxxone.one" \
"http://138.201.133.24:26657" \
"http://51.38.37.230:26687"
do
  echo "$RPC"
  curl -s "$RPC/block?height=$TRUST_HEIGHT" | jq -r '.result.block_id.hash'
done
```

> Obtain hash values of the same block height from three RPC servers and verify that they all match.

</details>

**Restart the container:**
```bash
./drive.sh down

./drive.sh up -d
```

---

## Step 5️⃣ Start Node and Verify

**Open GUI and start node**
```bash
./drive.sh exec qom node-ui
```

>Start node → **2 Node Operations → 2 Start Node**  
>Then, check the status: **"3 Node Monitoring → 4 Node Status(RPC)"**

```json
"catching_up": false
```

**If `false`, node is synced.**

If State Sync is enabled, sync to the latest block will complete in about 2–3 minutes. If sync does not complete, please check the logs using the **“2 View Logs (Last 50 lines)”** or **“3 Follow Logs (Real-time)”** options.

---

## Step 6️⃣ Send QOM to Wallet

### Add Account to Keyring

1. Go to **“1 Key Management → 3 Add Existing Key From Seed Phrase.”**
    
2. Give your key a name (for example, `validator`). ⚠️ _You will use this name later._
    
3. Set a passphrase for the keyring.
    
4. Restore the key using the **mnemonic phrase** that you generated for your validator account in **`Step 3️⃣`**.
    
5. Enter the keyring passphrase again when prompted.
    
6. Copy and keep the displayed wallet address (`qom1xxxx....`) for later use.

### Add Funds to Your Wallet

Convert bech32 address to EVM-style:  [Address Converter](https://converter.teamtuzi.xyz/)  
Then send QOM to EVM-style `0x...` address. (**⚠️It is highly recommended to first test with a small amount.**)

---

## Step 7️⃣ Submit Create-validator Transaction

### Check Balance from your node
```bash
docker exec -it qom \
  /home/ubuntu/bin/qomd q bank balances qom1xxx... \
  --home /home/ubuntu/.qomd
```

> Replace `qom1xxx...` with **your own wallet address**.  
> Check that the funds you recently transferred have **arrived in your wallet**.  
> The balance is displayed in units of **`aqom`**.  
> **1 QOM = 1 × 10¹⁸ aqom**

### Submit Create-validator Transaction

```bash
docker exec -it qom bash -lc '
  /home/ubuntu/bin/qomd tx staking create-validator \
    --amount=1000000000000000000aqom \
    --pubkey="$(
      /home/ubuntu/bin/qomd tendermint show-validator --home /home/ubuntu/.qomd
    )" \
    --moniker="Your moniker here" \
    --chain-id=qom_766-1 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation="1" \
    --gas="auto" \
    --gas-adjustment=1.2 \
    --from="validator" \
    --fees=10000000aqom \
    --keyring-backend=file \
    --home=/home/ubuntu/.qomd
'
```

**Example:**  
The command above sends a transaction to stake **1 QOM**.

`--amount=1000000000000000000aqom \`

> **1QOM = 1 × 10¹⁸ aqom**

`--moniker="Your moniker here" \`

> The **moniker** is your chosen nickname.

`--from="validator" \`

> Use the **same key name** that you created and named in **Step 6️⃣**.

Replace these values with your own settings — such as your desired **moniker** and the **amount** you wish to stake.  Also, make sure to send **a slightly larger balance** to your wallet beforehand to cover the **gas fees** required for this transaction.

---

## 🔍 Validation Checks

```bash
# Check your own validator information  
# (Replace qomvaloper1xxxxx... with your actual validator address)
docker exec -it qom /home/ubuntu/bin/qomd q staking validator qomvaloper1xxxxxx.... --home /home/ubuntu/.qomd


# Verify that your validator appears in the validator list
docker exec -it qom /home/ubuntu/bin/qomd q staking validators --home /home/ubuntu/.qomd


# Check validator status (Bonded / Unbonding / Unbonded)  
# Replace "moniker" with your own moniker
docker exec -it qom /home/ubuntu/bin/qomd q staking validators | grep -A 6 moniker
```

---

## ⚙️ Common Commands

```bash
# Add delegation (increase self-stake)
docker exec -it qom /home/ubuntu/bin/qomd tx staking delegate qomvaloper1xxxx 1000000000000000000aqom \
--from validator --keyring-backend file --home /home/ubuntu/.qomd --chain-id qom_766-1 \
--gas auto --gas-adjustment 1.5 --fees 7000000aqom -y


# Check rewards
docker exec -it qom /home/ubuntu/bin/qomd q distribution rewards qom1xxxx.... \
  --home /home/ubuntu/.qomd

# Withdraw rewards
docker exec -it qom /home/ubuntu/bin/qomd tx distribution withdraw-rewards qomvaloper1xxxx... \
  --from validator --commission --home /home/ubuntu/.qomd --chain-id qom_766-1 --fees 1000000aqom -y
```


---

## Delegation Example

Find target validator: 
Example: find "deep thought" validator's valoper address

```bash
docker exec -it qom bash -lc '
  /home/ubuntu/bin/qomd q staking validators -o json --home /home/ubuntu/.qomd | jq -r ".validators[] | select(.description.moniker | test(\"deep thought\"; \"i\")) | {moniker: .description.moniker, valoper: .operator_address}"
'
```


For example, if you want to delegate **10 QOM** to another validator such as `qomvaloper1abcdxyz...`, use the following command as a reference:

```bash
docker exec -it qom \
  /home/ubuntu/bin/qomd tx staking delegate qomvaloper1abcdxyz... 10000000000000000000aqom \
  --from validator \
  --keyring-backend file \
  --home /home/ubuntu/.qomd \
  --chain-id qom_766-1 \
  --fees 7000000aqom \
  -y
```

---
