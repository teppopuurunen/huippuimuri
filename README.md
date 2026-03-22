# Huippuimuri ESPHome -projekt

Tämä projekti ohjaa pölynimurin puhaltimen nopeutta ESP32-S3-mikrokontrollerilla käyttäen ESPHomea.

## Ominaisuudet

- **Pulssilaskuri**: Mittaa puhaltimen kierrosnopeuden (RPM) GPIO47-pinistä.
- **PWM-säätö**: Ohjaa puhaltimen nopeutta GPIO21-pinillä LEDC:llä.
- **Web-palvelin**: Tarjoaa käyttöliittymän portissa 80.
- **WiFi-yhteys**: Yhdistää kotiverkkoon salaisilla tiedoilla.
- **Logger**: Tallentaa debug-tason logit.

## Vaatimukset

- ESP32-S3 DevKitC-1
- ESPHome (asennettu)
- USB-kaapeli flashaukseen

## Komponentit

- **MCU:** ESP32-S3 DevKit
- **Transistori:** BC547 (NPN)
- **Vastus:** 1 kΩ (Base resistor)

## Asennus

1. Kloonaa tämä repo:
   ```
   git clone https://github.com/teppopuurunen/huippuimuri.git
   cd huippuimuri
   ```

2. Muokkaa `secrets.yaml` WiFi-tiedoilla:
   ```
   wifi_ssid: "sinun_ssid"
   wifi_password: "sinun_salasana"
   ```

3. Flasaa firmware:
   ```
   esphome run huippuimuri.yaml
   ```

4. Avaa selaimessa: `http://huippuimuri.local`

## Käyttö

- **Logit**: `esphome logs huippuimuri.yaml --device COM3`
- **Päivitys WiFi:n yli**: `esphome upload huippuimuri.yaml` (kun `ota:` on lisätty)

## ECo125 FLOW -kytkentä

Tämä ohje perustuu miltei ECo125 FLOW -malleihin (E6/BB). Puhaltimen ohjaus tapahtuu 0–10 V inputista ja takaa dokumentissa valkoisella 4-vaiheisella liittimellä.

- Vihreä/keltainen (PE) maadoitus
- Ruskea (L) 230 V vaihe
- Sininen (N) 230 V nollajohdin
- Keltainen (signaali) 0–10 V ohjaus signaali (joudutaan muuntamaan PWM:ksi ESP32:n kautta)
- Valkoinen (tacho) pulssilähtö (kierrosnopeus)

## GPIO-kytkentä ESP32-S3 kohdalla

- `GPIO21` -> PWM-signaali (LEDC) FAN OUT (muunnin 0–10V) -> tuulettimen ohjaus
- `GPIO47` -> Pulssilähtö (tach, 1 puoli kierros n. 2-7V) -> `pulse_counter` sensor

Voit käyttää optoerotinta tai relettä 230 V/10 V erotteluun turvallisuuden vuoksi. Aina kytkentöjen yhteydessä katkaise verkkojännite ja varmista pätevä sähköjen ammattiosaaja.

## Tarkennettu kytkentäohje

### 1. PWM-ohjaus (Nopeuden säätö)
- **GPIO21** -> 1kΩ vastus -> BC547 Base (B)
- **BC547 Collector (C)** -> Puhaltimen Keltainen (PWM input)
- **BC547 Emitter (E)** -> GND

*Huom: Kytkentä on invertoitu (Transistori ON = PWM 0V). YAML-konfiguraatiossa on `inverted: true`.*

### 2. RPM-mittaus (Pyörintänopeus)
- **GPIO47** -> Puhaltimen Valkoinen (Tacho)
- Puhaltimen Sininen -> GND

*Käytössä ESP32:n sisäinen INPUT_PULLUP.*

## Konfiguraatio

Katso `huippuimuri.yaml` tiedostosta yksityiskohdat. Lisää tarvittaessa `api:` ja `ota:` OTA-päivityksiä varten.

## Lisenssi

Tämä projekti on avointa lähdekoodia.