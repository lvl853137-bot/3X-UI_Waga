<div align="center">

<!-- Замените путь ниже на вашу обложку, если она называется иначе -->
<img src="banner.jpg" alt="WAGA · Дядя Вова VPN" width="100%" />

# 🚀 WAGA · Дядя Вова VPN

**Автономная система развёртывания, защиты и управления VPN-сервером**  
*3X-UI v3.9.0 · Xray v26.6.27 · Nginx SNI (443) · waga-sub (HWID)*

<p>
  <img src="https://img.shields.io/badge/version-1.3.1-00d2ff?style=for-the-badge" alt="Version 1.3.1" />
  <img src="https://img.shields.io/badge/OS-Ubuntu%2022.04%2F24.04%20%7C%20Debian%2012%2F13-0a7ea4?style=for-the-badge" alt="OS" />
  <img src="https://img.shields.io/badge/3X--UI-v3.9.0-6f42c1?style=for-the-badge" alt="3X-UI" />
  <img src="https://img.shields.io/badge/Xray-v26.6.27-2ea44f?style=for-the-badge" alt="Xray" />
  <img src="https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge" alt="License" />
</p>

[⚡ Быстрая установка](#-быстрая-установка) •
[🧩 Что умеет WAGA](#-для-чего-нужен-проект-и-что-он-умеет) •
[🛡️ Защита (waga sec)](#️-защита-и-оптимизация-waga--4--waga-sec) •
[🩺 Мониторинг](#-мониторинг-и-диагностика-waga--3) •
[🔄 Миграция и туннели](#-миграция-warp-брендирование-и-ssh-туннели)

</div>

---

> [!WARNING]
> Проект представляет собой набор скриптов с открытым исходным кодом. Он не оказывает услуг связи и не предоставляет доступ к серверам. Вы устанавливаете его на собственный VPS и самостоятельно несёте ответственность за соблюдение законодательства своей страны. Проект создан в образовательных целях.

---

## ⚡ Быстрая установка

Поддерживаются чистые серверы **Ubuntu 22.04 / 24.04** и **Debian 12 / 13**. Установка занимает 3–5 минут и автоматически проверяет/чинит DNS при старте:

```bash
bash <(curl -4fsSL https://raw.githubusercontent.com/lvl853137-bot/3X-UI_Waga/main/scripts/waga-install.sh)
```

<details>
<summary><b>🔄 Обновить уже установленный WAGA до последней версии (без переустановки)</b></summary>

```bash
curl -4fsSL https://raw.githubusercontent.com/lvl853137-bot/3X-UI_Waga/main/scripts/waga-sub.py -o /usr/local/lib/waga-sub/waga_sub.py \
  && curl -4fsSL https://raw.githubusercontent.com/lvl853137-bot/3X-UI_Waga/main/scripts/waga.sh -o /usr/local/bin/waga \
  && chmod 755 /usr/local/bin/waga \
  && systemctl restart waga-sub \
  && waga
```

</details>

---

## 🧩 Для чего нужен проект и что он умеет

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