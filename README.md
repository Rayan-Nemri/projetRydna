# Projet Rydna

Exploration et visualisation de données avec Python et Jupyter.



Accès à la base publique des accidents corporels administrée par l’ONISR (via data.gouv.fr).

Téléchargement des fichiers CSV annuels (2005–2023), en se concentrant sur les 5 dernières années pour une volumétrie maîtrisée.

Chargement des données dans pandas pour exploration initiale.

Étude des colonnes disponibles :

date, heure, lieu, gravité, conditions météorologiques, type de route, véhicules impliqués, victimes

------

Sélection de la source
Utilisation de l’API Open-Meteo qui permet d’obtenir des données météo historiques sans clé API.

On renseigne les coordonnées du lieu de l’accident (latitude, longitude) et la date

On construit l’URL qui interroge l’API Open-Meteo avec nos paramètres:
url = f"https://archive-api.open-meteo.com/v1/archive?latitude={latitude}&longitude={longitude}&start_date={date}&end_date={date}&hourly=temperature_2m,precipitation,weathercode&timezone=Europe%2FParis"

Envoyer la requête et récupérer les données
On utilise requests pour envoyer la demande.

Si la réponse est correcte (status_code == 200), on la transforme en dictionnaire

Extraire la météo à une heure précise (ex : 12h)
On récupère les données météo correspondant à l’heure de midi 

Exemple de reponse:
Température à midi : 16.3 °C
Précipitation à midi : 0.0 mm
-------
Donnees de trafic routier:

Ces données indiquent le nombre moyen de véhicules circulant chaque jour sur chaque tronçon de route.

Le fichier utilisé contenait les mesures de trafic pour l’année 2019. 
J’ai d’abord lu ce fichier en tenant compte du bon séparateur, puis j’ai filtré les données pour ne garder que celles correspondant à l’année 2019. J’ai ensuite extrait les colonnes nécessaires, en particulier le code du département et la valeur du TMJA pour chaque tronçon.

Les lignes contenant des valeurs manquantes ont été supprimées afin de garantir la fiabilité des calculs. 
Ensuite, j’ai regroupé les données par département et calculé la moyenne du TMJA pour chaque zone. Cela permet d’obtenir une estimation globale du trafic moyen par département, en s’appuyant sur l’ensemble des tronçons mesurés.

Enfin, le résultat obtenu a été enregistré dans un nouveau fichier. 
Ce tableau représente l’exposition potentielle au risque routier à travers le volume de véhicules circulant quotidiennement dans chaque département. Il pourra être utilisé par la suite pour être croisé avec d’autres données, comme les accidents ou la météo, afin de construire une analyse complète du risque routier en France.
Dans cette étape, nous avons préparé les données d'accidents de la route en France pour l’année 2019, à partir de trois fichiers sources : caractéristiques, usagers et véhicules.

----
Tout d’abord, nous avons chargé les trois fichiers CSV contenant les informations de chaque accident, les personnes impliquées et les véhicules concernés. Les fichiers ont été lus en spécifiant le bon encodage (latin1) et le séparateur de colonnes (;), tels qu’utilisés dans les jeux de données fournis par le gouvernement.

Ensuite, nous avons nettoyé les champs de date. Les colonnes an, mois et jour ont été combinées pour former une colonne date au format standard datetime.

Nous avons également traité les heures : la colonne hrmn (heure-minute) a été convertie en un format horaire lisible (HH:MM). Cela a permis d’ajouter une colonne heure au tableau, facilitant l’analyse temporelle des accidents.

Puis, nous avons standardisé les conditions météorologiques : à partir des codes numériques présents dans la colonne atm, nous avons créé une colonne textuelle conditions_meteo avec des libellés explicites comme “Temps normal”, “Pluie légère”, ou “Neige”.

Un filtrage a ensuite été appliqué pour ne conserver que les accidents corporels, c’est-à-dire les cas impliquant au moins un blessé. Ce filtrage se base sur la colonne grav (gravité).

