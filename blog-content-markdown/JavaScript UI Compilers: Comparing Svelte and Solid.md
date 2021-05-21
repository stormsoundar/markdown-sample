Last month release of [Svelte](https://svelte.dev/) 3 has got to be the most interesting development in frontend UI libraries so far in 2019. Even more so the accompanying article and video by
[Rich Harris](https://medium.com/@Rich_Harris?source=post_page-----cbcba2120cea--------------------------------), [Rethinking Reactivity](https://svelte.dev/blog/svelte-3-rethinking-reactivity) really drive home points that are too often passed over. For me that culminates at the 23–25 minute mark of the video. “How to speedup your code?”, he asks. “Get rid of it.”

After watching most of the web content at Google.IO 2019, it was clear this message was not one everyone is looking at. Where the last few years Google has painted a standards forward new simpler web, with heavy placement on Web Components and Polymer, this year was all React, Vue, and Angular (and a touch of Preact when they were being bundle size conscious), paired with a number of talks on Web Assembly and Native mobile. Throughout talks of streaming SSR, and deferred partial hydration, I kept hearing Rich’s voice come back to me going “Get rid of it.”

Different context for sure. But it definitely led me to digging deeper into that claim. I updated the [JS Framework Benchmark](https://github.com/krausest/js-framework-benchmark) implementation of Svelte to the latest version. It definitely slimmed down the implementation code, but the performance improvements were negligible. I did knock another kilobyte off the implementation so it did live up to those claims. And that’s the thing Svelte 3 is a big change in syntax but ultimately delivers the great performance and extremely small bundle size it always has. That means that in practice it often produces bundles smaller than Preact or your favorite “just 2kb” library and probably out performs it.

But that’s still today. What I really want to know does this approach have legs. Solid.js is another library that uses this compilation approach but differs in methodology. I’m going to look at each of Rich’s claims and contrast each library’s approach with hopes of separating the facts from the hyperbole.

## Performance

Both libraries are based on Reactive fine-grained change propagation. The everything is a spreadsheet mentality. This approach has the benefit of requiring little overhead to do pinpoint updates. Classically the cost for these sort of libraries has been setting up the reactive graph on initial render. Pre-compilation significantly lowers the overhead here, making it a great match for the technique. And that is really the first thing to acknowledge here:

> _Svelte is still a library that gets bundled with your JavaScript code like every other library out there._

What makes it interesting is the compiler can tell what features you are using and through clever automation of generating import statements, ES Module Tree Shaking can only include the parts you use. It is a small runtime, but is something.

For performance benchmarks I’ve used the [JS Frameworks Benchmark](https://github.com/krausest/js-framework-benchmark). Unfortunately the Benchmark that Rich uses in his talk is pretty much useless. It isn’t just for the reasons [Dan Abramov](https://medium.com/@dan_abramov?source=post_page-----cbcba2120cea--------------------------------) makes in the twitter back and forth in the video. It actually tests nothing even remotely useful as noted [here](https://github.com/somebee/dom-reconciler-bench/issues/7#issuecomment-441683847) and agreed upon by the benchmarks author. Further in the thread it is brought to light the React implementation is poorly implemented. JS Frameworks Benchmark on the other hand provides tests across the board from startup, through update, including bundle size and even memory profiling.

![](https://miro.medium.com/max/700/1*DGGy6JXWpsl-EYBzvbxZHA.png)

As you can see it isn’t particularly close. In the scheme of things Svelte has good performance compared to other popular libraries but it is not class leading by any means. Why is Solid so much more performant? Both libraries do similar things, but there 5 things Solid does that specifically leads to superior performance:

### 1. Optimized DOM List Reconciliation

Svelte does have a keyed list reconciler but it uses a much more primitive implementation. Svelte’s reconciler is much smaller, but Solid’s is much more sophisticated. Svelte’s approach is not as naive as many libraries, properly handling swapping rows without moving every row in between, but it is not nearly as optimized for single item change operations. This is the majority of the performance difference when dealing with lists.

### 2. Template Cloning

Solid uses node.cloneNode instead of document.createElement to create nodes. This reduces temporary memory usage. It does require using comment nodes as placeholders but the overall impact on repeated list items with atleast 3 nodes in them is noticeable.

### 3. Event Delegation

Solid does implicit event delegation on camel case event handlers. This is more performant than letting the DOM handle adding 2 handlers per row. It obviously is possible to do explicit event delegation but it requires more code to do explicit lookup/tree traversal to tie event type and row data together. The previous version of Svelte implementation didn’t do it so I didn’t add it, and I’m currently waiting on Svelte community or Rich himself to weigh in on the relative importance of the “performance” vs “write less code”.

### 4. Synchronous Update Batching

When you are constantly blasting the DOM with updates deferring them with an Animation Frame or even a Promise is a must for performance. However, if you are making just a single set of changes the deferred timing of updates actually has a cost. Libraries generally either run all their updates synchronously one by one or batch them in a deferred microtask. You know the whole state not being updated after the setState call in React. Solid provides an explicit syntax for batching actions, and its setState helper can take lists of changes all applied at the same time.

### 5. Explicit Reactivity and Computations

Svelte’s compiler can automatically identify when you are connecting to a value, and will automatically setup everything you need to handle its value changing. But it is difficult for it to tell if you ever intend to change that value. Additionally it is difficult with plain JavaScript syntax to know the intent of dealing with the reactive atom or the value it holds. This can lead to additional synchronization on boundaries like Components. Svelte’s store mechanism uses a different syntax to handle this shortcoming. The existence of computations are some of the heaviest code for Reactive libraries so having the ability to not include their creation greatly slims down performance. Sometimes automatic isn’t better.

**Bottom Line:** Given Solid’s performance being so close to painfully hand optimized Vanilla JavaScript code, it is safe to say raw performance is not a bottleneck for this approach. In fact, performance might be one of pre-compilations greatest strength. It is likely we will see better performance out of Svelte in future versions.

## Write Less Code

Write less code, write less bugs. Everyone “knows” that less code has some correlation with writing less bugs. The classic being 1 bug for every 10 or so lines of code (LOCs). Whether this is just incidental or a matter of cognitive overhead is something people have been digging into for years. I’m not going to pretend to know what the right answer is. I intuitively feel that it has more to do with number of expressions than with LOCs or verbosity of syntax. It has more to do with number of expressions of intent than whether one chooses to spread a variable declaration over multiple lines. For Svelte 3’s release
[Rich Harris](https://medium.com/@Rich_Harris?source=post_page-----cbcba2120cea--------------------------------) also published an article [Write Less Code](https://svelte.dev/blog/write-less-code) which is also worth a quick skim.

Making your code more expressive, allowing the written code implicitly do more for you, is a pretty compelling advantage that pre-compilation makes possible. It is hard to argue that plain JavaScript assignments are not both clear in intent and lend to less noise. Svelte’s $ label to indicated computations is much less verbose than React like useEffect and the like. So Svelte’s approach must clearly be the best at approaching this goal.

Maybe. JavaScript already has a built in that is incredibly expressive: Functions. Taking this at an extreme if you’ve ever seen code written in RxJS versus it’s imperative counterpart the amount of implementation code is not even close. Have you seen Redux in one line of code:

```javascript
action$.scan(reducer).subscribe(renderer);
```

I’m not suggesting this is more or less bug-prone. Just that if less code is the desire reducing the syntax down to plain object access and simple HTML templates might not actually be the solution. Composing templates out of functions with JSX can also often lead to less code for repeated parts that you do not wish the overhead of componentizing. To my knowledge, template partials are not a feature in Svelte.

In practice I’ve found this to mean that Solid in many cases has less code in its implementations. For comparison in the JS Frameworks Benchmark implementation:

**Svelte - ([link](https://github.com/krausest/js-framework-benchmark/tree/master/frameworks/keyed/svelte/src))**

Lines of Code - 97

Characters (including spaces) - 3705

Characters (without spaces) - 2829

**Solid - ([link](https://github.com/krausest/js-framework-benchmark/tree/master/frameworks/keyed/solid/src))**

Lines of Code - 66

Characters (including spaces) - 3176

Characters (without spaces) - 2536

**Bottom Line:** Verdict might still be out on what is the optimal way to use the compiler to write less code, but it isn’t as simple as pointing at plain JS objects and no explicit API surface. Expressive constructs can often me more expressive than the plain language while being more explicit. Being simple and being easy can be 2 very different things. That being said these are 2 of the smallest implementations in terms of implementation code size.

## Small Bundles

This section is going to be a short one. Svelte pretty much wins hands down on bundle size against pretty much anything you are going to throw at it. The exception might be a small DSL driven library where all imports are implicit and features are very simple. Libraries like [attodom](https://github.com/hville/attodom) and [hyperapp](https://github.com/jorgebucaran/hyperapp) come to mind both using HyperScript like syntax allowing for easy explicit importing of just the functions used.

Even if Solid uses the same precompilation technique of only including what is used, Solid’s list reconciler adds +3kb to the minified bundle alone. It’s expressive setState method adds +1kb, it’s more sophisticated change management core another +1.5kb or so, and it’s event delegation another +1kb. Svelte’s whole runtime probably doesn’t add up to the sum of just those pieces. These are things that you will generally be bringing into most components even if they are opt in. Simply put even if using the same approach as Svelte’s, Solid’s tradeoffs for performance and writing less code come at a cost in bundle size. Not a significant one compared to many libraries, coming in with its common includes only slightly bigger than Preact, but the fact that Solid can shrink down to almost no runtime code isn’t actually that applicable in practice.

EDIT: As of Solid 0.17.0 the size has reduced drastically. The list reconciliation code has reduced to about 1kb minified and its reactive core down to 700bytes. In common bundles Solid is now generally smaller than Preact and within a few kilobytes of Svelte in this benchmark. Solid is currently one of the smallest implementations in the RealWorld Demo, weighing in 25% smaller than Svelte:

**Bottom Line:** While this approach of only including what is used allows for libraries like Svelte and Solid to ship with more features like transitions or implicit event delegation without fear of always bringing weight into your project, it still only achieves the small bundles if the implementations of those features are small.

## Moving Forward

I think Svelte, and Solid both represent an interesting radical change in how we view modern JavaScript UI development. I am all for the [Virtual DOM is pure overhead](https://svelte.dev/blog/virtual-dom-is-pure-overhead) mentality, but I’m concerned the rhetoric is too simple even if the overall message is on point. It is way too easy to find foils to each argument. In fact I recommend those who want a levelled view to read
[Boris Kaul](https://medium.com/@localvoid?source=post_page-----cbcba2120cea--------------------------------)’s [readme](https://github.com/localvoid/ivi#virtual-dom) for his library ivi. He has written the most performant Virtual DOM library, and goes into detail the tradeoffs.

The thing is for all the benefits in size at a certain point as your app grows you will likely include most of the bundle. Does Tree Shaking scale? Perhaps not alone. What is interesting is where this can go when combined with Code Splitting. I can only speak to Webpack, and this might already be possible with Rollup. What if we could use the same static analysis to split your runtime across multiple files as needed? If a larger runtime is ultimately unavoidable there is a reasonable chance at least that it is not all necessary at the onset. What is great about this if it works is no fancy technical mechanism. Just the same old dynamic script loading we’ve been doing for a while as long as the library supports it.

However, there is something here. Despite shortcomings today, between both Svelte and Solid, you can start to see the potential of where this precompiled approach can get to. Solid has shown that the highest pinnacle of performance is possible with this approach, and Svelte has shown how it can lead to the smallest bundle size. Both have some of the most streamlined syntax and lend to a development experience where you write less code.

Ok, so “Get rid of it” perhaps not so simple. But I feel we’ve just scratched the surface, and there is lot more room to explore.
