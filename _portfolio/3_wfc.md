---
title: "WFC"
permalink: /wfc/
excerpt: "A quick Unity implementation of the Wave Function Collapse algorithm"
header:
  image: /assets/images/portfolio/wfc/header.png
  teaser: /assets/images/portfolio/wfc/teaser.png
classes: wide
---

Implementation of the Wave Function Collapse algorithm to generate Backrooms like environments.   

[Source code available here.](https://github.com/tdelort/WFC-Backrooms)

Some ideas were taken from the following talk by Bad North's creator, Oskar Stalberg :

<iframe width="640" height="360" src="https://www.youtube.com/embed/0bcZb-SsnrA?si=xEEaOm3I8MRCW00a&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<br/>

One key idea that could be added is the possibility to make the generation infinite (which would fit the backrooms aesthetic).  
[This marian42 blog post](https://marian42.de/article/wfc/) contains pretty much all that is needed to add this feature.

## Pretty Pictures

{% include image.html path="/assets/images/portfolio/wfc/wfc_modules.png" alt="Image" caption="Overview of the different modules" %}  

{% include image.html path="/assets/images/portfolio/wfc/top_down_view.png" alt="Image" caption="Top down view of a chunk generated using the WFC algorithm" %}  

{% include image.html path="/assets/images/portfolio/wfc/inside_1.png" alt="Image" caption="Inside view with walls colored according to their normal" %}  

{% include image.html path="/assets/images/portfolio/wfc/inside_0.png" alt="Image" caption="Inside view with walls colored according to their normal" %}  
