# ansible-wp-stack

Automated WordPress stack provisioning with Ansible — Nginx, PHP-FPM, MySQL, SFTP isolation per site.

## Server Requirements

Before running the playbook, the server must meet these requirements:

### OS
- Ubuntu 22.04 / 24.04 LTS

### SSH access
- SSH key added to server for your sudo user (e.g. `alex3`)
- SSH key must NOT be password-protected (or added to `ssh-agent` before running)
- `sudo` configured without password for your user:
```bash
  printf 'USERNAME ALL=(ALL) NOPASSWD: ALL\n' | sudo tee /etc/sudoers.d/USERNAME
  sudo chmod 440 /etc/sudoers.d/USERNAME
```

### Required packages on server
- `python3` (for Ansible modules)
- `python3-pymysql` (for MySQL Ansible module) — installed automatically by playbook

### Local machine requirements
- Ansible `>= 2.15`
- Python package `passlib`
- Ansible collection `community.mysql`

Install with:
```bash
pip install ansible passlib
ansible-galaxy collection install community.mysql
```

## Setup

1. Copy example inventory:
```bash
cp inventory/hosts-example.yml inventory/hosts.yml
```

2. Fill in `inventory/hosts.yml` with your server details:
```yaml
ansible_host: YOUR_SERVER_IP
ansible_user: YOUR_SSH_USER
ansible_port: YOUR_SSH_PORT
ansible_ssh_private_key_file: ~/.ssh/YOUR_KEY
```

3. Make sure `inventory/hosts.yml` is not committed — it's in `.gitignore`

## Usage

### Deploy new WordPress site
```bash
ansible-playbook -i inventory/hosts.yml playbooks/new_site.yml -e "domain=mysite.com"
```

### Remove a site
```bash
ansible-playbook -i inventory/hosts.yml playbooks/remove_site.yml -e "domain=mysite.com"
```

## What gets created automatically from domain name

| From domain `mysite.com` | Result |
|---|---|
| Linux user | `mysite---admin` |
| SFTP chroot | `/var/www/mysite---admin` |
| Webroot | `/var/www/mysite---admin/mysite.com/public` |
| PHP-FPM pool | `/etc/php/8.5/fpm/pool.d/mysite---admin.conf` |
| PHP socket | `/run/php/php8.5-fpm-mysite.sock` |
| Nginx vhost | `/etc/nginx/sites-available/mysite.com` |
| MySQL DB | `mysite_com` |
| MySQL user | `mysite_com` |
| SFTP password | auto-generated, stable |
| DB password | auto-generated, stable |

## Credentials

After deployment, credentials are saved to `credentials/<domain>/credentials.txt`

### SFTP:
- Host:     YOUR_SERVER_IP
- Port:     2044
- User:     mysite---admin
- Password: ...

###MySQL:
- DB:       mysite_com
- User:     mysite_com
- Password: ...

⚠️ `credentials/` is in `.gitignore` — never commit it!

## Stack

- **Nginx** + FastCGI cache
- **PHP 8.5** (Ondřej Surý PPA) + OPcache
- **MySQL** — isolated DB per site
- **SFTP** — chroot per site, password auth for `*---admin` users only
- **WordPress** — latest, auto-configured

