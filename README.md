# Amazon Fine Food Reviews -- Istraživanje podataka 2

Seminarski rad iz predmeta **Istraživanje podataka 2**, Matematički fakultet, Univerzitet u Beogradu.

## Autori
- Dragana Katić 91/2021 
- Tamara Šaponjić 252/2021 

## Opis projekta

Analiza skupa [Amazon Fine Food Reviews](https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews) primenom tehnika istraživanja podataka:

- **Klasifikacija** -- predikcija ocene (Score, 5 klasa) i sentimenta (binarno), nad RAW i BAL skupovima, sa TF-IDF 10k/20k i SVD redukcijom
- **Klasterovanje** -- nenadzirana analiza nad tri skupa atributa (svi atributi, samo tekst, SVD redukovani)
- **Pravila pridruživanja** -- Apriori algoritam nad korisničkim transakcijama

## Preuzimanje skupa podataka

Skup podataka **nije uključen u repozitorijum** zbog veličine (~300 MB). Potrebno ga je preuzeti ručno:

1. Kreirati nalog na [Kaggle](https://www.kaggle.com) (ako ga već nema)
2. Preuzeti skup sa: https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews
3. Raspakivati arhivu i smestiti fajl `Reviews.csv` u koren projekta:

```
amazon-food-reviews/
├── Reviews.csv        # <-- ovde
├── clustering_all.ipynb
├── ...
```

### Alternativno -- Kaggle API

```bash
pip install kaggle
kaggle datasets download -d snap/amazon-fine-food-reviews
unzip amazon-fine-food-reviews.zip
```

> Za Kaggle API potrebno je postaviti `~/.kaggle/kaggle.json` sa API tokenom (Settings → API → Create New Token).

## Instalacija zavisnosti

```bash
pip install pandas numpy scikit-learn nltk matplotlib seaborn plotly mlxtend
```

Za NLTK resurse (potrebno jednom):

```python
import nltk
nltk.download('stopwords')
nltk.download('wordnet')
nltk.download('omw-1.4')
```

## Struktura repozitorijuma

```
├── clustering_all.ipynb          # Klasterovanje -- svi atributi
├── clustering_text_only.ipynb    # Klasterovanje -- samo tekst
├── clustering_reduced.ipynb      # Klasterovanje -- SVD redukovani
├── clustering_final.ipynb        # Uporedna analiza klasterovanja
├── classification*.ipynb         # Klasifikacija (Score i Sentiment)
├── apriori.ipynb                 # Pravila pridruživanja
├── seminarski_final.tex          # LaTeX izvorni kod rada
└── README.md
```

## Pokretanje

Notebooks se pokreću redom -- svaki učitava `Reviews.csv` iz korena projekta. Klasterovanje koristi stratifikovani uzorak od 10.000 recenzija koji se generiše unutar notebooka.
