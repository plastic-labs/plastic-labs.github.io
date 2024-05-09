---
title: "Comprehensive Analysis of Design Patterns for REST API SDKs"
date: 05.09.2024
tags: ["blog", "dev"]
---

This post is adapted from [vineeth.io](https://vineeth.io/posts/sdk-development)
and written by [Vineeth Voruganti](https://github.com/VVoruganti)

## TL;DR

After several months of managing the SDKs for Honcho manually, we decided to
take a look at the options available for automatically generating SDKs. 

From our research we picked a platform and have made brand new SDKs for Honcho
that use idiomatic code, are well documented, and let us support more languages.  

---

For the past few months I have been working on managing the
[Honcho](https://honcho.dev) project and its associated SDKs. We've been taking
the approach of developing the SDK manually as we are focused on trying to find
the best developer UX and maximize developer delight. 

This has led to a rather arduous effort that has required a large amount of
refactoring as we are making new additions to the project, and the capabilities
of the platform rapidly expand. 

While these efforts have been going on a new player in the SDK generation space
dropped on [hacker news](https://news.ycombinator.com/item?id=40146505).

When I first started working on **Honcho** I did a cursory look at a number of SDK
generators, but wasn't impressed with the results I saw. However, a lot of that
was speculative and Honcho was not nearly as mature as it is now. 

So spurred by the positive comments in the thread above I've decided to do a
more detailed look into the space and, also try to develop a better understanding
of what approaches are generally favorable in creating API client libraries. 

## Background

For a full understanding of Honcho I recommend the great [[A Simple Honcho
Primer|Simple Honcho
Primer]] post, but I'll
try to summarize the important details here. 

Honcho is a personalization platform for LLM applications. It is infrastructure
that developers can use for storing data related to their applications, deriving
insights about their data and users, and evaluating the performance of their
applications. This functionality is exposed through a REST API interface with
the following resource constructs. 

|\_\_\_\_Apps  
|\_\_\_\_|\_\_\_\_Users  
|\_\_\_\_|\_\_\_\_|\_\_\_\_Sessions  
|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_Messages  
|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_Metamessages  
|\_\_\_\_|\_\_\_\_|\_\_\_\_Collections  
|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_Documents  

So Apps have Users that have Sessions and Collections where Sessions can have
Messages and Metamessages and Collections can have Documents.

At the time of writing this post Honcho is being manually maintained with a
singular client SDK for the Python ecosystem. The SDK is co-located in the
[repo](https://github.com/plastic-labs/honcho/tree/main/sdk).

The SDK is written in an object-oriented style where the top level `Honcho`
object will return lower level objects such as a `User` object and `Session`
objects. These objects contain the CRUD methods necessary to use them i.e.

```python
from honcho import Honcho

honcho = Honcho("Test App Name")
honcho.initialize()

user = honcho.create_user("username")
session = user.create_session()
```

There is an Async version of the SDK with an `AsyncHoncho` class that uses
objects such as `AsyncSession` and `AsyncUser`.

## Guiding Questions

Before evaluating the below platforms I wanted to investigate a few questions I
had about how to design SDKs and how they are generally maintained in other
organizations. I've also included some questions I want to think about when
looking at the different platforms

I'm doing this through the lense of a Python developer as Honcho currently only
has a Python client library with plans to quickly expand to other ecosystems.

General SDK Questions

1. Do developers prefer an object-oriented approach or singleton approach for
   client SDKs that wrap a REST API?
2. What additional features are generally important in SDK design outside the
   functionality of the API (retry, pagination, etc.)?

Platform Specific Questions

1. How readable and easy to understand is the generated code?
2. How customizable is the end result?
3. How easy was it to use the tool?
4. What approach does the tool take? Object-oriented or singleton?
5. How does it handle async vs sync interfaces?

## Research

> First I took a look at sources and posts onlines that talk in general about
> developing SDKs. This isn't an exhaustive look at every link I looked at, but
> ones I thought were relevant. The notes are messy and not necessarily fully
> formed sentences.

[Any design patterns and tips on writing an API client library](https://www.reddit.com/r/Python/comments/vty3sx/any_design_patterns_and_tips_on_writing_an_api/)

Things they are laying out here. 

One person
- Auth is really hard to figure out 
- Retry logic and pagination is really important

Another person
- Keep data objects as just data and use other objects for transformations

^ basically advocating for the singleton model

Person 3
- Also arguing for singleton approach. Made a good case where if you really only
  care about lower level stuff it's annoying

Firstly, don't go over the top with object-oriented modelling. If you've got an API call like:

`GET /locations/12345/customers/65432/orders/87678768`
Don't implement this as:

```python
client.location(12345).customer(65432).order(87678768).get()
```
Just implement:

```python
client.get_order(12345, 65432, 87678768)
```

that last one is better tbh it's just managing that data isn't done within the
object, which is my main problem. 

So arguments for singleton approach are 
- harder to go to lower levels from the start

The object-oriented approach looks more readable.

[A Design Pattern for Python API Client Libraries](https://bhomnick.net/design-pattern-python-api-client/)

It mainly covers how to build an singleton library but has this one snippet at
the end. 

> Other types of APIs
> This pattern works well for RPC-style APIs, but tends to break down for more
> object-based or RESTful APIs as having a single interface class gets messy
> quickly. In those cases I find it makes more sense to break the interface down
> to resource-level, modeling things more like an ORM. I'll cover that in a later
> post, next time I find the need to build one.

At the time of this research there was no follow-up post.

[Ask HN: Best practices (and examples) for designing client libraries for
APIs?](https://news.ycombinator.com/item?id=23283551)

The first comment actually advocates for an object-oriented model but just using
the top level client object for authentication and setup stuff. 

Most of the sentiments kind of make me think using an object-oriented model
might make more sense. 

[How to design a good API and why it matters](https://dl.acm.org/doi/abs/10.1145/1176617.1176622)

Nothing really to note from there. It's more about the API itself and not the
SDK.

[Building A Creative & Fun API Client In Ruby: A Builder Pattern Variation](https://medium.com/rubyinside/building-a-creative-fun-api-client-in-ruby-a-builder-pattern-variation-f50613abd4c3)

This is basically a guy who saw an singleton approach and said I want an object
oriented approach. 

[How to design your API SDK](https://kevin.burke.dev/kevin/client-library-design/)

A developer from twilio talking about their approach to creating helper
libraries and client libraries. 

A point he makes is that "If you've designed your API in a RESTful way, your API
endpoints should map to objects in your system"

This point isn't explicitly asking for the object-oriented approach as the
singelton approach just moves the verbs to the singleton, but usually still has
data only objects for the different resources. 

I say this, but the examples seem to use an object-oriented model.

[How to build an SDK from scratch: Tutorial & best practices](https://blog.liblab.com/how-to-build-an-sdk/)

Written by one of the SDK generation platforms. 

It talks in general terms about creating data objects and mapping methods to
endpoints. One of the points is suggests as a good grouping method is to group
functions in service classes, essentially advocating for an object-oriented
model. 

[Designing Pythonic library APIs](https://benhoyt.com/writings/python-api-design/)

The two takeaways that are the most important to me when looking at these are

* Design your library to be used as import lib ... lib.Thing() rather than from lib import LibThing ... LibThing().
* Avoid global state; use a class instead

From that it seems using a singleton for are actions/verbs and then storing data
in dataclasses would support both of the requirements. The examples in the post
show a class that has functionality.

Using tree-shaking style imports should also allow for lower scopes. For example
when only worrying about messages for a particular session in honcho a user
could import just the messages namespace i.e. 

```python
from honcho.apps.users.sessions import messages

messages.create("sample")
```

so there are pythonic ways to make the code less verbose. However the benefit of
having the entire string is making it clearer what messages are being discusses.
Are these Honcho mesages? LangChain messages? It can get messy that way
especially in the LLM space where many libraries and components are
converging on similar naming schemes. 

[Build a Python SDK](https://wwt.github.io/building-a-python-sdk/)

Looks like a guide made by Cisco. I paid special attention to the "API Wrapper
Module" section. It was a really barebones example in this guide that just
implemented a very small client and put most of the attention on how to manage
the connection logic. 

It used one singleton object that had all the methods available for the API.
There was no concept of resources or data objects here as no data was being
persistently stored. 

[How to build a user-friendly Python SDK](https://medium.com/arthur-engineering/best-practices-for-creating-a-user-friendly-python-sdk-e6574745472a)

Noticing the trend of abstracting all connection logic for http requests to a
separate module and havign reusable methods for different http functions. 

Main focus of the post was just on good practices of documentation, testing, and
logical organization. 

[SDKs.io](https://sdks.io/docs/introduction/)

A more comprehensive repository of thoughts and principles around SDK design.
Made by APIMATIC. which seems to be another player in the code generation space. 

I paid special attention to the **Build** section under **Best Practices**, and
specifically the endpoints to methods and the models & serialization. 

They state putting all methods in a single class (singleton) has the advantage
of reducing the need to initialize classes, but can make the class size very
large if there are many endpoints. 

Grouping methods into different namespaces could probably remove this problem
too. A nested singleton can reduce the confusion, while still not needing to
mess with classes and objects. 

It generally seems popular to at the very least create types and data objects
for handling and storing API responses. They help with readability, type hints,
data validations, etc. Regardless of the singleton or object-oriented approach
data objects are something that should probably still be included. 

[Generating SDKs for your API](https://medium.com/codex/generating-sdks-for-your-api-deb79ea630da)

Advocates for using generators for making SDKs and talks about how different
languages have different idioms and conventions that will be hard to manage. 

Also mentions having the generator create data models. 

[Guiding Principles for Building SDKs](https://auth0.com/blog/guiding-principles-for-building-sdks/)

Some key insights

* Make sure documentation is very comprehensive
* Try to minimize external dependencies
* Have modular design patterns that make it easy to extend and pick and choose
features.

[Should I implement OOP in a REST
API?](https://www.reddit.com/r/flask/comments/1755ob0/should_i_implement_oop_in_a_rest_api/)

Most people seem to be saying a full OOP method is overkill, but there are
people advocating for having a controller class with methods that take data
objects as inputs. Essentially advocating for the singelton approach with data
only objects. 

### Analysis

Many of the generic concerns of SDK design do not have to do with the UX of the
SDK for the end developer, rather background processes that an SDK handle. This
includes:

* Authentication
* Retry Logic
* Pagination
* Logging

When it comes to the actual developer experience and interfaces for interacting
with the SDK the community seems a bit split. This is very much because of the
boring fact that REST APIs are designed very differently and so it depends on
the specifics of the API. 

Some APIs have many resources with basic CRUD operations. Others have many
different endpoints, but only have a few resources. The singleton architecture
vs a strict object-oriented approach again seems to depend a lot. Some sources
advocate for a strict object-oriented approach where classes have their own
methods, while others advocate for a singleton approach stating objects are
overkill. 

However, the singleton approach doesn't completely abandon the idea of objects
and almost always advocates for data objects, or some kind of models that can be
used for type hints and validation. 

There is some tradeoff regardless with problems arising at different levels of
scale. The singleton approach could be verbose and cumbersome at smaller scales,
but the object-oriented approach may not be a readable, and it could be unclear
what methods are doing in complex codebases. Even GPT-4 couldn't decide between
the two.

![Asking GPT-4 about Singleton vs Object-Oriented
Approaches](/assets/sdk-gpt-4.png)

Again and again, the best way to approach SDK development is to just do whatever
is easier, and create tons of documentation that will help developers navigate
your [API Ladder](https://blog.sbensu.com/posts/apis-as-ladders/). Someone will
get confused regardless of what you do, so the key is to make sure the SDK makes
sense (even if it's not the most efficient or clean) and remove hurdles for
users to navigate errors and mistakes. 

## SDK Generation Platforms

With a sense of the best standards for SDK design and additional features that
should be supported in the SDK I want to look at a few different options to
determine what is the best solution to go with.

Below is a list of the different platforms I wanted to review

- [Stainless](https://www.stainlessapi.com/)
- [Speakeasy](https://speakeasyapi.dev)
- [liblab](https://liblab.com/)
- [OpenAPI-Generator](https://openapi-generator.tech/)

I was using the OpenAPI Spec for Honcho that was housed at
https://demo.honcho.dev/openapi.json.

### Stainless

Since the hacker news thread for the release of stainless is what spurred this
research I decided to try them out first. 

From their web portal they were able to take a link to the OpenAPI spec and
generate a NodeJS and Python SDK immediately. There was no tweaking or anything
necessary. 

I mainly paid attention to the Python SDK. The code was very readable and made
sense. I also liked how it used `httpx` and `pydantic` by default and made an
`async` version of the interface. They took the singleton approach to the design
of the interface. There was also built in capabilities for retries, pagination,
and auth.

There's also capability for adding custom code such as utility functions. 

### Speakeasy

Speakeasy required me to do everything locally through their `brew` package. It
did not immediately accept the OpenAPI Spec and required me to make some tweaks.
These were low-hanging fruit, and their cli has a handly AI tool that will
diagnose the issue and tell you what to fix. 

I just had to add a list of servers and deduplicate some routes. I'm happy it
found these errors, but there was some friction for me to get started. Stainless
just worked out of the box and made some logical assumptions. 

The generated SDK didn't feel as strong as the stainless one. There didn't seem
to support `async` methods, it did not use `pydantic` and used the built-in
Python `@dataclass`. The methods had really unwieldy names, and looked like it
would need a lot of tweaking to get it more production ready.

### Liblab

Also had me do the generation from the cli using their npm package. It was
pretty straightforward to login and give it an API spec. Liblab seems to require
a lot tweaking to get better results. It gave me several warnings asking me to
add tags to my API Spec. I did not add them and went ahead to look at the
generation. 

> I'm not opposed to adding the tags if necessary, but I was able to get good
> results without adding them on other platforms.

The results were also lack luster. The SDK took the singleton approach as well,
which seems to be the industry standard for codegen tools. The method names
were also unwieldy. It also didn't make use of pydantic and instead implemented
its own `BaseModel` class. It was built on the `requests` model and doesn't seem
to support `async` methods.

### OpenAPI Generator

This is the only one on the list that is not expressly backed by a company
whose main goal is SDK generation. It is however a very popular project with
many sponsors. 

Again, I tried to generate a client from the cli using their npm package. I used
version `7.5.0` and once again gave it my API Spec. It gave a few warnings about
OpenAPI Spec v3.1 not being fully supported yet, but generated a package either
way. 

I again was not too impressed with the results, however I did like it more than
liblab. The method names were also unwieldy, and the project relies on `urllib3`.
I did not see an indication of support for an `async` client. 

The repo did use `pydantic` for typing and data classes, which is a plus.
Once again, the sdk use the `singleton` approach. 

I also did not see any indication of functionality for retry logic,
authentication, or pagination. 


### Conclusion

Overall, Stainless had the results that I liked the most. With almost no work
from me, it produced a high quality SDK that designed things in a sensible way
with many built-in features such as retries, pagination, and auth. 

All the platforms took the singleton approach with a host of data models so
there isn't much to compare in that regard.   

The other platforms did not produce anything unusable, but they seemed to use
less modern features and require a lot more massaging to get a desirable result. 

The docs for stainless also looked more clear, and it seems easier to add
customizations after the fact. 

I will give Speakeasy some kudos for having documentation for different API
frameworks. The FastAPI one made it easy to figure out what I needed to tweak
and how to do it. The AI debugging feature was also a nice help. 

What I'm looking for right now is the platform or tool that can reduce my work
the most and let me focus on other things and stainless achieved that. The
results are not perfect, but it doesn't look like it'll need more than some
slight tweaking and testing to get to a state I want. 

## Results

After reaching the conclusion in the previous section, I took some time to fully
implement Stainless to make SDKs for Honcho and am proud to announce the release
of a new Python SDK, and the launch of a brand-new NodeJS SDK. 

Both of these SDKs will be in separate open source repositories. 

- [Honcho Python SDK](https://github.com/plastic-labs/honcho-python) 
- [Honcho TypeScript SDK](https://github.com/plastic-labs/honcho-node)

Honcho will soon be available for a wide range of ecosystems and platforms,
making it even easier and more accessible to make personalized agents. 
