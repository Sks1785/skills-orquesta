---
name: verify
description: Vérifie réellement qu'une modification fonctionne avant de la considérer terminée. À utiliser lors de la création ou de la modification d'applications (Flutter/Android), de configurations de serveur, Docker, Frigate, Home Assistant, n8n ou de tout projet de sécurité et d'automatisation domestique. Impose des contrôles réels et des preuves au lieu d'affirmer que « ça devrait marcher ».
---

# Skill : verify (vérifier)

## Principe
Vous n'êtes pas le vérificateur par intuition : vous êtes le vérificateur par PREUVE.
Ne dites jamais « c'est corrigé », « ça devrait marcher » ou « j'ai trouvé le problème » sans avoir exécuté un contrôle réel et vu son résultat.
Si vous ne pouvez pas exécuter quelque chose, dites-le et marquez ce point comme NON VÉRIFIÉ.

## Protocole fixe (toujours les mêmes 6 étapes, dans cet ordre)

1. **Identifier le projet et son profil.** Décidez lequel s'applique (voir « Profils » ci-dessous). Si ce n'est pas clair, posez la question une seule fois.
2. **Lire avant d'affirmer.** Ouvrez les fichiers et la configuration réels concernés. Ne supposez pas leur contenu. Si vous allez dire « la panne vient de X », montrez d'abord que vous avez lu X.
3. **Définir le critère de réussite.** Écrivez en une ou deux lignes ce que signifie « ça marche » pour cette modification (ex. : « l'application compile et l'écran des réglages s'ouvre sans erreur »).
4. **Exécuter les contrôles du profil.** Utilisez de vraies commandes (pas inventées). Si une commande échoue, incluez-la telle quelle dans le rapport.
5. **Tester le cas qui échouait avant.** Reproduisez le problème d'origine et confirmez qu'il ne se produit plus. Si vous ne pouvez pas le reproduire, dites-le.
6. **Rendre compte avec le format obligatoire** (voir ci-dessous).

## Format obligatoire du rapport

```
VÉRIFICATION
Projet : <nom/type>
Critère de réussite : <une ligne>

Contrôles exécutés :
- [OK] <commande ou test> → <résultat pertinent>
- [ÉCHEC] <commande ou test> → <erreur>
- [NON VÉRIFIÉ] <ce qui manque et pourquoi>

Ce que j'ai lu : <fichiers/configurations examinés>
Conclusion : VÉRIFIÉ / PARTIEL / NON VÉRIFIÉ
Risques en suspens : <liste courte ou « aucun connu »>
Étape suivante recommandée : <une ligne>
```

Règles du format :
- Ne marquez VÉRIFIÉ que si TOUS les contrôles critiques sont OK.
- Citez les erreurs de façon littérale et courte ; ne les résumez pas de mémoire.
- Séparez ce qui a été contrôlé de ce qui a été supposé. Le supposé s'écrit « je suppose », jamais comme un fait.

## Profil A : applications (Flutter / Android)

Contrôles, du moins coûteux au plus coûteux (arrêtez-vous au premier qui échoue) :
1. `flutter analyze` → aucune erreur (listez les avertissements).
2. `flutter test` → tous les tests passent. S'il n'y a pas de tests pour ce qui a changé, dites-le et proposez-en un.
3. `flutter build apk --debug` (ou `./gradlew assembleDebug` en Android natif) → ça compile.
4. Si un émulateur/appareil est disponible : `flutter run` ou installer l'APK et tester le parcours concerné.
5. Examiner `adb logcat` (filtré sur le paquet de l'application) à la recherche d'exceptions à l'ouverture de l'écran modifié.

Compléments selon le cas :
- Changements de dépendances : `flutter pub get` sans conflit ; vérifiez la version minimale du SDK.
- Changements de permissions ou de notifications : examinez `AndroidManifest.xml` et testez sur un appareil avec un Android récent.
- Avant publication sur Google Play : build de release signé et vérification que `versionCode` a été incrémenté.
- Si la modification touche au stockage de données : testez avec des données existantes (migration), pas seulement avec une installation propre.

## Profil B : sécurité et automatisation domestique

Règle d'or : **d'abord en lecture seule**. Ne modifiez pas la configuration en production sans (a) sauvegarde, (b) confirmation explicite de l'utilisateur et (c) plan de retour arrière.

Contrôles typiques (adaptez-les à l'installation réelle ; ne supposez pas qu'une commande existe, vérifiez-le) :
- **Docker / docker-compose :** `docker compose config` (valide la syntaxe), `docker compose ps` (état), `docker logs --tail 100 <conteneur>` (erreurs récentes). Examinez les chemins de volumes et les ports avant de démarrer quoi que ce soit.
- **Frigate :** validez le fichier de configuration avec l'outil propre à Frigate s'il existe ; vérifiez dans les journaux que chaque caméra se connecte ; confirmez que la détection est active et que les enregistrements sont écrits sur disque. Contrôlez l'usage CPU si la détection se fait par CPU.
- **Caméras :** confirmez que le flux RTSP répond (par exemple avec `ffprobe` sur l'URL, sans afficher les identifiants dans le rapport). Examinez le budget PoE du switch (somme de la consommation des caméras face à la puissance totale).
- **Home Assistant :** lancez la vérification de configuration avant de redémarrer ; ensuite, examinez le journal à la recherche d'erreurs d'intégrations.
- **n8n ou autres automatisations :** exécutez le flux en mode test avec des données d'exemple et contrôlez la sortie de chaque nœud ; confirmez le fuseau horaire et les identifiants.
- **NAS, réseau et onduleur (UPS) :** contrôlez la connectivité (`ping`, ports ouverts), l'espace disque libre et l'état de l'onduleur (par exemple avec `upsc` s'il utilise NUT). Après une coupure de courant simulée ou réelle, confirmez que l'arrêt ordonné fonctionne.
- **Sécurité :** n'exposez pas les caméras, les panneaux ni le NAS à Internet sans authentification forte et, de préférence, un VPN. Examinez les ports ouverts et les alertes du système lui-même. N'incluez jamais de mots de passe, jetons ni URL avec identifiants dans le rapport.

Pour toute modification du réseau ou du pare-feu : vérifiez avant d'appliquer que vous ne perdrez pas l'accès (gardez une voie de retour).

## Que faire quand on ne peut pas vérifier
- Dites exactement ce que vous n'avez pas pu contrôler et pourquoi (pas d'accès à l'appareil, pas de réseau, pas de permissions).
- Proposez la commande ou le test concret que l'utilisateur peut exécuter et le résultat attendu.
- Marquez la conclusion PARTIEL ou NON VÉRIFIÉ. Ne comblez pas les lacunes avec des suppositions.

## Anti-modèles (interdits)
- Dire « ça marche maintenant » sans rien avoir exécuté.
- Donner la cause d'une panne sans avoir lu le code ou le journal pertinent.
- Inventer des noms de commandes, d'options, de fichiers ou de ports. Si vous n'êtes pas sûr, vérifiez ou demandez.
- Changer plusieurs choses à la fois : une modification, une vérification.
- Cacher des contrôles échoués dans le rapport.

## Maintenance de cette skill
Chaque fois qu'un défaut passe entre les mailles (quelque chose de « vérifié » qui échoue ensuite), ajoutez ici un nouveau contrôle qui l'aurait détecté. Cette liste s'améliore avec les erreurs réelles.
