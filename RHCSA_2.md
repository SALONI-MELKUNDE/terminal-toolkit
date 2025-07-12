
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

## Verify repo availability
```bash
dnf repolist
```


