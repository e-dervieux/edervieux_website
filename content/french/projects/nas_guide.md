---
title: 'Guide pour se monter un serveur perso'
hideDate: true
draft: false
ShowToc: true
TocOpen: true
---


## À propos

Ce guide contient différentes informations concernant le montage d'un serveur perso. Je suis arrivé à ce choix de solutions principalement en farfouillant à droite à gauche, j'ai essayé de mettre des liens vers des ressources externes si vous souhaitez en savoir plus sur tel ou tel aspect. En particulier, ce guide détaille comment:

  - arriver à une solution matérielle satisfaisante pour un serveur maison
  - monter une grappe de stockage avec de la redondance
  - monter un serveur Samba qui soit accessible sur le réseau local pour y stocker ses données
  - synchroniser ses données entre cette grappe de stockage et un "cold storage" hors site
  - pouvoir accéder à votre serveur via SSH
  - monter un VPN vous permettant d'accéder à ce serveur depuis l'exterieur
  - installer un serveur multimédia Jellyfin sur ce serveur
  - servir ce serveur Jellyfin sur le web grâce à un tunnel Cloudflare

**Note :** ce guide présuppose que vous ayez une connaissance basique de la ligne de commande Linux, si ce n'est pas le cas vous pouvez trouver pléthore de ressources sur internet à ce sujet, à commencer par [ceci](https://ubuntu.com/tutorials/command-line-for-beginners). De plus, la plupart des liens hypertextes sont archivés quand c'était pertinent, voir [ici](about/links) pour plus de détails.

⚠️ **Attention** : je ne suis pas sysadmin, et je ne prétends pas détenir **là** vérité sur la meilleure façon de monter son serveur maison. En particulier, j'insiste sur le fait que les solutions présentées dans ce guide sont un compromis entre ne rien faire, et avoir une solution sécurisée digne du cador des responsables de la sécurité informatique chez IBM. J'essayerai de donner, tout au long du guide, des pistes pour aller plus loin et / où des solutions alternatives plus ou moins sécurisées. Ça reste à vous de mettre en adéquation le niveau de sécurité et de redondance de votre système d'information avec la criticité des données que vous y stockez.

## Pourquoi réinventer la roue ?

Plutôt que de suivre ce guide, il est tout à fait possible d'utiliser une des deux solutions suivantes:

  - **acheter un NAS Synology** : Synology n'est probablement pas la seule marque à faire ça, mais ils proposent des [NAS avec plusieurs baies de stockages et un logiciel clefs-en-mains](https://web.archive.org/web/20250504110217/https://www.synology.com/fr-fr/products/DS923+), notamment en ce qui concerne l'accès au NAS depuis l'extérieur. On est néanmoins limité en termes de fonctionnalités à ce que Synology a bien voulu implémenter, et peu robuste en cas de changement de politique / faillite de l'entreprise : c'est une solution propriétaire boîte noire, là où la plupart des solutions proposées ci-dessous sont Open Source et / ou possèdent des alternatives.
  - **utiliser TrueNAS Scale**: [TrueNAS Scale](https://www.truenas.com/truenas-scale/) est un système d'exploitation basé sur Debian, qui propose un certain nombre de fonctionnalités par défaut pour gérer un serveur de stockage. Là aussi on est limité par leurs solutions applicatives, mais comme c'est un Linux dessous, on peut toujours reprendre la main via le terminal.

Le but de monter son propre serveur de stockage *from scratch* est multiple. Cela permet notamment de :

  - Pouvoir utiliser la machine comme bon nous semble et installer des applicatifs qui ne seraient pas fournis avec les solutions clefs-en-main : **évolutivité**.
  - Avoir le contrôle sur toutes les étapes du processus et ne pas dépendre d'une entreprise en particulier : **résilience**.
  - Apprendre et comprendre ce qu'on fait au fur et à mesure : **pédagogie**.

À titre personnel, j'ai énormément appris en me montant ce serveur, que ce soit sur la façon dont fonctionne le système Linux ou les connexions réseau en général. C'est très formateur pour comprendre un petit peu les notions de services système, de routage, *etc.* qui sont au fondement de l'informatique et de l'internet.

# Partie I : choix de la plateforme

## Quel matériel pour un serveur perso ?

### Consommation électrique

Le serveur va tourner h24, 7j/7, la consommation électrique est donc un des principaux points de vigilance à considérer. En effet, là où une tour risque de consommer assez facilement de l'ordre de 100 W au repos (sources: [ici](https://web.archive.org/web/20250504101356/https://www.reddit.com/r/watercooling/comments/16szgmr/whats_your_idle_power_usage/?rdt=41155) et [là](https://web.archive.org/web/20250504101310/https://www.techpowerup.com/forums/threads/high-power-consumption-in-idle.314694/)), un laptop récent ou un Mac mini permettent de descendre sous la dizaine de watts (*cf.* [ici](https://web.archive.org/web/20250504101801/https://www.reddit.com/r/linuxquestions/comments/zqolh3/normal_power_consumption_for_laptop/?rdt=32814) et [là](https://web.archive.org/web/20250504101903/https://www.reddit.com/r/linux/comments/zwar52/haha_suck_on_dat_windows_finally_got_idle_power/?rdt=37144)) mais sont potentiellement (très) onéreux (même en occasion un Mac mini M*x* coûte quelques centaines d'euros). Cela dit, dans les années qui viennent, un vieux laptop ou un Mac mini deviendront des alternatives alléchantes...

En attendant, pour une solution ultra cheap, les *thin clients* (ou clients légers, en bon français), sont une option intéressante car ils ne coûtent pratiquement rien en reconditionné. Avec 16 Go de Ram et 240 Go de SSD, un Dell Wyse 5070 ne coûte que 110€ sur des sites comme Remarkt (achat en janvier 2025). Les prix sont très fluctuants, mais il y a souvent de bonnes affaires et moyen de se monter un serveur pour une centaine d'euros. L'avantage de ce genre de machine, même si le processeur est peu véloce, c'est une consommation variant entre 4 et 10 W. À titre d'information, **10 W de consommation à 20 cts du kilowatt-heure se traduisent par un coût annuel de l'ordre de la vingtaine d'euros en facture d'électricité**.

### Capacité de décodage matériel

