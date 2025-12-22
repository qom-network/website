---
title: "QL1 Scan Platform Guide (Using Drive)"
description: "Step-by-step guide for running a Blockscout using Drive."
slug: "set-up-blockscout"
toc: true
---
In this article, we will use Drive to set up BlockScout. Anyone can run a QL1 scanning platform by managing a domain with Cloudflare or similar. Running a scanning platform is not a developer's privilege. We believe anyone can run it freely.Here we will introduce the most basic settings, but feel free to customize the rest.

## Prerequisites

- Running the qom node on Drive

## 📂 Directory structure (planned)

```
~/blockscout/ ← Clone the Blockscout repository to the root

~/drive/
└─ services/
├─ run-scan0-infinite/ ← .env and override for node0-infinite
│ ├─ scan0.env.        ← Create if necessary
│ └─ docker-compose.override.yml
└─ run-scan3-qom/      ← .env and override for node3-qom
├─ scan3.env.          ← Create if necessary
└─ docker-compose.override.yml
```

## 🚀 Blockscout setup procedure

1️⃣ **Prepare the Blockscout repository**
```bash
cd ~ 
git clone https://github.com/blockscout/blockscout.git
```

2️⃣ **Rewrite the CORS settings in Nginx (proxy)**

Edit the `default.conf.template` and `default.conf.template` files.

```bash
cd ~/blockscout/docker-compose/proxy

nano default.conf.template
```

```bash
# Change the following for ports 8080 and 8081:
add_header 'Access-Control-Allow-Origin' 'http://localhost' always;
↓
# ✏️After change
add_header 'Access-Control-Allow-Origin' '$http_origin' always;
```

-　`default.conf.template`
```bash
# Find this section
server {
    listen       8080;
    .
    .
    .
    # Change this line to the following:
    add_header 'Access-Control-Allow-Origin' '$http_origin' always;
    .
    .
    .
    location / {
      proxy_pass          http://stats:8050/;
      .
      .
      .
        }
}

server {
    listen       8081;
    .
    .
    .
    # Change this line to the following:
    add_header 'Access-Control-Allow-Origin' '$http_origin' always;
    .
    .
    .
    location / {
      proxy_pass          http://visualizer:8050/;
      .
      .
      .
      }
}
```

> You can use `$http_origin` to simply allow the origin of the request.

If you want to make the scan site public, you will also need to configure the following settings. (This is not necessary if you are running it locally.)

```bash
nano explorer.conf.template
```

- `explorer.conf.template`
```bash
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

server {
    listen       80;
    # use your domain here 
    server_name  test.qom.network;
    proxy_http_version 1.1;

    location = /stats { return 301 /stats/; }
    location ^~ /stats/ {
        proxy_pass http://stats:8050/;
        proxy_http_version 1.1;
        proxy_set_header Host "$host";
        proxy_set_header X-Real-IP "$remote_addr";
        proxy_set_header X-Forwarded-For "$proxy_add_x_forwarded_for";
        proxy_set_header X-Forwarded-Proto "$scheme";
    }

    location ^~ /api/v1/ {
        proxy_pass http://stats:8050/api/v1/;
        proxy_http_version 1.1;
        proxy_set_header Host "$host";
        proxy_set_header X-Real-IP "$remote_addr";
        proxy_set_header X-Forwarded-For "$proxy_add_x_forwarded_for";
        proxy_set_header X-Forwarded-Proto "$scheme";
    }

    location = /visualizer { return 301 /visualizer/; }
    location ^~ /visualizer/ {
        proxy_pass http://visualizer:8050/;
        proxy_http_version 1.1;
        proxy_set_header Host "$host";
        proxy_set_header X-Real-IP "$remote_addr";
        proxy_set_header X-Forwarded-For "$proxy_add_x_forwarded_for";
        proxy_set_header X-Forwarded-Proto "$scheme";
    }

    location ~ ^/(api(?!-docs$)|socket|sitemap.xml|auth/auth0|auth/auth0/callback|auth/logout) {
        proxy_pass            ${BACK_PROXY_PASS};
        proxy_http_version    1.1;
        proxy_set_header      Host "$host";
        proxy_set_header      X-Real-IP "$remote_addr";
        proxy_set_header      X-Forwarded-For "$proxy_add_x_forwarded_for";
        proxy_set_header      X-Forwarded-Proto "$scheme";
        proxy_set_header      Upgrade "$http_upgrade";
        proxy_set_header      Connection $connection_upgrade;
        proxy_cache_bypass    $http_upgrade;
    }

    location / {
        proxy_pass            ${FRONT_PROXY_PASS};
        proxy_http_version    1.1;
        proxy_set_header      Host "$host";
        proxy_set_header      X-Real-IP "$remote_addr";
        proxy_set_header      X-Forwarded-For "$proxy_add_x_forwarded_for";
        proxy_set_header      X-Forwarded-Proto "$scheme";
        proxy_set_header      Upgrade "$http_upgrade";
        proxy_set_header      Connection $connection_upgrade;
        proxy_cache_bypass    $http_upgrade;
    }
}
```

