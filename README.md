# CS52 Workshops:  Social Media Auth Integration! 

![](https://media.giphy.com/media/gcajW7oKirCdW/giphy.gif)

Authentication is a necessary feature for any app or website that wants to keep its users' information safe. In lab 5 part 2, we created our own authentication, but realistically developers would need something more robust and secure. This is where social media authentication comes in. Social media companies such as Facebook, Google, and Twitter provide their own authentication APIs that allow users to easily log in with their own credentials for that website to become authenticated users. These authentication APIs are secure and easy to use, so they are a great choice for creating a safe user experience. Let's see how we can integrate one of these authentication APIs into our web apps!

## Overview

We're going to be adding Facebook authentication to the simple site we've made with our starter pack! Once we're finished, you will be able log into your starter pack site with your facebook credentials, and your site will have access to basic info from your profile like your name and profile photo! 

## Setup
In this workshop, we're going to modify the code from your starter pack, but we don't want to mess with what's currently in your cs52-20s repo, so we're going to fork! 
Fork your starter pack repo by clicking the button in the top right on the github console) 
![](https://i.imgur.com/rm90tKJ.png)
This will allow you to create a new repo that contains all of the code from your starter pack. Now we can modify it without the fear that we'll mess with what's working now! 

clone your new repo 
```
git clone https://github.com/YOURUSERNAME/NEWREPONAME.git
```
and don't forget to run `yarn install`!
   
Next, head over to the [Facebook Developers](https://developers.facebook.com/) site and login to your facebook account. 

Once you're logged in, navigate to the "My Apps" tab in the upper right-hand corner and click on the "Create App" button.

A modal should pop-up asking for a display name and contact email. The email is pre-filled in from whatever email is associated with your Facebook account. Display name is just the name of your new app! 

Click through the security check, and you should see a console that looks somewhat like this: 
![](https://i.imgur.com/btBIXsZ.png)

Facebook Login is right at the top! Just click set-up and web when it prompts you. 

To start, we're going to set our site url to `http://localhost` but you can always change this later once you've deployed your site to a surge or heroku url. 

Once you save your site url, you can click through the rest of the pages under the quick setup. We'll handle all of that ourselves! 


## Step by Step
Okay now we can start with actually implementing facebook login. 

First we're going to add a helpful library: react-facebook-login. This library allows us to easily connect our web app to the Facebook login service. Here are the [docs](https://github.com/ruvictor/facebook-login-react) if you're interested in learning more. 

```
yarn add react-facebook-login
```
Great! Now we can use components from this library in our app! 

Add a `.env` file to your root directory with the contents: `HTTPS=true` since Facebook doesn't let us use their authentication service if our app does not utilize SSL certificates.

Create a new file in your components directory called `Facebook.js`. This will be a react component so make sure to import `React` and `{ Component }`. 

We're going to be using the login button from the react-facebook-login library so go ahead and import that too. 
 ```javascript
import FacebookLoginBtn from 'react-facebook-login';
```
We're going to be building a class called FacebookLogin which we'll export to use in our `App.js` file. 

Define a constructor for our component with state variables `auth`, `name`, and `picture`. Name and picture can be empty strings, and we'll set `auth` to be false initially. 
```javascript 
constructor(props) {
    super(props);
    this.state = {
      auth: false,
      name: '',
      picture: '',
    };
  }
```

Let's actually start with our render function, a bit counter-intuitive, I know, but trust me. We're going to want to render two different states, either an option to login with `auth == false` or the name and profile photo of whoever is logged in. 

Set a const `facebookData` to be assigned conditionally, depending on `this.state.auth`. This is a great chance to practice the syntax for using a conditional operator. 

```javascript
const facebookData = this.state.auth
    ? ( <IF TRUE, DO THIS> )
    : ( <ELSE, DO THIS> )

```

If the user is logged in, we want to display what's stored in our state, but if not, we want to present a button where they can log in with facebook...if only we had imported a facebook login button earlier. 

```javascript
const facebookData = this.state.auth
    ? (
      <div className="userInfo">
        <img src={this.state.picture} alt={this.state.name} />
        <h2>Welcome {this.state.name}!</h2>
      </div>
    )
    : (
      <FacebookLoginBtn
        appId="12345678910"
        autoLoad
        fields="name,email,picture"
        callback={this.responseFacebook}
      />
    );
```
Let's add some quick, basic styling for our userInfo class. Open your `style.scss` file and add the following: 
```css
.userInfo{
  text-align: center;
  width: '400px';
  margin: 'auto';
  background: '#f4f4f4';
  padding: '20px';
}
```
This will just resize and center the profile photo and text once we do have a user's login info. 

Okay, back to `Facebook.js`.
We still need to write our return method. The return method for the FacebookLogin component will just contain whatever's in our constant facebookData. Go ahead and write that return statement inside the render function. 

You'll notice we haven't actually defined our callback function `responseFacebook` yet. Let's do that now. 

Our callback function is specifying what happens right after the authentication API returns a response for the authentication request. If the user is successfully authenticated, we need to update our state with the user's Facebook information.

The function should look like this: 
```javascript
responseFacebook = (response) => {
    if (response.status !== 'unknown') {
      this.setState({
        auth: true,
        name: response.name,
        picture: response.picture.data.url,
      });
    }
  }
```

In the `response` object we receive from Facebook is an accessTooken which, along with the facebookid, is what we might send to our own API server to then double check with Facebook that our user is authenticated. If you choose to integrate social media auth in your final projects, the accessToken will be super important for validating auth. 

<details>
 <summary>Your final Facebook.js file </summary>
 
 ```javascript
import React, { Component } from 'react';
import FacebookLoginBtn from 'react-facebook-login';


class FacebookLogin extends Component {
  constructor(props) {
    super(props);
    this.state = {
      auth: false,
      name: '',
      picture: '',
    };
  }

  responseFacebook = (response) => {
    if (response.status !== 'unknown') {
      this.setState({
        auth: true,
        name: response.name,
        picture: response.picture.data.url,
      });
    }
  }

  render() {
    const facebookData = this.state.auth
      ? (
        <div className="userInfo">
          <img src={this.state.picture} alt={this.state.name} />
          <h2>Welcome {this.state.name}!</h2>
        </div>
      )
      : (
        <FacebookLoginBtn
          appId="169627331126024"
          autoLoad
          fields="name,email,picture"
          callback={this.responseFacebook}
        />
      );
    return (
      <div>
        { facebookData}
      </div>
    );
  }
}

export default FacebookLogin;


 ```
</details>


Now we need to add our `FacebookLogin` component to our `App.js` file. 

We added it to our Welcome component, but you can choose to add it to any tab on your page. Don't forget to import it at the top of your file!

Go ahead, run `yarn start` and checkout your site. Do you see a button that says "login with facebook"? 

![](https://i.imgur.com/g24BHvB.png)


Let's test it. Click on the button. 

Oh no! It doesn't work! 

This is because we didn't add our appId from Facebook to our React app. Facebook doesn't actually know that we're trying to connect to it. 

Go back to your `Facebook.js` file. Do you notice anything strange in the specs for the `FacebookLoginBtn`? 

That's right. We never inputted our own appId so there's no connection between our React App and the site we registered on the Facebook Developers site. Go back to your project on the [FB Developers](https://developers.facebook.com/) site and copy your unique AppID. Now use that to replace the fake appId from before. Save...

And voila! You're done! 
Now try clicking on the login button, and you should be able to log into your facebook account (make sure you allow pop-ups). 

![](https://i.imgur.com/D7A2DLK.png)


This isn't the most clean or pretty site, but with a little bit of styling, it could be a welcome page for a website after login or the beginning of a profile. 



## Summary / What you Learned

* [ ] Use facebook-login-react to build a Facebook Login feature
* [ ] Apply unique appId to login
* [ ] Effectively use conditional operator in the render function

## Reflection

* [ ] Why do we need the callback function `responseFacebook`, and how does it work?
* [ ] When we build our Apps, we can add Facebook Login at `Facebook Login > Settings page`, under `Valid Oauth Redirect URIs`. What are some advantages of using OAuth?


## Resources

* https://auth0.com/docs/connections/social/facebook
