# Installing Code-Server on Ubuntu Server

This guide outlines the steps to install `code-server` on an Ubuntu Server. `Code-server` allows you to run Visual Studio Code on a remote server, making it accessible via a web browser.

## Prerequisites

Ensure `curl` is installed on your Ubuntu Server. If not, follow these steps to install it:

### Install Curl on Ubuntu Server

1. **Access your server** - Connect to your Ubuntu Server via SSH or through your local terminal.

2. **Update your package list**:

    ```bash
    sudo apt update
    ```

3. **Install curl**:

    ```bash
    sudo apt install curl
    ```

## Installation Steps for Code-Server

With `curl` installed, proceed with installing `code-server`:

1. **Run the Installation Script**: In your terminal, execute the following command:

    ```bash
    curl -fsSL https://code-server.dev/install.sh | sh
    ```

    This command downloads and runs the installation script for `code-server`.

### Enabling Code-Server at Boot
- After the installation completes, execute the provided systemctl command to enable code-server to start automatically at boot:
    ```bash
    sudo systemctl enable --now code-server@$USER
    ```

### Configuration File
- The configuration file for `code-server` is located at `~/.config/code-server/config.yml`. Here is an example configuration:
    ```yaml
    bind-addr: 127.0.0.1:8080
    auth: password
    password: password
    cert: false
    ```

#### Restarting Code-Server
- After making changes to the configuration file, restart `code-server` to apply the changes:
    ```
    sudo systemctl restart code-server@$USER
    ```

## Installing NGINX
- Install NGINX to use as a reverse proxy for code-server:
    ```
    sudo apt install nginx
    ```

### Configuring NGINX for Code-Server
1. Create an NGINX configuration file for code-server:
    ```
    sudo nano /etc/nginx/sites-available/code-server
    ```
2. Add the following configuration to the file:
    ```nginx
    server {
        listen 80;
        server_name <your-domain.com>;

        location / {
            proxy_pass http://localhost:8080/;
            proxy_set_header Connection upgrade;
            proxy_set_header Host $host;
            proxy_set_header Accept-Encoding gzip;
            proxy_set_header X-Forwarded-Host $host;

            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $http_connection;
            proxy_http_version 1.1;
        }
    }
    ```
3. Enable the configuration by linking the file to the sites-enabled directory:
    ```
    sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
    ```
4. Test the NGINX configuration for syntax errors:
    ```
    sudo nginx -t
    ```
5. Reload NGINX to apply the changes:
    ```
    sudo systemctl reload nginx
    ```

## Setting Up SSL with Certbot
- Install NGINX and Certbot for automatic SSL configuration:
    ```bash
    sudo apt install -y certbot python3-certbot-nginx
    ```
- Configure SSL for your domain with Certbot:
    ```bash
    sudo certbot --non-interactive --redirect --agree-tos --nginx -d mydomain.com -m me@example.com
    ```
    **Note:** Replace `mydomain.com` with your actual domain name and `me@example.com` with your email address for certificate expiration alerts.
