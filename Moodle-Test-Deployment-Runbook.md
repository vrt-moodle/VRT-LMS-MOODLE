# Moodle Test Environment Deployment Runbook

Tai lieu ban giao cho tro ly ky thuat/DevOps: lam sach repo, bao mat cau hinh, va trien khai Moodle test tren Linux bang Docker (port `5090`).

## 1) Muc tieu

- Trien khai Moodle test on dinh tren `5090`.
- Repo GitHub khong con file nhay cam (`config.php`, `.env`, `moodledata`, dump DB, log local).
- Secret cu coi nhu da lo, phai rotate truoc khi deploy.

## 2) Pham vi

- Repo + GitHub hygiene
- PostgreSQL account va network policy
- Cau hinh Moodle (`config.php`)
- Docker image/container
- Kiem tra DB connect, install, cron, checklist nghiem thu

## 3) Dieu kien tien quyet

- Server Linux co Docker + Docker Compose plugin:
  - `docker --version`
  - `docker compose version`
- Co quyen SSH vao server test
- Da xac dinh DB host/port/name
- Co quyen sua repo + push GitHub
- Co domain/IP truy cap env test (neu dung domain phai tro den `IP:5090`)

## 4) Bat buoc bao mat truoc khi trien khai

- Secret DB trong `config.php` cu da lo -> **bat buoc rotate credential**.
- Khong dung user `postgres` cho Moodle.
- Tao user rieng cho app (vi du `moodle_app`) va cap quyen day du.

## 5) Chuan hoa repo local

Thuc hien tren may local.

### 5.1 `.gitignore`

Dam bao file goc `/.gitignore` da co cac nhom sau:

- `/moodledata/`
- `/vrt-lms-moodle/config.php`
- `/vrt-lms-moodle/config.local.php`
- `.env`, `.env.*`, `!.env.example`
- `*.sql`, `*.sql.gz`, `*.dump`, `*.dump.gz`, `*.backup`, `*.psql`
- `*.log`, `*.tmp`, `*.bak`, `*.old`
- `.idea/`, `.vscode/`

### 5.2 Bo file nhay cam khoi Git tracking

```bash
git rm --cached vrt-lms-moodle/config.php
git rm -r --cached moodledata
git add .gitignore
git commit -m "cleanup: ignore local secrets and runtime data"
git push
```

Ghi chu: `--cached` chi bo tracking, khong xoa file local.

### 5.3 Tao file mau cau hinh

Tao `vrt-lms-moodle/config.php.example` (da co trong repo nay) de team copy va dien bien moi truong.

## 6) PostgreSQL hardening (khuyen nghi toi thieu)

```sql
CREATE USER moodle_app WITH PASSWORD 'REPLACE_WITH_NEW_PASSWORD';
GRANT CONNECT ON DATABASE "Moodel_PHP_First" TO moodle_app;
GRANT CREATE ON DATABASE "Moodel_PHP_First" TO moodle_app;
\c Moodel_PHP_First
GRANT USAGE, CREATE ON SCHEMA public TO moodle_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO moodle_app;
GRANT USAGE, SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA public TO moodle_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO moodle_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT, UPDATE ON SEQUENCES TO moodle_app;
```

Network:
- Allow DB from IP cua Linux server
- Mo firewall/security-group dung port DB
- Co rule phu hop trong `pg_hba.conf`

## 7) Trien khai tren server Linux

### 7.1 Tao thu muc deploy

```bash
sudo mkdir -p /opt/vtr-lms-moodle-test/moodledata
sudo chown -R <server-user>:<server-user> /opt/vtr-lms-moodle-test
```

### 7.2 Clone source

```bash
cd /opt/vtr-lms-moodle-test
git clone <GIT_REPO_URL> .
```

### 7.3 Tao `config.php` that tren server

Copy tu `vrt-lms-moodle/config.php.example` -> `vrt-lms-moodle/config.php`, dien gia tri that.

**Quan trong:** `wwwroot` phai la domain/IP that + `:5090`, khong de `localhost` neu truy cap tu may khac.

### 7.4 Permission cho moodledata

```bash
sudo chown -R 33:33 /opt/vtr-lms-moodle-test/moodledata
sudo chmod -R 770 /opt/vtr-lms-moodle-test/moodledata
```