Une fois les données nettoyées, nous avons fusionné les trois sources. Les données des usagers ont été reliées aux caractéristiques des accidents via l’identifiant commun Num_Acc, puis les véhicules ont été ajoutés à partir de la clé composée de Num_Acc et num_veh.

Enfin, le jeu de données final, consolidé et propre, a été sauvegardé dans un nouveau fichier CSV. Ce fichier contient les informations complètes sur chaque accident, les usagers et les véhicules impliqués, dans un format prêt à l’analyse.

---- Analyse Exploratoire
Cette étape vise à mieux comprendre les données d’accidents corporels en France sur l’année 2023. L’objectif est de repérer les tendances globales (périodes à risque, lieux, conditions météo, etc.) avant d’aller vers des analyses plus poussées.

Nombre total d’accidents
Le jeu de données contient 54 822 accidents corporels enregistrés en 2023.

Répartition temporelle
Les accidents ont été répartis sur toute l’année. On observe que :

Le mois avec le plus d’accidents est Juin

Le mois avec le moins d’accidents est Fevrier

Cela peut s’expliquer par la meteo , la periode d'hiver les gens sont moins ammenees a prendre leur vehicule

Répartition géographique
Certains départements présentent un nombre plus élevé d’accidents :

Paris , Seine-Saint-Denis , plus globalement l'Ile de France et les Bouches du Rhone   arrivent en tête.

Les départements les moins touchés sont le 90 ainsi que les DOM-TOM

Cela peut refléter à la fois la densité de population et le volume de trafic.

Conditions météorologiques
La majorité des accidents ont lieu par temps normal(98463), mais on recense aussi :

14974 en pluie légère (atm = 2)

3437 en pluie forte (atm = 3)

2180 en brouillard ou fumée (atm = 7)

Cela montre que même en bonne météo, le risque reste élevé (car plus de circulation).

Conditions de luminosité
Les accidents se produisent principalement :

en pleine journee (82 988)

Mais aussi en nuit avec éclairage public éteint (19367)

Agglomération
Une forte majorité des accidents se produisent en hors agglomeration.

✅ Bilan
Cette première analyse exploratoire met en évidence plusieurs facteurs de répartition spatiale et temporelle des accidents. Elle permet de poser les bases pour les étapes suivantes du projet, notamment le croisement avec les données météo précises et les indicateurs d’exposition au risque (trafic, parc automobile...).




### Modélisation du nombre d'accidents avec un GLM Poisson

Nous avons entraîné un modèle Poisson en prenant en compte une exposition basée sur le nombre d’usagers dans chaque département.

Les variables explicatives sélectionnées ont été :
- `lum_moy`, `int_moy`, `col_moy` : conditions de circulation
- `meteo_pluie` : condition météo binaire (pluie vs sec)
- `age_moyen_usager_std`, `catv_moyen_std` : versions standardisées des variables continues

Le modèle Poisson utilise une fonction de lien logarithmique, permettant une interprétation directe des coefficients en termes de multiplicateurs de risque.

### Résultats

Les coefficients estimés permettent d’identifier les facteurs influençant significativement le taux d’accidents. Par exemple :
- Un effet positif de la météo pluvieuse pourrait indiquer une augmentation du risque d’accidents sous la pluie.
- Un coefficient négatif pour `col_moy` indiquerait une diminution du risque selon le type de collision moyen.

Ce type de modèle permet ainsi de quantifier et d'interpréter l’effet marginal de chaque variable sur le risque, tout en tenant compte de l’exposition.


### Analyse critique du modèle et pistes d’amélioration

Le modèle GLM Poisson entraîné fournit une première estimation du risque d'accident par département. Toutefois, plusieurs **limites** ont été identifiées :

#### 1. Prédicteurs non significatifs
Plusieurs variables explicatives (comme `lum_moy`, `int_moy`, `col_moy`) ne sont pas significatives statistiquement. Cela peut s'expliquer par :
- Une trop grande homogénéité entre départements (peu de variabilité).
- Une agrégation trop forte des données (moyennes peu informatives).
- Une mauvaise codification (traitement linéaire de variables qualitatives).

