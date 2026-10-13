---
title: "Constexpr conversion tables"
slug: constexpr-conversion-tables
excerpt: "Conversion tables are everywhere in an Abstraction Layer. But how would you make them fast and future proof ?"
tags:
  - c++
  - veer
---

If you've ever written an abstraction layer, you surely had to write something like this at one point : 

```cpp
struct blend_state
{
    enum class operation
    {
        add = 0,
        subtract,
        reverse_subtract,
        min,
        max,

        COUNT
    };
};

// Then, in another file :

D3D12_BLEND_OP s_convert(blend_state::operation _blend_operation)
{
    static constexpr D3D12_BLEND_OP s_conversion_table[] = {
        D3D12_BLEND_OP_ADD,          // add
        D3D12_BLEND_OP_SUBTRACT,     // subtract
        D3D12_BLEND_OP_REV_SUBTRACT, // reverse_subtract
        D3D12_BLEND_OP_MIN,          // min
        D3D12_BLEND_OP_MAX,          // max
    };
    static_assert(VEER_STATIC_ARRAY_SIZE(s_conversion_table) == static_cast<size_t>(blend_state::operation::COUNT));

    return s_conversion_table[static_cast<size_t>(_blend_operation)];
}
```

And hoped that any of the 2 **enums did not change** too much later on since the only check you did was on the number of elements.  
Or maybe you opted for the following : 

```cpp

D3D12_BLEND_OP s_convert(blend_state::operation _blend_operation)
{
    static constexpr std::pair<D3D12_BLEND_OP, blend_state::operation> s_conversion_table[] = {
        { D3D12_BLEND_OP_ADD,          blend_state::operation::add },
        { D3D12_BLEND_OP_SUBTRACT,     blend_state::operation::subtract },
        { D3D12_BLEND_OP_REV_SUBTRACT, blend_state::operation::reverse_subtract },
        { D3D12_BLEND_OP_MIN,          blend_state::operation::min },
        { D3D12_BLEND_OP_MAX,          blend_state::operation::max }
    };
    static_assert(VEER_STATIC_ARRAY_SIZE(s_conversion_table) == static_cast<size_t>(blend_state::operation::COUNT));

    for(const std::pair<D3D12_BLEND_OP, blend_state::operation>& mapping : s_conversion_table)
    {
        if (mapping.second == _blend_operation);
          return mapping.first;
    }

    // Error handling down here
}
```

But having to put **a loop** for that kind of things makes me sad :c  

You could also just write down a full **switch statement** which, works as intended since it will be optimized down to what is basically indexing into an array. But this is quite verbose and not very funny to do (which to be fair is more important than any other metric).  
{: .notice--info }

## The solution  

This is where I come in and ask you if you know about our lord and saviour **variadic templates**.  

Let's start with a very similar thing to what we are trying to achieve. **A constexpr table of values**. 

We start by introducing a simple wrapper around a basic static array (basically `std::array`) :

```cpp
template <typename T, size_t N>
struct static_array
{
    T m_data[N];

    constexpr const T& operator[](size_t _i) const { return m_data[_i]; }
    constexpr T& operator[](size_t _i) { return m_data[_i]; }
};
```

Then, we create a utility to **store** that array and **get** values from it :  

```cpp
template <typename TABLE_BUILDER>
    requires requires {
        { TABLE_BUILDER::build() } -> std::same_as<typename TABLE_BUILDER::table_type>;
    }
struct constexpr_table
{
    using builder_type = TABLE_BUILDER;
    static constexpr size_t size = builder_type::table_size;

    template <size_t INDEX>
    static consteval const builder_type::value_type& get()
    {
        return m_data[INDEX];
    }

    static constexpr const builder_type::value_type& get(const size_t& _index)
    {
        return m_data[_index];
    }

    static constexpr builder_type::table_type m_data = builder_type::build();
};
```

And finally, we create a type that satisfies the `TABLE_BUILDER` constraints and that will fill the static array :  

