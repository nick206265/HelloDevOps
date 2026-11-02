# Nginx на CentOS 7 — установка, настройка и диагностика

## 1. Установка

Проверить, установлен ли Nginx:

```bash
rpm -q nginx
```

Если пакет не установлен:

```bash
sudo yum install nginx
```

В CentOS 7 пакет Nginx может отсутствовать в стандартных репозиториях CentOS. В этом случае используется официальный репозиторий Nginx.

После установки проверить:

```bash
nginx -v
```

Например:

```text
nginx version: nginx/1.x.x
```

---

## 2. Управление сервисом

Основные команды:

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl status nginx
```

`reload` перечитывает конфигурацию без полного перезапуска процесса. Для изменения конфигурации это обычно предпочтительнее `restart`.

Добавить Nginx в автозапуск:

```bash
sudo systemctl enable nginx
```

Или одновременно включить автозапуск и запустить сейчас:

```bash
sudo systemctl enable --now nginx
```

Проверить:

```bash
systemctl is-active nginx
systemctl is-enabled nginx
```

Ожидаемый результат:

```text
active
enabled
```

---

## 3. Основные файлы конфигурации

Главный конфигурационный файл:

```text
/etc/nginx/nginx.conf
```

Дополнительные конфигурации обычно находятся здесь:

```text
/etc/nginx/conf.d/
```

Посмотреть содержимое:

```bash
ls -la /etc/nginx/
ls -la /etc/nginx/conf.d/
```

Типичная структура:

```text
/etc/nginx/
├── nginx.conf
├── conf.d/
│   └── *.conf
└── ...
```

`nginx.conf` содержит глобальные настройки Nginx и обычно подключает конфигурации из `conf.d/`.

Проверить это можно:

```bash
grep -n 'conf.d' /etc/nginx/nginx.conf
```

---

## 4. Проверка конфигурации

Перед перезапуском или reload рекомендуется проверить конфигурацию:

```bash
sudo nginx -t
```

Успешный результат выглядит примерно так:

```text
syntax is ok
test is successful
```

Это означает, что конфигурация синтаксически корректна.

Важно: `nginx -t` проверяет конфигурацию, но не гарантирует, что Nginx сможет успешно занять нужный порт или что backend Apache доступен.

---

## 5. Проверка порта

В нашей схеме:

```text
Nginx   → 80
Apache  → 8080
```

Проверить, кто слушает порты:

```bash
sudo ss -lntp
```

Или конкретно:

```bash
sudo ss -lntp | grep -E ':80|:8080'
```

Ожидаем примерно:

```text
LISTEN ... :80   ... nginx
LISTEN ... :8080 ... httpd
```

Это означает:

* Nginx принимает внешние HTTP-запросы на `80`;
* Apache принимает запросы на `8080`.

---

## 6. Как Nginx работает с Apache

Nginx в нашем случае не должен сам отдавать HTML-страницу.

Он выступает как **reverse proxy**:

```text
Client
   │
   │ HTTP :80
   ▼
Nginx
   │
   │ HTTP :8080
   ▼
Apache
   │
   ▼
