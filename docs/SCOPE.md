# Périmètre — trafficBot (Systèmes intelligents autonomes)

Un TurtleBot3 Burger équipé d'une caméra suit un circuit routier miniature et respecte en temps réel le code de la route : stop (arrêt 3 s), sens interdit, panneaux d'interdictions de tourner à gauche et à droite, et feu tricolore connecté. Bloqué par un mur, il fait demi-tour. Ce repo couvre la chaîne ROS2 perception → décision → commande, validée dans Gazebo puis sur le robot réel.

Le modèle de détection des panneaux est produit par le repo jumeau [`trafficbot-model`](https://github.com/Loren844/trafficbot-model) (cours Edge AI). Ce repo l'utilise sans le modifier.

## Socle technique

Imposé par la doc du prof ([gerald-rocher/Turtlebot3](https://github.com/gerald-rocher/Turtlebot3)) :

- Ubuntu 22.04, ROS2 Humble, Cyclone DDS (`RMW_IMPLEMENTATION=rmw_cyclonedds_cpp`)
- Gazebo via `ros_gz`, `TURTLEBOT3_MODEL=burger`
- URDF `turtlebot3_burger_pi_cam` (ou `turtlebot3_burger_oak_d_pro`) fourni dans `projects/turtlebot3_descriptions`
- Packages Python `ament_python` ; exemple de machine à états réutilisable : `turtlebot3_fsm`

## Matériel

| Élément | Détail |
| --- | --- |
| Robot | TurtleBot3 Burger, calculateur Raspberry Pi 4, LiDAR LDS |
| Caméra | Raspberry Pi Camera montée sur le robot (URDF `turtlebot3_burger_pi_cam` en simulation) |
| Circuit | pas encore de tapis routier : voir « Décision ouverte » ci-dessous |

Toute la chaîne embarquée (suivi du chemin, détection des panneaux, FSM) doit tenir sur le Pi 4 du robot, sauf si le benchmark de `trafficbot-model` conclut qu'il faut déporter la détection sur un PC.

## Inclus (obligatoire)

1. **Monde Gazebo du circuit** : route avec lignes au sol, au moins une intersection, une voie fermée par un mur, modèles 3D des panneaux (stop, sens interdit, interdit à gauche, interdit à droite) et du feu.
2. **Suivi du chemin** par vision classique (OpenCV) avec un régulateur PID sur l'écart au centre, publié sur `/cmd_vel`. La méthode dépend du support physique choisi (voir « Décision ouverte »).
3. **Nœud de détection** qui charge le modèle publié par `trafficbot-model` et publie les panneaux détectés.
4. **Machine à états de décision** : `SUIVRE`, `ARRET_STOP` (3 s puis repart), `ATTENTE_FEU` (arrêt au rouge, repart au vert), `CHOIX_BRANCHE` (à l'intersection, exclure les branches interdites par `sens_interdit`, `interdit_gauche` ou `interdit_droite`), `DEMI_TOUR` (mur détecté devant le robot par le LiDAR).
5. **Feu tricolore connecté** : nœud qui publie l'état du feu sur un topic ROS2 (simulé dans Gazebo, puis vrai feu piloté par une carte sur le même réseau DDS).
6. **Portage sur le robot réel** : circuit physique selon l'option retenue, panneaux imprimés, mêmes nœuds et mêmes topics qu'en simulation.
7. **Launch files** : un pour la simulation, un pour le robot réel.

## Décision ouverte : comment le robot détecte le chemin

Il n'y a pas de tapis routier pour l'instant. Le choix est à faire avant le 02/12/2026, car il conditionne le monde Gazebo, le dataset et le circuit physique.

| Option | Principe | Coût et mise en place | Avantages | Limites |
| --- | --- | --- | --- | --- |
| **1. Ruban adhésif au sol** (recommandée) | Deux lignes de bord blanches (et une ligne centrale jaune pointillée si besoin) collées sur un sol sombre ; la Pi Cam inclinée vers le sol suit le centre entre les lignes | Quelques rouleaux de ruban d'électricien ou de masquage, circuit modifiable en 30 min | Simple, robuste, rapide sur Pi 4, facile à reproduire à l'identique dans Gazebo (texture de sol) | Sensible aux reflets et à l'éclairage ; sol sombre et mat nécessaire |
| 2. Circuit imprimé sur feuilles ou bâche | Tronçons de route imprimés (A3 ou bâche) assemblés en modules : ligne droite, virage, intersection, impasse | Impression de 20 à 40 feuilles A3, ou une bâche sur mesure | Rendu de vraie route, modules réarrangeables | Jointures visibles, feuilles qui bougent, plus cher pour une bâche |
| 3. Bordures physiques + LiDAR | Route délimitée par des murets (carton, tasseaux) ; le LiDAR du Burger suit le couloir, la caméra ne sert qu'aux panneaux | Carton ou tasseaux de 10 à 15 cm de haut | Indépendant de l'éclairage, réutilise les exemples LiDAR du prof (`turtlebot3_braitenberg`) | Ce n'est plus du suivi de lignes au sol, ce qui s'éloigne du sujet ; impasse et intersection plus délicates |
| 4. Ligne au sol + marqueurs ArUco | Option 1, avec un marqueur ArUco à chaque intersection ou impasse pour savoir où est le robot | Option 1 + marqueurs imprimés | Facilite le choix de branche et le checkpoint (optionnel) | Un élément de plus à détecter |

Recommandation : partir sur l'option 1 tout de suite (ou l'option 4 si le checkpoint entre dans le périmètre). Elle se fabrique sans attendre un tapis, se modélise facilement dans Gazebo, et une segmentation de route par réseau de neurones reste hors périmètre.

## Optionnel

- Passage piéton : ralentir (vitesse réduite de 50 % pendant la traversée).
- Checkpoint à rejoindre : choix de branche aux intersections, ou Nav2 + carte Cartographer.
- Arrêt d'urgence sur obstacle avec le LiDAR du Burger.
- Feu orange : état `orange` du feu connecté, traité comme un rouge.
- Visualisation Foxglove ou RViz des détections et de l'état de la FSM.

## Exclus

- Plusieurs robots simultanés sur le circuit.
- Conduite de bout en bout par réseau de neurones (le suivi de ligne reste classique).
- Entraînement et optimisation du modèle (dans `trafficbot-model`).
- Sécurité DDS (SROS2).

## Interface avec trafficbot-model

| Élément | Contrat | Produit par | Utilisé par |
| --- | --- | --- | --- |
| Classes | `stop`, `sens_interdit`, `interdit_gauche`, `interdit_droite`, `feu_rouge`, `feu_vert` ; `passage_pieton` en option | décidé ensemble ([BENCHMARK.md](https://github.com/Loren844/trafficbot-model/blob/main/docs/BENCHMARK.md)) | les deux repos |
| Modèle | `model.onnx` + `model_int8.tflite` + `labels.txt`, en release GitHub versionnée de `trafficbot-model` | trafficbot-model | nœud de détection |
| Entrée | image RGB 320×320, prétraitement défini dans un `preprocess.py` partagé | trafficbot-model | nœud de détection |
| Caméra | `/camera/image_raw` (`sensor_msgs/Image`) | Gazebo ou caméra réelle | détecteur, enregistrement du dataset |
| Détections | `/traffic/detections` (`vision_msgs/Detection2DArray`) | nœud de détection | FSM |
| Feu connecté | `/traffic_light/state` (`std_msgs/String` : `rouge`, `vert` ; `orange` en option) | carte du feu | FSM |
| Latence | p95 < 100 ms capture → détection sur le Pi 4 du robot | vérifié par le benchmark | dimensionne la vitesse |
| Données | rosbags enregistrés en simulation et sur le circuit réel | ce repo | dataset de trafficbot-model |

Le benchmark de `trafficbot-model` tranche une question de ce repo : exécuter la détection sur le Raspberry Pi du robot, ou déporter l'image vers un PC par DDS.

## Livrables

- Packages ROS2, monde Gazebo, launch files, README d'installation reproductible.
- Vidéo du scénario complet en simulation, puis sur le robot réel.
- Rendus d'évaluation de la matière (rapport, soutenance) : **à confirmer**, le support d'introduction du cours n'a pas pu être lu.

## Risques et questions ouvertes

- Écart simulation / réel (éclairage, couleurs des lignes) : seuils paramétrables, dataset mixte.
- Gauche et droite confondues par la vision (panneaux symétriques) : le modèle est entraîné sans retournement horizontal.
- Rouge et vert confondus par la vision : le topic du feu connecté fait foi.
- Pi 4 du robot trop lent : INT8, résolution réduite ou inférence déportée sur PC.
- [ ] Dates et forme des rendus de la matière ?
- [ ] Méthode de détection du chemin (voir « Décision ouverte ») ?
- [ ] Passage piéton et checkpoint : obligatoires ou non ?
- [ ] Quelle carte pilote le feu, et comment rejoint-elle le réseau ROS2 ?
