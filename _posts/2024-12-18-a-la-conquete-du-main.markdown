---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: a la conquete du main"
date:   2024-12-18
categories: RP2040 Dessassemblage
---


Ce billet est le second d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Maintenant bien équipé, nous allons commencer notre descente dans les entraille du raspberry pico en cherchant des points d'entrées.

### Par ou commencer? 


Maintenant que nous avons reussi a ouvrir notre firmware dans cutter, il faut désormais comprendre ou regarder. En effet, si un survol rapide nous fait appercevoir des endroits qui semblent effectivement correspondre a du code et des fonctions bien délimitées, d'autres endroits sont plus... cryptique. 


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
