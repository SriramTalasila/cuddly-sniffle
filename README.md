# Mosquitto MQTT Broker on Docker (Ubuntu)

## 1. Create Project Directory

```bash
mkdir -p ~/server/configs/mqtt
cd ~/server/configs/mqtt
```

---

## 2. Create Directory Structure

```bash
mkdir -p mosquitto/config
mkdir -p mosquitto/data
mkdir -p mosquitto/log
```

Verify:

```bash
tree .
```

Expected:

```text
.
├── docker-compose.yaml
└── mosquitto
    ├── config
    ├── data
    └── log
```

---

## 3. Create Mosquitto Configuration

Create:

```bash
nano mosquitto/config/mosquitto.conf
```

Contents:

```conf
persistence true
persistence_location /mosquitto/data/

listener 1883

allow_anonymous false
password_file /mosquitto/config/passwd
```

---

## 4. Create Docker Compose File

Create:

```bash
nano docker-compose.yaml
```

Contents:

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto
    container_name: mosquitto
    restart: unless-stopped

    ports:
      - "1883:1883"
      - "9001:9001"

    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
```

---

## 5. Create Log File

Mosquitto may fail if the configured log file does not exist.

```bash
touch mosquitto/log/mosquitto.log
```

Verify:

```bash
ls -l mosquitto/log
```

---

## 6. Set Permissions

### Log Directory

```bash
chmod 777 mosquitto/log
chmod 666 mosquitto/log/mosquitto.log
```

### Config Directory

```bash
chmod 755 mosquitto/config
```

### Data Directory

```bash
chmod 755 mosquitto/data
```

### If Files Are Owned by Root

Check ownership:

```bash
ls -lR mosquitto
```

Fix ownership:

```bash
sudo chown -R $USER:$USER mosquitto
```

---

## 7. Start Mosquitto

```bash
docker compose up -d
```

Verify:

```bash
docker ps
```

Expected:

```text
STATUS
Up
```

---

## 8. Create MQTT User

Generate password file:

```bash
docker exec -it mosquitto \
  mosquitto_passwd -c /mosquitto/config/passwd iot_user
```

Enter password when prompted.

Verify:

```bash
ls -l mosquitto/config/passwd
```

---

## 9. Set Password File Permissions

If needed:

```bash
sudo chown $USER:$USER mosquitto/config/passwd
chmod 644 mosquitto/config/passwd
```

Verify:

```bash
ls -l mosquitto/config/passwd
```

Expected:

```text
-rw-r--r--
```

---

## 10. Restart Mosquitto

```bash
docker restart mosquitto
```

Verify:

```bash
docker ps
```

Expected:

```text
STATUS
Up
```

---

## 11. Check Logs

View startup logs:

```bash
docker logs mosquitto
```

Follow logs:

```bash
docker logs -f mosquitto
```

Expected:

```text
mosquitto version 2.x running
Opening ipv4 listen socket on port 1883
Opening ipv6 listen socket on port 1883
```

---

## 12. Test Subscription

Terminal 1:

```bash
docker exec -it mosquitto \
  mosquitto_sub \
  -h localhost \
  -u iot_user \
  -P YOUR_PASSWORD \
  -t test/topic
```

---

## 13. Test Publishing

Terminal 2:

```bash
docker exec -it mosquitto \
  mosquitto_pub \
  -h localhost \
  -u iot_user \
  -P YOUR_PASSWORD \
  -t test/topic \
  -m "Hello MQTT"
```

Expected output on Terminal 1:

```text
Hello MQTT
```

---

## Useful Commands

### View Running Containers

```bash
docker ps
```

### View All Containers

```bash
docker ps -a
```

### Restart Broker

```bash
docker restart mosquitto
```

### Stop Broker

```bash
docker stop mosquitto
```

### Start Broker

```bash
docker start mosquitto
```

### Remove Broker

```bash
docker stop mosquitto
docker rm mosquitto
```

### Remove Everything

```bash
docker stop mosquitto
docker rm mosquitto

sudo rm -rf ~/server/configs/mqtt
```

---

## Troubleshooting

### Restarting (13)

Check logs:

```bash
docker logs mosquitto
```

Common causes:

* Missing passwd file
* Missing log file
* Incorrect permissions
* Invalid mosquitto.conf

### Verify Password File Exists

```bash
ls -l mosquitto/config/passwd
```

### Verify Log File Exists

```bash
ls -l mosquitto/log/mosquitto.log
```

### Verify Configuration

```bash
cat mosquitto/config/mosquitto.conf
```

### Verify Mounted Files Inside Container

```bash
docker exec -it mosquitto ls -la /mosquitto/config
docker exec -it mosquitto ls -la /mosquitto/data
docker exec -it mosquitto ls -la /mosquitto/log
```

### Verify Container Status

```bash
docker ps -a
```


# Home Assistant Docker Installation

## Directory Structure

Home Assistant configuration is stored in:

```text
/server/configs/homeAssist
```

Create the directory if it does not exist:

```bash
mkdir -p /server/configs/homeAssist
```

---

## Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    restart: unless-stopped
    privileged: true
    network_mode: host
    environment:
      TZ: Asia/Kolkata
    volumes:
      - /server/configs/homeAssist:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
```

---

## Start Home Assistant

Navigate to the Home Assistant directory:

```bash
cd /server/configs/homeAssist
```

Start the container:

```bash
docker compose up -d
```

Verify the container is running:

```bash
docker ps
```

View logs:

```bash
docker logs -f homeassistant
```

---

## Access Home Assistant

Open:

```text
http://<SERVER_IP>:8123
```

Example:

```text
http://192.168.1.100:8123
```

Complete the initial Home Assistant setup wizard.

---

# Configure Reverse Proxy Support

If Home Assistant is accessed through a reverse proxy such as Cloudflare Tunnel, Nginx, Traefik, or Caddy, configure trusted proxies.

Edit:

```bash
nano /server/configs/homeAssist/configuration.yaml
```

Add:

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
```

For Docker-based reverse proxies, you may also need:

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - 172.17.0.0/16
    - 172.16.0.0/12
```

Save the file and restart Home Assistant:

```bash
docker restart homeassistant
```

Verify the configuration:

```bash
docker logs -f homeassistant
```

The following error should no longer appear:

```text
A request from a reverse proxy was received from 127.0.0.1,
but your HTTP integration is not set-up for reverse proxies
```

---

## Verify External Access

Test local access:

```text
http://<SERVER_IP>:8123
```

Test external access through your reverse proxy or Cloudflare Tunnel URL.

Once both local and external access are working, proceed with installing HACS and other integrations.
