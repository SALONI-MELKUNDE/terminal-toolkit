
# Section 2


> ⚠️ **Note:**  
> Break root password as given in *Additional Configuration*.  
> **Do not touch Network Settings of Node2.**



## 📦 1. Repository Setup

Configure your system to enable access to the **BaseOS** and **AppStream** repositories using a mounted ISO/DVD located at `/dev/sr0`.

### 📁 Repository Details:

- **Mount Point**: `/mnt`
- **BaseOS Repo Path**: `/etc/yum.repos.d/BaseOS.repo`
- **AppStream Repo Path**: `/etc/yum.repos.d/AppStream.repo`
- **Base URLs**:  
  - `file:///mnt/BaseOS`  
  - `file:///mnt/AppStream`
- **GPG Check**: Disabled (`gpgcheck=0`)

> 🛠️ **Ensure**
> - The ISO/DVD is mounted at `/mnt`
> - Both repositories are created using `.repo` files in `/etc/yum.repos.d/`
> - Repositories are enabled and visible with `dnf repolist`


## ✅ Verification Steps

1. Run `dnf repolist` to check that `BaseOS` and `AppStream` are listed.
2. Use `ls /etc/yum.repos.d/` to verify the `.repo` files exist.
3. View file content with `cat` to confirm correct `baseurl` values.


## Ans:

```bash
mount /dev/sr0 /mnt

# BaseOS repository
cat <<EOF > /etc/yum.repos.d/BaseOS.repo
[BaseOS]
name=BaseOS
baseurl=file:///mnt/BaseOS
enabled=1
gpgcheck=0
EOF

# AppStream repository
cat <<EOF > /etc/yum.repos.d/AppStream.repo
[AppStream]
name=AppStream
baseurl=file:///mnt/AppStream
enabled=1
gpgcheck=0
EOF
```

Verify repo availability
```bash
dnf repolist
```

###########################################################################################################

## 💾 2. Extend Swap Partition to 512M

Extend a **swap partition** of size **512 MB** on `/dev/vdb` and ensure it is **automounted after reboot**.

### 📁 Partition and Swap Details

- **Disk**: `/dev/vdb`
- **Partition**: `/dev/vdb2`
- **Partition Type**: Linux swap (ID 82)
- **Size**: +512M
- **Mount Info**: `/etc/fstab` with UUID

> 🛠️ **Ensure**
> - The swap partition is exactly 512 MB  
> - Proper partition type is used (`82`)  
> - Entry is added in `/etc/fstab` using UUID  
> - Swap is activated and persists after reboot

## ✅ Verification Steps

1. Use `lsblk` or `free -h` to check memory and swap.
2. Use `swapon -s` or `df -h` to verify the partition.
3. Reboot and confirm swap is active with `free -h`.

## Ans:

1. Check current memory and disk
   ```bash
   free -h
   lsblk
   ```

2. Partition the disk
   ```bash
   fdisk /dev/vdb
   ```

- In fdisk prompt:
- p       → print partitions
- n       → new partition
-          → First Sector: <Enter>
-          → Last Sector: +512M
- t       → type → 82 (Linux swap)
- w       → write changes


