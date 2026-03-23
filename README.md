# Huippuimuri ESPHome -projekti

Tämä projekti ohjaa huippuimurin (ECo125 FLOW) puhaltimen nopeutta ESP32-S3-mikrokontrollerilla käyttäen ESPHomea. Puhallin säätyy automaattisesti huoneilman CO2-pitoisuuden mukaan hyödyntäen IKEA Dirigera -keskittimen kautta luettavia CO2-antureita.

## Ominaisuudet

- **CO2-automaattiohjaus**: Hakee CO2-arvot kahdelta IKEA-anturilta Dirigera-hubilta (HTTPS) 30 sekunnin välein ja säätää puhaltimen tehon automaattisesti.
- **Porrastettu nopeussäätö**: 5 porrasta hysteresiksellä — ei turhia heilahteluja.
- **Eksponentiaalinen tasoitus (EMA)**: CO2-arvo tasoitetaan (70/30) ennen portaiden laskentaa.
- **Varatoiminto**: Jos CO2-data on vanhentunut ≥ 3 sykliä, puhallin asetetaan automaattisesti 55% tehoon.
- **CO2 automaattiohjaus -kytkin**: Kytkimen voi sammuttaa, jolloin puhallin siirtyy käsiajoon eikä CO2-logiikka enää overridaa asetettua nopeutta.
- **RPM-mittaus**: Pulssilaskuri GPIO47:ssä mittaa puhaltimen kierrosnopeuden.
- **PWM-ohjaus**: Puhaltimen nopeus ohjataan GPIO21:n LEDC-PWM:llä (5 kHz, invertoitu).
- **Web-palvelin**: ESPHome-web-käyttöliittymä portissa 80.
- **OTA-päivitys**: Firmware päivitettävissä WiFin yli.
- **Kahden WiFi-verkon tuki**: Ensisijainen ja varaverkko prioriteetin mukaan.

## CO2-ohjauksen portaat

| Porras | Nopeus | CO2 nousu (kynnys) | CO2 lasku (hystereesi) |
|--------|--------|-------------------|----------------------|
| 0 | 20 % | ≥ 650 ppm → porras 1 | — |
| 1 | 35 % | ≥ 850 ppm → porras 2 | < 550 ppm → porras 0 |
| 2 | 55 % | ≥ 1050 ppm → porras 3 | < 750 ppm → porras 1 |
| 3 | 75 % | ≥ 1250 ppm → porras 4 | < 950 ppm → porras 2 |
| 4 | 100 % | — | < 1150 ppm → porras 3 |

Varatoiminto (data vanhentuu): 55 % (porras 2).

## Vaatimukset

- ESP32-S3 DevKitC-1
- ESPHome 2026.3.0 tai uudempi
- IKEA Dirigera -hub (CO2-anturit liitetty hubiin)
- USB-kaapeli ensimmäistä flashausta varten

## Komponentit

- **MCU:** ESP32-S3 DevKit
- **Transistori:** BC547 (NPN)
- **Vastus:** 1 kΩ (base-vastus)
- **Vastus:** 1 kΩ – 10 kΩ (pull-up punaisen ja keltaisen väliin)

## Asennus

1. Kloonaa tämä repo:
   ```
   git clone https://github.com/teppopuurunen/huippuimuri.git
   cd huippuimuri
   ```

2. Luo `secrets.yaml` (ei commitoida, gitignoressa):
   ```yaml
   wifi_ssid: "sinun_ssid"
   wifi_password: "sinun_salasana"
   wifi2_ssid: "varaverkko_ssid"
   wifi2_password: "varaverkko_salasana"
   dirigera_auth_header: "Bearer <Dirigera-token>"
   ```

3. Flasaa firmware (ensimmäinen kerta USB:n kautta):
   ```
   esphome run huippuimuri.yaml
   ```

4. Jatkossa OTA WiFin yli:
   ```
   esphome run huippuimuri.yaml --device 192.168.100.xx
   ```

5. Avaa selaimessa: `http://huippuimuri.local`

## Käyttö

- **Logit**: `esphome logs huippuimuri.yaml --device 192.168.100.xx`
- **CO2-automaattiohjaus**: Web-käyttöliittymässä tai Home Assistantissa kytkin "CO2 automaattiohjaus" — ON = automaatti, OFF = käsiajo
- **Sensorit**: CO2 anturi 1, CO2 anturi 2, CO2 maksimi (tasoitettu arvo), Puhaltimen nopeus (RPM), WiFi Signal Strength

## ECo125 FLOW -kytkentä

Tämä ohje perustuu ECo125 FLOW -malleihin (E6/BB). Puhaltimen ohjaus tapahtuu 0–10 V -inputilla. Liitin on valkoisessa 4-johtimisessa liittimessä.

- Vihreä/keltainen (PE) — maadoitus
- Ruskea (L) — 230 V vaihe
- Sininen (N) — 230 V nollajohdin
- Keltainen (signaali) — 0–10 V ohjaussignaali (muunnetaan PWM:ksi ESP32:n kautta)
- Valkoinen (tacho) — pulssilähtö (kierrosnopeus)

## GPIO-kytkentä ESP32-S3

- `GPIO21` → 1 kΩ vastus → BC547 Base (B)
- BC547 Collector (C) → Puhaltimen keltainen (PWM input)
- BC547 Emitter (E) → ESP32 GND
- Puhaltimen punainen (+10V) → 1 kΩ – 10 kΩ pull-up → Puhaltimen keltainen (PWM input)
- `GPIO47` → Puhaltimen valkoinen (tacho) → `pulse_counter`-sensori
- Puhaltimen sininen → ESP32 GND

Voit käyttää optoerotinta tai relettä 230 V/10 V erotteluun turvallisuuden vuoksi. Katkaise aina verkkojännite kytkentöjen yhteydessä ja varmista pätevä sähköammattilainen.

## Tarkennettu kytkentäohje

Tämä kytkentä mahdollistaa 3.3 V ESP32-signaalin muuntamisen puhaltimen vaatimalle 0–10 V tasolle.

### 1. PWM-ohjaus (nopeuden säätö)
- **GPIO21** → 1 kΩ vastus → BC547 Base (B) (keskijalka)
- **BC547 Collector (C)** (vasen jalka) → Puhaltimen keltainen (PWM)
- **BC547 Emitter (E)** (oikea jalka) → GND (yhteinen maa)
- **PULL-UP VASTUS:** Kytke 1 kΩ – 10 kΩ vastus puhaltimen punaisen (+10V) ja keltaisen (PWM) johdon välille.

*Huom: Kytkentä on invertoitu (transistori ON = PWM 0V). YAML-konfiguraatiossa `inverted: true`.*

### 2. RPM-mittaus (pyörintänopeus)
- **GPIO47** → Puhaltimen valkoinen (tacho)
- **Puhaltimen sininen** → ESP32 GND (yhteinen maa)

*Käytössä ESP32:n sisäinen INPUT_PULLUP.*

## Konfiguraatio

Katso `huippuimuri.yaml` yksityiskohdista. Tärkeimmät parametrit:

- `http_request.buffer_size_rx: 4096` — riittävä Dirigera-vastausten (~1200 B) lukemiseen
- `max_response_buffer_size: 8192` — per HTTP-pyyntö
- CO2-haku 30 s välein, EMA-tasoitus 70/30
- Vanhentuneen datan raja: 120 s (2 min)

## Lisenssi

Tämä projekti on avointa lähdekoodia.
