# Балансировка нагрузки Nginx → Apache (Debian 12 + CentOS 7)

## 1. Постановка задачи

Две виртуальные машины VirtualBox находятся в одной сети.

| ВМ | ОС | Пример IP | Apache | Nginx |
|---|---|---|---|---|
| VM1 | Debian 12 | `192.168.1.74` | `8080` | `80` |
| VM2 | CentOS 7 | `192.168.1.70` | `8080` | `80` |

На обеих ВМ Nginx принимает HTTP-запросы на порту `80` и распределяет их между двумя Apache на порту `8080`.

**Важно:** IP-адреса в примерах необходимо заменить на актуальные адреса ВМ. CentOS 7 снят с поддержки, поэтому установка пакетов может потребовать настройки архивных репозиториев.


<details>
<summary>Дополнительно: восстановление репозиториев CentOS 7 (EOL)</summary>

### Почему возникает проблема

CentOS 7 снят с поддержки (EOL). Его стандартные репозитории больше не обслуживаются на обычных зеркалах и перенесены в CentOS Vault.

### 1. Переключить базовые репозитории на архив

Создать резервную копию:

```bash
sudo cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

Заменить содержимое файла:

```bash
sudo tee /etc/yum.repos.d/CentOS-Base.repo > /dev/null <<'EOF'
[base]
name=CentOS-7.9.2009 - Base
baseurl=https://vault.centos.org/7.9.2009/os/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-7.9.2009 - Updates
baseurl=https://vault.centos.org/7.9.2009/updates/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-7.9.2009 - Extras
baseurl=https://vault.centos.org/7.9.2009/extras/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
EOF
```

Обновить кэш:

```bash
sudo yum clean all
sudo yum makecache
```

Теперь проверить установку:

```bash
sudo yum install -y httpd curl
```

### 2. Nginx

Nginx может потребовать отдельного репозитория, например EPEL или официального репозитория Nginx.
Тогда по инструкции с сайта nginx (https://nginx.org/en/linux_packages.html) добавим репозиторий для него:

```bash
sudo tee /etc/yum.repos.d/nginx.repo >> /dev/null <<'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=https://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF
```

</details>




## 2. Узнать IP-адреса

На каждой ВМ:

```bash
hostname -I
ip -4 addr
```

Записать IPv4-адреса Debian и CentOS. Проверить, что обе ВМ находятся в одной сети и могут обмениваться трафиком.

<details>
<summary>Дополнительно: Как проверить соединение между двумя ВМ</summary>

```bash
ping -c 4 IP_ADDRESS_OF_ANOTHER_VM
```
IP_ADDRESS_OF_ANOTHER_VM - адрес другой машины. Если ping не проходит (`100% packet loss` или `Destination Host Unreachable`), стоит поверить что:
- Обе ВМ подключены к одной сети VirtualBox.
- У обеих ВМ правильные IPv4-адреса и маски подсети.
- Сетевые интерфейсы включены.
- Межсетевой экран не блокирует ICMP-пакеты.

Про адреса:
```bash
ip route
```
выдаёт что-то вроде:
```bash
default via 192.168.1.254 dev enp0s3
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.74
```

Здесь важно:
- default via 192.168.1.254 — адрес шлюза.
- 192.168.1.0/24 — локальная подсеть (/24 — длина префикса сети, соответствующая маске 255.255.255.0).
- src 192.168.1.74 — IP-адрес этой ВМ.

Нужно проверить, что у обеих ВМ:
1. Есть IPv4-адрес.
2. Маска подсети соответствует нужной (настроенной в Virtualbox) сети.
3. IP-адреса относятся к одной подсети.
4. Нет конфликта IP-адресов.

На CentOS можно проверить состояние firewalld:
```bash
sudo systemctl status firewalld
```

На Debian аналог firewalld обычно не используется по умолчанию, но могут пригодиться проверки:
```bash
sudo ufw status
sudo systemctl status nftables
sudo nft list ruleset
```

UFW является инструментом управления правилами firewall, а nftables — механизмом фильтрации сетевого трафика в Linux. Отсутствие активного сервиса nftables само по себе не доказывает, что firewall отключён: правила могут управляться другим инструментом.

</details>

## 3. Debian 12: установка Apache и Nginx

```bash
sudo apt update
sudo apt install -y apache2 nginx curl
```

### 3.1. Настроить Apache на порту 8080

**1. Проверить текущую настройку порта:**

```bash
grep -n '^[[:space:]]*Listen' /etc/apache2/ports.conf
```

Заменить `Listen 80` на:

```bash
Listen 8080
```


**2. Настроить VirtualHost на порту 8080:**

Открыть конфигурацию сайта:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Убедиться, что блок VirtualHost использует порт `8080`:

```apache
<VirtualHost *:8080>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**3. Создать тестовую HTML-страницу:**