>Replace the value of `server_name` with your actual domain name, or `localhost` if you are planning to use it locally.

That's all for setting up the Blockscout directory.

---

## 🚀 Drive side configuration procedure (drive/services/node3-qom)

This section explains the setup procedure for connecting nodes and Blockscout only between Docker containers (without exposing the host).

> RPCs are not exposed to the host, meaning they cannot be accessed from the outside of the network. This means that only Blockscout can reach them via Docker's dedicated network.

### **Create a Docker shared network**

```bash
docker network create chains || true
```

1️⃣ **Add network settings to the qom container**
(Add network settings to the docker-compose.yml compose file in the drive)
```bash
cd ~/drive/services/node3-qom

nano docker-compose.yml
```

Add the following configuration:
```yaml
services:
  qom:
	networks:
      - chains
・
・
・
・
# Add the following near the end of the lines:
networks:
  chains:
    external: true

 ### End environment variables
  ### End service qom
### End docker-compose.yml
```

- Simply adding the above settings will not result in RPC (8545/8546) responding. This is most likely because JSON-RPC is bound only to 127.0.0.1 in the qom container. The following changes will allow access only from within the same network. (External exposure is not required.)


2️⃣ **Bind RPC to `0.0.0.0`** 
```bash
# Automatically determine the location of app.toml and rewrite
docker exec -it qom bash -lc '
CONFIG=/home/ubuntu/.qomd/config/app.toml; [ -f "$CONFIG" ] || CONFIG=/root/.qomd/config/app.toml;
cp "$CONFIG" "${CONFIG}.bak.$(date +%s)";
sed -i -E "s/^(address *= *\").*(:8545\".*)$/\10.0.0.0\2/" "$CONFIG";
sed -i -E "s/^(ws-address *= *\").*(:8546\".*)$/\10.0.0.0\2/" "$CONFIG";
sed -i -E "s/^(enable *= *).*/\1true/" "$CONFIG";
echo UPDATED: $CONFIG
'
````


3️⃣ **Restart the container**
```bash
# Restart the node (in the compose directory on the drive)
docker compose down

docker compose up -d
```


4️⃣ **Operation Check (from the chains network)**
Check whether block information can be accessed from the `chains` network you just configured.
```bash
# Name resolution + HTTP JSON-RPC response (5-second timeout, detailed display)
docker run --rm --network chains curlimages/curl:8.11.1 \
-v --max-time 5 -H 'Content-Type: application/json' \
-X POST http://qom:8545 \
-d '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}'

# Get block height
docker run --rm --network chains curlimages/curl:8.11.1 \
-s -H 'Content-Type: application/json' \
-X POST http://qom:8545 \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":2}'
```

It's OK if JSON like `{"result":"0x...","id":2,"jsonrpc":"2.0"}` is returned.

> Now **RPC is unreachable from the external internet, only Blockscout can connect directly to `qom:8545` through the network.**


### Create `docker-compose.override.yml`

1️⃣ **Create a configuration directory on drive**
```bash
mkdir -p ~/drive/services/scan3-qom 
```

2️⃣ **Create the `docker-compose.override.yml` and `.env` files**
```bash
cd ~/drive/services/scan3-qom

