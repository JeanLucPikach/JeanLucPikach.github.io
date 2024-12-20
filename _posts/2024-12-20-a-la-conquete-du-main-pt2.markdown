---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: a la conquete du main - partie 2"
date:   2024-12-20
categories: RP2040 Dessassemblage
---


Ce billet est le troisieme d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Une fois que nous avons compris la séquence d'amorcage du pico et trouvé le reset vector, nous allons pouvoir comprendre comment est initialisée la stack logicielle. Nous devrions pouvoir voir du code utilisateur a la fin de cette séquence 

### Le reset vector

Bien, maintenant que nous avons compris par ou prendre tout cette assembleur et réalisé que tout code n'est pas bon a lire, il est temps de tirer sur le fil. Sautons donc a l'addresse 0x100001e3 pour commencer a lire de l'assembleur (il etait temps!) 

> NB: certains procos ARM peuvent lire des opcodes en 16 bits ou en 32 bits... Pour savoir a quoi s'attendre, le dernier bit est set a 1 si le code a cette est en 16 bit (comme dans notre cas)  ou 0 en 32 bits. Si une addresse est impaire, il faut donc retrancher 1

 

