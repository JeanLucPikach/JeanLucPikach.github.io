---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: un peu de contexte en guise d'interlude"
date:   2024-12-19
categories: RP2040 Dessassemblage
---


Ce billet est le troisieme d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Il est temps de donner un peu de contexte sur le firmware qui est discuté dans cette suite d'articles.

### Le bytebeat

Le bytebeat est une methode de generation de musique procédurale. Pour ce faire, une fonction mathématique f(t) est évaluée.La variable "t" representant souvent le temps, et f(t) correspond alors au son a l'instant t. Souvent exprimé sur un octet, la fonction `f(t) = t` est alors une onde en dent de scie allant de 0 a 255 sur une periode de 255 echantillons. C'est alors le taux d'echantillonage qui défini le tempo de notre musique. Souvent, car historiquement, le taux d'echantillonage est de 8khz. Dans notre exemple précedent, nous aurions donc une onde en dent de scie a une frequence de 31hz.   

[Plus de details ici] [bb-link]
### Fluorine

> [Bonjour, inspecteur Disiz !  Vous avez les papiers de la boîte à rythmes s'il vous plaît? Elle est en règle?] [inspecteur-link]


Au cours de mes recherches, je suis tombé sur un synthétiseur amateur nommé [Fluorine] [fluorine-link]. Malheureusement, la personne qui le commercialise n'est pas très avare en informations, et je ne peut m'assurer de l'inter-opérabilité de ce synthétiseur avec le reste de mon setup. Je ne sais meme pas quels sont les formules utilisés lors de la génération de musique, leurs taux d'echantillonage... rien. Tout ce que je peux voir c'est un pico, 8 potards, et des démo sonores dont je ne sais meme pas comment elles ont été réalisées.   

> Another bit that it adds to the classic bytebeat synthesis is the addition of modular and stackable algorithms that process the mathematics under the hood and a recursive Mode that cycles between this algorithms and that turns it all into a new whole synthesis technique, now called "Recursive HD Bytebeats", that altogeher with the stereo modes and Bitdepth features turns this machine, the Fluorine, in probably most advanced way to explore Bytebeat synthesis as in november of 2023.

> Merci mec 👍 

Le prix prohibitif du dit synthétiseur (150 euros!) ainsi que sa [documentation][fluorine-doc] qui ne m'avance pas plus ne me donne pas envie de courrir de risques en depensant autant d'argent. En revanche, le fait que son firmware soit accessible sur le même dépot [git][fluorine-git] que sa documentation me rends... Joueur. Il est donc temps de jeter un oeuil de manière un peu plus approfondie avant de proceder a l'achat. 


[inspecteur-link]: https://www.youtube.com/watch?v=UFqZQJAg1IQ
[bb-link]: https://stellartux.github.io/websynth/guide.html
[fluorine-link]: https://www.tindie.com/products/jc2046/fluorine-futuristic-beat-generator/
[fluorine-doc]: https://github.com/spherical-sound-society/fluorine/blob/main/Fluorine%20user%20guide%201.0.pdf
[fluorine-git]: https://github.com/spherical-sound-society/fluorine/blob/main
