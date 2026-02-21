# 📘 Linux Interview Questions & Answers

A structured collection of commonly asked **Linux interview questions** for DevOps, Cloud, and System Administration roles.

---

### 🐧 Q1: How can you show hidden files in Linux?

<details>
<summary><b>Click to Expand Answer</b></summary>

### 📌 Answer:

In Linux, hidden files and directories start with a dot (`.`).

To display hidden files, use the `-a` option with the `ls` command:

```bash
ls -a
```

---

## 🔍 Explanation

| Command | Description |
|----------|-------------|
| `ls` | Lists directory contents |
| `-a` | Shows all files including hidden ones |

Hidden files always begin with a dot (`.`).

Example hidden files:

```bash
.bashrc
.profile
.ssh/
.git/
.env
```

---

## 📂 Detailed View (Recommended in Production)

To see additional file details such as:

- File permissions  
- Ownership  
- File size  
- Last modified time  

Use:

```bash
ls -la
```

### Option Breakdown

| Option | Meaning |
|--------|---------|
| `-l` | Long listing format |
| `-a` | Includes hidden files |

---

## 📁 View Hidden Files in a Specific Directory

```bash
ls -la /etc/
```

This will list all files (including hidden ones) inside `/etc`.

---

## 🛠 Real-World DevOps Usage

Hidden files are commonly checked when:

- Debugging SSH issues (`~/.ssh/`)
- Reviewing Git repositories (`.git`)
- Checking environment variables (`.env`)
- Troubleshooting user shell configurations (`.bashrc`, `.profile`)
- Inspecting application configuration files

---

## 🎯 Interview-Ready Answer (Short Version)

> "In Linux, hidden files start with a dot. We use `ls -a` to display hidden files and `ls -la` to view detailed information such as permissions, ownership, and timestamps. Hidden files are commonly used for configuration in DevOps environments."

</details>

---

### 🐧 Q2: `df -h` Shows Free Space, But You Still Get "No Space Left on Device" — Why?

<details>
<summary><b>Click to Expand Answer</b></summary>

---

### 🎯 Short Explanation

If `df -h` shows plenty of free disk space but you still receive:

```
No space left on device
```

The most common reason is **inode exhaustion**.

---

## 🔍 Why This Happens

### 📦 What `df -h` Shows

```bash
df -h
```

- Displays disk usage in human-readable format
- Reports available disk space in bytes (blocks)

However…

> ⚠️ Every file also consumes an **inode** (metadata entry).

If all inodes are used, you cannot create new files — even if disk space is available.

---

## 🧠 Understanding Inodes

| Resource | Purpose |
|-----------|----------|
| Disk Blocks | Store actual file data |
| Inodes | Store metadata (owner, permissions, timestamps) |

Each file consumes:
- ✔ Disk space  
- ✔ One inode  

If inode usage reaches 100%, new file creation fails.

---

## 🛠 How to Confirm the Issue

Run:

```bash
df -i
```

Example:

```bash
Filesystem      Inodes   IUsed   IFree   IUse% Mounted on
/dev/xvda1      655360   655360     0    100%  /
```

If:

```
IUse% = 100%
```

👉 The filesystem has run out of inodes.

---

## 🚨 Other Possible Causes

### 1️⃣ User Quotas

Disk space may be available globally, but your user quota is exhausted.

```bash
quota -v
```

---

### 2️⃣ Reserved Blocks (ext4)

Around 5% of disk space is reserved for root.

Normal users may see "No space left", while root can still write.

Check with:

```bash
tune2fs -l /dev/xvda1 | grep "Reserved block count"
```

---

### 3️⃣ Filesystem Corruption (Rare)

Metadata inconsistencies can cause incorrect reporting.

```bash
fsck /dev/xvda1
```

⚠ Use carefully in production.

---

## 🔥 Real-World Production Example

> I faced this issue when log rotation failed and millions of small log files accumulated in `/var/log`.  
> The disk had free space, but inode usage was at 100%.  
> Cleaning unnecessary files resolved the issue immediately.

---

## 🎤 Concise Interview Delivery

