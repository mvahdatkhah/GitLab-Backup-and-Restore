# 🚀 GitLab Backup and Restore

## 📌 Overview
This repository provides scripts and instructions for **backing up and restoring a GitLab instance**, including repositories, CI/CD data, configuration files, and secrets.

## 🎯 Features
✅ Full GitLab backup using `gitlab-backup`
✅ Automated scheduled backups with cron 🕒
✅ Secure storage of configuration and secrets 🔐
✅ Step-by-step restore instructions 🛠️
✅ **Keep only the latest 10 backups** and automatically delete older ones 🗑️

---

## 🛡️ Backup Process

### 1️⃣ **Run a Full GitLab Backup**
```bash
sudo gitlab-backup create
```
📂 By default, backups are stored in:
```
/var/opt/gitlab/backups/
```
This command creates a compressed archive of all GitLab data, including repositories, database, and uploads.

### 2️⃣ **Backup Configuration and Secrets**
```bash
sudo cp /etc/gitlab/gitlab.rb /var/opt/gitlab/backups/
sudo cp /etc/gitlab/gitlab-secrets.json /var/opt/gitlab/backups/
```
🛠️ These files contain important settings and secrets used by GitLab. Keeping them safe ensures a successful restore.

### 3️⃣ **Schedule Automated Backups with Cleanup**
Edit the crontab:
```bash
sudo crontab -e
```
Add the following line to schedule a **daily backup at 2:00 AM** and keep only the last 10 backups:
```bash
0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1 && cp /etc/gitlab/gitlab.rb /var/opt/gitlab/backups/ && cp /etc/gitlab/gitlab-secrets.json /var/opt/gitlab/backups/ && find /var/opt/gitlab/backups/ -name '*.tar' -type f -printf '%T@ %p\n' | sort -n | head -n -10 | cut -d' ' -f2- | xargs -r rm >> /var/log/gitlab-backup.log 2>&1
```

📖 **Explanation:**
- 🕒 `0 2 * * *` → Runs every day at **2:00 AM**
- 📦 `gitlab-backup create CRON=1` → Creates a new backup
- 🔐 `cp /etc/gitlab/gitlab.rb` and `cp /etc/gitlab/gitlab-secrets.json` → Copies GitLab configuration files
- 🗑️ `find ... | sort -n | head -n -10 | cut -d' ' -f2- | xargs -r rm` → Deletes backups older than the last 10
- 📜 `>> /var/log/gitlab-backup.log 2>&1` → Saves output to log file

---

## 🔄 Restore Process

### 1️⃣ **Move Backup File to the Correct Directory**
If the backup is on another server, transfer it:
```bash
scp backup_filename.tar root@your_gitlab_server:/var/opt/gitlab/backups/
```
📂 This ensures the backup is in the correct location before starting the restore process.

### 2️⃣ **Stop GitLab Services**
```bash
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
```
🛑 Stopping these services prevents conflicts while restoring data.

### 3️⃣ **Restore the Backup**
```bash
sudo gitlab-backup restore BACKUP=<timestamp>
```
📦 Example:
```bash
sudo gitlab-backup restore BACKUP=1707600000_2024_02_11
```
This command restores all repositories, database, and uploads from the backup.

### 4️⃣ **Restore Configuration and Secrets**
```bash
sudo cp /var/opt/gitlab/backups/gitlab.rb /etc/gitlab/gitlab.rb
sudo cp /var/opt/gitlab/backups/gitlab-secrets.json /etc/gitlab/gitlab-secrets.json
```
🛠️ Restoring these files ensures GitLab runs with the correct settings and encryption keys.

Reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```
⚙️ This step applies the restored configuration.

### 5️⃣ **Restart GitLab**
```bash
sudo gitlab-ctl restart
```
🔄 GitLab must be restarted for the restored data to take effect.

### 6️⃣ **Verify the Restore**
```bash
sudo gitlab-rake gitlab:check
sudo gitlab-ctl tail
```
✅ Running these checks helps ensure everything is working correctly after restoration.

---

## 📝 Notes
- 🔐 Ensure backups are stored in a **secure location** (e.g., external storage, S3, or another server).
- ☁️ If using **object storage**, configure it in `/etc/gitlab/gitlab.rb` before backup/restore.
- ⚡ For **High Availability (HA) setups**, back up all nodes separately.