nano docker-compose.override.yml
```

Edit the file as follows:
```
networks:
  chains:
    external: true
    name: chains

services:
  backend:
    networks: [default, chains]
    environment:
      ETHEREUM_JSONRPC_HTTP_URL:  "http://qom:8545/"
      ETHEREUM_JSONRPC_WS_URL:    "http://qom:8546/"
      ETHEREUM_JSONRPC_TRACE_URL: "http://qom:8545/"
      NFT_MEDIA_HANDLER_ENABLED: "false"
      NFT_MEDIA_HANDLER_BACKFILL_ENABLED: "false"
      NFT_MEDIA_HANDLER_REMOTE_DISPATCHER_NODE_MODE_ENABLED: "false"
      INDEXER_DISABLE_PENDING_TRANSACTIONS_FETCHER: "true"
      CHAIN_ID: "766"
      NETWORK: "QOM"
      SUBNETWORK: "scan3-qom"

  frontend:
    environment:
      NEXT_PUBLIC_NETWORK_ID: "766"
      NEXT_PUBLIC_NETWORK_NAME: "QOM ONE"
      NEXT_PUBLIC_NETWORK_SHORT_NAME: "QL1"
      NEXT_PUBLIC_NETWORK_CURRENCY_SYMBOL: "QOM"
      NEXT_PUBLIC_NETWORK_CURRENCY_NAME: "QOM"
      NEXT_PUBLIC_APP_PROTOCOL: "https"
      NEXT_PUBLIC_APP_HOST: "test.qom.network"
      NEXT_PUBLIC_API_PROTOCOL: "https"
      NEXT_PUBLIC_API_HOST: "test.qom.network"
      NEXT_PUBLIC_API_WEBSOCKET_PROTOCOL: "wss"
      NEXT_PUBLIC_IS_TESTNET: "false"
      NEXT_PUBLIC_STATS_API_HOST: "https://test.qom.network/stats"
      NEXT_PUBLIC_VISUALIZE_API_HOST: "https://test.qom.network/visualizer"

  proxy:
    networks: [default, chains]
    ports:
      - "${FRONTEND_PORT:-8085}:80"

  nft_media_handler: { profiles: ["off"] }
  user-ops-indexer:  { profiles: ["off"] }
```

> **Configuration notes**
>  - Replace `test.qom.network` with your actual domain: 
> 	 - "NEXT_PUBLIC_APP_HOST" 
> 	 - "NEXT_PUBLIC_API_HOST"
> 	 - "NEXT_PUBLIC_STATS_API_HOST" 
> 	 - "NEXT_PUBLIC_VISUALIZE_API_HOST"
>   
>  `nft_media_handler` and `user-ops-indexer` may cause errors,
>    so we recommend keeping them disabled initially. 

<details>
<summary>Settings when running BlockScout locally</summary>

```yaml
networks:
  chains:
    external: true
    name: chains

