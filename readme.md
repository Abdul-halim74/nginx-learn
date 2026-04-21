## Part 1: Basic Setup
1. Nginx & OpenSSL install
sudo apt update
sudo apt install nginx openssl -y

2. Directory create
sudo mkdir -p /var/www/secure-app

3. HTML file create
sudo nano /var/www/secure-app/index.html

## Part 2: SSL Setup
1. SSL folder create
sudo mkdir -p /etc/nginx/ssl

2. Self-signed SSL generate (365 days)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/nginx/ssl/secure.key \
-out /etc/nginx/ssl/secure.crt

## Part 3: Nginx Configuration
1. New config file
sudo nano /etc/nginx/sites-available/secure-app

2. Full Config Code
# HTTP → HTTPS redirect
server {
    listen 80;
    server_name localhost;

    return 301 https://$host$request_uri;
}

# HTTPS server
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/ssl/secure.crt;
    ssl_certificate_key /etc/nginx/ssl/secure.key;

    root /var/www/secure-app;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Reverse Proxy
    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

3. Enable config
sudo ln -s /etc/nginx/sites-available/secure-app /etc/nginx/sites-enabled/

4. Default config disable
sudo rm /etc/nginx/sites-enabled/default

#Part 4: Reverse Proxy
1. Backend server run (Node.js example)
nano server.js

const http = require('http');

http.createServer((req, res) => {
    res.write("Backend running on port 3000");
    res.end();
}).listen(3000);

console.log("Server running on port 3000");

Run backend
node server.js

Part 5: Testing
1. Test config
sudo nginx -t

2. Reload Nginx
sudo systemctl reload nginx

3. Browser Testing
http://localhost

