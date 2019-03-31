# Faire de la sorcellerie (en utilisant Siri avec sa PS4)

## État des lieux

De base la PS4 n'est pas compatible HomeKit (Pour des raisons évidentes -et compréhensibles- de flemme de la part de Sony), il faut donc ruser un peu. Et donc remonter de la PS jusqu'à l'iPhone.

### Matériel requis

* Une PS4 (N'importe quel modèle, tant qu'elle est en mode repos et connectée au même réseau local)
* Un Raspberry Pi (Ou n'importe quel serveur qui est disponible h24)

### Software utilisé

* Node.js
* Homebridge
* ps4-waker (dispo sur npm)


## Procédé

### 1. Ps4-waker, notre gars sur

Comme vous le savez peut-être, il existe une app nommée Second Screen, par Sony, qui permet d'allumer et d'éteindre sa PS4.

En fait on ne va pas se servir de cette app, mais du paquet [**ps4-waker**](https://www.npmjs.com/package/ps4-waker) qui imite son fonctionnement.

Tout le fonctionnement est indiqué sur la page npm du paquet, mais voici les instructions à suivre pour ceux qui voudraient s'économiser des neurones 🙄 :

1. Installer le paquet sur votre rpi/serveur :  ``` npm install ps4-waker -g ```
2. Installer l'app PS4 Second Screen sur votre smartphone ([iOS](https://itunes.apple.com/fr/app/ps4-second-screen/id1201372796), [Android](https://play.google.com/store/apps/details?id=com.playstation.mobile2ndscreen))
3. Lancer ps4-waker sans arguements
* Utilisez sudo si une erreur se rapprochant de "Impossible de lancer la PS4 virtuelle" est retournée
4. Ouvrir l'app Second Screen et sélectionner PS4-WAKER (Et pas votre PS4 si elle apparaît).
5. Le paquet lancé sur le rpi va demander un code. Il faut alors allumer la console, aller dans les réglages et accéder à "Paramètres de connexion à PlayStataion App", puis sélectionner "Ajouter un périphérique". Le code affiché est celui à entrer dans le terminal.
6. La PS4 devrait s'emballer et indiquer que PS4-WAKERMACHINCHOSE a été ajouté.

* Maintenant, exécutez ``` ps4-waker search```
* Et notez le semblant d'adresse IP (**192.168.1.xxx** dans la plupart des cas) que vous obtiendrez dans le résultat. Dans la suite du tutoriel, je vais employer ADDR pour symboliser cette IP.


Maintenant que tout est prêt, on peut procéder à des tests.

1.  Entrez ``` ps4-waker -d ADDR standby```
    * La console devrait d'éteindre
2. Entrez ``` ps4-waker -d ADDR```
    * La console devrait s'allumer


> Si la console ne se réveille pas, pensez à activer "Autoriser l'activation de la PS4 depuis le réseau".
>   * Je ne sais pas si ça aura un impact mais on ne sait jamais ¯\\_(ツ)_/¯

> Je ne vais pas faire de sav au cas par cas, experimentez par vous-même.
>   * Sauf vraiment une "mauvaise manip" vraiment grossière (Vu comment les fonctions critiques sont balisées), il ne peut pas vous arriver grand chose de mal.
>   * Et au pire, il y a toujours moyen de remettre tous les paramètres dans leur état par défaut sans toucher aux données.
>   * Dans tous les cas, je décline toute responsabilité en cas de perte de données ou endommagement du matériel... ou de guerre Indo-Pakistanaise ~


### 2. Homebridge all the things

Homebridge est un pont logiciel qui permet de faire le lien entre HomeKit (La plateforme domotique d'Apple), et basiquement strictement n'importe quoi (Même une PS2 si vous êtes déterminé).

Pour la partie d'intallation de Homebridge, je vous redirige vers le tutoriel du projet --> [Le tuto](https://github.com/nfarina/homebridge/wiki/Running-HomeBridge-on-a-Raspberry-Pi)

>   * En cas de besoin d'aide pour le fichier config.json, [un fichier modèle est dispo ici](https://github.com/nfarina/homebridge/blob/master/config-sample.json). Pensez à retirer l'accessoire et la plateforme qui sont mis à titre d'exemple justement. Si vous les laissez alors que les plugins associés ne sont pas installés, sinon vous risquez de rencontrer une erreur.
>   * Pensez à scanner le QR code envoyé par Homebridge avec l'app Camera ou l'app Maison, c'est plus pratique pour la suite

Je vous conseille aussi d'installer le paquet [homebridge-config-ui-x](https://www.npmjs.com/package/homebridge-config-ui-x) pour vous simplifier la vie quand vous aurez besoin de retravailler le fichier de configuration de homebridge (Gestion du versionnage, de l'installation et mises à jour des plugins nottamment)

> Localhost est l'adresse IP locale de la machine, donc si vous installez config-ui-x sur une machine distante, pensez à remplacer localhost par l'ip de la machine sur le réseau quand vous voulez accéder à l'interface.

Pour les besoins de ce tutoriel, il va falloir installer le plugin cmdSwitch. Il permet de créer un interrupteur exploitable par HomeKit qui fait ce qu'on lui demande quand il change d'état.

Exemples de fonctionnement :
* Quand le bouton passe à ON, alors exécuter la commande A.
* Quand le bouton passe à OFF, alors exécuter la commande B.

Il va par contre falloir l'installer à la mano (``` npm install -g homebridge-cmdswitch```), config-ui-x n'a pas celui-ci en catalogue.

Enfin, il a la version 2...

qui est une aberration à mes yeux...

et qui semble être un enfer à configurer...

donc voilà.


**Breeeeeeef**, il va falloir adapter la configuration de homebridge pour qu'il prenne en charge cmdSwitch.
Si vous n'avez pas fait les fous depuis le début du tuto, votre catégorie "Accessoires" doit ressembler à ceci après configuration :


```json
"accessories": [
    {
        "accessory": "cmdSwitch",
        "name": "Playstation 4",
        "on_cmd": "ps4-waker -d ADDR --skip-login",
        "off_cmd": "ps4-waker -d ADDR standby",
        "state_cmd": "ps4-waker search | grep -i '200 Ok'"
    }
]
```
> Si vous avez plusieurs PS4 sur le même réseau, state_cmd ne fonctionnera pas correctement. Il va falloir que vous mettiez la main dans le cambouis et fassiez de la magie à l'aide de grep, vous allez devoir vérifier que c'est bien votre PS qui est allumée (Ça implique de chercher quelque chose qui contient plusieurs lignes. Je n'ai pas le temps au moment de la modification de cette page de vous expliquer toute la procédure donc bonne chance mdr)

> Si vous voulez être directement connecté votre compte utilisateur PS lors de l'allumage, retirez simplement` ``` --skip-login```.


### 3. Conclusion time


**Normalement**, si vous avez bien tout suivi (et si vous avez eu de la chance car la technologie est toujours imprévisible mdr), vous pouvez maintenant allumer et éteindre votre console depuis Siri, l'app Maison ou via des automatisations (Genre à 9h, allumer la console).

Si vous avez des questions, des suggestions ou des idées à me faire remonter, je vous invite à m'envoyer un petit tweet à [@latech_tarti](http://twitter.com/latech_tarti) ou [@tartiflemme](http://twitter.com/tartiflemme).

Bonne journée !
