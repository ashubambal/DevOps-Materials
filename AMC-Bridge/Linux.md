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

### 🐧 Q3: Attached New EBS Volumes to EC2 But Not Showing in `df -h` — How to Use Them?

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
xvda      8G   (root)
nvme1n1   12G
nvme2n1   20G
```

If device name differs (like `/dev/nvme1n1`), that’s normal in newer instances.

---

## ✅ Step 2: Create Filesystem (Format the Volume)

⚠ Do this only once.

Example for 12GB volume:

```bash
sudo mkfs -t ext4 /dev/nvme1n1
```

For 20GB volume:

```bash
sudo mkfs -t ext4 /dev/nvme2n1
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
sudo mount /dev/nvme1n1 /data12
sudo mount /dev/nvme2n1 /data20
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
/dev/nvme1n1  /data12  ext4  defaults,nofail  0  2
/dev/nvme2n1  /data20  ext4  defaults,nofail  0  2
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
sudo mount /dev/nvme1n1 /var/lib/docker
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

### 🐧 Q4: What System Diagnosis Tools Can You Name? What Does "Load Average" Mean in `top`?

<details>
<summary><b>Click to Expand Answer</b></summary>

---

# 🛠️ System Diagnosis Tools (Linux)

In production environments, system diagnosis tools help identify:

- CPU bottlenecks  
- Memory leaks  
- Disk I/O issues  
- Network congestion  
- Stuck or zombie processes  

---

## 📊 Commonly Used Tools

| Tool | Purpose | Production Use Case |
|------|----------|--------------------|
| `top` / `htop` | Real-time CPU, memory, load average, process monitoring | Identify high CPU-consuming process |
| `vmstat` | Memory, swap, CPU, I/O statistics | Detect memory pressure or swapping |
| `iostat` | Disk I/O monitoring | Diagnose disk latency issues |
| `sar` | Historical performance data | Analyze past CPU spikes |
| `netstat` / `ss` | Network connections & ports | Detect suspicious connections |
| `dstat` | Combined CPU, disk, network stats | Quick system health overview |
| `strace` | System call tracing | Debug stuck process |
| `lsof` | List open files | Identify file locks |
| `free` | Memory usage | Check RAM availability |
| `uptime` | Uptime & load average | Quick health snapshot |

---

# 🔥 Production Scenarios (Real Examples)

---

## 🚨 Scenario 1: High CPU Usage

Problem:
Application is slow.

Command Used:
```bash
top
```

What You Check:
- `%CPU`
- Load average
- Which process is consuming CPU

Example:
Java process consuming 350% CPU on 4-core machine → CPU bottleneck identified.

---

## 💾 Scenario 2: Application Crashing Due to Memory

Command Used:
```bash
free -m
vmstat 1
```

What You Check:
- Available memory
- Swap usage
- si/so (swap in/out)

If swap usage is high → Memory leak suspected.

---

## 💿 Scenario 3: Disk I/O Bottleneck

Command Used:
```bash
iostat -x 1
```

What You Check:
- %util near 100%
- High await time

If disk utilization is constantly 100% → storage bottleneck.

---

## 🌐 Scenario 4: Suspicious Network Traffic

Command Used:
```bash
ss -tulnp
```

What You Check:
- Open ports
- Unexpected established connections

Used during security investigation.

---

## 🧵 Scenario 5: Process Hanging

Command Used:
```bash
strace -p <PID>
```

Used to trace system calls and identify where process is stuck.

---

# 📈 What Is Load Average in `top`?

When you run:

```bash
top
```

You will see:

```
load average: 0.75, 1.20, 0.90
```

---

## 📌 Definition

Load average represents:

> The average number of processes that are either:
> - Running on CPU  
> - Waiting for CPU  

---

## ⏱️ The Three Numbers Represent

| Value | Meaning |
|--------|---------|
| 1st | Last 1 minute average |
| 2nd | Last 5 minutes average |
| 3rd | Last 15 minutes average |

---

## 🧠 How to Interpret Load Average

Compare load average with number of CPU cores.

### ✅ Healthy System

If:

```
Load Average ≤ Number of CPU cores
```

System is stable.

Example:
- 4-core CPU
- Load average: 2.0  
→ System is comfortable.

---

### ❌ Overloaded System

If:

```
Load Average > Number of CPU cores
```

Processes are waiting in run queue.

Example:
- 4-core CPU
- Load average: 10.0  
→ Heavy overload, performance degradation.

---

# 🔥 Real Production Example

In one production case:

- Load average suddenly increased to 15.0 on 4-core server
- `top` showed high `%wa` (I/O wait)
- `iostat` confirmed disk utilization was 100%
- Root cause: Log file flooding disk
- Solution: Clean logs + optimize log rotation

