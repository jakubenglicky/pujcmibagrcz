# pujcmibagr.cz

Webová prezentace půjčovny minibagru v Kněževsi u Rakovníka.

## O projektu

Statický web pro službu pronájmu minibagru. Obsahuje informace o stroji, ceník, galerii příslušenství, rezervační formulář s automatickým výpočtem ceny a napovídáním adres.

## Funkce

- Ceník s dynamickým výpočtem ceny pronájmu
- Rezervační formulář napojený na Make (Integromat) webhook
- Napovídání adres a výpočet vzdálenosti přes Mapy.cz API
- Zobrazení obsazenosti z Google Calendar
- reCAPTCHA ochrana formuláře

## Technologie

- HTML + Tailwind CSS (CDN)
- Vanilla JavaScript
- GitHub Pages (hosting)
- Make (webhook pro zpracování formuláře)
- Mapy.cz API (geokódování, vzdálenosti)
- Google Calendar API (obsazenost)

## Spuštění lokálně

```bash
python3 -m http.server 8000
```

Web bude dostupný na `http://localhost:8000`.

## Produkce

Web běží na [www.pujcmibagr.cz](https://www.pujcmibagr.cz) přes GitHub Pages.
