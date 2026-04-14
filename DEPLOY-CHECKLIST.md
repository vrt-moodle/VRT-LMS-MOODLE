# Deploy Checklist - Moodle Test (Port 5090)

Checklist thao tac nhanh de trien khai va nghiem thu env test.

## A. Pre-deploy (Repo + Security)

- [ ] `.gitignore` da dung chuan (ignore `config.php`, `.env`, `moodledata`, dump/log).
- [ ] `vrt-lms-moodle/config.php` khong con duoc track tren Git.
- [ ] Co `vrt-lms-moodle/config.php.example` trong repo.
- [ ] Secret cu da rotate, dang dung credential moi.
- [ ] DB app user la `moodle_app` (hoac user rieng), khong dung `postgres`.

## B. Server readiness

- [ ] SSH vao server thanh cong.
- [ ] `docker --version` va `docker compose version` OK.
- [ ] Da tao `/opt/vtr-lms-moodle-test` va `/opt/vtr-lms-moodle-test/moodledata`.
- [ ] Clone repo ve dung thu muc deploy.
- [ ] `moodledata` da set permission (`33:33`, `770`).

## C. Runtime config

- [ ] Tao `vrt-lms-moodle/config.php` tren server tu file example.
- [ ] DB host/port/name/user/pass dung moi truong test.
- [ ] `$CFG->wwwroot` dung `http://<domain-or-ip>:5090`.
- [ ] `$CFG->dataroot` la `/var/www/moodledata`.

## D. Docker up

- [ ] Chay tu thu muc `docker/`.
- [ ] `docker compose up -d --build` thanh cong.
- [ ] `docker compose ps` thay `moodle-web` (va `moodle-cron` neu su dung).
- [ ] `docker compose logs -f moodle-web` khong co loi fatal.

## E. Connectivity tests

- [ ] `curl -I http://127.0.0.1:5090` tra ve HTTP hop le.
- [ ] Truy cap duoc `http://<domain-or-ip>:5090` tu ben ngoai.
- [ ] Tu container, `php -m` co `pgsql` va `pdo_pgsql`.
- [ ] Test `pg_connect()` tu container tra ve `OK`.

## F. Install/init

- [ ] Neu DB moi/trong: da chay `admin/cli/install.php`.
- [ ] Neu DB da co schema Moodle: bo qua install, chi verify app.

## G. Cron strategy (chon 1)

- [ ] Cach A: su dung service `moodle-cron` trong compose.
- [ ] Cach B: su dung host crontab `docker exec ... cron.php`.
- [ ] Khong dung ca 2 cach cung luc.

## H. Acceptance

- [ ] Login admin thanh cong.
- [ ] Vao duoc Site administration.
- [ ] Tao user, tao course thanh cong.
- [ ] Upload file thanh cong.
- [ ] Cron chay khong loi DB.
- [ ] Repo khong chua secret/runtime files.

## I. Handover metadata

- [ ] URL test:
- [ ] Commit deploy:
- [ ] DB host/port:
- [ ] App DB user:
- [ ] Cron mode (A/B):
- [ ] Nguoi trien khai:
- [ ] Thoi gian:
- [ ] Ghi chu:
