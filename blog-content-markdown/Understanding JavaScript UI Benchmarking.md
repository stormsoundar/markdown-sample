It seems every time there is a new UI Framework or a new feature in your favourite Framework we are shown a new Benchmark showcasing how we are witnessing the next big thing. But fear not. All these fancy tests and benchmarks boil down to 3 categories. Understanding that you will be able to properly discern what the benchmarks are showing, and what value you should put on them. When it comes down to it, regardless of what shiny new thing that will revolutionize the developer experience, these libraries are responsible for ultimately doing one thing: Rendering the screen. I’m going to keep things introductory but if you want to dig deeper into the technical details I strongly recommend reading:

I’m going to focus on runtime benchmarks not loading benchmarks. Loading and size benchmarks are important. However, unless you are serving a static site only make up a small part of the performance characteristics. Yes, first impressions are everything but don’t confuse TTI (Time to Interactive) with general performance.

I’m also going to arbitrarily score how I feel the benchmark category should be valued by the respective audience. Keep in mind that it is completely my opinion and that the value of a particular value of particular benchmarks can vary greatly based on their quality.

## Animation

![](https://miro.medium.com/max/11522/1*tjNnEoMeaihNBucXp5cGhQ.jpeg)

### Overview

These are the oldest benchmarks. Back before we could accurately measure execution time we’d just throw up a test and the end-user would view it themselves to decide which library is better. They generally involve moving several small objects on the screen and maybe use FPS to indicate how performant the library is. The key defining characteristic of these tests is they usually don’t involve adding or removing objects and simply involve updating properties on many objects simultaneously.

### Examples

- Sierpinski’s Triangle Benchmark — The benchmark that introduced React Fiber. I’ve gathered examples from across the web here: [https://github.com/ryansolid/solid-sierpinski-triangle-demo](https://github.com/ryansolid/solid-sierpinski-triangle-demo)

* Circles Benchmark — A remake of the original 2010 Backbone vs Ember Benchmark [https://github.com/ryansolid/circles-benchmark](https://github.com/ryansolid/circles-benchmark)

### What to Expect

These tests traditionally cater to Reactive DOM libraries over Virtual DOM libraries and mutable over immutable data. A similar technique was used for this [MobX vs Redux comparison](https://hackernoon.com/an-artificial-example-where-mobx-really-shines-and-redux-is-not-really-suited-for-it-1a58313c0c70). These tests are usually too extreme to be considered real-world, but they do visually give you some idea of how libraries degrade as the browser gets overloaded.

### Watch Out For

These benchmarks test scheduling overhead so implementations using different methods make a bigger difference. Things like `requestAnimationFrame`, `Promise.resolve`, or `requestIdleCallback` make a much bigger difference than most things the library does. These are techniques anyone can use. A lot of these tests are from an earlier time where these techniques were not expected, so implementors can make certain libraries appear quicker than others purely by using these techniques.

### Strengths & Weaknesses

These benchmarks can show the theoretical maximum performance of the library. They are generally lighter on complex DOM operations so the overhead you see is mostly in the JavaScript library.

For the same reason, these tests generally are not particularly interesting anymore since most modern libraries are quite efficient. Without moving to absurd unrealistic levels short of say making a 3D shooter with DOM elements (why aren’t you using WebGL?) this is not very relevant.

### Score

- Real-World Value: 3/10

- Library Writer Value: 5/10

## Snapshots

![](https://miro.medium.com/max/700/1*-Pr8KdxNc-F715Ov4W_dzg.jpeg)

### Overview

These benchmarks started showing up with the advent of React and the Virtual DOM. They work by generating some unknown blob of nested data and then throw them at the library to figure out what to do with. By looking at how libraries handle varying types of inputs you can get an idea of how the library performs various types of actions.

### Examples

- JS Repaint Performance (DBMon) — Database monitoring scenario with varying mutation rate [http://mathieuancelin.github.io/js-repaint-perfs/](http://mathieuancelin.github.io/js-repaint-perfs/)

* UIBench — A library writer’s swiss army knife of reconciler tests [https://localvoid.github.io/uibench/](https://localvoid.github.io/uibench/)

* DOM Reconciler Benchmark — Calculates the number of DOM mutations caused by different data snapshots [https://somebee.github.io/dom-reconciler-bench/index.html](https://somebee.github.io/dom-reconciler-bench/index.html)

### What to Expect

Essentially the complete opposite of the previous category. These tests cater to immutable data structures and diffing engines. Since the library has no access to the data construction these tests tend to exclude Reactive libraries that use specialized data structures. It’s a test that invites a scenario. Very little real-world application here unless you are really intent on diffing large trees. There are other places in need of optimization if re-rendering large trees of data from the server at a regular rate seems like a reasonable idea.

### Watch Out For

Since these benchmarks tend to involve just blasting data, libraries have some room to interpret how to handle it. This makes these benchmark the hardest for an end-user to discern. DOM operations, especially creation and deletion, are always the most expensive, and it is possible to exploit in some tests that the DOM will always be in the same shape even if the values differ. Be careful of libraries that use DOM recycling, reusing, or pooling. These patterns might be harmless in benchmarks but can cause visual jankiness, incorrect behaviour, and view corruption. If you are confused I recommend reading this and the linked articles: [https://www.stefankrause.net/wp/?p=342](https://www.stefankrause.net/wp/?p=342).

> _It is actually more serious than even those articles indicate when you consider things like Web Components and the future of the web platform. Most Virtual DOM libraries have moved away from these tricks but every few months there is some new framework believing they cracked the performance boundary by inventing these methods. Don’t be fooled. Run away as fast as you can. You will thank me later._

These benchmarks can be valuable for comparison between some libraries but completely useless for others. Simply the solutions that accompany certain problems will differ with the approach making portions of these benchmarks non-applicable. Most Web Applications are tables and grids of text and media data that you incrementally load(if you are efficient) and perform incremental actions on. These tests are the opposite of incremental. For some libraries, these can represent a worst-case scenario. For others, the benchmark really makes no sense.

Watch out for inclusions of Reactive libraries in these benchmarks, as they generally will score poorly(and I mean really poor, like worse than `innerHTML`). It is possible to engineer a solution for these libraries to work but they live in userland since it is outside their intended usage. And so implementors of these benchmarks generally don’t bother with anything beyond a naive(and incorrect) approach.

### Strengths & Weaknesses

These benchmarks are amazing at being able to characterize initial render bottlenecks under various scenarios. They also let you test worst-case scenarios readily. There is a lot of control here so you can spend days doing different variations of benchmarks to understand different things. You can get good measurements on operations without resorting to absurd amounts of data.

Their greatest weakness as already mentioned is their specificity. Even different reconciler approaches interfere with comparable testing. Their detachment from the vast majority of real-world cases does not help their general appeal either. These are the hardest to pull off right. [UIBench](https://localvoid.github.io/uibench/) really is a standout for quality in this category given its variety of testing and measuring approaches and its ability to categorize different techniques being used.

### Score

- Real-World Value: 2/10

- Library Writer Value: 8/10

### Todos

![](https://miro.medium.com/max/700/1*78o38k-_H7cbSiFL4pKPGA.jpeg)

### Overview

These benchmarks are usually based on taking a demo application and then ramping it up to a place where meaningful measurements can be made. They aren’t all Todos apps (and not all todos benchmarks fall in this category) but a list app has all the major DOM operations you typically perform in an application. These benchmarks attempt to present a real-world view of performance by starting from a demo app and triggering tests by simulated user input.

### Examples

JS Framework Benchmark — One of the most popular benchmarks out there [https://github.com/krausest/js-framework-benchmark](https://github.com/krausest/js-framework-benchmark)

TodoMVC Perf Comparison — Older Todo comparison but has all parts [https://github.com/lhorie/todomvc-perf-comparison](https://github.com/lhorie/todomvc-perf-comparison)

### What to Expect

These benchmarks generally let the implementor manage their data so they do get to showcase what real library code looks like (when used under optimization scenarios). Libraries can show off the features they wish to showcase, so viewing the code is generally a much more pleasant experience since it should look more or less like a familiar Todos app. When looking at these benchmarks it is almost pointless to look at the performance alone. The idea is you picture yourself writing the code. If the code isn’t inviting for this type of benchmark, the implementor should consider it a fail.

### Watch Out For

These are not actually real-world. The extremes that are needed to get meaningful results on modern devices even with CPU throttling and mobile simulation only do so much. Since these tests give so much control to the implementor, one can often just bypass the library at times and add plain JavaScript optimizations on top. The test scenarios are completely known so you can just write code to do the work optimally in a certain scenario if you want to win.

Pretty much all cheats applicable to other benchmarks apply here so maintainers need to be vigilant. In specific, look for differences in event handling(delegated or not), render scheduling (`requestAnimationFrame`), node reuse(keyed vs non-keyed).

### Strengths & Weaknesses

These benchmarks are good at showcasing the developer experience. And they can cover a wide variety of real-world cases even if the scale is inappropriate.

However, they are likely the easiest to abuse intentionally or not. You need to make sure the comparison is attempting to be fair to get the most of them. Even if starting from a real-world intent don’t end up being much more realistic when push comes to shove.

### Score

- Real-World Value: 5/10

- Library Writer Value: 7/10

## Conclusion

Beware of benchmarks.

No seriously. Beware of benchmarks. Very few seek to show more than what the presenter wishes. Almost all are flawed in some way. Hopefully, after this article, you will be able to at least bucket them as you come across new ones. They aren’t all useless to everyone. They can test very specific things that are especially useful for library authors to ensure they have good performance in various situations. But when looking at them to make decisions, make sure to understand what they are attempting to show.
