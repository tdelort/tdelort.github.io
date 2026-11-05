---
title: "Veer Framework"
permalink: /portfolio/veer/
excerpt: "A HAL with many helpers to create a custom game engine with very few dependencies"
header:
  teaser: /assets/images/portfolio/veer/teaser.png
classes: wide
---

Veer is a framework I am working on that I will use to create a tiny game engine. Its first goal is to abstract the **rendering APIs** (D3D12, Vulkan, and Metal), but it also contains **custom containers**, **glm** like vector type, and will abstract **OS features** (like window, event handling, filesystem, mutex/critical section, etc). The main goal being to have as few dependencies as possible (not even the STL, because why not).  

You can find the code of the framework [here on Github](https://github.com/tdelort/Veer).

Here are some blog posts I already posted on things related to Veer :  

{% include collection.html 
    entries_layout="text" 
    category="veer-devblog"
    include_date="true"
%}