> "Even if `df -h` shows free space, you can still get 'No space left on device' if the filesystem has run out of inodes. Each file consumes an inode, and if those are exhausted—often due to millions of small files—you can’t create new files. I would verify using `df -i` and clean up unnecessary logs or files."

---

</details>

---

## ☁️ Q3: Attached New EBS Volumes to EC2 But Not Showing in `df -h` — How to Use Them?

You have:

- Root volume: **8GB**
- Attached EBS volume 1: **12GB** → `/dev/sdb`
- Attached EBS volume 2: **20GB** → `/dev/sdc`

But when running:

```bash
df -h
```

The new volumes are not showing.

Also, you want applications like **Docker, tree, kubectl** to use the 12GB volume.

---

<details>
<summary><b>Click to Expand Answer</b></summary>

---

# 🎯 Why New Volumes Are Not Showing in `df -h`

`df -h` only shows **mounted filesystems**.

When you attach an EBS volume:
- ✅ It is attached at the OS level
- ❌ But it is NOT formatted
- ❌ And NOT mounted

So it won’t appear in `df -h`.

---

# 🛠 Step-by-Step Solution

---

## ✅ Step 1: Verify the Attached Disks

```bash
lsblk
```

You should see something like:

```
xvda    8G   (root)
xvdb   12G
xvdc   20G
```

If device name differs (like `/dev/nvme1n1`), that’s normal in newer instances.

---

## ✅ Step 2: Create Filesystem (Format the Volume)

⚠ Do this only once.

Example for 12GB volume:

```bash
sudo mkfs -t ext4 /dev/xvdb
```

For 20GB volume:

```bash
sudo mkfs -t ext4 /dev/xvdc
```

---

## ✅ Step 3: Create Mount Directory

```bash
sudo mkdir /data12
sudo mkdir /data20
```

---

## ✅ Step 4: Mount the Volumes

```bash
sudo mount /dev/xvdb /data12
sudo mount /dev/xvdc /data20
```

Now check:

```bash
df -h
```

Now the volumes will appear.

---

## ✅ Step 5: Make Mount Permanent (Important)

Edit `/etc/fstab`:

```bash
sudo vi /etc/fstab
```

Add:

```
/dev/xvdb  /data12  ext4  defaults,nofail  0  2
/dev/xvdc  /data20  ext4  defaults,nofail  0  2
```

Test:

```bash
sudo mount -a
```

---

# 🚀 How to Install Applications on 12GB Volume?

By default, applications install in:

```
/usr
/var
/opt
```

Which are on the root volume (8GB).

To use the 12GB volume, you have options:

---

## 🔹 Option 1: Move Specific Application Data (Recommended for Docker)

Example for Docker:

Stop Docker:

```bash
sudo systemctl stop docker
```

Move Docker directory:

```bash
sudo mv /var/lib/docker /data12/
```

Create symbolic link:

```bash
sudo ln -s /data12/docker /var/lib/docker
```

Start Docker:

```bash
sudo systemctl start docker
```

Now Docker uses the 12GB volume.

---

## 🔹 Option 2: Mount Volume Directly to a System Path

Example:

```bash
sudo mount /dev/xvdb /var/lib/docker
```

This directly makes Docker use that disk.

---

## 🔹 Option 3: Use LVM (Advanced Production Setup)

- Combine multiple volumes
- Extend root partition
- Better for scalable production environments

---

# 🔥 Real DevOps Production Approach

In production, we usually:

- Keep root volume minimal
- Attach separate volume for:
  - `/var`
  - `/var/lib/docker`
  - `/data`
  - Logs
- Use LVM for flexibility
- Ensure `/etc/fstab` is configured

---

# 🎤 Interview-Ready Answer (Concise Version)

> "Newly attached EBS volumes don’t appear in `df -h` because they are not formatted and mounted. I would verify using `lsblk`, create a filesystem using `mkfs`, mount it to a directory, and update `/etc/fstab` for persistence. To use the 12GB volume for Docker, I would move `/var/lib/docker` to the new mount or mount the volume directly to that path."

---

</details>

---