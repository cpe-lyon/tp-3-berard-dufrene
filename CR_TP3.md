# Compte rendu TP1
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
La petite différence est dû au fait que dpkg ne prend pas en compte les dépendances alors que apt installe aussi les dépendances nécessaires. APT est un système de management de paquets.
