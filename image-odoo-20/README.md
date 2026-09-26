Ce dossier ne contient AUCUN module Odoo.

Il contient la recette pour construire l'IMAGE Docker `odoo:20`, celle
qu'Odoo n'a pas encore publiee sur Docker Hub. C'est elle que le
`FROM odoo:20` du Dockerfile principal va chercher.

Les quatre fichiers viennent tels quels du depot officiel d'Odoo :
https://github.com/odoo/docker — branche master, dossier 20.0.

Ils ne sont pas de nous et ne doivent pas etre modifies a la legere : le
Dockerfile epingle la version du paquet nightly et son empreinte SHA.
Pour passer a un paquet plus recent, relever les deux valeurs chez Odoo
plutot que de les deviner (voir le README a la racine).
