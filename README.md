# wordpress-hosting

# Setting Up a WordPress Site with Nginx and SSL

This guide provides a step-by-step process for setting up a WordPress site on an Nginx server, including SSL configuration.

---

## Table of Contents

1. [Extract WordPress Files](#extract-wordpress-files)
2. [Set Permissions](#set-permissions)
3. [Configure Nginx](#configure-nginx)
4. [Enable the Nginx Site](#enable-the-nginx-site)
5. [Update WordPress Configuration](#update-wordpress-configuration)
6. [Reload Services](#reload-services)

---

## 1. Extract WordPress Files

Extract the WordPress files into your desired directory using the following command:

```bash
tar -xvzf latest.tar.gz -C /specific-path
```

---

## 2. Set Permissions

Adjust the ownership and permissions of the WordPress directory to ensure proper functionality:

```bash
sudo chown -R www-data:www-data /var/www/site_path
sudo chmod -R 755 /var/www/site_path
```

---

## 3. Configure Nginx

Edit the Nginx configuration file for your site:

```bash
sudo nano /etc/nginx/sites-available/site
```

Add the following configuration:

```nginx
server {
    server_name test.kepalabergetar.es;

    root /var/www/test.kepalabergetar.es;
    index index.php index.html index.htm;

    access_log /var/log/nginx/wordpress_access.log;
    error_log /var/log/nginx/wordpress_error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    listen 443 ssl; # Enable SSL for this block
    ssl_certificate /etc/letsencrypt/live/test.kepalabergetar.es/fullchain.pem; # SSL certificate path
    ssl_certificate_key /etc/letsencrypt/live/test.kepalabergetar.es/privkey.pem; # SSL private key path
    include /etc/letsencrypt/options-ssl-nginx.conf; # SSL config
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # DH params
}

# HTTP to HTTPS Redirect (port 80)
server {
    listen 80;
    server_name test.kepalabergetar.es;

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}
```

---

## 4. Enable the Nginx Site

Enable the site configuration by creating a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

Reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

---

## 5. Update WordPress Configuration

Edit the `wp-config.php` file in your WordPress installation directory to set up database credentials:

```bash
sudo nano /var/www/site_path/wp-config.php
```

Update the following lines with your database details:

```php
define('DB_NAME', 'your_database_name');
define('DB_USER', 'your_database_user');
define('DB_PASSWORD', 'your_database_password');
define('DB_HOST', 'localhost');
```

Save and exit.

---

## 6. Reload Services

After completing the configuration, reload the necessary services to ensure everything is working correctly:

```bash
sudo systemctl reload nginx
sudo systemctl restart php8.3-fpm
```

---

## Additional Notes

- Ensure your SSL certificates are correctly installed. You can use [Certbot](https://certbot.eff.org/) to generate free SSL certificates.
- If you encounter issues, check the Nginx logs:
  - Access Log: `/var/log/nginx/wordpress_access.log`
  - Error Log: `/var/log/nginx/wordpress_error.log`

---

## References

- [Nginx Documentation](https://nginx.org/en/docs/)
- [WordPress Codex](https://codex.wordpress.org/)
