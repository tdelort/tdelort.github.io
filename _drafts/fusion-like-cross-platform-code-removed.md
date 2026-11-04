---
title: "Veer Devblog : Ubisoft's Fusion like Cross-Platform Code (Removed parts)"
slug: fusion-like-cross-platform-code-removed 
excerpt: "A technique borrowed from Ubisoft's Fusion HAL to write cross platform code"
classes: wide
categories:
  - veer-devblog
tags:
  - veer
  - c++
  - dx12
  - vulkan
published: false
---

## Benefits 

Let's compare it to the "platform specific code in class specialization" approach (which is what I used before this).  

First of all, for any class, even simple ones like the above swap_chain, **you will need a vtable**. If you can avoid having one, you should, and with the header injection thing, you don't need a vtable.  

You also don't need to change the hardware agnostic code API and class hierarchy to be able to specialize certain classes if you don't specialize them, *hehe smurt*.  

Finally, even if you chose the PImpl idiom to do cross platform code instead of specializing every classes, you still save some pointer dereferencing and some allocations by using the Fusion HAL approach.  

### No more 'static_cast'-ing your way through the inheritance tree

But for more complex type hierarchies, things get dirty very **VERY quickly**. Let's say we have a very small engine with the following type hierarchy :  

<div class="gallery" markdown="1">

```cpp
class resource {};

class buffer : public resource {};
class texture : public resource {};
```

{% include image.html path="/assets/plantuml/posts/fusion-like-cross-platform-code/figure_0.png" alt="Class diagram of the code snippet" height="10em" %}

</div>

For each of the leaves, you would implement them by specializing them (`dx12_buffer` and `dx12_texture`), and you would probably use a common `dx12_resource` class to factorize common D3D12 code in there.  

<div class="gallery" markdown="1">

```cpp
class resource {};

class buffer : public resource {};
class texture : public resource {};

class dx12_resource;

class dx12_buffer : public buffer, public dx12_resource {};
class dx12_texture : public texture, public dx12_resource {};
```

{% include image.html path="/assets/plantuml/posts/fusion-like-cross-platform-code/figure_1.png" alt="Class diagram of the code snippet" height="18em" %}

</div>

Now, each time someone gives you a hardware agnostic handle to a `texture` or a `buffer`, you just need to `static_cast` you way to the corresponding `dx12_*` equivalent.  

```cpp
void buffer_I_hardly_even_know_her(const buffer& _buffer)
{
    const dx12_buffer& dx12_buffer_handle = static_cast<const dx12_buffer&>(_buffer);
    // now you can do your dx12 specific things
}
```

And what if you want to implement something that can work on both textures and buffers, and you just need to have access to the `dx12_resource` API to implement it ? You either create a overload for each types and `static_cast` your way up (which is not ideal), or you add a way to get a pointer to the `dx12_resource` from a `resource` (which is even less ideal).  

```cpp
void fun_with_resources(const texture& _texture)
{
    fun_with_resources_impl(static_cast<const dx12_texture&>(_texture));
}

void fun_with_resources(const buffer& _buffer)
{
    fun_with_resources_impl(static_cast<const dx12_buffer&>(_buffer));
}

void fun_with_resources_impl(const dx12_resource& _resource)
{
}

// or

class resource 
{
    virtual void* as_backend() = 0;
};

class dx12_buffer : public dx12_resource, public buffer
{
    void* as_backend() override { return static_cast<dx12_resource*>(this); } 
};

void fun_with_resources(const resource& _resource)
{
    const dx12_resource& dx12_resource_handle = static_cast<const dx12_resource&>(*_resource.get_backend());
}
```

I used both of these schemes while developing Veer, with varying results. But the worse bugs to diagnose were caused by the latter so **PLEASE DON'T USE THIS!!!** (or do, but you can't say I didn't warn you)  
{: .notice--error }

Now imagine when you had other resource types, like `texture_2d` and `texture_3d`, or start implementing the following class hierarchy (which will need to work with a similar tree of `command_queues`) :  

```cpp
class command_buffer {};
class copy_command_buffer : public command_buffer {};
class compute_command_buffer : public copy_command_buffer {};
class graphics_command_buffer : public compute_command_buffer {};
```

There are multiple ways to do it, but using the Fusion HAL approach is much simpler, lighter, and easier to debug.  

### Flexibility
