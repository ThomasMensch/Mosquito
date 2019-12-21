# Projet SD701: Mosquito

*Hélène Danlos, Chloé Younes, Nicolas Louis, Thomas Mensch*

## Introduction

**objectif:** Le but de ce projet est de prédire le nombre de cas de dengue dans deux villes d'Amérique du Sud, San Juan à Puerto Rico et Iquitos au Pérou.

La dengue, aussi appelée « grippe tropicale », est une maladie virale transmise à l’homme par des moustiques du genre Aedes.
L’incidence de la dengue progresse actuellement de manière très importante, et l’inscrit aujourd’hui aux rangs des maladies dites «ré-émergentes».
L’OMS estime à 50 millions le nombre de cas annuels, dont 500 000 cas de dengue hémorragique qui sont mortels dans plus de 2,5% des cas. Deux milliards et demi de personnes vivent dans des zones à risque.
Initialement présente dans les zones tropicales et subtropicales du monde, la dengue a désormais touché l’Europe où les 2 premiers cas autochtones ont été recensés en 2010. 
Elle se manifeste le plus souvent par un syndrome grippal (fièvre, douleurs musculaires) et peut évoluer en forme grave avec des complications sévères.

Les facteurs environmentaux (e.g., températures, précipitations humidité ...) jouent un rôle très important dans le développement et la prolifération des moustiques et sont donc des indicateurs précieux dans la prévision des épidémies.

Au regard des enjeux sanitaires que cette menace représente, un challenge, "DengueAI", a été mis en place.
Il récompense les meilleures équipes qui à partir d'un jeu de données arrivent à obtenir le modèle prédictif le plus performant.
Après avoir fait du "*benchmarking*" auprès de travaux antérieurs, l'originalité de notre approche réside dans l'utilisation de Spark pour le traitement des données.
Il nous a été aussi nécessaire de créer des séries temporelles pour lesquelles les librairies Spark ne osnt pas les plus adaptées.
Pour autant, nous avons obtenu des résultats très honorables sur DengueAI.
Les travaux sont évalués en fonction du MAE (Mean Absolute Error) des modèles soumis au challenge.

Pour nos travaux, nous avons réalisé 3 notebooks : un prétraitement des données, un pipeline avec 2 modèles distincts et enfin un dédié aux CrossCorrelation.

### Description des données

Les données sont disponibles sur le site suivant:
[DengAI: Predicting Disease Spread ](https://www.drivendata.org/competitions/44/dengai-predicting-disease-spread/)

Il s'agit de deux fichiers:

 - `data/dengue_features_train.csv`: 1456 lignes x 24 colonnes
 - `data/dengue_labels_train.csv`: 1456 lignes x 4 colonnes

#### Les variables du jeu de données `dengue_features_train.csv`

Le jeu de données est fourni sur une échelle de temps (`year`, `weekofyear`).
Malheureusement les relevés ne sont pas journaliers ce qui aurait peut-être permis une meilleure précision par la suite.

Description des données :

*Villes et dates*

 - `city` – abbreviations des villes : `sj` pour San Juan et `iq` pour Iquitos
 - `week_start_date` – Date données au format yyyy-mm-dd

*Les mesures quotidiennes de la station météorologique GHCN de la NOAA*

 - `station_max_temp_c` – température Maximum
 - `station_min_temp_c` – température Minimum
 - `station_avg_temp_c` – température Moyenne
 - `station_precip_mm` – Total des précipitations en mm
 - `station_diur_temp_rng_c` – Plage de températures diurnes
 
*PERSIANN satellite precipitation measurements (Echelle de 0.25x0.25 degrés)*

 - `precipitation_amt_mm` – Total des précipitations en mm

*Mesures de réanalyse du système de prévisions climatiques NCEP de la NOAA (Echelle de 0.5x0.5 degrés)*

 - `reanalysis_sat_precip_amt_mm` – Total précipitation
 - `reanalysis_dew_point_temp_k` – Température moyenne du point de rosée
 - `reanalysis_air_temp_k` – Température moyenne de l'air
 - `reanalysis_relative_humidity_percent` – Humidité relative moyenne
 - `reanalysis_specific_humidity_g_per_kg` – Humidité spécifique moyenne
 - `reanalysis_precip_amt_kg_per_m2` – précipitations totale
 - `reanalysis_max_air_temp_k` – temperature de l'air maximum
 - `reanalysis_min_air_temp_k` – temperature de l'air minimum
 - `reanalysis_avg_temp_k` – température moyenne de l'air
 - `reanalysis_tdtr_k` – Plage des températures diurnes

*Relevé de végétation satellitaire - Indice de végétation par différence normalisée (NDVI) - Mesures de l'indice de végétation par différence normalisée du CDR de la NOAA (échelle de 0,5x0,5 degré)*

 - `ndvi_se` – niveau de végétation au sud est de la ville
 - `ndvi_sw` – niveau de végétation au sud ouest de la ville
 - `ndvi_ne` – niveau de végétation au nord est de la ville
 - `ndvi_nw` – niveau de végétation au nord ouest de la ville


## Pré-traitement (fichier 01)

Nous avons effectué sur les données chargées :
- *recast* de certaines colonnes de chaines de caractères en *float*, *int*, *date*
- suppression des colonnes redondantes
- remplissage des données manquantes par celles immédiatement précédentes
- unification des unités de température
- seuillage de la densité de végétation (nvdi). Comparativement au paludisme, la dengue est davantage présente dans les villes alors que le paludisme est plutôt en zone rurale. A travers ce seuillage, nous avons cherché à mettre en évidence ce facteur.
- ajout de nouvelles features : prise en compte des valeurs des 4 semaines précédentes. Seules les features les plus pertinentes (températures moyennes et différents relevés de niveau de précipitation). En faisant du benchmarking et en s'appuyant sur le cycle habituel de vie du moustique et des périodes d'incubation, nous nous sommes rendus compte qu'un pas de 4 semaines était le plus efficace.

Pour décider de la pertinence des features, nous avons utilisé le calcul de corrélation de la librairie stat de spark et converti la matrice résultante pour l'afficher via seaborn (fichier 03)

## Modèle (fichier 02)

Nous avons décidé de tester deux modèles différents : un modèle de régression linéaire et un modèle de random forest.

Pour le test, nous avons gardé la dernière année de relevé de données.

La régression linéaire, après une cross-validation, donnait un MAE de `11.860`. Quant au random forest, le MAE était de `5.601`.
Le premier graphique représente les prédictions obtenues avec le random forest, le second celles obtenues avec la régression linéaire. 

## Cross Correlation (fichier 03)

Ce notebook nous a permis d'identifier quelles étaient les *features* les plus pertinentes pour construire nos modèles présentés précédemment.

