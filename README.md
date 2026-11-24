# trafficBot

Projet du cours Systèmes intelligents autonomes (Polytech, 2026-2027).

Un TurtleBot3 Burger suit un circuit routier miniature et applique quatre règles : stop (arrêt de 3 s), sens interdit, cul-de-sac (demi-tour) et feu tricolore connecté. Le code est développé avec ROS2, testé dans Gazebo, puis exécuté sur le robot.

Le modèle de détection des panneaux vient du dépôt [trafficbot-model](https://github.com/Loren844/trafficbot-model) (cours Edge AI).

## Matériel

- TurtleBot3 Burger (Raspberry Pi 4, LiDAR LDS)
- Raspberry Pi Camera

## Environnement

- Ubuntu 22.04
- ROS2 Humble
- Cyclone DDS (`RMW_IMPLEMENTATION=rmw_cyclonedds_cpp`)
- Gazebo (`ros_gz`)
- Paquets TurtleBot3 de ROBOTIS (branche `humble`)

L'installation suit le dépôt du cours : [gerald-rocher/Turtlebot3](https://github.com/gerald-rocher/Turtlebot3).

```bash
export TURTLEBOT3_MODEL=burger
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

## Topics ROS2 prévus

| Topic | Type | Rôle |
| --- | --- | --- |
| `/camera/image_raw` | `sensor_msgs/Image` | image de la caméra |
| `/traffic/detections` | `vision_msgs/Detection2DArray` | panneaux et feux détectés |
| `/traffic_light/state` | `std_msgs/String` | état du feu connecté (`rouge`, `orange`, `vert`) |
| `/cmd_vel` | `geometry_msgs/Twist` | commande de vitesse |

## Documentation

- [docs/SCOPE.md](docs/SCOPE.md) : périmètre, interfaces, jalons, répartition

## Conventions de commit

Les messages suivent [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/) :

```
<type>(<portée>): <description>
```

- `type` : voir le tableau ci-dessous.
- `portée` : optionnelle, la partie du dépôt concernée (par exemple `gazebo`, `fsm`, `detection`, `feu`, `launch`).
- `description` : à l'impératif, en minuscules, sans point final, 72 caractères maximum.
- Un changement incompatible est signalé par `!` après le type (`feat!: ...`) ou par une ligne `BREAKING CHANGE:` dans le corps.

| Type | Usage |
| --- | --- |
| `feat` | nouvelle fonctionnalité |
| `fix` | correction de bug |
| `docs` | documentation uniquement |
| `refactor` | modification du code sans changement de comportement |
| `test` | ajout ou modification de tests |
| `perf` | amélioration des performances |
| `build` | dépendances, configuration de build |
| `chore` | maintenance (fichiers ignorés, scripts annexes) |

Exemples :

```
feat(fsm): ajouter l'arrêt de 3 s au panneau stop
fix(gazebo): corriger la position du feu dans le monde du circuit
docs: ajouter le schéma des topics au README
```
