# Analyse des accidents d'aviation : quels avions assurer ?
 
![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-analyse%20de%20données-150458?logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![Données](https://img.shields.io/badge/Données-NTSB%201983--2022-1f4e79)
 
Analyse exploratoire de la base d'accidents d'aviation civile de la NTSB (National Transportation Safety Board) pour aider un **assureur aérien** à identifier les marques et modèles d'avions qui présentent, en cas d'accident, le **plus faible risque de blessures graves ou mortelles** et le **plus faible taux de destruction**.
 
---
 
## Sommaire
 
- [Contexte](#contexte)
- [Questions métier](#questions-métier)
- [Données](#données)
- [Structure du dépôt](#structure-du-dépôt)
- [Méthodologie](#méthodologie)
- [Résultats clés](#résultats-clés)
- [Recommandations](#recommandations)
- [Limites](#limites)
- [Reproduire l'analyse](#reproduire-lanalyse)
- [Auteur](#auteur)
---
 
## Contexte
 
Un assureur de compagnies aériennes souhaite savoir quels types d'appareils sont les plus sûrs **lorsqu'un accident survient**. Le périmètre fixé par le client :
 
- uniquement des **avions** de **construction professionnelle** (pas de construction amateur) ;
- des appareils susceptibles d'être **encore en service** : durée de vie maximale de 40 ans, donc accidents **à partir de 1983** ;
- des recommandations **séparées** pour les **petits avions** et les **grands avions de passagers** ;
- des conclusions **statistiquement solides**, fondées sur des échantillons suffisants.
## Questions métier
 
1. Quelles **marques** ont le plus faible taux de morts ou blessés graves et le plus faible taux de destruction ?
2. Quels **modèles** précis recommander, pour les petits et pour les grands avions ?
3. Quels **facteurs** (météo, nombre de moteurs, phase de vol) influencent la gravité d'un accident ?
## Données
 
| Fichier | Description |
|---|---|
| `AviationData.csv` | Base NTSB : 88 889 événements, 31 variables (1948-2022, essentiellement à partir de 1982) |
| `USState_Codes.csv` | Table de correspondance des États américains (fournie, non utilisée dans l'analyse finale) |
| `AviationData_clean.csv` | Données nettoyées produites par le notebook de nettoyage (56 579 accidents, 26 variables) |
 
Source : [Aviation Accident Database & Synopses (NTSB)](https://www.ntsb.gov/Pages/AviationQueryV2.aspx), version diffusée sur Kaggle.
 
## Structure du dépôt
 
```
.
├── data/
│   ├── AviationData.csv                       # données brutes
│   ├── USState_Codes.csv
│   └── AviationData_clean.csv                 # données nettoyées (sortie du notebook 1)
├── images/                                    # figures utilisées dans ce README
├── Aviation_Accidents_Cleaning_FR.ipynb       # 1. chargement, nettoyage, création des mesures
├── Aviation_Accidents_Data_Analysis_FR.ipynb  # 2. analyse exploratoire et recommandations
├── requirements.txt
└── README.md
```
 
Les notebooks s'exécutent dans l'ordre : le premier produit `data/AviationData_clean.csv`, que le second charge.
 
## Méthodologie
 
### 1. Nettoyage des données
 
| Étape | Décision | Justification |
|---|---|---|
| Période | Accidents à partir de 1983 inclus | Durée de vie maximale de 40 ans demandée par le client |
| Texte | Mise en majuscules, suppression des espaces, conversion de « UNK » et « UNKNOWN » en valeurs manquantes | « Cessna » et « CESSNA » doivent être la même marque |
| Colonnes supprimées | Latitude, Longitude, Airport.\*, Schedule, Air.carrier, FAR.Description, etc. | Entre 41 % et 86 % de valeurs manquantes, ou sans intérêt pour la question |
| `Aircraft.Category` (66 % manquant) | Imputation par le mode du même couple marque + modèle (66 % puis 15 % manquant) | Un modèle d'avion a toujours la même catégorie, la valeur est retrouvée dans la base elle-même |
| `Number.of.Engines`, `Engine.Type` | Même imputation par marque + modèle | Même logique |
| Filtre client | `Aircraft.Category == "AIRPLANE"` et `Amateur.Built == "NO"` | Avions de construction professionnelle uniquement |
| Blessures manquantes (7 à 15 %) | Remplacement par 0 | Vérifié avec `Injury.Severity` : 99,5 % des cases vides correspondent à des accidents non mortels. Dans 14 % des lignes, seule la catégorie non nulle était saisie |
| Marques (`Make`) | Suppression des mentions juridiques (CO, INC, CORP…), regroupement des variantes, séparation des biplans Stearman enregistrés sous « BOEING » | 998 marques ramenées à 797, puis 52 gardées (au moins 50 accidents), soit 95,7 % des lignes |
| Modèles | Création de `Make.Model` | Un même nom de modèle (ex. « 500 ») existe chez plusieurs marques |
 
### 2. Mesures créées
 
| Variable | Définition |
|---|---|
| `Total.Occupants` | Morts + blessés graves + blessés légers + indemnes |
| `Fatal.Serious.Rate` | (Morts + blessés graves) / `Total.Occupants` : probabilité estimée, pour une personne à bord, d'être tuée ou gravement blessée en cas d'accident |
| `Severe.Accident` | 1 si l'accident a fait au moins un mort ou un blessé grave |
| `Destroyed` | 1 si l'avion a été détruit, 0 sinon (dégâts inconnus exclus) |
 
### 3. Petits et grands avions
 
Un avion est classé **grand** si le nombre **médian** d'occupants de son modèle dépasse **20**, ou s'il a **3 moteurs ou plus** (pour bien classer les avions cargo, qui volent avec 2 ou 3 membres d'équipage seulement). On obtient 54 616 accidents de petits avions et 1 963 de grands avions.
 
### 4. Robustesse statistique
 
- Seuils minimaux : au moins 50 accidents par marque, 20 par marque dans chaque groupe, 10 par type d'avion (et une vérification avec au moins 100).
- Intervalles de confiance à 95 % affichés sur tous les diagrammes en barres.
- Tests du **khi-deux** pour chaque facteur étudié.
## Résultats clés
 
### Les deux groupes ne sont pas comparables directement
 
| Groupe | Occupants moyens | Morts / blessés graves | Avions détruits |
|---|---|---|---|
| Petits avions | 2,1 | 27,3 % | 20,6 % |
| Grands avions | 119,8 | 6,6 % | 10,9 % |
 
### Marques
 
![Taux de morts et blessés graves par marque](images/marques_blesses.png)
 
| Groupe | Marque | Accidents | Morts / blessés graves | Détruits |
|---|---|---|---|---|
| Petits | Maule | 486 | 15,8 % | 8,8 % |
| Petits | Aviat | 223 | 20,0 % | 6,3 % |
| Petits | Diamond | 158 | 21,0 % | 6,4 % |
| Petits | Cessna | 24 851 | 24,1 % | 18,7 % |
| Grands | Embraer | 94 | 5,0 % | 3,8 % |
| Grands | Bombardier | 99 | 4,6 % | 5,9 % |
| Grands | McDonnell Douglas | 268 | 2,6 % | 7,3 % |
| Grands | Boeing | 1 198 | 7,0 % | 11,8 % |
| Grands | Airbus | 180 | 6,8 % | 12,9 % |
 
![Taux de destruction par marque](images/marques_destruction.png)
 
### Modèles de petits avions (au moins 100 accidents)
 
![Petits avions les plus sûrs](images/petits_types_100.png)
 
Les Cessna 180, 185 et 170, les Piper PA-18 et PA-20 et les Cessna 172R/172S présentent à la fois peu de blessés graves (8 à 17 %) et peu de destructions (5 à 10 %).
 
### Facteurs de gravité
 
**Météo** : en conditions de vol aux instruments (IMC), la part d'occupants tués ou gravement blessés est 3 fois plus élevée (66,3 % contre 21,9 % en VMC) et l'avion est détruit 3,6 fois plus souvent (58,7 % contre 16,2 %). Test du khi-deux : p < 0,001.
 
![Effet de la météo](images/facteur_meteo.png)
 
**Nombre de moteurs (petits avions)** : les accidents de bimoteurs sont plus graves (36,7 % contre 26,0 %) et plus destructeurs (32,7 % contre 18,9 %). L'écart persiste par beau temps (28,9 % contre 21,5 % en VMC), il ne s'explique donc pas seulement par la météo.
 
![Effet du nombre de moteurs](images/facteur_moteurs.png)
 
**Phase de vol (1983-2007)** : les manœuvres (52,2 %) et la montée (41,2 %) sont les phases les plus graves, l'atterrissage (3,5 %) et le roulage (1,9 %) les moins graves, alors que l'atterrissage est la phase avec le plus d'accidents.
 
![Effet de la phase de vol](images/facteur_phase.png)
 
## Recommandations
 
**Petits avions (20 personnes ou moins)**
- Marques à privilégier : **Maule, Aviat, Diamond**. **Cessna** est une valeur sûre, légèrement meilleure que la moyenne et avec l'estimation la plus fiable.
- Modèles : **Cessna 172R / 172S**, **Cessna 180 / 185**, **Piper PA-18**.
- À éviter : Mitsubishi, Socata, Cirrus, Mooney, Beech (au-dessus de la moyenne pour les deux critères).
**Grands avions (plus de 20 personnes)**
- Marques à privilégier : **Embraer** et **Bombardier** (avions régionaux), sous la moyenne pour les blessés comme pour la destruction.
- Boeing et Airbus sont proches de la moyenne, sans différence significative entre eux.
- Modèles prometteurs : **Boeing 787, 757, 717, 747-400** (aucune destruction dans les accidents de la base, échantillons de 12 à 26 accidents à confirmer). À éviter : les anciens **Boeing 737-200**.
**Tarification et prévention**
- Intégrer l'exposition au vol aux instruments (qualification IFR des équipages, conditions d'exploitation).
- Ne pas considérer un bimoteur comme plus sûr qu'un monomoteur pour les petits avions.
- Porter une attention particulière aux usages à forte part de manœuvres à basse altitude (épandage agricole, voltige).
## Limites
 
- La base ne contient **pas d'heures de vol** : l'analyse mesure la **gravité** d'un accident, pas la **probabilité** d'en avoir un.
- Les blessures manquantes ont été remplacées par 0 après vérification. Lorsque le nombre d'indemnes était absent, le taux de blessés peut être légèrement surestimé (lignes repérées par `Uninjured.Imputed`).
- La phase de vol n'est plus renseignée à partir de 2008 : ce facteur porte sur 1983-2007.
- Les libellés de modèles sont très détaillés (ex. « 757223 » et « 757232 »), ce qui fragmente les échantillons des grands avions.
- Plusieurs marques et modèles ont peu d'accidents : se référer aux intervalles de confiance avant de conclure.
## Reproduire l'analyse
 
```bash
git clone https://github.com/<votre-compte>/<nom-du-depot>.git
cd <nom-du-depot>
pip install -r requirements.txt
jupyter notebook
```
 
1. Placer `AviationData.csv` et `USState_Codes.csv` dans `data/`.
2. Exécuter `Aviation_Accidents_Cleaning_FR.ipynb` (génère `data/AviationData_clean.csv`).
3. Exécuter `Aviation_Accidents_Data_Analysis_FR.ipynb`.
**Outils** : Python, pandas, NumPy, Matplotlib, Seaborn, SciPy, Jupyter.
 
## Auteur
 
**[Votre nom]**
[LinkedIn](https://www.linkedin.com/in/votre-profil) · [GitHub](https://github.com/votre-compte)
 
