# Amazon Reviews — Istraživanje podataka

> Seminarski rad | Matematički fakultet, Univerzitet u Beogradu  
> Katedra za računarstvo i informatiku | Februar 2026

Analiza skupa Amazon recenzija primenom tehnika istraživanja podataka:
klasifikacija, klasterovanje i pravila pridruživanja.

---

## Sadržaj

- [Pregled projekta](#pregled-projekta)
- [Struktura repozitorijuma](#struktura-repozitorijuma)
- [Skup podataka](#skup-podataka)
- [Metodologija](#metodologija)
- [Rezultati](#rezultati)
- [Instalacija i pokretanje](#instalacija-i-pokretanje)
- [Autori](#autori)

---

## Pregled projekta

Projekat analizira ~568 000 Amazon recenzija prehrambenih proizvoda sa ciljem predviđanja ocene i sentimenta, otkrivanja latentnih grupisanja recenzija i pronalaženja obrazaca kupovine.

**Tri glavna zadatka:**

| Zadatak | Cilj | Metoda |
|---|---|---|
| Klasifikacija | Predikcija ocene (1–5) i sentimenta | LinearSVC, LogReg, SGD, Random Forest, ComplementNB |
| Klasterovanje | Grupisanje recenzija bez labela | K-Means, MiniBatch K-Means, Agglomerative, Spectral, DBSCAN |
| Pravila pridruživanja | Obrasci kupovine korisnika | Apriori algoritam |

---

## Struktura repozitorijuma

```
amazon-reviews-mining/
│
├── classification.ipynb              # Klasifikacija – Score i Sentiment (osnovni notebook)
├── classification_all_.ipynb         # Klasifikacija – ALL atributi (tekst + numerički)
├── classification__text.ipynb        # Klasifikacija – Text-only atributi
├── classification_Sentiment.ipynb    # Klasifikacija – binarni cilj Sentiment
│
├── clustering/                       # Klasterovanje (po skupovima atributa)
│   ├── clustering_all.ipynb
│   ├── clustering_text.ipynb
│   └── clustering_svd.ipynb
│
├── apriori.ipynb                     # Pravila pridruživanja (Apriori)
│
├── grafici_all/                      # Vizualizacije – svi atributi
├── grafici_final/                    # Vizualizacije – finalne slike za izveštaj
│
├── best_model.joblib                 # Sačuvani best model (Score – LinearSVC)
├── best_sentiment_model.joblib       # Sačuvani best model (Sentiment – LinearSVC)
│
├── seminarski_v2.tex                 # LaTeX izveštaj
└── README.md
```

---

## Skup podataka

**Amazon Fine Food Reviews** — ~568 000 recenzija prehrambenih proizvoda sa Amazon platforme.

### Preuzimanje

Skup podataka nije uključen u repozitorijum zbog veličine (~290 MB). Potrebno ga je preuzeti ručno:

1. Otvoriti stranicu: [https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews](https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews)
2. Prijaviti se na Kaggle nalog (besplatna registracija)
3. Kliknuti **Download** — preuzima se arhiva `archive.zip`
4. Raspakivati arhivu — dobija se fajl `Reviews.csv`

### Gde smestiti fajl

Svi notebook-ovi učitavaju podatke relativnom putanjom `../../Reviews.csv`.
To znači da `Reviews.csv` mora biti smešten **dva nivoa iznad foldera sa notebookima**:

```
Reviews.csv                     ← Reviews.csv ide OVDE
└── projekat/
    └── notebooks/
        ├── classification_all_.ipynb
        ├── classification__text.ipynb
        ├── classification_Sentiment.ipynb
        ├── classification.ipynb
        └── apriori.ipynb
```

> ⚠️ Ako je struktura foldera drugačija, putanju u svakom notebooku treba ručno izmeniti:
> ```python
> data = pd.read_csv('../../Reviews.csv')  # prilagoditi po potrebi
> ```

### Opis kolona

| Kolona | Tip | Opis |
|---|---|---|
| `Score` | int (1–5) | Ocena proizvoda — ciljna promenljiva za klasifikaciju |
| `Summary` | str | Kratak naslov recenzije |
| `Text` | str | Pun tekst recenzije |
| `HelpfulnessNumerator` | int | Broj korisnika koji su recenziju ocenili kao korisnu |
| `HelpfulnessDenominator` | int | Ukupan broj glasova za recenziju |
| `ProductId` | str | Identifikator proizvoda — koristi se u pravilima pridruživanja |
| `UserId` | str | Identifikator korisnika — koristi se u pravilima pridruživanja |
| `Time` | int | Unix timestamp recenzije |

---

## Metodologija

### Preprocesiranje

- Spajanje `Summary` i `Text` u jednu kolonu `TextFull`
- Čišćenje teksta: lowercase, uklanjanje URL-ova i specijalnih karaktera
- Tokenizacija, uklanjanje stop-reči (NLTK), lematizacija
- TF-IDF vektorizacija sa bigramima (`ngram_range=(1,2)`), dve konfiguracije: **10k** i **20k** osobina

### Balansiranje

| Varijanta | Opis |
|---|---|
| RAW | Originalna distribucija klasa (dominira ocena 5: ~64%) |
| BAL | Random undersampling do veličine najmanje klase |

Primarna metrika evaluacije: **F1-macro** (zbog nebalansiranosti).

### Klasifikacija

- Stratifikovana podela 80/20 (train/test)
- 5-fold cross-validation na trening skupu
- GridSearchCV za hiperparametre najboljeg modela

### Klasterovanje

- Uzorak: 10 000 recenzija (stratifikovano po `Score`)
- Tri skupa atributa: svi atributi, samo tekst, SVD-redukovani
- Optimalni K određen Elbow metodom i Silhouette analizom

### Pravila pridruživanja

- Market-basket analiza nad korisnicima i proizvodima
- `min_support=0.01`, `min_confidence=0.3`

---

## Rezultati

### Klasifikacija — best modeli

| Cilj | Model | Accuracy | F1-macro |
|---|---|---|---|
| Score (1–5) | LinearSVC, ALL, TF-IDF 20k, RAW | 0.736 | 0.521 |
| Sentiment (0/1) | LinearSVC, ALL, TF-IDF 20k, RAW | 0.940 | 0.879 |

AUC (Sentiment, LinearSVC): **0.9669** (ALL) / **0.9656** (Text-only)

### Klasterovanje — ključni nalazi

- **Svi atributi**: optimalni K=2, Silhouette=0.579 — podela po `HelpfulnessRatio`, ne po sentimentu
- **Samo tekst**: optimalni K=2, Silhouette≈0.23 — nema jasne separacije u TF-IDF prostoru
- **SVD-redukovani**: bolji klasteri u redukovanom prostoru, interpretabilnija vizualizacija

### Pravila pridruživanja

- 601 čest skup stavki, 32 česta para, 64 pravila sa confidence ≥ 0.3
- Jaka pravila između popularnih kategorija proizvoda

---

## Instalacija i pokretanje

### Zahtevi

```
Python >= 3.9
```

### Instalacija zavisnosti

```bash
pip install -r requirements.txt
```

<details>
<summary>requirements.txt</summary>

```
numpy
pandas
scikit-learn
nltk
matplotlib
seaborn
plotly
mlxtend
joblib
jupyter
```

</details>

### NLTK resursi

Potrebno jednom pokrenuti:

```python
import nltk
nltk.download('stopwords')
nltk.download('wordnet')
nltk.download('punkt')
```

> ⚠️ Fajl `Reviews.csv` mora biti dostupan u korenu projekta pre pokretanja bilo kog notebooka.

---

## Autori

| Student | Indeks |
|---|---|
| Dragana Katić | 91/2021 |
| Tamara Šaponjić | 252/2021 |

**Mentor:** Prof. dr Mirjana Maljković Ružičić  
Katedra za računarstvo i informatiku, Matematički fakultet
