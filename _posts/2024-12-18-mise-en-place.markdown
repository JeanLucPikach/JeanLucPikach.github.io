---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: mise en place"
date:   2024-12-18 11:04:45 +0100
categories: RP2040 Dessassemblage
---


Ce billet est le premier d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Nous verrons ici quels sont les outils qui vont nous servir lors de notre descente dans les entrailles d'une puce bien connu des makers: le chip RP2040 de la Rasberry fondation.

### Le debug ne tient qu'a un fil

Le RPI pico est un soc ARM cortex-M0+, comme toute puce cortex M cette derniere utilise le protocole SWD (single wire debug). La premiere des choses a faire sera donc d'établir un lien sur cette interface pour diverse raison: 

- Si le binaire du firmware n'est pas disponible, il sera possible de dumper la flash depuis ce lien

- Cela va permettre de dumper les registres, de manière a obtenir de précieuses informations telles que les interruptions et les péripheriques utilisés

- L'analyse statique a ses limites, et cela va permettre de d'analyser dynamiquement notre binaire  

A cette fin de nombreuses sonde SWD existent, la plupart etant compatible avec openOCD, un outil open source (c'est comme le porc salut) permettant de lancer un serveur GDB controllant notre MPU via le lien SWD. Après quelques recherches, une sonde a particulierement attirée mon attention pour deux raisons: la [Black magic debug probe] [bm-link]
- La premiere raison etant qu'elle semble compatible avec un maximum de micro-controlleur possedant un lien SWD (la ou un ST Link par exemple ne sera compatible qu'avec un MCU ST). 
- La deuxieme raison est le fait que son firwmare ai été décliné sur plusieurs hardware dont une devboard dont mes tiroirs débordent: les [black pills avec leurs MCU STM32F4][bp-link] disponibles pour quelque euros sur aliexpress.    

La mise en place est on ne peut plus simple: on télécharge le firmware de la sonde et on le flash sur notre board en suivant ce [readme] [rdbpbm-link], puis on connecte notre nouvelle sonde a notre RP2040 en suivant le pinout indiqué: 

|RP2040|Sonde|
|---|---|
|SWDIO|B9|
|SWCLK|B8|

Dès lors, il n'y a plus qu'a lancer GDB et à se connecter au serveur: 

```
> target extended-remote /dev/ttyACMx
> monitor swdp_scan

Available Targets:
No. Att Driver
 1      Raspberry RP2040 M0+
 2      Raspberry RP2040 M0+
 3      Raspberry RP2040 Rescue (Attach to reset!) 

> attach x 

Attaching to Remote target
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x10000ad8 in ?? ()
>
 
```

> *hacker voice*: i'm in




[bm-link]: https://www.black-magic.org/
[bp-link]: https://stm32-base.org/boards/STM32F411CEU6-WeAct-Black-Pill-V2.0.html
[rdbpbm-link]: https://github.com/koendv/blackmagic-firmware/blob/master/INSTALL.md
