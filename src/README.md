# TP03 - Votre produit

Remplacez ce titre par le nom de votre produit, puis décrivez brièvement sa clientèle et le service rendu.

## Compilation du projet

Le code source est écrit en C++ pour ESP32 avec PlatformIO. Ce dépôt fournit un gabarit de remise : initialisez votre projet PlatformIO dans ce répertoire `src/` ou adaptez un projet du cours. Le fichier `platformio.ini` doit se trouver à côté du présent README; les fichiers C++ vont dans le sous-répertoire `src/` du projet PlatformIO.

Depuis la racine du dépôt, la commande de compilation est :

```sh
pio run -d src
```

La commande suppose que PlatformIO est installé et que `src/platformio.ini` existe. Ajoutez `-e NOM` si un environnement doit être précisé.

Complétez cette section avec le modèle exact de carte, l'environnement utilisé, les dépendances et toute autre étape nécessaire pour reproduire la compilation.

## Configuration et utilisation

Décrivez la configuration du Wi-Fi et de MQTT, l'intégration avec Home Assistant et les étapes de mise en route. Fournissez des valeurs d'exemple pour les paramètres confidentiels.

## Vérifications et limites

Indiquez comment reproduire les vérifications, où consulter les résultats et quelles limites sont connues. Référencez le manuel et la documentation technique remis dans `documents/`.
