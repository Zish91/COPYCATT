# KB - Agent GLPI arrêté : la remontée d'inventaire ne se fait plus

**Niveau :** N1
**Catégorie :** Inventaire / Agent GLPI
**Environnement :** serveur GLPI 192.168.56.40, machine Ubuntu du laboratoire

## 1. Objectif

Diagnostiquer et résoudre l'arrêt de la remontée d'inventaire de l'agent GLPI vers le serveur, en distinguant bien deux chaines techniques différentes :
- la chaine **agent → serveur GLPI** (l'inventaire)
- la chaine **client → serveur Web (HTTP)** (l'accès à l'interface)

## 2. Symptômes

- Absence ou retard anormal de mise à jour des informations d'inventaire d'une machine Ubuntu dans GLPI.
- Le service `glpi-agent` est arrêté (`inactive` / `dead`) ou ne communique plus avec le serveur GLPI.
- Le service Web (nginx ou apache2) qui héberge GLPI peut être fonctionnel ou non, selon le périmètre de la panne.

## 3. Vérifications (diagnostic)

Avant toute modification, on distingue la chaine de remontée de l'agent et la disponibilité HTTP du serveur Web.

### État des services

```bash
systemctl status glpi-agent nginx apache2
```
Attendu : `glpi-agent` doit être `active (running)`, et le service Web utilisé (nginx ou apache2) doit lui aussi être `active (running)`.

### Journaux de l'agent

```bash
journalctl -u glpi-agent -n 50
```
Attendu : repérer d'éventuelles erreurs d'arrêt, de timeout réseau ou d'échec d'envoi vers l'URL du serveur GLPI.

### Test de connexion HTTP vers le serveur GLPI

```bash
curl -I http://192.168.56.40
```
Attendu : une réponse HTTP valide (`200`, `301` ou `302`), qui prouve que le serveur Web répond.

### Ports en écoute

```bash
ss -lntp
```
Attendu : la présence des ports d'écoute attendus (80 / 443 côté serveur Web si pertinent) associés au bon processus.

## 4. Méthode de résolution

Dans le cas constaté (service `glpi-agent` à l'arrêt) :

### Démarrer le service

```bash
sudo systemctl start glpi-agent
```
Attendu : la commande s'exécute sans erreur, et `systemctl status glpi-agent` affiche `active (running)`.

### Activer le démarrage automatique (si nécessaire)

```bash
sudo systemctl enable glpi-agent
```
Attendu : confirmation du lien systemd pour le démarrage au boot.

## 5. Critères d'escalade N2

Escalader au niveau supérieur (N2 / administration système ou réseau), en joignant les sorties de commandes, si :
- `glpi-agent` refuse de démarrer malgré le `start`
- les journaux (`journalctl -u glpi-agent -n 50`) montrent des erreurs persistantes de configuration, d'authentification ou de réseau
- le `curl -I http://192.168.56.40` échoue (le problème est côté serveur Web ou réseau, pas l'agent)

## 6. Test de validation

### État du service

```bash
systemctl status glpi-agent
```
Attendu : `active (running)`, sans redémarrage en boucle.

### Derniers journaux

```bash
journalctl -u glpi-agent -n 20
```
Attendu : plus de nouvelles erreurs critiques, activité normale de l'agent.

### Côté interface GLPI

Ouvrir la fiche de l'équipement concerné et vérifier que la date de dernier inventaire / dernière mise à jour s'actualise bien.

## 7. Clôture du ticket

- **Cause identifiée :** service `glpi-agent` arrêté sur la machine Ubuntu du laboratoire.
- **Action réalisée :** redémarrage du service (`systemctl start glpi-agent`), vérification de l'état (`systemctl status`) et de la remontée dans GLPI.
- **Statut :** résolu, la remontée d'informations est de nouveau opérationnelle.
