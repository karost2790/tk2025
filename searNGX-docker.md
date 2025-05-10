# searxng-docker

Create a new SearXNG instance in five minutes using Docker

## What is included?

| Name | Description | Docker image | Dockerfile |
| -- | -- | -- | -- |
| [Caddy](https://github.com/caddyserver/caddy) | Reverse proxy (create a LetsEncrypt certificate automatically) | [docker.io/library/caddy:2-alpine](https://hub.docker.com/_/caddy)           | [Dockerfile](https://github.com/caddyserver/caddy-docker/blob/master/Dockerfile.tmpl) |
| [SearXNG](https://github.com/searxng/searxng) | SearXNG by itself                                              | [docker.io/searxng/searxng:latest](https://hub.docker.com/r/searxng/searxng) | [Dockerfile](https://github.com/searxng/searxng/blob/master/Dockerfile)               |
| [Valkey](https://github.com/valkey-io/valkey) | In-memory database                                             | [docker.io/valkey/valkey:8-alpine](https://hub.docker.com/r/valkey/valkey)        | [Dockerfile](https://github.com/valkey-io/valkey-container/blob/mainline/Dockerfile.template)             |

## How to use it
There are two ways to host SearXNG. The first one doesn't require any prior knowledge about self-hosting and thus is recommended for beginners. It includes caddy as a reverse proxy and automatically deals with the TLS certificates for you. The second one is recommended for more advanced users that already have their own reverse proxy (e.g. Nginx, HAProxy, ...) and probably some other services running on their machine. The first few steps are the same for both installation methods however.

1. [Install docker](https://docs.docker.com/install/)
2. Get searxng-docker
  ```sh
  cd /usr/local
  git clone https://github.com/searxng/searxng-docker.git
  cd searxng-docker
  ```
3. Edit the [.env](https://github.com/searxng/searxng-docker/blob/master/.env) file to set the hostname and an email
4. Generate the secret key `sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml`
   On a Mac: `sed -i '' "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml`
5. Edit [searxng/settings.yml](https://github.com/searxng/searxng-docker/blob/master/searxng/settings.yml) according to your needs

> [!NOTE]
> On the first run, you must remove `cap_drop: - ALL` from the `docker-compose.yaml` file for the `searxng` service to successfully create `/etc/searxng/uwsgi.ini`. This is necessary because the `cap_drop: - ALL` directive removes all capabilities, including those required for the creation of the `uwsgi.ini` file. After the first run, you should re-add `cap_drop: - ALL` to the `docker-compose.yaml` file for security reasons.

> [!NOTE]
> Windows users can use the following powershell script to generate the secret key:
> ```powershell
> $randomBytes = New-Object byte[] 32
> (New-Object Security.Cryptography.RNGCryptoServiceProvider).GetBytes($randomBytes)
> $secretKey = -join ($randomBytes | ForEach-Object { "{0:x2}" -f $_ })
> (Get-Content searxng/settings.yml) -replace 'ultrasecretkey', $secretKey | Set-Content searxng/settings.yml
> ```

### Method 1: With Caddy included (recommended for beginners)
6. Run SearXNG in the background: `docker compose up -d`

### Method 2: Bring your own reverse proxy (experienced users)
6. Remove the caddy related parts in `docker-compose.yaml` such as the caddy service and its volumes.
7. Point your reverse proxy to the port set for the `searxng` service in `docker-compose.yml` (8080 by default).
8. Generate and configure the required TLS certificates with the reverse proxy of your choice.
9. Run SearXNG in the background: `docker compose up -d`

> [!NOTE]
> You can change the port `searxng` listens on inside the docker container (e.g. if you want to operate in `host` network mode) with the `BIND_ADDRESS` environment variable (defaults to `0.0.0.0:8080`). The environment variable can be set directly inside `docker-compose.yaml`.

## Troubleshooting - How to access the logs

To access the logs from all the containers use: `docker compose logs -f`.

To access the logs of one specific container:

- Caddy: `docker compose logs -f caddy`
- SearXNG: `docker compose logs -f searxng`
- Valkey: `docker compose logs -f redis`

### Start SearXNG with systemd

You can skip this step if you don't use systemd.
1. Copy the service template file:
   ```sh
   cp searxng-docker.service.template searxng-docker.service
   ```

2. Edit the content of ```WorkingDirectory``` in the ```searxng-docker.service``` file (only if the installation path is different from ```/usr/local/searxng-docker```)

3. Enable the service:
   ```sh
   systemctl enable $(pwd)/searxng-docker.service
   ```

4. Start the service:
   ```sh
   systemctl start searxng-docker.service
   ```

**Note:** Ensure the service file path matches your installation directory before enabling it.

## Note on the image proxy feature

The SearXNG image proxy is activated by default.

The default [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) allows the browser to access to ```${SEARXNG_HOSTNAME}``` and ```https://*.tile.openstreetmap.org;```.

If some users want to disable the image proxy, you have to modify [./Caddyfile](https://github.com/searxng/searxng-docker/blob/master/Caddyfile). Replace the ```img-src 'self' data: https://*.tile.openstreetmap.org;``` by ```img-src * data:;```.

## Multi Architecture Docker images

Supported architecture:

- amd64
- arm64
- arm/v7

## How to update ?

To update the SearXNG stack:

```sh
git pull
docker compose pull
docker compose up -d
```

Or the old way (with the old docker-compose version):

```sh
git pull
docker-compose pull
docker-compose up -d
```



---
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




 
