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
