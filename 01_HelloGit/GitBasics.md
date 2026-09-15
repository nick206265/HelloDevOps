# Задание
1. Зарегистрироваться и создать репозиторий (публичный или приватный) на Github.
2. Склонировать репозиторий себе на комп.
3. Создать новую ветку.
4. Попробовать создать файл, запушить, поизучать основные команды.
5. Создать файл формата .md и описать в нем инструкцию по клонированию репозитория и по базовым командам для работы с гитом из консоли.
6. Добавить Collaborators проекта и прислать Merge Request на проверку.

# Установка

Для Ubuntu:

```bash
sudo apt install git
```

Проверка версии:

```bash
git --version
```

# Базовые команды Git:

## Работа с репозиториями:

Клонировать себе удалённый репозиторий:

```bash
git clone <repo-address>
```

где <repo-address> может быть для HTTPS:

```bash
https://github.com/username/repoName.git
```

или для SSH (потребуется создавать и привязывать ключи доступа):

```bash
git@github.com:username/repoName.git
```

Создать репозиторий в текущей директории:

```bash
git init
```

Просмотр текущего состояния локального репозитория:

```bash
git status
```

Добавить удалённый репозиторий:

```bash
git remote add <repo-name> <repo-address>
```

например:

```bash
git remote add origin git@github.com:username/repoName.git
```

Создать основную ветку и отправить первый коммит в удалённый репозиторий:

```bash
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

Здесь *origin* — имя удалённого репозитория, а *main* — основная ветка.

Список удалённых репозиториев с именами:

```bash
git remote -v
```

## Работа с ветками

Создание новой ветки <branch_name>:

```bash
git branch <branch_name>
```

Переход на существующую ветку <branch_name>:

```bash
git checkout <branch_name>
```

Список всех веток:

```bash
git branch
```

Удалить ветку <branch_name>:

```bash
git branch -d <branch_name>
```

Переименовать *текущую* ветку в <new_branch_name>:

```bash
git branch -M <new_branch_name>
```



## Отправка/получение изменений

Индекс (staging area) - промежуточное хранилище между текущими файлами(рабочим состоянием) и коммитом, чтобы можно было выбрать, что именно попадёт в следующий коммит.

Добавить изменения в файлах file1 и file2 в индекс:
```bash
git add file1 file2
```

Создать коммит с описанием "Files changed":
```bash
git commit -m "Files changed"
```

Отправить локальные изменения в ветку <branch_name> удалённого репозитория <repo_name>:

```bash
git push <repo_name> <branch_name>
```

например:
```bash
git push origin main
```

Указать удалённую ветку как upstream для локальной:
```bash
git push -u origin main
```

после этого
```bash
git push
```
будет отправлять изменения в указанную upstream-ветку.

Просмотр текущих изменений в рабочей директории (разница между рабочим состоянием и индексом):
```bash
git diff
```

Просмотр текущих изменений, добавленных в индекс (будут включены в следующий коммит):
```bash
git diff --cached
```

Получить изменения из удалённого репозитория, не изменяя текущую локальную ветку:
```bash
git fetch
```

Получить изменения из удалённого репозитория и интегрировать их в текущую локальную ветку:
```bash
git pull
```

