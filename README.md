# mephi-session-project-2026

Установка и настройка РЕД ОС 8 (Standard Server, Minimal) для веб-сервера ГИС.
Личный номер: **265205**. Имя хоста: `mephi-2026.domain.local`.

## Выполнение заданий

| Пункт | Что сделано | Команды | Подтверждение |
|---|---|---|---|
| 1.1 | IPv4 по DHCP на `enp0s3` (задано в установщике) | `nmcli -g ipv4.method con show enp0s3` → `auto` | history.out |
| 1.2 | Имя хоста `mephi-2026.domain.local` (задано в установщике) | `hostnamectl` | mephi-screenshot.png |
| 1.3 | Проверка связи | `ping -c 4 8.8.8.8 > ~/ping.out` | ping.out |
| 2.1 | Обновление системы | `dnf -y update` | dnf.out |
| 2.2 | Установка nginx, libcap-ng-utils, policycoreutils-python-utils | `dnf -y install nginx libcap-ng-utils policycoreutils-python-utils` | dnf.out |
| 2.3 | Скачивание и установка tcpdump через rpm | `dnf download tcpdump --destdir /tmp`; `rpm -ivh /tmp/tcpdump-*.x86_64.rpm` | dnf.out, history.out |
| 3.1 | Раздел на весь `/dev/sdb`, ext4 с меткой MEPHI_WEB | `echo ',' \| sfdisk -X gpt /dev/sdb`; `mkfs.ext4 -L MEPHI_WEB /dev/sdb1` | history.out |
| 3.2 | Точка монтирования и автомонтирование по метке | `mkdir /mephi-web`; строка `LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2` в fstab; `mount /mephi-web` | fstab, stat.out |
| 4.1 | Запуск nginx и автозапуск | `systemctl enable --now nginx` | journalctl.out |
| 4.2 | Журнал nginx за текущую загрузку | `journalctl -u nginx -b > ~/journalctl.out` | journalctl.out |
| 5.1 | Группа mephi-dev (5500), user1–3 (UID 5501–5503), группа curators (GID 4444), curator1–2; каталог 2770 с setgid, ACL и ACL по умолчанию для curators только на чтение | `chown root:mephi-dev`; `chmod 2770`; `setfacl -m g:curators:rx`; `setfacl -d -m u::rwx,g::rwx,g:curators:rx,o::-` | passwd, group, stat.out |
| 5.2 | SUID снят, tcpdump получил capabilities, запуск от user1 проверен | `chmod u-s /usr/sbin/tcpdump`; `setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump`; `sudo -u user1 tcpdump -c 4 -n -i enp0s3 icmp` | getcap.out |
| 5.3 | SELinux в режиме Enforcing; корень nginx перенесён в /mephi-web, задан тип httpd_sys_content_t | `sed -i 's\|/usr/share/nginx/html\|/mephi-web\|' /etc/nginx/nginx.conf`; `semanage fcontext -a -t httpd_sys_content_t '/mephi-web(/.*)?'`; `restorecon -Rv /mephi-web` | getenforce.out, stat.out |
| 6.1 | Запрет локального входа для группы curators | `-:(curators):LOCAL` в `/etc/security/access.conf`; `account required pam_access.so` в `/etc/pam.d/login` | history.out |
| 6.2 | Смена пароля каждые 90 дней; минимальная длина 12 | `PASS_MAX_DAYS 90` в login.defs; `chage -M 90` для всех пользователей; `minlen = 12` в pwquality.conf | shadow, pwquality.conf |
| 7.1 | Страница index.html | `echo 'Hello from Student: 265205' > /mephi-web/index.html` | curl.out |
| 7.2 | Проверка веб-сервера | `curl http://localhost/ > ~/curl.out` | curl.out, mephi-screenshot.png |
| 8.1 | Артефакты собраны после перезагрузки системы | `history > ~/history.out` и команды из задания | все файлы репозитория |

## Файлы

| Файл | Что проверяется |
|---|---|
| mephi-screenshot.png | Визуальное подтверждение |
| history.out | Выполненные команды |
| ping.out | Работоспособность сети |
| dnf.out | Управление пакетами |
| stat.out | Права доступа и контекст SELinux |
| journalctl.out | Запуск веб-сервера |
| getcap.out | Настройка привилегий |
| getenforce.out | Режим SELinux |
| curl.out | Результат тестирования |
| fstab | Монтирование файловых систем |
| passwd | Пользователи |
| shadow | Параметры паролей |
| group | Группы |
| pwquality.conf | Требования к паролю |

Все настройки проверены после перезагрузки: SELinux в режиме Enforcing, /dev/sdb1 смонтирован в /mephi-web, nginx запущен и отдаёт страницу.
