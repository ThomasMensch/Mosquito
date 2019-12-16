# Projet SD701: Mosquito

*Hélène Danlos, Chloé Younes, Nicolas Louis, Thomas Mensch*

## Introduction

**objectif:** Le but de ce projet est de prédire le nombre de cas de dengue dans deux villes d'Amérique du Sud, San Juan au Puerto Rico et Iquitos au Pérou.

La dengue est une maladie virale due au virus de la Dengue, transmise par les piqûres de moustique. Elle se manifeste le plus souvent par un syndrome grippal (fièvre, douleurs musculaires) et peut évoluer en forme grave avec des complications sévères.

Les facteurs environmentaux (e.g., températures, précipitations humidité ...) jouent un rôle très important dans le développement et la prolifération des moustiques et sont donc des indicateurs précieux dans la prévision des épidémies.

### Description des données

Les données sont disponibles sur le site suivant:
[DengAI: Predicting Disease Spread ](https://www.drivendata.org/competitions/44/dengai-predicting-disease-spread/)

Il s'agit de deux fichiers:

 - `data/dengue_features_train.csv`: 1456 lignes x 24 colonnes
 - `data/dengue_labels_train.csv`: 1456 lignes x 4 colonnes

#### Les variables du jeu de données `dengue_features_train.csv` (en anglais)

You are provided the following set of information on a (`year`, `weekofyear`) timescale:

(Where appropriate, units are provided as a `_unit` suffix on the feature name.)

*City and date indicators*

 - `city` – City abbreviations: `sj` for San Juan and `iq` for Iquitos
 - `week_start_date` – Date given in yyyy-mm-dd format

*NOAA's GHCN daily climate data weather station measurements*

 - `station_max_temp_c` – Maximum temperature
 - `station_min_temp_c` – Minimum temperature
 - `station_avg_temp_c` – Average temperature
 - `station_precip_mm` – Total precipitation
 - `station_diur_temp_rng_c` – Diurnal temperature range
 
*PERSIANN satellite precipitation measurements (0.25x0.25 degree scale)*

 - `precipitation_amt_mm` – Total precipitation

*NOAA's NCEP Climate Forecast System Reanalysis measurements (0.5x0.5 degree scale)*

 - `reanalysis_sat_precip_amt_mm` – Total precipitation
 - `reanalysis_dew_point_temp_k` – Mean dew point temperature
 - `reanalysis_air_temp_k` – Mean air temperature
 - `reanalysis_relative_humidity_percent` – Mean relative humidity
 - `reanalysis_specific_humidity_g_per_kg` – Mean specific humidity
 - `reanalysis_precip_amt_kg_per_m2` – Total precipitation
 - `reanalysis_max_air_temp_k` – Maximum air temperature
 - `reanalysis_min_air_temp_k` – Minimum air temperature
 - `reanalysis_avg_temp_k` – Average air temperature
 - `reanalysis_tdtr_k` – Diurnal temperature range

*Satellite vegetation - Normalized difference vegetation index (NDVI) - NOAA's CDR Normalized Difference Vegetation Index (0.5x0.5 degree scale) measurements*

 - `ndvi_se` – Pixel southeast of city centroid
 - `ndvi_sw` – Pixel southwest of city centroid
 - `ndvi_ne` – Pixel northeast of city centroid
 - `ndvi_nw` – Pixel northwest of city centroid


## Pré-traitement (fichier 01)

Nous avons effectué sur les données chargées :
- recast de certaines colonnes de chaines de caractères en float, int, date
- suppression des colonnes redondantes
- remplissage des données manquantes par celles immédiatement précédentes
- unification des unités de température
- seuillage de la densité de végétation (nvdi)
- ajout de nouvelles features : prise en compte des valeurs des 4 semaines précédentes (conditions favorables au développement des moustiques)

Pour décider de la relevance des features, nous avons utilisé le calcul de corrélation de la librairie stat de spark et converti la matrice résultante pour l'afficher via seaborn (fichier 03)

## Modèle
