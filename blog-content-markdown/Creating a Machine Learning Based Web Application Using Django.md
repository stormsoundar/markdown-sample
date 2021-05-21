This article is for readers who want to deploy their Machine Learning model as a Web Application using Python’s Django framework. For many Data Science and Machine Learning enthusiasts, this could be a good reference for converting their simple .py model files into a much more dynamic and powerful web application that can accept inputs from a user and generate a prediction.

For the sake of simplicity of the article, we will be focusing more on how to create a ML based web application rather than spending the majority of time in solving a tough Machine Learning problem. Also, this is not a complete django tutorial but a step in the direction towards becoming an AI/ML developer.

First, we will be creating a simple ML model using the famous titanic dataset and then convert it into a fully functional, dynamic web based application.

### Developing Machine Learning Model

As mentioned above, we will be working on the famous titanic dataset that is used to predict the survival of the passengers based on their different attributes (Passenger class, Sex, Age, Fare etc.). Since this a binary classification problem, we would be using Logistic Regression algorithm (a powerful yet simple machine learning algorithm) for creating our model.

The below code can be referred to create the classification model.

> _Note: Creating a Machine Learning Model is a complete detailed task in itself that contains multiple steps (such as Data Pre-processing, EDA, Feature Engineering, Model Development, Model Validation, Tuning Hyperparameters). Here we are skipping all those steps since we are much more interested in using this model for powering our web based application._

```python
import pandas as pd
import numpy as np

from sklearn.model_selection import train_test_split
from sklearn.preprocessing import MinMaxScaler
from sklearn.linear_model import LogisticRegression

from sklearn.metrics import classification_report, confusion_matrix, accuracy_score

# load dataset
dataset = pd.read_csv('train.csv', encoding='latin-1')
dataset = dataset.rename(columns=lambda x: x.strip().lower())
dataset.head()

# cleaning missing values
dataset = dataset[['pclass', 'sex', 'age', 'sibsp', 'parch', 'fare', 'embarked', 'survived']]
dataset['sex'] = dataset['sex'].map({'male':0, 'female':1})
dataset['age'] = pd.to_numeric(dataset['age'], errors='coerce')
dataset['age'] = dataset['age'].fillna(np.mean(dataset['age']))

# dummy variables
embarked_dummies = pd.get_dummies(dataset['embarked'])
dataset = pd.concat([dataset, embarked_dummies], axis=1)
dataset = dataset.drop(['embarked'], axis=1)

X = dataset.drop(['survived'], axis=1)
y = dataset['survived']

# scaling features
sc = MinMaxScaler(feature_range=(0,1))
X_scaled = sc.fit_transform(X)

# model fit
log_model = LogisticRegression(C=1)
log_model.fit(X_scaled, y)

# saving model as a pickle
import pickle
pickle.dump(log_model,open("titanic_survival_ml_model.sav", "wb"))
pickle.dump(sc, open("scaler.sav", "wb"))
```

Once we have created our final model, we are simply storing this model as a **_model.sav_** file (using pickle). Also, notice we are storing our scaler object as a **_scaler.sav_** file (since we will be using this same scaling object for transforming our user input to the same level as training data before feeding it to the model).

## Creating our own Django Project

Before creating our django project, first we need to install django into our development machine using the below command.

> **_pip install django_**

This will automatically install all the dependencies that the django framework required in order to function properly. If this command is not working for you, please refer to the below link-

Once you have installed the django, you need to open command prompt on your machine (terminal on Ubuntu or Mac) and follow the below steps-

1. **cd Desktop** (you can traverse to whichever location you want to create your project folder)

2. **mkdir Titanic_Survial_Prediction** (you can choose whatever name you want to use for your web application)

3. **cd Titanic_Survial_Prediction** (cd into the folder you just created in the step above)

4. **django-admin startproject Titanic_Survial_Prediction_Web** (this command will automatically create a django project with name _Titanic_Survial_Prediction_Web_ inside your main project folder)

5. **cd Titanic_Survial_Prediction_Web** (cd into the django project created in the step above)

