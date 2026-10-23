---
title: "Parlons images par seconde"
lang: fr
---

Le projet avance bien, et il est temps de se poser pour faire une petite mise à jour technique. Un point crucial dans ce portage de Doom, c'est le nombre d'images par seconde. Sur PC, Doom tourne à 35 images par seconde. Sur Super Nintendo, on est entre 10 et 15, et sur 32X entre 15 et 20. Mais combien est-ce que je pourrais en obtenir concrètement ?

## Comment ça marche

L'architecture établie est la suivante : le RP2350, le coprocesseur de la cartouche, calcule une image et l'écrit dans une mémoire de la cartouche (la SRAM). Il passe ensuite la main à la Megadrive, qui recopie ces données dans sa mémoire vidéo (la VRAM). Il y a deux SRAM : pendant que la Megadrive recopie une image depuis l'une, le RP2350 prépare la suivante dans l'autre.

Il y a donc trois étapes : le calcul de l'image, son écriture en SRAM, et sa copie en VRAM. L'écriture en SRAM devrait être brève, et ne sera mesurable que sur un prototype physique. En revanche, les deux autres étapes sont mesurables dès maintenant, et doivent l'être. Ce n'est qu'une fois ces chiffres connus que l'on pourra avancer sur l'émulateur, pour qu'il reproduise le comportement de la cartouche aussi fidèlement que possible.

C'est pour cela que j'ai changé l'ordre annoncé dans le billet précédent : avant de modifier un émulateur, il faut connaître les temps qu'il devra respecter.

## La résolution

La Megadrive a deux modes d'affichage en largeur : H40, avec une image de 320 pixels de large, et H32, avec une image de 256 pixels de large. Les écrans fixes seront envoyés en H40, car ils ne demandent pas de débit élevé. Cela inclut l'écran titre, les écrans entre les niveaux et de fin de jeu, ainsi que les menus : quand on ouvre un menu, il s'affiche en H40 sur un fond noir, et le jeu revient à l'écran précédent à la fermeture. Les démos qui tournent derrière le menu dans le Doom d'origine sont donc désactivées. En revanche, pour limiter la quantité de données à générer et à transférer, les niveaux seront envoyés en H32, avec la barre d'état (le HUD) stockée dans la ROM et affichée par la console elle-même.

## Côté Megadrive

J'ai bien avancé sur la ROM, le programme qui tourne sur le processeur de la console. Le but est d'avoir des conditions aussi proches que possible de la version finale. Elle sait donc afficher les écrans fixes en H40, et les niveaux en H32 en ajoutant la barre d'état. Je la teste pour l'instant dans l'émulateur BlastEm, avec des images de test générées.

À chaque changement de mode, un fondu au noir remplace l'effet de « fonte » de Doom, où l'image coule vers le bas de l'écran. Pendant ce fondu, Doom se met en pause : la transition ne fait pas perdre de temps au joueur, quelle que soit sa durée.

Et les premiers chiffres sont tombés.

La Megadrive rafraîchit l'écran 60 fois par seconde en NTSC (Japon, Amérique du Nord), contre 50 fois en PAL (Europe). On ne peut pas changer ces chiffres. On affichera donc la même image à l'écran jusqu'à ce que la suivante soit prête. Avec les images de test, j'ai pu établir qu'il faut 3 rafraîchissements de l'écran pour transférer une image de niveau en NTSC, et seulement 2 en PAL. Cette différence vient du fait que la console ne peut copier des données en VRAM rapidement qu'entre deux images, quand l'écran n'est pas en train d'être dessiné : en PAL, l'écran compte plus de lignes pour la même image affichée, et cette pause est nettement plus longue.

On peut en conclure que le nombre d'images par seconde pourrait atteindre 20 en NTSC (60/3) et 25 en PAL (50/2). Dans le billet précédent, j'estimais le plafond à 30 images par seconde en NTSC : la mesure le ramène à 20, ce qui reste mon objectif minimum. En PAL, les 25 images par seconde sont confirmées.

Ces chiffres ne concernent que l'affichage : le jeu lui-même continue de tourner à sa vitesse normale, il montre simplement moins d'images.

Mais ils ne sont valables que si le RP2350 met moins de temps à calculer ses images, ce qu'on ne saura qu'après d'autres tests.

## Latence

Autre point important : la latence. Combien de temps s'écoule entre le moment où le RP2350 commence à calculer une image et le moment où la Megadrive l'affiche ? En partant du principe que le RP2350 tient la cadence de la Megadrive, 20 images par seconde en NTSC signifient une image toutes les 0,05 s. Le RP2350 calcule une image pendant que la Megadrive transfère la précédente : une image passe donc 0,05 s à être calculée, puis 0,05 s à être transférée. La latence est ainsi de 0,1 s entre le début du calcul et l'affichage. En PAL, ce chiffre est de 0,08 s.

## La suite

Prochaine étape : mesurer le temps que met le RP2350 pour calculer une image. Les tests seront faits avec un Raspberry Pi Pico 2, en commençant par y faire tourner Doom.

Pour la version finale de la ROM, il reste à gérer les flashs de couleur dans la barre d'état, et à redessiner les visages.