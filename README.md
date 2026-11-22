# 🚀 WAGA · Дядя Вова VPN (v1.4.0)

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

## Маршрутизация Happ и Incy по умолчанию (v1.4.0)
Сервис подписки `waga-sub` автоматически отдаёт в заголовках подписки `Routing` и `Routing-Enable: 1` готовый профиль маршрутизации **RoscomVPN** для клиентов **Happ** и **Incy**:
- **DNS:** DoH `77.88.8.8` (РФ) + DoH `8.8.8.8` (зарубежные домены), плюс статические записи `lkfl2.nalog.ru` / `lknpd.nalog.ru`.
- **Напрямую (`Direct`):** `geosite:private`, `geosite:category-ru`, `geosite:whitelist`, `microsoft`, `apple`, `steam`, `epicgames`, `riot`, `twitch`, `pinterest`, `faceit`, `geoip:private`, `geoip:direct`.
- **Через VPN (`Proxy`):** `youtube`, `telegram`, `github`, `google-play`, `twitch-ads` и все остальные зарубежные сайты (`GlobalProxy: true`).
- **Блокировка (`Block`):** `geosite:win-spy`, `geosite:torrent`, `geosite:category-ads`.
- **Управление:** `waga → 2 → 15` (`waga net happ`) или файл `/etc/waga-sub/happ_routing.txt` (можно подставить свою ссылку `happ://routing/...` или записать `off`, чтобы отключить).

## Встроенные домены GEMINI
Автоматически прописываются в `xrayTemplateConfig` панели 3X-UI, в `/etc/waga-sub/rules.yaml` и в генератор роутеров (`tools/`):
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
- `waga net happ` — баннер `announce.txt`, внешние ноды `extra_nodes.txt` и маршрутизация Happ/Incy
- `waga speed` / `waga res` / `waga online` — тест скорости, ресурсы сервера и активные подключения
- `waga backup` / `waga uninstall` — резервная копия и полное удаление

## Защита и оптимизация сервера (`waga → 4` / `waga sec`)
- **Fail2ban с защитой от самобана** (`waga sec fail2ban`) — джейлы `sshd` + `nginx` + белый список `ignoreip`.
- **Тюнинг ядра `sysctl`** (`waga sec sysctl`) — `BBR` + `fq`, UDP-буферы 16 МБ (`rmem_max`/`wmem_max`) для Hysteria2/AmneziaWG, защита TCP.
- **Автообновления безопасности ОС** (`waga sec updates`) — только security-патчи без перезагрузки.
- **Автообновление домена** (`waga sec domain`) — DDNS (DuckDNS или свой HTTPS URL) + продление TLS.
- **Rate limiting и заголовки Nginx** (`waga sec ratelimit`, `waga sec headers`) — защита панели и тихий сброс `444`.
- **2FA (TOTP) в 3X-UI** (`waga sec 2fa`) — вход в панель по коду с телефона.
- **SSH-туннель между VPS** (`waga sec tunnel` / `waga → 4 → 9`) — TCP-проброс (`relay`) и обратный туннель (`reverse`).
- **Защита от абуз и детекта** (`waga sec smtp`, `waga sec sni-auto`, `waga sec ip-check`) — блок SMTP 25/465/587, плановая ротация SNI и проверка IP по DNSBL.

---

## Каскадное подключение между VPS (`waga sec cascade` / `waga → 4 → 13`)

Помогает обойти блокировки мобильных операторов (МТС, Билайн, Yota, Т2, МегаФон), когда прямой доступ к выходному VPS нестабилен, а промежуточный VPS (точка входа) доступен стабильно.

### 1. Поднятие моста на входном VPS (Entry VPS)

```bash
waga sec cascade add <имя> <порт_на_входе> <IP_выхода> <порт_на_выходе> [both|tcp|udp]
```

Пример (`Швеция` пробрасывает `8443` TCP + UDP на `Македонию` `138.124.252.98:443`):

```bash
waga sec cascade add mk 8443 138.124.252.98 443 both
```

Команда создаёт службы `waga-cascade-<имя>-tcp.service` и `waga-cascade-<имя>-udp.service` (с `UDP4-LISTEN`) и открывает порты в `ufw`.

### 2. Автопривязка инбаундов и подписки одной командой (v1.4.0)

Вместо ручной правки JSON и SQLite достаточно выполнить (или выбрать пункт **`3`** в `waga → 4 → 13`):

```bash
waga sec cascade bind bonbon.zanity.net 8443 Македония
```

Команда сделает резервную копию `/etc/x-ui/x-ui.db`, обновит `externalProxy` и таблицу `hosts` для подходящих записей и перезапустит `x-ui`.

### 3. Рекомендации против блокировок мобильных операторов
- Используйте **каскад (Entry VPS → Exit VPS)** для проблемных локаций: клиенты подключаются к стабильному входному IP/домену, а выходят в интернет с нужной страны.
- Держите включённым **тюнинг ядра** (`waga sec sysctl on` — BBR + UDP 16 МБ) на обоих серверах.
- Для **VLESS REALITY / XHTTP** выбирайте чистый, реально отвечающий по TLS 1.3 SNI (`waga sec sni-auto`) и избегайте заблокированных подсетей.
- Для **Hysteria2** и **VLESS XHTTP** держите их в подписке как запасные транспорты на случай троттлинга TCP или UDP отдельным оператором.
