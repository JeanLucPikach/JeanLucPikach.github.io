---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: a la conquete du main - partie 3"
date:   2024-12-24
categories: RP2040 Dessassemblage
---


Ce billet est le quatrieme d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Nous venons d'arriver a l'initialisation de l'ordonnanceur, la boucle du main n'a jamais été aussi proche! 


### Un peu de théorie 

La librairie arduino est basée sur un OS temps réel (RTOS), ARM Mbed. Dans le cadre d'un RTOS, le code est séparé en tache, et ces taches sont gérées par un ordonnanceur qui va choisir a interval régulier quelle tache lancer/continuer/arreter. Cet ordonnanceur tourne dans un mode dit privilegié, alors que les taches elles tournent dans un mode dit utilisateur. 
Ces modes ne sont pas que des constructions logicielles, ils sont implémentés dans le processeur en lui meme (il y a par exemple en réalité deux registres stack pointer, un pour chaque mode!). On peut passer en mode utilisateur de maniere transparente depuis le mode privilegié, mais on ne peut plus repasser en mode privilegié de maniere transparente. On peut cependant trigger un appel systeme, ce dernier va faire sauter le processeur a l'addresse a laquelle se trouve le code censé etre privilegié (dans notre cas l'ordonnanceur). Ce dernier choisira de revenir là ou se situait l'appel systeme en mode utilisateur comme si rien ne s'etait passé ou bien de passer a un autre processus. 
Il est également possible, et c'est monnaie courante pour un ordonnanceur de repasser de manière periodique en mode privilegié pour s'assurer qu'un processus ne monopolise pas le processeur. Dans le cadre d'ARM Mbed et donc par extension du SDK arduino pour la RP2040, la fonction main est en réalité un thread. Lors de la création d'un thread, l'ordonnanceur est donc appelé avec en parametre un pointeur vers la fonction executée par ce thread.   

### Reprenons 

```
fcn.100097d0();
; var int32_t var_10h @ stack - 0x10
0x100097d0      push    {r0, r1, r4, lr}
0x100097d2      ldr     r3, [0x10009818]
0x100097d4      ldr     r4, [0x1000981c]
0x100097d6      movs    r1, 0
0x100097d8      str     r3, [r4, 0x10]
0x100097da      movs    r3, 0x80
0x100097dc      lsls    r3, r3, 8
0x100097de      str     r3, [r4, 0x14]
0x100097e0      movs    r3, 0x44   ; 'D'
0x100097e2      str     r3, [r4, 0xc]
0x100097e4      ldr     r3, [0x10009820]
0x100097e6      movs    r2, r4
0x100097e8      str     r3, [r4, 8]
0x100097ea      movs    r3, 0x18
0x100097ec      str     r3, [r4, 0x18]
0x100097ee      ldr     r3, [str.main] ; 0x10018e33
0x100097f0      ldr     r0, [aav.aav.0x100097ad]
0x100097f2      str     r3, [r4]
0x100097f4      bl      fcn.1000949c ; fcn.1000949c
0x100097f8      subs    r3, r0, 0
0x100097fa      bne     0x10009808
0x100097fc      movs    r2, r4
0x100097fe      ldr     r1, [str.Pre_main_thread_not_created] ; 0x10018e38
0x10009800      str     r0, [sp]
0x10009802      ldr     r0, [0x10009830] ; int16_t arg2
0x10009804      bl      fcn.1000d1a0 ; fcn.1000d1a0
0x10009808      bl      fcn.10007d98 ; fcn.10007d98
0x1000980c      movs    r2, 0
0x1000980e      ldr     r1, [str.Failed_to_start_RTOS] ; 0x10018e54
0x10009810      movs    r3, r2
0x10009812      str     r2, [sp]
0x10009814      b       0x10009802
0x10009816      mov     r8, r8
0x10009818      .dword 0x20002f78
0x1000981c      .dword 0x20002f4c
0x1000981e      movs    r0, 0
0x10009820      .dword 0x20002e04
0x10009824      .dword 0x10018e33
0x10009828      .dword 0x100097ad ; fcn.100097ac
0x1000982c      .dword 0x10018e38 ; aav.0x10018e38 ; str.Pre_main_thread_not_created
0x10009830      .dword 0x8001011d
0x10009834      .dword 0x10018e54 ; aav.0x10018e54 ; str.Failed_to_start_RTOS
```

