# Article — Station de relevage : labo OT Modbus & NIS2

Nov 20, 2026 · @Samuel

Squelette de l'article. Chaque section liste ce qu'elle doit contenir et les visuels à insérer ; il reste à rédiger le texte.

## 1. Accroche & angle

- **Pistes de titre** : « Attaquer et détecter une station de relevage : un labo OT Modbus à l'heure de NIS2 » ; variante courte et accrocheuse : « Une station d'épuration pilotée à distance en trois commandes ».
- **Hook (2-3 phrases)** : un scénario concret — une pompe coupée à distance, la bâche qui déborde, l'opérateur qui ne voit rien d'anormal à l'écran. Pas de 0-day : juste du Modbus sans authentification.
- **Positionnement, à poser dès les premières lignes** : vous êtes ingénieur dans le traitement de l'eau, vous connaissez les postes de relevage de l'intérieur. Ce n'est pas un énième lab Kali générique, c'est un regard métier sur la sécurité OT — c'est ce qui vous distingue.

## 2. Introduction : le contexte

- **Pourquoi maintenant** : NIS2 et sa transposition française font entrer les opérateurs d'eau et d'assainissement dans un périmètre de cybersécurité contraignant.
- **Le problème de fond** : l'OT (automates, SCADA) a été conçu pour la disponibilité et la sûreté du procédé, pas pour la sécurité informatique. Modbus, protocole des années 70 encore omniprésent, n'a ni authentification ni chiffrement : qui peut parler à l'automate peut le commander.
- **Ce que l'article démontre** : on monte chez soi, sans matériel industriel, une station de relevage simulée ; on l'attaque ; puis on la détecte.
- **Pour qui** : exploitants d'eau, RSSI de collectivités, et toute personne curieuse de l'OT. Terminer l'intro par une phrase annonçant le plan (labo → attaque → détection).

## 3. Le labo & la cible

- **Architecture** : OpenPLC (l'automate) et FUXA (la supervision) en Docker, Kali comme attaquant, le tout sous VirtualBox. → *insérer un schéma* : automate ↔ supervision ↔ attaquant, avec le segment réseau.
- **Reproductibilité** : insister — gratuit, zéro matériel industriel, tout en logiciel. → lien vers le dépôt GitHub (docker-compose, programme automate, scénarios).
- **La cible** : une station de relevage simulée — une bâche, deux pompes en alternance, seuils haut/bas, alarme de débordement. → *schéma du procédé*.
- **La table Modbus** : coils (pompes, mode auto) et holding registers (niveau, seuils). → *insérer la table* des registres.
- **Un extrait du programme automate** (Structured Text) pour montrer la logique de régulation et d'alternance — pas tout le code, juste le cœur.
- **Garde-fou** : préciser que tout est générique et simulé ; aucune infrastructure cliente réelle n'est décrite.

## 4. L'attaque pas à pas

- **Reconnaissance** : `nmap --script modbus-discover` → l'automate répond et s'identifie, sans aucune authentification. Montrer la sortie.
- **Lecture** : `mbpoll` lit l'état (niveau, pompes) exactement comme le ferait la supervision — un attaquant « voit » déjà tout.
- **Écriture 1, le forçage de pompe** : passer en mode manuel puis couper la pompe → la bâche déborde. → *captures FUXA avant/après* : le niveau qui grimpe, l'alarme.
- **Écriture 2, la falsification de seuil** : trafiquer le seuil haut → la régulation déraille alors que le mode reste « auto » et que l'écran paraît normal. → *capture*.
- **Le point à marteler** : l'impact est **physique**, et la deuxième attaque est **furtive** — l'opérateur voit un système sain qui dysfonctionne. C'est la différence entre casser et tromper.

## 5. Détection & défense

- **Capturer le trafic** pendant l'attaque (`tshark`, ou Zeek via Docker) → montrer la trame d'écriture.
- **La règle** : une écriture Modbus (fonctions 5/6/15/16) émanant d'une IP qui n'est pas la supervision = alerte. → *extrait de sortie* avec l'IP de l'attaquant qui ressort.
- **Monter en généralité** : segmentation IT/OT, liste blanche des maîtres Modbus autorisés, supervision **passive** du réseau (on ne touche pas aux automates en production).
- **Le réflexe OT** à transmettre : on surveille **qui** écrit et **quelle fonction**, pas le contenu applicatif. Un attaquant qui lit est discret ; un attaquant qui écrit se trahit.

## 6. Limites, portée & conclusion

- **Honnêteté sur le modèle** : un labo simplifié n'est pas une vraie station (pas de redondance, pas de sûreté matérielle câblée, physique réduite). Le dire évite les critiques faciles et renforce la crédibilité.
- **Évoquer, sans les coder, les vraies cibles plus fines** : registres de consigne (dérégulation silencieuse), temps de marche max (casse pompe si trop long, débordement/pollution si trop court). La détection se durcit d'autant sur les **registres de configuration/protection**.
- **Message aux exploitants** : ce n'est ni théorique ni hors de portée ; les contre-mesures (segmentation, supervision du trafic, démarche NIS2) existent et sont accessibles.
- **Ressources** : lien vers le dépôt GitHub (`s-robinet`), et les outils cités (OpenPLC, FUXA, mbpoll, nmap, Zeek/tshark).
- **Prudence** : labo strictement personnel ; ne jamais publier d'élément sur l'infrastructure réelle de l'employeur ou des clients.
