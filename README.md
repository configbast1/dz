# ДЗ Docker: образи, теги, прибирання

## 1. Паспорт образів

Завантажені образи: `alpine`, `busybox`, `nginx`, `redis`, `python:3.13-slim`.

| Образ | Тег | Розмір на диску | Для чого потрібен |
|---|---|---|---|
| busybox | latest | 6.81 MB | Набір базових Unix-утиліт (sh, ls, cp, wget…) в одному файлі. Для тестів і дуже малих контейнерів. |
| alpine | latest | 13 MB | Мінімальний дистрибутив Linux (musl + BusyBox). База для легких образів. |
| python | 3.13-slim | 178 MB | Python 3.13 на урізаному Debian. Для запуску Python-застосунків. |
| redis | latest | 213 MB | Сховище даних у пам'яті (ключ–значення): кеш, черги, сесії. |
| nginx | latest | 243 MB | Веб-сервер, reverse proxy, балансувальник навантаження. |

![docker images](screenshots/1_pasport_docker_images.png)

**Висновок.** Найлегший образ — busybox (6.81 MB): у ньому немає повноцінної ОС, лише один файл з утилітами. Alpine теж дуже легкий (13 MB), бо це мінімальний Linux без зайвих програм.
Найважчий — nginx (243 MB). Він побудований на Debian, тобто містить базову систему з бібліотеками і сам веб-сервер. Redis і python-slim теж на Debian, тому вони в 15–20 разів важчі за alpine.
Redis і python мають спільний шар `37fb86c2a767` (під час завантаження він був `Already exists`): Docker не завантажує однакові шари двічі.

## 2. Полювання на версії

| Образ | Тег | Розмір | Чим відрізняється |
|---|---|---|---|
| python | 3.10 | 1.6 GB | Стара версія Python, повний образ (з компіляторами та dev-бібліотеками), тому в 9 разів важчий за 3.13-slim. |
| alpine | 3.19 | 11.6 MB | Конкретний старіший реліз, на відміну від latest не зміниться з часом. |
| ubuntu | 22.04 | 120 MB | Ubuntu 22.04 LTS (Jammy), старіші версії пакетів. |
| ubuntu | 24.04 | 119 MB | Ubuntu 24.04 LTS (Noble), новіші версії пакетів. |
| mariadb | 10 | 459 MB | Остання версія гілки MariaDB 10.x. |
| mariadb | 11 | 470 MB | Остання версія гілки MariaDB 11.x. Побудована на Ubuntu 24.04 (спільний шар `26538842dfbf`). |

![docker pull](screenshots/2_pull_versii.png)

![docker images з версіями](screenshots/3_versii_docker_images.png)

**Коментар.** Тег визначає версію образу. `latest` змінюється з часом, тому в проєктах краще писати конкретний тег, щоб збірка завжди була однаковою. Суфікс `-slim` означає урізаний образ: python:3.10 займає 1.6 GB, а python:3.13-slim лише 178 MB.

## 3. Прибирання

Запуск hello-world:

![hello-world](screenshots/4_hello_world.png)

**До прибирання:** 13 образів, 3.028 GB, можна звільнити 2.615 GB (86%).

![docker system df до](screenshots/5_system_df_do.png)

Видалення трьома способами:
1. За тегом: `docker rmi ubuntu:22.04`
2. За IMAGE ID: `docker rmi 6baf43584bcb` (видалено alpine:3.19)
3. Усі невикористовувані: `docker image prune -a` (звільнено 2.837 GB)

![rmi за тегом](screenshots/6_rmi_po_tegu.png)

![rmi за ID і prune](screenshots/7_rmi_id_i_prune.png)

**Після прибирання:** залишилось 2 образи, 177.4 MB, звільняти більше нічого (0B).

![docker images і docker system df після](screenshots/8_pislia_images_i_df.png)

**Чому hello-world залишився.** `docker image prune -a` видаляє лише ті образи, які не використовує жоден контейнер. Після `docker run hello-world` залишився зупинений контейнер, створений з цього образу (його видно в `docker ps -a`). Поки контейнер існує, Docker не видаляє його образ: позначка `U` (In Use) у `docker images` і ACTIVE = 2 у `docker system df`. З тієї ж причини залишився `httpd:2.4`: його використовує контейнер `mysite2`.
Щоб видалити hello-world, спочатку треба видалити контейнер (`docker rm <id>`), а потім образ.
