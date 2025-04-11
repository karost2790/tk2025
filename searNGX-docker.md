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
```
# cd /mnt/c/Users/YourUser ## go to windows drive c:\Users\YoursUser
cd /mnt/f/Ai-Srv
mkdir SearXNG
cd SearXNG
```




 
