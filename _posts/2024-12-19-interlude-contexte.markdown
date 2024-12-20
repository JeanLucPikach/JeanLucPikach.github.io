---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: un peu de contexte en guise d'interlude"
date:   2024-12-19
categories: RP2040 Dessassemblage
---


Ce billet est le troisieme d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Il est temps de donner un peu de contexte sur le firmware qui est discuté dans cette suite d'articles.

### Le bytebeat

Le bytebeat est une methode de generation de musique procédurale. Pour ce faire, une fonction mathématique f(t) est évaluée.La variable "t" representant souvent le temps, et f(t) correspond alors au son a l'instant t. Souvent exprimé sur un octet, la fonction `f(t) = t` est alors une onde en dent de scie allant de 0 a 255 sur une periode de 255 echantillons. C'est alors le taux d'echantillonage qui défini le tempo de notre musique. Souvent, car historiquement, le taux d'echantillonage est de 8khz. Dans notre exemple précedent, nous aurions donc une onde en dent de scie a une frequence de 31hz.   

### Fluorine

> [Bonjour, inspecteur Disiz !  Vous avez les papiers de la boîte à rythmes s'il vous plaît? Elle est en règle?] [inspecteur-link]



[inspecteur-link]: https://www.youtube.com/watch?v=UFqZQJAg1IQ
[bb-link]: https://stellartux.github.io/websynth/guide.html
