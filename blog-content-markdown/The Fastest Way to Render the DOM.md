While writing [Solid](https://github.com/solidjs/solid), I’ve had the opportunity to look at a number of different libraries attempt to maximize performance on benchmarks. The truth is the DOM itself is often the biggest bottleneck. That means that when it comes down to it, vastly different approaches can perform comparably. I see new libraries pop up every week, each taking a bit from here and there to put together their perfect combination, and after a while, it becomes hard to wade through some of the noise.

Now don’t get me wrong, I love to see these ideas come together, but they all have tradeoffs and costs, sometimes in ergonomics and most of the time in performance. There isn’t a total understanding of the full repercussions of these decisions, but I want to share a bit of what I’ve observed.

## The Comparison

The [JS Frameworks Benchmark](https://github.com/krausest/js-framework-benchmark) is the best general-purpose benchmark for JavaScript UI Frameworks. For this test since I’m playing with variations, it is necessary to run everything locally instead of using the official results. Results vary from machine to machine and because I’m running on a less powerful one, as we test, performance will fall off more prominently.

We will take what I feel is the best of each rendering approach to illustrate the overhead of specific decisions. I will do this by using Solid’s ability to support multiple rendering techniques to juxtapose different costs as we change different variables and compare that against the best examples of libraries that use similar techniques. Let’s look at the competitors.

### The Solid Variants:

1. **[solid](https://github.com/solidjs/solid)** — This is the stock ES2015 proxy version with succinct setter syntax on top of fine-grained change detection feeding into DOM Template Node cloning. It accomplishes this with precompiled JSX templates. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/blob/solid-variations/frameworks/keyed/solid/src/main.jsx)

2. [**solid-signals**](https://github.com/solidjs/solid) — This version is the same as above but eschews the convenience of proxies for raw Signals. It makes the implementation heavier, but it leaves a smaller bundle and greater performance. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/blob/solid-variations/frameworks/keyed/solid-signals/src/main.jsx)

3. [**solid-lit**](https://github.com/ryansolid/solid) — This version eschews JSX precompilation for Just in Time Tagged Template Literal runtime compilation to achieve DOM Template Node cloning. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/blob/solid-variations/frameworks/keyed/solid-lit/src/main.js)

4. [**solid-h**](https://github.com/solidjs/solid) — This version uses HyperScript to translate to `document.createElement` on the fly. But otherwise uses the same Solid implementation as the others. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/blob/solid-variations/frameworks/keyed/solid-h/src/main.js)

### The Other Libraries:

1. [**domc**](https://github.com/Freak613/domc) — This micro-render library is designed to do the least possible work in change management while maximizing on DOM Template Nodes with a custom runtime HTML String DSL that is hydrated from the index.html. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/tree/solid-variations/frameworks/keyed/domc)

2. [**surplus**](https://github.com/adamhaile/surplus) — This library uses fine-grained change detection with pre-compiled JSX templates to create a series of document.createElement. Solid uses the same underlying change management library. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/tree/solid-variations/frameworks/keyed/surplus/src)

3. [**ivi**](https://github.com/localvoid/ivi) — You may have heard of Inferno but this library is the true pinnacle of the Virtual DOM. It uses a HyperScript Helpers-esque API with Hooks. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/blob/solid-variations/frameworks/keyed/ivi/src/main.js)

4. [**lit-html**](https://github.com/Polymer/lit-html) — Not the first library of its type, but it is the best performing. It uses Tagged Template Literals at runtime to feed into DOM Template Nodes. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/blob/solid-variations/frameworks/keyed/lit-html/src/index.js)

5. [**inferno**](https://github.com/infernojs/inferno) — The quickest of the React clones and one of the fastest Virtual DOM libraries. It uses JSX with special directives to indicate how to get the best performance. [**(Code)**](https://github.com/ryansolid/js-framework-benchmark/tree/solid-variations/frameworks/keyed/inferno/src)

Some of your favourite libraries may not be there, but essentially this combination reflects highly optimized versions of all the techniques that you will see in those libraries. You can view this as today’s best indicator of the ceiling potential of your library. If you are interested in seeing a comparison of popular libraries I recommend [this comparison.](https://medium.com/@ajmeyghani/javascript-frameworks-performance-comparison-c566d19ab65b)

The one notable absentee that I would have liked to add is a Web Assembly entry. Unfortunately, at this point, the WASM entries are essentially vanilla implementations without higher-level data-driven abstractions. The fastest which only slots in the middle of this pack. So it’s safe to say WASM has some potential but we’re not there yet.

### The Results

#### HyperScript (inferno, ivi, solid-h)

HyperScript is a way of representing views as a composition of functions (often an h or perhaps React.createElement). For example:

```javascript
h("div", { id: "my-element" }, [h("span", "Hello"), h("span", "John")]);
```

This is the category owned by Virtual DOM libraries. Even if they use JSX or other templating DSLs ultimately it is converted to per element render methods. This suits constructing a virtual DOM tree in JS every render cycle, but as shown here it can also be used to construct a Reactive dependency graph in Solid’s case.

![](https://miro.medium.com/max/700/1*QmcWyqfVrlgsK_6cmgSn_g.png)

As you can see, the Virtual DOM libraries are much faster here. The overhead of creating the reactive graph hurts Solid here. Notice the difference in benchmarks #1, #2, #7, #8, #9. Conversely, Solid is slightly quicker on the remaining benchmarks which measure partial updates.

![](https://miro.medium.com/max/700/1*6yDVUECT3cBhRpUHixvi7g.png)

Memory is less conclusive. Inferno and this version of Solid are mostly neck and neck, whereas the more performant ivi uses more memory. This is the most memory-intensive version of Solid, but it is worth noting how close memory usage is here.

This is the classic VDOM vs Fine-Grained comparison. Fine-Grained takes the hit upfront to perform better on updates. If this was the end of the story it would be easy to explain VDOM’s dominance in the past years. Suffice to say if you are using HyperScript with Fine-Grained you are probably better off using the Virtual DOM.

### String Templates (domc, lit-html, solid-lit)

Each library here has a few things in common. They render based on cloning template elements, they execute at runtime and make no use of a Virtual DOM. However, each does so differently. DomC and lit-html do top-down diffing similar to a Virtual DOM whereas Solid uses fine-grained reactive graph. Lit-html splits templates into parts. DomC and Solid just in time compile the template into separate code paths for creation and updates.

![](https://miro.medium.com/max/700/1*Se35vdDzgiSlHLmPtc-eHA.png)

This category has the widest range of performance, DomC is one of the fastest libraries and lit-html is the slowest. Solid Lit is right in the middle of the pack. DomC is a testament to how keeping code simple can lead to incredible performance. Its only weakness is #4 since it works by diffing at the leaf nodes, which is more expensive the deeper the structure. It is plenty fast enough but we will need to validate how it scales. Solid Lit is much more performant than Solid HyperScript. At runtime, just-in-time compilation negates most of the downsides of creating the reactive-graph, letting it sneak just in front of ivi, the fastest Virtual DOM library (see full Performance Results table at the end of the article).

![](https://miro.medium.com/max/700/1*O0hGe-PSi0k-9SPVxhYjew.png)

Memory is much better with this bunch. DomC has the smallest memory footprint out of all the competitors. A decent amount of savings comes from rendering by cloning Template elements.

The most interesting takeaway from this group is that runtime code generation can have a minimal performance cost over pre-compilation at the build step. It is perhaps an unfair comparison for lit-html in that sense since it doesn’t leverage this technique, but it is fair to say that currently lit-html, or similar libraries like hyperHTML or lighterHTML, are not the most performant way to use Tagged Template Literals and it is possible to get really good performance even at runtime without the Virtual DOM.

### Precompiled JSX (solid, solid-signals, surplus)

Now on to the heavy-weight class. These libraries use JSX compiled at build time down to DOM and Reactive graph instructions. Unlike the last 2 categories for Fine-Grained libraries, the overhead for initial construction is almost completely removed making this an ideal for libraries of this type. The templates really could be anything, but JSX provides a clear syntax tree that lends to better tooling and developer experience.

![](https://miro.medium.com/max/700/1*pOK0LoIVBW6CH3KKUf9BAg.png)

This group has the closest performance results, but the differences are really important here. All 3 of these libraries use the same change management library in [S.js](https://github.com/adamhaile/S). Using Solid Signals as a baseline shows how observing functions with template element cloning offers the best performance. Solid’s standard implementation adds the overhead of using ES2015 Proxies which adds overhead on all benchmarks. Surplus on the other hand uses `document.createElement` which has more overhead on benchmarks that test creating many rows #1, #2, #7, #8.

![](https://miro.medium.com/max/700/1*H7hQ1HWm7ZrRRx9ncVek7g.png)

Memory scales up similarly. However, in this case, it’s the proxies that have more overhead than the template element cloning.

The takeaway here is that proxies have a real performance cost, and more libraries should be cloning template elements. On the other hand, you could take this performance hit with proxies as a small investment. Solid’s official implementation has the smallest amount of implementation code of all libraries by a long shot weighing in at 66 lines and is even 13% less written non-whitespace characters than Svelte, a library that prides itself in writing less code.

### Best in Class (domc, ivi, solid-signals, vanillajs)

Next, we take the winners of each category and compare them against a brutally efficient hand-crafted version written in plain vanilla JavaScript. What is nice here is that the best in each category represents each of the popular change management approaches. You can even draw similarities from these libraries to the big 3. Solid → Vue, DomC → Angular, ivi → React. That is if you strip them right down to their renderers and shed 60–200kb of code.

So how did we fair?

![](https://miro.medium.com/max/700/1*NCehudMdyE5Q8t7-HksT8w.png)

DomC and Solid are close here and ivi is no slouch either, but DomC is generally faster. Its overhead over the vanilla version is remarkably small but is less efficient for nested partial updates. This benchmark alone is not going to conclusively separate these approaches. Anyone claiming the Virtual DOM is slow or has unnecessary overhead should check themselves. Most libraries will never have this performance.

![](https://miro.medium.com/max/700/1*Ve6YFunf2wuR5JkS7I4T_w.png)

With memory, DomC again shows how small of a footprint it has. Fine-Grained Solid leads Virtual DOM ivi on memory usage.

The most interesting takeaway from these results might simply be how little overhead these libraries have over the vanilla JavaScript version irrespective of method. These libraries are all very fast.

### Bundle Size

Lastly, I wanted to call out bundle size for a moment because I feel this area gets way too much attention. Recent “real world” benchmarks put almost all their attention on these metrics. Yes, bundle size is important and there is a direct correlation on performance, but how much of a difference does it make. I suspect variation in code loading overhead has a larger impact than size.

![](https://miro.medium.com/max/700/1*Ea0Ns22xOiBWeSzpNycI_Q.png)

## Conclusion

As usual with these sorts of comparisons, the results are never conclusive. It’s the journey that matters and what we learn along the way. In this case, we see that the DOM itself is the biggest bottleneck when it comes to the forefront of performance. So much so that there is no clear best technique.

> But there can be only one.

![](https://miro.medium.com/max/645/1*RzPQRIy-WqyEbDsflcsfFw.jpeg)

Nope, it’s never that simple. The DOM is not slow. The Virtual DOM is not slow. But I suppose one good turn deserves another. I will admit it was React’s rhetoric about the Virtual DOM’s performance that led me into this space in the first place. The ignorance of opinions going around was infuriating.

Similarly, the recent chorus of “The Virtual DOM is slow” is just as ill-informed. Rendering a virtual DOM tree and diffing it is going to be pure overhead compared to not doing so, but does not doing so scale? And what if you have to deal with a data snapshot?

What I’ve seen is that for every rule there is an exception. Generally, pre-compilation is the fastest approach when paired with Fine-Grained Reactive libraries, but DomC matches performance with neither. Native JavaScript methods, like template element cloning with Tagged Template Literals, might be the most performant approach backed by a major company (Google) in lit-html. However, it is the slowest of this batch and not even the most performant way to use those technologies. Svelte might be the loudest community around “superior” performance and the library isn’t even able to compete in this crowd.

So even if Reactive programming wins the day, it doesn’t mean all Reactive libraries are fast or that benchmarks are everything. Perhaps contradictory given the depth of this comparison, I think the real take away is there are fast libraries and slower libraries. As much as we’d like to attribute this to some superior technology we are still constrained by the physics of it all.

All Libraries tested on a Single Chart:

![](https://miro.medium.com/max/2820/1*UTkJYuYKJw9TW2qGyRFY0Q.png)
