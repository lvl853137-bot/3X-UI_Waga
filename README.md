<div align="center">

<!-- Обложка проекта (положите картинку в папку images/banner.png в репозитории) -->
<img src="https://raw.githubusercontent.com/lvl853137-bot/3X-UI_Waga/main/images/banner.png" alt="WAGA · Дядя Вова VPN Banner" width="900" style="border-radius: 12px;" />

# 🚀 WAGA · Дядя Вова VPN `v1.4.0`

[![Core](https://img.shields.io/badge/Xray-v26.6.27-00B4D8?style=for-the-badge&logo=go&logoColor=white)](https://github.com/XTLS/Xray-core)
[![Panel](https://img.shields.io/badge/3X--UI-v3.9.0-7B2CBF?style=for-the-badge&logo=linux&logoColor=white)](https://github.com/Morytyann/3X-UI)
[![OS](https://img.shields.io/badge/OS-Ubuntu_22.04%2F24.04_%7C_Debian_12%2F13-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](#)
[![Anti-DPI](https://img.shields.io/badge/Anti--DPI-Cascade_%2B_Routing-2EC4B6?style=for-the-badge&logo=shieldsdotio&logoColor=white)](#)

</div>

> **WAGA** — собственная система развёртывания и управления VPN на базе **3X-UI v3.9.0 + Xray v26.6.27 + Nginx (SNI 443) + waga-sub**. Разработана для стабильной работы в условиях ТСПУ и блокировок мобильных операторов, с готовой маршрутизацией и каскадными мостами между VPS.

---

## ⚡ Установка одной командой

Поддерживаются **Ubuntu 22.04 / 24.04** и **Debian 12 / 13**:

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/lvl853137-bot/3X-UI_Waga/main/scripts/waga-install.sh)
```

---

## 🛠 Протоколы по умолчанию

| Протокол | Порт / Транспорт | Особенности | Назначение |
| :--- | :--- | :--- | :--- |
| **VLESS REALITY** | `443/tcp` | `xtls-rprx-vision` | Основной стабильный транспорт |
| **Hysteria2** | `443/udp` | `masquerade` под живой сайт | Максимальная скорость на мобильных сетях |
| **VLESS XHTTP + REALITY** | `443/tcp` | `mode=auto`, `xPaddingBytes` | Маскировка под обычный HTTP-трафик |
| **AmneziaWG** | `51821/udp` | Обфусцированный WireGuard | Быстрый нативный туннель |
| **AmneziaWG 3.1** | `51822/udp` | Усиленная обфускация | Запасной UDP-транспорт для сложных сетей |

---

## 🗺 Маршрутизация Happ и Incy по умолчанию (`v1.4.0`)

Сервис подписки `waga-sub` автоматически отдаёт в заголовках `Routing` и `Routing-Enable: 1` готовый профиль маршрутизации **RoscomVPN** для клиентов **Happ** и **Incy**:

```text
[Клиент] ──► { Happ / Incy }
                ├──► Direct (РФ)  ──► category-ru, whitelist, microsoft, apple, steam, epicgames, riot, twitch, pinterest, faceit
                ├──► Proxy (VPN)  ──► youtube, telegram, github, google-play, twitch-ads + все остальные зарубежные сайты (GlobalProxy: true)
                └──► Block        ──► win-spy, torrent, category-ads
```

* **🌐 DNS:** DoH `77.88.8.8` (РФ) + DoH `8.8.8.8` (зарубежные домены), плюс статические записи `lkfl2.nalog.ru` / `lknpd.nalog.ru`.
* **🟢 Direct:** `geosite:private`, `geosite:category-ru`, `geosite:whitelist`, `microsoft`, `apple`, `steam`, `epicgames`, `riot`, `twitch`, `pinterest`, `faceit`, `geoip:private`, `geoip:direct`.
* **🔵 Proxy:** `youtube`, `telegram`, `github`, `google-play`, `twitch-ads` и весь остальной зарубежный трафик.
* **🔴 Block:** `geosite:win-spy`, `geosite:torrent`, `geosite:category-ads`.
* **⚙️ Управление:** `waga → 2 → 15` (`waga net happ`) или файл `/etc/waga-sub/happ_routing.txt` (можно подставить свою ссылку `happ://routing/...` или записать `off`, чтобы отключить).

---

## 🤖 Встроенные домены GEMINI

Автоматически прописываются в `xrayTemplateConfig` панели 3X-UI, в `/etc/waga-sub/rules.yaml` и в генератор роутеров (`tools/`):

* `geosite:google-gemini`
* `domain:gemini.google.com`
* `domain:aistudio.google.com`
* `domain:generativelanguage.googleapis.com`
* `domain:alkalimining-pa.googleapis.com`
* `domain:proactivebackend-pa.googleapis.com`

---

## 🎮 Управление (`waga`)

| Команда | Описание |
| :--- | :--- |
| `waga` | Интерактивное консольное меню |
| `waga check --deep` / `waga fix` | Диагностика сервера и безопасное автоисправление |
| `waga net gemini ipv4\|warp\|direct` | Переключение маршрута доменов Gemini |
| `waga net awg classic\|mid\|full` | Переключение уровня обфускации AmneziaWG |
| `waga net happ` | Баннер `announce.txt`, внешние ноды `extra_nodes.txt` и маршрутизация Happ/Incy |
| `waga speed` / `waga res` / `waga online` | Тест скорости, ресурсы сервера и активные подключения |
| `waga backup` / `waga uninstall` | Резервная копия и полное удаление |

---

## 🛡 Защита и оптимизация сервера (`waga → 4` / `waga sec`)

```text
🛡 Fail2ban (waga sec fail2ban)    ──► Джейлы sshd + nginx + белый список ignoreip (защита от самобана)
🚀 sysctl (waga sec sysctl)        ──► BBR + fq, UDP-буферы 16 МБ (rmem_max/wmem_max), защита TCP
🔄 Updates (waga sec updates)      ──► Автообновления безопасности ОС без перезагрузки
🌐 Domain (waga sec domain)        ──► DDNS (DuckDNS или свой HTTPS URL) + продление TLS
🧱 Nginx (ratelimit / headers)     ──► Rate limiting, защита панели и тихий сброс 444
🔑 2FA (waga sec 2fa)              ──► Вход в панель 3X-UI по TOTP-коду с телефона
🚇 Tunnel (waga sec tunnel)        ──► SSH-туннель между VPS: TCP-проброс (relay) и обратный (reverse)
🚦 Anti-Abuse (smtp/sni/ip-check)  ──► Блок SMTP 25/465/587, ротация SNI (sni-auto), проверка IP по DNSBL
```

---

## 🌉 Каскадное подключение между VPS (`waga sec cascade` / `waga → 4 → 13`)

Помогает обойти блокировки мобильных операторов (**МТС, Билайн, Yota, Т2, МегаФон**), когда прямой доступ к выходному VPS нестабилен, а промежуточный VPS (точка входа) доступен стабильно.

```text
[Клиент] ──(TCP + UDP :8443)──► [Entry VPS (Вход)] ──(прозрачный мост)──► [Exit VPS (Выход :443)] ──► Интернет
```

### 1. Поднятие моста на входном VPS (Entry VPS)

```bash
waga sec cascade add <имя> <порт_на_входе> <IP_выхода> <порт_на_выходе> [both|tcp|udp]
```

Пример (Швеция пробрасывает `8443` TCP + UDP на Македонию `138.124.252.98:443`):

```bash
waga sec cascade add mk 8443 138.124.252.98 443 both
```

Команда создаёт службы `waga-cascade-<имя>-tcp.service` и `waga-cascade-<имя>-udp.service` (с `UDP4-LISTEN`) и открывает порты в `ufw`.

### 2. Автопривязка инбаундов и подписки одной командой (`v1.4.0`)

Вместо ручной правки JSON и SQLite выполните на сервере с панелью (или выберите пункт **3** в `waga → 4 → 13`):

```bash
waga sec cascade bind bonbon.zanity.net 8443 Македония
```

Скрипт сделает резервную копию `/etc/x-ui/x-ui.db`, обновит `externalProxy` и таблицу `hosts` для подходящих записей и перезапустит `x-ui`.

---

## 📡 Рекомендации против блокировок мобильных операторов

1. **Используйте каскад (`Entry VPS → Exit VPS`)** для проблемных локаций: клиенты подключаются к стабильному входному IP/домену, а выходят в интернет с нужной страны.
2. **Включите тюнинг ядра** (`waga sec sysctl on` — BBR + UDP 16 МБ) на обоих серверах.
3. **Подбирайте чистый SNI** для VLESS REALITY / XHTTP (`waga sec sni-auto`), реально отвечающий по TLS 1.3.
4. **Комбинируйте транспорты:** держите в подписке `Hysteria2` и `VLESS XHTTP` как запасные варианты на случай троттлинга TCP или UDP отдельным оператором.

---

<div align="center">
  <sub>🚀 WAGA Project · Дядя Вова VPN</sub>
</div>