# Перемещение по файловой системе:

1. Перейти в корень диска.

```bash
cd /
```

2. Перейти в домашний каталог.

```bash
cd ~
```

3. Перейти в каталог на уровень выше.

```bash
cd ..
```

4. Отобразить текущий каталог.

```bash
ls .
```

# Система:

1. Как посмотреть дистрибутив системы.

```bash
cat /etc/os-release
```

2. Как посмотреть ядро системы.

```bash
uname -a
```

3. Как посмотреть ip адрес системы.

```bash
hostname -I
```

4. Как проверить доступ до сайта vl.ru .

```bash
ping vl.ru
```

5. Как посмотреть установлены ли редакторы vi и nano.

```bash
which vi
which nano
```

# Работа с файлами:

1. Вывести всё содержимое /etc/passwd

```bash
cat /etc/passwd
```

2. Вывести первые 2 строки /etc/passwd

```bash
head -2 /etc/passwd
```

3. Вывести последние 3 строки /etc/passwd

```bash
tail -3 /etc/passwd
```

4. Вывести 4-ю строку /etc/passwd

```bash
sed -n '4p' /etc/passwd
```

5. Вывести строку содержащую слово "root" из /etc/passwd

```bash
grep "root" /etc/passwd
```

6. Скопировать /etc/passwd в /tmp

```bash
cp /etc/passwd /tmp
```

7. Посмотреть права /tmp/passwd

```bash
ls -l /tmp/passwd
```

8. Разрешить всем изменять /tmp/passwd

```bash
chmod a+w /tmp/passwd
```

9. Добавить в /tmp/passwd новую строку произвольного содержания редактором vi

```bash
vi /tmp/passwd
```

10. Добавить в /tmp/passwd новую строку произвольного содержания редактором nano

```bash
nano /tmp/passwd
```

11. Выведи в консоль текущую дату

```bash
date
```

12. Создай файл time.sh, в котором будет команда вывода текущей даты. Как его сделать исполняемым? Как его запустить? Как его запустить находясь в корне?

```bash
vim time.sh
```

```bash
chmod +x time.sh
```

```bash
./time.sh
```

```text
<from '/'>: ~/time.sh
```
