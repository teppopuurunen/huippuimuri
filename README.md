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

## Konfiguraatio

Katso `huippuimuri.yaml` tiedostosta yksityiskohdat. Lisää tarvittaessa `api:` ja `ota:` OTA-päivityksiä varten.

## Lisenssi

Tämä projekti on avointa lähdekoodia.