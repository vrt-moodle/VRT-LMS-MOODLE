# Security Rotation Checklist (Moodle Test)

Muc tieu: dam bao moi secret tung lo trong source/lich su thao tac duoc rotate truoc khi ban giao env test.

## 1) Kich hoat quy trinh incident nho

- [ ] Danh dau secret cu "compromised".
- [ ] Ngung su dung ngay user/password cu tren moi moi truong co lien quan.
- [ ] Tao ticket noi bo theo doi rotation + nguoi phu trach + thoi gian.

## 2) Rotate PostgreSQL credential

- [ ] Tao user app moi (khuyen nghi): `moodle_app`.
- [ ] Dat password moi manh, random, dai >= 24 ky tu.
- [ ] Cap quyen toi thieu dung cho Moodle:
  - [ ] `CONNECT`, `CREATE` tren database
  - [ ] `USAGE`, `CREATE` tren schema `public`
  - [ ] DML tren bang hien tai + default privileges cho bang/sequence moi
- [ ] Thu hoi quyen khong can thiet cua user cu.
- [ ] (Neu can) khoa/xoa user cu sau khi xac nhan app chay on.

## 3) Rotate secret trong he thong lien quan

- [ ] Cap nhat `vrt-lms-moodle/config.php` tren server bang credential moi.
- [ ] Cap nhat secret tai CI/CD (neu co).
- [ ] Cap nhat password manager/noi luu tru secret trung tam.
- [ ] Khong luu secret trong file commit len GitHub.

## 4) Kiem tra khong con secret trong repo

- [ ] Xac nhan `.gitignore` da ignore `config.php`, `.env`, `moodledata`, dump DB.
- [ ] Xac nhan file local nhay cam khong con bi Git track:
  - [ ] `git ls-files | rg "config.php|\\.env|moodledata"`
- [ ] Quet nhanh keyword nhay cam:
  - [ ] `rg -n "dbpass|password|postgres://|PGPASSWORD" .`

## 5) Xu ly lich su Git (neu secret tung commit)

- [ ] Danh gia muc do can thiet rewrite history.
- [ ] Neu rewrite, thong bao team ve force-push + cach sync lai clone.
- [ ] Sau rewrite van phai rotate secret (khong thay the rotation).

## 6) Xac nhan sau rotation

- [ ] Tu container `moodle-web`, test ket noi DB: `OK`.
- [ ] Moodle login/CRUD co ban hoat dong.
- [ ] Cron chay khong loi DB.
- [ ] Log khong con thong tin secret in plain text.

## 7) Bang ket qua ban giao

- [ ] Rotation hoan tat (Yes/No)
- [ ] Thoi diem hoan tat:
- [ ] Nguoi thuc hien:
- [ ] Nguoi xac nhan:
- [ ] Ghi chu:
