# Apache HTTP Server (`apache2`) — конфигурация и диагностика в Debian 12

В Debian 12 Apache HTTP Server устанавливается как пакет `apache2`, а управляется через `systemd` как сервис `apache2`.

**Главные отличия от CentOS 7:**

| Назначение | CentOS 7 | Debian 12 |
|---|---|---|
| Пакет | `httpd` | `apache2` |
| Сервис | `httpd` | `apache2` |
| Главный каталог конфигурации | `/etc/httpd/` | `/etc/apache2/` |
| Главный конфигурационный файл | `httpd.conf` | `apache2.conf` |
| Дополнительные конфиги | `conf.d/` | `conf-enabled/`, `conf-available/` |
| Виртуальные хосты | `conf.d/` | `sites-available/`, `sites-enabled/` |
| Модули | `conf.modules.d/` | `mods-available/`, `mods-enabled/` |
| Web root | `/var/www/html/` | `/var/www/html/` |
| Логи | `/var/log/httpd/` | `/var/log/apache2/` |

---

# 1. Основные команды

## Проверить, установлен ли Apache

```bash
dpkg -l apache2
```

Информация о пакете:

```bash
apt show apache2
```

Список файлов, установленных пакетом:

```bash
dpkg -L apache2
```

## Управление сервисом

Запустить:

```bash
sudo systemctl start apache2
```

Остановить:

```bash
sudo systemctl stop apache2
```

Перезапустить:

```bash
sudo systemctl restart apache2
```

Перечитать конфигурацию без полного перезапуска:

```bash
sudo systemctl reload apache2
```

Включить автозапуск:

```bash
sudo systemctl enable apache2
```

Включить автозапуск и сразу запустить:

```bash
sudo systemctl enable --now apache2
```

Проверить состояние:

```bash
systemctl status apache2
```

Быстрые проверки:

```bash
systemctl is-active apache2
systemctl is-enabled apache2
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

Также можно использовать:

```bash
sudo apache2ctl configtest
```

Важно: `configtest` проверяет **синтаксис конфигурации**, но не гарантирует, что Apache сможет успешно запуститься или получить доступ ко всем ресурсам.

Например, синтаксис может быть правильным, но порт уже занят другим процессом.

---

# 3. Главный каталог конфигурации

Основной каталог Apache в Debian 12:

```text
/etc/apache2/
```

Структура:

```text
/etc/apache2/
├── apache2.conf
├── ports.conf
├── conf-available/
├── conf-enabled/
├── mods-available/
├── mods-enabled/
├── sites-available/
├── sites-enabled/
└── envvars
```

Главный конфигурационный файл:

```text
/etc/apache2/apache2.conf
```

В отличие от CentOS, Debian использует отдельную систему каталогов для включения конфигураций сайтов, модулей и дополнительных настроек.

---

# 4. `/etc/apache2/apache2.conf`

Это основной конфигурационный файл Apache.

Посмотреть:

```bash
sudo less /etc/apache2/apache2.conf
```

Или:

```bash
sudo cat /etc/apache2/apache2.conf
```

Найти конкретную настройку:

```bash
grep -n 'DocumentRoot' /etc/apache2/apache2.conf
```

Найти все `Listen`:

```bash
grep -Rni '^[[:space:]]*Listen' /etc/apache2/
```

Найти `ServerName`:

```bash
grep -Rni '^[[:space:]]*ServerName' /etc/apache2/
```

Найти `DirectoryIndex`:

```bash
grep -Rni '^[[:space:]]*DirectoryIndex' /etc/apache2/
```

Важно: в Debian некоторые настройки находятся не в `apache2.conf`, а в отдельных файлах и подключаемых конфигурациях.

---

# 5. `DocumentRoot`

Одна из главных настроек:

```apache
DocumentRoot /var/www/html
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

В Debian эта настройка стандартного сайта обычно находится здесь:

```text
/etc/apache2/sites-available/000-default.conf
```

Проверить:

```bash
grep -Rni 'DocumentRoot' /etc/apache2/
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

В Debian настройка обычно находится в:

```text
/etc/apache2/mods-enabled/dir.conf
```

Проверить:

```bash
grep -Rni '^[[:space:]]*DirectoryIndex' /etc/apache2/
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

Стандартный document root в Debian 12:

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
sudo nano /var/www/html/index.html
```

Например:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Debian Apache</title>
</head>
<body>
    <h1>Apache on Debian 12</h1>
</body>
</html>
```

После изменения HTML перезапуск Apache обычно **не требуется**.

---

# 8. `sites-available/` и `sites-enabled/`

Это важная особенность Debian.

Каталоги:

```text
/etc/apache2/sites-available/
/etc/apache2/sites-enabled/
```

`sites-available/` содержит конфигурации доступных виртуальных хостов.

`sites-enabled/` содержит конфигурации реально включённых виртуальных хостов. Обычно это символические ссылки на файлы из `sites-available/`.

Посмотреть:

```bash
ls -la /etc/apache2/sites-available/
ls -la /etc/apache2/sites-enabled/
```

Стандартная конфигурация сайта:

```text
/etc/apache2/sites-available/000-default.conf
```

Включить сайт:

```bash
sudo a2ensite 000-default.conf
```

Отключить сайт:

```bash
sudo a2dissite 000-default.conf
```

После изменения:

```bash
sudo apachectl configtest
sudo systemctl reload apache2
```

Таким образом, вместо ручного подключения каждого сайта в главном конфигурационном файле Debian использует команды `a2ensite` и `a2dissite`.

---

# 9. `mods-available/` и `mods-enabled/`

Каталоги:

```text
/etc/apache2/mods-available/
/etc/apache2/mods-enabled/
```

содержат конфигурации модулей Apache.

`mods-available/` — доступные модули.

`mods-enabled/` — включённые модули.

Посмотреть:

```bash
ls -la /etc/apache2/mods-available/
ls -la /etc/apache2/mods-enabled/
```

Включить модуль:

```bash
sudo a2enmod rewrite
```

Отключить модуль:

```bash
sudo a2dismod rewrite
```

Проверить загруженные модули:

```bash
sudo apache2ctl -M
```

или:

```bash
sudo apachectl -M
```

---

# 10. Порты

Основной файл настройки портов:

```text
/etc/apache2/ports.conf
```

Посмотреть:

```bash
sudo cat /etc/apache2/ports.conf
```

Найти все настройки `Listen`:

```bash
sudo grep -Rni '^[[:space:]]*Listen' /etc/apache2/
```

Например:

```text
Listen 8080
```

Это означает:

> Apache должен слушать TCP-порт 8080.

В нашей работе для этого в `/etc/apache2/ports.conf` добавляется:

```apache
Listen 8080
```

А в конфигурации сайта `/etc/apache2/sites-available/000-default.conf` используется:

```apache
<VirtualHost *:8080>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Но наличие `Listen 8080` в конфигурации ещё не означает, что Apache реально слушает этот порт.

Проверить фактическое состояние:

```bash
sudo ss -lntp | grep ':8080'
```

Например:

```text
LISTEN ... 0.0.0.0:8080 ... users:(("apache2",...))
```

То есть `apache2` действительно слушает порт.

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
Server: Apache/2.4.x (Debian)
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
/var/log/apache2/
```

Посмотреть:

```bash
sudo ls -la /var/log/apache2/
```

Обычно наиболее важные файлы:

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
```

## Access log

Показывает HTTP-запросы клиентов:

```bash
sudo tail -f /var/log/apache2/access.log
```

Например:

```text
127.0.0.1 - - [07/Oct/2026:20:05:44 +0300] "GET / HTTP/1.1" 200 35
```

Здесь видно:

- клиент;
- время;
- HTTP-запрос;
- HTTP-код;
- размер ответа.

## Error log

Показывает ошибки Apache:

```bash
sudo tail -f /var/log/apache2/error.log
```

Последние 20 строк:

```bash
sudo tail -20 /var/log/apache2/error.log
```

---

# 13. Логи systemd

Apache также пишет информацию в systemd journal.

Посмотреть:

```bash
sudo journalctl -u apache2
```

Последние сообщения:

```bash
sudo journalctl -u apache2 -n 50
```

Следить в реальном времени:

```bash
sudo journalctl -u apache2 -f
```

После неудачного запуска особенно полезно:

```bash
sudo systemctl status apache2
sudo journalctl -u apache2 -n 50
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
sudo tail -20 /var/log/apache2/error.log
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

Это не обязательно проблема AppArmor или permissions.

---

# 15. SELinux и AppArmor

В отличие от CentOS 7, в стандартной установке Debian 12 обычно используется AppArmor, а не SELinux.

Проверить AppArmor:

```bash
sudo aa-status
```

Если команда отсутствует:

```bash
sudo apt install apparmor-utils
```

Проверить SELinux:

```bash
getenforce
```

Если команда отсутствует, это обычно означает, что утилиты SELinux не установлены. Это не означает, что SELinux обязательно не работает или отключён.

Для обычной установки Apache на Debian 12 специальная настройка SELinux обычно не требуется.

Если возникает подозрение на блокировку AppArmor, проверить сообщения ядра:

```bash
sudo journalctl -k | grep -i apparmor
```

Не следует отключать AppArmor или SELinux только потому, что они присутствуют в системе. Сначала нужно установить, что именно они блокируют.

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
sudo chmod -R 777 /var/www/html
```

Это не является нормальным способом исправления Apache.

---

# 17. Проверка, какой конфигурационный файл реально содержит настройку

Вместо ручного поиска по всем файлам можно использовать:

```bash
sudo grep -Rni 'DocumentRoot' /etc/apache2/
```

Например:

```bash
sudo grep -Rni 'Listen' /etc/apache2/
```

```bash
sudo grep -Rni 'ServerName' /etc/apache2/
```

```bash
sudo grep -Rni 'DirectoryIndex' /etc/apache2/
```

```bash
sudo grep -Rni 'ProxyPass' /etc/apache2/
```

Посмотреть итоговую конфигурацию виртуальных хостов:

```bash
sudo apache2ctl -S
```

Эта команда особенно полезна, если несколько виртуальных хостов используют один порт.

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
sudo systemctl reload apache2
```

Если изменение требует полного перезапуска:

```bash
sudo systemctl restart apache2
```

После этого:

```bash
systemctl status apache2
```

---

# 19. Что проверять при проблеме с Apache

Полезная последовательность:

```bash
systemctl status apache2
```

```bash
sudo apachectl configtest
```

```bash
sudo ss -lntp
```

```bash
sudo tail -50 /var/log/apache2/error.log
```

```bash
sudo tail -50 /var/log/apache2/access.log
```

```bash
sudo journalctl -u apache2 -n 50
```

```bash
sudo aa-status
```

И затем проверить HTTP:

```bash
curl -I http://127.0.0.1:8080
```

---

# 20. Минимальный набор для текущей задачи

Для текущей задачи нам реально нужны в основном:

## Конфигурация

```text
/etc/apache2/apache2.conf
/etc/apache2/ports.conf
/etc/apache2/sites-available/000-default.conf
/etc/apache2/sites-enabled/
/var/www/html/
```

## Сервис

```bash
systemctl status apache2
sudo systemctl enable apache2
sudo systemctl restart apache2
```

## Проверка конфигурации

```bash
sudo apachectl configtest
```

## Проверка порта

```bash
sudo ss -lntp | grep ':8080'
```

## Проверка HTTP

```bash
curl -I http://localhost:8080
curl http://localhost:8080
```

## Логи

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
```

```bash
sudo tail -f /var/log/apache2/error.log
```

## AppArmor / SELinux

```bash
sudo aa-status
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
 apache2.conf   systemctl    ss :8080
        │
        ▼
      Files
   /var/www/html
        │
        ▼
       Logs
 /var/log/apache2/
```

Например:

```text
apachectl configtest → конфигурация синтаксически корректна

systemctl status     → процесс/сервис работает

ss -lntp             → процесс действительно слушает порт

curl                  → HTTP-запрос действительно работает

error.log             → почему запрос/операция могла завершиться ошибкой
```

Это разделение очень полезно при диагностике:

**«Apache установлен» ≠ «Apache запущен» ≠ «Apache слушает нужный порт» ≠ «Apache отдаёт нужный файл».**