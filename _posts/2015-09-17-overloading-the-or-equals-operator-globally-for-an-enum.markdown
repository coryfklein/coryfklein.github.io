---
layout: post
title:  "Overloading the 'or equals' operator globally for an enum"
date:   2015-09-17 13:18:53
categories: c++
---

Here is how to overload the `|=` operator globally for an enum in C++.

{% highlight c++ %}
class MyClass {
    public:
    enum Property {
        NoProperty = 0, Flat = 1, Orange = 2, Soft = 4
    };
    //...//
};

inline MyClass::Property&
operator|=(MyClass::Property& a, MyClass::Property const b) {
    a = static_cast<MyClass::Property>(static_cast<int>(a)
                                     | static_cast<int>(b));
    return a;
}
{% endhighlight %}

