I was looking around the web and realized we haven’t had a good JavaScript Framework Performance Shootout in over 2 years. So before 2020 wraps up, let’s have a bit of fun pitting these libraries against each other.

How about taking the 20 most popular JavaScript Frameworks and put them head to head using the [JS Framework Benchmark](https://github.com/krausest/js-framework-benchmark)?

> _**Disclaimer**: This comparison is meant to be fun and maybe educational along the way. As always, every library here is performant enough for most things. If anything this should emphasize that performance can come from a variety of different technologies and techniques. While you can use this for reference, you should independently verify performance for your individual use cases. You can find the latest official results [here](https://krausest.github.io/js-framework-benchmark/index.html). If you want understand more about what this benchmark tests I’ve posted a [guide here](https://dev.to/ryansolid/making-sense-of-the-js-framework-benchmark-25hl)._

> _It’s also worth mentioning I’m the author of the Solid Framework. So it impossible for me to show no bias here. But it is my intention to let the numbers largely speak for themselves. With that all aside, enjoy the comparison._

## The Competition

I have taken the latest [Chrome 87 keyed results of the JS Framework Benchmark](https://krausest.github.io/js-framework-benchmark/2020/table_chrome_87.0.4280.66.html). These were run on a Core i7 Razor Blade 15 under Fedora 33 with mitigations turned off.

I have filtered out all the implementations that have issues, and then I grabbed the top 20 libraries by Github stars. For libraries with multiple versions, I grabbed their latest version and most performant variant not using a 3rd party library.

1. [Vue](https://github.com/vuejs/vue) (177k)

2. [React](https://github.com/facebook/react) (161k)

3. [Angular](https://github.com/angular/angular) (68.9k)

4. [Svelte](https://github.com/sveltejs/svelte) (40.5k)

5. [Preact](https://github.com/preactjs/preact) (27.9k)

6. [Ember](https://github.com/emberjs/ember.js) (21.7k)

7. [HyperApp](https://github.com/jorgebucaran/hyperapp) (18.2k)

8. [Inferno](https://github.com/infernojs/inferno) (14.6k)

9. [Riot](https://github.com/riot/riot) (14.4k)

10. [Yew](https://github.com/yewstack/yew) (14.2k)

11. [Mithril](https://github.com/MithrilJS/mithril.js) (12.5k)

12. [Alpine](https://github.com/alpinejs/alpine) (12.4k)

13. [Knockout](https://github.com/knockout/knockout) (9.9k)

14. [Marko](https://github.com/marko-js/marko) (9.9k)

15. [Rax](https://github.com/alibaba/rax) (7k)

16. [lit-html](https://github.com/lit/lit) (6.9k)

17. [Elm](https://github.com/elm) (6.2k)

18. [Ractive](https://github.com/ractivejs/ractive) (5.8k)

19. [Solid](https://github.com/solidjs/solid) (4.7k)

20. [Imba](https://github.com/imba/imba) (4.1k)

> _**Note**: I will be using the LitElement implementation as the lit-html example as the standard one has been marked with an issue. The overhead should be minimal as it is raw lit-html wrapped in a single web component._

This is a pretty good slice of the current web development ecosystem. While Github Stars aren’t the end-all, there are over 100 libraries in the comparison so I needed criteria to select our contenders.

Each library will be compared in 3 categories: DOM performance, Startup Metrics, and Memory Usage. In addition, I have broken down the frameworks into 4 groups to best compare them with their raw performance peers. However, I will be ranking the libraries on all 3 aspects.

In each group, there will be the reference Vanilla JavaScript entry. This implementation is extremely optimized to the best performance using all the best techniques. It will serve as the baseline of all comparisons.

So let’s get started.

## Group 4 — Standard Performance

This is the biggest group and consists of some of the most popular libraries. Also many of the corporate-backed ones from the likes of Facebook, Google, eBay, and Alibaba. These are the libraries that either has less of a focus on performance or highlight performance in one area but suffer in others.

![](https://miro.medium.com/max/1620/1*hfkTdeFMwMErL3WOt-WQBg.png)

> _There is a lot of red and orange here but keep in mind these libraries on average only around 2x slower than the painfully handcrafted imperative Vanilla JavaScript example we are using here. How different is 400ms from 200ms?_

In raw performance, React is the leader of this pack. Although it never ceases to amaze given how different the architecture, how close React, Marko, Angular, and Ember are here overall. Still, it’s React, and that’s the React Hooks implementation, that is the leader here. For all those pointing extra function creations and holding on to classes, the performance argument is not on your side. [React Hooks are the most performant way to use React](https://blog.usejournal.com/react-hooks-the-most-performant-way-to-write-react-393e135e1cc).

Most libraries here either have naive list sorting which leads to really poor swap row performance or they have expensive creation costs. Ember is an extreme case of this as it has much better update performance than the rest of this group but creation among some of the worst.

The slowest libraries(Knockout, Ractive, and Alpine) are all fine-grained reactive libraries with similar architecture. Knockout and Ractive(also by Rich Harris the author of Svelte) are from the early 2010s before VDOM library dominance. I also doubt Alpine was expecting to ever render 10k rows with its sprinkle of JavaScript approach. We won’t see another pure fine-grained reactive library until much later in the comparison.

Next, we will compare startup metrics a category largely based on the libraries’ bundle size.

![](https://miro.medium.com/max/700/1*3H_SP7iaed0F5rM9RR2otA.png)

The order changes a good amount here. Where Alpine has the worst performance we can see it has the smallest bundle size and quickest startup time of the bunch. Marko(from eBay) is right behind it, followed by Rax(from Alibaba). All 3 of these libraries are built for server-side rendering primarily with lighter client interaction. It’s largely why they are in Group 4 for performance but lead startup here.

The latter half of the table are some of the largest bundles we have in the benchmark ending with Ember that is more than 2x times the size of any other implementation. I don’t know why it takes more than half a megabyte to render this table. But it really hurts startup performance.

The last category we will look at is memory consumption.

![](https://miro.medium.com/max/700/1*aAdVBLu0VV4H-SpItpghEA.png)

Memory tends to reflect the patterns we’ve already seen as it has a big impact on performance and larger libraries tend to use more of it. Alpine, Marko, and React lead the way. The ageing fine-grained reactive libraries use the most leading up to Ember. Ember is just gigantic. It already is using more memory than Vanilla will use throughout the whole suite after rendering just 6 buttons to the page.

### Group 4 Results

In general, this group represents over 300k stars on GitHub and probably the largest portion of NPM downloads, but it’s Marko and Alpine that place the highest on average ranking in this crowd. React is 3rd after them holding its own especially in performance.

This is the group where we have frameworks of titanic proportions and where our old reactive libraries have gone to die. Let’s move on to something a little more optimistic.

## Group 3 — The Performance Conscious

These are the frameworks where you can tell some consideration has been given to performance. They are conscious of size and they’ve found a balance between creation and update costs.

We see a mixed bag of approaches. One Web Assembly framework in Yew (written in Rust), and one Web Component in LitElement. The recently released Vue 3 is a major step forward sees it move out of a Group 4, to compete with a different crop of frameworks than you may have seen in the past.

Without further ado, let’s see how they do.

![](https://miro.medium.com/max/1306/1*OmeVVeC1r5_EAHE8gqlTyw.png)

The scores have jumped a bit and we are seeing a much more even spread. Preact is the highest performer in this group, just inching out LitElement. Vue 3 and Riot are tied in the middle of the pack, both libraries that enjoy being in the middle having a history that includes both reactivity and VDOM. Mithril one of the earliest VDOM libraries to put performance first, and Yew the only WASM library tail.

Performance profile-wise all of these libraries are similar. No pure reactive libraries in this bunch. They all use top-down rendering whether VDOM or simple Tag Template Literal diff. They have a smarter list reconciling than the last group (see swap row performance). However, most still have some of the slowest select row performance.

Yew is the exception but it is slower across the board otherwise. Let’s see if the remaining tests shed any light.

![](https://miro.medium.com/max/655/1*zsGSLpUDUU9SOOyyO7Fq5w.png)

Things have shuffled a little but Preact is still out front when it comes to startup metrics. Yew is the only really large library of this bunch. WASM libraries do tend to be on the larger side.

Again we see a sort of pairing of results. Vue is the largest 2nd to Yew. Preact and Riot are quite compact. Mithril and LitElement are smack in the middle

Preact, the 4kb alternative to React, is definitely the smallest library we’ve seen so far. But the smallest libraries are still to come. Still, no library in this range should be too concerned with their bundle size.

![](https://miro.medium.com/max/655/1*ySIUf76oohSGShv-xo6ZNw.png)

Yew takes the win this time. It has the smallest memory footprint of all frameworks tested. WASM libraries tend to do good here. The others are all pretty close. Mithril and Preact are the worst of the bunch but not by much.

There isn’t too much to pull from here. You might be thinking the LitElement example might be lighter than the rest of the non-Yew libraries as it does not use a Virtual DOM like the rest. But as we will see later VDOM doesn’t mean more memory.

### Group 3 Results

Riot and Preact average the best rankings, followed by LitElement in 3rd. Riot while not standout had no weaknesses in this group and pulled out the win. But it would be hard to be disappointed with any of these Frameworks. With WASM and Web Components, they represent what many feel to be the future of the web.

But we aren’t done yet. The next group represents a different idea of the future of the web. Enter the compilers…

## Group 2 — The Performance Champions

This group is stiff competition. We have the majority of the libraries known as being compiled languages. Each brings its own flavour. We have the immutable structured Elm, the Ruby inspired Imba, and the “disappearing” Svelte.

> _**Note**: It’s been brought to my attention that not everyone is familiar with Svelte’s former “Disappearing Framework” moniker. It describes its ability to basically compile itself out of the output. I am not suggesting Svelte is going anywhere. Sorry for the confusion._

The odd one out is HyperApp, which is completely opposite to the others. No compiler. No templates. Just `h` functions and a minimal Virtual DOM.

Care to guess how this will go?

![](https://miro.medium.com/max/505/1*8FePptPRfusiR51LjhpClQ.png)

Well the minimal Virtual DOM wins. Contrary to recent rhetoric it turns out Virtual DOM isn’t just a recipe for poor performance and compilation didn’t give the other libraries a leg up.

Of the compiled libraries we are actually seeing 3 different approaches to rendering all with about the same average performance. Imba uses DOM reconciliation (closer to LitElement we saw earlier), Elm uses a Virtual DOM, and last Svelte uses a component reactivity system.

You should note the Virtual DOM libraries have the worst select rows as this is where their extra work shows. But these libraries also have faster initial render. If you look closer at the results so far you should notice this shared characteristic between Virtual DOM libraries in contrast to Reactive libraries. But beyond that performance is tight.

So let’s continue. How do our compilers fare on startup time/bundle size?

![](https://miro.medium.com/max/503/1*1X8TxnfLnBg-CyY69xTdVg.png)

Well, as you can see the little Virtual DOM library that could, not only is more performant but smaller than the others. In fact, HyperApp is the smallest implementation among all our libraries. Compilers do not win the day on size.

It along with Svelte both are smaller than our Vanilla JavaScript reference build. How is that possible? The abstractions are written in a way that is more re-usable producing less code. The Vanilla JS implementation is optimized for performance, not size.

Elm is still in competition size-wise in this group. However, Imba is starting to get into the range of some of the group 4 libraries.

Well that leaves memory. One last chance for compilers to shine.

![](https://miro.medium.com/max/506/1*w9-3GAXlu_Fdsaj4P8aZPg.png)

Memory is close, almost a tie but Svelte finally gets a win for compilers. Some sweet revenge on the Virtual DOM shown to be smaller and faster than it.

Honestly, all these libraries have great memory profiles. It should be pretty clear by now the correlation between less memory and better performance.

### Group 2 Results

Don’t believe the hype?

No. More things are a bit more complex than they seem on the surface. Well-engineered systems whether runtime or compile time or regardless of what technical approach they take can be made into performant systems.

HyperApp is the clear winner of this group, with Svelte on its tail, followed by Elm and Imba. With this sort of dedication to performance, you know that these libraries will deliver in most cases, and always show up at the top of benchmarks.

So what’s left?

What if I told you there are declarative JavaScript libraries so confident in their performance they aren’t afraid of raw WASM, web workers, or whatever technique you throw at them. Welcome to…

## Group 1 — The Performance Elite

At one point this might have been referred to as “Blazingly Fast”, and I believe it used to be in one of these libraries’ taglines. If you are keeping track there are only 2 libraries left. In reality, there are a handful of libraries in this category constantly pushing the boundaries. But of the popular ones only 2. They average less than 20% slower than raw hand-optimized Vanilla JS.

![](https://miro.medium.com/max/714/1*KcsfyfZVYnI-WINZo9WAaQ.png)

This is something to look at. Here we have 2 libraries who if looking at their code might be considered siblings but use completely different approaches. Inferno is one of the world’s most performant Virtual DOM libraries. That’s right 3 out of the top 5 performers are Virtual DOM libraries. The slowdown on the select row test can be seen as evidence.

Solid, on the other hand, uses fine-grained reactivity like some of the slowest old libraries in Group 4. Strange place for this technique to re-appear but Solid has addressed their weakness as we can see. Creation time is fast along with update time. The 5% gap from Vanilla JavaScript is uncanny.

What Inferno and Solid have in common strangely enough is JSX templates and React inspired API. Maybe not something you’d expect to find at the pinnacle of performance with all these other libraries with optimized custom DSLs. But as HyperApp showed some things have less impact on performance than one might think.

![](https://miro.medium.com/max/357/1*dIxP-wBTGuM1tcB0ESiFuQ.png)

Solid joins HyperApp and Svelte as the third library with a smaller than the Vanilla JS implementation. But Inferno is no slouch either.

It does seem like while performant libraries are on the smaller side sometimes adding more code can improve performance. Better list reconciling algorithms, more explicit guards, more granular updates.

Inferno might be larger than some libraries in the last couple of groups but it is still a sub 10kb library and it outshines almost all on performance.

![](https://miro.medium.com/max/357/1*i7UEH7r4mkzj4UxbS37VVw.png)

And there it is. Other than Yew and its use of WASM, these are the lowest memory consuming frameworks in the whole competition. It shouldn’t be too surprising given their performance.

This sort of memory consumption numbers reflect very careful consideration of the objects, and closure’s created. A lot of this does come from the custom JSX transformation both libraries do.

This memory performance improvement is especially important for Solid which like most fine-grained reactive libraries trades CPU overhead for memory consumption. Being able to conquer the memory overhead is a large part of how Solid can take a technique mostly similar to some of the slowest libraries in this comparison and make it the fastest.

### Group 1 Results

The sky’s the limit.

…Or well Vanilla JavaScript is. But we have declarative libraries here so close in performance you’d never know the difference. When the DOMs the thing we are up against with careful consideration a number of different techniques can efficiently render the DOM.

And we see it here. Solid takes the performance crown with a technique thought to be old and slow from a decade back, and Inferno shows yet again there is nothing the Virtual DOM can’t do efficiently.

## Conclusion

We have a lot of choices when it comes to building our JavaScript frontends. This is only a quick dip into looking at the performance overhead the framework brings. When it comes to real performance in applications user code has a bigger impact.

But what I really want to impress here is that it is important to test your solutions and understand what the performance looks like. The reality is always going to be different than marketing. Virtual DOMs are not guaranteed to be slow. Compilers aren’t guaranteed to produce the smallest bundles. Custom template DSLs aren’t guaranteed to be most optimal.

Finally, I will leave you with the full tables showing all libraries together. Just because a library is towards the end doesn’t necessarily mean it is slow but that it scored worse compared to these stiff competitors.
