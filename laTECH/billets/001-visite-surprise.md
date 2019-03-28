# Visite surprise

Concrètement, j'ai eu la surprise en rentrant chez moi que homebridge ne répondait plus (Impossible d'éteindre mon pc à la voix. Roooh). Je suis donc allé voir si le processus était tué sur le Pi et là, *SURPRIIIIISE* ! Je ne pouvais plus me connecter en ssh car les mots de passe étaient considérés comme incorrects.

Breeeef, ayant fait une estimation de la situation (Aka, quelqu'un s'est probablement introduit dedans et a fait comme chez lui), je débranche mon pi de ma box et le rebranche sur mon bureau, histoire de me log en physique et donc de voir si c'est pas un soucis au niveau du protocole SSH.

Plot twist : Impossible de se log : Un script s'exécute en boucle au démarrage, ce qui est pas ouf. En plus il n'est pas de moi.

Breeef, du coup je boote en sh au lieu de bash (thanks le **init=/bin/sh** dans les instructions de boot) qui, si j'ai bien compris, est une sorte de shell en single user mode (et donc en root). 
Je regarde dans etc/rc.local et là *RESURPRIIIIISE* ! Il y avait dedans une instruction appelant un script */opt/machinchose* (J'ai plus le nom en mémoire et je ne peux pas le retrouver au moment de la rédaction), je l'ai cat et ô surprise (oui, encore), des lignes mentionnent un changement de mot de passe pour les sessions utilisateur mais également des interactions ssh si j'ai bien suivi.

À noter qu'il y a un autre plot twist : Le disque système est en read only quand on utilise sh. Trop bien. Après avoir passé quelques dizaines de minutes avec des amis à voir comment faire pour extraire les données du pi,  en modifier le /etc/shadow (qui permet d'effectuer un reset du mot de passe si on remplace la sorte de hash associé au compte par un \*), puis pour réinjecter les nouvelles données, j'ai fini par trouver comment remonter le disque système en rw (read+write pour les moldus).

Ayant toutes les clés en main, je l'ai joué stratégique : J'ai changé les mots de passe de tous les comptes du pi, et mis en commentaire la ligne de commande problématique de rc.local histoire de désamorcer le problème et de pouvoir remettre une tête dans le script dans le futur (Pour la science, je vous droppe le lien pastebin en dessous btw).

J'ai aussi renforcé ma protection dans du côté du vecteur d'attaque utilisé : Il n'y a désormais plus qu'un de ses ports d'accessible depuis internet.


Voilà voilà pour ce détail qui m'aura fait perdre plusieurs heures inutilement à cause d'une négligence de sécurité. Leçon retenue !


[Et voici le lien vers le pastebin du code sympa](http://gg.gg/scriptpasinoffensif)


Et bonne journée ! (Et sans intrusions j'espère)


Tarti
