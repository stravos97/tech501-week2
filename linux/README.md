**Application Deployment & VM Image Creation Guide**

_Streamlined process for deploying applications and creating reusable cloud images_

---

### Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Initial VM Setup](#2-initial-vm-setup)
3. [Application Deployment](#3-application-deployment)
4. [VM Imaging Preparation](#4-vm-imaging-preparation)
5. [Image Creation](#5-image-creation)
6. [Verification](#6-verification)
7. [File Transfer Guide](#7-file-transfer-guide)
8. [Troubleshooting](#8-troubleshooting)
9. [Appendix: Process Management](#9-appendix-process-management)

---

### 1. Prerequisites

**Checklist**

- **SSH Private Key**: `~/.ssh/<your-key>.pem`
    
- **VM Access Details**:
    
    |Field|Example Value|
    |---|---|
    |**Username**|`adminuser`|
    |**Public IP**|`203.0.113.45`|
    |**Local App Path**|`/home/user/my-app`|
    
- **Required Tools**:
    
    - **Terminal with SCP**: Git Bash or WSL recommended
    - **Git**: Installed locally and on VM

---

### 2. Initial VM Setup

#### Connect to VM

- Use SSH to connect to your VM:
    
    `ssh -i ~/.ssh/<your-key> adminuser@<vm-ip>`
    

#### Install Dependencies

- **Update system packages**:
    
    `sudo apt update && sudo apt upgrade -y`
    
- **Install essential tools**:
    
    `sudo apt install -y curl gnupg2`
    
- **Install Node.js 20.x**:
    
    `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`
    
    `sudo apt install -y nodejs`
    
- **Install NGINX**:
    
    `sudo apt install -y nginx`
    

---

### 3. Application Deployment

#### Transfer Application Code

**Option A: Git Clone**

- Clone the repository and navigate into it:
    
    `git clone https://github.com/<your-repo>.git`
    
    `cd <repo-name>`
    

**Option B: SCP Transfer**

- From your local machine, use SCP to transfer the application folder to the VM:
    
    `scp -i ~/.ssh/<your-key> -r /local/app/path adminuser@<vm-ip>:/home/adminuser/`
    

#### Run Application

- Navigate to the application directory, install dependencies, and start the application:
    
    `cd /home/adminuser/<app-directory>`
    
    `npm install`
    
    `npm start`
    

**Verify Operation**

- Check if the application is running correctly:
    
    `curl -I http://localhost:3000`
    
    _Expected Output_: `HTTP/1.1 200 OK`
    

---

### 4. VM Imaging Preparation

#### Standardize Directory Structure

- Create a standardized directory and move the application folder:
    
    `sudo mkdir /repo`
    
    `sudo mv /home/adminuser/<app-directory> /repo/`
    

#### Clean System for Imaging

- **Remove temporary files**:
    
    `sudo apt clean`
    
    `sudo rm -rf /tmp/*`
    
- **Azure-specific deprovision**:
    
    `sudo waagent -deprovision+user`
    

---

### 5. Image Creation

#### Azure Portal Steps

1. **Stop the VM**
    
2. **Navigate**: Virtual Machines → Capture
    
3. **Configuration**:
    
    - **Image Name**: `<your-image-name>`
    - **Generalize**: **Yes**
4. **Create new VMs from**: Images → Your Image
    

_For AWS/GCP: Use respective "Create Image" functions in cloud consoles_

---

### 6. Verification

#### Test New VM Instance

- SSH into the new VM and start the application:
    
    `ssh -i ~/.ssh/<your-key> adminuser@<new-vm-ip>`
    
    `cd /repo/<app-directory>`
    
    `npm start`
    

**Access Remotely**

- Open a web browser and navigate to:
    
    `http://<new-vm-ip>:3000`
    

---

### 7. File Transfer Guide

#### SCP Command Builder

- **Windows/WSL path conversion example**:
    
    `C:\Users\Alice\app` → `/mnt/c/Users/Alice/app`
    
- **SCP Command**:
    
    `scp -i ~/.ssh/<key> -r /local/path adminuser@<vm-ip>:/target/path/`
    

#### Post-Transfer Check

- Verify the files have been transferred successfully:
    
    `ssh adminuser@<vm-ip> "ls -l /target/path"`
    

---

### 8. Troubleshooting

|**Issue**|**Solution**|
|---|---|
|**Connection Refused**|Check security group/firewall rules for port `3000`|
|**`npm install` Errors**|Run `sudo npm cache clean -f` then retry|
|**Permission Denied**|`chmod 600 ~/.ssh/<your-key>`|
|**Application Not Running**|Check running processes: `ps aux|

---

### 9. Appendix: Process Management

#### Essential Commands

- **Run as background service**:
    
    `npm start &`
    
- **Persistent process management (install PM2)**:
    
    `sudo npm install -g pm2`
    
    `pm2 start app.js`
    
    `pm2 save`
    
    `pm2 startup`
    
