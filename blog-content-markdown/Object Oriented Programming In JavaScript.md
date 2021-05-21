Object Oriented Programming, or OOP, is one of those concepts which helps you boost your knowledge of programming. Many people are still confused about what it is, and I hope to cover the concepts of OOP in this article.

![](https://miro.medium.com/max/2560/0*wgDCQoZtprrPg272.jpg)

### What Is OOP?

**OOP** is a **convention of writing code**. It is a way for programmers to write their code in a really organized manner. When working on big applications, it is inevitable that your code will get really messy and hard to maintain. In such situations, implementing the concepts of OOP can really help you. Because of the way **object-oriented programming** is designed, it helps the developer by allowing for code to be easily reused by other parts of the program or even by other people.

As Linus Torvalds says:

_“Anyone can write code that a machine can understand. Good programmers write code that humans can understand”._

Also, most of the programming languages do support OOP, for example python, Java, c++, and more. If ever in future you plan to switch to another language, you will see that it becomes really easy to learn it because you already knew the principles of OOP.

### Prerequisites To OOP:

- Basic Knowledge of a programming language, in our case — JS.

### Why Should I Learn About OOP?

Object Oriented Programming encourages organization of code. It is really easy to maintain, and also makes your code reusable. As you all know, programmers are quite lazy, and having the ability to reuse a piece of code is phenomenal for you. One of the biggest reasons to learn and implement OOP is because every large scale application uses it. If you plan to work in those big tech companies like Facebook or Google, Then you need to learn how to write object oriented programs. Those companies have BILLIONS of lines of code that they have to maintain, and their way to do this is through OOP.

Now, let us have a look at some of the fundamental concepts of OOP.

### Fundamentals Of OOP

#### Objects:

Objects are basically containers of data and methods. They follow a key:value pair, something similar like a hash map.

Let us make a user object, with a first name and a last name.

```javascript
let user = {
  firstName: "David",
  lastName: "Smith",
};
```

Let us print out the first name of the user.

```javascript
console.log(user.firstName);
```

What we are doing here is calling the firstName attribute from the user object. But there is another way to do this.

```javascript
console.log(user["firstName"]);
```

You might be wondering when to use which. The . notation is quite simple to read, although on the other hand, the bracket notation allows you to pass in 2 worded attributes. Lets say for example you have a “Side Hustles” attribute of the object as such:

```javascript
let user = {
  firstName: "David",
  lastName: "Smith",
  "Side Hustles": ["blogging", "YouTube"],
};
```

Well, Using the . notation will give you an error if you try to log it in the console. On the other hand, the bracket notation will not give you any errors. Let us look at the syntax:

```javascript
console.log(user.Side Hustles); //gives an error
console.log(user["Side Hustles"]); //returns blogging and //boxing.
```

Next, Lets create a function inside of the user object, that prints out the full name of the user.

```javascript
let user = {
  firstName: "David",
  lastName: "Smith",
  fullName: (user) => {
    return `${user.firstName} ${user.lastName}`;
  },
};
console.log(user.fullName(user));
```

What you might find weird here is that we are passing in a parameter called “user” while making the “fullName” function. This is because we need an instance of the object inside of the function, as we are using an attribute that is inside of the object. Same thing while printing out the full name.

Let us check if an attribute exists inside of an object or not.

```javascript
if ("firstName" in user) {
  console.log("first name exists");
} //we get "first name exists" in the console.
```

Let us see how we loop through an object.

```javascript
for (let prop in user) {
  console.log(user[prop]);
} //returns all the keys inside of the user object
```

In the above example, you are seeing that we are passing a variable inside of the square brackets. This is not possible in the . notation.

Let us now have a look at one of the key concepts of OOP and that is classes.

#### Classes:

![](https://miro.medium.com/max/279/0*6_Po0H045sIQFL0A)

Classes are basically templates for creating objects, providing initial values, and implementing methods. In a nutshell, that is the purpose of classes. Let us now look at some real life examples of classes and their implementation.

In the example which we had taken before, we saw that there was a user object with a function inside of it, that basically printed out the full name of the user. Let us try to make a class out of it and see what happens.

To define a class, we basically say:

```javascript
class User {
  //your code here...
}
```

What you might have noticed is that we capitalized the first letter of the class name. This is a common JavaScript convention. Inside of this class, we need to have something called a **constructor** method. A constructor method basically initializes the values of the class. In a class, the first thing that is called is a constructor.

We define a constructor as such:

```javascript
class User {
  constructor() {
    //constructor
  }
}
```

Inside of the (), we pass the parameters that we want to initialize. Let us pass in the following parameters:

```javascript
class User {
  constructor(firstName, lastName, sideHustles) {
    //constructor
  }
}
```

To actually initialize the parameters, we use the “this” keyword. The “this” keyword basically tells that the constructed parameter is inside of the current class. It tells the compiler that the current constructed keywords are in an instance of the class. Let me show you how to implement the “this” keyword.

```javascript
class User {
  constructor(firstName, lastName, sideHustles) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.sideHustles = sideHustles;
  }
}
```

But if you remember from what I have said earlier, you know that the class is just a template. It is not an object **yet**. So, to make it an object, we need to make an instance of it.

Let us do that by making a new instance of the class User. We will call this “David”.

```javascript
let david = new User("David", "Smith", ["YouTube", "blogging"]);
```

Now these parameters are going to go inside of the constructor and then assign the values of firstName to David, lastName to Smith, and side hustles to the side hustles.

Now let us add a function called fullName which returns the full name of the user.

```javascript
fullName = () => {
  return `${this.firstName} ${this.lastName}`;
};
```

So, we are replacing the user with “this”, pointing to the current class.

#### Conclusion

I hope that you were able to understand the basic concept of OOP. Learning OOP is a must for every programmer, especially if you want to get a job at a big tech company.

In my opinion, there are a lot more concepts to cover for OOPs, and these were some of the concepts which are really important for your development journey.

And If you want to master JavaScript, and become a boss at being extremely effective as a developer… make sure to check out our free training 👉 [HERE](https://event.webinarjam.com/register/1/0z8y0u8?utm_source=organic&utm_medium=medium&utm_content=oop-pri-javascript&utm_campaign=medium-blogs).

Thank you!

Priyanshu Saraf.

If you have any recommendations for blogs and content, do let me know in the comment section down below!
