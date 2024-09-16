# NGINX Load Balancer Setup with Three Machines

This guide outlines how to set up a load balancer using NGINX across three machines. One machine will serve as the proxy server responsible for load balancing, while the other two will act as backend servers, all running on RHEL with NGINX.

## Architecture Overview

- **Proxy Server (Load Balancer):** Handles incoming requests and distributes them between the backend servers.
- **Backend Server 1:** A server that processes requests from the load balancer.
- **Backend Server 2:** Another server that processes requests from the load balancer.

### IP Addresses

- **Proxy Server (Load Balancer):** `192.168.1.222`
- **Backend Server 1:** `192.168.1.225`
- **Backend Server 2:** `192.168.1.133`

## Prerequisites

- **Operating System:** All machines are running RHEL (Red Hat Enterprise Linux).
- **NGINX Installed:** NGINX is installed on the proxy server and both backend servers.
- **Firewall:** Ensure ports (especially 8080) are open on all servers.

## Step 1: Install NGINX on All Servers

### On the Proxy Server (192.168.1.222):

1. **Install NGINX:**

    ```bash
    sudo yum install nginx -y
    ```

2. **Start and enable NGINX:**

    ```bash
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

### On the Backend Servers (192.168.1.225 and 192.168.1.133):

1. **Install NGINX:**

    ```bash
    sudo yum install nginx -y
    ```

2. **Start and enable NGINX:**

    ```bash
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

## Step 2: Configure NGINX as a Load Balancer on the Proxy Server

1. **Open the NGINX configuration file on the proxy server:**

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

2. **Configure the `nginx.conf` file as follows:**

    ```nginx
    user nginx;
    worker_processes auto;

    events {
        worker_connections 1024;
    }

    http {
        upstream backend {
            server 192.168.1.225:8080;
            server 192.168.1.133:8080;
            keepalive 64;
        }

        server {
            listen 8080;

            location / {
                proxy_pass http://backend;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
    }
    ```

3. **Test the configuration:**

    ```bash
    sudo nginx -t
    ```

4. **Restart NGINX to apply the changes:**

    ```bash
    sudo systemctl restart nginx
    ```

## Step 3: Configure NGINX on Backend Servers

Ensure that the backend servers (`192.168.1.225` and `192.168.1.133`) are properly configured to serve content via NGINX.

1. **Create or modify the default NGINX configuration:**

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

    Example of a simple NGINX server block to serve a static HTML page:

    ```nginx
    server {
        listen 8080;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
    ```

2. **Ensure that the `/usr/share/nginx/html/index.html` file exists and contains your web content.** For testing, you can create a simple HTML file:

    ```bash
    echo "<h1>Hello from Backend Server 1</h1>" | sudo tee /usr/share/nginx/html/index.html
    ```

    Repeat this step on `192.168.1.133` with different content to differentiate between the two servers.

3. **Test NGINX locally on each backend server:**

    ```bash
    curl http://localhost:8080
    ```

    You should see the HTML content served by each backend server.

4. **Restart NGINX:**

    ```bash
    sudo systemctl restart nginx
    ```

## Step 4: Open Port 8080 on All Servers

### On All Servers (Proxy and Backend Servers):

1. **Allow traffic on port 8080:**

    ```bash
    sudo firewall-cmd --permanent --add-port=8080/tcp
    sudo firewall-cmd --reload
    ```

## Step 5: Test the Load Balancer

1. **Access the proxy server’s IP address on port 8080:**

    ```plaintext
    http://192.168.1.222:8080
    ```

2. **Verify Load Balancing:**

    - Refresh the page multiple times.
    - You should see responses from both backend servers, alternating based on the load balancing algorithm used by NGINX.

## Additional Notes

- **Load Balancing Algorithm:** By default, NGINX uses the round-robin method to distribute requests. You can configure other algorithms like `least_conn` or `ip_hash` if needed.
- **Firewall Rules:** Ensure that firewalls are configured to allow traffic on port `8080` for all involved machines.
- **Monitoring and Logging:** Regularly monitor NGINX logs (`/var/log/nginx/access.log` and `/var/log/nginx/error.log`) to ensure everything is functioning correctly.

## Conclusion

This setup provides a basic load balancing solution using NGINX on a proxy server to distribute requests across two backend servers running on RHEL. You can scale this by adding more backend servers or adjusting the NGINX configuration to suit your needs.

If want the same with Docker option see the Folder Docker
