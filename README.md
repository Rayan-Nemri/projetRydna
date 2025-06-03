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
