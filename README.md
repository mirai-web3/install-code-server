---

## Tutorial for Installing **code-server v4.96.4** on Ubuntu 20.04 to 24.04

---

### **Prerequisites:**
- Ubuntu Minimum 20.04
- A **non-root** user with **sudo** privileges
- A firewall configured to allow access on **port 8080** (or your chosen port)

---

## Step 1: Update Your System
Ensure your system is up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2: Install Dependencies
Install the required dependencies:
```bash
sudo apt install -y curl wget gnupg
```

---

## Step 3: Download and Install `code-server v4.96.4`
1. **Download the specific version (4.96.4)**:
   - Visit the [code-server GitHub releases page](https://github.com/coder/code-server/releases) to find the download link for version 4.96.4.
   - Alternatively, use the following command to download it directly:
     ```bash
     wget https://github.com/coder/code-server/releases/download/v4.96.4/code-server_4.96.4_amd64.deb
     ```

2. **Install the downloaded `.deb` package**:
   ```bash
   sudo dpkg -i code-server_4.96.4_amd64.deb
   ```

3. **Resolve any missing dependencies**:
   If the installation reports missing dependencies, run:
   ```bash
   sudo apt --fix-broken install
   ```

---

## Step 4: Start and Enable `code-server`
1. **Start the `code-server` service**:
   ```bash
   sudo systemctl start code-server@$USER
   ```

2. **Enable `code-server` to start on boot**:
   ```bash
   sudo systemctl enable code-server@$USER
   ```

3. **Check the status** of the service:
   ```bash
   sudo systemctl status code-server@$USER
   ```

   You should see an output indicating that the service is running:
   ```
   Active: active (running) since Mon 2024-11-11 22:42:49 UTC; 1min 3s ago
   ```

---

## Step 5: Configure `code-server` for HTTP (No SSL)
1. **Edit the `code-server` configuration file**:
   ```bash
   nano ~/.config/code-server/config.yaml
   ```

2. **Modify the configuration** to disable SSL and bind to the desired address:
   ```yaml
   bind-addr: 0.0.0.0:8080  # Listen on all interfaces and port 8080
   auth: password           # Enable password authentication
   password: your-password  # Set your password here
   cert: false              # Disable SSL
   ```

3. **Save the file** and exit the editor (`Ctrl + X`, then `Y` to confirm, and `Enter`).

4. **Restart the `code-server` service**:
   ```bash
   sudo systemctl restart code-server@$USER
   ```

---

## Step 6: Set Up Nginx (Optional Reverse Proxy)
If you want to use Nginx as a reverse proxy to forward requests from port 80 (HTTP) to port 8080, follow these steps:

1. **Install Nginx**:
   ```bash
   sudo apt install nginx
   ```

2. **Create an Nginx configuration file**:
   ```bash
   sudo nano /etc/nginx/sites-available/code-server
   ```

3. **Add the following configuration**:
   ```nginx
   server {
       listen 80;
       server_name your-ip;  # Replace with your server's IP or domain

       location / {
           proxy_pass http://127.0.0.1:8080;  # Forward traffic to code-server
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;

           # WebSocket support (important for code-server)
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
       }
   }
   ```

4. **Enable the site configuration**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
   ```

5. **Test the Nginx configuration**:
   ```bash
   sudo nginx -t
   ```

6. **Reload Nginx**:
   ```bash
   sudo systemctl reload nginx
   ```

---

## Step 7: Open Firewall Ports
Allow traffic on ports 80 (HTTP) and 8080 (code-server):

1. **For UFW**:
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 8080/tcp
   sudo ufw enable
   ```

---

## Step 8: Access `code-server`
- **Direct Access**:
  ```
  http://your-ip:8080
  ```
- **Via Nginx** (if configured):
  ```
  http://your-ip
  ```

Enter the **password** you set in the `config.yaml` file to access the `code-server` interface.

---

### **Notes:**
- **Security Warning**: Running `code-server` over HTTP is not secure. For production environments, always use HTTPS with a valid SSL certificate.
- **Version-Specific**: This tutorial is tailored for **code-server v4.96.4**. If you need a different version, adjust the download link accordingly.
- **Password Protection**: Ensure you set a strong password in the `config.yaml` file to prevent unauthorized access.

---
