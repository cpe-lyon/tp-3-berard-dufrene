# Compte rendu TP3
## Administration systèmes
BERARD Lucas
DUFRENE Mélic


### Exercice 1

**Question 1:** Quels sont les 5 derniers paquets installés sur votre machine ?
```
ii xkb-data ...
ii xxd ...
ii xz-utils ...
ii zerofree ...
ii zlibb1g:amd64
```

**Question 2:** Utiliser dpkg et apt pour compter le nombre de paquets installés (ne pas hésiter à consulter le manuel !).
Comment explique-t-on la (petite) différence de comptage ?

```
apt list --installed | wc -l

553
```
Renvoie le nombre de paquets installés
```
dpkg -l | wc  -l

549
```
La petite différence est dû au fait que dpkg contient plus de lignes d'explication au début du fichier


**Question 3:** Combien de paquets sont disponibles en téléchargement ?
```
nombreDispo=$(apt list | wc -l)
nombreInstalle=$(apt list --installed | wc -l)
echo $(($nombreDispo - $nombreInstalle))
```


**Question 4:** Créer un alias “maj” qui met à jour le système.
On ouvre le fichier .bashrc avec vim
```
cd ~
vim .bashrc
```
Puis on écrit:
```
alias maj='sudo apt upgrade'
```

**Question 5:** A quoi sert le paquet fortunes ? Installez-le.
La commande fortune permet d'avoir accès à des citations.
```
apt install fortune
```

**Question 6:** Quels paquets proposent de jouer au sudoku ?
Le paquet 'sudoku' permet de jouer au sudoku.
```
sudo apt install sudoku
```

**Question 7:** Lister les derniers paquets installés explicitement avec la commande apt install.
```
apt list --installed | tail -2
```
Donne comme réponse zerofree (...) et zlibb1g (...)

### Exercice 2
Pour trouver qui a installé un package, on tape:
```
dpkg -S /bin/ls
```
qui renvoie
```
coreutils:  /bin/ls
```
De manière plus générale, si l'on cherche qui a installé un paquet 'commande', on fait
```
dpkg -S $(which -a commande | tail -1)
```
#### Création du script origine-commande

```
vim origine-commande
chmod u+x origine-commande
```
Dans le script, on écrit:
```
#!/bin/bash

dpkg -S $(which -a $1 | tail -1)
```
### Exercice 3: Commande installee

```
vim testpack.sh
chmod u+x testpack.sh
```
Dans le script:
```
#!/bin/bash

var=$( apt list --installed | cut -d/ -f1 | grep "^$1$" | wc -l)
if [ $var -eq 0] ; then
    echo 'NON INSTALLE'
else
    echo 'INSTALLE'
fi
```

### Exercice 4
Lister les programmes installés avec coreutils:
```
dpkg -L coreutils
man [

echo $[ EXPRESSION ]
```
La commande '[]'permet d'évaluer une expression.

### Exercice 5
Installation de emacs via l'interface graphique aptitude.
```
sudo aptitude
/emacs
+
g
gg
```
### Exercice 6
**Question 1:** Installer la version Oracle de Java (avec l’ajout des PPA)
```
sudo add-apt-repository ppa:linuxuprising/java
sudo apt update
sudo apt install oracle-java11-installer
cd /etc/apt/sources.list.d
ls
```
**Question 2:** Que contient le fichier dans /etc/apt/sources.list.d
Ce fichier contient l'adresse de dépot du répertoire virtuel. Cette adresse permet d'accéder aux éléments du répertoire.

### Exercice 7

**Question 1:**
Dans le dossier scripts créé lors du TP 2, créez un sous-dossier origine-commande où vous créerez un
sous-dossier DEBIAN, ainsi que l’arborescence usr/local/bin où vous placerez le script écrit à l’exercice 2

```
cd ~/scripts
mkdir -p origine-commande/DEBIAN
cd ..
mkdir -p usr/local/bin
cp ~/origine-commande ./origine-commande
```

