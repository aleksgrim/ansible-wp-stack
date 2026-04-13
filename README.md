# ansible-wp — автодеплой WordPress сайтов

## Структура

```
ansible-wp/
├── inventory/
│   └── hosts.yml              ← IP сервера и SSH данные
├── group_vars/
│   └── all.yml                ← глобальные переменные (php версия и т.д.)
├── roles/
│   ├── system_user/           ← создаёт Linux юзера + папки
│   ├── sftp_chroot/           ← добавляет блок в sshd_config
│   ├── php_fpm_pool/          ← создаёт PHP-FPM пул
│   ├── nginx_vhost/           ← создаёт Nginx vhost
│   ├── mysql_db/              ← создаёт БД и MySQL юзера
│   └── wordpress/             ← скачивает WP, права, wp-config.php
├── playbooks/
│   └── new_site.yml           ← главный плейбук
└── credentials/               ← сюда сохраняются пароли (gitignore!)
```

## Установка зависимостей

```bash
pip install ansible --break-system-packages
ansible-galaxy collection install community.mysql
```

## Настройка

1. Скопируй пример inventory:
```bash
cp inventory/hosts-example.yml inventory/hosts.yml
```
2. Заполни `inventory/hosts.yml` своими данными
3. Убедись что `inventory/hosts.yml` не попал в git — он в `.gitignore`


## Запуск нового сайта

```bash
ansible-playbook -i inventory/hosts.yml playbooks/new_site.yml -e "domain=mysite.com"
```

## Что создаётся автоматически из домена

| Из домена `mysite.com` | Результат |
|---|---|
| Linux юзер | `mysite---admin` |
| SFTP chroot | `/var/www/mysite---admin` |
| Webroot | `/var/www/mysite---admin/mysite.com/public` |
| PHP-FPM пул | `/etc/php/8.5/fpm/pool.d/mysite---admin.conf` |
| PHP сокет | `/run/php/php8.5-fpm-mysite.sock` |
| Nginx vhost | `/etc/nginx/sites-available/mysite.com` |
| MySQL БД | `mysite_com` |
| MySQL юзер | `mysite_com` |
| SFTP пароль | генерируется случайно |
| DB пароль | генерируется случайно |

## Credentials

После деплоя пароли сохраняются в `credentials/<domain>.txt`

```
=== mysite.com ===
SFTP:
  Host:     192.168.62.136
  Port:     2044
  User:     mysite---admin
  Password: Xk9#mP...

MySQL:
  DB:       mysite_com
  User:     mysite_com
  Password: zR7$nQ...
```

⚠️ Добавь `credentials/` в `.gitignore`!

## Удаление сайта

```bash
ansible-playbook -i inventory/hosts.yml playbooks/remove_site.yml -e "domain=mysite.com"
```
