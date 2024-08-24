# lerning wireguard

To install WireGuard on Ubuntu 22.04, you can follow these steps:

### Step 1: Update Your System
First, ensure that your package list is up to date by running the following commands:

```bash
sudo apt update
sudo apt upgrade
```

### Step 2: Install WireGuard
WireGuard is included in the default Ubuntu repositories, so you can install it directly using the `apt` package manager:

```bash
sudo apt install wireguard
```

### Step 3: Generate Key Pairs
WireGuard requires a private and public key for both the server and clients. You can generate these keys using the `wg` command:

```bash
# Generate the server's private key
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```

This will generate two files: `privatekey` and `publickey` in the `/etc/wireguard/` directory.

### Step 4: Create the WireGuard Configuration File
Create a configuration file for your WireGuard interface, usually named `wg0.conf`:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Add the following content to the configuration file:

```ini
[Interface]
# The private key for the server
PrivateKey = <server-private-key>
# The IP address assigned to the WireGuard interface
Address = 10.0.0.1/24
# The port WireGuard listens on
ListenPort = 51820

# Enable IP forwarding
PostUp = sysctl -w net.ipv4.ip_forward=1
PostUp = iptables -A FORWARD -i %i -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o <external-interface> -j MASQUERADE
PostDown = sysctl -w net.ipv4.ip_forward=0
PostDown = iptables -D FORWARD -i %i -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -o <external-interface> -j MASQUERADE

[Peer]
# The public key of the client
PublicKey = <client-public-key>
# The IP addresses allowed to route through this peer
AllowedIPs = 10.0.0.2/32, 192.168.10.0/24
```

Replace `<server-private-key>` and `<client-public-key>` with the actual keys. Also, replace `<external-interface>` with your actual network interface name, which you can find using the `ip a` command (e.g., `eth0`, `ens33`).

### Step 5: Set Permissions for the Private Key
Ensure that the private key is accessible only by the root user:

```bash
sudo chmod 600 /etc/wireguard/privatekey
```

### Step 6: Enable IP Forwarding
To allow your server to route traffic between the WireGuard interface and other networks, enable IP forwarding:

1. Open the sysctl configuration file:

   ```bash
   sudo nano /etc/sysctl.conf
   ```

2. Find the line that says `#net.ipv4.ip_forward=1` and uncomment it (remove the `#`).

3. Save and exit, then apply the changes:

   ```bash
   sudo sysctl -p
   ```

### Step 7: Start and Enable WireGuard
Now you can start the WireGuard interface and enable it to start automatically on boot:

```bash
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

### Step 8: Verify the WireGuard Interface
You can verify that WireGuard is running and check the status of the interface with:

```bash
sudo wg
```

This command will show the current status of the WireGuard interface, including connected peers.

### Step 9: (Optional) Firewall Configuration
If you're using `ufw` (Uncomplicated Firewall) on your server, you may need to allow the WireGuard port:

```bash
sudo ufw allow 51820/udp
```

This allows traffic on the WireGuard port (51820 by default).

### Step 10: Client Configuration
After setting up the server, configure your clients with their respective private keys and the server's public key to establish a connection.

That's it! Your Ubuntu 22.04 server should now be running WireGuard, ready to accept connections from clients.
