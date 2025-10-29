# Installation et Configuration de Howdy 3.0.0-5 sur Fedora 42

## 1. Télécharger les dépendances
Avant d'installer Howdy, il faut récupérer les paquets nécessaires :

```bash
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/07780296-python-pyv4l2/python3-pyv4l2-1.0.2-3.20240124gitf12f0b3.fc41.x86_64.rpm
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/07780294-python-keyboard/python3-keyboard-0.13.5-3.fc41.noarch.rpm
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/07780292-python-elevate/python3-elevate-0.1.3-3.20240124git78e82a8.fc41.noarch.rpm
```

## 2. Installer les dépendances
Assurez-vous que vous êtes dans le dossier où se trouvent les fichiers téléchargés :

```bash
cd ~/Téléchargements
sudo dnf install \
    python3-elevate-0.1.3-3.20240124git78e82a8.fc41.noarch.rpm \
    python3-keyboard-0.13.5-3.fc41.noarch.rpm \
    python3-pyv4l2-1.0.2-3.20240124gitf12f0b3.fc41.x86_64.rpm
```

Ensuite, installez OpenCV et V4L2 :

```bash
sudo dnf install -y opencv opencv-devel python3-opencv
sudo dnf install -y v4l-utils
```

## 3. Télécharger et installer Howdy
Téléchargez les fichiers d'installation de Howdy :

```bash
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/09288128-howdy/howdy-data-3.0.0-7.20250714gitd3ab993.fc42.noarch.rpm
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/09288128-howdy/howdy-3.0.0-7.20250714gitd3ab993.fc42.x86_64.rpm
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/09288128-howdy/howdy-gtk-3.0.0-7.20250714gitd3ab993.fc42.noarch.rpm
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/09288128-howdy/howdy-debuginfo-3.0.0-7.20250714gitd3ab993.fc42.x86_64.rpm
wget https://download.copr.fedorainfracloud.org/results/principis/howdy-beta/fedora-42-x86_64/09288128-howdy/howdy-debugsource-3.0.0-7.20250714gitd3ab993.fc42.x86_64.rpm

```

Installez les fichiers :

```bash
sudo dnf install ~/Téléchargements/howdy-*.fc41.*.rpm
```

## 4. Configurer Howdy
### Identifier le périphérique vidéo
Utilisez les commandes suivantes pour identifier votre caméra :

```bash
v4l2-ctl --list-devices
ls /dev/video*
```

Si plusieurs périphériques sont détectés, testez-les :

```bash
ffplay /dev/video0
ffplay /dev/video1
```

### Modifier le fichier de configuration de Howdy
Ouvrez le fichier de configuration :

```bash
sudo howdy config
```

Modifiez les paramètres suivants :

```
detection_notice = true
timeout_notice = true
no_confirmation = false
suppress_unknown = false
abort_if_ssh = true
abort_if_lid_closed = true
disabled = false
use_cnn = false
workaround = input
certainty = 4.0
timeout = 10
device_path = /dev/video0  # Remplacez par votre périphérique détecté
warn_no_device = true
max_height = 480
frame_width = 640
frame_height = 480
dark_threshold = 80
recording_plugin = opencv
device_format = v4l2
force_mjpeg = true
exposure = -1
device_fps = 15
rotate = 1
```

Sauvegardez et quittez.

## 5. Ajouter un modèle de visage
Ajoutez un modèle de reconnaissance faciale pour votre utilisateur :

```bash
sudo howdy add
```

Donnez un nom au modèle lorsque demandé.

## 6. Tester la reconnaissance faciale
Testez si la reconnaissance fonctionne :

```bash
sudo howdy test
```

## 7. Configurer l’authentification
### Modifier les fichiers PAM

Ajoutez Howdy à la configuration PAM pour sudo :

```bash
sudo nano /etc/pam.d/sudo
```

Ajoutez cette ligne au début du fichier :

```
auth       sufficient   pam_howdy.so no_confirmation
```

Faites de même pour GDM :

```bash
sudo nano /etc/pam.d/gdm-password
```

Ajoutez :

```
auth       sufficient   pam_howdy.so
```

## 8. Gérer les permissions
Donnez les autorisations nécessaires à votre caméra :

```bash
sudo chmod o+rw /dev/video*
```

```bash
sudo chmod -R o+rx /usr/share/howdy/dlib-data
```
```bash
sudo chmod +x /usr/bin/howdy
```
```bash
sudo usermod -aG video gdm
```
```bash
sudo chmod 666 /dev/video*
```
```bash
sudo chmod 755 /usr/lib64/security/pam_howdy.so
```

## 9. Créer un module SELinux personnalisé pour autoriser Howdy
Créer un fichier de politique SELinux, par exemple howdy.te :

```bash
sudo nano howdy.te
```

Ajoutez-y le contenu suivant :

```
module howdy 1.0;
require {
    type lib_t;
    type xdm_t;
    type v4l_device_t;
    type sysctl_vm_t;
    class chr_file map;
    class file { create getattr open read write };
    class dir add_name;
}

allow xdm_t lib_t:dir add_name;
allow xdm_t lib_t:file { create write };
allow xdm_t sysctl_vm_t:file { getattr open read };
allow xdm_t v4l_device_t:chr_file map;

```

Compiler le module :
 
```bash
checkmodule -M -m -o howdy.mod howdy.te
semodule_package -o howdy.pp -m howdy.mod
```

Installer le module SELinux :

```bash
sudo semodule -i howdy.pp
```

## 🎯 Conclusion
Votre système Fedora 42 est maintenant configuré pour utiliser la reconnaissance faciale avec Howdy. 🚀

En cas de problème, vérifiez les permissions et les logs avec :

```bash
journalctl -xe | grep howdy
```

Amusez-vous bien avec la reconnaissance faciale ! 😊

