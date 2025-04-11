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
    networks:
      - searxng  # Join SearXNG’s network    
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
    #cap_drop:
      #- ALL
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
### 4.Config Caddyfile 
- Open Caddyfile and replace with this:
  ```js
  {
    admin off
    log {
        output stderr
        format filter {
            request>remote_ip ip_mask 8 32
            request>client_ip ip_mask 8 32
            request>remote_port delete
            request>headers delete
            request>uri query {
                delete url
                delete h
                delete q
            }
        }
    }
  }
  :8081
    encode zstd gzip
	log {
        output stderr
        format json
    }
    @api {
        path /config
        path /healthz
        path /stats/errors
        path /stats/checker
    }
    @search {
        path /search
    }
    @imageproxy {
        path /image_proxy
    }
    @static {
        path /static/*
    }
    header {
        Content-Security-Policy "default-src 'none'; script-src 'self'; style-src 'self' 'unsafe-inline'; form-action 'self' https://github.com/searxng/searxng/issues/new; font-src 'self'; frame-ancestors 'self'; base-uri 'self'; connect-src 'self' https://overpass-api.de; img-src * data:; frame-src https://www.youtube-nocookie.com https://player.vimeo.com https://www.dailymotion.com https://www.deezer.com https://www.mixcloud.com https://w.soundcloud.com https://embed.spotify.com;"
        Permissions-Policy "accelerometer=(),camera=(),geolocation=(),gyroscope=(),magnetometer=(),microphone=(),payment=(),usb=()"
        Referrer-Policy "no-referrer"
        X-Content-Type-Options "nosniff"
        X-Robots-Tag "noindex, noarchive, nofollow"
        -Server
    }
    header @api {
        Access-Control-Allow-Methods "GET, OPTIONS"
        Access-Control-Allow-Origin "*"
    }
    route {
        header Cache-Control "max-age=0, no-store"
        header @search Cache-Control "max-age=5, private"
        header @imageproxy Cache-Control "max-age=604800, public"
        header @static Cache-Control "max-age=31536000, public, immutable"
        reverse_proxy searxng:8080 {
            header_up X-Forwarded-For {remote_host}
            header_up X-Forwarded-Port {http.request.port}
            header_up X-Real-IP {remote_host}
            header_up Host {host}
            header_up Connection "close"
            transport http {
                read_timeout 10s
                write_timeout 10s
                dial_timeout 5s			
            }
		
        }
    }    
  }
  ```
### 5.Start Docker Containers
- In the searxng-docker directory,( in ubuntu ) run:
```base
docker-compose up -d
```
### 6.Post-First-Run Security
- Stop the containers:
```base
docker-compose down
```
- Re-add cap_drop: - ALL to docker-compose.yaml for enhanced security, then restart by un comment:
```js
cap_drop:
  - ALL
```
- Start Docker Containers again
```bash
docker-compose up -d
# docker-compose up -d --force-recreate
docker ps
```
### 7.Test to use by open browser then input URL:
```
# open browser in windowns host
http://localhost:8080

# in other machine like mac , ipad, iphone : input ip of windows host:8081
http://192.168.1.35:8081
```




 
