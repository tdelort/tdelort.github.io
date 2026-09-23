---
title: "Veer Devblog : Starting Point"
excerpt: "A summary of what Veer is, and what it will aim to be"
categories:
  - devblog
tags:
  - veer
  - c++
  - dx12
  - vulkan
---

First (and hopefully not last) Veer devblog !  

Veer is a framework I'm building that I will use to create a tiny game engine. It's called Veer because I know I will change my mind about what I want to do with it at least a gazillion times before it's actually in usable shape, so one core idea is that it should abstract hardware without adding too much constraints for the user.  

## The Direction

Obviously it will never be finished since there is always new features that needs to be exposed, or new needs being raised. So this section is **not about a end goal but a general direction**.  

Everything I do, I do it because it's what I love. Thus I will try to rely on as few dependencies as possible to see how much I can do with only C++, rendering APIs, and the OS.  

As of writing this, the dependencies I'm working on removing are : D3D12MA, GLFW, and as much C++ std library headers as possible.
{: .notice--info }

Another consequence of this (and of course my ethics, ecological and political ideas, and my uttermost respect for any form of art), is that **none of it will ever be made using Generative AI**. All of the code (even the dubious bits) will be 100% made by yours truly.  

Inspired by the comfort I felt when reading [this demofox blog](https://blog.demofox.org/2025/04/05/simulating-dice-rolls-with-coin-flips/) (you never want someone you look up to to be a piece of turd, but he is the GOAT), and because now more than ever we should raise our voices, this blog will always be **proudly woke** and publicly **political**.  
{: .notice--danger }

Finally, one larger goal will be to be as efficient as possible, **Watts/Pixels** being a metric I will try to keep as low as possible.  

## What's already done

Before starting devblogging about it, I had already started working a bit on it. Here is a short list of what is already done.

### math::vec<T,N>  

One of the first feature I ever did was a vector of N arithmetic values class. Something that looks **glm**, but without the very verbose template specialization. SIMD operations are not yet implemented and will only be done when needed. Here is a short example of what it looks like and what it can do :

```cpp
// we have common vec<T,N> values aliases
using vec2u = vec<uint32_t, 2u>;

vec2u current_window_size = window.get_size();

// ...

// Here, the binary operation actually returns a vec<bool, 2u> that is fed to the any(...) function checking if any values are true
if (any(current_window_size != window.get_size()))
{
    // do something
}

// ...

const vec3f points[3] = { 
  //   1    
  //  / \   
  // 0---2 
};

const vec3f a = points[1] - points[0];
const vec3f b = points[2] - points[0];
const vec3f n = normalize(cross(b, a));
const vec2f packed = math::vec2f(n.x(), n.y());
```

The goal is to have something very close vectors you usually have in shading languages. On the last line, one might expect `n.xy()`, but swizzle is not yet implemented. This is quite verbose to do if not actually done via the compiler or by generating some code, and since it's only to save a few keystrokes it's very low in priority.  

There is also quaternion support (basically a vec4f with added functions), and the support for matrices is on the way.  

### Containers  

The philosophy is fewer is better, because from experience most containers end up being a basic one with a different set of operations.  

Here is the list of the containers implemented so far :  
 - `static_array<T,N>` which is basically a `T[N]`,  
 - `resizable_array<T>`, my take on a `std::vector<T>`,  
 - `string<CHAR_TYPE>`.  

The `string` type was for some time a simple passthrough to a `resizable_array`. But I needed to take care of the null terminator, and since there is some smart things we can do if we know that elements can only be char types, it made sense for it to have its own type.   

I still need to implement a `set<T>` type which can also be used to make a map (`class map<T,U> : public set<pair<T,U>>`).  

And with this, we are done. There might be some punctual needs for types (like linked lists) but I'll work with the ones listed here as much as possible and add new ones only if they are really vital.  

For the linked list example (but the same applies for the sets), I could simply write one based on `resizable_array` as a node freelist which has the added advantage of being cache friendly. Having the memory addresses stable might one day be interesting but we'll see when we get there.  
{: .notice--info }

Finally, there is a small `span<T>` utility that make API way simpler to implement.  

### Rendering  

Veer is at a stage where we can write the following code :

```cpp
compute_technique& my_custom_technique;
render_device& device;
render_thread& render_thread;
render_device_texture_2d& texture_output;
render_device_buffer& buffer_input;

// ... 

shader_parameter_id parameter_id = my_custom_technique.get_constant_id("my_constant_buffer.parameter");

command_queue& compute_queue = device.get_compute_command_queue();

compute_command_buffer compute_command_buffer(render_thread);
        
{
  // holds temporary draw/submit data and wraps a command buffer
  compute_submit_context submit_ctx(compute_command_buffer, my_custom_technique);

  submit_ctx.set_constant("my_constant_buffer.parameter", 4.2f);
  // or
  submit_ctx.set_constant(parameter_id, 4.2f);

  submit_ctx.set_texture("my_read_write_texture", texture_output);
  submit_ctx.set_buffer("my_read_only_buffer", buffer_input);

  submit_ctx.dispatch(1024u, 1u, 1u);
}

// we can also simply use the command buffer :
compute_command_buffer.clear_texture(m_texture_output, vec4f(0.f, 0.f, 0.f, 0.f));

// finally, yield the command buffer to the command_buffer_queue for execution 
compute_queue.execute(std::move(compute_command_buffer));
```

The idea is simple, the `command_buffer` abstracts the API command list / command buffer concepts, and the submit context is a wrapper around to make things easier.  
The main tasks of the submit context are handling constant buffers, resources transitions, and also make sure the API is easier to use (enforce calls order and check basics things like viewport count == scissors count).  

For now, only the DirectX 12 backend is here, but the Vulkan implementation will soon come. I also plan to do a Metal backend, but even though I heard it's a quite nice API to work with, it's not in my top priority list (maybe when I get bored with some other feature and crave the need to go through a lot of documentation).  

The actual current implementation is as boring to explain as it is to just read the code, so go check on Github for more details if you are interested. New upcoming features will be more interesting, so see you in upcoming devblogs.  

### OS

Nothing major implemented, I still use the `std` to read files from disk and I fully rely on **GLFW** to handle the window and events. The end goal is to get rid of **GLFW**, but I will let it stay a little bit longer because it's frankly so easy that way, and replacing it will take a few liters of coffee.  

### Conclusion

Progress is slow, but this is a really fun project. In a world dominated by commercial game engines, I hope I can (when there will be a little bit more to see) motivate other people to write their own framework/engine from scratch. It's not as hard as it looks, especially if you have a specific scope in mind.  

Be gay, do crime !
