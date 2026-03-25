# Doc Project — Working Guide

**Рабочая директория:** `~/.openclaw/workspace/doc-project/`  
**Оригинал:** `/root/doc/` (read-only для меня)

---

## Workflow

### 1. Я работаю в workspace
- Все изменения делаю в `~/.openclaw/workspace/doc-project/`
- Создаю новые файлы, редактирую существующие
- Коммичу изменения локально

### 2. Ты проверяешь и применяешь
```bash
# Смотришь что я сделал
cd ~/.openclaw/workspace/doc-project
git log --oneline -5
git diff HEAD~1

# Если всё ок — копируешь в /root/doc
sudo rsync -av --exclude='.git' ~/.openclaw/workspace/doc-project/ /root/doc/

# Коммитишь в оригинальном репо
cd /root/doc
git add .
git commit -m "Applied changes from workspace"
```

### 3. Синхронизация
```bash
# Если ты что-то изменил в /root/doc напрямую
cd ~/.openclaw/workspace/doc-project
git fetch origin
git reset --hard origin/master  # или origin/main
```

---

## Текущий статус

- ✅ Workspace создан: `~/.openclaw/workspace/doc-project/`
- ✅ Git инициализирован (branch: workspace-copy)
- ✅ Скопированы все 59 файлов из `/root/doc/`
- ✅ Initial commit сделан

**Готов к работе.** Говори номер задачи (например "1.1") — начинаю.

---

## Quick Commands

```bash
# Мой статус
cd ~/.openclaw/workspace/doc-project && git status

# Мои изменения
cd ~/.openclaw/workspace/doc-project && git diff

# Мои коммиты
cd ~/.openclaw/workspace/doc-project && git log --oneline -10

# Применить мои изменения в /root/doc
sudo rsync -av --exclude='.git' ~/.openclaw/workspace/doc-project/ /root/doc/
```
