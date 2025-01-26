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

## Requirements
Before being able to work with the spotify api, you first gotta setup OAuth settings
as explained at the bottom [at the spotify-contrib page](https://flows.nodered.org/node/node-red-contrib-spotify)

## Table of Contents
I will focus on the use cases I needed in my implementation, but if you require other
API endpoints to work you can try out the different formats of input data I have shown here.

### Start Playback of a Playlist
The function node right before the `Spotify:play` node should contain the following code in some form:
```js
msg.params = [{
    context_uri: "spotify:playlist:0hKdgfk66xiKChBS3fuc9O" // Replacew \w playlist/album URI
}];
return msg;
```
So `list<object>`.


### Change Playback Device
Although the official spotify docs tell you to provide an object with the `device_ids` key,
in Node Red you need a list of a list.
```js
msg.params = [["77df5afe949e308e43d4d2c8ffedf00b16c879bb"],true];
return msg;
```
The format is `list<list,bool>`, with the boolean defaulting to false and telling spotify to 
start playback on the new device. 



