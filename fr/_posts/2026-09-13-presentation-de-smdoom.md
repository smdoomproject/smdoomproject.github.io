---
title: "Présentation de SMDoom"
lang: fr
---

J'essaie de faire tourner Doom sur une Sega Megadrive standard, en utilisant un coprocesseur dans la cartouche, dans l'esprit de ce que Sega faisait avec le SVP de Virtua Racing. J'ai choisi comme coprocesseur le RP2350, utilisé notamment dans le Raspberry Pi Pico 2.

## Pourquoi c'est difficile

Le processeur 68000 et la puce graphique (VDP) de la Megadrive n'ont jamais été conçus pour faire tourner un moteur de rendu 3D en temps réel. Toutes les tentatives sérieuses de porter un rendu façon Doom sur du matériel 16 bits butent sur le même mur : pas assez de puissance CPU, pas assez de mémoire vidéo, et un pipeline d'affichage pensé pour du défilement de tuiles en 2D, pas pour une vue 3D en mouvement libre.

L'approche retenue ici consiste à laisser le coprocesseur faire le vrai calcul 3D (la logique du moteur Doom, la projection, la visibilité), puis à convertir son résultat en quelque chose que la puce vidéo de la Megadrive sait afficher nativement : tuiles, palettes et valeurs de défilement qu'elle sait déjà envoyer à l'écran. Le rôle du 68000 devient alors de recevoir ces données et de les transmettre au VDP, plutôt que de calculer lui-même la scène 3D. 

## Où en est le projet

C'est un projet personnel, mené sur mon temps libre, encore en phase d'exploration. Plusieurs questions de fond ne sont pas encore tranchées : comment faire tenir une vue 3D dans le budget de tuiles et de couleurs de la Megadrive sans dépasser ses limites de mémoire vidéo, à quelle vitesse le coprocesseur peut réellement transmettre les données via le port cartouche, et à quel point le résultat peut s'approcher de quelque chose de vraiment jouable plutôt que d'une curiosité technique tournant à quelques images par seconde.

Je ne sais pas encore si ça va marcher. Cette incertitude fait justement partie des raisons pour lesquelles je voulais un journal public plutôt qu'une grande annonce : je préfère documenter le vrai processus, impasses comprises, plutôt que de laisser croire que le résultat allait de soi dès le départ.

## Objectifs

Le projet sera considéré comme concluant si je parviens à faire tourner Doom sur une vraie Megadrive, avec une qualité graphique et un framerate suffisants pour jouer de façon agréable. Je vise donc au moins 20 images par seconde.

## À quoi s'attendre ici

Des billets courts et occasionnels, publiés au fil des vraies étapes franchies : une approche de rendu qui tient enfin la route, une mesure de framerate, un problème qui s'avère plus dur (ou plus simple) que prévu. Pas de rythme fixe.

Le code complet sera rendu public plus tard, quand le projet sera suffisamment abouti. Un lien sera ajouté ici à ce moment là.
