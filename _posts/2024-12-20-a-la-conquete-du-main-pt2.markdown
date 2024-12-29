---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: a la conquete du main - partie 2"
date:   2024-12-20
categories: RP2040 Dessassemblage
---


Ce billet est le troisieme d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Une fois que nous avons compris la séquence d'amorcage du pico et trouvé le reset vector, nous allons pouvoir comprendre comment est initialisée la stack logicielle. Nous devrions pouvoir voir du code utilisateur a la fin de cette séquence 

### Le reset vector

Bien, maintenant que nous avons compris par ou prendre tout cet assembleur et réalisé que tout code n'est pas bon a lire, il est temps de tirer sur le fil. Sautons donc a l'addresse 0x100001e3. 

> NB: certains procos ARM peuvent lire des opcodes en 16 bits ou en 32 bits... Pour savoir a quoi s'attendre, le dernier bit est set a 1 si le code a cette addresse est en 16 bit (comme dans notre cas)  ou 0 en 32 bits. Si une addresse est impaire, il faut donc retrancher 1

```
0x100001e2  ldr   r0, [0x1000025c] ; 0xd0000000
0x100001e4  ldr   r0, [r0]
0x100001e6  cmp   r0, 0
0x100001e8  bne   0x10000246
0x100001ea  adr   r4, 0x30   ; 0x1000021c
0x100001ec  ldm   r4!, {r1, r2, r3}
0x100001ee  cmp   r1, 0      ; arg2
0x100001f0  beq   0x100001f8
0x100001f2  bl    fcn.10000216 ; fcn.10000216
0x100001f6  b     0x100001ec
0x100001f8  ldr   r1, [0x10000260] ;  0x20001110
0x100001fa  ldr   r2, [0x10000264] ; 0x2000b00c
0x100001fc  movs  r0, 0
0x100001fe  b     0x10000202
0x10000200  stm   r1!, {r0}
0x10000202  cmp   r1, r2
0x10000204  bne   0x10000200
0x10000206  ldr   r1, [aav.aav.0x100002cd] ; 0x100002cd
0x10000208  blx   r1
0x1000020a  ldr   r1, [aav.aav.0x1000d3c5] ; 0x1000026c 
// selon gdb nous n'arrivons jamais ici. 
0x1000020c  blx   r1
0x1000020e  bkpt  0
``` 

Le registre situé en 0xd0000000 est un registre commun a tous les processeurs ARM. Il contient le numéro de CPU actuellement entrain de tourner. Les 4 premieres lignes sont dont une verification du core actuellement entrain de tourner, si nous sommes sur le deuxieme core alors nous sautons en 0x10000246 qui est une boucle infinie (toujours pas de multi-threading pour l'instant!). 

La suite du code correspond a la copie des sections data, scratch_x, et scratch_y. L'adresse 0x1000021c correspond a un tableau de pointeurs. 

```
0x1000021c  .dword 0x1001b43c
0x10000220  .dword 0x200000c0
0x10000224  .dword 0x20001104
0x10000228  .dword 0x1001c48c 
0x1000022c  .dword 0x20040000
0x10000230  .dword 0x20040000
0x10000234  .dword 0x1001c48c 
0x10000238  .dword 0x20041000
0x1000023c  .dword 0x20041000
0x10000240  .dword 0x00000000
```

Les trois premiers elements du tableau sont chargés dans les registres r1, r2, r3. Le registre r1 correstion a l'adresse de debut de la section en flash (la source), le registre r2 correspond a la destination, la taille de la section a copier correspond a la différence `r3-r2`. La fonction `fcn.10000216` se charge de la copie en temps que tel. Puis la section boucle pour recuperer les 3 pointeurs suivants jusqu'a ce qu'un \0 indique la fin du tableau de pointeurs a copier. Nous noterons egalement que les sections scratch_x et scratch_y sont vides (r2 et r3 etant egaux).  

```
0x10000212  ldm   r1!, {r0}  <-|
0x10000214  stm   r2!, {r0}    |
0x10000216  cmp   r2, r3       |
0x10000218  blo   0x10000212  -|
0x1000021a  bx    lr
```

Une fois la copie de ces sections effectués, le code efface la section bss de `0x20001110` à `0x2000b00c`. Le code charge alors un pointeur vers la fonction situé en `0x10000268` et saute dedans. Selon GDB nous n'en retournerons jamais, et nous allons comprendre pourquoi en suivant le code. 
