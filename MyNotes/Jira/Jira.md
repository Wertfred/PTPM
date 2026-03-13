### Инструкция по установке Jira в Docker

#### 1. Подготовка системы
Убедитесь, что установлен Docker:
```bash
docker --version
```

#### 2. Создание сети для Jira (рекомендуется)
```bash
docker network create jira-network
```

#### 3. Запуск контейнера с базой данных (MySQL)
Jira требует базу данных для хранения информации. Запустите MySQL:

```bash
# Создать папку для данных MySQL
mkdir mysql_data

# Запустить MySQL
docker run -d \
  --name mysql-jira \
  --network jira-network \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=jiradb \
  -e MYSQL_USER=jirauser \
  -e MYSQL_PASSWORD=jirapassword \
  -v ./mysql_data:/var/lib/mysql \
  mysql:8.0 \
  --character-set-server=utf8mb4 \
  --collation-server=utf8mb4_bin
```

**Пояснение параметров:**
- `-e MYSQL_ROOT_PASSWORD` — пароль для root
- `-e MYSQL_DATABASE` — автоматическое создание базы данных jiradb
- `-e MYSQL_USER` и `MYSQL_PASSWORD` — создание пользователя для Jira
- `--character-set-server` и `--collation-server` — обязательная кодировка для Jira 

#### 4. Запуск контейнера Jira

**Вариант А: Официальный образ Atlassian (требуется учетная запись):**
```bash
# Создать папки для данных Jira
mkdir jira_home

# Запустить Jira
docker run -d \
  --name jira \
  --network jira-network \
  -p 8080:8080 \
  -v ./jira_home:/var/atlassian/application-data/jira \
  atlassian/jira-software:latest
```

**Вариант Б: Альтернативный образ (для тестирования):**
```bash
# Создать папки для данных Jira
mkdir jira_home

# Запустить Jira
docker run -d \
  --name jira \
  --network jira-network \
  -p 8080:8080 \
  -v ./jira_home:/var/atlassian/application-data/jira \
  cptactionhank/atlassian-jira-software:8.1.0
```

**Пояснение параметров:**
- `-p 8080:8080` — проброс порта для веб-интерфейса
- `-v ./jira_home:/var/atlassian/application-data/jira` — монтирование папки для сохранения данных Jira 

#### 5. Проверка установки
```bash
# Проверить статус контейнеров
docker ps

# Посмотреть логи Jira (подождите 1-2 минуты для полного запуска)
docker logs -f jira
```

#### 6. Доступ к веб-интерфейсу
Откройте браузер и перейдите по адресу:
```
http://localhost:8080
```

#### 7. Настройка Jira через веб-интерфейс

**Шаг 1: Выбор языка**
Нажмите "Language" в правом верхнем углу и выберите русский или английский 

**Шаг 2: Способ настройки**
Выберите "I'll set it up myself" (Настрою самостоятельно) 

**Шаг 3: Подключение к базе данных**
Заполните параметры подключения к MySQL:
- **Database Type**: MySQL 8.0
- **Hostname**: mysql-jira (имя контейнера MySQL)
- **Port**: 3306
- **Database**: jiradb
- **Username**: jirauser
- **Password**: jirapassword 

**Шаг 4: Настройка Jira**
Укажите название приложения, режим доступа и базовый URL

**Шаг 5: Лицензия**
Введите лицензионный ключ или получите 30-дневную пробную лицензию на сайте Atlassian 

**Шаг 6: Создание администратора**
Задайте имя пользователя, пароль и email для администратора Jira

#### 8. Основные команды управления
```bash
# Остановка контейнеров
docker stop jira mysql-jira

# Запуск остановленных
docker start jira mysql-jira

# Перезапуск
docker restart jira

# Просмотр логов
docker logs -f jira

# Подключиться к контейнеру Jira
docker exec -it jira bash

# Удаление контейнеров (данные сохраняются в папках)
docker rm jira mysql-jira
```

#### 9. Docker Compose (удобный вариант)
Создайте файл `docker-compose.yml`:

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql-jira
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: jiradb
      MYSQL_USER: jirauser
      MYSQL_PASSWORD: jirapassword
    volumes:
      - ./mysql_data:/var/lib/mysql
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
    networks:
      - jira-network

  jira:
    image: atlassian/jira-software:latest
    container_name: jira
    ports:
      - "8080:8080"
    volumes:
      - ./jira_home:/var/atlassian/application-data/jira
    depends_on:
      - mysql
    networks:
      - jira-network

networks:
  jira-network:
    driver: bridge
```

Запуск:
```bash
docker-compose up -d
```

#### 10. Бэкап и восстановление

**Бэкап данных Jira:**
```bash
# Остановить Jira перед бэкапом (рекомендуется)
docker stop jira

# Создать архив с данными
tar -czf jira-backup-$(date +%Y%m%d).tar.gz ./jira_home

# Бэкап базы данных MySQL
docker exec mysql-jira mysqldump -u root -prootpassword jiradb > jiradb-backup.sql

# Запустить Jira обратно
docker start jira
```

**Восстановление:**
```bash
# Остановить контейнеры
docker stop jira mysql-jira

# Восстановить данные Jira
rm -rf ./jira_home/*
tar -xzf jira-backup-20240101.tar.gz -C ./

# Восстановить базу данных
cat jiradb-backup.sql | docker exec -i mysql-jira mysql -u root -prootpassword jiradb

# Запустить контейнеры
docker start mysql-jira jira
```

#### Важно
- Jira — платный продукт, требуется лицензия (доступна 30-дневная пробная версия) 
- Для production используйте внешнюю базу данных и настройте регулярные бэкапы 
- Минимальные требования: 2-4 GB RAM для Jira 
- Ограничьте ресурсы контейнера: `--memory="4g" --cpus="2.0"` 
- Для доступа по HTTPS рекомендуется использовать Nginx как reverse proxy 