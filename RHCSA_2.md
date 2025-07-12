
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

## 📂 4. Create Logical Volume `datastore` in VG `database`

Create a new logical volume named `datastore` inside the volume group `database` with the following requirements:

### 📁 Configuration Details

- Create a partition of **1 GB** on `/dev/vdb` (type `8e`)
- Initialize it as a physical volume
- Volume group name: `database` with physical extent size **16 MB**
- Logical volume: `datastore` of **50 extents** or **800 MB**
- Format it with **VFAT**
- Mount it at `/common/classes`
- Ensure it auto-mounts after reboot via `/etc/fstab`

> 🛠️ **Ensure**
> - Logical volume is created with `50` extents or `800M`  
> - Physical extent size is set to `16M` during VG creation  
> - Filesystem type is **VFAT**
> - Entry in `/etc/fstab` ensures persistence after reboot

## ✅ Verification Steps

1. Run `lvdisplay` and `df -h` to verify logical volume and mount.
2. Use `mount -a` to check for errors in `/etc/fstab`.
3. Reboot and confirm the mount persists.

## Ans:

1. Create partition of 1GB on /dev/vdb and set type to 8e (Linux LVM)
   ```bash
   fdisk /dev/vdb
   ```

2. Initialize physical volume
   ```bash
   pvcreate /dev/vdb3
   ```

3. Create volume group with 16MB extent size
   ```bash
   vgcreate -s 16M database /dev/vdb3
   ```

4. Create logical volume with 50 extents
   ```bash
   lvcreate -n datastore -l 50 database
   ```
   Or alternatively:
   ```bash
   lvcreate -n datastore -L 800M database
   ```
   
5. Format with vfat filesystem
   ```bash
   mkfs.vfat /dev/database/datastore
   ```

6. Create mount point
   ```bash
   mkdir -p /common/classes
   ```

7. Add entry to /etc/fstab for automount
   ```bash
   vim /etc/fstab
   ```
   > - Add the following line:
     ```bash
     /dev/mapper/database-datastore  /common/classes  vfat  defaults 0 0
     :wq
     ```

8. Mount all entries
   ```bash
   mount -a
   ```

9. Confirm
   ```bash
   df -h
   ```
###########################################################################################################

## 🎛️ 5. Set Tuned Profile to Recommended

Change the system’s Tuned profile to the one suggested under the **recommended** category.

### 📁 Configuration Details

- Use `tuned-adm` to list and identify recommended profile
- Enable and start the `tuned` service
- Apply the recommended profile (e.g., `virtual-guest`)

> 🛠️ **Ensure**
> - Tuned is installed and running
> - The system is set to use the recommended tuned profile
> - The change is persistent across reboots

---

## ✅ Verification Steps

1. Run `tuned-adm recommend` to identify the recommended profile.
2. Use `tuned-adm active` to confirm the currently active profile.
3. Confirm service is enabled with `systemctl is-enabled tuned`.

## Ans:

1. Install tuned package
   ```bash
   yum install tuned -y
   ```

2. Enable and start the service
   ```bash
   systemctl enable tuned
   systemctl start tuned
   ```
   
3. List available tuned profiles
   ```bash
   tuned-adm list
   ```

4. Display the recommended profile
   ```bash
   tuned-adm recommend
   ```

5. Show the currently active profile
   ```bash
   tuned-adm active
   ```

6. Set the recommended profile (example: virtual-guest)
   ```bash
   tuned-adm profile virtual-guest
   ```

###########################################################################################################

## 📁 6. Prepare Persistent Storage for Rootless Container

Create a directory `/srv/web` and extract the archive from `/home/containers/web-content.gz` into it.  
Configure the directory so that a **rootless container** can use it for persistent storage.  
Install `container-tools` and `podman`.

### 📦 Task Requirements

- Extract HTML content from `/home/containers/web-content.gz` to `/srv/web`
- Make the directory owned by user/group `containers`
- Install necessary container tools

> 🛠️ **Ensure**
> - Ownership is set properly (`containers:`)  
> - Directory exists and contains `html` subdirectory  
> - Podman and container-tools are installed  
> - Suitable for rootless container mounting

## ✅ Verification Steps

1. Run `ls -ld /srv/web` to check ownership and permissions.
2. Confirm `html` directory exists inside `/srv/web`.
3. Verify container-tools and `podman` are installed with `rpm -qa | grep podman`.

## Ans:

1. Create and navigate to the directory
   ```bash
   mkdir -pv /srv/web
   cd /srv/web
   ```
2. Extract archive contents
   ```bash
   tar -xvzf /home/containers/web-content.gz
   ```

3. Verify extracted content (should include html/)
   ```bash
   ls
   ```

4. Set ownership to containers user/group
   ```bash
   chown -R containers: /srv/web
   ```

5. Confirm ownership and permissions
   ```bash
   ls -ld /srv/web
   ```

6. Install container tools and Podman
   ```bash
   yum module install container-tools -y
   yum install podman -y
   ```

###########################################################################################################

## 🐳 8. Deploy Apache HTTP Container Using Podman

Using the `containers` user, deploy an Apache HTTP server container named `web` with the following specifications.

### 📦 Deployment Details

- **Image**: `registry.lab.example.com/rhel8/httpd-24:1-105`
- **Container Name**: `web`
- **Port Mapping**: Host port `8888` → Container port `8080`
- **Volume Mount**: Host path `/srv/web` → Container path `/var/www`
- **Environment Variable**: `HTTPD_MPM=event`
- **Startup Behavior**: Configure `systemd` to start the container at boot (user service)

> 🛠️ **Ensure**
> - The container runs in **detached mode**
> - It uses the correct image with tag `1-105`
> - Volume mount and port mapping are correctly applied
> - Environment variable is passed
> - `systemd --user` service is created for auto-start


## ✅ Verification Steps

1. Run `podman ps` to confirm the container is running.
2. Use `curl localhost:8888` to check if Apache is serving.
3. Reboot and ensure the container starts via `systemctl --user status container-web`.

## Ans:



