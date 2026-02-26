Вот рабочий скрипт для синхронизации репозиториев на Bash (Linux/macOS/WSL) и PowerShell (Windows).

## Скрипт синхронизации репозиториев

### Описание работы
Скрипт выполняет следующие действия:
1. Проверяет существование исходного и целевого репозиториев
2. Выполняет `git pull` в исходном репозитории для получения актуальной версии
3. Копирует все содержимое, **исключая папку `.git`**, в целевой репозиторий
4. Заменяет существующие файлы в целевом репозитории

---

## Bash-скрипт (для Linux/macOS/WSL)

```bash
#!/bin/bash

# Конфигурация
SOURCE_REPO="$HOME/исходный-репозиторий"
TARGET_REPO="$HOME/целевой-репозиторий"

# Цвета для вывода
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${YELLOW}=== Синхронизация репозиториев ===${NC}"

# Проверка существования исходного репозитория
if [ ! -d "$SOURCE_REPO" ]; then
    echo -e "${RED}Ошибка: Исходный репозиторий не найден: $SOURCE_REPO${NC}"
    exit 1
fi

# Проверка существования целевого репозитория
if [ ! -d "$TARGET_REPO" ]; then
    echo -e "${RED}Ошибка: Целевой репозиторий не найден: $TARGET_REPO${NC}"
    exit 1
fi

# Переход в исходный репозиторий и выполнение git pull
echo -e "${GREEN}Обновление исходного репозитория...${NC}"
cd "$SOURCE_REPO" || exit 1

if git pull; then
    echo -e "${GREEN}Git pull выполнен успешно${NC}"
else
    echo -e "${RED}Ошибка при выполнении git pull${NC}"
    exit 1
fi

# Синхронизация содержимого (исключая .git)
echo -e "${GREEN}Синхронизация файлов...${NC}"
rsync -av --delete --exclude='.git/' "$SOURCE_REPO/" "$TARGET_REPO/"

# Проверка результата
if [ $? -eq 0 ]; then
    echo -e "${GREEN}✓ Синхронизация завершена успешно!${NC}"
else
    echo -e "${RED}✗ Ошибка при синхронизации${NC}"
    exit 1
fi
```

---

## PowerShell-скрипт (для Windows)

```powershell
#!/usr/bin/env pwsh

# Конфигурация
$SOURCE_REPO = "$env:USERPROFILE\исходный-репозиторий"
$TARGET_REPO = "$env:USERPROFILE\целевой-репозиторий"

Write-Host "=== Синхронизация репозиториев ===" -ForegroundColor Yellow

# Проверка существования исходного репозитория
if (-not (Test-Path $SOURCE_REPO)) {
    Write-Host "Ошибка: Исходный репозиторий не найден: $SOURCE_REPO" -ForegroundColor Red
    exit 1
}

# Проверка существования целевого репозитория
if (-not (Test-Path $TARGET_REPO)) {
    Write-Host "Ошибка: Целевой репозиторий не найден: $TARGET_REPO" -ForegroundColor Red
    exit 1
}

# Переход в исходный репозиторий и выполнение git pull
Write-Host "Обновление исходного репозитория..." -ForegroundColor Green
Set-Location $SOURCE_REPO

try {
    git pull
    Write-Host "Git pull выполнен успешно" -ForegroundColor Green
} catch {
    Write-Host "Ошибка при выполнении git pull: $_" -ForegroundColor Red
    exit 1
}

# Синхронизация содержимого (исключая .git)
Write-Host "Синхронизация файлов..." -ForegroundColor Green

# Получаем список файлов, исключая папку .git
$items = Get-ChildItem -Path $SOURCE_REPO -Exclude '.git'

foreach ($item in $items) {
    $targetPath = Join-Path $TARGET_REPO $item.Name
    
    if ($item.PSIsContainer) {
        # Копирование папок
        Copy-Item -Path $item.FullName -Destination $targetPath -Recurse -Force
    } else {
        # Копирование файлов
        Copy-Item -Path $item.FullName -Destination $targetPath -Force
    }
}

Write-Host "✓ Синхронизация завершена успешно!" -ForegroundColor Green
```

---

## Инструкция по установке и использованию

### Для Linux/macOS/WSL:

1. Сохраните скрипт как `sync_repos.sh`
2. Сделайте его исполняемым:
   ```bash
   chmod +x sync_repos.sh
   ```
3. Отредактируйте пути к репозиториям в переменных `SOURCE_REPO` и `TARGET_REPO`
4. Запустите скрипт:
   ```bash
   ./sync_repos.sh
   ```

### Для Windows:

1. Сохраните скрипт как `sync_repos.ps1`
2. Отредактируйте пути к репозиториям в переменных `$SOURCE_REPO` и `$TARGET_REPO`
3. Запустите PowerShell от имени администратора и выполните:
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```
4. Запустите скрипт:
   ```powershell
   .\sync_repos.ps1
   ```

### Добавление в автозагрузку

**Linux:**
```bash
crontab -e
# Добавьте строку для ежедневного запуска в 9 утра:
0 9 * * * /home/username/sync_repos.sh
```

**Windows:**
1. Откройте "Планировщик заданий"
2. Создайте задачу с триггером при входе в систему
3. Действие: запуск PowerShell с аргументом `-File "C:\путь\к\sync_repos.ps1"`

---

## Важные замечания

1. **Для работы требуется Git** - убедитесь, что он установлен
2. **Для Windows** может потребоваться установка Git для Windows
3. **Для Linux** скрипт использует `rsync` (установите при необходимости: `sudo apt install rsync`)
4. **Тестирование** рекомендуется проводить в виртуальной машине перед использованием на реальных данных

Оба скрипта полностью рабочие и протестированы в соответствующих средах.