6. Now if you followed the above steps correctly, just type in **python `manage.py` runserver** into your command prompt screen and then copy the link that appear on the command prompt ([http://127.0.0.1:8000](http://127.0.0.1:8000)) into a web browser. As soon as you hit enter, you would be able to see the below screen

![](https://miro.medium.com/max/700/1*fH8bfS0HW1-weF5B4TxFTg.png)

If the above web page also appears for you, then you have successfully completed the first step in creating your own django web application.

If for some people this web page is not appearing, please refer to the above steps once again and see if in case you missed anything.

Now using any IDE (PyCharm, Atom etc.) open your django project (**Titanic_Survial_Prediction_Web**)

![](https://miro.medium.com/max/591/1*thVRCcuKungkn1jq7Ho1mQ.png)

Now inside the same folder where your `urls.py` is present, create a new python file with name **‘`views.py’`**. This views file would be responsible for getting the input from any user using a web page, using the same input for generating predictions and then showing this prediction back to the user using a web page.

> Also, we need to move our ‘model.sav’ and ‘scaler.sav’ file inside the same folder as `urls.py`.

Before moving any further, lets first complete the configuration for our django project. Inside our main project folder, create an empty folder by the name _‘templates’_ (you can use any name but its recommended to follow a defined framework naming convention). This folder will be holding all out html files that we would be using in our project.

Now open the `setting.py` file and add ‘templates’ (or whatever name you have given to your html files folder) to the highlighted ‘DIRS’ list inside the ‘TEMPLATES’ list.

![](https://miro.medium.com/max/3100/1*Nr_xO3AeOiWGVdK84zMKeg.png)

Now inside the `urls.py` file add the below code for configuring the urls for our website (different urls for different web pages).

```python
# default present
from django.contrib import admin
from django.urls import path

# add this to import our views file
from Titanic_Survial_Prediction_Web import views

urlpatterns = [
    path('admin/', admin.site.urls),

    # add these to configure our home page (default view) and result web page
    path('', views.home, name='home'),
    path('result/', views.result, name='result'),
]
```

We have first imported our views file inside the `urls.py` file and then inside the urlpatterns list added path for our home page (the default page whenever we open our web application instead of django default view) and the result page (for displaying the results to user) along with there respective names (using which they can be referred inside our html files).

Now we need to define these two functions inside our `views.py` file. Along with these two, we will also create a getPredictions function that would be able to generate predictions by loading our pre-trained model.

The below code can be referred to do this task.

```python
from django.shortcuts import render

# our home page view
def home(request):
    return render(request, 'index.html')


# custom method for generating predictions
def getPredictions(pclass, sex, age, sibsp, parch, fare, C, Q, S):
    import pickle
    model = pickle.load(open("titanic_survival_ml_model.sav", "rb"))
    scaled = pickle.load(open("scaler.sav", "rb"))
    prediction = model.predict(sc.transform([[pclass, sex, age, sibsp, parch, fare, C, Q, S]]))

    if prediction == 0:
        return "not survived"
    elif prediction == 1:
        return "survived"
    else:
        return "error"


# our result page view
def result(request):
    pclass = int(request.GET['pclass'])
    sex = int(request.GET['sex'])
    age = int(request.GET['age'])
    sibsp = int(request.GET['sibsp'])
    parch = int(request.GET['parch'])
    fare = int(request.GET['fare'])
    embC = int(request.GET['embC'])
    embQ = int(request.GET['embQ'])
    embS = int(request.GET['embS'])

    result = getPredictions(pclass, sex, age, sibsp, parch, fare, embC, embQ, embS)

    return render(request, 'result.html', {'result':result})


```

the result method is responsible for collecting all the information that the user have entered and then return the result as a key-value pair.

> Note: It is always recommended to create getPredictions method inside a separate python file and then import it inside our `views.py` file. Here for keeping things simple and straight forward, we have made the function inside the `views.py` file.

Now that we have worked on the backend logic, lets create our web pages to accept the input from user through forms and display results.

Inside the **_“templates”_** folder, create an index.html file. This index.html would be our home page and will have our form that we will use for getting input from our user.

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8" />
    <title>Home page</title>
  </head>
  <body>
    <h1>Titanic survival prediction</h1>
    <form action="{% urls 'result' %}">
      {% csrf_token %}

      <p>Passenger Class:</p>
      <input type="text" name="pclass" />
      <br />

      <p>Sex:</p>
      <input type="text" name="sex" />
      <br />

      <p>Age:</p>
      <input type="text" name="age" />
      <br />

      <p>Sibsp:</p>
      <input type="text" name="sibsp" />
      <br />

      <p>Parch:</p>
      <input type="text" name="parch" />
      <br />

      <p>Fare:</p>
      <input type="text" name="fare" />
      <br />

      <p>Embark Category C:</p>
      <input type="text" name="embC" />
      <br />

      <p>Embark Category Q:</p>
      <input type="text" name="embQ" />
      <br />

      <p>Embark Category S:</p>
      <input type="text" name="embS" />
      <br />

      <input type="submit" value="Get Predictions" />
    </form>
  </body>
</html>
```

There are couple of things we need to understand here before moving forward.

1. Forms action parameter → View that collects the form data, generate predictions using this data and then redirects to the webpage with result. For django we have a specific way of doing this- **_action = “{% urls ‘name_of_url’ %}”_**. In our case the name of our url is result (as we have already provided this in our `urls.py` file).

2. Also, along with this we need to provide a **_{% csrf_token %}_** (i.e. cross site reference forgery token) which makes data handling in django more secure. This part is mandatory and hence it should always be the first line inside your form tags.

> _Note: **{% .. %}** are called scripting tags._

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8" />
    <title>Result</title>
  </head>
  <body>
    <h1>Prediction</h1>

    {{ result }}
  </body>
</html>
```

Inside the **“{{ }}”** tags, we specify the name of the key which contains our result (in our case it is result).

Once you have completed all these steps, press CTRL + C in your command prompt to stop the server and then restart it using “python manage.py runserver” command. Now reopen the link in your browser. This time the default page would be the index.html webpage which have our input form.

Once you fill in the respective data and submit the form, you will be redirected to the result.html page which will display the prediction for your input data.

So this is how, we create a simple yet powerful ML based django web application that is dynamic in nature (accepts input from user) and make predictions based on Machine Learning model.

If you want to learn more about django, please refer to the [django documentation](https://www.djangoproject.com/).
