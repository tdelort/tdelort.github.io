---
title: "Veer Framework"
permalink: /veer/
excerpt: "A HAL with many helpers to create a custom game engine with very few dependencies"
header:
  teaser: /assets/images/portfolio/veer/teaser.png
classes: wide
---

Veer is a framework I am working on that I will use to create a tiny game engine. Its first goal is to abstract the **rendering APIs** (D3D12, Vulkan, and Metal), but it also contains **custom containers**, **glm** like vector type, and will abstract **OS features** (like window, event handling, filesystem, mutex/critical section, etc). The main goal being to have as few dependencies as possible (not even the STL, because why not).  

You can find the code of the framework [here on Github](https://github.com/tdelort/Veer).

The rest of this page are nice techniques I encountered / used during the development of Veer.

## Fusion-like multi-platform code 

**TODO** : turn this part into a blog post and just link it here
{: .notice--info}

One very fun idea borrowed from [Ubisoft 2025 talk at REAC](https://enginearchitecture.org/downloads/REAC_2025_Anvil.pdf), is the way they implement class differently on multiple platforms by using one common `.h` header, different `.cpp` depending on the backend (pretty standard here), but injecting code inside the header to add platform specific class members, in their Fusion HAL.  

Inside command_queue.h, we have our class declaration as usual, but with some (quite ugly for now) `#if defined` clauses to inject some platform specific code  :
{% highlight cpp linenos %}
class command_queue
{
public:
    command_queue(render_device& _device, command_buffer::type _type);

private:
    command_buffer::type m_type;

#if defined(D3D12_RENDER_BACKEND)
#include "backends/dx12/dx12_command_queue.inl"
// #elif defined(VULKAN_RENDER_BACKEND)
// #include "backends/vulkan/vk_command_queue.inl"
// #elif defined(METAL_RENDER_BACKEND)
// #include "backends/metal/mtl_command_queue.inl"
#endif 
};
{% endhighlight %}

dx12_command_queue.inl then contains the API specific part of the header :
{% highlight cpp linenos %}
private:
    ComPtr<ID3D12CommandQueue> m_command_queue_api_handle;
{% endhighlight %}

And finally, in command_queue.cpp, we can implement the class like if we were working only on a D3D12 renderer :   
{% highlight cpp linenos %}
command_queue::command_queue(render_device& _device, command_buffer::type _type)
    : m_type{_type}
{
    D3D12_COMMAND_QUEUE_DESC desc = {};
    desc.Type = command_buffer::s_convert(_type);
    desc.Priority = D3D12_COMMAND_QUEUE_PRIORITY_NORMAL;
    desc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;
    desc.NodeMask = 0;

    HRESULT hr = _device.get_api_handle()->CreateCommandQueue(&desc, IID_PPV_ARGS(&m_command_queue_api_handle));
    VEER_ASSERT(SUCCEEDED(hr), "Failed to create command queue (" << hr << ")");
}
{% endhighlight %}

This might look ugly and complicated for something you can simply do with inheritance (`command_queue` being specialized into `dx12_command_queue`), but there are actually so many advantages.  

First of all, I switched to this when I tried to implement a class hierarchy in a platform agnostic way. For example, something like this :

{% highlight cpp linenos %}
class render_device_resource
{};

class render_device_buffer : public render_device_resource 
{};

class render_device_texture_base : public render_device_resource 
{};

class render_device_texture_2d : public render_device_texture_base 
{};

class render_device_texture_3d : public render_device_texture_base 
{};
{% endhighlight %}

If you need to implement the different backends using inheritance, you either get a very complicated class hierarchy, or some weird patterns to prevent code duplication.

This technique also allows for simpler implementation, since you basically implement the base class without worrying about other backends. The code formatter and syntax highlighting don't have this much trouble with it, and the drawbacks of having to can be alleviated with a bit of macro tricks / build system magic (this is still a TODO).