services:
  backend:
    networks: [default, chains]
    environment:
      ETHEREUM_JSONRPC_HTTP_URL:  "http://qom:8545/"
      ETHEREUM_JSONRPC_WS_URL:    ""
      ETHEREUM_JSONRPC_TRACE_URL: "http://qom:8545/"
      NFT_MEDIA_HANDLER_ENABLED: "false"
      NFT_MEDIA_HANDLER_BACKFILL_ENABLED: "false"
      NFT_MEDIA_HANDLER_REMOTE_DISPATCHER_NODE_MODE_ENABLED: "false"
      INDEXER_DISABLE_PENDING_TRANSACTIONS_FETCHER: "true"
      CHAIN_ID: "766"
      NETWORK: "QOM"
      SUBNETWORK: "scan3-qom"

  frontend:
    environment:
      NEXT_PUBLIC_NETWORK_ID: "766"
      NEXT_PUBLIC_NETWORK_NAME: "QOM ONE"
      NEXT_PUBLIC_NETWORK_SHORT_NAME: "QL1"
      NEXT_PUBLIC_NETWORK_CURRENCY_SYMBOL: "QOM"
      NEXT_PUBLIC_NETWORK_CURRENCY_NAME: "QOM"
      NEXT_PUBLIC_APP_PROTOCOL: "http"
      NEXT_PUBLIC_APP_HOST: "YOUR_SERVER_IP"
      NEXT_PUBLIC_API_PROTOCOL: "http"
      NEXT_PUBLIC_API_HOST: "YOUR_SERVER_IP"
      NEXT_PUBLIC_IS_TESTNET: "false"
      NEXT_PUBLIC_STATS_API_HOST: "http://YOUR_SERVER_IP:8080"
      NEXT_PUBLIC_VISUALIZE_API_HOST: "http://YOUR_SERVER_IP:8081"

  proxy:
    networks: [default, chains]
    ports:
      - "${FRONTEND_PORT:-8085}:80"

  nft_media_handler: { profiles: ["off"] }
  user-ops-indexer:  { profiles: ["off"] }
  ```
> **Configuration notes**
> 
> - Replace `YOUR_SERVER_IP` with your server's IP address:
>     
>     - `NEXT_PUBLIC_APP_HOST`
>         
>     - `NEXT_PUBLIC_API_HOST`
>         
>     - `NEXT_PUBLIC_STATS_API_HOST`
>         
>     - `NEXT_PUBLIC_VISUALIZE_API_HOST`
>         
> - `nft_media_handler` and `user-ops-indexer` may cause errors,  
>     so we recommend keeping them disabled initially.     

</details> 

3️⃣ **Create a `.env` file**
```bash
nano .env
```

```.env
COMPOSE_PROJECT_NAME=scan3-qom
```

> By doing this, you can avoid having to specify the project name every time.


4️⃣ **Start Blockscout**
```bash
# up
docker compose \
  -f ~/blockscout/docker-compose/docker-compose.yml \
  -f ~/drive/services/scan3-qom/docker-compose.override.yml up -d
  
# down(if needed)
docker compose \
  -f ~/blockscout/docker-compose/docker-compose.yml \
  -f ~/drive/services/scan3-qom/docker-compose.override.yml down
```

> Specify the paths to `docker-compose.yml` and `docker-compose.override.yml` and execute it.
> Indexing will begin within a few minutes to a dozen minutes.

### Check the frontend display

Check the display in your browser
```
# replace your actual domain
https://test.qom.network

# If you run locally
http://<Your IP>:8085
```

>If it doesn't display properly, try to identify the problem using the error code or the `status check commands` below.
### 🧠 note

- The initial indexing will take some time. Don't worry if CPU and I/O usage reaches 100%.

- DB capacity will increase dramatically as the indexing progresses, so NVMe disks are recommended.

- You can monitor the load on each container with `docker stats`.

- The next time you start `node0-infinite`, create `scan0-infinite/` using the same procedure, changing only `.env` and the port number.

---
<details>
<summary>Status check commands </summary>

## 🔍 Status check commands

```bash
# Check container status
docker compose \
-f ~/blockscout/docker-compose/docker-compose.yml \
-f ~/drive/services/scan3-qom/docker-compose.override.yml ps

# Check communication with the backend
curl -s http://localhost:8085/api/v2/config/backend-version

# Display backend logs
docker compose \
-f ~/blockscout/docker-compose/docker-compose.yml \
-f ~/drive/services/scan3-qom/docker-compose.override.yml logs -f backend
```


---

## Check network connection status
```bash
# chains network presence
docker network ls | grep chains || true

# Is qom participating in chains?
docker network inspect chains | jq -r '.[0].Containers | to_entries[] | .value.Name' | grep -E '^qom$' || true