Si vous comptez vous monter un serveur multimédia et vous servir de la fonctionnalité de streaming proposée par [Jellyfin](https://jellyfin.org/), ça peut valoir le coup d'investir dans une configuration plus musclée, capable de transcoder un flux h265 en 4K. Le hic, c'est que ça rajoute facilement quelques centaines d'euros de matos (format tour, coût de la carte graphique) et fait grimper la facture d'électricité (de l'ordre de 200€ / an pour une tour qui consomme une centaine de watts).

**Note :** si vous désactivez le [transcodage](https://web.archive.org/web/20250505104453/https://jellyfin.org/docs/general/post-install/transcoding/) dans vos paramètres Jellyfin, le flux vidéo d'origine sera transmis tel quel. Cela se traduit par une charge de travail bien moindre pour votre serveur, mais impose que le client dispose d'un matériel et / ou de codecs adéquats pour effectuer le décodage du flux vidéo. Typiquement mon DELL Wyse 5070 est à genoux (tous les cœurs utilisés à 100% et la lecture saccade) sur un flux 1080p en h265 @9.2Mbps avec le transcodage, tandis qu'une lecture sans transcodage utilise les cœurs à moins de 5%... Mais il faut que le client utilisé (PC / smartphone / télé) soit capable de décoder les flux vidéo reçus !

### Architecture du processeur

La plupart des outils utilisés dans le présent guide, ainsi que l'écrasante majorité de la documentation sur de l'administration système, existent pour un système Linux. De plus, étant donné le support un peu hasardeux pour l'instant de Linux sur les architectures ARM d'Apple, il est plutôt recommandé de partir sur une plateforme x86 standard, exit donc les Mac minis et autres MacBooks M*x* (même si la situation s'améliore, voir [ici](https://web.archive.org/web/20250508083522/https://asahilinux.org/)).

### Connectivité

Potentiellement important si vous comptez *travailler* sur des données stockées sur le serveur (pour de l'édition de vidéos par exemple), veillez à avoir une vitesse de port Ethernet suffisante. À minima un port Gigabit (~110 Mo/s réels), mais un port (ou une carte d'extension en PCI-Express) 2.5GbE voire 10GbE (~1 Go/s) peuvent valoir le coup pour [une soixantaine d'euros](https://web.archive.org/web/20250504103958/https://www.amazon.fr/TP-Link-TX401-Ethernet-ultra-faible-Compatible/dp/B08GFGG888).

### Stockage

#### Disques durs ou SSD ?

En janvier 2025, le coût au téraoctet est de l'ordre de [25€ pour du stockage sur disque dur](https://web.archive.org/web/20250504111740/https://www.amazon.fr/Seagate-IronWolf-Disque-interne-ST12000VNZ008/dp/B08147LZFD) contre [60€ pour du stockage sur SSD](https://web.archive.org/web/20250504111639/https://www.amazon.fr/Technology-Comprend-Western-Digital-Migration/dp/B0D7MLB76V). À cela s'ajoute le fait que le stockage long terme est déconseillé sur SSD ([volatilité de la donnée](https://web.archive.org/web/20250504112057/https://storedbits.com/ssd-data-retention-period-without-power/)), le disque dur reste donc l'option de prédilection pour du stockage long terme de gros volumes de données, malgré le bruit engendré (la consommation électrique n'est pas nécessairement plus importante pour un disque dur que pour un SSD, voir [ici](https://superuser.com/questions/1171866/) et [là](https://web.archive.org/web/20250504113158/https://theoverclockingpage.com/2024/02/25/review-ssd-acer-fa200-2tb-the-fastest-qlc-ssd-we-ever-tested/?lang=en)).

#### Quel type de boîtier ?

Si on s'oriente vers une solution tour, on pourra mettre les disques durs dans la tour. Si on part sur une solution laptop ou barebone, il sera nécessaire d'acheter un boîtier externe. Pas besoin que ce dernier soit RAID, car on verra plus bas que ZFS permet de se passer de ce genre de prérequis.

Cependant, les stations d'accueil apacher pour disques durs ont plutôt mauvaise réputation, d'où leur surnom de "disk fryer", ça peut donc valoir le coup mettre un peu d'argent dans une solution de type [QNAP TR-004](https://web.archive.org/web/20250504113940/https://www.qnap.com/fr-fr/product/tr-004) par exemple.

#### Combien de disques, quelle capacité ?

Ce choix est essentiellement fonction de la taille de la baie que vous avez choisie (en termes de nombre de disques), de vos moyens financiers, et de combien d'espace de stockage vous souhaitez disposer. Si vous partez sur une baie 4 disques et un RAIDZ (équivalent ZFS du RAID 5), la capacité utilisable au final sera d'environ trois fois la capacité d'un seul des disques (vous perdez 30%). Autrement dit, pour 4 disques de 12 To, vous aurez *in fine* environ 34 To de stockage effectif.

**Note :** si vous n'êtes pas familier avec la notion de RAID, lisez [ceci](https://fr.wikipedia.org/wiki/RAID_(informatique)) ou [cela](https://web.archive.org/web/20250504115209/https://eshop.macsales.com/blog/56056-a-beginners-guide-to-understanding-raid/). Pour un calculateur de capacité disponible avec RAIDZ, c'est [par ici](https://wintelguy.com/raidcalc.pl).

## Le système d'exploitation

Comme dit plus haut, pour des questions d'outils, on s'orientera vers une distribution Linux. Il y en a [pléthore](https://en.wikipedia.org/wiki/List_of_Linux_distributions), et j'ai personnellement opté pour **Lubuntu**, qui est une variante d'Ubuntu utilisant LxQt, ce qui permet d'avoir un système Debian avec les avantages que ça comporte, tout en ayant une empreinte RAM système parmi les plus réduites possible (voir [ici](https://web.archive.org/web/20250508133609/https://www.androidauthority.com/linux-distro-least-ram-3489365/) et [là](https://web.archive.org/web/20250320065859/https://www.reddit.com/r/Lubuntu/comments/1g2gmp8/general_appreciation_lubuntu_is_a_welloptimised/)).

Avoir Lubuntu permet aussi de se connecter au serveur avec un écran en cas de besoin, et de quand même bénéficier d'une interface graphique. Ça peut également être pratique si on veut se servir du serveur comme NAS **et** comme serveur multimédia relié à sa télé par exemple. Si vous comptez uniquement accéder au serveur via la ligne de commande, vous pouvez carrément installer un Ubuntu Server, qui devrait être un chouïa plus léger en termes d'usage RAM.


# Partie II : mise en place des différents services

À partir de maintenant, ce guide considère que vous avez un système Linux fonctionnel, raccordé à internet, et à un certain nombre de disques durs, qu'ils soient internes ou externes. Les commandes suivantes sont à exécuter dans le terminal (je n'ai pas rajouté `sudo` systématiquement, mais c'est souvent nécessaire) sur la machine en question, ou par SSH. On peut donc commencer par configurer un serveur SSH sur notre machine, comme ça on peut la mettre dans un coin sans avoir besoin de la raccorder à un écran / clavier / souris, et faire toute notre administration à distance.

## Créer un serveur SSH

Suivre les étapes suivantes (voir la [doc](https://web.archive.org/web/20250504214212/https://documentation.ubuntu.com/server/how-to/security/openssh-server/index.html)) :

  - Installer `open-ssh` avec `sudo apt-get install openssh-server`
  - (Re)démarrer le service avec `sudo systemctl restart ssh.service`, on peut aussi en profiter pour vérifier si le service est bien activé avec `sudo systemctl status ssh.service`, et faire un `sudo systemctl enable ssh.service` si ce n'est pas le cas.

**Notes :**

  - Pas besoin d'ouvrir le port 22 dans le pare-feu car il est ouvert par défaut normalement. Sinon `sudo ufw allow 22/tcp`.
  - Si vous avez besoin de changer la configuration par défaut, ça se fait en éditant `/etc/ssh/sshd_config` et en redémarrant le service.
  - **Attention** si jamais vous voulez pouvoir vous connecter en SSH depuis l'extérieur (internet). C'est possible de faire du [port-forwarding](https://en.wikipedia.org/wiki/Port_forwarding) pour envoyer un port de votre IP publique sur le port 22 de votre serveur, mais il est recommandé dans ce cas-là de ne pas utiliser le port 22 et de désactiver le login par mot de passe pour privilégier une solution de type [certificats de sécurité](https://web.archive.org/web/20250505100403/https://goteleport.com/blog/how-to-configure-ssh-certificate-based-authentication/).

## Stockage : grappes ZFS et état des disques

Avant toute chose, vous pouvez vous renseigner sur la notion de sauvegarde de données informatiques en général, et de [stratégie de sauvegarde 3-2-1](https://en.wikipedia.org/wiki/Backup#3-2-1_Backup_Rule) en particulier. En bref, il s'agit d'avoir 3 copies de la donnée, sur 2 supports de stockages de natures différentes, dont 1 hors site (en cas d'incendie ou autre, par exemple). D'autre part, il est aussi important de garder à l'esprit que [**le RAID n'est PAS une sauvegarde**](https://web.archive.org/web/20250428002448/https://www.raidisnotabackup.com/).

Cela étant dit, vous montez un serveur perso et les considérations susmentionnées, bien qu'idéales pour la sauvegarde des données d'une institution, ne sont pas forcément réalisable par un particulier. À titre personnel, j'ai opté pour la solution RAID (donc redondance en local) + 1 copie hors site (cold storage). Ça me semblait être un bon compromis en termes de coût / sécurité / redondance de la donnée.

### À propos du RAID

Comme dit plus haut, je vous recommande de lire [ceci](https://fr.wikipedia.org/wiki/RAID_(informatique)) ou [cela](https://web.archive.org/web/20250504115209/https://eshop.macsales.com/blog/56056-a-beginners-guide-to-understanding-raid/) pour vous familiariser avec la notion de RAID. En bref, pour du stockage, je vous recommande d'utiliser du :
  - RAID 1 (miroir) si vous ne disposez que de deux disques, mais vous perdrez la moitié de votre capacité de stockage.
  - RAID 5 (stripe) si vous disposez de trois disques ou plus, vous ne perdrez qu'un N-ième de capacité, N étant votre nombre de disques.

Pour votre stockage, vous pouvez donc opter pour un simple RAID si votre contrôleur / carte mère le permet. Pour ma part, j'ai préféré utiliser ZFS, qui dispose de certains avantages par rapport au RAID.

### À propos de ZFS

ZFS, pour Zettabyte File System, est un système de fichier qui dispose de certaines propriétés intéressantes dans le cadre d'un serveur de stockage :

  - On peut monter un nombre arbitraire de disques durs au sein de *pools* (ou [*tanks*](https://serverfault.com/questions/562564/why-are-all-the-zpools-named-tank)) de stockages, avec [différents niveaux de RAID](https://web.archive.org/web/20250323043434/https://www.raidz-calculator.com/raidz-types-reference.aspx), similaires (en gros) aux RAIDs 1 et 5.
  - ZFS dispose de la notion d'instantanés ([*snapshots*](https://web.archive.org/web/20250216204631/https://docs.oracle.com/cd/E19253-01/819-5461/gbcya/index.html)), qui permettent de capturer l'état du système à un instant *t* et d'y revenir (*rollback*) *a posteriori*.
  - ZFS est purement logiciel. Les disques durs peuvent donc être branchés sur différents ports du serveur. On peut même imaginer un pool ZFS qui utiliserait dans le même RAIDZ un HDD interne et un SSD externe connecté en USB. Plus intéressant, cela permet notamment de faire un *export* de son pool sur une machine donnée, de brancher les disques sur une autre machine, et de faire un *import* sur cette nouvelle machine. On devient donc indépendant d'un éventuel contrôleur de disque / RAID matériel (et surtout d'une défaillance de ce dernier !).
  - ZFS offre des performances comparables à un RAID matériel ([source](https://web.archive.org/web/20250508152314/https://www.krenger.ch/blog/raid-z-vs-hardware-raid-5/)).

**Fun fact :** un zettaoctet vaut 10^21 octets, soit environ 2^70 octets. ZFS peut en fait allouer des volumes de 2^128 octets, soit plus de 10^26 téraoctets ! ([source](https://en.wikipedia.org/wiki/ZFS))

### Setup final

À titre perso, j'ai opté pour :

  - une baie QNAP TR-004 pour le stockage du serveur, équipée de quatre disques de 12 To (SST12000VN0008, Seagate Ironwolf) montés en RAIDZ, pour un total d'environ 31 To de stockage utilisable.
  - une station d'accueil pour deux disques 3.5" dans laquelle j'ai monté deux disques de 12 To montés en miroir ZFS, pour un total de 12 To utilisables qui me serviront de cold storage hors site que je viendrai connecter et synchroniser périodiquement avec mon serveur.

### Création d'un pool ZFS

Une fois vos disques durs connectés, vous pouvez créer votre pool comme ceci :

`zpool create [pool name] [raid type] [device id1, device id2, etc.]`

`[raid type]` : type de RAID voulu, typiquement `raidz` pour un équivalent de RAID 5 (stripe), `mirror` pour un équivalent de RAID 1 (miroir).

`[device idx]` : identifiant unique des disques que vous souhaitez inclure dans le pool. Typiquement sur mes disques ça ressemble à `ata-ST12000VN0008-2YS101_XXXXXXXX` (où `XXXXXXXX` est un numéro de série). Pour lister tous les disques installés sur le système et récupérer ces fameux identifiants, lancer la commande `ls -lh /dev/disk/by-id/`.

**Notes :**

  - Certains tutoriels utilisent `/dev/sdx` comme identifiants de disque. [Ce n'est pas recommandé](https://web.archive.org/web/20250508160418/https://unix.stackexchange.com/questions/474371/how-do-i-create-a-zpool-using-uuid-or-truly-unique-identifier) car le `x` peut être amené à changer, tandis que l'identifiant susmentionné est vraiment propre à un matériel en particulier.
  - Une fois le pool créé, il est monté automatiquement sous `/` par défaut, mais cela peut être changé avec l'option `-m [mountpoint]`.
  - Si vous voulez déconnectez vos disques pour les stocker (dans le cas du cold storage) ou les connecter sur une autre machine, faites un `zpool export pool_name` avant (et lors d'une reconnexion, faites un `zpool import pool_name`).
  - En termes de documentation, outre le `man` des commandes sus-mentionnées, vous pouvez aller sur [la doc d'Oracle](https://web.archive.org/web/20250508155543/https://docs.oracle.com/cd/E19253-01/820-2315/index.html) qui est très complète, ainsi que sur [ce tuto](https://web.archive.org/web/20250508155709/https://ubuntu.com/tutorials/setup-zfs-storage-pool#1-overview) (basique) et [celui-ci](https://web.archive.org/web/20250508160141/https://ikrima.dev/dev-notes/homelab/zfs-for-dummies/) (plus complet).

Si tout s'est bien passé, vous avez maintenant un pool de stockage ZFS monté à la racine de votre serveur, et vous pouvez stocker vos données dessus comme sur n'importe quelle autre partition.

### Résoudre le non-montage après un redémarrage

Un problème courant avec les baies d'expansion (ou boîtiers externes) contenant plusieurs disques durs est que ces derniers ne démarrent pas tous en même temps : ils s'initialisent successivement, et il leur faut environ 5 secondes chacun pour atteindre leur pleine vitesse de fonctionnement. Si vous laissez votre configuration telle quelle et redémarrez le serveur, il est donc probable que votre pool ZFS ne se monte pas automatiquement. En effet, la commande `mount` lancée par le service ZFS au démarrage du système s'exécutera **avant** que tous les disques aient eu le temps de démarrer et d'être reconnus par le système. Dans ce cas, il faut ré-importer manuellement le pool avec un `zpool import`.

Pour éviter ce problème, il est recommandé d'ajouter un délai lors du montage du pool ZFS, afin de laisser le temps aux disques de démarrer correctement (voir [ce guide](https://web.archive.org/web/20250510133455/https://ounapuu.ee/posts/2021/02/01/how-to-fix-zfs-pool-not-importing-at-boot/)) :

  - regarder le journal zfs pour voir ce qui cloche : `journalctl -u zfs-import-cache.service -n 50`
  - éditer le fichier de configuration pour rajouter un délai avec `sudo systemctl edit zfs-import-cache.service` et rajouter les lignes suivantes (liste les disques disponibles avec `lsblk`, ce qui permet de voir lesquels manquent dans le log, puis attend 20 s)

```
[Service]
ExecStartPre=/usr/bin/lsblk
ExecStartPre=/usr/bin/sleep 20
```

Il se trouve que 20 s est un délai suffisant pour mon QNAP TR-004 équipé de quatre disques, mais il peut falloir plus en fonction de votre configuration et du temps que prennent vos disques pour atteindre leur vitesse de croisière. À noter que bien que fonctionnelle, cette solution est assez crasseuse, et il serait plus propre de sonder les disques jusqu'à ce qu'ils soient tous détectés avant de monter la grappe ZFS, plutôt que de simplement attendre une durée fixe.

### Lecture des attributs S.M.A.R.T.

Les attributs SMART pour [Self-Monitoring, Analysis and Reporting Technology](https://en.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology) sont un outil qui permet de suivre l'état de santé de ses disques. De façon générale, la lecture des attributs SMART d'un disque se fait via la commande `smartctl`. Typiquement `smartctl -a /dev/sdx` va donner l'ensemble des informations SMART pour le disque en question (plus d'infos [ici](https://web.archive.org/web/20250508162741/https://wiki.evolix.org/HowtoSmart)).

**Notes :**

  - Dans le cas du QNAP TR-004, il faut spécifier le modèle du contrôleur de disque présent dans le boîtier grâce à l'option `--device`, soit `smartctl -a /dev/sdx --device=jmb39x-q,N` (voir [ici](https://web.archive.org/web/20241208154350/https://github.com/smartmontools/smartmontools/pull/47)), où N est dans 0-3 et correspond au disque qu'on veut sonder parmi les quatre disques présents dans la baie.
  - Pour une version plus robuste qui résiste au changement de nomenclature des disques (un disque peut être `/dev/sdb` un jour et devenir `/dev/sdc` suite à un redémarrage ou un ajout / retrait de matériel), on utilisera plutôt (voir [ici](https://superuser.com/questions/1848213/predictable-device-names-in-smartd-conf-on-various-linux-like-systems)) `/dev/disk/by-id/ata-ST12[...]` qui est propre à chaque disque.
  - Le pilote du contrôleur JMB39x qui équipe le TR-004 ne supporte pas de lire les résultats des tests SMART, on n'a accès qu'aux principales métriques visibles avec `smartctl -a` (voir [là](https://web.archive.org/web/20250510062531/https://github.com/smartmontools/smartmontools/issues/64)).

On peut se satisfaire de lancer périodiquement la commande `sudo smartctl -a /dev/sdx | grep overall-health` pour avoir une vague idée de l'état de santé général d'un disque. Cependant, le flag SMART `overall-health` [peut être trompeur](https://web.archive.org/web/20250227141728/https://unix.stackexchange.com/questions/502693/smartctl-reports-overall-health-test-as-passed-but-the-tests-failed) (voir aussi [ici](https://web.archive.org/web/20250430041248/https://www.smartmontools.org/wiki/FAQ#ATAdriveisfailingself-testsbutSMARThealthstatusisPASSED.Whatsgoingon)). Pour avoir une meilleure idée de ce qui se passe et automatiser les tests, on peut utiliser `smartd`, un démon système qui peut lancer des tests de façon automatique et en notifier l'administrateur système par courriel. Pour cela, il est nécessaire de :

  - Mettre en place un moyen pour notre serveur d'envoyer des courriels.
  - Configurer `smartd` pour qu'il lance un test périodiquement et envoie un courriel d'avertissement en cas de début de défaillance sur un des disques.

#### Mise en place d'un client mail

On peut utiliser `msmtp` pour permettre à notre serveur d'envoyer des courriels. Pour cela, commencer par installer le package `msmtp` (typiquement via un `apt-get install msmtp`). Il faudra ensuite éditer le fichier de configuration situé sous `/etc/msmtprc` pour y rajouter les paramètres du compte courriel que vous souhaitez utiliser, typiquement quelque chose comme :

```
# Valeurs par défaut pour tous les comptes.
defaults
auth           on
tls            on
tls_starttls   on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp

# Exemple pour un compte Gmail
account        gmail
auth           plain
host           smtp.gmail.com
port           587
from           your.account@gmail.com
user           your.account
password       your password (voir les notes ci-dessous pour gmail !)

account default : gmail
```

Pour tester si votre configuration est bonne, la commande suivante :

```
printf "Subject:DeQuoiOnParle\n\nLeCorpsDuMessage" | sudo msmtp your.account@gmail.com
```

devrait vous envoyer un courriel avec l'objet et le corps de message défini par le `printf`.

**Notes :**

  - en installant `msmtp` il est recommandé de désactiver AppArmor en raison des [erreurs que ça peut causer](https://web.archive.org/save/https://askubuntu.com/questions/878288/msmtp-cannot-write-to-var-log-msmtp-msmtp-log).
  - la ligne `account default` permet de choisir le compte à utiliser quand rien n'est passé à l'argument `-a` de `msmtp`. Il est en effet tout à fait possible d'avoir plusieurs comptes courriels configurés dans le `msmtprc`, on choisira alors celui à utiliser avec par exemple `msmtp -a gmail` ou `msmtp -a outlook` lors de l'envoi.
  - pour un compte Gmail, et probablement pour d'autres services de messagerie, il n'est pas possible d'utiliser son mot de passe principal pour des raisons de sécurité. Google permet d'obtenir un ["mot de passe d'application"](https://web.archive.org/web/20250123175809/https://support.google.com/accounts/answer/185833?hl=fr) à la place, qui remplacera votre mot de passe dans le `msmtprc`.
  - Pensez à faire un `sudo chmod 400` sur `msmtprc`.
  - N'oubliez pas le `sudo` devant `msmtp`, pour que `msmtp` aille bien chercher sa configuration dans `/etc/msmtprc` (qui est le fichier de configuration de `root`) et puisse écrire dans son fichier de log (`/var/log/msmtp`). Si vous voulez lancer la commande sans `sudo`, il faut placer votre fichier de configuration sous `~/.msmtprc` comme indiqué [ici](https://web.archive.org/web/20250406031854/https://wiki.debian.org/msmtp) ou bien faire un `sudo chmod 666` sur `/var/log/msmtp`.

Une fois `msmtp` installé et fonctionnel, il faut également installer `mailutils` afin de disposer de la commande `mail` qui est utilisée par `smartd` et utilisera votre configuration `msmtp`. Si tout va bien, la commande

```
printf "LeCorpsDuMessage" | mail -s "DeQuoiOnParle" your.account@gmail.com
```

devrait également vous envoyer un courriel avec le corps de message définit par le `printf` et le sujet définit avec l'option `-s` (voir [la doc](https://web.archive.org/web/20250510115259/https://manpages.ubuntu.com/manpages/noble/man1/bsd-mailx.1.html)  de `mail`).

#### Configurer `smartd`

Maintenant, il faut configurer `smartd` pour envoyer un courriel si jamais il détecte un problème sur un disque. Cela se fait à travers l'édition de `/etc/smartd.conf`. Dans un premier temps, on peut laisser toutes les lignes commentées et seulement rajouter la ligne `DEVICESCAN -a -n standby -m your.account@gmail.com -M test`. Si tout est bien configuré, lancer un redémarrage de `smartd` avec `sudo systemctl restart smartd` devrait vous envoyer un mail d'information. Si jamais ce n'est pas le cas, regardez le journal de `smartd` avec `journalctl -xeu smartmontools.service` ou bien les logs de `msmtp` avec `tail /var/log/msmtp`.

Pour une explication des commandes smartd, on se référera à [la doc](https://web.archive.org/web/20250510120341/https://man.freebsd.org/cgi/man.cgi?smartd.conf%285%29) qui est très bien foutue. Pour lancer des alertes si un des disques présente un souci, j'ai personnellement ajouté pour chacun de mes disques une ligne du type :

`/dev/disk/by-id/ata-[...] -d jmb39x-q,N -a -m your.account@gmail.com -n standby
`

où N est entre 0 et 3. L'option `-a` alertera en cas d'erreurs ou de dégradation des attributs SMART, tandis que `-n standby` permet de ne pas faire tourner les disques si ceux-ci était arrêtés (en hibernation).

### Utiliser `rsync` pour synchroniser son *cold storage*

*Dans le cas où votre *cold storage* serait aussi un pool ZFS, vous pouvez sautez directement à la section suivante.*

Comme dit plus haut, c'est une bonne idée d'avoir une copie de sauvegarde de ses données hors site, qu'on viendra synchroniser périodiquement avec le pool de son serveur. Pour ce faire, on peut utiliser la commande `rsync` ([man](https://web.archive.org/web/20250509091025/https://linux.die.net/man/1/rsync)) :

`rsync -a --delete --no-i-r --info=progress2 SOURCE/ DESTINATION/`

Les options :

  - `-a` : mode archive. Active la récursivité et conserve les permissions, les dates de création et de modification, etc.
  - `--delete` : supprime du dossier de destination tous les fichiers qui ne sont pas présents dans la source.
  - `--no-i-r` : désactive la récursivité incrémentale, ce qui rend l'affichage du pourcentage de progression légèrement plus précis.
  - `--info=progress2` : donne des informations sur la progression du transfert (voir [ici](https://web.archive.org/web/20250403092628/https://unix.stackexchange.com/questions/215271/understanding-the-output-of-info-progress2-from-rsync/261139#261139)).
  - On pourrait éventuellement ajouter l'option `-z` pour compresser la donnée, mais c'est peu utile si on transfère des données déjà compressées par ailleurs (*e.g.* `.jpg`, `.mp3`, `mp4`, *etc.*).

**ATTENTION:** bien ajouter les `\` de fin ! Éventuellement, lancer la commande ci-dessus avec l'option `-n` dans un premier temps pour faire un *dry-run*.

**Note :** dans le cas d'un très long transfert, et en particulier si vous utilisez une machine distante pour lancer le transfert depuis un terminal SSH, vous pouvez utiliser `tmux` pour ne pas que le transfert soit interrompu en cas de perte de connexion. Plus d'infos [ici](https://web.archive.org/web/20250419034253/https://www.redhat.com/en/blog/introduction-tmux-linux).

## Créer un serveur Samba

Avoir un serveur de stockage avec quelques dizaines de téraoctets, c'est bien, pouvoir y accéder depuis une autre machine, c'est mieux ! Pour pouvoir accéder à votre serveur comme un disque dur réseau depuis vos autres machines, on va utiliser [Samba](https://fr.wikipedia.org/wiki/Samba_(informatique)), un protocole supporté par Windows, MacOS et Linux, pour un maximum d'inter-opérabilité.

Je ne vais pas détailler toutes les étapes ici car il y a [beaucoup](https://web.archive.org/web/20250510125753/https://documentation.ubuntu.com/server/how-to/samba/file-server/index.html) de [guides](https://web.archive.org/web/20250510130158/https://shape.host/resources/installer-samba-sur-ubuntu-22-04-un-guide-complet) qui font ça très bien sur internet, globalement il s'agit de suivre les étapes suivantes :

- Installer Samba avec `sudo apt-get install samba`
- Modifier le fichier de configuration `/etc/samba/smb.conf` pour y détailler les paramètres du partage que vous voulez mettre en place, typiquement :

```
[Nom_du_partage]
   comment = Un commentaire
   path = /chemin/vers/le/partage
   writable = yes
   browsable = yes
```

- Ajouter un utilisateur et son mot de passe à Samba avec `sudo smbpasswd -a mon_utilisateur`. Cet utilisateur peut être un utilisateur existant de la machine, ou un nouveau, créé pour l'occasion avec `sudo adduser mon_utilisateur`
- Redémarrer le service Samba avec `sudo systemctl restart smbd`. On en profitera pour vérifier que le serveur Samba démarre bien au démarrage du serveur avec `systemctl status smbd` qui devrait être à `enable` (sinon faire un `sudo systemctl enable smbd`).
- N'oubliez pas d'autoriser Samba dans le pare-feu Linux avec `sudo ufw allow samba`

Pour accéder à votre serveur depuis Windows, rentrer ensuite `\\ip.ip.ip.ip\Nom_du_partage` avec l'adresse IP de votre serveur dans la barre d'adresse de l'explorateur, et rentrez le login de l'utilisateur ajouté plus haut.

**Notes :**

  - Pour savoir quels sont les utilisateurs ajoutés à Samba, lancer `sudo smbstatus`.
  - Sous Windows, n'hésitez pas à monter votre serveur Samba comme un lecteur réseau, cela permettra qu'une lettre de lecteur (*e.g.* `Z:`) lui soit assignée.

## Monter un VPN avec OpenVPN

Monter un VPN vous permettra d'accéder de façon sécurisée à votre serveur depuis n'importe où dans le monde, c'est donc plutôt puissant. Je ne saurais que trop vous recommander de suivre [la doc](https://web.archive.org/web/20250505064141/https://openvpn.net/community-resources/how-to/) d'OpenVPN, qui détaille étape par étape la marche à suivre. Commencez par installer les outils nécessaires avec :

`sudo apt-get install openvpn openssl easy-rsa`

**Note :** l'installation que je détaille ci-dessous permet de se connecter de façon sécurisée à un réseau virtuel comprenant le serveur ainsi que les autres machines qui y sont connectées. Cela veut dire que, du point de vue du client, le serveur aura pour adresse IP `10.50.50.1` par exemple, le client aura l'adresse `10.50.50.2`, et une autre machine connectée au serveur aura l'adresse `10.50.50.3`. Ces trois machines pourront ainsi communiquer entre elles comme si elles étaient sur le même réseau. C'est tout ce que cette configuration fait. En particulier, **ça ne signifie PAS** que l'intégralité de votre traffic internet passera *via* un tunnel sécurisé pour ressortir par le point d'accès internet du serveur (contrairement à l'acceptation commune du terme de VPN, et ce que proposent des services comme NordVPN, par exemple). C'est possible de configurer OpenVPN pour faire ça cela dit, voir [ici](https://web.archive.org/web/20250505064141/https://openvpn.net/community-resources/how-to/#routing-all-client-traffic-including-web-traffic-through-the-vpn).

### Génération de clefs de sécurité

Pour se connecter à notre VPN on va utiliser des clefs de sécurité. Le principe général étant que le client ET le serveur ait chacun leur jeu de clefs, protégées par mot de passe, et que les clefs soient transmises aux clients par un canal sécurisé (clef USB par exemple, ou [chiffré par GPG](https://web.archive.org/web/20250511071055/https://stackoverflow.com/questions/55383876/encrypt-and-decrypt-files-with-password/55384047#55384047) si envoi par internet). À noter qu'avec ce schéma d'authentification, il est nécessaire de générer une nouvelle clef pour chaque nouveau client que vous voulez autoriser à se connecter à votre VPN.

Commencer par trouver ou `easy-rsa` est installé (typiquement dans `/usr/share/easy-rsa/`) et copier le dit répertoire dans votre `home` avec quelque chose comme :
`cp -r easy-rsa ~/easy-rsa`. Ensuite faire un `cp vars.example vars` et éditer `vars` pour renseigner les champs nécessaires, par exemple changer la date d'expiration des certificats et renseigner vos informations personnelles si vous le souhaitez. Autrement, vous pouvez laisser les champs vierges comme ceci :

```
set_var EASYRSA_REQ_COUNTRY	"."
set_var EASYRSA_REQ_PROVINCE	"."
set_var EASYRSA_REQ_CITY	"."
set_var EASYRSA_REQ_ORG	    "John Doe"
set_var EASYRSA_REQ_EMAIL	"thisemailadress@doesnot.exist"
set_var EASYRSA_REQ_OU		"."
set_var EASYRSA_CA_EXPIRE	365000
set_var EASYRSA_CERT_EXPIRE	36500
```

Ensuite (depuis le dossier dans lequel se trouvent le script `easy-rsa` et le fichier vars que vous venez d'éditer) :

```
./easyrsa clean-all
./easyrsa build-ca
./easyrsa build-server-full server-name
./easyrsa build-client-full client-name
./easyrsa gen-dh
```

Sachant que chaque étape de `build` vous demandera de créer un mot de passe :

  - Le mode de passe `ca`, qui sera à rentrer si vous voulez utiliser la chaîne d'authentification à nouveau (*e.g.* pour rajouter de nouveaux clients).
  - Le mot de passe `server`, qui sera utilisé par le serveur OpenVPN pour démarrer son service.
  - Le mot de passe `client`, qui sera utilisé par le client pour se connecter au serveur.

Normalement toutes les clefs générées se situent dans `easy-rsa/pki`. Ensuite mettre :

```
ca.crt
server-name.crt
server-name.key
dh.pem
```

dans `/var/local/openvpn` et faire un `chmod 400` dessus. Finalement, mettre

```
ca.crt
client-name.crt
client-name.key
```

quelque part (dans `home/openvpn` par exemple) sur votre machine cliente. Attention à bien sécuriser le transfert de ces certificats, même si le risque est limité grâce à la double protection par mot de passe.

### Avoir un fichier de conf propre pour OpenVPN

Vous pouvez partir du fichier de configuration donné en exemple [ici](https://web.archive.org/web/20250425085619/https://github.com/OpenVPN/openvpn/blob/master/sample/sample-config-files/server.conf), le modifier à votre guise, et le mettre dans `/etc/openvpn/server`. Le mien ressemble à cela en ne gardant que les lignes non commentées :

```
askpass pass
port 443
proto tcp
dev tun
ca /var/local/openvpn/ca.crt
cert /var/local/openvpn/server-name.crt
key /var/local/openvpn/server-name.key  # This file should be kept secret
dh /var/local/openvpn/dh.pem
topology subnet
server 10.50.50.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt
keepalive 10 120
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```

Les lignes importantes que j'ai modifié sont :

  - `askpass pass`, qui permet de ne pas avoir à rentrer le mot de passe du serveur à chaque fois que le service est démarré. `pass` est un fichier chmodé en 400 situé à côté du fichier de configuration dans `/etc/openvpn/server` et qui contient le mot de passe `server` définit ci-dessus.
  - `port 443` et `proto tcp` permettent de dissimuler votre VPN pour que ce dernier s'apparente à une [connexion HTTPS classique](https://fr.wikipedia.org/wiki/Hypertext_Transfer_Protocol_Secure). Ça permet de se connecter à votre VPN depuis n'importe où, tandis que le port par défaut utilisé par OpenVPN (1194) est souvent bloqué par les pare-feus publics ou d'entreprise. **Attention :** si vous voulez aussi héberger un serveur web HTTPS sur la même machine, c'est possible grâce au `port-share`, mais requiert une configuration supplémentaire, voir [ici](https://web.archive.org/web/20250318175242/https://memo-linux.com/openvpn-sur-le-port-443-partage-avec-un-serveur-web/).
  - Les fichiers de clefs dans `local` doivent aussi idéalement être chmodés en 400.
  - `server 10.50.50.0 255.255.255.0` donne l'adresse IP du sous-réseau sur lequelle sera monté votre VPN. Cela signifie que pour le client, le serveur sera disponible à l'IP `10.50.50.1`. Typiquement [la documentation](https://web.archive.org/web/20250505064141/https://openvpn.net/community-resources/how-to/#numbering-private-subnets) d'OpenVPN recommande une adresse quelque part au milieu de `10.X.X.X`.

**Note :** on trouve sur internet [des](https://web.archive.org/web/20240822022408/https://www.reddit.com/r/OpenVPN/comments/y4jdvy/configuring_the_openvpn_on_tcp_or_udp/?rdt=62515) [références](https://web.archive.org/web/20250320043004/https://www.reddit.com/r/VPN/comments/41pky3/udp_vs_tcp/) au fait que le protocole TCP serait plus lent que l'UDP et que ce dernier serait donc à privilégier pour configurer OpenVPN. Il semble que ce ne soit pas nécessairement le cas en pratique, voir [ici](https://web.archive.org/web/20250217062459/https://www.fastvue.co/sophos/blog/testing-sophos-ssl-vpn-performance-udp-or-tcp/) et [là](https://web.archive.org/web/20241213215156/https://www.top10vpn.com/fr/guides/udp-vs-tcp/). À titre personnel, depuis un réseau distant, j'avais exactement la même bande passante (environ 100 Mbps) en TCP et en UDP. J'ai donc choisi TCP pour pouvoir être sûr de pouvoir accéder à mon VPN quel que soit le réseau depuis lequel je me connecte. D'autant plus que j'envisage d'utiliser mon VPN comme un moyen d'administrer mon serveur à distance, pas pour transférer de gros volumes de données.

### Ouvrir les bons ports

Pour que votre VPN soit accessible, il faut ensuite ouvrir le port correspondant dans votre pare-feu. On peut voir quels ports sont actuellement ouverts avec
`sudo ufw status` et ajouter un port et un protocol avec `sudo ufw allow portnumber/protocol`. Dans notre cas, il faut donc faire `sudo ufw allow 443/tcp` si ce dernier n'est pas déjà ouvert.

### Mettre en place le port-forwarding sur votre box internet

Cette étape consiste à rediriger vers votre serveur les connexions entrantes qui arrivent sur votre IP publique. Typiquement, cela signifie que les connexions qui arrivent depuis l'extérieur (internet) sur l'IP `156.124.87.35:443`, où `156.124.87.35` est votre IP publique, seront redirigées par votre box internet sur l'IP locale `192.168.0.1:443`, où `192.168.0.1` est l'IP locale de votre serveur.

La configuration est propre à chaque box internet mais consiste généralement en trois opérations :

  - Si votre FAI le permet, vous pouvez demander à avoir une IP fixe, ce n'est pas vraiment lié au port-forwarding à proprement parler, mais facilitera la connexion du client.
  - Assigner à votre serveur une IP locale fixe.
  - Mettre en place une règle de redirection ([NAT](https://en.wikipedia.org/wiki/Network_address_translation)) qui redirige les requêtes externes vers votre IP publique et le port 443 en TCP vers l'IP locale fixe de votre serveur (sur le port 443 et en TCP).

**Note :** on aurait aussi tout aussi bien pu laisser le port par défaut d'OpenVPN (1194) plus haut, et faire la redirection `IP_publique:443` vers `IP_locale:1194`.

### Démarrer le VPN automatiquement avec le système

Openvpn démarre automatiquement tous les serveurs qu'il voit dans `/etc/openvpn/server`, c'est à dire tous les fichiers présents dans ce dossier qui se terminent par `.conf`. En théorie, il suffit donc de lancer : `sudo systemctl start openvpn-server@server-name` et la même chose en remplaçant `start` par `enable` pour que ce soit par défaut. Le nom après l'arobase est celui du `.conf` contenant la configuration du serveur. Pour vérifier si le serveur s'est bien lancé, `systemctl status openvpn-server@server-name` devrait renvoyer les statuts `enable` et `active (running)`.

### Se connecter côté client

Côté client, on supposera une ligne de commande Unix (Linux ou MacOS, fonctionnera avec [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) sous Windows), sinon [des clients avec GUI](https://web.archive.org/web/20250510193829/https://openvpn.net/client/) existent aussi mais je n'ai pas testé. Après installation d'OpenVPN, placer dans le même dossier les fichiers (chmodés en 400)

```
ca.crt
client.conf
client-name.crt
client-name.key
```

où le fichier `client.conf` contient les lignes suivantes

```
client
dev tun
proto tcp
remote your.public.ip.address 443
resolv-retry infinite
nobind
persist-key
persist-tun
ca /path/to/the/ca.crt
cert /path/to/the/client-name.crt
key /path/to/the/client-name.key
remote-cert-tls server
verb 3
```

Pour vous connecter à votre VPN il suffit ensuite de faire un

```
sudo openvpn /path/to/client.conf
```

qui devrait vous demander, en plus de votre mot de passe admin, le mot de passe `client` défini lors de la génération de clef pour ce client particulier. Si tout s'est bien passé, vous devriez avoir dans les logs qui apparaissent une ligne contenant `Initialization Sequence Completed`. Vous pouvez désormais accéder à votre serveur à l'adresse 10.50.50.1 normalement. Si c'est le cas, félicitations ! Votre VPN est pleinement fonctionnel.

### Bonus : mesurer la bande passante de son VPN

Si vous souhaitez mesurer la bande passante que vous avez entre votre client et votre serveur, c'est possible grâce au logiciel `iperf`. Sur votre client et votre serveur, installer `iperf` avec

```
sudo apt-get install iperf  # Sous Linux
brew install iperf          # sous MacOS
```

Ensuite, côté serveur on fera un `iperf -s` pour lancer un serveur `iperf`. Par défaut le serveur écoute sur le port 5001, donc il faut penser à ouvrir ce port avec `sudo ufw allow 5001/tcp`. Côté client, après avoir connecté le VPN, on fera `iperf -c 10.50.50.1 -r`, où `-c` indique l'adresse IP du serveur et `-r` lance un test de débit symétrique (client vers serveur, puis serveur vers client).

## Servir Jellyfin sur internet

Cette section va nous permettre de mettre en place un serveur multimédia [Jellyfin](https://web.archive.org/web/20250509030531/https://jellyfin.org/docs/), qui présente une interface similaire à celle de la plupart des services de VOD existant. On verra ensuite comment rendre ce serveur accessible depuis internet de façon sécurisée grâce aux tunnels Cloudflare.

### Installation

Globalement rien de bien compliqué, il suffit de suivre la procédure d'installation détaillée [ici](https://web.archive.org/web/20250508172635/https://jellyfin.org/docs/general/installation/linux/):

`curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash`

Normalement le service s'active automatiquement, et on peut le vérifier avec `systemctl status jellyfin` (sinon faire un `sudo systemctl enable jellyfin` et `sudo systemctl restart jellyfin`), et il est servi sur le port 8096 en local. Cela signifie qu'on peut normalement accéder au serveur depuis un navigateur internet : soit à l'adresse `localhost:8096` sur le serveur lui-même, soit à l'adresse `ip.locale.du.serveur:8096` sur une autre machine du même réseau local (à condition d'avoir ouvert le port 8096 avec `sudo ufw allow 8096/tcp`, bien sûr). Au premier lancement, une interface guide l'utilisateur pour la création d'un compte d'utilisateur admin. On peut aussi créer des utilisateurs secondaires, ajouter des dossiers à scanner pour organiser ses collections, *etc.*, voir [la documentation](https://web.archive.org/web/20250509030531/https://jellyfin.org/docs/).

### Sauvegarde

Typiquement à faire en même temps qu'une sauvegarde de votre *cold storage*, il est possible de sauvegarder votre configuration Jellyfin à tout moment, afin de garder vos paramètres, utilisateurs, collections, *etc.*. Pour ce faire, suivez la [doc](https://web.archive.org/web/20250426045259/https://jellyfin.org/docs/general/administration/backup-and-restore/).

### Obtenir un nom de domaine

Maintenant qu'on a notre serveur Jellyfin qui tourne sur le réseau local, on va voir comment le servir sur internet. Il existe plusieurs solutions pour cela, l'approche classique étant de tout configurer soi-même, avec de l'ouverture et redirection de port, un DNS, un reverse proxy, *etc.* (voir [ici](https://web.archive.org/web/20250511122939/https://lbrito.ca/blog/2020/06/free_https_home_server.html)).

Si vous êtes utilisateurs de NordVPN, ils proposent aussi une solution clefs-en-main avec Meshnet, voir [ici](https://meshnet.nordvpn.com/how-to/remote-files-media-access/access-jellyfin-media-sever-remotely). Il existe aussi [plusieurs autres solutions](https://web.archive.org/web/20250414090813/https://dev.to/jagkush/a-quick-way-to-access-your-local-server-on-the-internet-4kei) comme localhost.run, Ngrok, ou les tunnels Cloudflare. J'ai choisi cette dernière solution avant tout car elle est gratuite, tout en permettant de lier le tunnel à un nom de domaine qui vous appartient (ce que ne permettent pas les versions gratuites des comptes localhost ou Ngrok) et en restant relativement simple à mettre en œuvre.

Avant toute chose, il vous faudra un nom de domaine sur lequel servir votre site web. Il existe [pléthore de fournisseurs](https://web.archive.org/web/20250504012124/https://tld-list.com/), et les prix varient d'environ 1 à 15€ / mois en fonction de l'extension que vous choisissez. J'ai personnellement opté pour Infomaniak, qui propose un nom de domaine en `.fr` pour un prix parmi les moins cher du marché (5€ la première année, 7€ / an pour le renouvellement), tout en étant basé en Suisse et en ayant un alignement politique [assez sympa](https://web.archive.org/web/20250417215405/https://www.infomaniak.com/fr/ecologie). À noter que **l'achat d'un nom de domaine chez Infomaniak vous octroie automatiquement un hébergement web de 10 Mo sur lequel héberger votre site web statique**. Le présent site profite de cet hébergement ! Plus d'infos [ici](about/this_site). En bon point, il est possible d'activer le chiffrement et la connexion à votre site web en HTTPS avec Let's Encrypt via un simple clic dans les paramètres de votre site web.

### Router un tunnel Cloudflare

Une fois que vous avez votre nom de domaine, vous pouvez l'ajouter à votre compte (gratuit) Cloudflare. Pour ce faire, vous pouvez suivre [ce guide](https://medium.com/@fabrice_/setting-up-a-media-server-jellyfin-and-making-it-securely-accessible-from-anywhere-in-the-world-ca3b4d9dd19e), et la [doc Cloudflare](https://web.archive.org/web/20250511124347/https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/local-management/create-local-tunnel/), qui est très bien foutue.

En bref, après avoir ajouté le nom de domaine `edervieux.fr` à mes domaines Cloudflare (en suivant le tutoriel pas-à-pas depuis l'interface de Cloudflare) installer `cloudflared` sur votre serveur :

```
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt-get update && sudo apt-get install cloudflared
```

Le configurer via (depuis `~/`) :

```
cloudflared tunnel login
cloudflared tunnel create jellyfin_tunnel
cloudflared tunnel list
touch .cloudflared/config.yml
nano .cloudflared/config.yml
cloudflared tunnel list
```

Contenu du fichier `config.yml` (`a1****z0` est l'UUID du tunnel précédemment créé):

```
url: http://localhost:8096
tunnel: a1****z0
credentials-file: /home/your_username/.cloudflared/a1****z0.json
```

Et normalement la commande `cloudflared tunnel list` devrait lister votre tunnel. Ensuite, pour router votre tunnel `jellyfin_tunnel` vers un sous-domaine de votre nom de domaine principal :

```
cloudflared tunnel route dns jellyfin_tunnel jellyfin.edervieux.fr
cloudflared tunnel run jellyfin_tunnel
```

À noter que j'ai choisi [jellyfin.edervieux.fr](https://jellyfin.edervieux.fr/) mais j'aurai tout aussi bien pu mettre n'importe quel nom de sous-domaine. Aussi, rien ne vous empêche d'utiliser les tunnels Cloudflare pour servir d'autres services web locaux sur votre nom de domaine public principal !

### Démarrer le tunnel automatiquement

Normalement, pour éviter d'avoir à lancer la commande `cloudflared tunnel run jellyfin_tunnel` à chaque redémarrage de votre serveur, on est censé pouvoir ajouter un service à `systemd` comme on a fait plus haut dans ce guide pour Jellyfin ou OpenVPN. La procédure est détaillée [ici](https://web.archive.org/web/20250511125505/https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/local-management/as-a-service/linux/), mais pour l'instant ça ne fonctionne pas foire, voir [là](https://github.com/cloudflare/cloudflared/issues/1457).

**Workaround :** en attendant, on peut démarrer le tunnel automatiquement en créant nous-même un service avec [`systemd`](https://web.archive.org/web/20250304090858/https://manpages.ubuntu.com/manpages/focal/man5/systemd.service.5.html). On ne peut pas utiliser [`crontab`](https://web.archive.org/web/20250505211623/https://man7.org/linux/man-pages/man5/crontab.5.html) car la commande `cloudflared` ne termine pas et s'occupe de maintenir le tunnel ouvert. Pour ce faire, on va créer le fichier `/etc/systemd/system/clouded_jelly.service` qui contiendra les lignes suivantes (voir [ici](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6)) :

```
[Unit]
Description=Cloudflare tunneling for Jellyfin
After=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=5
User=your_username
ExecStart=cloudflared tunnel run jellyfin_tunnel

[Install]
WantedBy=multi-user.target
```

On pourra tester que le service est bien configuré en le démarrant *via* `sudo systemctl restart clouded_jelly`, et activer ce service au redémarrage avec `sudo systemctl enable clouded_jelly`.

# Conclusion

*That's all folks!* Si vous constatez des inexactitudes dans le présent guide, n'hésitez pas à me les faire remonter grâce au [formulaire de contact](contacts). Idem si vous trouvez que certains paragraphes ne sont pas très compréhensibles ou manquent de ressources externes pour être vraiment *standalone*, j'essayerai de tenir ce guide à jour et de l'améliorer au fur et à mesure en fonction de vos retours.

Bonne chance.