```cpp
template <typename T, T... VALUES>
struct table_from_pack
{
    static constexpr size_t table_size = sizeof...(VALUES);
    using value_type = T;
    using table_type = containers::static_array<T, table_size>;

    static consteval table_type build()
    {
        table_type data{};
        size_t i = 0ull;
        ((data[i++] = VALUES), ...);
        return data;
    }
};

template <typename T, T... VALUES>
struct constexpr_pack_table : public constexpr_table<table_from_pack<T, VALUES...>>
{
};
```

I remember the first time I saw a co-worker using **pack expansion**, I thought they were writing some pseudo code before actually implementing the real thing ^^. But I urge you to go look at the [documentation for it](https://en.cppreference.com/cpp/language/pack#Pack_expansion), it is a really powerful tool.
Here, `((data[i++] = VALUES), ...);` simply expands to `(data[i++] = value_0), (data[i++] = value_1), ...` with `value_0` being the first value in the `VALUES` pack, `value_1` the second, etc. This is the **recursion-free** solution to unpack the parameter pack.
{: .notice--info }

For completeness sake, a short example of how one would use the `constexpr_pack_table` :  

```cpp
using pi_decimals_table_t = constexpr_pack_table<int, 3, 1, 4, 1, 5, 6, 5>;

constexpr int pi_0 = pi_decimals_table_t::get<0u>();
const int pi_5 = pi_decimals_table_t::get<5u>();
const int pi_6 = pi_decimals_table_t::get(6u);
```

With this, we can store multiple **literal values** (values that can be made `constexpr`) in an table **built at compilation time**. Use cases I can think of on the top of my head would be things like storing specific sequences (Halton sequences, permutations) or data that can be generated (for example, some hashing algorithms depend on an internal lookup table).  

The `constexpr_table` is just a wrapper to a `static constexpr` array, with a way to fill said array as template argument.  

And `constexpr_pack_table` is a specialization that fills the array using all values provided as template arguments (after the underlying type which we could deduce but it makes it explicit that all values types should match that way).  

Separating the storage and the building parts is really important for the next step, which is to turn this simple table into a conversion table.  

We first introduce a new table builder, which from a pack of `std::pair<FROM, TO> ...VALUES` stores each `VALUES.second` at the index `static_cast<size_t>(VALUES.first)` :  

```cpp
template <typename FROM, typename TO, std::pair<FROM, TO>... VALUES>
    requires is_contiguous_v<FROM, VALUES.first...>
struct conversion_table_builder
{
    static constexpr size_t table_size = static_cast<size_t>(get_max_v<FROM, VALUES.first...>) + 1u;
    using value_type = TO;
    using table_type = containers::static_array<value_type, table_size>;

    static consteval table_type build()
    {
        table_type data{};
        size_t i = 0ull;
        ((data[static_cast<size_t>(VALUES.first)] = VALUES.second), ...);
        return data;
    }
};
```

And then, we can specialize our `constexpr_table` and add a few getters to hide the `static_cast<size_t>()` calls :   

```cpp
template <typename FROM, typename TO, std::pair<FROM, TO>... VALUES>
struct constexpr_conversion_table : public constexpr_table<conversion_table_builder<FROM, TO, VALUES...>>
{
    using base_type = constexpr_table<conversion_table_builder<FROM, TO, VALUES...>>;

    template <FROM FROM_VALUE>
    static consteval const TO& get()
    {
        return base_type::template get<static_cast<size_t>(FROM_VALUE)>();
    }

    static constexpr const TO& get(const FROM& _from)
    {
        return base_type::get(static_cast<size_t>(_from));
    }
};
```

Which, applied to our starting problem would look like this : 

```cpp
D3D12_BLEND_OP s_convert(blend_state::operation _blend_operation)
{
    using conversion_table_t = constexpr_conversion_table<
        blend_state::operation, D3D12_BLEND_OP,
        {blend_state::operation::add,               D3D12_BLEND_OP_ADD},
        {blend_state::operation::subtract,          D3D12_BLEND_OP_SUBTRACT},
        {blend_state::operation::reverse_subtract,  D3D12_BLEND_OP_REV_SUBTRACT},
        {blend_state::operation::min,               D3D12_BLEND_OP_MIN}, 
        {blend_state::operation::max,               D3D12_BLEND_OP_MAX}>;

    static_assert(conversion_table_t::size == static_cast<size_t>(blend_state::operation::COUNT), "");
    return conversion_table_t::get(_blend_operation);
}
```

We end up with the same array as in the first code snippet with the **same performance**, but we rely a less on the enum underlying values. Now, the **table gets updated** if any **enum values gets changed or swapped**. And we can still rely on the usual `static_assert` on size **to check if a value was added**.  

## Sanity checks  

This construct has one really strong constraint (which the first code snippet also had). The conversion is done by casting values of the `FROM` type to `size_t` to index into an array. Thus it will **work best** if the `FROM` values are **contiguous**.  

Actually, the constraint is not that hard, and the values could be very sparse and it would still work, but the table would be mostly empty, which would be sad.  
{: .notice--info }

So in the `conversion_table_builder`, I added the `is_contiguous_v` constraint on the `VALUES.first...` pack :  

```cpp
template <typename FROM, typename TO, std::pair<FROM, TO>... VALUES>
    requires is_contiguous_v<FROM, VALUES.first...> // HERE
struct conversion_table_builder 
{
    /* ... */
}
```

Which is implemented like so :  

```cpp
template <typename T, T... VALUES>
struct is_contiguous
{
    static consteval bool compute()
    {
        constexpr size_t array_size = static_cast<size_t>(get_max_v<T, VALUES...>) + 1u;
        containers::static_array<bool, array_size> array{};
        for (size_t i = 0; i < array.size(); ++i)
            array[i] = false;
        
        // Array filled with "false" every where

        ((array[static_cast<size_t>(VALUES)] = true), ...);

        // For each value, fill the corresponding slot in the array with "true"

        bool contiguous = true;
        for (size_t i = 0; i < array.size(); ++i)
            contiguous = contiguous && array[i];

        // IF there is a slot not filled with "true", there was a gap and the values are not contiguous

        return contiguous;
    }

    static constexpr bool value = compute();
};

template <typename T, T... VALUES>
constexpr bool is_contiguous_v = is_contiguous<T, VALUES...>::value;
```

In the end, this is not a big problem since most of the time I will be using this utility to convert from one of my custom enums. In cases that don't fit the requirements, I can still use one of the aforementioned methods (array of pairs and for loop or a switch case).  

## Multi max

I've detailed most constructs, I might as well explain the `get_max_v` to get the maximum value from a pack of values since it's used at least twice in the code snippets give here.  

It is based on my `math::max(...)` function that also works on a variadic number of parameters :  

```cpp
// Let's start with the simple max function

template <typename T>
constexpr const T& max(const T& _lhs, const T& _rhs)
{
    return _lhs < _rhs ? _rhs : _lhs;
}

// and then, using a fold expression we can extend the binary max to any number of arguments.  

template <typename T, typename... ARGS>
constexpr T max(const T& _arg0, const ARGS&... _args)
    requires (sizeof...(ARGS) > 1)
{
    T accum = _arg0;
    ((accum = math::max<T>(_args, accum)), ...);
    return accum;
}

// We can obviously do this for other binary functions like min
```

It performs a **copy**, which is not as pretty as returning the right reference, and it ends up doing `NumArguments - 1` max invocation instead of `log2(NumArguments)`, but it's still very useful and for **simple arithmetic types** it's not a horrible cost to pay (+ I have yet to use it for more than 3 values ^^) :  

```cpp
int a, b, c, d;
const int max_value = max(max(a,b), max(c, d));
const int max_value = max(a, b, c, d);
```

And I simply added the very straight forward `type_traits` syntax :  

```cpp
template <typename T, T... VALUES>
struct get_max
{
    static constexpr T value = math::max<T>(VALUES...);
};

template <typename T, T... VALUES>
constexpr T get_max_v = get_max<T, VALUES...>::value;
```
