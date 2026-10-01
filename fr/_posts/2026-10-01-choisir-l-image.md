---
title: "Choisir l'image : le mode H32, des options graphiques et des couleurs à soigner"
lang: fr
---

Depuis le premier billet, le projet a franchi une étape importante : Doom produit désormais directement des images au format de la Megadrive. Le moteur fabrique lui-même les tuiles, les palettes et la grille qui les place, exactement ce que la puce vidéo de la console sait afficher. Pour l'instant, tout cela tourne sur mon Mac, où un petit simulateur de la puce vidéo affiche le résultat. Pas encore sur une Megadrive, ni même sur un émulateur : ce sera la prochaine étape.

Ce billet raconte les choix qui ont mené là.

## Le vrai problème : la mémoire vidéo

La puce vidéo de la Megadrive (le VDP) n'affiche pas des pixels un par un. Elle assemble l'écran à partir de petits carrés de 8×8 pixels, les tuiles, rangés dans sa mémoire vidéo de 64 Ko, et chacun ne peut utiliser que 15 couleurs. Dans un jeu classique, ces tuiles changent peu : un décor qui défile réutilise les mêmes tuiles encore et encore.

Doom, c'est l'inverse. Dès que la caméra bouge, presque toute l'image change. Chaque nouvelle image demande donc de nouvelles tuiles, qu'il faut copier dans la mémoire vidéo pendant le court instant où l'écran n'est pas en train d'être dessiné. Il y a ainsi deux limites : le nombre total de tuiles que la mémoire peut contenir, et le nombre de tuiles à transférer entre deux images, qui limite le nombre d'images par seconde. Mon premier essai visait une image pleine largeur, 320 pixels. Il fallait alors compter sur les tuiles qui se ressemblent pour tenir dans la mémoire, ce qui obligeait à appauvrir l'image… et ça débordait quand même dans plus d'un tiers des images.

## La solution de Virtua Racing

La réponse est venue d'un jeu de 1994 : Virtua Racing. Sa cartouche contient une puce spéciale, la SVP, qui calcule la 3D et dessine le résultat directement sous forme de tuiles, que la console n'a plus qu'à copier. C'est exactement l'architecture de SMDoom. Et Virtua Racing fait un choix décisif : il utilise le mode d'affichage étroit de la Megadrive, appelé H32, avec 256 pixels de large au lieu de 320.

J'ai repris ce choix. La vue de SMDoom fait 256×160 pixels, soit 32×20 = 640 tuiles. Chaque case de l'écran a sa propre tuile, recopiée à chaque image. Le nombre de tuiles est donc toujours le même, quoi qu'il se passe à l'écran : la mémoire vidéo ne peut plus déborder, et le détail de l'image ne coûte plus rien en mémoire.

Ces 640 tuiles représentent environ 20 Ko à copier par image. D'après mes calculs, la console peut en recevoir jusqu'à 30 images par seconde en version 60 Hz, et 25 en version 50 Hz. C'est un plafond théorique : la vraie limite sera sans doute la vitesse du coprocesseur, que je n'ai pas encore mesurée. Mais il est au-dessus de mon objectif de 20 images par seconde, à comparer aux 10 à 15 de la version Super Nintendo et aux 15 à 20 de la version 32X. Clin d'œil : 256×160, c'est aussi la taille de la vue de la version 32X.

Quelques conséquences de ce choix :

- les pixels du mode H32 sont plus larges que hauts ; sans correction, l'image serait étirée d'environ un tiers. Le moteur compense en étirant tout verticalement d'autant ;
- la barre d'état en bas de l'écran sera redessinée en 256 pixels de large et affichée par la console elle-même, pas recalculée à chaque image ;
- les écrans fixes (titre, fins de niveau) gardent leur taille d'origine, 320 pixels : la console change de mode d'affichage à chaque changement d'écran.

## Des options graphiques au choix

Les versions Super Nintendo et 32X de Doom ont dû renoncer à certains effets, en particulier les textures des sols et des plafonds, remplacées par des aplats de couleur. Plutôt que de trancher une fois pour toutes, j'ai découpé le rendu en réglages que je peux activer ou désactiver : sols et plafonds unis ou texturés, trait de séparation entre le sol et les murs, nombre de paliers de lumière. D'autres viendront : textures des murs, ombrage, flashs de couleur quand on est touché ou qu'on ramasse un bonus.

Ça me permet de comparer les variantes sur de vraies images de jeu. Et à terme, j'aimerais proposer certains de ces réglages au joueur, pour qu'il choisisse entre une image plus lisible et une image plus détaillée.

## Les couleurs, pour plus tard

La Megadrive connaît 512 couleurs, mais n'en affiche que quelques dizaines à la fois : 4 palettes de 15 couleurs, et chaque tuile ne peut utiliser qu'une seule palette. Doom, lui, a 256 couleurs et beaucoup de dégradés sombres. Il faut donc, à chaque image, choisir les couleurs de chaque palette et décider quelle palette va à chaque tuile.

Voici où en est ce travail.

![Comparaison de trois rendus de la même image](/assets/images/2026-10-01/compare_029.png)

![Comparaison de trois rendus de la même image](/assets/images/2026-10-01/compare_062.png)

Sur chaque image, celle de gauche est la référence : chaque pixel y prend la couleur la plus proche parmi les 512 de la Megadrive, sans aucune limite de palette. C'est ce qu'on obtiendrait si la console pouvait afficher toutes ses couleurs à la fois. Les deux autres sont des versions de travail, avec les vraies contraintes de palette : au milieu, toutes les couleurs comptent autant ; à droite, l'algorithme donne la priorité aux ennemis et à l'arme. On le voit à la flamme du tir, dont le cœur blanc devient rose au milieu et reste presque blanc à droite. Mais on voit aussi le prix à payer : quelques carrés de sol prennent une mauvaise couleur.

Les images sont agrandies trois fois, avec des pixels carrés : sur la console, elles paraîtront un peu plus larges.

Il reste beaucoup à faire de ce côté. Mais la répartition des couleurs n'a aucun effet sur la vitesse du jeu, et je préfère d'abord prouver que l'ensemble fonctionne. Je reviendrai aux couleurs à la fin.

## La suite

Prochaine étape : faire tourner tout ça sur une Megadrive émulée. Je vais modifier un émulateur pour qu'il simule une cartouche équipée d'un coprocesseur, et écrire le petit programme 68000 qui reçoit les images et les affiche. Ce sera la première fois que SMDoom tournera sur une Megadrive, même virtuelle.