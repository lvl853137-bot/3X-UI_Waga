
<!-- Замените путь ниже на вашу обложку, если она называется иначе -->
<img width="1024" height="339" alt="3f68ff0f-bbae-4882-a407-09623a9a0bfe" src="https://github.com/user-attachments/assets/d5b70e47-fbd3-4242-8a63-f19b1a9e8fad" />
<div align="center">


# 🚀 WAGA · Дядя Вова VPN

Собственная система развёртывания и управления VPN на базе **3X-UI v3.9.0 + Xray v26.6.27 + Nginx (SNI 443) + waga-sub**.

## Установка одной командой (Ubuntu 22.04/24.04, Debian 12/13)

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/lvl853137-bot/3X-UI_Waga/main/scripts/waga-install.sh)
```

## Протоколы по умолчанию
- **VLESS REALITY** (`443/tcp`, `xtls-rprx-vision`)
- **Hysteria2** (`443/udp`, masquerade под сайт)
- **VLESS XHTTP + REALITY** (`443/tcp`)
- **AmneziaWG** (`51821/udp`) и **AmneziaWG 3.1** (`51822/udp`)

## Встроенные домены GEMINI
Автоматически прописываются в `xrayTemplateConfig` панели 3X-UI, в шаблон `/etc/waga-sub/rules.yaml` и в браузерный генератор роутеров (`tools/`):
- `geosite:google-gemini`
- `domain:gemini.google.com`
- `domain:aistudio.google.com`
- `domain:generativelanguage.googleapis.com`
- `domain:alkalimining-pa.googleapis.com`
- `domain:proactivebackend-pa.googleapis.com`

## Управление (`waga`)
- `waga` — интерактивное консольное меню
- `waga check --deep` / `waga fix` — диагностика сервера и безопасное автоисправление
- `waga net gemini ipv4|warp|direct` — переключение маршрута доменов Gemini
- `waga net awg classic|mid|full` — переключение обфускации AmneziaWG
- `waga net happ` — редактирование баннера `announce.txt` и внешних нод `extra_nodes.txt`
- `waga backup` / `waga uninstall` — резервная копия и полное удаление

## Защита и оптимизация сервера (`waga` → пункт `4` / `waga sec`)
Все модули ставятся и включаются вручную через меню **`WAGA · Дядя Вова VPN – управление сервером → 4. Защита и оптимизация сервера`**:
- **Fail2ban с защитой от самобана** (`waga sec fail2ban`) — джейлы `sshd` + `nginx` (сканеры 404/444 и лимиты) + интерактивное управление белым списком `ignoreip`.
- **Сетевой тюнинг ядра `sysctl`** (`waga sec sysctl`) — `BBR` + `fq`, UDP-буферы 16 МБ (`rmem_max`/`wmem_max`) для Hysteria2/AmneziaWG, `tcp_syncookies`, `rp_filter`, отключение ICMP-редиректов.
- **Автообновления безопасности ОС** (`waga sec updates`) — `unattended-upgrades` только для security-патчей, без автоперезагрузки и без затрагивания `x-ui`/`xray`.
- **Автообновление домена** (`waga sec domain`) — systemd-таймер DDNS (DuckDNS или свой HTTPS URL каждые 5 минут) + проверка и продление TLS-сертификата (`acme.sh`).
- **Rate limiting в Nginx** (`waga sec ratelimit`) — `limit_req_zone` (`5r/s`, `burst=10`) и `limit_conn` на панель и `waga-sub` (с исключением `127.0.0.1` для `waga check`).
- **2FA (TOTP) в панели 3X-UI** (`waga sec 2fa`) — генерация ключа, вывод QR-кода для Google Authenticator / Bitwarden, проверка 6-значного кода перед активацией и аварийное отключение.
- **Жёсткие заголовки и сброс `444` в Nginx** (`waga sec headers`) — security-заголовки, запрет всех HTTP-методов кроме `GET/POST/HEAD` и тихий сброс `444` при запросах по чужому `Host` / прямому IP.

## Миграция, WARP и SSH-туннели (v1.2.0)
- **Безопасный импорт клиентов из старой БД** (`waga user import /root/old.db` или `waga → 1 → 7`) — переносит только пользователей и их `subId`/`UUID`/лимиты, не ломая настройки сервера.
- **Совместимость старого пути `/sub/`** (`waga net sub-legacy on|off` или `waga → 2 → 16`) — старые ссылки `/sub/<subId>` продолжают работать через `waga-sub`.
- **Автоустановка Cloudflare WARP (`127.0.0.1:40000`)** (`waga net warp install` или `waga → 2 → 13`) — локальный SOCKS5-прокси для Gemini без изменения системных маршрутов.
- **Автопроверка и починка DNS** — при установке и в `waga check` / `waga fix`.
- **SSH-туннель между двумя VPS без VPN/tun** (`waga sec tunnel` или `waga → 4 → 9`) — TCP-проброс (`relay`) и обратный SSH-туннель (`reverse`) со сменой портов.

**WAGA** решает главную проблему ручной настройки VPN-сервера: вместо часов возни с Nginx, сертификатами, несовместимыми версиями Xray и поломанными подписками вы получаете **единый защищённый стек**, где весь TCP-трафик маскируется на порту `443`, панель спрятана за `127.0.0.1`, а все дополнительные модули включаются вручную из красивого консольного меню `waga`.

| 🚀 Скорость и маскировка | 🛡️ Безопасность и автономность |
| :--- | :--- |
| **Всё через порт `443`:** Nginx SNI-роутинг разделяет REALITY, XHTTP, панель и сайт-заглушку без лишних открытых портов. | **Чистая база по умолчанию:** при установке не ставится ничего лишнего — модули защиты включаются вручную в `waga sec`. |
| **Умная подписка `waga-sub`:** одна ссылка для Happ, Hiddify, Karing, v2rayN, Clash Verge и FlClash с полной поддержкой **лимита HWID**. | **Без утечки имён:** в названиях подключений отображается только сервер (`Швеция` вместо `Швеция-admin`). |
| **Проверенная связка ядер:** `3X-UI v3.9.0` + `Xray v26.6.27` — стабильная работа REALITY на всех клиентах (включая sing-box). | **Умная миграция и туннели:** безопасный перенос клиентов из старой БД и SSH-туннели между двумя VPS без VPN/tun. |

---

## 🌐 Протоколы по умолчанию

| Протокол | Порт | Особенности |
| :--- | :---: | :--- |
| **VLESS REALITY** | `443/tcp` | Поток `xtls-rprx-vision`, маскировка под свой домен (self-steal) или соседа по подсети |
| **VLESS XHTTP + REALITY** | `443/tcp` | Запасной транспорт поверх HTTP/2 с REALITY |
| **Hysteria2** | `443/udp` | Максимальная скорость на мобильных сетях, masquerade под сайт-заглушку |
| **AmneziaWG** | `51821/udp` | Классический обфусцированный WireGuard |
| **AmneziaWG 3.1** | `51822/udp` | Расширенная обфускация (`classic` / `mid` / `full`) |

<details>
<summary><b>🤖 Встроенная маршрутизация доменов GEMINI (6 доменов из коробки)</b></summary>

Автоматически прописываются в `xrayTemplateConfig` панели 3X-UI, в шаблон `/etc/waga-sub/rules.yaml` и в генератор роутеров (`tools/`):

* `geosite:google-gemini`
* `domain:gemini.google.com`
* `domain:aistudio.google.com`
* `domain:generativelanguage.googleapis.com`
* `domain:alkalimining-pa.googleapis.com`
* `domain:proactivebackend-pa.googleapis.com`

Переключение маршрута в 1 команду: `waga net gemini ipv4 | warp | direct` (или через меню `waga → 2 → 13`, там же — установка **Cloudflare WARP `127.0.0.1:40000`** в один клик).

</details>

---

## 🎛️ Дизайнерское меню управления (`waga`)

Запустите `waga` в терминале — откроется цветное меню с ASCII-логотипом, живой строкой состояния (**IP, маскировка, статус панели, Xray, клиенты, RAM, диск**) и краткими пояснениями в скобках для каждого пункта.

| Команда | Назначение |
| :--- | :--- |
| `waga` | Интерактивный центр управления сервером |
| `waga user …` | `add`, `list`, `link`, `limit`, `on`, `off`, `del`, `import` |
| `waga net …` | Протоколы, порты, SNI, `gemini`, `warp`, `awg`, `happ`, `sub-legacy`, `brand` |
| `waga check [--deep\|--fix]` | Быстрая и глубокая диагностика + безопасное автоисправление (включая DNS) |
| `waga sec …` | Центр защиты и оптимизации сервера (ручное включение модулей) |
| `waga backup` / `waga uninstall` | Резервная копия сервера / полное чистое удаление |

---

## 🩺 Мониторинг и диагностика (`waga → 3`)

* **⚡ Тест скорости канала (`waga speed`)** — замер реальной скорости скачивания/отдачи и пинга VPS.
* **📊 Ресурсы сервера (`waga res`)** — аптайм, нагрузка CPU, RAM, заполнение диска и потребление памяти процессами `xray`, `x-ui`, `nginx`, `waga-sub`.
* **📡 Живые подключения (`waga online`)** — список клиентов онлайн из API Xray, число активных TCP/UDP сессий и Топ-5 пользователей по трафику.

---

## 🛡️ Защита и оптимизация (`waga → 4` / `waga sec`)

Все модули безопасности устанавливаются и выключаются **вручную** из меню (есть также кнопка **«Установить рекомендуемый пакет разом»**):

| Модуль | Команда | Что делает |
| :--- | :--- | :--- |
| **Fail2ban + защита от самобана** | `waga sec fail2ban` | Джейлы `sshd` + `nginx` (бан за перебор и массовые 404/444) + управление белым списком `ignoreip`. |
| **Тюнинг ядра `sysctl`** | `waga sec sysctl` | `BBR` + `fq`, UDP-буферы 16 МБ (`rmem_max`/`wmem_max`) для Hysteria2/AWG, `tcp_syncookies`, `rp_filter`, отключение ICMP-редиректов. |
| **Security-обновления ОС** | `waga sec updates` | `unattended-upgrades` только для патчей безопасности Ubuntu/Debian, без автоперезагрузки и без затрагивания `x-ui`/`xray`. |
| **Автообновление домена** | `waga sec domain` | Systemd-таймер DDNS (DuckDNS или свой HTTPS URL каждые 5 мин) + проверка и продление TLS-сертификата (`acme.sh`). |
| **Rate limiting в Nginx** | `waga sec ratelimit` | `limit_req_zone` (`5r/s`, `burst=10`) и `limit_conn` на панель и `waga-sub` (с исключением `127.0.0.1`). |
| **2FA (TOTP) в 3X-UI** | `waga sec 2fa` | Генерация ключа, QR-код в терминале (Google Authenticator / Bitwarden), проверка кода перед активацией и аварийное отключение. |
| **Жёсткие заголовки и `444`** | `waga sec headers` | Security-заголовки, запрет всех HTTP-методов кроме `GET/POST/HEAD` и тихий сброс `444` на чужой `Host` / прямой IP. |
| **Блокировка SMTP-портов** | `waga sec smtp` | Блокировка исходящих портов `25, 465, 587` в Xray — защита VPS от спам-абуз хостера. |
| **Авторотация SNI** | `waga sec sni-auto` | Плановая смена сайта маскировки раз в неделю или месяц (старое имя работает ещё 7 дней). |
| **Проверка чистоты IP** | `waga sec ip-check` | Проверка IP сервера по публичным блэклистам DNSBL (`Spamhaus`, `Spamcop`, `Barracuda`, `SORBS`) и геолокация. |

---

## 🔄 Миграция, WARP, брендирование и SSH-туннели

* **📦 Безопасный импорт клиентов (`waga user import /root/old.db`)** — переносит из старой базы 3X-UI **только пользователей** (`subId`, `UUID`, пароли, лимиты, сроки), не затирая настройки панели, порты и Nginx.
* **🔗 Совместимость `/sub/` (`waga net sub-legacy on`)** — старые ссылки `/sub/<subId>` продолжают работать через `waga-sub` без перераздачи клиентам.
* **🎨 Брендирование подписки (`waga net brand`)** — настройка названия подписки, ссылки поддержки Telegram, своей HTML-заглушки (`/var/www/waga/index.html`) и скрытия имени клиента из названий подключений.
* **🚇 SSH-туннель между двумя VPS (`waga sec tunnel`)** — режимы **Relay** (TCP-проброс) и **Reverse** (обратный SSH) со сменой портов, чтобы заходить по SSH на заблокированный VPS через второй сервер **без включения VPN/tun**.

---

<div align="center">
  <sub>Сделано для личного пользования · <b>WAGA · Дядя Вова VPN</b></sub>
</div>
