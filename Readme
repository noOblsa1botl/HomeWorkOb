Установить и настроить Prometheus.
Результатом выполнения данного дз будет являться публичный репозиторий в системе контроля версий (Github, Gitlab, etc.) 
в котором будет находится Readme с описание выполненых действий. Файлы конфигурации prometheus и alertmanager должны находится в директории GAP-1.


Описание/Пошаговая инструкция выполнения домашнего задания:
Для выполнения данного домашнего задания вам потребуется выполнить следующие действия:

На виртуальной машине установите любую open source CMS которая включает в себя следующие компоненты: nginx, php-fpm, database (MySQL or Postgresql)

Install nginx, php-fpm, mysql:
```bash
sudo apt install -y nginx mysql-server php-fpm php-mysql php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```

Prepare MySQL for wordpress:
```bash
# ubuntu20.04 root issue
sudo mysql -u root -p

sudo mysql_secure_installation
sudo mysql
```

Create database, user, and grant priveleges:
```sql
CREATE DATABASE wp DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Prepare nginx config file:
```nginx
server {
  listen 443 ssl;

  include snippets/self-signed.conf;
  include snippets/ssl-params.conf;

  server_name 192.168.0.200;
  root /var/www/wp;

  index index.php;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
  }

  location = /favicon.ico { log_not_found off; access_log off; }
  location = /robots.txt { log_not_found off; access_log off; allow all; }
  location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
    expires max;
    log_not_found off;
  }


  location ~ /\.ht {
    deny all;
  }

}

server {
  listen 80;
  server_name 192.168.0.200;
  access_log off;
  return 301 https://$server_name$request_uri;
}
```

Download Wordpress and configure them:
```bah
curl -LO https://wordpress.org/latest.tar.gz
tar zxvf latest.tar.gz 
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
sudo cp -a /tmp/wordpress/. /var/www/wp
sudo useradd www-data
sudo usermod -aG www-data www-data
sudo chown -R www-data:www-data /var/www/wp/
curl -s https://api.wordpress.org/secret-key/1.1/salt/
sudo vim /var/www/wp/wp-config.php
```

Restart services:
```bash
sudo systemctl reload nginx
sudo systemctl restart php7.4-fpm
```


На этой же виртуальной машине установите Prometheus exporters для сбора метрик со всех компонентов системы (начиная с VM и заканчивая DB, не забудьте про blackbox exporter который будет проверять доступность вашей CMS)

Next, I download three exporters blackbox, node and mysql:
```bash
# create user for running services
sudo useradd -m exporters
sudo usermod -aG exporters exporters

# download the exporters
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.14.0/mysqld_exporter-0.14.0.linux-amd64.tar.gz
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.22.0/blackbox_exporter-0.22.0.linux-amd64.tar.gz

# unzip them
tar zxf *.tar.gz

# create env variable with mysql password
MYSQLD_EXPORTER_PASSWORD=password
```

Create service user for mysql and grant previliges:

```mysql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'password' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON \*.\* TO 'exporter'@'localhost';
```

Configs for blackbox and mysql [there](GAP-1/). Then create a services for each exporter and run them

На этой же или дополнительной виртуальной машине установите Prometheus задачей которого будет раз в 5 секунд собирать метрики с экспортеров

Install Prometheus:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.40.2/prometheus-2.40.2.linux-amd64.tar.gz
tar zxf prometheus-2.40.2.linux-amd64.tar.gz 
cd prometheus-2.40.2.linux-amd64/
cp prometheus /usr/local/bin/
sudo cp prometheus /usr/local/bin/
```
Then I use this [config](GAP-1/prometheus.yml) and create service for running in background

На этой же или дополнительной виртуальной машине установите Alertmanager и сконфигурируйте его таким образом чтобы в случае недоступности какого либо компонента был отправлен alert с важность Critical в один из канал оповещений (канал оповещений на выбор: slack or telegram)

Install Alertmanager and [configure](GAP-1/alertmanager.yml) them:
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
tar zxf alertmanager-0.24.0.linux-amd64.tar.gz

sudo useradd alertmanager
sudo cp alertmanager /usr/local/bin/
sudo chown alertmanager: /usr/local/bin/alertmanage
```
Then creating service for background job

---

In the second task I need to implement Grafana monitoring.

For this task I use Docker to run Grafana:
```bash
docker run -d -p 3000:3000 --name grafana grafana/grafana-oss
```

Next, I go to Grafana Labs to get dashboards for my exporters. All screenshots [there](GAP-2/)
