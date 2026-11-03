# Apache HTTP Server (`httpd`) — конфигурация и диагностика в CentOS 7

В CentOS 7 Apache HTTP Server устанавливается как пакет `httpd`, а управляется через `systemd` как сервис `httpd`.

## 1. Основные команды

### Проверить, установлен ли Apache

```bash
rpm -q httpd
```

Информация о пакете:

```bash
rpm -qi httpd
```

Список файлов, установленных пакетом:

```bash
rpm -ql httpd
```

### Управление сервисом

Запустить:

```bash
sudo systemctl start httpd
```

Остановить:

```bash
sudo systemctl stop httpd
```

Перезапустить:

```bash
sudo systemctl restart httpd
```

Перечитать конфигурацию без полного перезапуска:

```bash
sudo systemctl reload httpd
```

Включить автозапуск:

```bash
sudo systemctl enable httpd
```

Включить автозапуск и сразу запустить:

```bash
sudo systemctl enable --now httpd
```

Проверить состояние:

```bash
systemctl status httpd
```

Быстрые проверки:

```bash
systemctl is-active httpd
systemctl is-enabled httpd
```

Ожидаемые результаты:

```text
active
enabled
```

---

# 2. Проверка конфигурации

Перед перезапуском Apache полезно проверить синтаксис:

```bash
sudo apachectl configtest
```

Успешный результат:

```text
Syntax OK
```

Важно: `configtest` проверяет **синтаксис конфигурации**, но не гарантирует, что Apache сможет успешно запуститься или получить доступ ко всем ресурсам.

Например, синтаксис может быть правильным, но порт уже занят другим процессом.

---

# 3. Главный каталог конфигурации

Основной каталог Apache в CentOS 7:

```text
/etc/httpd/
```

Структура:

```text
/etc/httpd/
├── conf/
│   └── httpd.conf
├── conf.d/
├── conf.modules.d/
├── logs/
├── modules/
├── run/
└── state/
```

Главный конфигурационный файл:

```text
/etc/httpd/conf/httpd.conf
```

---

# 4. `/etc/httpd/conf/httpd.conf`

Это основной конфигурационный файл Apache.

Посмотреть:

```bash
sudo less /etc/httpd/conf/httpd.conf
```

Или:

```bash
sudo cat /etc/httpd/conf/httpd.conf
```

Найти конкретную настройку:

```bash
grep -n 'DocumentRoot' /etc/httpd/conf/httpd.conf
```

Найти все `Listen`:

```bash
grep -Rni '^Listen' /etc/httpd/conf /etc/httpd/conf.d
```

Найти `ServerName`:

```bash
grep -Rni '^ServerName' /etc/httpd/
```

Найти `DirectoryIndex`:

```bash
grep -Rni '^DirectoryIndex' /etc/httpd/
```

---

# 5. `DocumentRoot`

Одна из главных настроек:

```apache
DocumentRoot "/var/www/html"
```

Она определяет, какой каталог соответствует корню сайта `/`.

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
grep -Rni 'DocumentRoot' /etc/httpd/
```

---

# 6. `DirectoryIndex`

Когда клиент запрашивает:

```text
http://server/
```

Apache получает запрос к каталогу:

```text
/var/www/html/
```

Дальше Apache ищет файл, указанный в `DirectoryIndex`.

Например:

```apache
DirectoryIndex index.html
```

Проверить:

```bash
grep -Rni '^DirectoryIndex' /etc/httpd/
```

Если `/var/www/html/` пуст:

```text
/var/www/html/
└── nothing
```

Apache не может найти `index.html`.

Если просмотр содержимого каталога запрещён, результатом будет:

```text
403 Forbidden
```

---

# 7. Web root

Стандартный document root в CentOS 7:

```text
/var/www/html/
```

Проверить содержимое:

```bash
ls -la /var/www/html/
```

Проверить сам каталог:

```bash
ls -ld /var/www /var/www/html
```

Создать страницу:

```bash
sudo vi /var/www/html/index.html
```

Например:

```html
<!DOCTYPE html>
<html>
<head>
    <title>CentOS Apache</title>
</head>
<body>
    <h1>Apache on CentOS 7</h1>
