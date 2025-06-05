# Securing a Website with Docker, Nginx, and Certbot

This repository provides a simple example of a web server secured with **HTTPS using Nginx**, **Let's Encrypt certificates (via Certbot)**, and **Docker Compose**.  
It also includes recommendations to **secure the underlying Linux host**, including SSH hardening, brute-force protection with `fail2ban`, and port scan detection with `portsentry`.

---

## Web Security with HTTPS

This project includes:
- **Nginx** as a reverse proxy
- **Certbot** for free SSL certificates
- **Docker Compose** for orchestration

### Project structure

```

.
├── docker-compose.yml
├── nginx/
    └── nginx.conf

```

### Quick Start

1. **Change the settings**

    In docker-compose.yml, change the domain name and the email in the certbot command.

    ```bash
    certonly --webroot --webroot-path=/var/www/certbot
        --email your_email@gmail.com --agree-tos --no-eff-email
        -d your_domain_name.com -d www.your_domain_name.com
    ```

2. **Launch the containers** (make sure ports 80 and 443 are available):

    ```bash
    docker-compose up -d
    ```

3. **Obtain an SSL certificate using Certbot**:

    Check the directory certbot, you should have www and conf.

4. **Change the nginx.conf**:

    You should uncomment some lines to enable https.

    You could comment the certbot service in docker-compose.yml (until the next certificate generation)

5. **Restart your nginx**

    ```bash
    docker-compose restart nginx 
    ```

---

## Securing the Host System

Some advices to avoid surprises on your server.

### SSH Hardening

* **Use key-based authentication only**:

  ```bash
  sudo nano /etc/ssh/sshd_config
  # Make sure the following lines are set:
  PasswordAuthentication no
  PermitRootLogin no
  ```

    Avoid login with root, create another user for ssh.

* **Optional: change the default SSH port**:

  ```bash
  Port 2222
  ```

* **Restart SSH**:

  ```bash
  sudo systemctl restart ssh
  ```

---

### Install and Configure `fail2ban`

Blocks IPs that show malicious signs (e.g., too many password failures).

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
```

Sample configuration in `/etc/fail2ban/jail.local`:

```ini
[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
maxretry = 3
```

sometimes we should use the backend `systemd` on debian.

```ini
[sshd]
enabled = true
backend = systemd
```

This should avoid brute force access to your ssh.

Check the status of fail2ban from time to time :

```bash
fail2ban-client status sshd
```
You will have a report as :

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     674
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 4
   |- Total banned:     63
   `- Banned IP list:   188.166.217.79 165.154.201.122 46.191.141.152 34.91.0.68
```

---

### Install `portsentry` for Port Scan Detection

```bash
sudo apt install portsentry
sudo systemctl enable --now portsentry
```

Edit `/etc/default/portsentry`:

```bash
TCP_MODE="atcp"
UDP_MODE="audp"
```

Then review `/etc/portsentry/portsentry.conf` for further tuning.

---

## 📌 Additional Recommendations

* Use a firewall like `ufw` or `iptables`:

  ```bash
  sudo ufw allow 80,443/tcp
  sudo ufw allow 2222/tcp
  sudo ufw enable
  ```

---

## 🔗 References

* [Let's Encrypt](https://letsencrypt.org/)
* [Certbot](https://certbot.eff.org/)
* [Nginx Docker Image](https://hub.docker.com/_/nginx)
* [Fail2Ban Documentation](https://www.fail2ban.org/)
* [Portsentry](https://portsentry.xyz)
