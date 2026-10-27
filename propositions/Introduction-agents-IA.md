# Développer le TP03 avec un agent IA

> Guide proposé aux étudiants. Les règles de remise et d'évaluation sont celles du sujet officiel. Ce guide accompagne la proposition de révision du TP03 et reste indépendant du fournisseur de l'agent.

## 1. Ce que vous allez apprendre

À la fin de cette introduction, vous devriez pouvoir confier une petite tâche à un agent, délimiter ses changements, relire le résultat et montrer comment vous avez vérifié son fonctionnement.

Le TP03 vous place dans une démarche entrepreneuriale : la station météo fournit la mise en contexte, puis vous développez votre propre produit et cherchez à convaincre des investisseurs. L'agent doit vous aider à concrétiser une proposition plus ambitieuse pendant les quatre semaines disponibles. Le socle commun comprend Home Assistant par MQTT, l'autodiscovery, la température et la pression, ainsi que l'humidité si le capteur la fournit. Un projet limité à ce socle est plafonné à 60 %. Un BMP280 est accepté sans mesure d'humidité et sans pénalité.

Avant de demander du code, écrivez ensemble : « Notre produit aide [clientèle] à [résoudre un problème] grâce à [service rendu]. Le jury pourra voir [scénario fonctionnel]. » Vous devez pouvoir défendre cette proposition et les hypothèses économiques qui l'accompagnent. L'agent peut aider à les examiner, mais ses affirmations sur le marché ou les clients doivent être vérifiées.

Dans le mode de travail attendu ici, l'agent intervient dans votre dépôt : il consulte les fichiers, effectue des modifications et utilise les commandes que vous autorisez. Il peut vous faire gagner du temps, mais il peut aussi ajouter une bibliothèque inutile, mal interpréter une exigence ou proposer un programme qui compile sans fonctionner sur votre montage.

Votre responsabilité est de choisir le besoin, de comprendre la solution et de vérifier les résultats. Vous pouvez demander des explications à tout moment. Demander une explication fait partie du travail.

## 2. Avant la première session

Utilisez l'outil et les modalités d'accès présentés en cours. Ouvrez le dépôt dans votre environnement de développement et vérifiez que l'agent travaille dans le bon dossier, sur une copie de travail dont l'état est connu.

Avant de lui confier une modification :

1. Identifiez la carte et l'environnement PlatformIO utilisés par l'équipe.
2. Compilez vous-mêmes le projet de départ. Gardez le message d'erreur si la compilation échoue.
3. Conservez un point de retour avec Git et examinez les changements déjà présents.
4. Repérez le fichier d'instructions reconnu par votre outil et la façon dont il le charge. Si nécessaire, fournissez explicitement vos consignes au début de la session.
5. Gardez les véritables mots de passe et clés hors du dépôt et des échanges avec l'agent. Utilisez des valeurs d'exemple dans la documentation.

Lisez les demandes d'autorisation de l'agent avant de les accepter. Pour démarrer, accordez les accès utiles au projet et aux vérifications prévues, avec validation des actions sensibles. N'activez pas une autonomie générale pour éviter de lire les demandes.

## 3. Donner un cadre court à l'agent

Les instructions du projet servent à rappeler les contraintes stables. Elles ne remplacent pas le sujet du TP et ne garantissent pas que l'agent les respectera.

Voici un exemple à adapter et à placer dans le fichier reconnu par votre outil. Les emplacements des fichiers d'instructions varient selon l'outil; un fichier nommé `AGENTS.md` n'est pas chargé automatiquement par tous les agents.

```text
Projet : TP03, prototype ESP32 en C++ avec PlatformIO.
Objectif : réaliser un produit entrepreneurial et convaincre un jury d'investir.
Clientèle : [qui utilise et qui achète].
Problème et valeur : [besoin concret, service rendu, différence avec l'existant].
Scénario produit à démontrer : [parcours complet, au-delà de la mesure].
Le projet PlatformIO se trouve dans src/.
Matériel : [modèle exact de carte], [BME280 ou BMP280], [autres composants].
Fonctionnement : mesures → Wi-Fi → MQTT → Home Assistant avec autodiscovery.
Mesures : température et pression; humidité si BME280, non exigée si BMP280.
Ne pas inventer une mesure que le capteur ne fournit pas.

Avant de modifier :
- Lire le sujet et les fichiers concernés.
- Expliquer le problème et proposer un plan court si le changement est important.
- Signaler les informations manquantes, sans inventer le câblage ou une API.

Conception :
- Respecter les responsabilités et les interfaces existantes.
- Réutiliser le code présent avant d'ajouter une abstraction.
- Garder une solution proportionnée au besoin demandé.
- Ne pas ajouter de dépendance sans expliquer son utilité et obtenir notre accord.
- Ne pas réécrire les autres modules pour une correction locale.
- Ne pas modifier les critères de vérification pour masquer un défaut.

Validation :
- Compiler depuis la racine du dépôt avec : pio run -d src
- Préciser la commande, son résultat et les limites des vérifications.
- Distinguer compilation, test sur ordinateur et essai réel sur ESP32.
- Ne pas déclarer un essai matériel réussi s'il n'a pas été effectué.

À la fin :
- Résumer les fichiers modifiés et les décisions importantes.
- Signaler les limites restantes et les essais à faire sur le montage.
- Ne pas créer de commit ni publier de changements sans notre demande.
```

