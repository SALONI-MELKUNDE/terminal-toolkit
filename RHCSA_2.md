
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
-         → First Sector: <Enter>
-         → Last Sector: +512M
- t       → type → 82 (Linux swap)
- w       → write changes


3. Apply changes
   ```bash
   partprobe
   lsblk
   ```
   
4. Format the new partition as swap
   ```bash
   mkswap /dev/vdb2
   ```

5. Get UUID
   ```bash
   blkid
   ```

6. Edit fstab and add the UUID
   ```bash
   vim /etc/fstab
   ```

7. Add line like:
   ```bash
   UUID="your-uuid-here" swap swap defaults 0 0
   ```

8. Save and exit
   ```bash
   :wq
   ```

9. Activate swap
   ```bash
   swapon -a
   swapon -s
   ```

10. Final check
    ```bash
    df -h
    free -h
    ```

###########################################################################################################

## 📏 3. Resize Logical Volume to 800MB

Resize an existing **logical volume** (e.g., `/dev/myvg/mylv`) to **800MB**.  
It should be able to store data between **765MB to 800MB** safely.

### 📁 Logical Volume Details

- **Volume Group**: `myvg`
- **Logical Volume**: `mylv`
- **Target Size**: 800MB
- **Command Used**: `lvextend` and `resize2fs`

> 🛠️ **Ensure**
> - The logical volume is resized to exactly 800MB
> - You use `resize2fs` to grow the file system
> - Data is not lost during resizing
> - You verify the volume size using `lvs` or `df -h`

## ✅ Verification Steps

1. Use `lvdisplay` to confirm the new size.
2. Run `df -h` to check filesystem space.
3. Ensure the volume can store 765MB to 800MB of data.

## Ans:

1. Check current disk usage
   ```bash
   df -h
   ```

2. Display logical volume details
   ```bash
   lvdisplay
   ```

3. Resize the logical volume
   ```bash
   lvextend -L 800M /dev/myvg/mylv
   ```

4. Resize the filesystem on the LV
   ```bash
   resize2fs /dev/myvg/mylv
   ```

5. Verify volume again
   ```bash
   lvdisplay
   df -h
   ```

###########################################################################################################




