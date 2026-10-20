<p align="center">
  <img src="medias/tangram-robot-overview.svg" alt="Architecture : éditeur Tangram en Python, communication TCP/IP et robot UR5e avec préhenseur" width="100%">
</p>

<h1 align="center">UR Tangram Control</h1>

<p align="center">
  <strong>EN:</strong> Arrange tangram pieces in a 2D editor and transmit their target poses to a Universal Robots UR5e pick-and-place program.<br>
  <strong>FR :</strong> Placer des pièces de tangram dans un éditeur 2D et transmettre leurs poses cibles à un programme de prise et de dépôt pour Universal Robots UR5e.
</p>

<p align="center">
  <a href="https://github.com/mlmdc">← Back to profile / Retour au profil</a>
  &nbsp;·&nbsp;
  <a href="#contents">Contents / Sommaire</a>
</p>

<a id="contents"></a>

## Contents / Sommaire

- [Overview / Présentation](#overview)
- [Problem and approach / Problématique et approche](#problem-and-approach)
- [Methods and features / Méthodes et fonctionnalités](#methods-and-features)
- [Technical decisions / Choix techniques](#technical-decisions)
- [Results and limitations / Résultats et limites](#results-and-limitations)
- [Getting started / Démarrage](#getting-started)
- [Project files / Fichiers du projet](#project-files)
- [Contributors / Contributeurs](#contributors)
- [Credits / Crédits](#credits)

<a id="overview"></a>

## Overview / Présentation

**EN.** This project was developed for the Robot Control course at the University of Ljubljana. It connects a desktop tangram editor to a Universal Robots UR5e: an operator places the seven standard pieces on a 2D board, and the Python application maps their screen positions and rotations to robot-frame target poses. Those poses are sent over TCP to a PolyScope program that runs a pick-and-place sequence. The project demonstrates this interaction and data-transfer workflow; it does not provide a general-purpose motion planner.

**FR.** Ce projet a été développé dans le cadre du cours de commande des robots de l’Université de Ljubljana. Il relie un éditeur de tangram sur ordinateur à un Universal Robots UR5e : l’opérateur place les sept pièces classiques sur un plateau 2D, puis l’application Python convertit leurs positions et rotations à l’écran en poses cibles dans le repère du robot. Ces poses sont envoyées par TCP à un programme PolyScope qui exécute une séquence de prise et de dépôt. Le projet illustre ce flux d’interaction et de données ; il ne constitue pas un planificateur de mouvement généraliste.

![Pygame editor showing the seven tangram pieces; this screenshot was captured without a robot connection.](medias/tangram-editor-example.png)

*EN: The editor view illustrates the interface only; no robot is connected in this screenshot.*<br>
*FR : Cette capture illustre uniquement l’interface ; aucun robot n’y est connecté.*

**EN.** The following layouts were also arranged in the editor. They illustrate different positions and rotations, not validated robot targets.

**FR.** Les dispositions suivantes ont également été composées dans l’éditeur. Elles illustrent différentes positions et rotations, sans constituer des cibles robot validées.

| Balanced layout / Disposition équilibrée | Rotated pieces / Pièces tournées | Grouped layout / Disposition regroupée |
| --- | --- | --- |
| ![Tangram pieces in a balanced layout](medias/tangram-layout-balanced.png) | ![Tangram pieces with varied rotations](medias/tangram-layout-rotated.png) | ![Tangram pieces in a grouped layout](medias/tangram-layout-grouped.png) |

<a id="problem-and-approach"></a>

## Problem and approach / Problématique et approche

**EN.** The operator plans a placement visually, but the robot program needs positions and orientations in its own coordinate frame. The application bridges these representations: it stores each piece’s type, 2D center and angle; converts the center using the configured work-area mapping; and sends the resulting six-value pose (position and rotation vector) to the robot. The mapping and offsets are installation parameters, not values inferred from a camera or from the physical table.

The workflow has two parts. Python provides the editor and opens a TCP server on the configured PC address and port `8888`. The PolyScope project connects as a client, receives seven pose slots, and uses its configured robot and gripper commands to carry out the sequence. The editor prepares targets; it does not plan collision-free paths or check that targets are reachable.

**FR.** L’opérateur définit visuellement une disposition, tandis que le programme robot nécessite des positions et des orientations dans son propre repère. L’application fait le lien entre ces représentations : elle conserve le type, le centre 2D et l’angle de chaque pièce, convertit le centre avec la correspondance configurée pour la zone de travail, puis transmet au robot une pose à six valeurs (position et vecteur de rotation). La correspondance et les décalages sont des paramètres d’installation ; ils ne sont pas calculés à partir d’une caméra ou de la table physique.

Le flux comporte deux parties. Python fournit l’éditeur et ouvre un serveur TCP à l’adresse configurée du PC sur le port `8888`. Le projet PolyScope se connecte comme client, reçoit sept emplacements de pose et utilise les commandes configurées pour le robot et le préhenseur afin d’exécuter la séquence. L’éditeur prépare les cibles ; il ne planifie pas de trajectoires sans collision et ne vérifie pas que les cibles sont atteignables.

<a id="methods-and-features"></a>

## Methods and features / Méthodes et fonctionnalités

**EN.** The editor defines an inventory of seven pieces: two small triangles, one medium triangle, two large triangles, one square and one parallelogram. Pieces can be added from the menu, dragged, rotated in 15-degree increments, snapped by bringing vertices within the configured pixel tolerance, removed using the trash area, or managed with undo, redo and reset. These interactions support manual layout iteration; snapping is geometric proximity, not a proof that pieces form a valid tangram solution.

On export, `src/main.py` creates a fixed-order list of seven poses: medium triangle, two small triangles, two large triangles, square and parallelogram. It maps screen coordinates to the configured robot area, applies type-specific angle adjustments, and sends the poses over the socket. The PolyScope program associates the slots with its piece-handling sequence. Export always contains seven slots: a piece that has not been placed retains the default pose from `src/cfg.py`. The robot program still runs seven cycles, so an incomplete layout can send the robot to those default targets.

**FR.** L’éditeur définit un inventaire de sept pièces : deux petits triangles, un triangle moyen, deux grands triangles, un carré et un parallélogramme. Les pièces peuvent être ajoutées depuis le menu, déplacées, tournées par incréments de 15 degrés, alignées lorsque leurs sommets sont suffisamment proches selon une tolérance en pixels, supprimées avec la zone de corbeille ou gérées avec les fonctions annuler, rétablir et réinitialiser. Ces interactions permettent d’ajuster manuellement une disposition ; l’alignement repose sur une proximité géométrique et ne prouve pas que les pièces forment une solution valide du tangram.

À l’export, `src/main.py` crée une liste ordonnée fixe de sept poses : triangle moyen, deux petits triangles, deux grands triangles, carré et parallélogramme. Il convertit les coordonnées écran vers la zone robot configurée, applique des corrections d’angle propres aux types de pièces et envoie les poses par socket. Le programme PolyScope associe les emplacements à sa séquence de manipulation. L’export contient toujours sept emplacements : une pièce non placée conserve la pose par défaut définie dans `src/cfg.py`. Le programme robot exécute tout de même sept cycles ; une disposition incomplète peut donc envoyer le robot vers ces cibles par défaut.

<a id="technical-decisions"></a>

## Technical decisions / Choix techniques

**EN.** Pygame is used for the 2D interface and NumPy for numeric operations; Python’s standard `socket`, `threading` and `queue` modules handle communication. The interface and robot controller have separate responsibilities: Python maps and sends targets, while PolyScope owns motion execution and gripper operation. The pose mapping, work-area bounds, fixed height and orientation offsets are constants in `src/cfg.py` and `src/main.py`; changing them changes the physical targets and requires calibration for the actual cell.

The coordinate conversion scales screen pixels using the configured work-area span, adds the lower work-area coordinates, and converts millimetres to metres for the robot pose. Height and two rotation components are fixed in configuration; the screen angle is adjusted for the piece type and sent in radians. These values describe the intended mapping, but the repository does not provide a calibration procedure or accuracy measurements.

The TCP exchange is a simple application protocol rather than a framed, fault-tolerant transport. The Python application sends an `ACK` string followed by the seven pose values; the robot program sends acknowledgements in response, but the Python export path does not wait for or validate them. The server accepts a connection with a 10-second timeout during startup and does not retry the accept if it expires. Therefore, the PC and robot addresses, port, start order and timing must be configured consistently before operation.

**FR.** Pygame fournit l’interface 2D et NumPy les opérations numériques ; les modules standard Python `socket`, `threading` et `queue` assurent la communication. Les responsabilités sont séparées : Python convertit et envoie les cibles, tandis que PolyScope contrôle les mouvements et le préhenseur. La correspondance des poses, les limites de la zone de travail, la hauteur fixe et les décalages d’orientation sont des constantes dans `src/cfg.py` et `src/main.py` ; leur modification change les cibles physiques et nécessite un étalonnage adapté à la cellule.

La conversion met à l’échelle les pixels selon l’étendue configurée de la zone de travail, ajoute les coordonnées basses de cette zone, puis convertit les millimètres en mètres pour la pose robot. La hauteur et deux composantes de rotation sont fixes dans la configuration ; l’angle écran est corrigé selon le type de pièce et transmis en radians. Ces valeurs décrivent la conversion prévue, mais le dépôt ne fournit ni procédure d’étalonnage ni mesure de précision.

L’échange TCP est un protocole applicatif simple, sans mécanisme de cadrage des messages ni garantie de reprise après erreur. Python envoie une chaîne `ACK`, puis les sept poses ; le programme robot renvoie des accusés de réception, mais le chemin d’export Python ne les attend pas et ne les valide pas. Le serveur accepte une connexion avec un délai de 10 secondes au démarrage et ne relance pas cette attente si le délai expire. Les adresses du PC et du robot, le port, l’ordre de lancement et le timing doivent donc être configurés de façon cohérente avant toute utilisation.

<a id="results-and-limitations"></a>

## Results and limitations / Résultats et limites

**EN.** The repository contains the Pygame editor, a PolyScope project and visual examples of layouts. Together, they implement the path from a manually arranged 2D layout to a seven-slot TCP transfer and a robot-side pick-and-place sequence. The screenshots and layout images illustrate possible editor arrangements; they are not evidence that those arrangements were executed successfully by the robot.

No quantitative physical-accuracy measurements, systematic comparison of methods, or automated end-to-end test results are provided. The project does not verify calibration, reachability, collisions, safe separation, gripper compatibility or successful receipt of every pose. In particular, the default pose used for unplaced pieces and the fixed work-area and orientation values must be checked against the installation. Follow the cell’s safety procedures and validate targets and paths before running the robot.

**FR.** Le dépôt contient l’éditeur Pygame, un projet PolyScope et des exemples visuels de dispositions. Ensemble, ils mettent en œuvre le passage d’une disposition 2D définie manuellement à un transfert TCP de sept emplacements, puis à une séquence de prise et de dépôt côté robot. Les captures et les images illustrent des dispositions possibles dans l’éditeur ; elles ne prouvent pas que le robot les a exécutées avec succès.

Aucune mesure quantitative de précision physique, comparaison systématique de méthodes ni résultat de test automatisé de bout en bout n’est fourni. Le projet ne vérifie ni l’étalonnage, ni l’accessibilité des cibles, ni les collisions, ni les distances de sécurité, ni la compatibilité du préhenseur, ni la réception de chaque pose. En particulier, la pose par défaut des pièces non placées ainsi que les valeurs fixes de zone de travail et d’orientation doivent être contrôlées pour l’installation. Respectez les procédures de sécurité de la cellule et validez les cibles et trajectoires avant d’exécuter le robot.

<a id="getting-started"></a>

## Getting started / Démarrage

### Requirements / Prérequis

**EN.**

- Python 3; the repository does not specify a minimum version.
- `pygame` and `numpy` (no dependency file or pinned versions are provided).
- For physical operation: a compatible UR5e running the project in `RobotControl/tangram.urp`, a configured gripper, and a calibrated robot work area.
- A network connection that allows the PC and robot to communicate over TCP port `8888`.

**FR.**

- Python 3 ; le dépôt ne précise pas de version minimale.
- `pygame` et `numpy` (aucun fichier de dépendances ni aucune version fixée n’est fourni).
- Pour une utilisation physique : un UR5e compatible exécutant le projet `RobotControl/tangram.urp`, un préhenseur configuré et une zone de travail étalonnée.
- Une connexion réseau permettant au PC et au robot de communiquer en TCP sur le port `8888`.

From the repository root, create an environment and install the Python dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install pygame numpy
```

Depuis la racine du dépôt, créez un environnement et installez les dépendances Python :

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install pygame numpy
```

### Network and robot setup / Configuration réseau et robot

**EN.** Before starting, set `ip_address` in `src/cfg.py` to an address assigned to the PC and permitted to accept inbound TCP on port `8888`. Set the host address used by `socket_open(...)` in the PolyScope project to that same PC address, and confirm the port matches `trgt_port`. The checked-in values are not aligned: `src/cfg.py` uses `192.168.65.168`, while the readable PolyScope export `RobotControl/tangram.txt` uses `192.168.65.161`. Verify and update the project loaded in PolyScope as well as the Python configuration.

The Python server waits up to 10 seconds for the robot client during startup. Start the PolyScope program while the Python application is initializing. If the connection attempt times out, restart the application after correcting the setup; the server does not automatically retry. Confirm that the coordinate mapping, default pose, offsets, gripper and robot workspace are appropriate and safe for the physical cell. Do not run an unvalidated layout.

**FR.** Avant le lancement, définissez `ip_address` dans `src/cfg.py` avec une adresse attribuée au PC et autorisée à recevoir des connexions TCP entrantes sur le port `8888`. Dans le projet PolyScope, configurez l’adresse hôte utilisée par `socket_open(...)` avec cette même adresse du PC et vérifiez que le port correspond à `trgt_port`. Les valeurs présentes dans le dépôt ne sont pas cohérentes : `src/cfg.py` utilise `192.168.65.168`, tandis que l’export PolyScope lisible `RobotControl/tangram.txt` utilise `192.168.65.161`. Vérifiez et mettez à jour le projet chargé dans PolyScope ainsi que la configuration Python.

Au démarrage, le serveur Python attend jusqu’à 10 secondes la connexion du robot. Lancez le programme PolyScope pendant l’initialisation de l’application Python. Si la tentative de connexion expire, relancez l’application après avoir corrigé la configuration ; le serveur ne réessaie pas automatiquement. Vérifiez que la conversion des coordonnées, la pose par défaut, les décalages, le préhenseur et la zone de travail sont adaptés et sûrs pour la cellule physique. N’exécutez pas une disposition non validée.

### Run the editor / Lancer l’éditeur

From the repository root:

```powershell
python src\main.py
```

Depuis la racine du dépôt :

```powershell
python src\main.py
```

**EN.** Add pieces from the menu, drag them into position and rotate a selected piece with the left or right arrow key. Use **Export** only after checking that all seven target slots are intentional; **Undo**, **Redo**, **Reset** and the trash area edit the layout. The application requires a graphical display and loads its sound asset from `src/medias/`.

**FR.** Ajoutez les pièces depuis le menu, déplacez-les et tournez une pièce sélectionnée avec les flèches gauche et droite. Utilisez **Export** uniquement après avoir vérifié que les sept cibles sont intentionnelles ; **Undo**, **Redo**, **Reset** et la corbeille permettent de modifier la disposition. L’application nécessite un affichage graphique et charge son fichier audio depuis `src/medias/`.

<a id="project-files"></a>

## Project files / Fichiers du projet

| Path | Purpose |
| --- | --- |
| `src/main.py` | Pygame event loop, coordinate mapping and pose export |
| `src/shapes.py` | Tangram piece geometry and inventory |
| `src/cfg.py` | Network, work-area, pose and interface constants |
| `src/init.py` and `src/ur_comm.py` | Application initialization and TCP communication |
| `src/medias/` | Interface assets, including the snap sound and trash icon |
| `RobotControl/tangram.urp` | PolyScope project to open on the robot controller |
| `RobotControl/tangram.script` and `RobotControl/tangram.txt` | Readable robot-program exports |
| `medias/` | README architecture diagram, editor screenshot and illustrative layouts |
| `ur_tcpip-main/` | Included TCP/IP example code; not required to launch the tangram editor |
| `tangram_layout.json` | Example data file; the current application does not load it |

| Chemin | Rôle |
| --- | --- |
| `src/main.py` | Boucle d’événements Pygame, conversion des coordonnées et export des poses |
| `src/shapes.py` | Géométrie des pièces et inventaire du tangram |
| `src/cfg.py` | Constantes réseau, zone de travail, poses et interface |
| `src/init.py` et `src/ur_comm.py` | Initialisation de l’application et communication TCP |
| `src/medias/` | Ressources de l’interface, dont le son d’alignement et l’icône de corbeille |
| `RobotControl/tangram.urp` | Projet PolyScope à ouvrir sur le contrôleur robot |
| `RobotControl/tangram.script` et `RobotControl/tangram.txt` | Exports lisibles du programme robot |
| `medias/` | Schéma d’architecture du README, capture de l’éditeur et dispositions illustratives |
| `ur_tcpip-main/` | Exemples de communication TCP/IP inclus ; non requis pour lancer l’éditeur |
| `tangram_layout.json` | Exemple de données ; l’application actuelle ne le charge pas |

<a id="contributors"></a>

## Contributors / Contributeurs

- Mael Madec
- [Riccardo Simion](https://github.com/simionRiccard0/)

<a id="credits"></a>

## Credits / Crédits

**EN.** The TCP communication examples and initial code base were provided by Sebastjan Šlajpah ([sebastjan.slajpah@fe.uni-lj.si](mailto:sebastjan.slajpah@fe.uni-lj.si)): [sslajpah/ur_tcpip](https://github.com/sslajpah/ur_tcpip).

**FR.** Les exemples de communication TCP et la base initiale du code ont été fournis par Sebastjan Šlajpah ([sebastjan.slajpah@fe.uni-lj.si](mailto:sebastjan.slajpah@fe.uni-lj.si)) : [sslajpah/ur_tcpip](https://github.com/sslajpah/ur_tcpip).
