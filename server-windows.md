On a Windows server, you can still configure WireGuard to route traffic to a specific network, such as `192.168.10.0/24`. Here’s how to set up the configuration file on the server side:

### Step 1: Install WireGuard

1. Download and install WireGuard from the official [WireGuard website](https://www.wireguard.com/install/).

### Step 2: Create the WireGuard Configuration File

1. **Create a New Configuration File**:
   - Open Notepad (or any text editor) and create a new configuration file named `wg0.conf`.

2. **Edit the Configuration File**:

```ini
[Interface]
# The private key for the server
PrivateKey = <server-private-key>
# The IP address assigned to the WireGuard interface
Address = 10.0.0.1/24
# The port WireGuard listens on
ListenPort = 51820

# Enable IP forwarding (you'll need to set this separately on Windows)
# PostUp = (Not applicable in Windows)

[Peer]
# The public key of the client
PublicKey = <client-public-key>
# The IP addresses allowed to route through this peer
AllowedIPs = 10.0.0.2/32, 192.168.10.0/24
```

Replace `<server-private-key>` and `<client-public-key>` with your actual keys.

### Step 3: Enable IP Forwarding on Windows

1. **Enable IP Forwarding via Registry**:
   - Open the Registry Editor (`regedit`).
   - Navigate to:
     ```
     HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
     ```
   - Set the `IPEnableRouter` value to `1`.
   - Restart your computer to apply the changes.

2. **Set Up NAT (Optional, if you need internet access through VPN)**:
   - Open PowerShell as Administrator.
   - Run the following command to set up NAT for the external network interface:
     ```powershell
     New-NetNat -Name "WireGuardNAT" -InternalIPInterfaceAddressPrefix "10.0.0.0/24" -ExternalIPInterfaceAddressPrefix "<external-ip-range>"
     ```
   Replace `<external-ip-range>` with the range of IP addresses that should be accessible outside of the VPN.

### Step 4: Start WireGuard

1. **Open WireGuard**: Launch the WireGuard app on your Windows server.
2. **Import the Configuration**: Click “Import tunnel(s) from file” and select the `wg0.conf` file you created.
3. **Activate the Tunnel**: Toggle the switch to activate the tunnel.

### Notes:
- **Firewall Settings**: Ensure that your Windows Firewall allows traffic on the WireGuard port (`51820` in this example). You might need to create inbound and outbound rules for this.
- **Routing Table**: If you need to add additional routes, you can use the `route` command in Windows.

This configuration will set up the WireGuard server on Windows to allow clients to connect and route traffic to the `192.168.10.0/24` network. The IP forwarding and NAT configuration will allow connected clients to access the specified network.
