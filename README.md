When you are building an enterprise-grade server, you never use the default `apt install docker` or `apt install npm` commands. The default OS repositories usually contain severely outdated versions of these tools. 

Furthermore, security is paramount. Giving the `jenkins` user raw `sudo` access is considered a massive security risk, but there is a specific, restricted way to do it if your pipelines absolutely require it.

Here is the exact, enterprise-standard bash execution to install everything correctly on an Ubuntu/Debian server.
The easiest and most reliable way to install Docker on Ubuntu is through the official Docker repository to ensure you get the latest stable version. [1, 2] 
## 1. Remove conflicting packages
Before starting, uninstall any unofficial or old Docker packages to avoid conflicts: [3] 

sudo apt-get remove docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc

## 2. Set up the repository [4] 
Install the necessary prerequisites and add Docker's official GPG key: [1, 5] 

# Update package list and install requirements
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
# Add GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

## 3. Install Docker Engine [3] 
Update your package index again and install the latest Docker components: [1, 2] 

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

## 4. Verify the installation
Run the hello-world image to confirm Docker is installed and running correctly: [3, 6] 

sudo docker run hello-world

## Optional: Run Docker without sudo [1] 
By default, only the root user can run Docker commands. To run them as a normal user, add yourself to the docker group: [1, 7, 8] 

sudo usermod -aG docker $USER

Note: You must log out and log back in (or restart) for these changes to take effect. [1, 9] 
------------------------------
I can provide more specific instructions if you're interested in:

* Setting up Docker Desktop for a graphical interface.
* Configuring Docker Compose for multi-container apps.
* Enabling GPU support for AI/ML workloads. [7, 10, 11] 


[1] [https://www.digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)
[2] [https://www.interserver.net](https://www.interserver.net/tips/kb/how-to-install-docker-on-ubuntu-24-04-lts/)
[3] [https://docs.docker.com](https://docs.docker.com/engine/install/ubuntu/)
[4] [https://www.youtube.com](https://www.youtube.com/watch?v=vCyf89qTdVo)
[5] [https://www.theserverside.com](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/How-to-install-Docker-and-docker-compose-on-Ubuntu)
[6] [https://www.youtube.com](https://www.youtube.com/watch?v=TBTJsM1Jhno)
[7] [https://www.digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)
[8] [https://cloudinfrastructureservices.co.uk](https://cloudinfrastructureservices.co.uk/how-to-install-and-use-docker-engine-on-ubuntu-22-04/#:~:text=By%20default%2C%20only%20the%20root%20user%20or,run%20all%20Docker%20commands%20without%20root%20privileges.)
[9] [https://docs.docker.com](https://docs.docker.com/engine/install/linux-postinstall/)
[10] [https://docs.docker.com](https://docs.docker.com/desktop/setup/install/linux/ubuntu/)
[11] [https://www.digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)


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