</body>
</html>
```

После изменения HTML перезапуск Apache обычно **не требуется**.

---

# 8. `conf.d/`

Каталог:

```text
/etc/httpd/conf.d/
```

содержит дополнительные конфигурационные файлы Apache.

Например:

```text
/etc/httpd/conf.d/example.conf
/etc/httpd/conf.d/my-site.conf
```

Обычно основной `httpd.conf` подключает `.conf`-файлы из этого каталога.

Проверить:

```bash
ls -la /etc/httpd/conf.d/
```

Найти подключения:

```bash
grep -n 'Include' /etc/httpd/conf/httpd.conf
```

Поэтому вместо изменения огромного `httpd.conf` часто удобно создать отдельный файл:

```bash
sudo vi /etc/httpd/conf.d/my-site.conf
```

После изменения:

```bash
sudo apachectl configtest
sudo systemctl reload httpd
```

---

# 9. `conf.modules.d/`

Каталог:

```text
/etc/httpd/conf.modules.d/
```

содержит конфигурацию загрузки Apache-модулей.

Например, здесь можно увидеть строки типа:

```apache
LoadModule ...
```

Посмотреть:

```bash
ls -la /etc/httpd/conf.modules.d/
```

Проверить загруженные модули:

```bash
sudo httpd -M
```

или:

```bash
sudo apachectl -M
```

---

# 10. Порты

Найти все настройки `Listen`:

```bash
sudo grep -Rni '^Listen' /etc/httpd/conf /etc/httpd/conf.d
```

Например:

```text
Listen 8080
```

Это означает:

> Apache должен слушать TCP-порт 8080.

Но наличие `Listen 8080` в конфигурации ещё не означает, что Apache реально слушает этот порт.

Проверить фактическое состояние:

```bash
sudo ss -lntp | grep ':8080'
```

В нашем случае мы получили:

```text
LISTEN ... [::]:8080 ... users:(("httpd",...))
```

То есть `httpd` действительно слушает порт.

---

# 11. Проверка HTTP

Проверить ответ целиком:

```bash
curl http://localhost:8080
```

Получить только HTTP-заголовки:

```bash
curl -I http://localhost:8080
```

Успешный результат обычно содержит:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.6
```

Основные HTTP-коды:

```text
200 OK
403 Forbidden
404 Not Found
500 Internal Server Error
```

Например:

```text
403
```

означает, что Apache получил запрос, но запрещает доступ.

---

# 12. Логи Apache

Основной каталог логов:

```text
/var/log/httpd/
```

Посмотреть:

```bash
sudo ls -la /var/log/httpd/
```

Обычно наиболее важные файлы:

```text
/var/log/httpd/access_log
/var/log/httpd/error_log
```

## Access log

Показывает HTTP-запросы клиентов:

```bash
sudo tail -f /var/log/httpd/access_log
```

Например:

```text
::1 - - [07/Oct/2026:20:05:44 +0300] "GET / HTTP/1.1" 403 4897
```

Здесь видно:

* клиент;
* время;
* HTTP-запрос;
* HTTP-код;
* размер ответа.

## Error log

Показывает ошибки Apache:

```bash
sudo tail -f /var/log/httpd/error_log
```

Последние 20 строк:

```bash
sudo tail -20 /var/log/httpd/error_log
```

---

# 13. Логи systemd

Apache также пишет информацию в systemd journal.

Посмотреть:

```bash
sudo journalctl -u httpd
```

Последние сообщения:

```bash
sudo journalctl -u httpd -n 50
```

Следить в реальном времени:

```bash
sudo journalctl -u httpd -f
```

После неудачного запуска особенно полезно:

```bash
sudo systemctl status httpd
sudo journalctl -u httpd -n 50
```

---

# 14. Как диагностировать `403 Forbidden`

Если:

```bash
curl -I http://localhost:8080
```

возвращает:

```text
HTTP/1.1 403 Forbidden
```

проверяем:

```bash
ls -la /var/www/html/
```

```bash
ls -ld /var/www /var/www/html
```

```bash
sudo tail -20 /var/log/httpd/error_log
```

Типичный случай:

```text
No matching DirectoryIndex (index.html) found,
and server-generated directory index forbidden
```

означает:

```text
/var/www/html/
        ↓
нет index.html
        ↓
directory listing запрещён
        ↓
403 Forbidden
```

Это не обязательно проблема SELinux или permissions.

---

