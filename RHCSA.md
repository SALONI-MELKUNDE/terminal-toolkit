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

```bash
nmcli connection show
```

## 2. Set static IP, gateway, and DNS (replace <CONN_NAME> with actual name)

```bash
nmcli connection modify <CONN_NAME> \
  ipv4.addresses 172.24.X.5/24 \
  ipv4.gateway 172.24.X.254 \
  ipv4.dns 172.24.254.254 \
  ipv4.method manual
```

## 3. Set hostname

```bash
hostnamectl set-hostname node1.domainX.example.com
```

## 4. Apply the connection settings

```bash
nmcli connection down <CONN_NAME>
nmcli connection up <CONN_NAME>
```

## 5. Verify

```bash
1. ip a
2. ping -c 3 google.com
3. hostnamectl
```


###########################################################################################################


## 📦 2. Create Default YUM Repositories

Set up the default YUM repositories using the provided repository links:

- **Repo 1**: `http://<link_given>/BaseOS`  
- **Repo 2**: `http://<link_given>/AppStream`  

> ⚠️ **Ensure** that `gpgcheck` is disabled and both repositories are enabled.  
> This is required to install and update packages properly.

## Ans: 

### YUM Repo Setup

To configure the YUM repositories, follow these steps:

```bash
# Navigate to the yum repository directory
cd /etc/yum.repos.d
```

```bash
# Create a new repository configuration file
vim rhel.repo
```


Add the following content to the file:

```bash
[BaseOS]
name=RedHat Enterprise Linux 8.0 BaseOS
baseurl=http://<link_given>/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=RedHat Enterprise Linux 8.0 AppStream
baseurl=http://<link_given>/AppStream
enabled=1
gpgcheck=0
```

Save and exit:

```bash
:wq
```

Then refresh YUM and test:

```bash
yum clean all
yum update -y
yum repolist
yum install httpd -y   # (Optional test installation)
```

🧪 Tip: If everything is set up correctly, yum repolist should show both BaseOS and AppStream as available repositories. 

###########################################################################################################


## 👥 4. User/Group Management

Perform the following user and group configuration tasks:

- Create a group named `sysadmins`
- Create a user `natasha` and add her to `sysadmins` as a secondary group
- Create a user `harry` and add him to `sysadmins` as a secondary group
- Create a user `sarah` without interactive shell access, and do not add her to `sysadmins`
- Set the password for all three users to `postroll`

> 🛠️ **Ensure**
> - `natasha` and `harry` are part of `sysadmins` as secondary group members
> - `sarah` cannot log in interactively (use `/sbin/nologin`)
> - All passwords are set using `--stdin` method

## Ans: 

User and Group Setup

```bash
# Create the group
groupadd sysadmins
```

## Create users and assign them to sysadmins group as secondary group

```bash
useradd -G sysadmins natasha
useradd -G sysadmins harry
```

## Create sarah without interactive shell and no group membership

```bash
useradd -s /sbin/nologin sarah
```