```bash
echo '<h1>Это Apache на Debian</h1>' | sudo tee /var/www/html/index.html
```

**4. Проверить конфигурацию Apache:**

```bash
sudo apache2ctl configtest
```

Ожидаемый результат:

```text
Syntax OK
```

**5. Включить Apache при загрузке системы и запустить его:**

```bash
sudo systemctl enable --now apache2
```

Если Apache уже запущен, а конфигурация была изменена, применить изменения:

```bash
sudo systemctl restart apache2
```

**6. Проверить состояние Apache:**

```bash
sudo systemctl status apache2
```

Ожидаемый результат: `active (running)`.


**7. Проверить, что Apache слушает порт 8080:**

```bash
sudo ss -lntp | grep ':8080'
```

**8. Проверить доступность Apache на порту 8080:**

```bash
curl http://127.0.0.1:8080/

```

Ожидаемый результат:

```html
<h1>Это Apache на Debian</h1>
```

**Важно:** Apache должен слушать порт `8080`, поскольку порт `80` предназначен для Nginx, который будет выступать в роли frontend/reverse proxy.

### 3.2. Настроить Nginx как балансировщик

Заменить `DEBIAN_IP` и `CENTOS_IP` на реальные IP-адреса ВМ:

```bash
sudo cp -a /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup
sudo tee /etc/nginx/sites-available/default > /dev/null <<'EOF'
upstream apache_backends {
    server CENTOS_IP:8080;
    server DEBIAN_IP:8080;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    location / {
        proxy_pass http://apache_backends;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF
```

Например:

```nginx
server 192.168.1.70:8080;
server 192.168.1.74:8080;
```

Проверить и перезапустить конфигурацию:

```bash
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
```

## 4. CentOS 7: установка Apache и Nginx

```bash
sudo yum install -y httpd nginx curl
```

Если пакет `nginx` не найден, необходимо подключить подходящий репозиторий Nginx.

### 4.1. Настроить Apache на порту 8080

### 4.1. Настроить Apache на порту 8080

**1. Проверить текущую настройку портов Apache:**

```bash
sudo grep -RniE '^[[:space:]]*Listen|<VirtualHost' /etc/httpd/conf /etc/httpd/conf.d/
```

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Заменить:

```apache
Listen 80
```

на:

```apache
Listen 8080
```

Если в файле присутствует блок:

```apache
<VirtualHost *:80>
```

изменить его на:

```apache
<VirtualHost *:8080>
```

Если такого блока нет, создавать его необязательно: Apache может обслуживать запросы через стандартную конфигурацию без явно заданного VirtualHost.

**Важно:** не должно оставаться других активных директив `Listen 80`, конфликтующих с новой конфигурацией. Также не добавляйте повторный `Listen 8080`, если он уже определён в другом загружаемом файле.


**2. Создать html страницу:**


```bash
echo '<h1>Это Apache на CentOS</h1>' | sudo tee /var/www/html/index.html
```

**3. Проверить конфигурацию Apache:**

```bash
sudo apachectl configtest
```

Ожидаемый результат:

```text
Syntax OK
```

**4. Перезапустить Apache:**

```bash
sudo systemctl restart httpd
```

**5. Проверить состояние Apache:**

```bash
sudo systemctl status httpd
```

Ожидаемый результат:

```text
active (running)
```

**6. Проверить, что Apache слушает порт 8080:**

```bash
sudo ss -lntp | grep ':8080'
```

**7. Проверить доступность Apache:**

```bash
curl http://127.0.0.1:8080/
```

**Итог:** Apache на CentOS 7 принимает HTTP-запросы на порту `8080`. Порт `80` остаётся для Nginx, который будет выступать в роли frontend/reverse proxy.

## 5. CentOS 7: SELinux и firewall

### 5.1. SELinux

Проверить состояние:

```bash
getenforce
```

Если SELinux включён (`Enforcing`), разрешить Apache использовать порт `8080`:

```bash
sudo yum install -y policycoreutils-python
sudo semanage port -a -t http_port_t -p tcp 8080
```

Если команда сообщает, что порт уже зарегистрирован, проверить его:

```bash
sudo semanage port -l | grep http_port_t
```

Если порт `8080` уже принадлежит другому SELinux-типу, не менять его вслепую: сначала проверить существующую политику.

