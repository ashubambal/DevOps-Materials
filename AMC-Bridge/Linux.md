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

---

⭐ This document is part of a Linux Interview Preparation Series.