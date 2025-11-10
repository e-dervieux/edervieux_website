---
title: 'Serveur Multimédia Jellyfin'
hideDate: true
draft: false
---

## En bref

**TL;DR:** merci de **NE PAS** regarder vos films en streaming. Téléchargez-les en local avant de les regarder.

## L'interface web et mes collections

J'héberge un serveur multimédia [Jellyfin](https://jellyfin.org/) à l'adresse suivante:

[https://jellyfin.edervieux.fr/](https://jellyfin.edervieux.fr/)

Si vous faites partie des heureux élus, j'ai dû vous passer un identifiant et un mot de passe pour vous connecter. L'interface est globalement très similaire à celle d'un Netflix / Amazon Video / Disney+, et autre GAFAM de l'industrie de la télédiffusion.

J'ai organisé ma médiathèque en cinq catégories :

  - **Films** pour tous les longs-métrages en live-action, incluant les mélanges de live-action et de dessin animé, comme par exemple *Roger Rabbit* ou *Cool World*.
  - **Animation** pour tous les longs-métrages d'animation, qu'elle soit 2D ou 3D.
  - **Courts-métrages** qui mélange live-action et animation en format court, souvent des choses très expérimentales.

À ces trois premières catégories, j'en ai ajouté deux autres :

  - **Captures de films** qui contient des captures d'écrans de films qui m'ont marqué, soit parce que le visuel est particulièrement bluffant, soit parce que ça correspondait à un moment fort du film que je voulais pouvoir me remémorer... N'hésitez pas à faire un tour là-dedans pour vous inspirer !
  - **Collections de films** qui contient plusieurs playlists thématiques (essentiellement sur des films de genre et / ou des origines géographiques spécifiques) qui ne sont pas ou mal traités par Jellyfin. En effet, en cliquant sur la base "Films" par exemple, on a accès à des genres taggés automatiquement en haut de l'écran (comme par exemple "Western", qui marche bien), mais il n'y avait pas de sous-genre "Giallos" ou "Sword & Sorcery" par exemple !

## Pourquoi ne pas regarder en streaming ?

J'héberge actuellement ce Jellyfin sur un matériel extrêmement modeste (Dell Wyse 5070) qui ne permet pas (ou mal) le streaming de sources en haute qualité (1080p-2160p en x265 typiquement), ces dernières mettant le serveur à genoux. Aussi, merci de **ne pas** regarder vos films en streaming dessus, mais de les télécharger en local avant de les regarder.

N'hésitez pas tout de même à vous servir de la lecture intégrée pour une prévisualisation, ou pour vérifier que le film est dans la bonne langue / a des sous-titres / une qualité suffisante à vos yeux.

## Une erreur, une demande ?

### Le DivX de 700 Mo c'est has been...

Cette collection d'environ 900 films a été rassemblée au fil des ans et à partir de sources très diverses. Aussi, certains films récupérés au début des années 2000 pendant l'âge d'or d'eMule sont dans une qualité très médiocre. Si vous voulez regarder un film et que vous trouvez la qualité vraiment pourrie, écrivez-moi et je le mettrai en meilleure qualité.

### ...sauf pour les nanards !

Un certain nombre de films, et en particulier certains films de genre des années 80 sont absolument introuvables en bonne qualité. Typiquement, l'incroyable *Alien Platoon* et autres nanards post-apo n'existent malheureusement pas (encore ?) en Blu-Ray 4K UHD.

### Une erreur d'identification

L'identification des films, c'est-à-dire la récupération de l'affiche et des autres métadonnées (réalisateur, synopsis, distribution, *etc.*), est gérée automatiquement par Jellyfin. J'ai refait une passe attentive pour corriger les mauvaises identifications et re-tagger les films proprement depuis IMDB ou affilié quand c'était possible, mais on n'est pas à l'abri d'une erreur. N'hésitez pas à me contacter si vous constatez qu'un *Cendrillon* de Disney est en fait un snuff movie de body horror des années 2010 !

### Mais où trouves-tu tout ça ?

Je ne recommanderai jamais assez [Rarelust](https://rarelust.com/) pour la quantité et la qualité de films obscurs introuvables par ailleurs qui y résident. Sinon principalement :

  - RARBG
  - The Pirate Bay
  - YggTorrent
  - RuTracker
  - KinoZal

**Note:** Malgré ce qu'on peut en lire, ces sites sont toujours pleinement fonctionnels au 1er mai 2025, mais sont peut-être bloqués dans votre pays ou par votre DNS (si c'est le cas, [un tuto ici](https://www.zdnet.fr/blogs/infra-net/pour-contourner-le-blocage-des-sites-web-il-suffit-de-changer-de-resolveur-dns-39810881.htm)).

### Comment as-tu monté ce serveur ?

Toutes les infos sur le montage de mon serveur perso en général, et la mise en place d'un serveur Jellyfin accessible depuis internet en particulier, sont disponibles dans [mon guide](projects/nas_guide#servir-jellyfin-sur-internet).