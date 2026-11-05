---
title: "Cosplay : Hatsune Miku and the Hohmann transfer skirt"
slug: hohmann-transfer-skirt 
excerpt: "Pattern design of Camellia's \"Insomnia C(o)urse\" Miku skirts with a little bit of orbital mechanics"
header_image: /assets/images/posts/hohmann-transfer-skirt/header.png
classes: wide
categories:
  - cosplay 
tags:
  - sewing
  - desmos
  - math
  - hatsune-miku
---

[Miku Expo 2026](https://mikuexpo.com/europe2026/) in Paris is just a few weeks away <span class="small-text">(OMGOMGOMGOMGOMG)</span> so it was time I actually started working on my **Miku cosplay** ! 

If you are only interested in the patterns, [click here to skip ahead](#making-the-skirts).  

## Why Insomnia C(o)urse Miku ?

It took me a really long time to settle on a specific design that I **liked**, that I would feel **comfortable** wearing, and that **I could do** with my current skills.  

**Snow Miku** has always been some of my favorite designs of hers, and it would be fitting since the show will take place in november. There is a lot of Snow Miku designs and some fit my "skill" and "comfort" constraints :  

<div class="gallery" markdown="1">
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/snowmiku2016.jpg" alt="Snow Miku 2016" caption="Snow Miku 2016" %}
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/snowmiku2021.jpg" alt="Snow Miku 2021" caption="Snow Miku 2021" %}
</div>

Snow Miku 2021 is in my TODO list since it's probably my favorite Snow Miku design of all time, but it will probably wait until I have the skills to really do it justice.  
{: .notice--info }

One design I almost settled on was this one [by ch-eescake](https://www.tumblr.com/ch-eesecake/795076279677452288/study-of-edmund-blair-leightons-the-accolade-lol) (left) based on [The Accolade by Edmund Blair Leighton](https://commons.wikimedia.org/wiki/File:Accolade_by_Edmund_Blair_Leighton.jpg) (right) :  

<div class="gallery" markdown="1">
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/cheesecake_accolade.jpg" alt="Cheescake's rendition of Edmund Blair Leighton's Accolade painting" %}

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/edmund_blair_leighton_accolade.jpg" alt="Edmund Blair Leighton's Accolade painting" %}
</div>

But something that bothered me with my last cosplay (Marcille from Dungeon Meshi) is that you don't get to wear them a lot outside of Cons. Thus, one goal I added for my Miku Expo cosplay is to be able to put as many parts of it in my day to day wardrobe (which is for transgender related reasons in a dire need of being reworked).  

Camellia has always been an artist I really liked, and earlier this year he released a new vocaloid album, **Insomnia C(o)urse**, with a cover art by [Temmie](https://tuyoki.itch.io/) (that you might know for Undertale/Deltarune or even the Dweller's Empty Path).  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/insomnia_course_miku.jpg" alt="Insomnia Course Miku" %}

I ended up selecting this one because it's a design I **really** like, for an album I listen quite frequently, not that far away from clothes I would actually wear, and from the start I had a rough idea of how to sew most of it.  

## Drafting the Patterns

On Temmie's artwork, we can spot that there is at least 2 skirts (or a dress and a skirt over it). One circle skirt going straight down, and one slanted.  

### The Circle Skirt

A circle skirt is one of the easiest pattern to design. It's a circular arc with a hole in the center. And when worn, it falls down into a cone shape with more or less **waviness** to it depending on the central angle and the fabric properties.  

<div class="gallery" markdown="1">
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/quarter_circle.png" alt="" caption="Quarter circle skirt pattern" %}
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/half_circle.png" alt="" caption="Half circle skirt pattern" %}
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/full_circle.png" alt="" caption="Full circle skirt pattern" %}
</div>

Drafting this pattern is simply drawing a big circle, with a hole whose **perimeter** is equal to your **waist measurements** `hole_radius = waist / (2 * pi)`. And the length from the big circle edge to the inner hole should be your desired skirt length (so `big_circle_radius = hole_radius + desired_skirt_length`).  

### The Hohmann Transfer Skirt

There is **a lot** of patterns online for asymmetrical skirts, but most of them are either a circle skirt with the hole offset to one side, or a with an extra blob added on the side. 

To be fair, there might be exactly the pattern I just made, but it was faster to do it than to find it.  
{: .notice--info }

We have the first skirt making a cone shape, and now we need some kind of other skirt, which would fit the same **cone** but without everything below a slanted **plane**. A plane that starts at a desired height on one side (let's call it the short side), and ends at the same length as the circle skirt (on the opposite long side).  

When you look at how we would construct a pattern that fits this description, we find that it should be an **ellipse** with one **focus point** being the **center of the hole** to fit your body (because it's a cone plane intersection), and **tangent** to 2 **circles** : one with radius equal to the length of a circle skirt as long as the short side, and another one equal to the length of a circle skirt as long as the long side.  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/hohmann_transfer_pattern.png" alt="Schema of the described pattern" caption="In dashed purple lines, the 2 tangent circles, and in black, the hole" %}

We could call this skirt an elliptical skirt. But if you look for elliptical skirt online, you'll find that it refers to the **"Elliptical hoop skirt"**. A large hoop skirt from the 1850s that is elliptical seen from above instead of round. For example, you might find [this blog post](http://www.vintagevictorian.com/costume_1860.html) with more details about it, or even the [Wikipedia page about Hoop Skirts](https://en.wikipedia.org/wiki/Hoop_skirt).  

This is where the **unnecessary space related stuff** comes in. This ellipse pattern shape fits exactly the constraints of a **Hohmann Transfer Orbit** ! It's the most efficient way to transit between 2 orbits that do not intersect, in 2 impulses.  

"In 2 impulses" is important because in some cases, you can make a 3 impulses transfer more efficient by overshooting the target orbit on your first burn. If you are interested, it's called a **"Bi-elliptic transfer"**.
{: .notice--info }

Below you can see a Hohmann Transfer manoeuver in **Kerbal Space Program**. The blue ellipse is the transfer orbit between the 2 circular orbits. You first burn fuel to get from the small orbit to the elliptical one, then you wait, and once you reach the highest point of the ellipse you burn again to turn your transfer orbit into a circular one.  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/hohmann_transfer_ksp.png" alt="Screenshot of a hohmann transfer manoeuver in Kerbal Space Program" %}

This post would have been posted before [my previous Veer Devblog](/_posts/2026-11-04-fusion-like-cross-platform-code.md) if I hadn't launched KSP to take screenshots instead of using the picture from wikipedia like everyone >:3. I love this game sooooo much! 
{: .notice--info }

In this analogy, the transfer orbit is the asymetrical skirt, and the target orbit is our big circle skirt. Space stuff is really fitting for a character like Miku (like Snow Miku, there is quite a lot of **Space Miku**). And she actually went to space through drawings and messages on JAXA's Akatsuki probe, among which we can find this pure gem :  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/pioneer_miku.jpg" alt="Screenshot of a hohmann transfer manoeuver in Kerbal Space Program" caption="The Miku version of the Pioneer Plaque" %}

## Making the skirts

This post is about the **shape** of the pattern, and I am NOT in any way a skilled seamstress, so there **won't be a lot of details** about every steps required to turn the above patterns into a finished garment (like details of how to make a zipper, how to hem the skirt, what type of stich should be used, etc). These details can be found on other blogs / Youtube / TikTok, without too much problem.  

But using Desmos for pattern drafting might not be the most widespread technique so if you want to use my graph to make your own skirts I will at least explain how it works.  

[Here is the link to the Desmos graph where I drafted the patterns.](https://www.desmos.com/calculator/rptnqprhuf)

### Input your measurements

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/desmos_params.png" alt="" %}

First, at the top of the graph you set **v_R** as the **length of the desired skirt**, and **v_waist** as **your waist measurements**. I used centimeters but you can use whatever you want as long as you use the **same units everywhere**.  

There is an orange ruler in the graph which can be useful to transfer the pattern on fabric.  
{: .notice--info }

### Transferring the Circle Skirt

Everything **black** in the graph is related to the **circle skirt**. Using Desmos is useful to visualize it, but in the end the easiest method is to use a long ruler and draw directly on your fabric.  

You will most likely need to **split the skirt in parts**, since a full or half circle for a long skirt will waste **a lot** of fabric.  
For my half circle skirt I used 2 quarter circles that I joined, which left a seam on each side in which I added pockets.  

### Transferring the Hohmann Transfer Skirt

Everything **purple** in the graph is related to the **elliptical skirt**. An ellipse is symmetrical along both it's axis so you only need to **transfer a quarter of the ellipse**.  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/desmos_params_2.png" alt="" %}

The `v_ratio` slider is to control the length of the short side : 0 and the short side is tangent to the hole (which is bad because you need a bit of fabric here), 1 and you have a circle skirt.  

You just have to transfer some points along the edge of the ellipse, and then you link them trying to be as smooth as possible (which can be a pain on some fabric).  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/ellipse_points.png" %}

Coordinates displayed are relative to the **middle left point**, so that when you fold your fabric along the major axis, the bottom left point is (0,0).  

When transferred on fabric, it might look like this (here the fabric is already folded along the major axis) :  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/pattern_transfer_0.jpg"  %}

Then you can fold along the major and minor axis to cut along the curve :  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/pattern_transfer_1.jpg"  %}

Finally, you can unfold once and transfer the hole measurements and cut while it is still folded along the major axis. To get the hole position you can use the ruler or know that it will be, by definition, at `v_ratio * v_R + v_r` from the left side.  

{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/pattern_transfer_2.jpg"  %}

### Draw the rest of the f\*\*\*ing howl

Now that your fabric is cut, you can finish the skirt by adding a lot of **time consuming details**. 

For the circle skirt, I added **pockets**, a **waist band** with an elastic, and did a **rolled hem** along the bottom edge.  

The bottom hem of a circle skirt is a real test of patience. Especially if you are tall and don't have a rolled hem presser foot.  
{: .notice--warning }

And for the elliptical orbit skirt, I decided to add **ruffles** and **lace** along the bottom edge to give a **more wavy look**. 

The perimeter of the ellipse is shorter than the circle skirt, BUT ruffles require that you multiply that length. So if you thought that the circle skirt rolled hem was painfully long, adding ruffles will be at least twice the pain.  
{: .notice--error }

## The results

Some pictures of the finished products !!!

<div class="gallery" markdown="1">
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/results_both_skirts.jpg" caption="Both skirts"  %}
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/results_circle_skirt.jpg" caption="Only the circle skirt" %}
{% include image.html path="/assets/images/posts/hohmann-transfer-skirt/results_pin.jpg" caption="The pin used to hold the asymmetrical skirt" %}
</div>

For the flower pin, I followed a tutorial on how to make a rose in fabric I found on TikTok. There are loads out there, and most of them follow the same idea. Create a fabric tube (but add an offset when sewing along the tube to create a bit of volume). Then gather it like ruffles, and roll it into a rose.  

I might add some inner structure, like a shallow petticoat, but as far as the skirts go, I'm quite happy how it turned out :3

The orbit skirt ended up being a bit longer because of the added ruffles but I really like the look of it (and the petticoat will most likely fix the length issue). I can also tie it higher on the short side with the pin but I'll need to work a bit more on it because it's a bit finicky at the moment.  