La commande suppose que `src/platformio.ini` existe et que `pio` est disponible. Adaptez-la à votre installation et indiquez l'environnement avec `-e NOM` si nécessaire. Lorsqu'une dépendance doit être téléchargée ou qu'un outil manque, l'agent doit signaler cette limite.

## 4. Travailler par petites boucles

Le cycle proposé pour ce TP est : **définir → examiner → planifier → modifier → relire → vérifier → conserver**.

Le périmètre du produit doit être assez ambitieux pour répondre au TP. Décomposez ensuite sa réalisation en petites tâches vérifiables. Une petite tâche confiée à l'agent est une étape vers ce produit : gardez le scénario client complet comme objectif.

### Définir un comportement observable

Une demande trop large laisse l'agent choisir presque tout :

> Fais une station météo professionnelle avec MQTT, un écran et toutes les bonnes pratiques.

Une demande exploitable précise le besoin et ses limites :

> Lorsqu'une lecture de notre capteur échoue, je veux identifier le défaut dans les journaux et éviter de publier la valeur invalide comme une mesure. Examine le code actuel, explique où traiter ce cas et propose une correction limitée. N'ajoute pas de bibliothèque. Attends notre validation du plan avant de modifier.

Pour préparer une fonction différenciante de votre produit, partez du service rendu :

> Notre mini-serre doit maintenir les conditions choisies par son propriétaire pendant son absence. Nous avons retenu ce scénario : [capteurs pertinents, action attendue et cas de panne]. Examine notre architecture et propose les étapes pour réaliser ce scénario sans perturber l'acquisition et MQTT. Distingue ce qui peut être testé sur ordinateur de ce que nous devrons vérifier sur le montage. Ne modifie rien pour le moment.

### Examiner le plan

Avant d'accepter, posez-vous ces questions :

- Le comportement proposé répond-il à notre demande ?
- Les responsabilités restent-elles claires ?
- Les changements sont-ils limités aux parties concernées ?
- Pouvons-nous expliquer les nouvelles fonctions ou classes ?
- Avons-nous un moyen d'observer la réussite et l'échec ?

Si le plan est trop complexe, demandez une version plus simple en précisant ce qui vous semble inutile.

### Relire et vérifier

Lisez les différences dans Git avant de conserver les changements. Repérez les modifications sans rapport avec la tâche, les erreurs masquées, les valeurs magiques et les dépendances ajoutées.

Compilez, puis réalisez les essais adaptés. Pour une modification touchant le capteur, les communications ou le temps d'exécution, un essai sur le montage sera généralement nécessaire. Notez la version du code et le résultat observé. Si l'agent n'a pas accès à la carte, réalisez vous-mêmes l'essai et transmettez-lui les observations utiles.

Une modification comprise et vérifiée peut devenir un commit portant un message précis. Évitez d'accumuler plusieurs changements incompris dans un seul commit.

## 5. Une architecture propre et proportionnée

Commencez par expliquer les responsabilités de votre produit : qui lit le capteur, qui décide qu'une mesure est valide, qui prépare les données et qui les publie ?

Une séparation utile vous permet, par exemple, de modifier le format MQTT sans réécrire l'acquisition. Elle n'oblige pas à multiplier les interfaces, les fabriques ou les couches génériques.

Pour chaque ajout important, demandez-vous :

| Question | Décision à prendre |
| --- | --- |
| Quel besoin actuel cet ajout couvre-t-il ? | Reporter ou retirer les ajouts sans besoin identifié. |
| Une fonction, une classe ou une bibliothèque existante le couvre-t-elle déjà ? | Réutiliser avant de dupliquer. |
| L'abstraction rend-elle une modification réelle plus simple ? | Conserver une séparation dont le rôle peut être expliqué. |
| Quel est le coût sur l'ESP32 ? | Examiner mémoire, temporisation, fréquence des opérations et dépendances. |
| Comment vérifier le comportement ? | Définir une observation ou un test pertinent. |

Vous pouvez demander une revue ciblée :

> Examine uniquement les changements de cette tâche. Identifie les duplications, les abstractions sans besoin actuel et les chemins d'erreur masqués. Pour chaque point, cite le code concerné et explique l'effet concret. Propose la simplification minimale sans modifier les fichiers.

Ne simplifiez pas aveuglément parce que l'agent le suggère : une vérification d'erreur apparemment répétitive peut être nécessaire. Il faut pouvoir défendre ce que vous retirez comme ce que vous ajoutez.

## 6. Quand l'agent se trompe ou tourne en rond

Après plusieurs corrections sans progrès, arrêtez les modifications et revenez à des faits observables : message d'erreur exact, journaux utiles, version du code, procédure de reproduction.

Demande possible :

