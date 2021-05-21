We all know that react is extremely popular, and there is a good reason for that. React is one of the easiest libraries for front-end, and of course, it’s extremely flexible. Let’s look at some of the core fundamental concepts of react that you need to know so that you can create any project by yourself.

Why React?

React has several features that make it awesome, and one of them is the virtual DOM. React keeps a copy of whatever goes on in the real DOM, and updates the virtual DOM if needed so that your page itself doesn’t refresh if you update any part of the app through state.

What is JSX?

One of the hardest parts to understand about react as a beginner, I’m sure that you will be confused after looking at JSX for the first time.

Simply put, JSX is HTML and JavaScript combined together.

Something like this:

```javascript
import React from "react";

function Basics() {
  //traditional javascript code
  return (
    //jsx starts here
    <div>hello world!</div>
  );
}

export default Basics;
```

The best part about JSX is that you can insert javascript anywhere you want. Something like this:

```javascript
import React from "react";

function Basics() {
  return <div>hello world! 2 + 2 is {2 + 2}</div>;
}

export default Basics;
```

What is the difference between functional and class-based components?

In react, there are 2 types of components: functional-based and class-based. In the primitive versions of react, functional-based components were not supported. After some time, functional components did step in, but we still didn’t have most of the functionality of class-based components in functional components. Hence, developers were reluctant to enter in functional components. Although, from version 16.8 and onwards, react started supporting hooks.

What are hooks in React?

Hooks were introduced in react to provide the functionality that class-based components had, in functional components. Because of hooks, we now have the ability to use state, side effects, ref, and a lot more in functional components. I highly recommend you to check out our “[5 hooks in react](https://medium.com/sonny-sangha/5-react-hooks-that-you-will-use-in-your-everyday-life-as-a-developer-7c1e99780190)” article, to get an idea about the top 5 hooks that you can use in react.

Hooks generally begin with the “use” term and are imported from react like so:

```javascript
import React, { useState } from "react";
```

There are a few rules about hooks that you need to know:

- Only call Hooks **at the top level**. Don’t call Hooks inside loops, conditions, or nested functions.

- Only call Hooks **from React function components**.

What is State in React?

I’ve mentioned state before, although what does it actually mean?

State can be simply understood as variables in react. React doesn’t support the traditional variable creation, rather we have state that we create that stores the data which we want to pass in.

Let’s take an example of a counter in react.

```javascript
import React from "react";

function Basics() {
  let counter = 0;
  function add() {
    return counter + 1;
  }
  function subtract() {
    return counter - 1;
  }
  return (
    <div>
      <button onClick={add}>+</button>
      {counter}
      <button onClick={subtract}>-</button>
    </div>
  );
}

export default Basics;
```

This is what you’d expect to be happening, but that’s not correct. This won’t work at all, the reason being that the use of variables like this is not allowed in react.

Rather, what you should be doing is the following:

```javascript
import React, { useState } from "react";

function Basics() {
  const [counter, setCounter] = useState(0);

  return (
    <div>
      <button onClick={setCounter(counter + 1)}>+</button>
      {counter}
      <button onClick={setCounter(counter - 1)}>-</button>
    </div>
  );
}

export default Basics;
```

As you can see, we are using the useState hook in react. What we do here is that we define the hook at the beginning of the “Basics” function, and we destructure two values from it. We have a “counter” variable that actually stores the value and a setter function that lets us set the value of the counter somewhere else in the application.

Whenever the button is clicked, we are updating the value of the counter, and then we are displaying the counter just below using the JSX syntax.

What are side-effects in react?

Side effects in react are one of the most useful features that react provides us with. Let’s take an example that you want to fetch data from an API. How are you going to do that? You cannot really use the async-await syntax in your code without making it messy, and also, how do you do something specific when some part of your app changes?

That’s when we use side-effects.

The useEffect hook is what you’ll end up using if you’re using react functional components. This hook lets us do a bunch of things.

This is the basic example of a useEffect hook:

```javascript
import React, { useEffect } from "react";
function Basics() {
  useEffect(() => {}, []);
  return <div></div>;
}
export default Basics;
```

As you can see, there is a useEffect hook that we import from react, and then, when we define the hook, we give it two parameters: A function, and empty square brackets.

Inside the function, we enter whatever code we want to be executed, for example fetching data from an API, and in the square brackets, we enter the dependency for the useEffect. Whenever the dependency updates, the useEffect runs. If you keep the brackets empty, then it will run when the page itself is reloaded, and not again.

There is a lot more to useEffect as well, which might require a separate article. Let me know in the comments below if you would like to see that!

If you liked this article then make sure to tell me in the comments down below. Also, check out Sonny’s [YouTube channel](https://www.youtube.com/user/ssangha32) for more React content. Lastly, if you like my content then don’t hesitate to buy me a coffee!