## 8) Docker layout trong repo hien tai

Repo nay dang dung file trong thu muc `docker/`:

- `docker/Dockerfile`
- `docker/docker-compose.yml`

Chay lenh tu `docker/`.

## 9) Build va run

```bash
cd /opt/vtr-lms-moodle-test/docker
docker compose up -d --build
docker compose ps
docker compose logs -f moodle-web
```

## 10) Kiem tra sau khi len container

### 10.1 HTTP port 5090

```bash
ss -ltnp | rg 5090
curl -I http://127.0.0.1:5090
curl -I http://<DOMAIN_OR_IP>:5090
```

### 10.2 PHP extension PostgreSQL

```bash
docker exec -it moodle-web php -m | rg "pgsql|pdo_pgsql"
```

### 10.3 Test DB tu trong container

```bash
docker exec moodle-web php -r '$c=pg_connect("host=DB_HOST port=DB_PORT dbname=DB_NAME user=DB_USER password=DB_PASS"); echo $c ? "OK\n" : "FAIL\n";'
```

## 11) Neu DB moi/trong: install Moodle bang CLI

Chi chay khi DB chua co schema Moodle.

```bash
docker exec -it moodle-web php /var/www/html/admin/cli/install.php \
  --chmod=2770 \
  --lang=en \
  --wwwroot=http://<DOMAIN_OR_IP>:5090 \
  --dataroot=/var/www/moodledata \
  --dbtype=pgsql \
  --dbhost=<DB_HOST> \
  --dbport=<DB_PORT> \
  --dbname=<DB_NAME> \
  --dbuser=<DB_USER> \
  --dbpass='<DB_PASS>' \
  --fullname="VTR LMS Test" \
  --shortname="VTRLMS" \
  --adminuser=admin \
  --adminpass='AdminPasswordStrong123!' \
  --adminemail='admin@example.com' \
  --non-interactive \
  --agree-license
```

## 12) Cron - chon 1 trong 2 cach (khong dung cung luc)

### Cach A (khuyen nghi voi repo nay): container `moodle-cron`

- Giu service `moodle-cron` trong compose
- Chay cung `moodle-web`
- Khong can `crontab` host

### Cach B: host crontab

Neu khong dung `moodle-cron` container:

```bash
* * * * * docker exec -u www-data moodle-web php /var/www/html/admin/cli/cron.php >/dev/null 2>&1
```

## 13) Firewall

```bash
sudo ufw allow 5090/tcp
sudo ufw status
```

Neu dung cloud firewall/security-group, mo inbound TCP/5090.

## 14) Checklist nghiem thu

- HTTP: vao duoc `http://domain-or-ip:5090`
- Login admin thanh cong
- Vao duoc Site administration
- Tao user/course duoc
- Upload file duoc
- Kiem tra DB connect OK
- Cron chay khong loi
- Repo khong con track `config.php`, `.env`, `moodledata/`
- App dung user `moodle_app`, khong dung `postgres`

## 15) Loi thuong gap

- Redirect/login loi: `wwwroot` sai (dang `localhost` hoac sai domain)
- DB fail: sai host/port/user/pass, firewall, `pg_hba.conf`
- Upload fail: `moodledata` khong writable
- Chuc nang khong day du: cron chua chay
- Trang trang/500: thieu extension hoac loi permission (xem `docker compose logs`)

## 16) Thu tu thao tac de giao viec

1. Chuan hoa `.gitignore`
2. Bo `config.php` + `moodledata` khoi tracking
3. Them `config.php.example`, push repo sach
4. Rotate credential DB, tao `moodle_app`
5. SSH server, tao folder deploy
6. Clone repo
7. Tao `config.php` that tren server
8. Build + run docker
9. Test HTTP/DB
10. DB moi thi install CLI
11. Cau hinh cron (1 cach duy nhat)
12. Run checklist va ban giao

---

## Ghi chu van hanh

- Khong commit `config.php` that len GitHub.
- Neu secret tung xuat hien trong lich su Git, can xu ly lich su repo theo quy trinh bao mat cua team.
- Ban runbook nay uu tien su on dinh local/test tren port `5090` va phu hop cau truc repo hien tai.
