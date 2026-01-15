# Elektroapp Home Assistant Add-on

Add-on zobrazuje spotove ceny elektriny a vypocitane naklady na zaklade spotreby
z InfluxDB. UI je dostupne pres Home Assistant Ingress (panel v postrannim menu).

# Zdroj dat
Veskera data se stahuji z spotovaelektrina.cz.

## Instalace

1. V Home Assistant: Settings > Add-ons > Add-on Store > ... > Repositories.
2. Pridat URL GitHub repozitare s add-onem.
3. V seznamu add-onu vybrat "Elektroapp" a kliknout Install.
4. Otevrit konfiguraci add-onu, vyplnit InfluxDB a tarifni parametry.
5. Start add-onu a otevrit panel (Ingress).

## Pouziti

- Otevri Elektroapp z postranniho panelu Home Assistantu.
- Vyber datum pro denni graf "Naklady a spotreba".
- Volitelne zobraz mesicni souhrn a tabulku po dnech.
- Odhad vyuctovani ukazuje realny i projektovany odhad za mesic/rok.
- Planovac spotrebicu najde nejlevnejsi okna pro zadanou delku.

## Konfigurace

Add-on nacita nastaveni z Home Assistant options (Supervisor). 

### `dph`
- Vyse DPH v procentech, napr. `21`.

### `poplatky`
- Vsechny hodnoty jsou bez DPH (Kc / kWh). DPH se aplikuje az ve vypoctu.
- `komodita_sluzba`: Poplatek za sluzbu obchodu.
- `oze`: Cena na podporu vykupu elektiny (OZE/POZE).
- `dan`: Dan z elektriny.
- `systemove_sluzby`: Systemove sluzby (CEPS).
- `distribuce.NT`: Distribuce pro nizky tarif.
- `distribuce.VT`: Distribuce pro vysoky tarif.

### `fixni`
- Fixni poplatky bez DPH (Kc / den, Kc / mesic).
- `denni.staly_plat`: Staly plat (Kc/den).
- `mesicni.provoz_nesitove_infrastruktury`: Nesitova infrastruktura (Kc/mesic).
- `mesicni.jistic`: Jistic (Kc/mesic).

### `tarif.vt_periods`
- Casove intervaly VT (vysoky tarif), format `HH-HH` oddeleny carkou.
- Priklad: `6-7,9-10,13-14,16-17`.
- Pouziva se pro urceni VT/NT pri vypoctu ceny.

### `influxdb`
- `host`: IP nebo hostname InfluxDB.
- `port`: Port InfluxDB (defaultne 8086).
- `database`: Jmeno databaze.
- `retention_policy`: Volitelne, napr. `autogen`.
- `measurement`: Nazev measurement (napr. `kWh`).
- `field`: Nazev field (napr. `value`).
- `entity_id`: ID entity (napr. `solax_drinov_today_s_import_energy`).
- `username`: Volitelne uzivatelske jmeno.
- `password`: Volitelne heslo.
- `timezone`: Casove pasmo (napr. `Europe/Prague`).
- `interval`: Interval spotreby (napr. `15m`).

## API endpointy

Zakladni prefix: `/api`

### `GET /api/config`
- Vraci aktualni konfiguraci (merged z configu + options).

### `GET /api/prices`
- Vraci ceny pro dnes + zitra (pokud jsou dostupne).
- Volitelne `date=YYYY-MM-DD` pro konkretni den.

### `GET /api/consumption`
- Vraci spotrebu z InfluxDB.
- Parametry: `date=YYYY-MM-DD` nebo `start`, `end` (ISO).

### `GET /api/costs`
- Vraci naklady (spotreba * final cena).
- Parametry: `date=YYYY-MM-DD` nebo `start`, `end` (ISO).
- ie. http://192.168.82.120:8005/api/costs?date=2026-01-01

### `GET /api/daily-summary`
- Mesicni souhrn po dnech.
- Parametr: `month=YYYY-MM`.
- ie. http://192.168.82.120:8005/api/daily-summary?month=2025-12

### `GET /api/billing-month`
- Odhad vyuctovani pro mesic (realny + projekce).
- Parametr: `month=YYYY-MM`.

### `GET /api/billing-year`
- Odhad vyuctovani po mesicich v danem roce.
- Parametr: `year=YYYY`.

### `GET /api/schedule`
- Planovac spotrebicu podle nejlepsich oken.
- Parametry: `duration` (min), `count` (pocet navrhu, max 3).
- ie. http://192.168.82.120:8005/api/schedule?duration=60

### `GET /api/cache-status`
- Stav cache (cesta, pocet dni, posledni datum, velikost).

### `GET /api/version`
- Verze add-onu.

Api mozno vycitat v ramci HA na localhostu, nebo po otevreni portu i z externich aplikaci. Port je mozno si povolit v nastaveni add-onu (Network).
Doporucuju nastavit port 8005, s portem 8000 doplnek crashuje.

Api nema zadny overovaci mechanismus, takze opatrne pri otevirani do site.

## Poznamky

- Add-on bezi na portu 8000, ale primarne se pouziva Ingress panel v HA.
- `tarif.vt_periods` se uklada jako retezec a na backendu se prevadi na seznam.
- Odhad vyuctovani pocita fixni poplatky za cely mesic a variabilni naklady z namerene spotreby.
- Poplatky se ukladaji do historie podle data zmeny konfigurace, aby zpetne vypocty drzely puvodni hodnoty.

