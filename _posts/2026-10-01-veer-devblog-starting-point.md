---
title: "Veer Devblog : Starting Point"
slug: starting-point
excerpt: "A summary of what Veer is, and what it will aim to be"
categories:
  - veer-devblog
tags:
  - veer
  - c++
  - dx12
  - vulkan
---

First Veer devblog ! 

Veer is a framework I'm building that I will use to create a tiny game engine. It's called Veer because I know I will change my mind about what I want to do with it at least a gazillion times before it's actually in usable shape, so one core idea is that it should abstract hardware without adding too much constraints for the user.  

## The Direction

Obviously it will never be finished since there is always new features that needs to be exposed, or new needs being raised. So this section is **not about a end goal but a general direction**.  

Everything I do, I do it because it's what I love. Programming included. Thus I will try to rely on as few dependencies as possible to see how much I can do with only C++, rendering APIs, and the OS.  

The dependencies I'm still working on removing are : D3D12MA, GLFW, and as much C++ std library headers as possible (but not all of them, I don't plan on reimplementing ``char`` to ``wchar_t`` conversion for example).
{: .notice--info }

As another consequence of the fact I do it for fun (but also because of my ethics, my ecological and political convictions, and my respect for any form of art) is that **none of it will ever be made using Generative AI**. All of the code (even the dubious bits) will be 100% made by yours truly.  

Finally, one larger goal will be to be as efficient as possible, **Watts/Pixels** being a metric I will try to keep as low as possible (more details on "how" scheduled for a later post).  

## What's already done  

Before I started writing about it, I had already started working a bit on it. Here is a short list of what is already done.

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

The philosophy is **fewer is better**, because from experience most containers end up being a basic one with a different set of operations.  

Here is the list of the containers implemented so far :  
 - `static_array<T,N>` which is basically a `T[N]`,  
 - `resizable_array<T>`, my take on a `std::vector<T>`,  
 - `string<CHAR_TYPE>`.  

The `string` type was a simple passthrough to a `resizable_array`. But I needed to take care of the **null terminator**, and since there is some smart things we can do if we know that elements **can only be char types**, it made sense for it to have its own type.  

I still need to implement a `set<T>` type which can also be used to make a map (`class map<T,U> : public set<pair<T,U>>`).  

And with this, we are done. There might be some punctual needs for types (like linked lists) but I'll work with the ones listed here as much as possible and add new ones only if they are really vital.  

For the linked list example (but the same applies for the sets), I could simply write one based on `resizable_array` as a node freelist which has the added advantage of being cache friendly. Having the memory addresses stable might one day be interesting but we'll see when we get there.  
{: .notice--info }

Finally, there is a small `span<T>` utility that makes API way simpler to implement.  

### Rendering  

Veer is at a stage where we can write the following code (this is basically something I took from my test app).

We have a few types that are wrappers around rendering API specific types :
```cpp
namespace vdr = veer::display::render;

// Let's say we have all this accessible
vdr::render_device& render_device;
vdr::render_thread& render_thread;
vdr::compute_technique& technique;
// material here means that it is bespoke for this pass 
vdr::constant_buffer& material_constant_buffer; 
// frame here means that it does not change during the frame 
vdr::constant_buffer& frame_constant_buffer;
// This one has been filled with a gaussian noise computed on the CPU (or read from a file)
vdr::render_device_texture_2d& gaussian_noise_tex;
// This is our output
vdr::render_device_texture_2d& spectrum_tex;
```

Here we can retrieve the different constant buffer members info (offset, size) by name :
```cpp
// and we already did that once (we only to do it again if we somehow 
// change the constant buffer layout from the shader)
const vdr::constant_buffer_definition& def = material_constant_buffer.get_def();
vdr::buffer_elem_info patch_length_id = def.get_elem_info("patch_length");
vdr::buffer_elem_info output_tex_id = def.get_elem_info("output_tex");
vdr::buffer_elem_info gaussian_noise_tex_id = def.get_elem_info("gaussian_noise_tex");
```

And then we can start issuing some draw calls. It's basically split in 2. Filling the constant buffers (every parameters goes through a constant buffer) through the ``constant_buffer::update_context`` :
```cpp
// Other queues exists (copy and compute) and can actually be used, but the synchronization code is 
// still  very manual so I won't be using them YET 
vdr::command_queue& graphics_queue = device.get_graphics_command_queue();
vdr::graphics_command_buffer command_buffer(render_thread);

{
    vdr::constant_buffer::update_context update_ctx(material_constant_buffer, render_thread);

    update_ctx.set_constant(patch_length_id, 400.f);
    update_ctx.set_texture_read_only(gaussian_noise_tex_id, gaussian_noise_tex);
    update_ctx.set_texture_read_write(m_output_tex_id, spectrum_render_target);

    // constant buffer gpu data is updated during the update_context dtor here
}
```

And then pushing some commands / states using a ``submit_context``.  

```cpp
{
    vdr::compute_submit_context ctx(command_buffer, technique);

    ctx.set_constant_buffer(frame_constant_buffer, vdr::constant_buffer_type::frame);
    ctx.set_constant_buffer(material_constant_buffer, vdr::constant_buffer_type::material);

    // TODO : const veer::math::vec3u group_size = technique.get_group_size();
    static constexpr veer::math::vec3u s_group_size(8u, 8u, 1u);

    veer::math::vec2u output_size = spectrum_tex.get_desc().m_size;
    const veer::math::vec3u group_count =
        veer::math::ceiled_remainder(veer::math::vec3u(output_size.x(), output_size.y(), 1u), s_group_size);

    ctx.dispatch(group_count);
}

// command_buffer is not executed here, only closed and stored 
graphics_queue.enqueue(std::move(command_buffer));

// Then after multiple calls to enqueue, one can call :
graphics_queue.flush();
// To execute all command buffers staged all at once
```

The `command_buffer` abstracts the API command list / command buffer concepts, and the ``submit_context`` is a wrapper around to make things easier. But don't worry, in case we wan't to do some questionable things, we can still use the `command_buffer` directly.  
The main tasks of the submit context are **handling resources transitions**, make sure the API is **easier to use** (enforce calls order and check basics things like viewport count == scissors count), and hide other common calls.  

For now, only the DirectX 12 backend is here, but the **Vulkan implementation will come soon**. I also plan to do a Metal backend, but even though I heard it's a quite nice API to work with, it's not in my top priority list (maybe when I get bored with some other feature and crave the need to go through a lot of documentation).  

The actual current implementation is as boring to explain as it is to just read the code, so go check on [Github](https://github.com/tdelort/Veer) for more details if you are interested. New upcoming features will hopefully be more interesting :]  

### OS

Nothing major implemented, I still use the `std` to read files from disk and I fully rely on **GLFW** to handle the window and events. The end goal is to get rid of **GLFW**, but I will let it stay a little bit longer because it's frankly so easy that way, and replacing it will take a few liters of coffee.  

### Next post ?  

In the next post, we will see how those classes that abstract platform specific types are implemented in a way that makes it very easy to write, without inheritance (and the problems it can cause), but with some (relatively minor) harm caused to your syntax highlighting.  

Progress is slow, but this is a really fun project. In a world dominated by commercial game engines, I hope I can (when there will be a little bit more to see) motivate other people to write their own framework/engine from scratch. It's not as hard as it looks, especially if you have a specific scope in mind.  

And don't forget to be silly !  
