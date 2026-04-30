# 🚀 Projekt Nexus: Analiza kratera Jezero

## 📌 Izvršni sažetak (Executive Summary)
Cilj projekta je analiza geoprostornih i kemijskih podataka iz kratera Jezero na Marsu radi pronalaska optimalnih lokacija za bušenje. Ulazni podaci dolaze iz CSV datoteka s informacijama o lokacijama i uzorcima, a krajnji rezultat je generiranje automatiziranih navigacijskih naloga za robotsku misiju.

---

## 🧹 Metodologija obrade podataka (Data Wrangling)

### Učitavanje, spajanje i čišćenje podataka
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import requests

# Učitavanje podataka
df_lokacije = pd.read_csv('data/mars_lokacije.csv', sep=';')
df_uzorci = pd.read_csv('data/mars_uzorci.csv', sep=';')

# Spajanje
df = pd.merge(df_lokacije, df_uzorci, on='ID')

# Čišćenje (uklanjanje anomalija)
df_cisto = df[
    (df['TEMP'] > -150) & (df['TEMP'] < 70) &
    (df['PH'] >= 0) & (df['PH'] <= 14) &
    (df['VLAGA'] >= 0)
]
# 1. Temp vs vlaga
sns.scatterplot(data=df_cisto, x='TEMP', y='VLAGA', hue='METAN')
plt.savefig('assets/graph1_temp_h2o.png')
plt.clf()

# 2. Heatmap dubine
plt.scatter(df_cisto['GPS_LONG'], df_cisto['GPS_LAT'], c=df_cisto['DUBINA'])
plt.colorbar()
plt.savefig('assets/graph2_heatmap_depth.png')
plt.clf()

# 3. Metan scatter
colors = df_cisto['METAN'].map({1: 'red', 0: 'blue'})
plt.scatter(df_cisto['GPS_LONG'], df_cisto['GPS_LAT'], c=colors)
plt.savefig('assets/graph3_methane_scatter.png')
plt.clf()

# 4. Kandidati
kandidati = df_cisto[
    (df_cisto['METAN'] == 1) &
    (df_cisto['ORGANSKE_MOLEKULE'] == 1)
]

plt.scatter(df_cisto['GPS_LONG'], df_cisto['GPS_LAT'], alpha=0.3)
plt.scatter(kandidati['GPS_LONG'], kandidati['GPS_LAT'], marker='*', s=250, color='red')
plt.savefig('assets/scatter_plot.png')
plt.clf()

# 5. Satelitska mapa
plt.figure(figsize=(12, 8))

extent_koordinate = [
    df_cisto['GPS_LONG'].min(), df_cisto['GPS_LONG'].max(),
    df_cisto['GPS_LAT'].min(), df_cisto['GPS_LAT'].max()
]

slika = plt.imread('assets/jezero_crater_satellite_map.jpg')
plt.imshow(slika, extent=extent_koordinate, aspect='auto', alpha=0.7)

sns.scatterplot(data=df_cisto, x='GPS_LONG', y='GPS_LAT', alpha=0.3)
plt.savefig('assets/jezero_mission_map.jpg')
plt.clf()

komande = []

for _, row in kandidati.iterrows():
    komande.append({
        "id": row["ID"],
        "akcije": [
            {"tip": "NAVIGACIJA", "lat": row["GPS_LAT"], "lon": row["GPS_LONG"]},
            {"tip": "SONDIRANJE", "dubina": row["DUBINA"]},
            {"tip": "SLANJE_PODATAKA"}
        ]
    })

payload = {
    "misija": "Nexus",
    "komande": komande
}

response = requests.post("https://example.com/api", json=payload)
print(response.status_code)