```bash
[centos7user@centos7 ~]$ sudo semanage port -l | grep 8080
http_cache_port_t              tcp      8080, 8118, 8123, 10001-1001
```

тогда переназначим SELinux метку нашему порту на http_port_t (старую удалим, чтобы не допускать неясностей и конфликтов):
```bash
sudo semanage port -m -t http_port_t -p tcp 8080
sudo semanage port -d -t http_cache_port_t -p tcp 8080
```

Разрешить Nginx подключаться к backend-серверам:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

### 5.2. Firewall

Если используется `firewalld`:

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

Проверить правила:

```bash
sudo firewall-cmd --query-service=http
sudo firewall-cmd --query-port=8080/tcp
```

Обе команды должны вывести `yes`.

Порт `80` нужен для доступа к Nginx, а `8080` — для связи Nginx с Apache на обеих ВМ. В реальной сети доступ к `8080` желательно ограничить IP-адресами двух ВМ.

## 6. CentOS 7: настройка Nginx-балансировщика

Сделать резервные копии конфигураций:

```bash
sudo cp -a /etc/nginx/conf.d/apache.conf /etc/nginx/conf.d/apache.conf.backup
sudo cp -a /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.backup
```

Отключить стандартный сайт Nginx, чтобы он не конкурировал с балансировщиком:

```bash
sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

Создать конфигурацию:

```bash
sudo tee /etc/nginx/conf.d/apache.conf > /dev/null <<'EOF'
upstream apache_backends {
    server DEBIAN_IP:8080;
    server CENTOS_IP:8080;
}

server {
    listen 80 default_server;
    server_name _;

    location / {
        proxy_pass http://apache_backends;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF
```

Заменить `DEBIAN_IP` и `CENTOS_IP` на реальные IP-адреса.

Проверить и применить настройки:

```bash
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
sudo systemctl status nginx
```

Ожидаемый результат `nginx -t`:

```text
syntax is ok
test is successful
```

## 7. Проверка связи между ВМ

С Debian проверить оба Apache:

```bash
curl -sS --connect-timeout 5 http://DEBIAN_IP:8080/
curl -sS --connect-timeout 5 http://CENTOS_IP:8080/
```

С CentOS выполнить те же проверки:

```bash
curl -sS --connect-timeout 5 http://DEBIAN_IP:8080/
curl -sS --connect-timeout 5 http://CENTOS_IP:8080/
```

Ожидается, что первый запрос покажет страницу Debian, а второй — страницу CentOS. Если запрос не проходит, проверить IP-адреса, маршрутизацию, firewall и SELinux.

## 8. Проверка балансировки через оба Nginx

На Debian:

```bash
for i in {1..10}; do
    curl -sS http://127.0.0.1/
    echo
done
```

На CentOS:

```bash
for i in {1..10}; do
    curl -sS http://127.0.0.1/
    echo
done
```

В обоих случаях должны появляться обе страницы:

```text
Это Apache на Debian
Это Apache на CentOS
```

Балансировщик по умолчанию использует round-robin, то есть распределяет новые запросы между доступными серверами по очереди. При этом фактическое распределение может зависеть от состояния backend-серверов и характера запросов.

## 9. Проверка из браузера

На хостовой машине открыть:

- `http://DEBIAN_IP/` — Nginx на Debian.
- `http://CENTOS_IP/` — Nginx на CentOS.

На каждой странице должны появляться ответы обоих Apache при обновлении страницы.

<details>
<summary>Дополнительно: кириллица на странице Debian</summary>
Чтобы починить кракозябры на Debian можно добавить в html-заголовок UTF-8 charset по умолчанию:

```bash
debian12user@debian12:~$ sudo cat /etc/apache2/conf-available/charset.conf
# Read the documentation before enabling AddDefaultCharset.
# In general, it is only a good idea if you know that all your files
# have this encoding. It will override any encoding given in the files
# in meta http-equiv or xml encoding tags.

AddDefaultCharset UTF-8
```

</details>

## 10. Ключевые моменты

1. Конфигурация Apache на Debian с портом `8080`.
2. Конфигурация Apache на CentOS с портом `8080`.
3. Конфигурация Nginx на обеих ВМ с двумя backend-серверами.
4. Успешный вывод `nginx -t` на обеих ВМ.
5. Результаты проверки доступности обоих Apache.
6. Результаты повторных запросов через каждый Nginx.
7. Скриншоты страниц Debian и CentOS из браузера.
8. Команды настройки SELinux и firewall на CentOS.

Перед восстановлением исходных снимков VirtualBox сохранить все инструкции и конфигурации в Git, затем проверить, что по этим инструкциям работу можно полностью воспроизвести с нуля.