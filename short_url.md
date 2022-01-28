# Build and Deploy a URL Shortener using NodeJS, Firebase, and Heroku

**Uniform Resource Locator** (URL), also referred to as web address, is a specific type of [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) that specifies the address of  web resources on a computer network.

A URL comprises a Scheme (`https://`), a Host-Name (`dev.to`), and a Path (`/divofred`). Sometimes, a URL can contain Parameters and Anchors.

All these put together make the URL long and not decent. This calls for a URL Shortener.

As the name implies, URL Shortener helps to reduce the length of the web address while retaining the address to the web resources of the URL.

## Goal

In this tutorial, you will learn how to shorten URLs, store these short URLs in Firebase, and deploy the Application to Heroku.  

## Prerequisites

- Knowledge in NodeJS and ES6 Syntax
- A Text Editor preferably Visual Studio code.
- Firebase Account

## Getting Started

Here is the link to the [GitHub repository](https://github.com/divofred/short-url).

> Please note that this tutorial is a modification of [Traversy's YouTube Video](https://www.youtube.com/watch?v=Z57566JBaZQ)

Run `npm init` in your terminal to initialize our project.

Install the dependencies that will be used in this tutorial:

```bash
npm i express firebase shortid valid-url nodemon
```

`firebase` connects the application to firebase database.

`shortid`  creates the `short-url` code.

`valid-url` validates the URLs that are sent to the API.

### Setting up Firebase

> To get started with firebase, you need to own a [Gmail](https://gmail.com).

#### Creating a Project

 Navigate to [Firebase Official Website](https://console.firebase.google.com) and create a project.

![firebase_Homepage](https://github.com/divofred/Ata/blob/master/assests/images/firebase_Homepage.jpg)

Choose a name for the project:

![project_name](https://github.com/divofred/Ata/blob/master/assests/images/project_name.jpg)

Google Analytics would not be necessary for our project, toggle it off and create a project.

![Google Analytics](https://github.com/divofred/Ata/blob/master/assests/images/Google%20Analytics.jpg)

After successfully creating a new project, you will be brought to the screen below. When you click  on continue, you will be redirected to your dashboard.

![click_continue](https://github.com/divofred/Ata/blob/master/assests/images/click_continue.jpg)

#### Getting our Projects Details

You need the `Web API key` from your dashboard to connect the application to Firebase. 

On the sidebar, click on `Authentication` and `Get Started`. This auto-generates your `Web API key`.

Click on the `settings icon` on the sidebar and click on `Project settings` to get the projects details.

![Projects_details](https://github.com/divofred/Ata/blob/master/assests/images/Projects_details.jpg)

### Setting up the server

The next step is to set up an `express` listening event that will listen for a connection to port `5000`.

Create a new file named `server.js` and open it:

```js
const express = require("express");
const app = express();

app.use(express.json());

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => console.log("Sever running at port " + PORT));
```

## Shortening Links

Create two files, one for generating the `URL Code` and the other for redirecting the shortened URL to the long URL.

Create a new folder named `routes` and add two new files named `index.js` and `url.js`.

Open up `url.js` to create the short URL:

```js
const express = require("express");
const router = express.Router();

const valid_url = require("valid-url");
const shortid = require("shortid");

const Shorten = async (req, res) => {
  const { longURL } = req.body;
  const baseURL = "http://localhost:5000";

  // Check base URL
  if (!valid_url.isUri(baseURL)) {
    return res
      .status(401)
      .json({ success: false, message: "Invalid base URL" });
  }
  const urlCode = shortid.generate();
}

router.post("/shorten", Shorten);
```

This will get the `longURL` from the body and specify the `baseURL` of the application which is `http://localhost:5000`.

The package `valid-url` is used to check if the `baseURL` specified is valid. If it is not valid, it will send a status code of `401` else, it will randomly generate a `urlCode` using the `shortid` package.

#### Connecting our application to Firebase Database

In your dashboard, click on `Realtime Database` and create a new `web database`. 

> To create a new web database, click on an icon that looks like `</>` 

Create a new folder in our project's root directory named `firebase` and add `fb_connect.js` to it.

Open `fb_connect.js` to connect to the Database:

```js
const { initializeApp } = require("firebase/app");

const app = initializeApp({
  apiKey: "",
  authDomain: "projectID.firebaseapp.com",
  //authDomain: "short-url-4e0b5.firebaseapp.com",
  projectId: "projectID",
  //projectId: "short-url-4e0b5"
});

module.exports = app;
```

From `Project settings` in your dashboard, insert your `apiKey` and `projectID` in the above code.

#### Saving Data

Before saving data in the database, the validity of the `longURL` needs to be checked.

`url.js`:

```js
const app = require("../firebase/fb_connect");
const { getDatabase, ref, set } = require("firebase/database");

const database = getDatabase(app);

...

const urlCode = shortid.generate();

  // Check long URL
  if (valid_url.isUri(longURL)) {
    const shortURL = `${baseURL}/${urlCode}`;
    const ofirebase = async () => {
      await set(ref(database, "url/" + urlCode), {
        longURL, //From the body
        shortURL,
        urlCode,
      });
      res.status(200).json({ success: true, message: 'Inserted Successfully ' });
    };
    ofirebase();
  } else {
    res.status(401).json({ success: false, message: "Invalid longURL" });
  }
};

router.post("/shorten", Shorten);

module.exports = router;

```

The app was initialized, imported into `url.js`, and used to set up the  database. 

The `ref` function takes in the instance of the database, the name of the collection, and the `id` of each row (which is specified as the `urlCode`). 

The `set` function takes in the `ref` function and the data to be added to the database.

### Redirecting users to the Long URL

Open up the `index.js` file inside the `routes` folder:

`index.js`:

```js
const express = require("express");
const router = express.Router();

const app = require("../firebase/fb_connect");
const { getDatabase, ref, child, get } = require("firebase/database");
const dbRef = ref(getDatabase(app));

router.get("/:code", (req, res) => {
  get(child(dbRef, `url/${req.params.code}`))
    .then((snapshot) => {
      if (snapshot.exists()) {
        console.log("redirecting");
        const longURL = snapshot.val().longURL;
        return res.redirect(longURL);
      } else {
        return res
          .status(404)
          .json({ success: false, message: "URL doesn't exist" });
      }
    })
    .catch((error) => {
      return res.status(500).json({ success: false, message: error.message });
    });
});

module.exports = router;

```

The `urlCode` will be gotten from the parameter, that `urlCode` from the parameter will be used  to get the `LongURL` from our database, and then the user is redirected to the `LongURL`. If the `urlCode` doesn't exist, a status code of `404` will be sent.

A middleware will be created in `server.js` file, that uses the `url.js` and `index.js` file to navigate to specific routes.

`server.js`: 

```js
...

app.use("/", require("./routes/index"));
app.use("/api", require("./routes/url"));

...
```

> If you encounter any error or have questions, refer to the [GitHub repo](https://github.com/divofred/short-url) for this tutorial.

## Deploying to Heroku

You will need to [create an account](https://signup.heroku.com/) with Heroku or[ Sign in](https://id.heroku.com/) if you already have one.

After Signing in, you will be redirected to your [dashboard](https://dashboard.heroku.com/apps). Click on `New` at the top-right corner of the page and click [`create new app`](https://dashboard.heroku.com/new-app?org=personal-apps). 

![Heroku1](https://github.com/divofred/Ata/blob/master/assests/images/Heroku1.jpg)

Choose a unique name for the application and create the app:

![Heroku2](https://github.com/divofred/Ata/blob/master/assests/images/Heroku2.jpg)

Download and Install [Heroku's CLI](https://devcenter.heroku.com/articles/heroku-cli).

After Installing Heroku's CLI, log in to Heroku in your terminal:

```bash
heroku login
```

This will prompt for a Login in your browser.

If you get an error like `Ip address mismatch`, run this code instead:

```bash
heroku login -i
```

This will ask you to put your Heroku login details in your terminal.

Let's deploy the app to Heroku

```bash
git init
heroku git:remote -a divofred-short-url
git add .
git commit -am "make it better"
git push heroku master
```

### Changing our `baseURL`

Since the project has gone from Local to Production, the `baseURL` of the app needs to be changed to the URL of our Heroku app.

`url.js`:

```js
...

const baseURL = "https://divofred-short-url.herokuapp.com";

...
```

You can use Postman to test our application:

![postman](https://github.com/divofred/Ata/blob/master/assests/images/postman.jpg)

## Conclusion

You have successfully built a URL Shortener using Node, Firebase Database, and Heroku as the Hosting Platform. Using these technologies, you can build complex and scalable applications.

Here is the link to the [GitHub repository](https://github.com/divofred/short-url).
