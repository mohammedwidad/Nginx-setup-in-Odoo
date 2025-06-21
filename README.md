# Nginx-setup-in-Odoo
sudo apt update
sudo apt upgrade
sudo apt install nginx


#Find the Nginx Folder
cd /etc/nginx
ls -l

nginx.conf (main config)

sites-available/ – where you write your site config

sites-enabled/ – contains symlinks to active sites

conf.d/ – sometimes used for extra config

snippets/ – for reusable configs


sudo nano /etc/nginx/sites-available/odoo18

#View Your Nginx Site Configs
cd /etc/nginx/sites-available
ls -l
cd /etc/nginx/sites-enabled
ls -l

#Find/Create SSL Folder
cd /etc/ssl
ls -l
sudo mkdir -p /etc/ssl/nginx
cd /etc/ssl/nginx
This folder will contain:
certificate.pem → your SSL certificate
key.pem → your SSL key


#Configure Nginx for SSL (HTTPS)
sudo nano /etc/nginx/sites-available/odoo

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
    server_name www.familysportcenter.net;
    return 301 https://$server_name$request_uri;
}


# Main server
server {
    listen 443 ssl;
    server_name www.familysportcenter.net;

    ssl_certificate /etc/ssl/nginx/certificate.pem;
    ssl_certificate_key /etc/ssl/nginx/key.pem;

    access_log /var/log/nginx/odoo_access.log;
    error_log /var/log/nginx/odoo_error.log;

    location /longpolling {
        proxy_pass http://127.0.0.1:8072;
        proxy_connect_timeout 3600;
        proxy_read_timeout 3600;
        proxy_send_timeout 3600;
        send_timeout 3600;
    }

    location / {
        proxy_pass http://127.0.0.1:8016;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    gzip on;
    gzip_min_length 1000;
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
