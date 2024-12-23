# Ethereum_Node_TD

# Ethereum Node Setup

This guide outlines the steps to set up an Ethereum node with the Execution Client (Geth) and Consensus Client (Lighthouse).

## Prerequisites

- A Linux-based system.
- Basic familiarity with the command line.
- Sufficient disk space for Ethereum data.

---

## Execution Client: Geth

### Installation
0. Go installation :

   ![Go installation](sreenshots/go_vers.png)
   
2. Clone the Geth repository:
    ```bash
    git clone https://github.com/ethereum/go-ethereum.git
    ```

3. Navigate to the cloned directory:
    ```bash
    cd go-ethereum
    ```

4. Build Geth:
    ```bash
    make geth
    ```
![Interface utilisateur](sreenshots/Installation_GETH.png)

### Starting Geth

Use the following command to start Geth:

```bash
./build/bin/geth \
    --authrpc.addr=localhost \
    --authrpc.jwtsecret=/home/administrateur1/.ethereum/geth/jwtsecret \
    --authrpc.port=8551 \
    --http.api eth,net,admin \
    --mainnet
```
![Interface utilisateur](sreenshots/GethBuild.png)
---

## Consensus Client: Lighthouse

### Switching to Lighthouse
Initially, Prysm was tested as the consensus client, but due to syncing issues, Lighthouse was chosen instead.

### Installation

1. Install Rust:
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    source $HOME/.cargo/env
    ```

2. Clone the Lighthouse repository:
    ```bash
    git clone https://github.com/sigp/lighthouse.git
    ```

3. Navigate to the cloned directory:
    ```bash
    cd lighthouse
    ```

4. Build Lighthouse:
    ```bash
    make
    ```

![Interface utilisateur](sreenshots/Cargo_lighthouse.png)

### JWT Creation

Create the JWT secret for communication between the execution and consensus clients:

```bash
mkdir -p ~/.ethereum
openssl rand -hex 32 | tr -d "\n" > ~/.ethereum/jwtsecret
```

### Starting Lighthouse

Command to start Lighthouse and begin syncing:

```bash
/home/administrateur1/.cargo/bin/lighthouse bn \
    --network mainnet \
    --execution-endpoint http://localhost:8551 \
    --execution-jwt /home/administrateur1/.ethereum/geth/jwtsecret \
    --checkpoint-sync-url https://beaconstate-mainnet.chainsafe.io \
    --disable-deposit-contract-sync
```


![Interface utilisateur](sreenshots/ligh_exec.png)
---

## Creating Systemd Services

To ensure Geth and Lighthouse run as background services, you can create systemd service files for them.

### Geth Service

1. Create a new service file:
    ```bash
    sudo nano /etc/systemd/system/geth.service
    ```

2. Add the following configuration:
    ```ini
    [Unit]
    Description=Geth Execution Client
    After=network.target

    [Service]
    ExecStart=/path/to/geth \
        --authrpc.addr=localhost \
        --authrpc.jwtsecret=/home/administrateur1/.ethereum/geth/jwtsecret \
        --authrpc.port=8551 \
        --http.api eth,net,admin \
        --mainnet
    Restart=always
    User=administrateur1

    [Install]
    WantedBy=multi-user.target
    ```

3. Reload and start the service:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl start geth.service
    sudo systemctl enable geth.service
    ```

![Interface utilisateur](sreenshots/Geth_service.png)

### Lighthouse Service

1. Create a new service file:
    ```bash
    sudo nano /etc/systemd/system/lighthouse.service
    ```

2. Add the following configuration:
    ```ini
    [Unit]
    Description=Lighthouse Consensus Client
    After=network.target

    [Service]
    ExecStart=/home/administrateur1/.cargo/bin/lighthouse bn \
        --network mainnet \
        --execution-endpoint http://localhost:8551 \
        --execution-jwt /home/administrateur1/.ethereum/geth/jwtsecret \
        --checkpoint-sync-url https://beaconstate-mainnet.chainsafe.io \
        --disable-deposit-contract-sync
    Restart=always
    User=administrateur1

    [Install]
    WantedBy=multi-user.target
    ```

3. Reload and start the service:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl start lighthouse.service
    sudo systemctl enable lighthouse.service
    ```

![Interface utilisateur](sreenshots/Light_service.png)

---

## Troubleshooting

- **Execution Client Issues:**
  - Verify that Geth is running and reachable on `localhost:8551`.
  - Check the logs with:
    ```bash
    journalctl -u geth.service -b
    ```

- **Consensus Client Issues:**
  - Ensure Lighthouse can communicate with Geth using the JWT secret.
  - Check the logs with:
    ```bash
    journalctl -u lighthouse.service -b
    ```

---

## References
- [Ethereum Go Client (Geth)](https://github.com/ethereum/go-ethereum)
- [Lighthouse Consensus Client](https://github.com/sigp/lighthouse)
