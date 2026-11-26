# Meteo Ensemble – memoria del progetto

Sito meteo personale con i meteogrammi ensemble, partito da Vinci (FI). Tutto in un unico file: `index.html`.
Online su GitHub Pages: https://albeminimi.github.io/Vinci-FI-meteo-model/ (pubblica il branch `main`).

## Preferenze dell'utente (importanti)
- Scrive in italiano; risposte brevi e chiare, senza gergo tecnico.
- Il sito si usa al **90% da smartphone**: ogni modifica va pensata e provata prima sul telefono (360–390 px).
- Interfaccia in **stile Claude**: colori caldi (crema in chiaro, grigio caldo in scuro), accento terracotta
  (`#c96442` chiaro / `#d97757` scuro), titoli e temperature in serif (Source Serif 4), testo in Manrope.
  Niente titolo in stile terminale: in alto la località grande + coordinate, ora locale e fonte.
- Tema chiaro / scuro / automatico (pulsanti in alto), salvato nel browser.
- Previsioni ("Adesso" + prossimi giorni) **chiuse** all'apertura, si aprono con la barra riassuntiva.
- **Confronto modelli spento** a ogni apertura (non viene ricordato).
- Controlli dei meteogrammi compatti: sempre visibili solo Fonte, Variabile, Confronto; il resto
  (modelli, periodo, livelli, soglia, layout) nel pannello "Impostazioni", chiuso all'apertura.
- I grafici interattivi seguono le regole dei meteogrammi di **Wetterzentrale** (vedi sotto). Quando l'utente
  chiede di cambiare solo l'interfaccia, **non toccare i grafici** (verificare che restino identici al pixel).
- Prima di pubblicare l'utente vuole un **link di anteprima**; pubblica (merge su `main`) solo quando lo chiede.

## Struttura di index.html
Un solo `<script>` principale, diviso in sezioni `// ================= Nome =================`:
Configurazione (MODELS, VARIABLES, colori, LAYERS) → Stato (localStorage `meteoEnsemble.v2`, soglie con `thVer: 2`)
→ Utilità → Dati Open-Meteo → Motore dei grafici (Chart.js 4.4.1, canvas sovrapposto per mirino/evidenziazione,
legenda, confronto, analisi giornaliera, esportazioni PNG/CSV) → Rendering → Lightbox → Ricerca località →
Gestione località → Guida → Barra di analisi → Eventi → Tema → Adesso e prossimi giorni → Intestazione →
Impostazioni → Avvio. Nella `<head>` uno script applica subito il tema salvato.

## Dati
- **Ensemble**: `https://ensemble-api.open-meteo.com/v1/ensemble` con `timezone=GMT`, `past_days=1`,
  `wind_speed_unit=ms`. Modelli: `ecmwf_ifs025_ensemble`, `ncep_gefs05` (0,5° come WZ), `gem_global_ensemble`,
  `icon_global_eps` (niente 850 hPa né raffiche), `ecmwf_aifs025_ensemble` (niente raffiche).
- **Corsa operativa (Oper)**: `https://api.open-meteo.com/v1/forecast`, si provano più nomi:
  ecm `ecmwf_ifs025`; gfs `gfs_global`/`ncep_gfs_global`; gem `gem_global`/`cmc_gem_gdps`;
  ico `icon_global`/`dwd_icon_global`; aifs `ecmwf_aifs025_single`. Se fallisce, grafico senza Oper.
  Per ECMWF/GFS/GEM non ancora verificata con dati reali (quota API esaurita durante i test).
- **Orario della corsa**: `https://ensemble-api.open-meteo.com/data/<meta>/static/meta.json`
  (`last_run_initialisation_time`); meta: `ecmwf_ifs025_ensemble`, `ncep_gefs05`, `cmc_gem_geps`,
  `dwd_icon_global_eps_native`, `ecmwf_aifs025_ensemble`.
- **Adesso / prossimi giorni**: `https://api.open-meteo.com/v1/forecast` (best match, `timezone=auto`).
- **Ricerca località**: `https://geocoding-api.open-meteo.com/v1/search` (lingua it).
- **Immagini Wetterzentrale** (fonte "Wetterzentrale"): `ens_image.php?geoid=…`; Vinci = geoid 70549.
- L'IP condiviso degli ambienti di test può esaurire la quota giornaliera di Open-Meteo: non è un problema dell'utente.

## Wetterzentrale (cosa sappiamo)
- Prende i dati dai centri dei modelli (ECMWF, NOAA, ECCC, DWD), come Open-Meteo: per ECMWF 850 hPa i valori
  coincidono (differenza media 0,04 °C, stesso punto di griglia 43.75N 11E).
- Licenza dati/mappe: CC BY-NC 4.0 (uso non commerciale citando www.wetterzentrale.de).
- Feed dati JSONP: `https://wetterzentrale.de/ens_feed.php?geoid=…&var=2&run=00&date=YYYY-MM-DD&model=ecm&member=ENS&bw=1&tr=6&callback=…`
  (var 2 = T850, 4 = pioggia, 5 = T2m, 9 = vento, 23 = pressione). Ricerca città `Cities.php?query=…`:
  niente CORS né JSONP, quindi inutilizzabile dal sito.
- Regole replicate nei nostri grafici: ore UTC, un punto ogni 6 h da +6 h, durata ECMWF/AIFS 360 h,
  GFS/GEM 384 h, ICON 180 h; tavolozza di 20 colori (`WZ_COLORS`), Ens. mean `#fefefe` 3 px, Oper `#00dc00` 3 px,
  Control `#0000ff` 2 px (solo GFS/GEM; nascosto per ECMWF/AIFS; per ICON è uno scenario).
  Pioggia a 6 h sulla destra 0–40 mm; precipitazioni cumulate 0–45 mm; vento in m/s; linea dello zero per T2m.
  Scala temperatura: fondo = floor10(min media − 0,6·ampiezza media − 10), cima = ceil5(max + 0,2·ampiezza);
  tacche ≥ 2 (ogni 2 fino a ~35 di ampiezza, poi ogni 5). Combacia quasi sempre; a volte WZ ha +5 in alto.
- Manca solo la linea del clima 1991–2020 (non disponibile su Open-Meteo).

## Come lavorare
- Branch di lavoro: `claude/weather-interface-location-select-qxqsc0`. Dopo ogni merge ripartire da `main`
  (`git checkout -B <branch> origin/main`).
- GitHub: niente GraphQL; usare `gh api` REST (PR: `POST repos/albeminimi/Vinci-FI-meteo-model/pulls`,
  merge: `PUT …/pulls/<n>/merge` solo quando l'utente lo chiede).
- Anteprima: `https://raw.githack.com/albeminimi/Vinci-FI-meteo-model/<sha completo>/index.html`
  (verificare con curl che coincida col file). Dopo il merge verificare che il sito pubblicato coincida con `main`.
- Test: Playwright (`/opt/node22/lib/node_modules/playwright/index.mjs`, Chromium già installato), Chart.js
  scaricato con `npm pack chart.js@4.4.1`, risposte delle API simulate con `page.route` partendo da dati reali.
  Provare sempre telefono (390×844 con touch) e computer, temi chiaro e scuro, nessuno scorrimento laterale.
