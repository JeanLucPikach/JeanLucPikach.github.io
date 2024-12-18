---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: a la conquete du main"
date:   2024-12-18
categories: RP2040 Dessassemblage
---


Ce billet est le second d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Maintenant bien équipé, nous allons commencer notre descente dans les entrailles du raspberry pico a la recherche de points d'entrées.

### Par ou commencer? 


Maintenant que nous avons reussi a ouvrir notre firmware dans cutter, il faut désormais comprendre ou regarder. En effet, si un survol rapide nous fait appercevoir des endroits qui semblent effectivement correspondre a du code et des fonctions bien délimitées, d'autres endroits sont plus... cryptiques. 


```
0x10000468      push    {r1}
0x1000046a      mov     r1, lr
0x1000046c      lsrs    r1, r1, 1
0x1000046e      lsls    r1, r1, 1
0x10000470      ldrb    r1, [r1, r0]
0x10000472      lsls    r1, r1, 1
0x10000474      add     lr, r1
0x10000476      pop     {r1}
0x10000478      bx      lr

```
> Hmmm je ne sais pas encore ce que c'est, mais j'ai comme l'impression qu'il s'agit d'une fonction légitime

```
0x10000110      lsls    r1, r0, 7
0x10000112      asrs    r0, r0, 0x20
0x10000114      lsls    r1, r0, 7
0x10000116      asrs    r0, r0, 0x20
0x10000118      lsls    r1, r0, 7
0x1000011a      asrs    r0, r0, 0x20
0x1000011c      lsls    r1, r0, 7
0x1000011e      asrs    r0, r0, 0x20
0x10000120      lsls    r1, r0, 7
0x10000122      asrs    r0, r0, 0x20
0x10000124      lsls    r1, r0, 7
0x10000126      asrs    r0, r0, 0x20
0x10000128      lsls    r1, r0, 7
0x1000012a      asrs    r0, r0, 0x20
0x1000012c      lsls    r5, r0, 0xd
0x1000012e      asrs    r0, r0, 0x20
0x10000130      lsls    r1, r0, 7
0x10000132      asrs    r0, r0, 0x20
0x10000134      lsls    r1, r0, 7
0x10000136      asrs    r0, r0, 0x20
0x10000138      lsls    r1, r1, 0xf
0x1000013a      asrs    r0, r0, 0x20
0x1000013c      lsls    r5, r2, 0xf
0x1000013e      asrs    r0, r0, 0x20
0x10000140      lsls    r5, r1, 7
0x10000142      asrs    r0, r0, 0x20
0x10000144      lsls    r5, r1, 7
0x10000146      asrs    r0, r0, 0x20
```
> Mais qu'est-ce que je regarde???


### Un peu de lecture 

Reprenons notre [datasheet] [dsh-link] et interessons nous a la section 2.7, la boot sequence. Cette dernière nous apprends que lorsque le RP2040 sort de reset, celui-ci va effectuer la sequence suivante: 

1. Tout d'abord, un moment de flottement va être observé, de maniere a laisser les clocks se stabiliser. Cela ne nous sert a rien, mais valait la peine d'etre mentionné.
2. Ensuite, le code situé en ROM, une section de mémoire en lecture seule et flashé en usine va être executé
	- Si nous sommes sur le core 1, alors nous nous endormons, pas de multithreading tout de suite 
	- Si nous sommes sur le core 0, alors nous pouvons continuer
3. Le code utilisateur se situant en flash et cette dernière se trouvant derrière un lien QSPI, le CPU va configurer les pins du controlleur QSPI et le controlleur lui même dans "un etat par defaut, pas optimal mais relativement universel". Cela lui permettant de recuperer les 256 premiers octets de manière peu rapide, mais de manière a prendre en charge le plus de puce de flash QSPI possible.
4. Le bootrom saute dans ces 256 premiers octets a l'octet 0 de la flash (donc a l'addresse 0x10000000) et commence a executer le code dans ce qui peut-etre considéré comme le deuxieme etage d'une fusée (the second stage bootloader).  

### Le deuxieme etage du bootloader

Le deuxieme etage du bootloader varie d'une board a l'autre, et n'est plus propre au micro-controlleur lui-même. Son rôle va etre de configurer le controlleur QSPI de manière plus personnalisée vis a vis de la puce de flash utilisé par la devboard. Ce faisant, il existe [différents second stage bootloader] [boot2-link], selon la puce qui aura été adjointe par la personne aillant désigné la board. Le second stage bootloader va alors ensuite configurer le hardware chargé de mettre en cache les données recues de la flash pour permettre au RP2040 de les utiliser sans avoir a les stocker en RAM d'abord. Ce hardware porte un nom: le XIP pour "eXecute In Place". A partir de là, c'est toute la flash qui est adressable et non plus ses 256 premiers octets. Il est donc possible d'atteindre les addresses au dela de 0x10000100. Et cela tombe bien ce les données a partir de cette addresse qui nous interesse pour cette fin de second stage bootloader. 


### L'arm VTOR

Vous aimez la [préfiguration] [fore-link] ? Moi personnellement j'aime beaucoup. Il se trouve que le second bloc de code présenté dans ce billet, n'avait pas de sens car ce n'etait précisement pas du code mais de la donnée. La voici lorsque l'on dit a Cutter de ne pas chercher a la désassembler: 


```
0x10000100      .dword 0x20040000
0x10000104      .dword 0x100001e3
0x10000108      .dword 0x100001c3
0x1000010c      .dword 0x100003e9
0x10000110      .dword 0x100001c1
0x10000114      .dword 0x100001c1
0x10000118      .dword 0x100001c1
0x1000011c      .dword 0x100001c1
0x10000120      .dword 0x100001c1
0x10000124      .dword 0x100001c1
0x10000128      .dword 0x100001c1
0x1000012c      .dword 0x10000345
0x10000130      .dword 0x100001c1
0x10000134      .dword 0x100001c1
0x10000138      .dword 0x100003c9
0x1000013c      .dword 0x100003d5
0x10000140      .dword 0x100001cd
0x10000144      .dword 0x100001cd
0x10000148      .dword 0x100001cd
0x1000014c      .dword 0x100001cd
0x10000150      .dword 0x100001cd
```



[dsh-link]: https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#%5B%7B%22num%22%3A131%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C115%2C478.854%2Cnull%5D
[fore-link]: https://fr.wikipedia.org/wiki/Foreshadowing
[boot2-link]: https://github.com/raspberrypi/pico-sdk/tree/master/src/rp2040/boot_stage2
