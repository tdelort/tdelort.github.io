---
title: "GreedFall: The Dying World"
permalink: /greedfall-tdw/
excerpt: "Non exhaustive list of subjects I worked on during the production of the second GreedFall opus at Spiders"
header:
  image: /assets/images/portfolio/greedfall2/header.png
  teaser: /assets/images/portfolio/greedfall2/teaser.png
classes: wide
---

During the 3 years I was at Spiders, I had the opportunity to work on many features for our in house game engine, the SilkEngine, during the production of GreedFall: The Dying World.  

## Participating Media  

I worked on many aspects of the *Volumetric Fog* rendering in the *SilkEngine*. We started from [Wronski2014](https://bartwronski.com/wp-content/uploads/2014/08/bwronski_volumetric_fog_siggraph2014.pdf) and added multiple features and fixes (less noise, multiple phases, colored absorption, finer integration).  

![Volumetric Fog]({{site.baseurl}}/assets/images/portfolio/greedfall2/participating_media_0.png)

From the get go, it was important that the participating media could handle **very dense fog**. One very important aspect of *GreedFall: The Dying World* is the water (you switch levels by going through a boat hub, and many of such levels are on the seaside, filled with rivers, or both). So we wanted to be able to use the aforementioned volumetric fog system for the water.  

![Volumetric Fog Underwater]({{site.baseurl}}/assets/images/portfolio/greedfall2/participating_media_1.png)  

![Volumetric Fog Underwater]({{site.baseurl}}/assets/images/portfolio/greedfall2/participating_media_2.png)  

![Volumetric Fog Underwater]({{site.baseurl}}/assets/images/portfolio/greedfall2/participating_media_3.png)  

## Water  

The first step in improving the water rendering was to go from a *Gerstner Wave* based implementation, with very few octaves hardcoded in a vertex shader, to a *FFT* based one à la [Tessendorf2004](https://jtessen.people.clemson.edu/reports/papers_files/coursenotes2004.pdf) to which we added :  
 - a custom spatial and radial **spectrum**,  
 - **cascades** to drastically limit tiling,  
 - **local** control over the wave orientation (to be able to simulate refraction, reflection, and diffraction) and intensity (a big island will create a shadow behind it),  

![Waves]({{site.baseurl}}/assets/images/portfolio/greedfall2/waves_0.gif)  

![Waves]({{site.baseurl}}/assets/images/portfolio/greedfall2/waves_1.gif)  

We also needed to make it easier for artists to generate flowmaps so I also worked on an **offline fluid simulation** tool to generate rivers, lakes, oceans, and pretty much any body of water larger than our simulation spatial frequency.  
With this system, we simulate the water **flow speed**, **depth**, the **direction** of the waves using the wind parameters and bathymetry information, the inset of air **bubbles** in turbulent water, and (last but not least) **physics geometry** for the water closer to the shore to allow for some particle and sound systems.  

![River]({{site.baseurl}}/assets/images/portfolio/greedfall2/river_0.gif)  

![River]({{site.baseurl}}/assets/images/portfolio/greedfall2/river_1.gif)  

Along the ride, we also added many smaller features like :
 - **caustics reflections** above the water,  
 - flowed underwater caustics,   
 - better flow algorithm to support many different ranges of speed,  
 - small waves around the player's feet,  

## Mesh and Texture Streaming  

We added a **predictive**, per level of detail, meshes and textures streaming system to the *SilkEngine*.  
I mainly worked on the early part of the process that **gathers** all textures and meshes usages, computes for each the **needed LODs**, and **requests load and unload** operation using memory available, current LODs, and other metrics.  
Each frame, for **thousands of meshes and textures** across multiple loaded levels at once, taking into consideration each hardware recommendations (DirectStorage on PC and XBox, and its PS5 equivalent), within a very tight time budget (very SIMD friendly, and highly multithreaded code).  

But a system is as good as it's profiling tools, (and it's probably the only "pretty" picture I have of it), so I also implemented a tool that evaluates for many positions of the camera on the maps what would be the worst case budget needed on a level.  

![Streaming]({{site.baseurl}}/assets/images/portfolio/greedfall2/streaming_0.png)  

## Miscellaneous  

Unsorted list of features I worked on :  
 - GPU resource state tracking and transitions,
 - Lighting pass **tile classification** optimisations,
 - **Clustering** of similar meshes to reduce the CPU processing cost (see images below).

![Clusters]({{site.baseurl}}/assets/images/portfolio/greedfall2/clusters_0.png)  
*Without clustering*

![Clusters]({{site.baseurl}}/assets/images/portfolio/greedfall2/clusters_1.png)  
*With clustering (see how fewer colors there are among shelves items)*

 - Tool to shoot **omnidirectional shadow cookies**,

![Cookies]({{site.baseurl}}/assets/images/portfolio/greedfall2/cookie_0.gif)  
