# KB - Service HTTP Ubuntu indisponible (mauvais port nginx)

**Niveau :** N1 (escalade N2 si besoin)
**Catégorie :** Services / Web
**Environnement :** laboratoire lab.local (Proxmox), serveur Ubuntu et serveur GLPI

## 1. Objectif

Diagnostiquer et corriger l'indisponibilité du service HTTP sur un serveur Ubuntu, en vérifiant l'état du service, les journaux nginx et les ports en écoute.

## 2. Symptômes

- Le service HTTP est injoignable sur le port standard 80 (échec de connexion ou délai dépassé côté client / `curl`).
- nginx tourne ou a redémarré, mais n'écoute pas sur le port 80 : il écoute sur un mauvais port (par exemple 6000), suite à une erreur de syntaxe dans la configuration.

## 3. Vérifications (diagnostic)

À faire dans l'ordre.

### État du service nginx

```bash
systemctl status nginx
```
Attendu : voir si le service est `active (running)`, `failed`, ou s'il a rencontré un souci au rechargement de sa config.

### Journaux nginx

```bash
journalctl -u nginx -n 50
sudo tail -n 50 /var/log/nginx/error.log
```
Attendu : repérer les messages d'avertissement ou d'erreur liés au chargement de la config, à la syntaxe ou à l'ouverture des ports.

### Ports utilisés par nginx

```bash
ss -lntp
```
(ou `sudo netstat -tlpn`, ou `sudo lsof -i -P -n | grep LISTEN` selon les outils installés)

Attendu : vérifier quel port est réellement écouté par nginx. Dans l'incident constaté, c'est le port 6000 qui apparaissait au lieu du port 80.

## 4. Méthode de résolution

Localiser et corriger l'erreur dans la config nginx (`/etc/nginx/nginx.conf`, ou les fichiers de `/etc/nginx/sites-enabled/` ou `/etc/nginx/conf.d/`).

### Corriger la directive d'écoute

Remplacer `listen 6000;` (ou corriger la syntaxe erronée) par :
```nginx
listen 80;
```
Attendu : la directive d'écoute sur le port 80 est présente et syntaxiquement valide.

### Tester la syntaxe

```bash
sudo nginx -t
```
Attendu : `syntax is ok` et `test is successful`.

### Recharger nginx

```bash
sudo systemctl reload nginx
```
(ou `sudo systemctl restart nginx` si nécessaire)

Attendu : la commande se termine sans erreur et `systemctl status nginx` affiche `active (running)`.

## 5. Critères d'escalade N2

Escalader au N2 (administration système / réseau), avec les sorties de commandes, si :
- `nginx -t` signale une erreur de config qu'on ne sait pas corriger au N1
- le service redémarre puis retombe aussitôt (crash récurrent)
- le port 80 est occupé par un autre processus (conflit)
- la cause est plus profonde (disque plein, permissions, certificat)

## 6. Test de validation

### Le port 80 est de nouveau en écoute

```bash
ss -lntp | grep :80
```
Attendu : le port 80 apparait en `LISTEN` associé à nginx, et le port 6000 n'est plus utilisé à tort.

### Réponse HTTP valide

```bash
curl -I http://127.0.0.1
```
(ou `curl -I http://192.168.56.30`)

Attendu : le serveur répond avec un en-tête HTTP valide (`200`, `301` ou `302`), le service est de nouveau accessible.

## 7. Clôture du ticket

- **Cause identifiée :** erreur de syntaxe dans la config nginx entrainant une écoute sur le port 6000 au lieu du port 80, rendant le service HTTP indisponible sur son port standard.
- **Action réalisée :** correction du fichier de config pour rétablir l'écoute sur le port 80, validation avec `nginx -t`, rechargement du service, vérification via `ss -lntp` et `curl`.
- **Statut :** résolu, le service HTTP est opérationnel sur le port 80.
