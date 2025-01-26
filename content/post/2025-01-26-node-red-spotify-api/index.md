---
title: Working with the Node Red Spotify Nodes
subtitle: Helping you not get demoralized by the weird inputs
date: 2025-01-26
tags: ["node-red", "spotify"]
---

Being able to easily and quickly integrate different workflows is at the core of 
[Node Red](https://nodered.org/). In my case that means letting hardware control
what and where music is playing by simply sending a MQTT packet to my server 
(running mosquitto on a pi) which is monitored by Node Red.

The problem arises when working with the [node-red-contrib-spotify](https://flows.nodered.org/node/node-red-contrib-spotify)
package, which is really inconsistent with the types of input it requires and lets
you guess between a list, a list of lists or a list of JSON objects. To conquer this
for myself and future developers (since right now even chatGPT has no clue) I decided to
put down my findings with easy to follow examples right here, right now. 




