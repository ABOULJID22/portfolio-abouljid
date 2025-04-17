
```markdown
 🚀 Guide de Déploiement : Laravel + React + Breeze + Filament v3 avec Docker (Laravel Sail)

Ce guide complet vous accompagne étape par étape dans la création d'une application moderne avec Laravel**, React, Filament v3, et Docker via Laravel Sail.

---

### ✅ Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :

- [Visual Studio Code](https://code.visualstudio.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- Docker Compose v2+
- [Git](https://git-scm.com/) & [Composer](https://getcomposer.org/)
- [Node.js ≥ 18](https://nodejs.org/) et npm ≥ 9
- Ubuntu avec WSL2 (pour Windows) :

```bash
wsl --install -d Ubuntu
```

---

## 🧱 Installation des dépendances sous Ubuntu (WSL)

### 1. Installer PHP 8.4, Composer, et Laravel Installer :

```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
composer global require laravel/installer
source ~/.bashrc
```

---

## 🚀 Création du projet Laravel avec Sail

```bash
laravel new nom-projet
cd nom-projet
code .
```

### 2. Installer Laravel Sail :

```bash
composer require laravel/sail --dev
php artisan sail:install
php artisan sail:publish
```

---

## 🐳 Démarrer le projet avec Docker

```bash
./vendor/bin/sail up -d
```

---

## ⚙️ Configurer le fichier `.env`

Vérifiez et modifiez les variables d’environnement importantes :
- `DB_DATABASE=nom_projet`
- `DB_USERNAME=sail`
- `DB_PASSWORD=password`
- `APP_PORT=80`
- `VITE_PORT=5173`

---

## ⚙️ Configurer le fichier `docker-compose.yml`

Copiez et collez ce contenu dans le fichier `docker-compose.yml` à la racine du projet :

```yaml
version: '3'
services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.3
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.3/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mysql
            - redis
            - meilisearch
            - mailpit
            - selenium
    mysql:
        image: 'mysql/mysql-server:8.0'
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ROOT_HOST: '%'
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 1
        volumes:
            - 'sail-mysql:/var/lib/mysql'
            - './docker/mysql/create-testing-database.sh:/docker-entrypoint-initdb.d/10-create-testing-database.sh'
        networks:
            - sail
        healthcheck:
            test: ['CMD', 'mysqladmin', 'ping', '-p${DB_PASSWORD}']
            retries: 3
            timeout: 5s
    redis:
        image: 'redis:alpine'
        ports:
            - '${FORWARD_REDIS_PORT:-6379}:6379'
        volumes:
            - 'sail-redis:/data'
        networks:
            - sail
        healthcheck:
            test: ['CMD', 'redis-cli', 'ping']
            retries: 3
            timeout: 5s
    meilisearch:
        image: 'getmeili/meilisearch:latest'
        ports:
            - '${FORWARD_MEILISEARCH_PORT:-7700}:7700'
        volumes:
            - 'sail-meilisearch:/meili_data'
        networks:
            - sail
        healthcheck:
            test: ['CMD', 'wget', '--no-verbose', '--spider', 'http://localhost:7700/health']
            retries: 3
            timeout: 5s
    mailpit:
        image: 'axllent/mailpit:latest'
        ports:
            - '${FORWARD_MAILPIT_PORT:-1025}:1025'
            - '${FORWARD_MAILPIT_DASHBOARD_PORT:-8025}:8025'
        networks:
            - sail
    selenium:
        image: selenium/standalone-chrome
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        volumes:
            - '/dev/shm:/dev/shm'
        networks:
            - sail
networks:
    sail:
        driver: bridge
volumes:
    sail-mysql:
        driver: local
    sail-redis:
        driver: local
    sail-meilisearch:
        driver: local
```

---

## 🔧 Installer Breeze + React

```bash
./vendor/bin/sail composer require laravel/breeze --dev
./vendor/bin/sail artisan breeze:install react
npm install
npm run build
```

---

## ✨ Installer Filament v3

```bash
./vendor/bin/sail composer require filament/filament:"^3.3" -W
./vendor/bin/sail artisan filament:install --panels
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan make:filament-user
```

---

## 🧪 Tester la connexion à la base de données

```bash
./vendor/bin/sail mysql
```

Puis dans le terminal MySQL :

```sql
SHOW DATABASES;
USE nom_projet;
SHOW TABLES;
```

---

## ✅ Félicitations !

Votre projet **Laravel + React + Filament v3 + Docker** est prêt ! Vous pouvez maintenant commencer le développement de votre application.

---

## 📁 Structure recommandée

```
nom-projet/
│
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
│   ├── js/     # React components
│   └── views/  # Blade templates
├── routes/
├── storage/
├── tests/
└── docker-compose.yml
```

---

> Réalisé avec ❤️ par abouljid
```
