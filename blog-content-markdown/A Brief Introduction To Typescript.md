Typescript is one of the emerging languages, and over 25.7% of the developers showed love for typescript this year. Let's discuss why it might be important for you and why should you pick it up in 2020 too.

The main reason why several developers refrained from using JavaScript earlier was that JavaScript could not deal with types of variables. It did not warn you about the errors in your code and hence, the bugs could not be identified in the development time, rather we would have to figure it out by going through tons of files if our projects were bigger. This can become a huge problem and a waste of time, and this is exactly where typescript comes in.

#### What Is Typescript?

Typescript is a superset of JavaScript, so that means that all the code written in JavaScript is valid in typescript, although there is a lot more to it. This illustration might help you:

![](https://miro.medium.com/max/310/1*ifqCyFzVECvKGs77MwYaBQ.png)

#### Why Should You Use Typescript?

You can now make strongly typed variables and you can now see your errors when you are developing the application. Now, it is not a compulsion to declare the types of a variable in typescript, although I would highly recommend you to use it because it makes errors WAY more predictable. For example, this code snippet shows exactly how to make strong types in typescript.

```javascript
const name: string = "John";
const score1: number = 50;
const score2: number = 42.5;
```

The next thing which typescript lets us do is Object-Oriented Programming. Although JavaScript did let us do a little bit of Object-Oriented Programming, developers wanted more. Typescript fills this gap too. With typescript, you can create classes, interfaces, constructors, access modifiers like public and private, fields, properties, generics, and more.

As you can probably guess, typescript is really powerful. Although, the setup for typescript can get difficult. To fix this problem, the react team made a template for it.

The question that you might ask is:

_Why is it hard to setup typescript?_

That is a really good question. The main reason why JavaScript is used so widely is that it is the only language that can run in the browser. Not even typescript can run directly in the browser, and hence, we need a way to change typescript code to JavaScript. This is done through a **transpiler**.

Before moving forward, let us just install typescript along with react like so:

```javascript
npx create-react-app my-app --template typescript
```

This is gonna take some time to set up, but once it does, you will have react running on typescript instead of plain JavaScript.

This will help you when you are working on some project, but covering all of react with typescript is out of the scope for this article.

Moving on, let us try to have a look at how typescript works and change a file from plain JavaScript to typescript.

In the code given below, we will be adding 2 numbers using a function, but we will be passing a string as an argument instead of a number.

```javascript
function add(a, b) {
  return a + b;
}
add(2, "hello"); // 2hello is returned
```

As you can see, the code above returns an output that we did not expect. This is going to create some trouble, and if you have files with hundreds of lines of code, then it won’t be possible for you to figure such minute mistakes out in a few minutes. Here, typescript is going to come in and tell us where we are wrong in the development environment itself, like so:

```javascript
function add(a: number, b: number) {
  return a + b;
}
add("Hello", "World"); //gives a type error immediately
```

As you can see, in the above-written code, you will face a type error telling you that this code is not valid as you declared the type of the arguments to be “numbers”. This helps you avoid some serious bugs in your code.

#### Final Words

I am pretty sure you were able to guess that typescript is very powerful and can help you in a ton of hard situations, and it makes life very easy as a developer. This was just an introduction to typescript. It can help you create so many things, and some modern technologies like graph QL and react already support typescript and smoothly run along with it.

#### What is the catch?

The only problem is that if I were to go and make a website along with people who do not already use typescript, it is going to be bad work for me. This creates some chaos and haphazard situations. My take on this would be to only use it if you have to, in a production environment. You can definitely explore more about it with your side projects, but if it ain’t broke, don’t change it. Do not use typescript and make your application more complex than it already is.

I hope that you found value in this article. Make sure to tell me what you think in the comment section down below.

Thank you!

Priyanshu Saraf
