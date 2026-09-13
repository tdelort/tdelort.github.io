---
title: "School Project : Physics Engine"
permalink: "physics_engine"
excerpt: "Physics Engine developed as part of 8INF935 course at UQAC"
header:
  image: /assets/images/portfolio/school_project_physics_engine/header.png
  teaser: /assets/images/portfolio/school_project_physics_engine/teaser.png
classes: wide
---

Physics Engine developed as part of 8INF935 course at UQAC supporting Rigidbody and simple Particles. Made in C++, OpenGL 3.2, Dear ImGui, and GLFW.  

[Source code available here.](https://github.com/tdelort/8INF935)

## Features

Simple renderer with : 
 - basic primitives, 
 - .obj import, 
 - single point light with no shadows, 
 - and vertex normal stream generation in geometry shader. 

Particle with force and collisions.  

RigidBody with force, torque, and collision detection (using linear Octree).

![Image]({{site.baseurl}}/assets/images/portfolio/school_project_physics_engine/box_collision.gif)  
*2 box colliding*

![Image]({{site.baseurl}}/assets/images/portfolio/school_project_physics_engine/random_impulse.gif)  
*Rigidbody subjected to random upward impulses when its origin falls below the z=0 plane*

![Image]({{site.baseurl}}/assets/images/portfolio/school_project_physics_engine/spring.gif)  
*2 Rigidbody attached to springs (without collisions)*

![Image]({{site.baseurl}}/assets/images/portfolio/school_project_physics_engine/octree.png)  
*Debug view of the octree structure used to reduce collision detection time*