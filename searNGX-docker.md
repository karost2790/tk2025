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
  - In your WSL2 terminal, run:
  ```bash
  openssl rand -hex 32
  # then you will key:
  7a4e790126cc826606f619869ce2a471018b1ee15fa1d804210318e0ce9b8762
  ```
- replace config in settings.yaml:

  ```bash
  # see https://docs.searxng.org/admin/settings/settings.html#settings-use-default-settings
  use_default_settings: true
  server:
  # base_url is defined in the SEARXNG_BASE_URL environment variable, see .env and docker-compose.yml
  port: 8080              # Matches your docker-compose.yaml
  bind_address: "0.0.0.0" # Allows access beyond localhost
  secret_key: "< YOUR NEW KEY >"  # change this!
  limiter: false  # can be disabled for a private instance
  public_instance: false   # Enables public API access
  image_proxy: true
  # Search settings
  search:
    safe_search: 0          # No filtering, full results
    autocomplete: ""        # Disable autocomplete for API simplicity
    default_lang: "en"      # Default language
    formats:                # Ensure JSON is supported
      - html
      - json
      - rss
      - csv
  # Outgoing settings (for external sources)
  outgoing:
    request_timeout: 5.0    # Increase timeout for external fetches
    pool_connections: 100   # More connections for parallel queries
    pool_maxsize: 10        # Connection pool size    
  ui:
    static_use_hash: true
  redis:
    url: redis://redis:6379/0
  ```
    - Copy the secure secret key into settings.yml-> secret_key:
    ```bash
    bind_address: "0.0.0.0" # Allows access beyond localhost
    secret_key: "7a4e790126cc826606f619869ce2a471018b1ee15fa1d804210318e0ce9b8762"  # change this!
    limiter: false  # can be disabled for a private instance
    ```
### 3.Config docker-compose.yaml file 
- open docker-compose.yaml and replace with this:
  ```js
  services:
  caddy:
    container_name: caddy
    image: docker.io/library/caddy:2-alpine
    ports:
      - "8081:8081"
    restart: unless-stopped
    volumes:
      - /mnt/f/Ai-srv/SearXNG/Caddyfile:/etc/caddy/Caddyfile  # Host path : Container path
      - caddy-data:/data:rw
      - caddy-config:/config:rw
    environment:
      - SEARXNG_HOSTNAME=localhost
      - SEARXNG_TLS=internal
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"
  searxng:
    container_name: searxng
    image: docker.io/searxng/searxng:latest
    restart: unless-stopped
    networks:
      - searxng
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - ./searxng:/etc/searxng:rw
    environment:
      - SEARXNG_BASE_URL=http://localhost/
      - UWSGI_WORKERS=4
      - UWSGI_THREADS=4
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
    depends_on:
      - redis
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"
  redis:
    container_name: redis
    image: redis:7-alpine
    restart: unless-stopped
    networks:
      - searxng
    volumes:
      - redis-data:/data
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"
  networks:
    searxng:
  volumes:
    caddy-data:
    caddy-config:
    redis-data:
  ```




 
