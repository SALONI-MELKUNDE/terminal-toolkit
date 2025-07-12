# Section 1



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

1. Check available connections
   ```bash
   nmcli connection show
   ```

2. Set static IP, gateway, and DNS (replace <CONN_NAME> with actual name)
   ```bash
   nmcli connection modify <CONN_NAME> \
     ipv4.addresses 172.24.X.5/24 \
     ipv4.gateway 172.24.X.254 \
     ipv4.dns 172.24.254.254 \
     ipv4.method manual
   ```

3. Set hostname
   ```bash
   hostnamectl set-hostname node1.domainX.example.com
   ```

4. Apply the connection settings
   ```bash
   nmcli connection down <CONN_NAME>
   nmcli connection up <CONN_NAME>
   ```

5. Verify
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

1. To configure the YUM repositories, follow these steps:
   ```bash
   # Navigate to the yum repository directory
   cd /etc/yum.repos.d
   ```

   ```bash
   # Create a new repository configuration file
   vim rhel.repo
   ```


2. Add the following content to the file:
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

3. Save and exit:
   ```bash
   :wq
   ```

4. Then refresh YUM and test:
   ```bash
   yum clean all
   yum update -y
   yum repolist
   yum install httpd -y   # (Optional test installation)
   ```

🧪 Tip: If everything is set up correctly, yum repolist should show both BaseOS and AppStream as available repositories. 

###########################################################################################################


## 👥 3. User/Group Management

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

1. User and Group Setup
   ```bash
   # Create the group
   groupadd sysadmins
   ```

2. Create users and assign them to sysadmins group as secondary group
   ```bash
   useradd -G sysadmins natasha
   useradd -G sysadmins harry
   ```

3. Create sarah without interactive shell and no group membership
   ```bash
   useradd -s /sbin/nologin sarah
   ```

4. Set passwords for all users
   ```bash
   echo "postroll" | passwd --stdin natasha
   echo "postroll" | passwd --stdin harry
   echo "postroll" | passwd --stdin sarah
   ```

🧪 Verify

- Use id natasha and id harry to confirm group membership.
- Use getent passwd sarah to confirm shell assignment.

###########################################################################################################

## 📁 4. Automount User Home Directory via NFS

Configure `Node1` such that `remoteuserX` having home directory `/rhome/remoteuserX` gets mounted automatically upon login.  
The NFS share would be: `utility.domainX.example.com:/rhome/remoteuserX`

> 🛠️ **Ensure**  
> - Automount is enabled using `autofs`  
> - The user's home directory is mounted via NFS  
> - `/etc/auto.master` and `/etc/auto.misc` are configured correctly  
> - `nfs4` is used as the file system type  
> - The service starts automatically on boot

## Ans:

### 🧪 Steps to Configure Automount for `/rhome/remoteuserX`

1. Install autofs
   ```bash
   yum install autofs -y
   ```

2. Edit the master map file `/etc/auto.master`
   Add the following line:
   ```bash
   /-  /etc/auto.misc
   ```

3. Edit the map file /etc/auto.misc
   Add the following line:
   ```bash
   /rhome/remoteuserX  -rw,sync,fstype=nfs4  utility.domainX.example.com:/rhome/remoteuserX
   ```
   
4. Enable and restart autofs service
   ```bash
   systemctl enable autofs
   systemctl restart autofs
   ```
   
5. Verify automount by switching user
   ```bash
   su - remoteuserX
   ```

###########################################################################################################

## 🔍 6. Locate and Copy All Files Owned by User `student`

Find all files on the system that are owned by the user `student`, and copy them into the directory `/var/liststationx`.

> 🛠️ **Ensure**  
> - All files owned by the user `student` are identified  
> - A directory `/var/liststationx` is created  
> - Files are copied with attributes preserved  

## Ans:

### 🧪 Steps to Locate and Copy Files of User `student`

1. Create target directory
   ```bash
   mkdir /var/liststationx
   ```
   
2. Find and copy files owned by student
   ```bash
   find / -user student -exec cp -avp {} /var/liststationx/ \;
   ```

###########################################################################################################

## ⏰ 8. Configure Cron Job for User `natasha`

The user **Natasha** must configure a cron job that runs **daily at 06:25 local time** and executes the command:  
`/bin/echo "Hello Test"`

> 🛠️ **Ensure**
> - The cron job runs at the correct time
> - It runs under the `natasha` user's crontab
> - The `crond` service is running

## Ans:

### 🧪 Steps to Configure the Cron Job

1. Edit Natasha's crontab
   ```bash
   crontab -eu natasha
   ```
   
2. Add the following line to schedule the job at 06:25 daily
   ```bash
   25 06 * * * /bin/echo Hello_World
   ```
   
3. Restart the cron service
   ```bash
   systemctl restart crond
   ```
   
4. Verify the crontab job for Natasha
   ```bash
   crontab -lu natasha
   ```
   
### 🔁 If the same job is to run every minute from 06:00 to 06:59:
   ```bash
   */1 06 * * * /bin/echo Hello_World
   ```

###########################################################################################################

## 🗂️ 9. Copy and Configure Permissions for `/var/tmp/fstab`

Copy the file `/etc/fstab` to `/var/tmp`. Configure the permissions of `/var/tmp/fstab` so that:

- The file `/var/tmp/fstab` is **owned by the root user**.
- The file belongs to the **group root**.
- The file is **not executable by anyone**.
- The user **Natasha** can **read and write** to `/var/tmp/fstab`.
- The user **Harry** can **neither read nor write** to `/var/tmp/fstab`.
- **All other users** (current or future) can **read** `/var/tmp/fstab`.

## Ans:

### 🧪 Steps to Copy and Set ACL Permissions

```bash
cp /etc/fstab /var/tmp/

setfacl -m u:natasha:rw- /var/tmp/fstab
setfacl -m u:harry:---   /var/tmp/fstab
setfacl -m o::r--        /var/tmp/fstab

getfacl /var/tmp/fstab
```
###########################################################################################################

## 🕒 10. Configure System as an NTP Client of `classroom.example.com`

Configure your system so that it synchronizes time with the NTP server `classroom.example.com`.

> 🛠️ **Ensure**
> - The server is added to `chrony.conf`
> - The chronyd service is restarted
> - Time synchronization is verified

## Ans:

### 🧪 Steps to Configure NTP Client

1. Edit the chrony configuration file
   ```bash
   vim /etc/chrony.conf
   ```

2. Update the configuration
   - Comment out any existing server lines (add # at the beginning).
   - Add the new server:
     ```bash
     server classroom.example.com iburst
     ```
     
3. Save and exit
   ```bash
   :wq
   ```

4. Restart chronyd service
   ```bash
   systemctl restart chronyd.service
   ```

5. Check if time is synchronized
   ```bash
   timedatectl
   ```

6. Verify NTP source
   ```bash
   chronyc sources -v
   ```

Output: 
```bash
MS Name/IP address      Stratum Poll Reach LastRx Last sample
===============================================================================
^* classroom.example.com     8     6    17     3    +1783ns[+133us] +/- 265us
```

###########################################################################################################

## 👤 11. Create a User `stationX` Without Login Access and UID 1088

Create a user `stationX` on your system which has **no login access** and is assigned **user ID 1088**.

## Ans:

### 🧪 Command to Create the User

```bash
useradd -u 1088 -s /sbin/nologin stationX
```
- 🔒 `-u 1088` sets the user ID
- 🔒 `-s /sbin/nologin` disables login for the user