/var/www/html/index.html
```

Для этого используется директива:

```nginx
proxy_pass
```

Например:

```nginx
location / {
    proxy_pass http://127.0.0.1:8080;
}
```

Здесь:

```text
http://127.0.0.1:8080
```

означает Apache, работающий на этой же VM.

---

## 7. Конфигурация Nginx → Apache

Удобно создать отдельный файл:

```bash
sudo vi /etc/nginx/conf.d/apache.conf
```

Минимальная конфигурация:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

### Что означает конфигурация

```nginx
server {
```

Создаёт виртуальный HTTP-сервер Nginx.

```nginx
listen 80;
```

Nginx принимает HTTP-соединения на порту `80`.

```nginx
server_name _;
```

`_` здесь используется как простой catch-all вариант: сервер будет принимать запросы независимо от конкретного имени хоста.

```nginx
location / {
```

Правило применяется ко всем URL, начинающимся с `/`.

```nginx
proxy_pass http://127.0.0.1:8080;
```

Nginx передаёт запрос Apache на `127.0.0.1:8080`.

---

## 8. Проверка конфигурации и перезапуск

После изменения конфигурации:

```bash
sudo nginx -t
```

Если результат:

```text
syntax is ok
test is successful
```

можно применить конфигурацию:

```bash
sudo systemctl reload nginx
```

Проверить:

```bash
systemctl status nginx
```

---

## 9. Проверка через curl

Сначала Apache:

```bash
curl http://localhost:8080
```

Мы уже проверили, что он возвращает:

```text
Apache on CentOS 7
```

Теперь проверяем Nginx:

```bash
curl http://localhost
```

Ожидаемый результат:

```text
<!DOCTYPE html>
<html>
...
<h1>Apache on CentOS 7</h1>
...
</html>
```

То есть:

```text
curl → Nginx :80 → Apache :8080 → index.html
```

Чтобы посмотреть только HTTP-заголовки:

```bash
curl -I http://localhost
```

Ожидается:

```text
HTTP/1.1 200 OK
```

---

## 10. Проверка с IP-адресом VM

Сначала узнать IP:

```bash
ip addr
```

или:

```bash
hostname -I
```

Например:

```text
192.168.1.70
```

Тогда на самой VM:

```bash
curl http://192.168.1.70
```

А с Ubuntu-хоста:

```bash
curl http://192.168.1.70
```

В браузере можно открыть:

```text
http://192.168.1.70
```

В результате должна отображаться страница:

```text
Apache on CentOS 7
```

Важно: браузер обращается к Nginx, а страницу фактически генерирует Apache.

---

## 11. Логи Nginx

Основные логи:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Посмотреть последние записи:

```bash
sudo tail -20 /var/log/nginx/access.log
sudo tail -20 /var/log/nginx/error.log
```

Следить за логом в реальном времени:

```bash
sudo tail -f /var/log/nginx/access.log
```

И отдельно:

```bash
sudo tail -f /var/log/nginx/error.log
```

Например, после:

```bash
curl http://localhost
```

в `access.log` должна появиться запись о запросе.

---

## 12. Логи Apache после запроса через Nginx

Можно проверить сразу оба сервиса.

В одном терминале:

```bash
sudo tail -f /var/log/nginx/access.log
```

Во втором:

```bash
sudo tail -f /var/log/httpd/access_log
```

В третьем или с другого терминала:

```bash
curl http://localhost
```

Тогда можно увидеть цепочку:

```text
curl
 ↓
Nginx access.log
 ↓
Nginx proxy_pass
 ↓
Apache access_log
 ↓
index.html
```

Это очень наглядная проверка того, что reverse proxy действительно работает.

---

## 13. Если Nginx не запускается

Сначала:

```bash
sudo nginx -t
```

Затем:

```bash
sudo systemctl status nginx
```

И:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

Проверить занятые порты:

```bash
sudo ss -lntp
```

Особенно важно проверить `:80`.

Если Apache всё ещё слушает `80`, Nginx не сможет занять этот порт:

```text
Apache → :80
Nginx  → :80
```

На одном IP одновременно оба сервиса на одном TCP-порту работать не могут.

В нашей конфигурации должно быть:

```text
Nginx  → :80
Apache → :8080
```

---

## 14. Если Nginx работает, но возвращает ошибку

Например:

```text
502 Bad Gateway
```

Это обычно означает, что Nginx не может связаться с backend.

Проверить Apache напрямую:

```bash
curl http://127.0.0.1:8080
```

Если Apache отвечает:

```text
Apache on CentOS 7
```

то backend работает.

Затем проверить Nginx:

```bash
curl http://127.0.0.1
```

Если Apache работает, а Nginx получает `502`, проверить:

```bash
sudo tail -50 /var/log/nginx/error.log
```

и конфигурацию:

```bash
sudo nginx -T
```

---

## 15. Полезная команда `nginx -T`

```bash
sudo nginx -T
```

Показывает итоговую конфигурацию Nginx, включая подключённые файлы.

Это полезнее, чем смотреть только:

```text
/etc/nginx/nginx.conf
```

потому что конфигурация из `conf.d/*.conf` также будет показана.

Можно искать конкретные директивы:

```bash
sudo nginx -T | grep -n 'proxy_pass'
```

или:

```bash
sudo nginx -T | grep -n 'listen'
```

---

## 16. Основные команды для диагностики

### Сервис

```bash
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx
```

### Конфигурация

```bash
sudo nginx -t
sudo nginx -T
```

### Порты

```bash
sudo ss -lntp
```

### HTTP

```bash
curl http://localhost
curl -I http://localhost
curl http://127.0.0.1:8080
```

### Логи

```bash
sudo tail -20 /var/log/nginx/access.log
sudo tail -20 /var/log/nginx/error.log
sudo journalctl -u nginx -n 50 --no-pager
```

### Конфигурационные файлы

```bash
sudo vi /etc/nginx/nginx.conf
sudo vi /etc/nginx/conf.d/apache.conf
```

---

## 17. Важное различие Apache и Nginx

После настройки у нас будут два разных уровня:

```text
                  CentOS 7
┌─────────────────────────────────────────┐
│                                         │
│  Nginx                                   │
│  :80                                     │
│    │                                     │
│    │ proxy_pass                          │
│    ▼                                     │
│  Apache (httpd)                          │
│  :8080                                   │
│    │                                     │
│    ▼                                     │
│  /var/www/html/index.html                │
│                                         │
└─────────────────────────────────────────┘
```

Nginx отвечает за:

* приём входящих HTTP-запросов;
* reverse proxy;
* позднее — load balancing.

Apache отвечает за:

* обработку HTTP-запроса;
* выдачу содержимого сайта.

---

## 18. Следующий этап работы

Сначала проверяем всё на **одной CentOS VM**:

```text
curl
 ↓
Nginx :80
 ↓
Apache :8080
 ↓
CentOS HTML
```

После того как это работает, аналогичная схема создаётся на Debian:

```text
curl
 ↓
Nginx :80
 ↓
Apache :8080
 ↓
Debian HTML
```

И только после этого Nginx можно настроить на два backend:

```text
                  ┌── Apache Debian :8080
                  │
Browser → Nginx ──┤
                  │
                  └── Apache CentOS :8080
```

На этом этапе `proxy_pass` будет заменён/дополнен upstream-конфигурацией для балансировки между двумя Apache.