# Is the Blockscout backend participating in chains?
docker network inspect chains | jq -r '.[0].Containers | to_entries[] | .value.Name' | grep -E '^backend$' || true
```

## Backend → qom communication verification
```bash
# Obtain eth_blockNumber
docker compose \
  -f ~/blockscout/docker-compose/docker-compose.yml \
  -f ~/drive/services/scan3-qom/docker-compose.override.yml exec backend \
  curl -s -H 'Content-Type: application/json' \
       -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}' \
       http://qom:8545

# Get the latest block
docker compose \
  -f ~/blockscout/docker-compose/docker-compose.yml \
  -f ~/drive/services/scan3-qom/docker-compose.override.yml exec backend \
  curl -s -H 'Content-Type: application/json' \
       -d '{"jsonrpc":"2.0","id":2,"method":"eth_getBlockByNumber","params":["latest",true]}' \
       http://qom:8545
```

> If `result: "0x..."` is returned, **backend can reach qom via the chains network**.

## Check the status of block import into the database
```bash
# Indexer status
curl -s http://127.0.0.1:8085/api/v2/main-page/indexing-status | jq .

# Most recent blocks
curl -s "http://127.0.0.1:8085/api/v2/blocks?type=block&limit=5" | jq .

# How many blocks in the DB?
docker compose \
-f ~/blockscout/docker-compose/docker-compose.yml \
-f ~/drive/services/scan3-qom/docker-compose.override.yml exec db \
psql -U blockscout -d blockscout -c \
"SELECT COUNT(*) AS blocks, MAX(number) AS max_block FROM blocks;"
```

**Expected Results**
- `eth_blockNumber` in the backend responds.

- `blocks` starts to increase after a few minutes (`indexing-status` progresses).


If `eth_blockNumber` is not returned, the network may not be connected yet (`backend` is not in `chains`) or the `qom` side RPC may be closed. In that case, please double-check that **`backend` is listed** using `docker network inspect chains`.
</details>

---

## Setting up on Cloudflare

### Prerequisites

- The parent domain (e.g., `example.com`) is already registered with Cloudflare.

- Access to Cloudflare's DNS management screen.

---

**Step 1: Log in to Cloudflare**

1. Log in to [https://dash.cloudflare.com](https://dash.cloudflare.com)

2. Click the target **domain name (example.com)**

3. Select **DNS > Records** from the left menu.

**Step 2: Add a DNS record**

- Click the **Add record** button.

| Item         | Example                           |
| ------------ | --------------------------------- |
| Type         | **A**                             |
| Name         | `explorer` (example)              |
| IPv4 address | `xxx.xxx.xxx.xxx` (VPS global IP) |
| Proxy status | TTL                               |

👉 **`explorer.example.com`** will be created.

**Step 3: Save**

Simply click "`Save`" to complete the process. DNS updates usually take **a few seconds to a few minutes** to propagate.

**Step 4: Verify**

Execute the following command from your local computer or VPS.
```bash
dig explorer.example.com +short

# OR

ping explorer.example.com
```

> Replace `explorer.example.com` with your domain. If the VPS IP address is returned, the command is successful.

---
## Excluding Blockscout from Rate Limiting

**It is very common for Blockscout to fail to start or respond due to Cloudflare rate limiting (WAF rules).** If this occurs, please refer to the following settings.

### Adding Custom Rules

Cloudflare allows you to completely skip certain traffic using **custom rules**.

#### Steps

1. **Cloudflare Dashboard**

2. Select the target domain

3. **Security → Security rules** from the left menu

4. **Create rule → Custom rules**

#### Rule Settings (Example)

| Item | Example Input |
| ------------- | -------------------------- |
| Rule name | `Blockscout rate limiting` |
| Field | `Hostname` |
| Operator | `equals` |
| Value | `explorer.example.com` |
| Choose action | `skip` |

- ✅ All rate limiting rules
- ✅ All managed rules
- ✅ All Super Bot Fight Mode rules

- Press the `Deploy` button to create the rule

**All requests to Blockscout will now bypass Cloudflare's limits** (RPC / API / frontend are all stable)

---