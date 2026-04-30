## Izvršni sažetak (Executive Summary)

Ovaj projekt fokusira se na analizu geoprostornih i kemijskih podataka prikupljenih u krateru Jezero na Marsu s ciljem identifikacije optimalnih lokacija za bušenje. Ulazni podaci dolaze iz dviju CSV datoteka koje sadrže GPS koordinate, fizikalna svojstva tla (temperatura, vlaga, pH) te indikatore prisutnosti metana i organskih molekula. Kroz proces čišćenja podataka, uklanjanja anomalija i vizualne analize, izdvajaju se najperspektivnije lokacije za daljnje istraživanje. Konačni cilj sustava je generiranje automatiziranog JSON navigacijskog naloga koji omogućuje robotskoj misiji precizno kretanje, sondiranje tla i slanje prikupljenih podataka prema kontrolnom sustavu.

## Metodologija obrade podataka (Data Wrangling)

Proces obrade podataka temelji se na osiguravanju točnosti i pouzdanosti analize kroz uklanjanje senzorskog šuma i nelogičnih vrijednosti. Podaci su inicijalno učitani iz dviju CSV datoteka te spojeni u jedinstveni DataFrame koristeći zajednički identifikator (`ID`), čime je omogućena korelacija geoprostornih i kemijskih mjerenja.

Primijenjeni su specifični logički uvjeti kako bi se filtrirali podaci koji odstupaju od fizički mogućih vrijednosti. Na primjer, temperature niže od -150°C ili više od 70°C smatraju se nerealnima za uvjete u krateru Jezero te su uklonjene. Također, pH vrijednosti izvan raspona 0–14 nisu kemijski validne, dok negativne vrijednosti vlage nisu fizički moguće. Ovi kriteriji definirani su kako bi se eliminirali utjecaji grešaka senzora uzrokovanih radijacijom i ekstremnim uvjetima na Marsu.

Takav pristup omogućuje dobivanje “čistog” skupa podataka (`df_cisto`) koji predstavlja pouzdanu osnovu za daljnju analizu i vizualizaciju. Uklanjanjem anomalija smanjuje se rizik od pogrešnih zaključaka i osigurava veća preciznost u identifikaciji potencijalnih lokacija za bušenje.

## Geoprostorna analiza i vizualizacija

Vizualizacija podataka predstavlja ključni dokazni sloj analize jer omogućuje intuitivno razumijevanje odnosa između različitih varijabli i njihove prostorne distribucije.

### 1. Korelacija temperature i vlage
![temp i voda](assets0/graf1_temperatura_voda.png)

Ovaj graf prikazuje odnos temperature tla i razine vlage, uz dodatnu dimenziju prisutnosti metana. Uočava se da se određene koncentracije metana pojavljuju u specifičnim kombinacijama temperature i vlage, što može ukazivati na potencijalno aktivne geokemijske procese.


### 2. Toplinska karta dubine bušenja
![karta dubine](assets0/graf2_karta_dubine.png)

Geoprostorni prikaz dubine bušenja omogućuje identifikaciju područja s većim potencijalom za dublja istraživanja. Varijacije u dubini mogu ukazivati na razlike u sastavu tla ili geološkoj strukturi.

### 3. Rasprostranjenost metana
![metan](assets0/graf3_metan.png)

Jasna podjela između pozitivnih (crveno) i negativnih (plavo) očitanja metana omogućuje brzo lociranje zona od interesa. Grupiranje pozitivnih očitanja može sugerirati lokalizirane izvore metana.

### 4. Kandidatske lokacije
![kandidat](assets0/karta_kandidata.png)

Na ovoj karti označene su ključne lokacije koje istovremeno sadrže metan i organske molekule. Takve točke predstavljaju primarne kandidate za bušenje jer imaju najveći znanstveni potencijal.

### 5. Satelitska mapa s preklopljenim podacima
![mapa karta](assets0/misijska_karta_jezero.jpg)

Završna vizualizacija integrira stvarnu satelitsku snimku s analiziranim podacima. Ključni tehnički element ove metode je korištenje parametra `extent`, koji definira granice slike u odnosu na stvarne GPS koordinate.

`extent` se računa kao:
```python
[lon_min, lon_max, lat_min, lat_max]


## Komunikacijski protokol (JSON Uplink)

Za slanje podataka koristi se JSON struktura koja sadrži listu komandi za svaku odabranu lokaciju. Svaka komanda uključuje ID lokacije i niz akcija koje robot treba izvršiti (navigacija, sondiranje i slanje podataka).

### Primjer JSON paketa
```json
{
  "misija": "Nexus",
  "komande": [
    {
      "id": 101,
      "akcije": [
        {"tip": "NAVIGACIJA", "lat": 18.4, "lon": 77.5},
        {"tip": "SONDIRANJE", "dubina": 5},
        {"tip": "SLANJE_PODATAKA"}
      ]
    }
  ]
}

Inženjerski dnevnik (Troubleshooting Log)

Tijekom razvoja analitičkog sustava pojavilo se nekoliko tehničkih problema koji su utjecali na ispravnost obrade podataka i komunikacije s vanjskim sustavom. Svaki problem je analiziran, izoliran i uspješno riješen.

---

### Problem 1: Neispravno učitavanje CSV datoteka
**Simptom:** Podaci su se učitavali s pogrešnim stupcima ili u potpunosti prazni DataFrame.

**Uzrok:** Netočno definiran separator (`sep`). Datoteke su koristile `;` umjesto zareza `,`.

**Rješenje:**
- Provjerena struktura CSV datoteka
- Ispravljeno učitavanje podataka eksplicitnim definiranjem separatora

```python
df = pd.read_csv('data/mars_lokacije.csv', sep=';')
