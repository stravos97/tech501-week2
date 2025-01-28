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

**Process Management Guide**

---

### 1. What is a Process?

- A **process** is a running instance of a program (e.g., a web browser, text editor).
- Each process has a unique **PID (Process ID)**.
- Processes exist in a hierarchy: **parent processes** create **child processes** (e.g., a terminal spawning a program).

---

### 2. Basic Commands to View Processes

#### `ps`

Lists running processes in your current terminal session:

`ps`

_Example Output:_

yaml

Copy

`PID   TTY      TIME     CMD 1234  pts/0    00:00:00 bash 5678  pts/0    00:00:00 ps`

- **PID**: Process ID.
- **TTY**: Terminal associated with the process.
- **CMD**: Command that started the process.

#### `ps -ef` or `ps aux`

View **all** system processes, including those from other users or background services:

`ps -ef` # Shows full-format listing  
`ps aux` # Shows detailed resource usage

#### `top`

Interactive real-time view of processes and resource usage:

`top`

- Press **`Shift + M`** to sort by **memory usage**.
- Press **`Shift + P`** to sort by **CPU usage**.
- Press **`q`** to quit.

---

### 3. Managing Processes

#### Terminate a Process

- **Graceful termination** (asks the process to close):
    
    `kill <PID>` or `kill -15 <PID>`
    
- **Force termination** (use if the process is unresponsive):
    
    `kill -9 <PID>`
    

#### Kill by Process Name

`killall <process_name>` # Kills all processes with this name  
`pkill <process_name>` # Flexible matching (e.g., partial names)

#### Find a Process

`pgrep firefox` # Get PID of Firefox  
`pidof firefox` # Alternative to pgrep

---

### 4. Background & Foreground Jobs

- **Run a command in the background**:
    
    `sleep 100 &` # The `&` runs it in the background
    
- **List background jobs** in your current terminal:
    
    `jobs -l` # Shows job numbers and PIDs
    
- **Bring a job to the foreground**:
    
    `fg %1` # Replace `1` with the job number
    
- **Pause/resume jobs**:
    
    - Press **`Ctrl + Z`** to pause a foreground job.
        
    - Resume it in the background with:
        
        `bg %1`
        

---

### 5. Parent-Child Relationships

- **PPID (Parent PID)**: The PID of the process that created a child process.
    
- View parent-child relationships:
    
    `ps -o pid,ppid,cmd` # Shows PID, PPID, and command
    

_Example Output:_

yaml

Copy

`PID   PPID  CMD 
5678   1234  /usr/bin/firefox`

---

### 6. Zombie Processes

#### What is a Zombie?

- A process that has finished but isn’t fully removed from the system.
- Shows as **`<defunct>`** or **`Z`** in `ps`/`top`.
- Harmless but consumes a PID slot.

#### Fix Zombies

1. Terminate the **parent process**:
    
    `kill <parent_PID>`
    
2. If that fails, reboot your system.
    

---

### 7. Key Signals for Process Control

|Signal|Name|Purpose|
|---|---|---|
|`SIGHUP` (1)|Hangup|Reload a process (e.g., servers)|
|`SIGINT` (2)|Interrupt|Stop a process (like `Ctrl+C`)|
|`SIGTERM` (15)|Terminate|Ask politely to exit (default)|
|`SIGKILL` (9)|Kill|Force immediate termination|
|`SIGSTOP`|Stop|Pause a process (`Ctrl+Z`)|
|`SIGCONT`|Continue|Resume a paused process|

---

### 8. Best Practices

- Use **`SIGTERM`** (`kill -15`) before **`SIGKILL`** to allow cleanup.
- Avoid `killall` if unsure—it can terminate multiple processes at once.
- Check for zombies occasionally with `ps aux | grep 'Z'`.

---

**Quick Reference Cheat Sheet**

`ps` # List your terminal's processes  
`top` # Live system monitor  
`kill -9 1234` # Force-kill PID 1234  
`jobs -l` # List background jobs  
`fg %2` # Bring job 2 to the foreground
