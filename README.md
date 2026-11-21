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
