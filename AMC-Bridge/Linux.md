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
