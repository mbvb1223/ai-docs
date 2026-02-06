# Laravel 12.x Homestead Documentation

## Introduction

Laravel Homestead is a legacy pre-packaged Vagrant box providing a development environment without installing PHP, web server, or other software locally. **Laravel Sail** is recommended as a modern alternative.

## Included Software

- Ubuntu 22.04
- Nginx
- PHP 8.3, 8.2, 8.1, 8.0, 7.4, 7.3, 7.2, 7.1, 7.0, 5.6
- MySQL 8.0, PostgreSQL 15
- Redis, Memcached
- Node, Yarn, Bower, Grunt, Gulp
- Composer, Docker, Git

## Installation

```bash
git clone https://github.com/laravel/homestead.git ~/Homestead
cd ~/Homestead
git checkout release
bash init.sh
```

## Configuration

### Homestead.yaml

```yaml
provider: virtualbox

folders:
    - map: ~/code/project1
      to: /home/vagrant/project1

sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

### Update hosts file

```
192.168.56.56  homestead.test
```

## Launching

```bash
vagrant up
vagrant destroy
```

## Daily Usage

### SSH Access

```bash
vagrant ssh
```

### Adding Sites

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

Re-provision:
```bash
vagrant reload --provision
```

### PHP Versions Per Site

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "8.1"
```

### Database Connections

From host machine:
- MySQL: `127.0.0.1:33060` (user: `homestead`, password: `secret`)
- PostgreSQL: `127.0.0.1:54320`

### Database Backups

```yaml
backup: true
```

### Cron Scheduling

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

### Mailpit

Access at `http://localhost:8025`

```env
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
```

## Optional Features

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
    - elasticsearch:
        version: 7.9.0
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - rabbitmq: true
    - webdriver: true
```

## Debugging

### Xdebug

```bash
sudo phpenmod xdebug
sudo phpdismod xdebug
```

### CLI Debugging

```bash
xphp /path/to/script
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/homestead)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
