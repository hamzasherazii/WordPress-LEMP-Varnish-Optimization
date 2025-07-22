# 🚀 WordPress Performance Optimization Guide (LEMP + Varnish)

This guide walks you through implementing **Varnish caching** and optimizing a **WordPress site running on LEMP stack (Linux, NGINX, MySQL, PHP)** hosted on an **EC2 instance**. It is tailored for beginners and ideal for speeding up your site with efficient caching and performance enhancements. You can perform this on any server, EC2 isn't the limitation!

The domain and IP provided are my experimental environments and yes, they're live.

---

## 🧱 Stack Overview

* **Web Server**: NGINX
* **App**: WordPress (PHP-FPM)
* **Cache Layer**: Varnish (on port 80)
* **OS**: Ubuntu (EC2)

---

## 🔧 Step 1: Reconfigure NGINX to Port 8080

### File to edit:

```
/etc/nginx/sites-available/wordpress
```

### Update the `listen` directive:

```nginx
server {
    listen 8080;
    server_name lemp.hamzasherazi.dev 51.21.152.11;
    root /var/www/html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, max-age=2592000, immutable";
        access_log off;
    }

    # other configuration (PHP handling, etc.)
}
```

### Reload NGINX:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🚀 Step 2: Install and Configure Varnish

### Install Varnish:

```bash
sudo apt update
sudo apt install varnish
```

### Configure Varnish Backend:

```bash
sudo nano /etc/varnish/default.vcl
```

Replace with:

```vcl
vcl 4.0;
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
```

### Configure Varnish to Listen on Port 80:

```bash
sudo nano /etc/systemd/system/multi-user.target.wants/varnish.service
```

Edit the `ExecStart` line:

```ini
ExecStart=/usr/sbin/varnishd \
 -a :80 \
 -T localhost:6082 \
 -f /etc/varnish/default.vcl \
 -s malloc,256m
```

### Reload and Restart:

```bash
sudo systemctl daemon-reexec
sudo systemctl restart varnish
```

---

## ✅ Step 3: Verify Setup

### Check Varnish Headers:

```bash
curl -I http://localhost
```

Look for headers like:

```
X-Varnish: 123456
Via: 1.1 varnish
Age: 0
```

### Check Cache-Control in Browser:

Open Chrome DevTools → Network → Click a `.css` or `.js` file → Look for:

```
cache-control: public, max-age=2592000, immutable
```

---

## 🤉 Step 4: WordPress Performance Plugins (All Free)

Install the following **free** plugins:

| Plugin              | Purpose                                           |
| ------------------- | ------------------------------------------------- |
| **LiteSpeed Cache** | Full-page caching, CSS/JS minification, lazy load |
| **Autoptimize**     | JS/CSS/HTML optimization                          |
| **Query Monitor**   | Find slow queries/plugins                         |

These plugins can be installed directly from the WordPress plugin repository.

---

## 🔄 Optional Enhancements

* ✅ Use **Redis or Memcached** for object caching (both are free)
* ✅ Serve static files via **free CDN** (e.g., Cloudflare Free Plan)
* ✅ Audit slow plugins and database queries using Query Monitor

---

## 🧪 Test & Benchmark

You can test performance gains using:

* [GTmetrix](https://gtmetrix.com)
* [PageSpeed Insights](https://pagespeed.web.dev/)
* `curl -I` to view caching headers

---

## 🎉 Done!

You’ve successfully configured:

* Varnish as a reverse proxy cache
* Long-term static file caching
* WordPress performance plugins — all **100% free**

Your site should now load significantly faster 🚀

---

> 📘️ **Author:** \[Hamza Sherazi]
> 🕒 **Last updated:** July 2025