# 15. SELinux

Проверить режим:

```bash
getenforce
```

Возможные результаты:

```text
Enforcing
Permissive
Disabled
```

Проверить boolean, связанный с сетевыми соединениями HTTP-процессов:

```bash
getsebool httpd_can_network_connect
```

Посмотреть SELinux-контекст файла:

```bash
ls -Z /var/www/html/
```

Посмотреть контекст каталога:

```bash
ls -Zd /var/www/html
```

Если SELinux действительно вызывает проблему, ищем AVC-denial:

```bash
sudo ausearch -m AVC -ts recent
```

Не следует отключать SELinux только потому, что он включён. Сначала нужно установить, что именно он блокирует.

---

# 16. Права файлов

Проверить права:

```bash
ls -la /var/www/html/
```

Проверить каталог:

```bash
ls -ld /var/www/html
```

Типичный безопасный вариант для обычного HTML:

```text
drwxr-xr-x  /var/www/html
-rw-r--r--  /var/www/html/index.html
```

Изменять права без причины не стоит.

Особенно не следует автоматически делать:

```bash
chmod -R 777 /var/www/html
```

Это не является нормальным способом исправления Apache.

---

# 17. Проверка, какой конфигурационный файл реально содержит настройку

Вместо ручного поиска по всем файлам можно использовать:

```bash
sudo grep -Rni 'DocumentRoot' /etc/httpd/
```

Например:

```bash
sudo grep -Rni 'Listen' /etc/httpd/
```

```bash
sudo grep -Rni 'ServerName' /etc/httpd/
```

```bash
sudo grep -Rni 'DirectoryIndex' /etc/httpd/
```

```bash
sudo grep -Rni 'ProxyPass' /etc/httpd/
```

---

# 18. Проверка конфигурации перед изменением сервиса

Хороший порядок:

```bash
sudo apachectl configtest
```

Если:

```text
Syntax OK
```

тогда:

```bash
sudo systemctl reload httpd
```

Если изменение требует полного перезапуска:

```bash
sudo systemctl restart httpd
```

После этого:

```bash
systemctl status httpd
```

---

# 19. Что проверять при проблеме с Apache

Полезная последовательность:

```bash
systemctl status httpd
```

```bash
sudo apachectl configtest
```

```bash
sudo ss -lntp
```

```bash
sudo tail -50 /var/log/httpd/error_log
```

```bash
sudo tail -50 /var/log/httpd/access_log
```

```bash
sudo journalctl -u httpd -n 50
```

```bash
getenforce
```

```bash
ls -Z /var/www/html/
```

И затем проверить HTTP:

```bash
curl -I http://127.0.0.1:8080
```

---

# 20. Минимальный набор для текущей задачи

Для текущей задачи нам реально нужны в основном:

### Конфигурация

```text
/etc/httpd/conf/httpd.conf
/etc/httpd/conf.d/
/var/www/html/
```

### Сервис

```bash
systemctl status httpd
systemctl enable httpd
sudo systemctl restart httpd
```

### Проверка конфигурации

```bash
sudo apachectl configtest
```

### Проверка порта

```bash
sudo ss -lntp | grep ':8080'
```

### Проверка HTTP

```bash
curl -I http://localhost:8080
curl http://localhost:8080
```

### Логи

```text
/var/log/httpd/access_log
/var/log/httpd/error_log
```

```bash
sudo tail -f /var/log/httpd/error_log
```

### SELinux

```bash
getenforce
```

---

## Главное различие

Нужно различать эти четыре уровня:

```text
                 Apache
                    │
        ┌───────────┼───────────┐
        │           │           │
   Configuration   Service     Network
        │           │           │
   httpd.conf    systemctl    ss :8080
        │
        ▼
     Files
   /var/www/html
        │
        ▼
      Logs
 /var/log/httpd/
```

Например:

```text
apachectl configtest → конфигурация синтаксически корректна

systemctl status     → процесс/сервис работает

ss -lntp             → процесс действительно слушает порт

curl                  → HTTP-запрос действительно работает

error_log             → почему запрос/операция могла завершиться ошибкой
```

Это разделение очень полезно при диагностике: **"Apache установлен" ≠ "Apache запущен" ≠ "Apache слушает нужный порт" ≠ "Apache отдаёт нужный файл".**
