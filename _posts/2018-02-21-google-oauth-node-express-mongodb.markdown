---
layout: post
title:  "Google OAuth Node Express MongoDB!"
date:   2018-02-21 17:49:00 +0530
category: google-oauth
---

Authentication using google auth is very exciting and useful. Now-a-days many apps use
oauth system rather than traditional email and password authentication.

Firstly we will write our first route handler for `hello world` program.In project directory run `npm init` then `npm install --save express` for basic sructure and to install express. 

Make a file `index.js` and write following code in it.

{% highlight ruby %} 
  const express = require('express');

  const app = express();

  app.get('/',(req,res) => {
  res.send("hello world");
  });

  app.listen(3000);
{% endhighlight %} 

Now if  we will run `node index.js` then at `localhost:3000` in browser 'hello world' will be appeared.


 After this basic set up we are going dive deeper in Google Oauth. In this process we will go through following steps.

![My helpful screenshot]({{ "/assets/google.png" }})

Here we will use [passport.js]({{ site.baseurl }}{% post_url 2018-02-21-passport-introduction %}). So We will not have to write many boilertype code here. We install passport and passport's google-strategy.

{% highlight ruby %} 
  npm install --save passport passport passport-google-oauth20
{% endhighlight %}


We will go in `https://console.developers.google.com` and create a project(I have made emaily-google-auth). While configuring this app choose Oauth/ClientID and configure consent screen( it is used for asking user permission). This step are self-explanatory. Here you will get two field. One is `Authorized JavaScript origins`. As name suggest domain name or basic route url should be written here. If you are running your program at localhost(assume at port 3000) then write `http://localhost:3000`.
Another one is `Authorized redirect URIs`. As name suggest all uri which you want as authourized write here. This time write just `http://localhost:3000/*`, which means all uri at server. We will talk about this later.


Get clientId and secretId from there.Create a folder config and a keys.js file in it and store these keys in it because if you will push this project on github then you dont want to share your private key. If you have initialise git then you will have .gitignore file. Add keys.js in it.

In config/keys.js

{% highlight ruby %} 
  module.exports = {
    googleClientID = 'your clientID',
    googleSecretID = 'your secretID'
  };
{% endhighlight %}

In `index.js` we configure passport with googleClientId and googleSecretID. 
Also we have to add a callback funtion with four arguments(accessToken, refreshtoken, profile, done). This function will run when authentication is completed. We will get user data through profile variable.
{% highlight ruby %} 
  const keys = require('/config/keys');
  const passport = require('passport');
  const GoogleStrategy = require('passport-google-oauth20').Strategy;
  passport.use(new GoogleStrategy({
    clientID: keys.googleClientID,
    clientSecret: keys.googleClientSecret,
    callbackURL: '/auth/google/callback'
  }, (accessToken, refreshtoken, profile, done) => {
    console.log(profile);
  }));
{% endhighlight %}
Here one thing(last property 'callbackURL') we have to understand. If you will see the google-auth-flow(image) at very begining of this post. After user grants permission google send a callback in which a code is there in queryparameter. This code is very useful because we will send a request to google with this code. And we will ask google that we have code given by you now please give us user details. But where this code will go. (Remember in that image I have shown a url `http://localhost:4000/auth/google/callback?code='456'` ).This is where `callbackURL` comes in.


As discussed in diagram when user click `login with google` the authentication process(with passport here) will be initiated. So we will define a route in express server(index.js). We can take any path at server but I am taking `auth/google` here.
{% highlight ruby %} 
  app.get('/auth/google', passport.authenticate('google', {
    scope: ['profile', 'email']
  }))
{% endhighlight %} 

As we are making only backend for google auth. We assume user click `login with google` it goes to `http://localhost:3000/auth/google/`. Now we will go with the flow shown in diagram. We have define a route('/auth/google') just above and as route handler passport in initiated. We have given googleClientID and googleSecretID in passport configuration. So behind the scene passport make a call at `google.com/auth?appId='googleClientID'` and then google ask for permission and when user grant permission we get an error `redirect_uri_mismatch`. Why this!!! It should send a callback in which a code is there in queryparameter. Ya its true. Actually if you remember at `https://console.developers.google.com` in `Authorized redirect URIs` we have written `http://localhost:3000/*` but we have given our passport a different `callbackURI`. So just go there and replace that with `http://localhost:3000/auth/google/callback` and `redirect_uri_mismatch` error will be gone. It may take some time also. 

Now it will send a callback at url `http://localhost:3000/auth/google/callback?code='some unique code'` with a code. But we have not define a route handler for this. So we will define now.

{% highlight ruby %} 
  app.get('/auth/google/callback', passport.authenticate('google'));
{% endhighlight %} 

Now here this code (`passport.authenticate('google')`) is responsible for taking code coming from google and then ask google with this code for the user data. Authenticaton process is complete. In passport configuration we have define a callback and console profile. We can see it have some data now. 

Now our task is to save this user data in database(MongoDB). For this you can follow [next part]({{ site.baseurl }}{% post_url 2018-02-26-google-oauth-node-express-mongodb-part2 %}) 

If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[sub-cat-link]: http://localhost:4000/category/google-oauth.html
