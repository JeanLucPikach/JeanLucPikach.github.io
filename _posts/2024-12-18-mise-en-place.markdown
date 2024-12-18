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

A cette fin de nombreuses sonde SWD existent, la plupart etant compatible avec openOCD, un outil open source permettant de lancer un serveur GDB controllant notre MPU via le lien SWD. Après quelques recherches, une sonde a particulierement attirée mon attention pour deux raisons: la [Black magic debug probe] [bm-link]
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

### Configuration de cutter

Le choix du désassembleur est un choix plus religieux que technique (meme si j'admet volontier que Cutter a ses defauts, que voulez-vous, le désassembleur choisis son sorcier). Parmis les defauts précédemment evoqués, il y a notamment le fait que cutter ne reconnaisse pas automatiquement la nature du binaire ainsi que la tailles des opcodes lorsqu'il s'agit de thumb2.

![cutter_conf](https://raw.githubusercontent.com/lamashnikov/lamashnikov.github.io/blob/main/img/cutter_conf.png){:.ioda}

 Il faut donc penser a le configurer dans les options avancées au moment du chargement du binaire pour éviter toutes déconvenues. Il faut aussi penser a changer la base addresse. Le firmware vivant en flash, il faut penser a le mapper en 0x10000000 comme indiqué dans la [datasheet] [dsh-link] 





[bm-link]: https://www.black-magic.org/
[bp-link]: https://stm32-base.org/boards/STM32F411CEU6-WeAct-Black-Pill-V2.0.html
[rdbpbm-link]: https://github.com/koendv/blackmagic-firmware/blob/master/INSTALL.md
[dsh-link]: https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#%5B%7B%22num%22%3A27%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C115%2C675.352%2Cnull%5D
