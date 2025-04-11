## Set up searNGX-Docker in windows 11 with WSL2

### Prerquisites
- WSL2
- Docker Desktop
- Ubuntu in WSL2: install git, openssl, libssl-dev, curl, net-tools : sudo usermod -aG docker @ubuntuUsername
- Enable docker, docker-compose in Docker Desktop work in Ubunto: Docker Desktop -> Settings -> Resources -> WSL integration -> Ubuntu-xxx checked

### Install searNGX
### 1.Clone the SearXNG Docker Repository
- Open a Command Prompt, PowerShell, or WSL2 terminal (Ubuntu).
- Navigate to a directory where you want to store the SearXNG files (e.g., C:\Users\YourUser\SearXNG):
```bash
# cd /mnt/c/Users/YourUser ## go to windows drive c:\Users\YoursUser
cd /mnt/f/Ai-Srv
mkdir SearXNG
cd SearXNG
```
- Clone the official SearXNG Docker repository:
```bash
git clone https://github.com/searxng/searxng-docker.git .
```
### 2.Configure SearXNG Settings
- Inside the searxng-docker directory, locate the searxng/settings.yml file.
- Generate a secure secret key (best practice for security):
-- In your WSL2 terminal, run:





 
