# ansible-wp — WordPress Auto-Deployment Stack

## Structure

```
ansible-wp/
├── inventory/
│   └── hosts.yml              ← Server IP and SSH credentials
├── group_vars/
│   └── all.yml                ← Global variables (PHP version, etc.)
├── roles/
│   ├── system_user/           ← Creates Linux user and directories
│   ├── sftp_chroot/           ← Adds SFTP chroot block to sshd_config
│   ├── php_fpm_pool/          ← Creates PHP-FPM pool
│   ├── nginx_vhost/           ← Creates Nginx vhost
│   ├── mysql_db/              ← Creates Database and MySQL user
│   └── wordpress/             ← Downloads WP, sets permissions, wp-config.php
├── playbooks/
│   └── new_site.yml           ← Main deployment playbook
└── credentials/               ← Local storage for generated passwords (gitignore!)
```

## Prerequisites

```bash
pip install ansible
ansible-galaxy collection install community.mysql
```

## Setup

1. Copy the example inventory:
```bash
cp inventory/hosts-example.yml inventory/hosts.yml
```
2. Fill `inventory/hosts.yml` with your server data.
3. Ensure `inventory/hosts.yml` is ignored by git (it is in `.gitignore` by default).

## Deploying a New Site

```bash
ansible-playbook -i inventory/hosts.yml playbooks/new_site.yml -e "domain=mysite.com"
```

## Automatic Generation Details

| From domain `mysite.com` | Resulting value |
|---|---|
| Linux User | `mysite---admin` |
| SFTP Chroot | `/var/www/mysite---admin` |
| Webroot | `/var/www/mysite---admin/mysite.com/public` |
| PHP-FPM Pool | `/etc/php/8.5/fpm/pool.d/mysite---admin.conf` |
| PHP Socket | `/run/php/php8.5-fpm-mysite.sock` |
| Nginx Vhost | `/etc/nginx/sites-available/mysite.com` |
| MySQL DB | `mysite_com` |
| MySQL User | `mysite_com` |
| SFTP Password | Randomly generated |
| DB Password | Randomly generated |

## Credentials

After deployment, credentials are saved to `credentials/<domain>/credentials.txt`.

```
=== mysite.com ===
SFTP:
  Host:     YOUR_SERVER_IP
  Port:     22
  User:     mysite---admin
  Password: [GENERATED_PASSWORD]

MySQL:
  DB:       mysite_com
  User:     mysite_com
  Password: [GENERATED_PASSWORD]
```

⚠️ Remember to keep the `credentials/` folder in `.gitignore`!

## Removing a Site

```bash
ansible-playbook -i inventory/hosts.yml playbooks/remove_site.yml -e "domain=mysite.com"
```
