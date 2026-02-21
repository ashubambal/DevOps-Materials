# 📘 Linux Interview Q&A

---

## <span style="color:#2E86C1;">Q1: How can you show hidden files on Linux® systems?</span>

### ✅ Answer:

In Linux systems, hidden files are files or directories that start with a dot (`.`).

To display hidden files, we use the `-a` option with the `ls` command:

```bash
ls -a

🔍 Explanation:

ls → Lists directory contents

-a → Shows all files, including hidden ones

Hidden files begin with a dot (.)

📂 Detailed View (Recommended in Real Environments)

If you need additional details such as:

File permissions

Ownership

File size

Timestamps

You can combine -l (long listing) with -a:

```bash
ls -la

-l → Long listing format

-a → Includes hidden files

🛠 Real-World DevOps Usage

In practical DevOps environments, hidden files are commonly checked while:

Troubleshooting application configurations

Debugging SSH access issues

Reviewing user profile settings

Inspecting Git repositories

Common hidden files and directories:

```bash
.bashrc
.profile
.ssh/
.git/
.env

📁 View Hidden Files in a Specific Directory

To check hidden files inside a particular directory:

```bash