> N'effectue plus de modification pour le moment. Voici l'erreur et la procédure pour la reproduire. Distingue les faits observés de tes hypothèses. Propose le plus petit essai permettant de départager les causes possibles.

Ne répondez pas à chaque échec par « corrige encore » ou « réécris tout ». Vérifiez notamment le câblage, la configuration et la disponibilité du réseau avant d'accepter une réécriture logicielle.

Si une session devient confuse, commencez-en une nouvelle avec le besoin actuel, les décisions conservées, les fichiers pertinents et les résultats des essais. Évitez de recopier des pages de contexte devenu inutile.

## 7. Atelier de démarrage — environ 60 minutes

Réalisez l'atelier en binôme et échangez les rôles de pilotage et de relecture. Le projet de départ doit déjà compiler; si la chaîne matérielle n'est pas prête, utilisez une petite fonction de validation de mesures comme exercice et indiquez que les essais matériels restent à faire.

| Durée | Activité | Résultat attendu |
| --- | --- | --- |
| 10 min | Compiler le départ, repérer les modules et choisir une petite amélioration. | État initial connu et comportement attendu écrit en deux phrases. |
| 2 min | Créer un point de retour Git et noter les fichiers modifiés. | Commit initial avec message clair. |
| 10 min | Fournir les consignes et demander une analyse sans modification. | Plan court que les deux personnes savent expliquer. |
| 15 min | Faire réaliser une seule étape et relire les différences. | Changements limités, questions de compréhension résolues. |
| 15 min | Compiler et vérifier le cas normal ainsi qu'un cas d'échec. | Résultats et limites consignés; essai matériel si disponible. |
| 10 min | Expliquer le changement au partenaire et décider quoi conserver. | Commit cohérent ou retour à l'état précédent, avec justification. |

Exercice adapté au TP : faire en sorte qu'une lecture de capteur invalide ne soit pas publiée comme une mesure. Vérifiez le comportement normal et la lecture invalide, puis expliquez ce que le consommateur MQTT verra. Ne débranchez pas de composant sous tension; préparez une procédure matérielle adaptée avec l'enseignant.

À la fin, chacun doit pouvoir expliquer le changement, une décision de conception et une limite de la vérification. Aucun échec artificiel de l'agent n'est demandé.

## 8. Conserver des traces utiles sans tout archiver

Choisissez trois décisions significatives pour votre rapport. Vous pouvez utiliser le tableau suivant avec ou sans IA :

| Décision | Proposition ou options | Choix de l'équipe et raison | Vérification et trace | Apprentissage |
| --- | --- | --- | --- | --- |
| Exemple fictif : lecture invalide | Publier zéro, ne rien publier ou indiquer une indisponibilité. | Ne pas publier zéro, qui pourrait être interprété comme une mesure réelle; expliquer le signalement choisi. | Test de validation, observation sur montage et commit concerné. | Une valeur de remplacement peut masquer une panne. |
| À remplir | | | | |

Le registre des heures décrit votre travail humain : conception, échanges avec l'agent, relecture, essais, documentation. Ne transformez pas le temps de génération autonome en heures de contribution.

## 9. Vérifier avant la remise

- Le socle Home Assistant, MQTT et autodiscovery fonctionne pour les mesures disponibles sur votre capteur.
- Le scénario client différenciant fonctionne et dépasse ce socle technique.
- La publicité donne envie d'acheter le produit et correspond à ses capacités réelles.
- Le pitch explique la clientèle, la valeur, les coûts, le prix et l'utilisation du financement demandé.
- Un autre membre de l'équipe peut compiler avec les instructions du dépôt.
- Le manuel permet la configuration sans modification du code.
- Les modules et le diagramme correspondent à la version livrée.
- Les pannes prévues ont un comportement connu et des résultats documentés.
- Les preuves distinguent les essais réels des simulations et des estimations.
- Les fichiers inutilisés et les ajouts sans utilité ont été examinés.
- Les sources externes et le rôle de l'IA sont déclarés.
- Chacun peut expliquer ses décisions et la chaîne complète du produit.

## 10. Pour approfondir

Les pratiques proposées dans ce guide sont adaptées au contexte pédagogique et au matériel du TP. Deux principes sont également explicités par les documentations officielles : cadrer une tâche avant de laisser l'agent agir et vérifier ses résultats avec des moyens concrets.

- [OpenAI — bonnes pratiques pour Codex](https://learn.chatgpt.com/guides/best-practices) : pratiques éprouvée pour de meilleurs résultats.
- [Microsoft — bonnes pratiques pour utiliser l'IA dans VS Code](https://code.visualstudio.com/docs/agents/best-practices) : contexte ciblé, plan et revue des modifications.
- [Anthropic — bonnes pratiques pour Claude Code](https://code.claude.com/docs/en/best-practices) : exploration préalable et moyens de vérification.
- [GitHub — utilisation responsable des agents Copilot](https://docs.github.com/en/copilot/responsible-use/agents) : limites et responsabilité de la personne qui utilise le code produit.


Ces liens constituent des références de méthode, sans obligation de choisir ces produits. Les interfaces et modalités d'accès peuvent évoluer.
