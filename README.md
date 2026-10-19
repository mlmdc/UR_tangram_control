<p align="center">
  <img src="docs/assets/tangram-robot-overview.svg" alt="Architecture du projet : interface Tangram en Python, liaison TCP/IP et robot UR5e avec préhenseur" width="100%">
</p>

<h1 align="center">UR Tangram Control</h1>

<p align="center">
  <strong>EN:</strong> Arrange a tangram on screen, then send its seven target poses to a UR5e for pick-and-place.<br>
  <strong>FR :</strong> Composer un tangram à l’écran, puis transmettre ses sept poses au robot UR5e pour une opération de prise et de dépôt.
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
- [Technical decisions / Choix techniques](#technical-decisions)
- [Results and learnings / Résultats et apprentissages](#results-and-learnings)
- [Getting started](#getting-started)
- [Project files / Fichiers du projet](#project-files)
- [Credits / Crédits](#credits)

<a id="overview"></a>

## Overview / Présentation

**EN.** This project connects a desktop tangram editor to a Universal Robots UR5e. It was developed for the Robot Control course at the University of Ljubljana and demonstrates a path from a user-defined 2D layout to a physical pick-and-place sequence. The Python application lets an operator arrange the pieces; the robot program receives their poses and performs the corresponding moves.

**FR.** Ce projet relie un éditeur de tangram sur ordinateur à un robot Universal Robots UR5e. Développé dans le cadre du cours de commande des robots de l’Université de Ljubljana, il illustre le passage d’une disposition 2D définie par l’utilisateur à une séquence physique de prise et de dépôt. L’application Python permet de placer les pièces ; le programme robot reçoit leurs poses et exécute les mouvements correspondants.

### Python application

**EN.** The Pygame interface displays the seven tangram pieces. The operator can add, drag, snap, rotate, undo, redo, reset or remove pieces, then export the layout as robot poses.

**FR.** L’interface Pygame affiche les sept pièces du tangram. L’opérateur peut les ajouter, déplacer, aligner, tourner, annuler ou rétablir une action, réinitialiser la disposition ou supprimer une pièce, puis exporter la disposition sous forme de poses robot.

### Universal Robots program

**EN.** The PolyScope project in `RobotControl/tangram.urp` connects to the Python host, receives the seven six-value poses, and runs the pick-and-place sequence using the configured UR5e and gripper.

**FR.** Le projet PolyScope `RobotControl/tangram.urp` se connecte à l’ordinateur, reçoit les sept poses à six valeurs et exécute la séquence de prise et de dépôt avec l’UR5e et le préhenseur configuré.

### How the two parts work together / Lien entre les deux parties

**EN.** Python runs a TCP server on port `8888`; the robot program is the client. The exported layout is converted from screen coordinates to robot-frame positions and orientations, then sent over the socket. Python handles the UI and data transfer; PolyScope controls robot motion and the gripper.

**FR.** Python ouvre un serveur TCP sur le port `8888` ; le programme du robot en est le client. La disposition est convertie des coordonnées écran en positions et orientations dans le repère du robot, puis transmise par socket. Python gère l’interface et le transfert des données ; PolyScope pilote les mouvements et le préhenseur.

<a id="problem-and-approach"></a>

## Problem and approach / Problématique et approche

**EN.** Manually translating a visual 2D arrangement into robot targets is error-prone. The project provides a direct workflow from arranging the pieces to exporting their poses, while keeping the graphical interaction on the PC and the motion sequence on the robot controller. The setup depends on a reachable, calibrated work area and matching network addresses.

**FR.** Convertir manuellement une disposition 2D en cibles robot est source d’erreurs. Le projet propose un flux direct, de la composition des pièces à l’export de leurs poses, en séparant l’interaction graphique sur PC de la séquence de mouvement exécutée par le contrôleur. Le dispositif nécessite une zone de travail calibrée et atteignable, ainsi que des adresses réseau concordantes.

<a id="technical-decisions"></a>

## Technical decisions / Choix techniques

**EN.** Pygame provides a lightweight 2D editor; Python’s standard `socket`, `threading` and `queue` modules handle TCP communication without an additional networking dependency. Separate send and receive loops keep socket operations outside the main UI loop. The robot-side sequence is authored for PolyScope and uses its motion and gripper commands. This simple client/server design suits a local lab setup, but currently relies on fixed network and calibration values rather than a deployment configuration.

**FR.** Pygame fournit un éditeur 2D léger ; les modules standard Python `socket`, `threading` et `queue` assurent la communication TCP sans dépendance réseau supplémentaire. Des boucles d’envoi et de réception séparées isolent les opérations socket de la boucle d’interface. La séquence côté robot est écrite pour PolyScope et utilise ses commandes de mouvement et de préhenseur. Cette architecture client/serveur convient à un banc local, mais repose encore sur des paramètres réseau et de calibration fixes plutôt que sur une configuration de déploiement.

<a id="results-and-learnings"></a>

## Results and learnings / Résultats et apprentissages

**EN.** The repository contains both sides of the workflow: a 2D layout editor that exports seven poses and a UR5e program that consumes them for pick-and-place. It demonstrates the coordinate conversion and TCP hand-off between a desktop interface and a robot controller. The project does not include automated tests or recorded measurements of physical accuracy. Current limitations include hard-coded IP addresses and work-area calibration, a 10-second Python connection-accept timeout, no automatic recovery after a lost connection, and no ACK-gated send loop on the Python side. Export always sends seven pose slots; unplaced pieces retain the configured default pose. A next iteration could externalize configuration, make message acknowledgement and reconnect behavior explicit, and test pose conversion independently of the robot.

**FR.** Le dépôt contient les deux côtés du flux : un éditeur 2D qui exporte sept poses et un programme UR5e qui les utilise pour la prise et le dépôt. Le projet met en œuvre la conversion de coordonnées et le transfert TCP entre une interface de bureau et un contrôleur robot. Il ne contient ni tests automatisés ni mesures enregistrées de précision physique. Ses limites actuelles comprennent des adresses IP et une calibration de zone codées en dur, un délai de 10 secondes pour l’acceptation de la connexion Python, l’absence de reconnexion automatique après une perte de liaison et l’absence d’attente des ACK dans la boucle d’envoi Python. L’export transmet toujours sept emplacements de pose ; les pièces non placées conservent leur pose par défaut. Une évolution utile consisterait à externaliser la configuration, à expliciter les acquittements et la reconnexion, puis à tester la conversion des poses indépendamment du robot.

## Getting started

### Requirements

- Python 3
- A Universal Robots UR5e running the matching PolyScope program
- A configured gripper and a calibrated, safe robot work area for physical operation
- A network connection between the PC and robot that permits TCP traffic on port `8888`

The Python dependencies used by the application are Pygame and NumPy. From the repository root:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install pygame numpy
```

### Network configuration

1. Assign a reachable IP address to the PC and allow inbound TCP traffic on port `8888`.
2. In `src\cfg.py`, set `ip_address` to the PC address and keep `trgt_port` aligned with the robot program.
3. In the PolyScope program, set the host address used by `socket_open(...)` to the PC address. The repository includes the program project at `RobotControl\tangram.urp`.
4. Make sure the configured robot workspace and pose offsets match the physical cell. Do not run the program until the robot path and gripper setup have been checked for the installation.

The current Python server waits up to 10 seconds for the robot connection during startup. Start the robot program while the Python application is opening its connection.

On Linux, inspect the network interfaces and, if needed, assign an address on the lab network:

```bash
hostname -I
ip addr show
sudo ip addr add 192.168.65.XXX/24 dev __device_name__
```

Replace the example address and interface with values appropriate for the network.

### Run the application

From the repository root, start the Python interface:

```powershell
python src\main.py
```

Start the matching PolyScope program during the Python connection window. In the interface, add pieces from the shape buttons, drag them into position, rotate a selected piece with the left or right arrow key in 15-degree increments, and use **Export** to send the layout. **Undo**, **Redo**, **Reset** and the trash area edit the layout. Close the application and stop the robot program before restarting either side.

<a id="project-files"></a>

## Project files / Fichiers du projet

| Path | Purpose |
| --- | --- |
| `src\` | Python application, Pygame interface, coordinate conversion and TCP communication |
| `src\medias\` | Interface assets |
| `RobotControl\tangram.urp` | Main Universal Robots PolyScope project |
| `RobotControl\` | Robot programs, generated script exports and installation files, including `comm.installation` (identified in the original setup notes as the likely installation) and `tangram_inst.installation` |
| `tangram_layout.json` | Tangram layout data |

<a id="credits"></a>

## Credits / Crédits

**EN.** The TCP communication examples and initial code base were provided by Sebastjan Šlajpah ([sebastjan.slajpah@fe.uni-lj.si](mailto:sebastjan.slajpah@fe.uni-lj.si)): [sslajpah/ur_tcpip](https://github.com/sslajpah/ur_tcpip). Contributors: Mael Madec and Riccardo Simion.

**FR.** Les exemples de communication TCP et la base initiale du code ont été fournis par Sebastjan Šlajpah ([sebastjan.slajpah@fe.uni-lj.si](mailto:sebastjan.slajpah@fe.uni-lj.si)) : [sslajpah/ur_tcpip](https://github.com/sslajpah/ur_tcpip). Contributeurs : Mael Madec et Riccardo Simion.
