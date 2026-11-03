# NGINX — конфигурация и диагностика в Debian 12

В Debian 12 Nginx устанавливается как пакет `nginx`, а управляется через `systemd` как сервис `nginx`.

**Основные отличия от CentOS 7:**

| Назначение | CentOS 7 | Debian 12 |
|---|---|---|
| Пакет | `nginx` | `nginx` |
| Сервис | `nginx` | `nginx` |
| Главный каталог конфигурации | `/etc/nginx/` | `/etc/nginx/` |
| Главный конфигурационный файл | `nginx.conf` | `nginx.conf` |
| Дополнительные конфиги | `conf.d/` | `conf.d/` |
| Конфигурации сайтов | `conf.d/` | `sites-available/`, `sites-enabled/` |
| Web root по умолчанию | `/usr/share/nginx/html/` | `/var/www/html/` или `/usr/share/nginx/html/` |
| Логи | `/var/log/nginx/` | `/var/log/nginx/` |

---

# 1. Основные команды

## Проверить, установлен ли Nginx

```bash
dpkg -l nginx
```

Информация о пакете:

```bash
apt show nginx
```

Список файлов, установленных пакетом:

```bash
dpkg -L nginx
```

Проверить версию:

```bash
nginx -v
```

Подробная информация о сборке:

```bash
nginx -V
```

## Управление сервисом

Запустить:

```bash
sudo systemctl start nginx
```

Остановить:

```bash
sudo systemctl stop nginx
```

Перезапустить:

```bash
sudo systemctl restart nginx
```

Перечитать конфигурацию без полного перезапуска:

```bash
sudo systemctl reload nginx
```

Включить автозапуск:

```bash
sudo systemctl enable nginx
```

Включить автозапуск и сразу запустить:

```bash
sudo systemctl enable --now nginx
```

Проверить состояние:

```bash
systemctl status nginx
```

Быстрые проверки:

```bash
systemctl is-active nginx
systemctl is-enabled nginx
```

Ожидаемые результаты:

```text
active
enabled
```

---

# 2. Проверка конфигурации

Перед перезапуском или перезагрузкой Nginx полезно проверить синтаксис:

```bash
sudo nginx -t
```

Успешный результат:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Важно: `nginx -t` проверяет синтаксис конфигурации и возможность открытия используемых файлов, но не гарантирует, что все backend-серверы доступны.

Например, конфигурация может быть правильной, но Apache на `192.168.1.70:8080` может быть недоступен.

Посмотреть полную загруженную конфигурацию:

```bash
sudo nginx -T
```

Это полезно, когда непонятно, из какого файла загружается определённая настройка.

---

# 3. Главный каталог конфигурации

Основной каталог Nginx в Debian 12:

```text
/etc/nginx/
```

Структура:

```text
/etc/nginx/
├── nginx.conf
├── conf.d/
├── sites-available/
├── sites-enabled/
├── snippets/
├── mime.types
├── proxy_params
├── fastcgi_params
└── modules-enabled/
```

Главный конфигурационный файл:

```text
/etc/nginx/nginx.conf
```

В Debian используется система каталогов `sites-available/` и `sites-enabled/` для управления конфигурациями сайтов.

---

# 4. `/etc/nginx/nginx.conf`

Это основной конфигурационный файл Nginx.

Посмотреть:

```bash
sudo less /etc/nginx/nginx.conf
```

Или:

```bash
sudo cat /etc/nginx/nginx.conf
```

Найти конкретную настройку:

```bash
grep -n 'worker_processes' /etc/nginx/nginx.conf
```

Найти подключения конфигурационных файлов:

```bash
grep -n 'include' /etc/nginx/nginx.conf
```

Найти все директивы `listen`:

```bash
sudo grep -Rni 'listen' /etc/nginx/
```

Найти `server_name`:

```bash
sudo grep -Rni 'server_name' /etc/nginx/
```

Найти `proxy_pass`:

```bash
sudo grep -Rni 'proxy_pass' /etc/nginx/
```

Важно: в Debian конфигурация сайта обычно находится не непосредственно в `nginx.conf`, а в отдельном файле внутри `sites-available/`, который подключается через `sites-enabled/`.

---

# 5. `root`

Одна из главных настроек Nginx:

```nginx
root /var/www/html;
```

Она определяет каталог, из которого Nginx отдаёт статические файлы.

Например:

```text
URL                         Файл
─────────────────────────────────────────────────
/                           /var/www/html/
/index.html                 /var/www/html/index.html
/css/style.css              /var/www/html/css/style.css
/images/logo.png            /var/www/html/images/logo.png
```

Проверить:

```bash
sudo grep -Rni 'root' /etc/nginx/
```

Важно: в нашей работе Nginx не должен отдавать собственные HTML-страницы. Он должен передавать запросы Apache через `proxy_pass`.

---

# 6. `index`

Когда клиент запрашивает:

```text
http://server/
```

Nginx получает запрос к корню сайта `/`.

Далее он использует директиву `index`, чтобы определить файл, который нужно отдать.

Например:

```nginx
index index.html index.htm;
```

Проверить:

```bash
sudo grep -Rni 'index' /etc/nginx/