#### 2. Manque d’interactions
Certaines **interactions entre variables** importantes n’ont pas encore été testées. Par exemple :
- Interaction entre météo (`meteo_pluie`) et type de collision (`col_moy`)
- Interaction entre type de route (`int_moy`) et densité de trafic

Ces effets croisés pourraient expliquer des variations de risque non captées par les effets simples.

#### 3. Variables potentiellement manquantes
Le modèle pourrait être amélioré par l'ajout de variables explicatives comme :
- Densité de population ou densité de circulation
- Proportion de zones urbaines vs rurales
- Nombre moyen de kilomètres parcourus
- Conditions de visibilité
- Données comportementales (alcool, vitesse)

#### 4. Améliorations futures possibles
- Ré-encodage des variables catégorielles par classes ou regroupements plus pertinents
- Passage à un modèle binomial négatif (déjà testé) ou à un modèle avec interactions
- Sélection automatique de variables via AIC ou tests pas-à-pas
- Validation croisée (CV) pour évaluer la robustesse du modèle sur différents sous-ensembles

### Prédictions vs Réalité

La comparaison entre les nombres d'accidents observés et les prédictions du modèle montre une bonne correspondance dans la majorité des départements. Les écarts absolus restent modérés (ex. : 382 réels vs 389 prédits). Cela suggère une bonne capacité du modèle à capturer la tendance générale du risque d’accident, bien que des ajustements fins restent possibles pour certains départements.

Des métriques globales comme la MAE et la RMSE confirment cette précision globale. Une amélioration ultérieure pourrait inclure des effets d’interaction ou une segmentation par type de département.

Dans cette partie, j’ai développé un modèle de Random Forest Regressor pour prédire le nombre d’accidents. Contrairement au GLM (modèle linéaire utilisé précédemment), la Random Forest permet de capturer des relations non linéaires et des interactions complexes entre les variables.

🔧 Données et préparation
J’ai réutilisé le jeu de données préparé pour le GLM (df_modele). Celui-ci contient différentes variables explicatives moyennées ou agrégées par unité géographique (département ou commune).
Parmi les variables utilisées :

nombre_usagers (nombre total d’usagers impliqués),

catv_moyen (catégorie moyenne des véhicules),

lum_moy, atm_moy, int_moy (conditions moyennes de luminosité, météo, intersection),

et des variables géographiques encodées (dep_75, dep_69, etc.).

La cible du modèle est nombre_accidents.

⚙️ Entraînement du modèle
J’ai utilisé RandomForestRegressor de scikit-learn avec les paramètres par défaut (100 arbres, profondeur maximale automatique).
Les données ont été séparées en 80% pour l’entraînement et 20% pour le test.

📊 Résultats obtenus
Le modèle a donné les performances suivantes :

MAE (erreur absolue moyenne) : 35.41

RMSE (erreur quadratique moyenne) : 69.35

R² (coefficient de détermination) : 0.98

Ces résultats montrent une meilleure performance que le GLM utilisé précédemment. La Random Forest permet de mieux prédire le nombre d’accidents, avec une erreur plus faible et un meilleur ajustement.

🔍 Analyse des variables importantes
Le modèle permet aussi d’identifier les variables qui influencent le plus les prédictions.
D’après l’analyse des importances, la variable la plus déterminante est nombre_usagers, suivie de certaines variables géographiques (dep_75) et du type de véhicule (catv_moyen).
Les variables environnementales comme la luminosité ou la météo ont un poids beaucoup plus faible.

📈 Interprétation visuelle – PDP
Pour mieux comprendre l’effet de certaines variables, j’ai tracé des courbes de dépendance partielle.
Ces courbes montrent que :

Le nombre d’usagers a un effet très fort et croissant sur le risque d’accidents.

D’autres variables comme la météo ou l’intersection ont peu d’effet direct sur la prédiction.

✅ Conclusion
Le modèle Random Forest s’est révélé beaucoup plus performant que le GLM pour prédire le nombre d’accidents. Il met en avant l’importance du trafic (via nombre_usagers) comme facteur clé, et permet une meilleure compréhension des relations non linéaires dans les données.