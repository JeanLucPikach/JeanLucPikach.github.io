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
// selon gdb nous n'arrivons jamais ici. 
0x1000020a  ldr   r1, [aav.aav.0x1000d3c5] ; 0x1000026c 
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

### Derniere ligne droite ?

```
fcn.100002cc(int32_t arg_344h);
; arg int32_t arg_344h @ stack + 0x344
0x100002cc  ldr   r3, [0x10000328]
0x100002ce  cmp   r3, 0
0x100002d0  bne   0x100002d4
0x100002d2  ldr   r3, [0x10000324]
0x100002d4  mov   sp, r3
0x100002d6  movs  r2, 0x40   ; '@'
0x100002d8  lsls  r2, r2, 0xa
0x100002da  subs  r2, r3, r2
0x100002dc  mov   sl, r2
0x100002de  movs  r1, 0
0x100002e0  mov   fp, r1
0x100002e2  mov   r7, r1
0x100002e4  ldr   r0, [0x10000334]
0x100002e6  ldr   r2, [0x10000338]
0x100002e8  subs  r2, r2, r0
0x100002ea  bl    fcn.10011918 ; memset
0x100002ee  ldr   r3, [0x1000032c] ; null ptr
0x100002f0  cmp   r3, 0
0x100002f2  beq   0x100002f6
0x100002f4  blx   r3
0x100002f6  ldr   r3, [0x10000330] ; pointeur valant  0x100096d1
0x100002f8  cmp   r3, 0
0x100002fa  beq   0x100002fe
0x100002fc  blx   r3
0x100002fe  movs  r0, 0
0x10000300  movs  r1, 0
0x10000302  movs  r4, r0
0x10000304  movs  r5, r1
0x10000306  ldr   r0, [aav.aav.0x1000d3cb] ; 0x1000033c
0x10000308  cmp   r0, 0
0x1000030a  beq   0x10000312
0x1000030c  ldr   r0, [0x10000340]
0x1000030e  bl    fcn.1000d3ca ; se contente de retourner 1
0x10000312  bl    fcn.1001101c ; fcn.1001101c
//selon GDB nous n'arriveront pas ici non plus
0x10000316  movs  r0, r4
0x10000318  movs  r1, r5
0x1000031a  bl    fcn.10009770 ; main loop :)
0x1000031e  bl    fcn.1000d3c4 ; fcn.1000d3c4
0x10000322  mov   r8, r8
0x10000324  .dword 0x00080000
0x10000328  .dword 0x20040000
0x1000032c  .dword 0x00000000
0x10000330  .dword 0x100096d1 ; fcn.100096d0
0x10000334  .dword 0x20001110
0x10000338  .dword 0x2000b00c
0x1000033c  .dword 0x1000d3cb ; fcn.1000d3ca
0x1000033e  .dword 0x00001000
0x10000340  .dword 0x00000000
```

Nous voici donc arrivé dans la dernière fonction qui sera appelé par le reset vector, nous ne retournerons pas de cette dernière non plus. Nous pourrions être tenté de penser que c'est parceque la boucle infinie de l'utilisateur se touve a la fin lors de l'appel de la fonction `fcn.10009770`, mais il y a un twist. En réalite ce n'est pas le main qui ne rends jamais la main: c'est l'ordonanceur.  

Commencons par lire notre code de manière sequentielle. La première des choses faite est l'initialisation de la stack et du frame pointer. Puis nous appellons la fonction `fcn.10011918` qui est en fait un memset qui initalise une partie de la mémoire a 0. Nous allons ensuite charger un pointeur sur fonction et verifier si il n'est pas nul. Si il l'est nous passons a la suite, si il est non nul nous appelons la fonction. 

Le premier pointeur sur fonction est nul, mais le second nous envoie en 0x100096d1 

```
0x100096d0      ldr     r2, [0x100096f8]
0x100096d2      ldr     r3, [0x100096fc]
0x100096d4      push    {r4, lr}
0x100096d6      str     r2, [r3]
0x100096d8      ldr     r3, [0x10009700]
0x100096da      subs    r3, r3, r2
0x100096dc      ldr     r2, [0x10009704]
0x100096de      str     r3, [r2]
0x100096e0      ldr     r2, [0x10009708]
0x100096e2      ldr     r3, [0x1000970c]
0x100096e4      str     r2, [r3]
0x100096e6      ldr     r3, [0x10009710]
0x100096e8      subs    r3, r3, r2
0x100096ea      ldr     r2, [0x10009714]
0x100096ec      str     r3, [r2]
0x100096ee      bl      fcn.1000979c ; fcn.1000979c
0x100096f2      bl      fcn.100097d0 ; fcn.100097d0
0x100096f6      mov     r8, r8
0x100096f8      .dword 0x2003fc00
0x100096fc      .dword 0x20002a74
0x10009700      .dword 0x20040000
0x10009704      .dword 0x20002a70
0x10009708      .dword 0x2000b00c
0x1000970c      .dword 0x20002a68
0x10009710      .dword 0x2003fc00
0x10009714      .dword 0x20002a64
``` 

La routine dans laquelle nous venons d'atterir joue avec des addresses en RAM (0x2XXXXXXX), passons cette etape et interessons nous a la dernière fonction qui est appelée : fcn.100097d0 


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

Voilà qui est très interessant ! 