**Question 2:**  Dans le dossier DEBIAN, créez un fichier control avec les champs suivants :

```
cd origine-commande/DEBIAN
vim control

Package: origine-commande #nom du paquet
Version: 0.1 #numéro de version
Maintainer: Foo Bar #votre nom
Architecture: all #les architectures cibles de notre paquet (i386, amd64...)
Description: Cherche l'origine d'une commande
Section: utils #notre programme est un utilitaire
Priority: optional #ce n'est pas un paquet indispendable
```

**Question 3:**  Pour créer un paquet
```
cd $HOME
dpkg-deb --build origine-commande
```

#### Création du dépôt personnel avec reprepro

**Question 1:** Dans votre dossier personnel, commencez par créer un dossier repo-cpe. Ce sera la racine de votre
dépôt
```
mkdir repo-cpe
cd repo-cpe
```

**Question 2:** Ajoutez-y deux sous-dossiers : conf (qui contiendra la configuration du dépôt) et packages (qui contiendra nos paquets)
```
mkdir conf
mkdir packages
```

**Question 3:** Dans conf, créez le fichier distributions suivant
```
cd conf
vim distributions
Origin: Un nom, une URL, ou tout texte expliquant la provenance du dépôt
Label: Nom du dépôt
// Suite: stable
Codename: cosmic #le nom de la distribution cible
Architectures: i386 amd64 #(architectures cibles)
Components: universe #(correspond à notre cas)
Description: Une description du dépôt
```

**Question 4:** Dans le dossier repo-cpe, générez l’arborescence du dépôt avec la commande
```
reprepro -b . export
```

**Question 5:** Copiez le paquet origine-commande.deb créé précédemment dans le dossier packages du dépôt, puis,à la racine du dépôt, exécutez la commande
```
reprepro -b . includedeb cosmic origine-commande.deb
```

**Question 6:** Il faut à présent indiquer à apt qu’il existe un nouveau dépôt dans lequel il peut trouver des logiciels. Pour cela, créez (avec sudo) dans le dossier /etc/apt/sources.list.d le fichier repo-cpe.list contenant :
deb file:/home/VOTRE_NOM/repo-cpe cosmic multiverse
```
sudo vim /etc/apt/sources.list.d/repo-cpe.list
```


#### Signature du dépôt avec GPG

**Question 1:** Commencez par créer une nouvelle paire de clés avec la commande
```
gpg --gen-key
```

**Question 2:** Ajoutez à la configuration du dépôt (fichier distributions) la ligne suivante :
```
vim distribution
SignWith: yes
```

**Question 3:** Ajoutez la clé à votre dépôt :
```
reprepro --ask-passphrase -b . export
```

**Question 4:** Ajoutez votre clé publique à votre dépôt avec la commande
```
gpg --export -a "auteur" > public.key
```

**Question 5:** Enfin, ajoutez cette clé à la liste des clés fiables connues deapt:
```
sudo apt-key add public.key
```

### Exercice 8

**Question 1:** Commencez par cloner le dépôt git suivant :

```
git clone https://github.com/jubalh/nudoku
```

**Question 2:**  Rendez vous dans le dossier nudoku qui vient d’être créé et lancez la commande autoreconf -i (ainsi que spécifié dans le fichier README.md). A vous d’installer les éventuels paquets manquants (un peu d’aide : pour résoudre le problème de la macro AM_GNU_GETTEXT manquante, installez le paquet gettext).Relancez la commande autoreconf -i après chaque paquet installé jusqu’à cequ’elle se termine sans erreur.

```
cd nudoku
vim README.md
sudo apt-get install autoconf
sudo apt-get install gettext

```

**Question 3:**  Exécutez le script configure
```
./configure
```

**Question 4:** Normalement, à cette étape on exécute la commandemake install, qui procède à la compilation proprement dite et à l’installation (copie des fichiers compilés dans leur dossier de destination). Mais dans notre cas, on va demander à checkinstall de s’en charger et de créer un paquet au format.deb:
```
sudo checkinstall
```
