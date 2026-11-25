# SAIAHDANSAYMERIC

Article 1 - Agent GLPI arrêté : la machine ne remonte plus dans l'inventaire

Niveau : N1
Catégorie : Inventaire / Agent

Symptôme
Une machine n'apparait plus à jour dans GLPI. Le champ "Vu pour la dernière fois" affiche une date ancienne, l'inventaire ne se rafraichit plus.

Diagnostic (sur la machine concernée)

bash
systemctl status glpi-agent
journalctl -u glpi-agent -n 50
curl -I http://192.168.56.40

On vérifie que l'agent est arrêté et que le serveur GLPI répond bien (le curl doit renvoyer un code HTTP).

Résolution N1

bash
sudo systemctl start glpi-agent
sudo systemctl enable glpi-agent
sudo glpi-agent --force

Puis contrôler dans GLPI que la date "Vu pour la dernière fois" repasse à l'heure actuelle.

Critères d'escalade N2

L'agent refuse de démarrer malgré le start (erreur persistante dans journalctl)
Le curl vers le serveur GLPI échoue (le problème est côté serveur ou réseau, pas l'agent)
Après le --force, la machine ne remonte toujours pas (config agent à revoir : URL serveur, tag)
Article 2 - Site web Ubuntu (nginx) inaccessible

Niveau : N1
Catégorie : Services / Web

Symptôme
Le site http://192.168.56.30 ne répond plus : "Connection refused" ou page qui ne charge pas.

Diagnostic (sur le serveur Ubuntu)

bash
systemctl status nginx
ss -lntp | grep nginx
sudo nginx -t
journalctl -u nginx -n 20

On regarde si le service tourne, s'il écoute bien sur le port 80, et si la config est valide.

Résolution N1

bash
# Si le service est arrêté
sudo systemctl start nginx
sudo systemctl enable nginx

# Si nginx -t signale une erreur de config
# corriger le fichier pointé par l'erreur, puis
sudo systemctl reload nginx

Valider avec curl -I http://192.168.56.30 qui doit renvoyer HTTP/1.1 200 OK.

Critères d'escalade N2

nginx -t montre une erreur de config qu'on ne sait pas corriger au N1
Le service redémarre puis retombe aussitôt (crash récurrent)
Le port 80 est occupé par un autre processus (conflit)
Cause plus profonde : disque plein, permissions, certificat
