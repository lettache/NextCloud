# Создаём и настроаиваем ВМ на рабочем хосте 

## Гипервизор - VMware Workstation Pro v25.0.0.24995812

**VM_work.host — Ubuntu 26.04 server, RAM 8 GB, SSDHard disk (SCSI) 30 GB**

**Подключение к VM по SSH**
`ssh-copy-id <user>@*.*.*.*`

`ssh <user>@*.*.*.*`

**Настройка ВМ**

`sudo apt update && sudo apt upgrade -y`

`sudo timedatectl set-timezone Europe/Moscow`

**Установка docker**

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker vlk
```

**Добавляем зеркало**

`{ "registry-mirrors": ["https://dockerhub.timeweb.cloud", "https://mirror.gcr.io"] }`

`sudo systemctl restart docker`

**Подгатавливаем файлы для docker**

`mkdir -p ~/NextCloud/compose/web`

`curl -fsSL https://raw.githubusercontent.com/nextcloud/docker/master/.examples/docker-compose/insecure/postgres/fpm/web/nginx.conf -o web/nginx.conf`

`sudo nano .env`

`sudo nano compose.yaml`

``` yaml
services:
  db:                                # база данных
    image: postgres:16-alpine        # образ: PostgreSQL 16, alpine
    restart: always                  # restart после сбоя/ребута ВМ
    volumes:
      - db:/var/lib/postgresql       # том "db" = диск базы
    env_file:
      - .env                         # Переменные, настройки пароли

  redis:                             # кэш блокировок
    image: redis:7-alpine
    restart: always

  app:                               # Nextcloud: PHP-код + движок PHP-FPM
    image: nextcloud:35-fpm-alpine
    restart: always
    volumes:
      - nextcloud:/var/www/html      # том с кодом, конфигом и локальными файлами
    environment:
      - POSTGRES_HOST=db             
      - REDIS_HOST=redis
    env_file:
      - .env
    depends_on:
      - db                           # запуск после db и redis
      - redis

  web:                               # nginx
    image: nginx:alpine-slim
    restart: always
    ports:
      - "8080:80"                    # порт 8080 на ВМ → порт 80 в контейнере

    volumes:
      - ./web/nginx.conf:/etc/nginx/nginx.conf:ro   # конфиг
      - nextcloud:/var/www/html:ro   
    depends_on:
      - app

  cron:                              # фоновые задачи Nextcloud
    image: nextcloud:35-fpm-alpine   
    restart: always
    volumes:
      - nextcloud:/var/www/html      
    entrypoint: /cron.sh             
    depends_on:
      - db
      - redis

volumes:
  db:                                # объявление именованных "дисков"
  nextcloud:
  ```

**Запуск compose**

`docker compose up -d`

**На хостовой WSL создаем отдельный бакет**

`aws --profile s3_cloudru --endpoint-url https://s3.cloud.ru s3 mb s3://bucket-nextcloud-media-stage`

**На ВМ включаем приложение внешних хранилищ**

`docker compose exec -u www-data app php occ app:enable files_external`

> Для запуска плэйбука необходима интерактивная аунтификация
Варинаты решения:  
- 
```bash
sudo visudo -f /etc/sudoers.d/vlk
vlk ALL=(ALL) NOPASSWD: ALL
```

- `ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/site.yml --ask-become-pass`
