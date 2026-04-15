When you are building an enterprise-grade server, you never use the default `apt install docker` or `apt install npm` commands. The default OS repositories usually contain severely outdated versions of these tools. 

Furthermore, security is paramount. Giving the `jenkins` user raw `sudo` access is considered a massive security risk, but there is a specific, restricted way to do it if your pipelines absolutely require it.

Here is the standard and most reliable way to install the latest version of Docker Engine on Ubuntu using Docker's official repository.

### **Step 1: Update and install prerequisites**
First, update your existing list of packages and install the necessary dependencies that allow `apt` to access repositories over HTTPS.

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
```

### **Step 2: Add Docker’s official GPG key**
Set up the keyring directory and download Docker's GPG key so your system trusts the packages.

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### **Step 3: Add the Docker repository**
Add the official Docker repository to your system's package sources. This command automatically detects your Ubuntu version and architecture.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### **Step 4: Install Docker Engine**
Update your package index again (to recognize the new repository) and install Docker Engine, the CLI, containerd, and Docker Compose.

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### **Step 5: Verify the installation**
Check that Docker is installed and running by executing the built-in `hello-world` image. 

```bash
sudo docker run hello-world
```
If the installation was successful, you will see a "Hello from Docker!" message indicating that the Docker daemon is working correctly.

---

### **Optional: Run Docker without `sudo`**
By default, running `docker` commands requires administrator privileges (`sudo`). If you want to run Docker as a non-root user, add your user to the `docker` group:

```bash
sudo usermod -aG docker $USER
```
*Note: You will need to log out and log back in (or restart your terminal) for these group changes to take effect.*

### Step 2: The Permissions (Jenkins, Docker, and Sudo)
*Correction Note:* You actually want to add the **`jenkins` user to the `docker` group**, not the other way around. This allows Jenkins to run Docker commands without needing sudo passwords.

```bash
# 1. Add Jenkins to the Docker group
sudo usermod -aG docker jenkins

# 2. Apply the group changes immediately to the Jenkins service
sudo systemctl restart jenkins
```

**The Sudo Warning:** Giving Jenkins full `sudo` means if a hacker breaches your pipeline, they own the entire server. If you *must* do it, you must configure passwordless sudo specifically for the Jenkins user.

```bash
# Open the sudoers file securely
sudo visudo
```
*Scroll to the bottom of the file and add this exact line, then save and exit:*
```text
jenkins ALL=(ALL) NOPASSWD: ALL
```

### Step 3: Install Python (With Virtual Environments)
In an enterprise, you never install Python packages globally using `pip install`. It corrupts the server OS. You install the virtual environment module (`venv`) so your pipelines can build isolated Python sandboxes.

```bash
# Install Python 3, the package manager (pip), and the venv module
sudo apt-get install -y python3 python3-pip python3-venv
```

### Step 4: Install NPM and Node.js (The NodeSource Way)
If you run `apt install npm`, you will get a version of Node that is years old. The enterprise standard is to use **NodeSource** to pull the current LTS (Long Term Support) version (e.g., Version 20).

```bash
# 1. Download and run the NodeSource setup script for Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 2. Install Node.js (This automatically installs the correct, matching version of NPM)
sudo apt-get install -y nodejs
```

---

### Verify the Architecture
Run this block to confirm everything is installed at the correct, enterprise-level versions:

```bash
docker --version
python3 --version
node --version
npm --version
```
