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