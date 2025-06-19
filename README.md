# Nginx-setup-in-Odoo
sudo apt update
sudo apt upgrade
sudo apt install nginx

sudo nano /etc/nginx/sites-available/odoo18

# Odoo Server
upstream odoo {
    server 127.0.0.1:8069;
}

upstream odoochat {
    server 127.0.0.1:8072;
}

# HTTP -> HTTPS
server {
    listen 80;
    server_name familysportcenter.net www.familysportcenter.net;
    return 301 https://$server_name$request_uri;
}

# Main server
server {
    listen 443 ssl http2;
    server_name familysportcenter.net www.familysportcenter.net;

    # SSL parameters
    ssl_certificate /etc/letsencrypt/live/familysportcenter.net/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/familysportcenter.net/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    # Add headers
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

    # Proxy settings
    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    # Log files
    access_log /var/log/nginx/odoo.access.log;
    error_log /var/log/nginx/odoo.error.log;

    # Handle longpoll requests
    location /longpolling {
        proxy_pass http://odoochat;
    }

    # Handle main requests
    location / {
        proxy_redirect off;
        proxy_pass http://odoo;
    }

    # Cache static files
    location ~* /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://odoo;
    }

    # Gzip
    gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
    gzip on;
}




sudo ln -s /etc/nginx/sites-available/odoo18 /etc/nginx/sites-enabled/odoo18


sudo nginx -t
sudo service nginx restart



### Important notes:

1. **SSL Certificates**: You need to obtain SSL certificates from Let's Encrypt (or another provider). You can use Certbot:
   ```
   sudo certbot certonly --nginx -d familysportcenter.net -d www.familysportcenter.net
   ```

2. **DH Parameters**: Generate strong DH parameters:
   ```
   sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
   ```

3. **Odoo Configuration**: Make sure your Odoo config file (`sudo nano /etc/odoo.conf`) has:
   ```
   proxy_mode = True
   ```

4. **Restart Services** after making changes:
   ```
   sudo systemctl restart nginx
   sudo systemctl restart odoo
   ```

5. **DNS Settings**: Ensure your domain's DNS A records point to your server's IP address.

This configuration includes HTTP to HTTPS redirection, proper SSL settings, static file caching, and optimizations for Odoo v18.




sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d familysportcenter.net -d www.familysportcenter.net
