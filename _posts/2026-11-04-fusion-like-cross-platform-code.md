---
title: "Veer Devblog : Ubisoft's Fusion like Cross-Platform Code"
slug: fusion-like-cross-platform-code 
excerpt: "A technique borrowed from Ubisoft's Fusion HAL to write cross platform code"
classes: wide
categories:
  - veer-devblog
tags:
  - veer
  - c++
  - dx12
  - vulkan
---

One idea borrowed from [Ubisoft 2025 talk at REAC](https://enginearchitecture.org/downloads/REAC_2025_Anvil.pdf) that I use a lot in Veer is the way they implement classes differently on **multiple platforms** in their *Fusion* Hardware Abstraction Layer. They use one common `.h` header, different `.cpp` depending on the backends (pretty standard here), and they **inject code inside the header** to add platform specific class members.

I am not a big fan of those cooking website where you need to scroll through an 10K+ pages long AO3 fan fiction<sup>\*</sup> on butternut squash before actually being able to read the recipe. So I won't waste time explaining every other approaches I tried before this (but feel free to message me / comment if you are curious).  

*<sup>\*</sup>Sending strength to you if you happen to read this during the AO3 4 weeks maintenance.*

## The recipe  

Say you want to create a swap chain class, with different implementations depending on a value **known at build time** (for example you would like to build for different rendering APIs).  

I'm choosing the swap chain here because it's a simple example, but also because it's one of the few classes I had yet to convert to this approach.  
{: .notice--info }

How I do it in Veer, is with this kind of file hierarchy :  

```txt
.
|-swap_chain.h
|-swap_chain.cpp      <-- I don't use that a lot but you can put common function definitions here
|
|-backends/
   |-vk/
   |  |-swap_chain.cpp
   |  |-swap_chain.inl
   |  
   |-mtl/
   |  |-swap_chain.cpp
   |  |-swap_chain.inl
   |  
   |-dx12/
      |-swap_chain.cpp
      |-swap_chain.inl
```

You have the common API in a `swap_chain.h` header file :  

```cpp
class swap_chain
{
public:
    swap_chain(render_device& _device, veer::display::window::window& _window);

    render_device_texture_2d& get_current_backbuffer();
    size_t get_backbuffer_index();
    void present();

    static constexpr size_t s_swap_chain_buffer_count = 2u;

#include VEER_INL(swap_chain)
};
```

The actual implementation in a `swap_chain.cpp` file for you custom backend (here D3D12). The bodies of the functions are not important for the sake of the post so I only included snippets here and there :  

```cpp
#include <veer/display/render/swap_chain.h>
// Or this :
// #include "../swap_chain.h"

/* ... other includes and methods ... */

swap_chain::swap_chain(render_device& _device, veer::display::window::window& _window)
{
    ComPtr<IDXGIFactory4> factory4 = _device.get_dxgi_factory();

    /* ... */
}

swap_chain::~swap_chain()
{
    /* ... */
}

void swap_chain::present()
{
    /* ... */
}

size_t swap_chain::get_backbuffer_index()
{
    UINT index = m_api_swap_chain_handle->GetCurrentBackBufferIndex();
    return (size_t)index;
}

render_device_texture_2d& swap_chain::get_current_backbuffer()
{
    return *(m_back_buffers_resources[get_backbuffer_index()].get());
}
```

It's easy to swap between `.cpp`, because that's the **build system's job**. Having them in separate folders makes it really easy with **CMake** to simply `add_subdirectory()` or not 

Something we tend to forget is that, as long as we don't break the [**One Definition Rule**](https://en.cppreference.com/cpp/language/definition), we can split our implementation in however many `.cpp`. So we can actually implement common code in a dedicated `.cpp` file.
{: .notice--info }

The `dx12/swap_chain.inl`, containing the content to inject inside the class declaration :  

```cpp
private:
    ComPtr<IDXGISwapChain4> m_api_swap_chain_handle;
    ComPtr<ID3D12DescriptorHeap> m_back_buffers_descritor_heap;

    containers::static_array<unique_ptr<dx12_render_device_backbuffer>, s_swap_chain_buffer_count> m_back_buffers_resources = {};
```

And, to get from the line `#include VEER_INL(swap_chain)` in the header, to the right path, we need a bit of <span class="rainbow-text">macro magic</span> :  

```cpp
// Defined depending on the backend
#define RENDER_BACKEND_DIR dx12/

// ...

#define EXPAND(x) x
#define STR_INTERNAL(x) #x
#define STR(x) STR_INTERNAL(x)

#define VEER_INL(file) STR(EXPAND(backends/)EXPAND(RENDER_BACKEND_DIR)EXPAND(file)EXPAND(.inl))
```

Here, we use the `EXPAND(x)` macro that does nothing to be able to write valid code **without any whitespace between the different parts of the path**. Then we simply stringify the whole thing with the usual `STR` macro. We can't use the `##` operator because the things we are concatenating are not, and do not produce, valid tokens (mainly because of the `/`).  

I know you could have the things I put in `dx12/swap_chain.inl` directly inside the class declaration behind a `#if defined` clause, but that gets messy **really quickly** with more complex classes and the user doesn't want to see your implementation details, *gross*. Hence why I bothered with the semi cursed macro thingy.  

## Notes

This is a pretty useful approach to cross platform code because you **don't need a vtable**. 

You don't have the allocations / pointer dereferencing of the PImpl idiom.  

You can make it work regardless of the complexity of the API. If you ever tried to use the "platform specific code in class specialization" approach on a API that is already a complicated type tree you understand what I mean.  

Other platform specific code can **use your types as is**, without the need to static_cast your way to platform specific types or anything.  

And only the `.inl` **syntax highlighting will suffer a little bit**, the rest work flawlessly.  

I still need to handle specific include files needed by each `.inl`, but most of it are simply the core types from the os/render backend. As of posting this, I still use classic `#if defined` in the header to handle that.  

All in all, I think this approach is pretty nice, suits my personal needs perfectly, and removes all the things I hated about cross platform code, without any effects on performance.  
