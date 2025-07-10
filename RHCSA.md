## 📡 1. Network Setup

Configure `node1.domainX.example.com` with the following static network settings:

- **IP Address**: `172.24.X.5`  
- **Netmask**: `255.255.255.0`  
- **Gateway**: `172.24.X.254`  
- **DNS**: `172.24.254.254`  

> 🛠️ **Ensure**  
> - The hostname is correctly set as `node1.domainX.example.com`  
> - You apply the static IP without deleting any existing interface  
> - Use `nmcli`, `hostnamectl`, and `systemctl` appropriately

---

### 🧪 Verification Steps

1. Run `nmcli` or `ip a` to confirm the IP settings.  
2. Use `ping` to test external and DNS connectivity.  
3. Use `hostnamectl status` to confirm the hostname.

## Ans: 

## 1. Check available connections
nmcli connection show

## 2. Set static IP, gateway, and DNS (replace <CONN_NAME> with actual name)
nmcli connection modify <CONN_NAME> \
  ipv4.addresses 172.24.X.5/24 \
  ipv4.gateway 172.24.X.254 \
  ipv4.dns 172.24.254.254 \
  ipv4.method manual

## 3. Set hostname
hostnamectl set-hostname node1.domainX.example.com

## 4. Apply the connection settings
nmcli connection down <CONN_NAME>
nmcli connection up <CONN_NAME>

## 5. Verify
1. ip a
2. ping -c 3 google.com
3. hostnamectl


###########################################################################################################


## 📦 2. Create Default YUM Repositories

Set up the default YUM repositories using the provided repository links:

- **Repo 1**: `http://<link_given>/BaseOS`  
- **Repo 2**: `http://<link_given>/AppStream`  

> ⚠️ **Ensure** that `gpgcheck` is disabled and both repositories are enabled.  
> This is required to install and update packages properly.

## Ans: 




