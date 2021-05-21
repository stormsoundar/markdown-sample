As we all know, React is a really powerful front-end library. To get a good idea about react, it’s best to create applications and see how different hooks interact with each other. Let me give you a sneak peek related to what we will be building in this tutorial:

![](https://miro.medium.com/max/4368/1*Xgyb09qQWBVQkzJDK7iLFQ.png)

As you can see, this weather app is custom-made, fetches an image from the Unsplash API, and also provides the current temperature of any given location. I’m sure that you’re extremely excited to build this application. So let’s get started now.

Before anything else, you need to have node js installed. To install node, please go to [nodejs.org](https://nodejs.org/en/) and install the latest stable version on your PC.

With that done, head off to your terminal and enter the following command in any folder that you want the weather app to be:

```javascript
npx create-react-app weather-app-react
```

I’m going to create the application on my desktop, and I will call it weather-app-react.

The above command will simply create a react app for us that we can start working with immediately. After that is done, open up vs code in the terminal by using the command:

```javascript
code .
```

For those of you who are using vs code insiders, use:

```javascript
code-insiders .
```

This opens up vs code in the current directory.

Now, remove all the code in App.js, remove the setupTest.js, App.test.js, logo.svg, and then enter the following code in App.js:

```javascript
import React from "react";
import "./App.css";
function App() {
  return <div>Let's create a weather app!!</div>;
}
export default App;
```

And then go to your terminal and enter the following command:

```javascript
npm start
```

This spins up the react application for us, which is exactly what we want.

You should see something like this:

![](https://miro.medium.com/max/576/1*jaudQyYfXy2r2FXPWj3Mlg.png)

If you got till here, perfect. Let’s keep moving.

Proceed to [OpenWeatherMap](https://openweathermap.org/api) and create a free account for the current weather data API. Then, you will get an API key.

Also, go to the API of [Unsplash](https://unsplash.com/developers) and sign up for a free developer account. You will be prompted to create a project, name it however you wish to, and then you will again get an API key.

Unsplash will provide us with the images that we need and open weather map will provide us with the weather data.

With all this set, head back to your vs code and import useState and useEffect from react.

```javascript
import React, { useState, useEffect } from "react";
```

If you do not know much about useState and useEffect, I highly recommend checking out our hooks tutorial here.

Now, we want to create three pieces of state for our application like so:

```javascript
import React, { useState, useEffect } from "react";
import "./App.css";
function App() {
  const [weather, setWeather] = useState({});
  const [locations, setLocations] = useState("london");
  const [photos, setPhotos] = useState([]);
  return <div>Let's create a weather app!!</div>;
}
export default App;
```

The weather state will store the response that we get from the weather API, the locations state will store the response that we get from the user’s input, and the photos state will store the response that we get from the unsplash API.

You might wonder why we’ve given an initial state of “london” to the locations API. This is because we don’t want the user to see a blank screen upon the site’s load, as it results in a bad user experience.

The next thing that we want to do is setup a function that fetches data from the apis like so:

```javascript
import React, { useState, useEffect } from "react";
import "./App.css";
function App() {
  const [weather, setWeather] = useState({});
  const [locations, setLocations] = useState("london");
  const [photos, setPhotos] = useState([]);
  function ifClicked() {
    fetch(
      `http://api.openweathermap.org/data/2.5/weather?q=${locations}&APPID={API_KEY_FOR_WEATHER_API}&units=metric`
    )
      .then((res) => {
        if (res.ok) {
          console.log(res.status);
          return res.json();
        } else {
          if (res.status === 404) {
            return alert("Oops, there seems to be an error!(wrong location)");
          }
          alert("Oops, there seems to be an error!");
          throw new Error("You have an error");
        }
      })
      .then((object) => {
        setWeather(object);
        console.log(weather);
      })
      .catch((error) => console.log(error));
    fetch(
      `https://api.unsplash.com/search/photos?query=${locations}&client_id={API_KEY_FOR_UNSPLASH}`
    )
      .then((res) => {
        if (res.ok) {
          return res.json();
        } else {
          throw new Error("You made a mistake");
        }
      })
      .then((data) => {
        console.log(data);
        setPhotos(data?.results[0]?.urls?.raw);
      })
      .catch((error) => console.log(error));
  }
  return <div>Let's create a weather app!!</div>;
}

export default App;
```

What we are trying to do here is simply fetch data from the APIs, check if there is any problem via the catch and if-else statements, and also set the data that we will be needing to our pieces of state.

In the first API call, we are giving it the parameter of units=metric which might confuse some of you. Traditionally, we get all the data back in scientific terms (kelvin for ex-) which no one really likes to see. With this parameter in the Fetched URL, we get back the response in a format that is easy to view and understand. Then, we simply set the object response that we get back from the API to the piece of state that we discussed earlier.

While fetching data from the Unsplash API, what we really want is just the link of the first image that comes up. We enter the parameter of the query to be the location that the user enters, and we assume that the first result (hence the 0) will be optimized. If you want to, you can make this a lot more complex, but that’s not what I am aiming for here. When you log the output of both the responses, you will find a bunch of data that you can play around with. For the sake of simplicity, I have chosen the link that we get from the Unsplash API although you can take it to the next level.

Now it’s time for us to actually develop the UI of this application. This is the final piece of code that we need for the App.js file:

```javascript
import React, { useState, useEffect } from "react";

import "./App.css";

function App() {
  const [weather, setWeather] = useState({});
  const [locations, setLocations] = useState("london");
  const [photos, setPhotos] = useState([]);
  useEffect(() => {
    ifClicked();
  }, []);

  function ifClicked() {
    fetch(
      `http://api.openweathermap.org/data/2.5/weather?q=${locations}&APPID={APP_ID}&units=metric`
    )
      .then((res) => {
        if (res.ok) {
          console.log(res.status);
          return res.json();
        } else {
          if (res.status === 404) {
            return alert("Oops, there seems to be an error!(wrong location)");
          }
          alert("Oops, there seems to be an error!");
          throw new Error("You have an error");
        }
      })
      .then((object) => {
        setWeather(object);
        console.log(weather);
      })
      .catch((error) => console.log(error));
    fetch(
      `https://api.unsplash.com/search/photos?query=${locations}&client_id={APP_ID}`
    )
      .then((res) => {
        if (res.ok) {
          return res.json();
        } else {
          throw new Error("You made a mistake");
        }
      })
      .then((data) => {
        console.log(data);
        setPhotos(data?.results[0]?.urls?.raw);
      })
      .catch((error) => console.log(error));
  }
  return (
    <div className="app">
      <div className="wrapper">
        <div className="search">
          <input
            type="text"
            value={locations}
            onChange={(e) => setLocations(e.target.value)}
            placeholder="Enter location"
            className="location_input"
          />
          <button className="location_searcher" onClick={ifClicked}>
            Search Location
          </button>
        </div>
        <div className="app__data">
          <p className="temp">Current Temparature: {weather?.main?.temp}</p>
        </div>
        <img className="app__image" src={photos} alt="" />
      </div>
    </div>
  );
}

export default App;
```

We took the data that we got from the weather API, stored it into a piece of state, and then displayed it on the front-end. All that’s left is for us to now develop the CSS for this. We are going to be using a little bit of glassmorphism as well, which is really going to level up the application design.

Here is the CSS code for the app:

```javascript
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400&display=swap");
.app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: url(./background.jpg);
  height: 100vh;
  width: 100vw;
}

.wrapper {
  padding: 25px;
  border: 0.1px solid rgb(96, 96, 96);
  display: flex;
  flex-direction: column;
  width: 40%;
  height: auto;
  background: rgba(219, 216, 216, 0.25);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(4px);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.18);
}

.app__image {
  margin: 24px;
  max-height: 500px;
  width: auto;
  border-radius: 10px;
}
.location_searcher {
  padding: 8px;
  color: white;
  background-color: #333;
  border: none;
  border-radius: 5px;
  margin-left: 5px;
}
.location_input {
  padding: 8px;
  border: 1px solid rgba(255, 255, 255, 0.6);
  color: #333;
  border-radius: 5px;
}

.temp {
  margin: 5px;
  font-family: Poppins;
  color: #2e2e2e;
}
```

I’m using the Poppins font from Google fonts but feel free to use any font that you desire. In glassmorphism, we give the background element a blur filter so that we get a beautiful component design. That’s why we use the backdrop-filter: blur(10px) CSS code here.

With that said, we finally have the application developed. It was pretty simple, wasn’t it?

I hope that all of you enjoyed developing this application along with me. I left this application open-ended, and I want you to provide your own code to it and customize it however you can.

For more complex apps be sure to check out Sonny’s [YouTube channel](https://www.youtube.com/user/ssangha32). Thank you for reading the article and don’t hesitate to buy me a coffee if you like the content!
