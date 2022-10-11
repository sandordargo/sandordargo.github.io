---
layout: post
title: "Should you use static code analyzers as gatekeepers?"
date: 2022-10-12
category: dev
tags: [cicd, quality, automation, devops]
excerpt_separator: <!--more-->
---
With the [Code Insights plugin of Bitbucket](https://support.atlassian.com/bitbucket-cloud/docs/code-insights/), we have the possibility to make static code analyzers (from on *SCA*s) a gatekeeper for pull requests. Meaning that if a code analyzer reports certain kinds of issues, it can block pull requests from getting merged.

While first, it sounds like a helpful idea, let's see whether it's realistic and whether it helps improve quality or prevents improvements.

## The motivation behind

[High-quality code is still not the norm among coders](https://www.sandordargo.com/blog/2019/08/28/why-clean-code-is-not-norm) and it's especially not recognized among stakeholders as a necessity for delivering durable software. Even though defining quality, in particular software quality, is a difficult task and many disagree on whether clean code should be considered part of software quality, I strongly believe it should. [The definition of CISQ](https://www.it-cisq.org/standards/) enumerates 4 elements that are part of structural software quality:

*   security
*   reliability
*   maintainability
*   performance efficiency

Clean code is not part of the four points, but it clearly helps achieve maintainability and reliability. Even those who are not in favour of clean code would agree that it results in code that is easier to read. Something that is easier to read and easier to understand is simply more maintainable. If you combine clean code with Test-Driven Development, you'll end up with fewer bugs that further increases the maintainability and also the reliability of software.

Static code analysers with their recommendations on removing bugs, code smells and vulnerability issues are useful tools to deliver more correct, more clean and more maintainable code.

Sadly, code quality is often sacrificed by both developers and management in order to move forward faster. Even when a static code analyzer is turned on and the scan results are pushed to the pull requests, developers often ignore fixing the reported issues. It's slightly better when at least they comment on them and write why they don't fix them.

Considering that, making fixing the issues mandatory is an understandable approach to avoid the negligence of developers and therefore increase code quality.

## Potential issues

Delivering code that is as clean and as free of bugs and code smells as possible is of course desirable. Making an *SCA* a gatekeeper still raises some questions. Let's enumerate the potential issues introduced by an *SCA* as a gatekeeper.

### Rules or recommendations?

Most *SCA*s clearly call their policies _rules_, not guidelines or recommendations. As the set of enforced rules is configurable, it makes sense. You either adhere to some rules or not. It's your choice. At the same time, having rules does not mean that they are always applicable.

Static code analysis is not an easy and straightforward task and the rules are not complete or perfect. There are cases where rules are useful in a given context, but in another, they might even go against best practices. For example, having a `default` case in a `switch` statement might be enforced by an *SCA* rule and it probably makes sense when you `switch` over an integer value.

On the other hand, when the `switch` is combined with an `enum,` [deliberately deleting the `default` and introducing instead a runtime exception as Matt Godbolt suggested](https://youtu.be/nLSm3Haxz0I?t=1534) and treating warnings as errors is part of compile-time enforcement of dealing with all the different `enum` values. Having informally discussed this with some engineers working on an *SCA* product, they are also aware of some inconsistencies, they are actually trying to make `switch` related rules better and they'd advise using rules as guidelines

On the very same rules, other senior engineers said that they met cases when they explicitly didn't want to break the compilation when a new value is given to an `enum`, even if that new value is not handled everywhere. Obviously, this was only one example, we can easily find others.

In some cases, even an *SCA* asks the developer to "verify" whether a reported issue is a real issue or not. Sometimes it's just a potential problem, but not necessarily a real one. Blocking based on such is clearly not acceptable.

### Blocking incremental improvements

Having a clean *SCA* report is obviously something we prefer, but it's not always realistic. Such an expectation makes gradual improvements too difficult. Imagine the following use case. Bob sees a badly named variable. Maybe the variable name is simply not meaningful enough, or maybe it's using [a pattern that makes an *SCA* cry](https://rules.sonarsource.com/cpp/RSPEC-978). Bob updates the name and creates a pull request. Imagine that the mentioned variable was a non-private class member. For example, Sonar will raise an issue mentioning that.

What should we do?

Should we silence that error? Definitely not. That's something we want to fix but it might require more time and effort.

Should we make Bob run errands to report somehow that this is an old issue and should be ignored? Maybe. But does anyone think that Bob will raise any other improvement if reporting on something takes more time than the improvement he made?

Should we keep updating the pull request until all the old issues related to the improvements are fixed? That's probably fine for devs, but it's hard to imagine any sane product owner supporting this idea...

### Blocking meaningful organization of changes

When teams take time for housekeeping and dedicate some time to clean up the codebase from SCA issues they have many different ways to get organized - if they get organized at all. One meaningful way is that everyone gets a module or a repository to work on and a given rule. It can be for example that someone has to [use in-class member initialization when possible](https://rules.sonarsource.com/cpp/RSPEC-3230). If the member declaration has other issues, such as using a reserved name, shadowing another one or if it's just not private, the developer with the best intentions in mind cannot merge a code quality improvement without additional efforts.

### Blocking delivery of new versions

It can easily happen that a new feature or a bugfix will require touching a line which has some rule violations. Or maybe the line violated no rules in the past, but new rules have been added and it's not _clear_ anymore. It might be something easy to fix and in such cases, an immediate fix is usually preferable, sometimes it's not possible. Either not possible at all due to some strange 3rd party APIs, or it's not possible easily. In the former case, it makes sense to mark it as an exception and it should be rare, but in the latter case, the developer is just being blocked to deliver without additional hassle even when the delivered code is not responsible for any new SCA issues.

### Lack of trust

Using a static code analyzer with a complex set of rules as a gatekeeper also shows a lack of trust toward developers. _"We, project managers, don't trust you guys on the line that you can deliver high-quality code, we don't trust that you can make a good judgement based on the analysis you're given. We don't trust that code reviewers will do a thorough job and block any PR with any meaningful outstanding static code analyzer warnings."_

I see the following vital counterarguments.

A failing unit/integration test can also block a pull request and we accept such "tooling". While it's completely true, a failing test

*   indicates a functional failure
*   was directly introduced by developers and they can delete or fix it in case it's considered a faulty test.

There are cases when the development teams don't consider such warnings and they deliver low-quality code. That's true. Developers rarely enjoy such situations and they do deliver low-quality code because they are either pushed to deliver too much by their management or because their level of experience is not adequate and the ratio of junior/senior developers is unhealthy. Neither of those reasons will be fixed by tooling, it'll only add additional frustration both for the developers and the management. Developers will see that meeting the unrealistic expectations becomes even more difficult and management will see a further decreased output.

## Software engineers disapprove this solution

I discussed this topic with several engineers who are dedicated to high code quality, and I found nobody who liked the idea of introducing *SCA*s as gatekeepers. I talked to 3 groups of developers

*   Senior engineers who are responsible for setting the company-wide C++ Coding Guidelines
*   Mostly senior engineers at an [C++ On Sea](https://www.sandordargo.com/blog/2022/07/27/cpp-on-sea-trip-report)
*   Engineers from a company delivering static code analyzers

While I found literally no one thinking that using an *SCA* as a gatekeeper is a good idea, many despised the idea of using one in the first place. They were not against higher quality though! They preferred turning on more and more compiler warnings and using sanitizers coming with major compilers, such as the [address sanitizer](https://clang.llvm.org/docs/AddressSanitizer.html), the [undefined behaviour sanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) and the [memory sanitizer](https://clang.llvm.org/docs/MemorySanitizer.html).

I was a bit surprised but also reassured that even engineers working on *SCA*s don't think it's a good idea to use their products as gatekeepers. After all, it makes sense, they have a better view of the above troubles.

## Conclusion

Static code analyzers are useful tools to spot certain code smells or even bugs with its own limitations. Rules might not be complete or might not be applicable in several circumstances. While the idea behind using *SCA* reports as gatekeeper is benign, the road to hell is paved with good intentions. Using a tool as a gatekeeper degrading developer judgement (or making its expression way too cumbersome) will reduce incremental improvements to the codebase which is exactly the opposite of the original intentions. At the same time, it will make it more difficult to integrate new features on time. While it will presumably drive down the introduction of new issues in unhealthy teams, it will increase frustration and distrust everywhere.

*SCA* reports should be kept what they are. An input for engineers to make educated decisions.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

**Special thanks to my [Patreon Supporter](https://www.patreon.com/sandordargo): Piotr Fusik.**