---

# 🎤 Concise Interview Delivery

> "For system diagnosis, I use tools like top, htop, vmstat, iostat, sar, and ss depending on the issue. In top, load average shows the average number of processes running or waiting for CPU over 1, 5, and 15 minutes. If load average exceeds the number of CPU cores, it indicates system overload."

---

# 💡 Interview Tip

Always:
- Mention real production scenario
- Compare load average with CPU cores
- Explain both CPU-bound and I/O-bound cases

This shows real-world experience.

---

</details>

---

### 💽 Q5: What If a Disk Is Corrupted or Failed? How Do You Retrieve Data?

<details>
<summary><b>Click to Expand Answer</b></summary>

---

# 🎯 Overview

When a disk becomes corrupted or fails, recovery depends on the type of failure:

| Failure Type | Description |
|--------------|------------|
| 🧠 Logical Failure | Filesystem corruption, deleted files, partition damage |
| 💿 Hardware Failure | Bad sectors, disk I/O errors |
| 🔧 Severe Physical Damage | Motor failure, head crash, PCB damage |

Correctly identifying the failure type is critical before taking action.

---

# 🧠 1️⃣ Logical Corruption (Filesystem Issues)

### Examples:
- Accidental file deletion  
- Corrupted filesystem metadata  
- Lost partitions  

---

## 🚨 First Rule

> Always mount the disk as **read-only** to prevent further damage.

```bash
mount -o ro /dev/sdb1 /mnt/recovery
```

---

## 🔧 Recovery Tools

| Tool | Purpose |
|------|----------|
| `fsck` | Repair filesystem inconsistencies |
| `testdisk` | Recover lost partitions & boot sectors |
| `photorec` | Recover deleted files at block level |

---

## 🛠 Example Scenario

Problem:
`/var` partition corrupted after improper shutdown.

Steps:
1. Unmount the partition
2. Run:

```bash
fsck -y /dev/sdb1
```

If recovery fails:

```bash
testdisk
```

For deleted file recovery:

```bash
photorec
```

---

# 💿 2️⃣ Hardware Failure (Bad Sectors / Failing Drive)

Symptoms:
- I/O errors in logs  
- Slow disk performance  
- System freezing  

---

## 🔍 Check Disk Health

```bash
smartctl -a /dev/sdb
```

Look for:
- Reallocated sector count
- Pending sectors
- SMART failure warnings

---

## 🚨 Critical Step

> Never work directly on a failing disk.

Always clone it first.

---

## 🧰 Use `ddrescue` to Clone

```bash
ddrescue /dev/sdb /dev/sdc rescue.log
```

- Clones disk
- Skips bad sectors
- Retries later

After cloning:
- Perform recovery on the cloned disk
- Run `fsck`, `testdisk` on the copy

---

## 🛠 Production Example

Disk showing repeated I/O errors.

Steps taken:
1. Checked SMART status
2. Cloned using `ddrescue`
3. Recovered files from cloned image
4. Replaced failing disk

Zero additional data loss.

---

# 🔧 3️⃣ Severe Physical Failure

Symptoms:
- Clicking noise
- Disk not detected
- Burnt PCB smell
- Motor failure

At this stage:

> ❌ Software tools will NOT work.

Only solution:
- Send disk to professional recovery lab
- Clean-room facility required

Used in:
- Banking
- Healthcare
- Critical compliance environments

---

# 🔥 Senior-Level Best Practices

## ✅ Always Image First
Never attempt repair before creating a disk image.

## ✅ Identify Failure Type
Logical vs Physical — shows troubleshooting maturity.

## ✅ Prevention Is Better Than Recovery

In production environments:

- Use RAID
- Enable automatic backups
- Monitor SMART alerts
- Monitor I/O errors in logs
- Implement snapshot strategies (EBS snapshots in AWS)

---

# 📊 Real Production Architecture Prevention

Example setup:

- RAID 1 for redundancy
- Daily backups
- Weekly offsite backup
- SMART monitoring alerts
- Cloud snapshot automation

This prevents catastrophic loss.

---

# 🎤 Concise Interview Delivery

> "If a disk is corrupted, I first determine whether the issue is logical or physical. For logical issues, I mount it read-only and use tools like fsck, testdisk, or photorec. For hardware failure, I clone the disk using ddrescue and recover data from the copy. For severe physical damage, professional recovery services are required. The key principle is to always image the disk first before attempting recovery."

---

# 💡 Interview Tip

To impress senior panels:
- Mention SMART monitoring
- Mention ddrescue
- Mention RAID & backups
- Emphasize prevention strategy

This demonstrates production-level experience.

---

</details>

---