# GAP-1
Первое домашнее задание otus. 
### Описание ДЗ:
1. На виртуальной машине установите любую open source CMS, которая включает в себя следующие компоненты: nginx, php-fpm, database (MySQL or Postgresql)
2. На этой же виртуальной машине установите Prometheus exporters для сбора метрик со всех компонентов системы (начиная с VM и заканчивая DB, не забудьте про blackbox exporter, который будет проверять доступность вашей CMS)
3. На этой же или дополнительной виртуальной машине установите Prometheus, задачей которого будет раз в 5 секунд собирать метрики с экспортеров.
4. Задание со звездочкой (повышенная сложность)
+ на VM с установленной CMS слишком много “портов экспортеров торчит наружу” и они доступны всем, попробуйте настроить доступ только по одному и добавить авторизацию.
+ Если вы выполнили задание со звездочкой номер 1, то - добавить SSL =)
## Решение. 
### Установка node-exporter. На ВМ с CMS.
- sudo apt install prometheus-node-exporter
- systemctl status prometheus-node-exporter
- curl -v http://localhost:9100
### Установка mysql-exporter. На ВМ с CMS.
- sudo apt install prometheus-mysql-exporter
- sudo vi /var/lib/prometheus/.my.cnf
```
[client]
user=пользователь для доступа к базе
password=пароль для доступа к базе
```
- sudo systemctl status prometheus-mysqld-exporter
- curl -v http://localhost:9104
### Установка php-fpm_exporter. На ВМ с CMS.
- настройка php-fpm
  - в /etc/php/8.1/fpm/pool.d/www.conf раскомментировать строки
```
pm.status_listen = 127.0.0.1:9001
pm.status_path = /status
```
  - sudo systemctl restart php8.1-fpm
  - systemctl status php8.1-fpm
  - curl -v http;//localhost:9001
- установка php-fpm_exporter
  - wget https://github.com/hipages/php-fpm_exporter/releases/download/v2.2.0/php-fpm_exporter_2.2.0_linux_amd64
  - sudo mv ./php-fpm_exporter_2.2.0_linux_amd64 /usr/bin/php-fpm_exporter
  - sudo chmod +x /usr/bin/php-fpm_exporter
- проверка работы php-fpm_exporter и доступности php-fpm
  - php-fpm_exporter get --phpfpm.scrape-uri tcp://127.0.0.1:9001/status
- запуск как службы
  - sudo vi /etc/systemd/system/php-fpm-exporter.service
```
[Unit]
Description=PHP-FPM Exporter
After=network.target

[Service]
ExecStart=/usr/bin/php-fpm_exporter server --phpfpm.scrape-uri tcp://127.0.0.1:9001/status
Restart=always
User=www-data
Group=www-data

[Install]
WantedBy=multi-user.target
```
 - sudo systemctl daemon-reload
 - sudo systemctl start php-fpm-exporter
 - sudo systemctl enable php-fpm-exporter
 - sudo systemctl status php-fpm-exporter
 - curl -v http://localhost:9253
### Установка blackbox-exporter, prometheus и grafana.
- Blackbox-exporter, prometheus и grafana разворачивается в docker-compose. Файлы настроек в директории config.
