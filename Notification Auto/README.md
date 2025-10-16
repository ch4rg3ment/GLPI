# GLPI 11 – Actions automatiques qui n’envoient pas les mails (Debian)



Fix via systemd timer (remplace le crontab).

Laisser les tâches GLPI en mode CLI (interface GLPI → Configuration → Actions automatiques).



Créer un service et un timer systemd qui appellent front/cron.php.



Création du fichier service :


nano /etc/systemd/system/glpi-cron.service 


Dans l'editeur de texte :


[Unit]
Description=GLPI actions automatique

[Service]
User=www-data
Group=www-data
ExecStart=/usr/bin/php /var/www/html/front/cron.php



Création du fichier timer :


nano /etc/systemd/system/glpi-cron.timer


Dans l'editeur de texte :



[Unit]
Description=Run GLPI automatic actions every minute

[Timer]
OnCalendar=*-*-* *:*:00
Persistent=true

[Install]
WantedBy=timers.target




Activation du service :


sudo systemctl daemon-reload
sudo systemctl enable --now glpi-cron.timer
sudo systemctl status glpi-cron.timer



Tests & logs



# Lancer une exécution immédiate
sudo systemctl start glpi-cron.service

# Logs systemd
journalctl -u glpi-cron.service -n 100 --no-pager

# Logs GLPI
tail -n 100 /var/www/html/files/_log/cron.log

Fréquence (exemples)

Toutes les 5 minutes : OnCalendar=*:0/5

Toutes les 15 minutes : OnCalendar=*:0/15
Après modification :

sudo systemctl daemon-reload
sudo systemctl restart glpi-cron.timer

Vérifier que les mails partent

GLPI → Administration > Notifications > File d’envoi (se vide).

files/_log/cron.log : entrée “queuednotification … done”.

Points d’attention

Adapter les chemins si GLPI n’est pas dans /var/www/html.

PHP : vérifier le binaire (which php).

L’utilisateur systemd doit être celui du serveur web (www-data sur Debian).

Si vous aviez un crontab existant, le désactiver pour éviter les doublons.
