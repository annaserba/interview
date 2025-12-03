---
title: "Git и контроль версий"
description: "Основные команды Git и best practices"
category: "Tools"
difficulty: "easy"
tags: ["git", "version-control", "github"]
order: 23
---

## Основные команды

### Инициализация и клонирование

```bash
# Создать новый репозиторий
git init

# Клонировать репозиторий
git clone https://github.com/user/repo.git

# Клонировать конкретную ветку
git clone -b branch-name https://github.com/user/repo.git
```

### Базовые операции

```bash
# Проверить статус
git status

# Добавить файлы в staging
git add file.js
git add .                    # все файлы
git add *.js                 # по маске

# Коммит
git commit -m "Add feature"
git commit -am "Fix bug"     # add + commit для tracked файлов

# Отправить изменения
git push origin main

# Получить изменения
git pull origin main
git fetch origin             # без merge
```

## Ветки

```bash
# Создать ветку
git branch feature-name

# Переключиться на ветку
git checkout feature-name

# Создать и переключиться
git checkout -b feature-name

# Список веток
git branch                   # локальные
git branch -r                # удалённые
git branch -a                # все

# Удалить ветку
git branch -d feature-name   # безопасное удаление
git branch -D feature-name   # принудительное

# Переименовать ветку
git branch -m old-name new-name
```

## Merge и Rebase

### Merge

```bash
# Слить ветку в текущую
git checkout main
git merge feature-branch

# Отменить merge
git merge --abort
```

### Rebase

```bash
# Перебазировать текущую ветку
git checkout feature-branch
git rebase main

# Интерактивный rebase
git rebase -i HEAD~3         # последние 3 коммита

# Продолжить после разрешения конфликтов
git rebase --continue

# Отменить rebase
git rebase --abort
```

## История и логи

```bash
# Просмотр истории
git log
git log --oneline            # компактный вид
git log --graph              # с графом веток
git log -n 5                 # последние 5 коммитов

# История файла
git log -- file.js

# Кто изменял строки
git blame file.js

# Разница между коммитами
git diff
git diff HEAD~1              # с предыдущим коммитом
git diff branch1 branch2     # между ветками
```

## Отмена изменений

```bash
# Отменить изменения в файле
git checkout -- file.js

# Убрать из staging
git reset HEAD file.js

# Отменить последний коммит (сохранить изменения)
git reset --soft HEAD~1

# Отменить последний коммит (удалить изменения)
git reset --hard HEAD~1

# Создать коммит, отменяющий изменения
git revert commit-hash
```

## Stash (временное сохранение)

```bash
# Сохранить изменения
git stash
git stash save "message"

# Список stash
git stash list

# Применить последний stash
git stash apply
git stash pop                # apply + drop

# Применить конкретный stash
git stash apply stash@{0}

# Удалить stash
git stash drop stash@{0}
git stash clear              # все
```

## Удалённые репозитории

```bash
# Список удалённых репозиториев
git remote -v

# Добавить удалённый репозиторий
git remote add origin https://github.com/user/repo.git

# Изменить URL
git remote set-url origin new-url

# Удалить удалённый репозиторий
git remote remove origin
```

## Теги

```bash
# Создать тег
git tag v1.0.0
git tag -a v1.0.0 -m "Version 1.0.0"

# Список тегов
git tag

# Отправить теги
git push origin v1.0.0
git push origin --tags       # все теги

# Удалить тег
git tag -d v1.0.0            # локально
git push origin :refs/tags/v1.0.0  # удалённо
```

## Конфликты

```bash
# При merge/rebase возникает конфликт
# Открыть файл и разрешить конфликт вручную:
<<<<<<< HEAD
текущие изменения
=======
входящие изменения
>>>>>>> branch-name

# После разрешения:
git add file.js
git commit                   # для merge
git rebase --continue        # для rebase
```

## Best Practices

### Коммиты

```bash
# Хорошие сообщения коммитов
git commit -m "feat: add user authentication"
git commit -m "fix: resolve memory leak in parser"
git commit -m "docs: update API documentation"

# Conventional Commits
# feat: новая функция
# fix: исправление бага
# docs: документация
# style: форматирование
# refactor: рефакторинг
# test: тесты
# chore: рутинные задачи
```

### Workflow

```bash
# Feature Branch Workflow
git checkout -b feature/user-auth
# ... работа ...
git add .
git commit -m "feat: implement user authentication"
git push origin feature/user-auth
# Создать Pull Request на GitHub

# После ревью и одобрения
git checkout main
git pull origin main
git merge feature/user-auth
git push origin main
git branch -d feature/user-auth
```

## .gitignore

```gitignore
# Node
node_modules/
npm-debug.log
.env

# IDE
.vscode/
.idea/

# Build
dist/
build/
*.log

# OS
.DS_Store
Thumbs.db
```

## Вопросы для собеседования

### 1. В чём разница между merge и rebase?

**Ответ:**
- **Merge**: создаёт merge commit, сохраняет историю
- **Rebase**: переписывает историю, делает её линейной
- Merge для публичных веток, rebase для локальных

### 2. Что такое git stash?

**Ответ:**
Временное сохранение незакоммиченных изменений для быстрого переключения между задачами.

### 3. Как отменить последний коммит?

**Ответ:**
- `git reset --soft HEAD~1` — отменить коммит, сохранить изменения
- `git reset --hard HEAD~1` — отменить коммит и изменения
- `git revert HEAD` — создать новый коммит, отменяющий изменения

### 4. Что такое git cherry-pick?

**Ответ:**
Применение конкретного коммита из одной ветки в другую:
```bash
git cherry-pick commit-hash
```

### 5. Как разрешить конфликт при merge?

**Ответ:**
1. Открыть файл с конфликтом
2. Найти маркеры `<<<<<<<`, `=======`, `>>>>>>>`
3. Выбрать нужные изменения
4. Удалить маркеры
5. `git add file.js`
6. `git commit`
