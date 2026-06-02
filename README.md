# 🏠 Mikzone Boti — Okos Otthon Vezérlőpanel

> ESP32-S3 alapú, 4.3" érintőképernyős Home Assistant vezérlőpanel — ESPHome + LVGL

Egy teljesen egyedi, fába épített okosotthon-vezérlő, ami egyetlen érintőképernyőn fogja össze az otthon összes fontos információját és vezérlését: időjárás, levegőminőség, napelem, elektromos autó, jelenlétérzékelés, világítás és fűtés.

![Platform](https://img.shields.io/badge/platform-ESP32--S3-blue)
![Framework](https://img.shields.io/badge/ESPHome-LVGL-green)
![Display](https://img.shields.io/badge/kijelző-800×480-orange)
![Nyelv](https://img.shields.io/badge/nyelv-magyar-red)

<p align="center">
  <img src="images/panel.jpg" alt="Mikzone Boti vezérlőpanel - kezdőlap" width="700">
</p>

---

## 📸 Képernyők

| Oldal | Leírás |
|-------|--------|
| 🏠 **Kezdőlap** | Nagy óra, dátum, benti hőm./pára, időjárás kártya, napkelte/nyugta, HA kapcsolat státusz |
| 🌫️ **Levegőminőség** | SEN55 szenzor — VOC index nagy ívben + PM2.5, PM10, NOx, hőm, pára |
| ☀️ **Napenergia** | Growatt inverter — PV termelés, fogyasztás, akku, hálózat + EV töltés, animált energiaáramlás |
| 📡 **Jelenlét** | LD2410 radar — forgó sweep animáció + 1 órás aktivitás histogramm |
| 💡 **Fények** | 10 lámpa + okos aljzat, kapu, hangulatvilágítás, „MIND KI" gomb |
| 🔥 **Fűtés** | Faskazán termosztát + puffer/padlófűtés/bojler hőmérsékletek és vezérlés |
| 🌦️ **Előrejelzés** | 5 napos időjárás overlay (Glass Morphism dizájn, PirateWeather) |
| ⚙️ **Beállítások** | Éjszakai mód időzítés, képváltó, stílus választó |
| 🌙 **Éjszakai mód** | Két stílus: Amber (meleg ambient) + Matrix (digital rain) |

---

## 🛠️ Hardver

| Komponens | Típus |
|-----------|-------|
| **Vezérlő** | Waveshare ESP32-S3 Touch LCD 4.3B |
| **Kijelző** | 800×480 RGB IPS, parallel interface |
| **Érintő** | GT911 kapacitív touch |
| **IO expander** | CH422G (I2C) |
| **Ház** | Egyedi fa keret |

### Csatlakoztatott szenzorok / eszközök
- **Sensirion SEN55** — levegőminőség (PM2.5/PM10/VOC/NOx/hőm/pára)
- **HLK-LD2410** — 24GHz jelenlétérzékelő radar
- **Growatt SPF5000 ES** — napelemes inverter (Modbus)
- **Dacia Spring** — elektromos autó (Renault integráció)
- Faskazán + puffer tartály + padlófűtés + bojler
- Okos aljzatok, fali kapcsolók, bejárati kapu

---

## 📐 Architektúra

A konfiguráció **moduláris** — az ESPHome `packages:` funkciójával külön YAML fájlokba szervezve:

```
esphome-web-bac910.yaml   ← fő belépési pont (hardver, WiFi, OTA, packages)
└── dashboard_43/
    ├── _config.yaml       ← KÖZPONTI: minden HA entitás + szín egy helyen
    ├── core.yaml          ← display, touch, fontok, ikonok, tab bar, boot screen, stílusok
    ├── home.yaml          ← kezdőoldal
    ├── air_quality.yaml   ← levegőminőség oldal
    ├── solar_ev.yaml      ← napenergia + EV oldal
    ├── radar.yaml         ← jelenlétérzékelő oldal
    ├── lights.yaml        ← világítás vezérlés
    ├── heating.yaml       ← fűtés vezérlés
    ├── night_mode.yaml    ← éjszakai mód logika + carousel + konfig entitások
    ├── settings.yaml      ← beállítások képernyő UI
    └── forecast.yaml      ← 5 napos előrejelzés overlay
```

**Tervezési elv:** ha egy HA entitást átnevezel, **csak a `_config.yaml`-t kell módosítani** — a többi fájl `${entity_xxx}` substitutionökön keresztül hivatkozik rájuk. A színek is itt vannak központilag.

---

## 🚀 Telepítés

### Előfeltételek
- [Home Assistant](https://www.home-assistant.io/) + [ESPHome](https://esphome.io/) addon
- A fenti szenzorok/integrációk beállítva a HA-ban
- [PirateWeather](https://www.home-assistant.io/integrations/pirateweather/) integráció (időjárás)

### Lépések
1. Másold a repo tartalmát az ESPHome konfig könyvtárba:
   ```
   /config/esphome/esphome-web-bac910.yaml
   /config/esphome/dashboard_43/*.yaml
   /config/esphome/icons/*.png        ← időjárás ikonok + hátterek
   /config/esphome/materialdesignicons-webfont.ttf
   ```
2. Hozz létre egy `secrets.yaml`-t a WiFi adatokkal:
   ```yaml
   wifi_ssid: "A_WIFI_NEVED"
   wifi_password: "A_JELSZAVAD"
   ```
3. Nyisd meg a `_config.yaml`-t és **írd át az entitás neveket** a sajátjaidra.
4. Flasheld az ESPHome-ból (első alkalommal USB-n, utána OTA).

---

## ⚙️ Konfiguráció

Minden testreszabható érték a [`_config.yaml`](dashboard_43/_config.yaml)-ben:

- **Színek** — `color_accent_blue`, `color_accent_heat`, stb. (LVGL hex formátum)
- **Entitások** — időjárás, szenzorok, napelem, EV, fűtés, lámpák, kapu

Az **éjszakai mód időzítését** (kezdő/záró óra) a HA-ból is állíthatod:
- `number.ejszakai_mod_kezdete`
- `number.ejszakai_mod_vege`

A **háttérvilágítás** `switch.hattervilagitas` néven érhető el (switch, hogy a „minden lámpa le" parancs ne kapcsolja le a kijelzőt).

---

## ✨ Funkciók kiemelve

- 🎨 **Egyedi boot screen** — „powered by Mikzone Boti" branding animációval
- 🌙 **Éjszakai mód** — automatikus időzítés (HA-ból állítható) vagy manuális, két stílussal
- 🔄 **Automata képváltó** — beállítható időközönként lapozza az oldalakat
- 📊 **Animált energiaáramlás** — a napenergia oldalon mozgó pontok mutatják az áram irányát
- 🛰️ **Élő radar animáció** — forgó sweep + aktivitás histogramm
- 🔔 **Értesítések** — küszöbérték-riasztások (kazán túlmelegedés, rossz levegő, kapu nyitva)
- 🌡️ **Glass Morphism előrejelzés** — modern, áttetsző kártyás 5 napos időjárás

---

## 🙏 Köszönet

- [ESPHome](https://esphome.io/) — a firmware keretrendszer
- [LVGL](https://lvgl.io/) — a grafikus könyvtár
- [Material Design Icons](https://pictogrammers.com/library/mdi/) — az ikonkészlet
- [PirateWeather](https://pirateweather.net/) — időjárás API

---

*Készítette: Mikzone Boti • Miercurea-Ciuc 🇷🇴*
