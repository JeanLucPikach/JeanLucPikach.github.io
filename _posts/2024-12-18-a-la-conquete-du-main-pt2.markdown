---
layout: post
title:  "Retro ingenieurie d'un firmware RP2040: a la conquete du main - partie 2"
date:   2024-12-18
categories: RP2040 Dessassemblage
---


Ce billet est le troisieme d'une suite d'articles concernant le dessassemblage d'un firmware tournant sur RP2040. Une fois que nous avons compris la séquence d'amorcage du pico et trouvé le reset vector, nous allons pouvoir comprendre comment est initialisée la stack logicielle. Nous devrions pouvoir voir du code utilisateur a la fin de cette séquence... Si dieu le veut 

### Le reset vector



[dsh-link]: https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#%5B%7B%22num%22%3A131%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C115%2C478.854%2Cnull%5D
[fore-link]: https://fr.wikipedia.org/wiki/Foreshadowing
[boot2-link]: https://github.com/raspberrypi/pico-sdk/tree/master/src/rp2040/boot_stage2
