---
layout: post
title: "Won't extend it more than once!"
date: 2022-7-6
category: dev
tags: [cpp, lifetimemanagement, lifetimeextension, bugstories]
excerpt_separator: <!--more-->
---
A few months ago I changed teams and I started to work on a library that helps its users to perform cryptographic operations. Those operations need a so-called [Hardware Security Module (HSM)](https://en.wikipedia.org/wiki/Hardware_security_module) that is provided by a third party. My first project was to migrate from one provider to another.
<!--more-->

Though we decided to make the changes without breaking the API, the configuration files had to change. All the client applications have to take the new library version and change the config files. Taking a new version is always a pain as it requires redeploying their applications. Therefore one of the requirements was to deliver a bug-free version on a short notice so that they have to deploy only once.

And we started to work.

And we worked and worked.

And shipped on time.

The next Monday our first adopters loaded their software with the new version of our library.

In a few minutes, they reported a regression.

That was fast. Faster than I expected. I was not particularly confident with the change anyway. Our QA went on vacation during the last few weeks, I lacked the functional expertise and we had to change a lot of code.

Still, the error report came in faster than expected.

It had some particularities though.

Only one of the adopters experienced it even though both of them used the same version and pretty much the same configuration file.

And the error only happened on one of the servers...

## Some disturbance in the force

Having an error not happening everywhere is already bad enough, but there was more to that!

The first error code was about a bad input and that seemed interesting, something to consider. Sadly, later on, we got a myriad of different poorly documented error codes which made little sense.

This situation seriously raised the question of whether the problem is coming from our update or from the 3rd party service?

Falling back our library to the previous version didn't solve the issues, but we had to also restart the 3rd party server. Our manager was convinced that the error is due to our update, but more and more we analyzed the logs and read our changeset over and over again (\~1000 lines of code), and we were less and less convinced.

After the fallback, we ran all our integration tests over and over again. While they were failing before the server reboot both with the old and the new version, now they were succeeding again.

## Don't believe in coincidences!

In the meanwhile, we blacklisted this new version so no matter how much we wanted to retest it with a client application, we couldn't. We decided to fix some long-known issues to get a new version delivered.

I kept thinking.

My manager could be right. I used to say both at work and outside that I don't believe in coincidences. Why should I believe in coincidences in this case? Only because I cannot find a bug? Only because most probably I introduced it?

Those are not good reasons.

But it's also true that I investigated a lot.

Well, a lot, but apparently not enough. I even used gdb, something I rarely do. Now I used it more than ever. Still, it didn't help reveal the issue.

I always wanted to get more familiar with clang and the related tools. I decided this was the right time. I had no idea how to run them in our corporate environment, so I installed them locally and simplified our critical path into something like this piece of code ([coliru link](http://coliru.stacked-crooked.com/a/bcd73f6aa62b4703)):

```cpp
#include <iostream>
#include <string>
#include <boost/variant.hpp>

struct VariantA {
    std::string url;
    std::string port;
    std::string token;
};

struct VariantB {
    std::string username;
    std::string password;
};

class Parameters {
public:
    Parameters(VariantA a) : params(a) {}
    Parameters(VariantB b) : params(b) {}
    boost::variant<VariantA, VariantB> get() const {return params;}
private:
    boost::variant<VariantA, VariantB> params;
};

Parameters makeParams(VariantA a) {
    return {a};
}

void print(unsigned char* p) {
    std::cout << p << '\n';
}

void foo(const Parameters& p) {
     const auto& va = boost::get<VariantA>(
      p.get()
    );
     print((unsigned char*)va.url.c_str());
     print((unsigned char*)va.port.c_str());
     print((unsigned char*)va.token.c_str());
}

int main() {
    VariantA a;
    a.url = "url";
    a.port = "port";
    a.token = "token";
    
    auto p = makeParams(a);
    
    foo(p);
}
```

I ran the address, the memory and the undefined behaviour sanitizers. I expected something from the last one, but I got an error from the first one, from the address sanitizer.

***ERROR: stack-use-after-scope***

No freaking way...

I already looked at `const auto& va = boost::get<VariantA>(p.get());` and I was thinking that while it would be probably worth it to remove the reference which I shouldn't have added in the first place, still, the lifetime of the returned variable from `Parameters::get()` must have been extended. So I decided to do it later once we fixed the error.

And then it seemed that THAT was the error...

## The 5 stages of grief

In the next half an hour I went through [the 5 stages of grief](https://grief.com/the-five-stages-of-grief/). Yes, luckily it was quite fast. Mine looked like this.

- ***Denial***: Okay, okay. It's not sane to have the reference there. But the real issue must be somewhere else. The lifetime of a temporary is extended until that `const&` is used. In any case, even the ASAN said it might be a false positive. But if I made some very tiny changes to the code, such as declaring `va` just a `const auto` instead of `const auto&` or returning in `Parameters::get` a `const&` instead of a `const`, the ASAN report became clean. I arrived at the next stage.
- ***Anger***: stupid me, this line was already suspicious! But I didn't want to fix it so that we can simply test the real fix of the real issue. Aaaaaah!
- ***Bargaining***: At this stage, I was asking myself the question, what if I wasn't in a hurry and if I paid more attention to that update, to that piece of code. This path was still related to the old service provider and I only introduced some technical changes as our architecture changed a bit... I should have paid more attention... To the hell with that! Others should have also paid more attention to the code reviews, how could that pass!
- ***Depression***: My bad feelings went away quite fast, especially towards the others. It was replaced with depression. Fine. I made a mistake. It doesn't work. But I still have absolutely no idea, why it doesn't work. It should work. This is impossible...
- ***Acceptance***: Okay, okay. So it's really that line, it must be about lifetime extension. I simply remove the `&` and say some bullshit that most people will accept, or I take some extra time and try to understand it. This whole bug is just a freaking bug if I don't understand it. If I do, then it was an opportunity to get better.

## Then it hit me!

First I read about lifetime extension [here, in this article](https://quuxplusone.github.io/blog/2020/03/04/field-report-on-lifetime-extension/). I shared it a few times and revisited it a few times. But in the recent days, I read about it somewhere else too. I cannot recall where. Maybe it was just a tweet. It said something like that lifetime extension will only happen once. It cannot be done twice.

I looked up what [C++ Reference says about reference initialization](https://en.cppreference.com/w/cpp/language/reference_initialization#Lifetime_of_a_temporary)

> *"In general, the lifetime of a temporary cannot be further extended by "passing it on": a second reference, initialized from the reference variable or data member to which the temporary was bound, does not affect its lifetime.*

But why would it happen twice here?

Cannot I pass that `c_str` to the next call? Removing the call didn't clean up the ASAN report.

Then it hit me.

```cpp
const auto& va = 
    boost::get<VariantA>( // no second extension...
      p.get() // first extension
    );
```
The first call is to `Parameters::get`. It returns a temporary and its lifetime is extended. Then comes `boost::get<VariantA>`. It takes this temporary whose lifetime was already extended, but it won't be extended for the second call. By the time the full expression is executed, the reference will be destroyed. 

In fact, if I used clang as a compiler and the standard C++17, and therefore `std::variant` instead of the boost option, I could have also used `-Wdangling-gsl`. The compiler would have told me that there is an error in my code!

So that's another reason, why to compile with multiple compilers and why to use an as recent version of C++ as possible.

## Conclusion

In my first project in my new team, I introduced a subtle bug related to lifetime extension. Once there, it's hard to notice and it can manifest itself in unexpected circumstances.

I heartily recommend to run builds with multiple compilers, tons of warnings turned on and also don't forget about the different analyzers and sanitizers,

They might require a bit of time, but they can save you so much. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
