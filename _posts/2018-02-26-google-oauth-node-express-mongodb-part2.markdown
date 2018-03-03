---
layout: post
title:  "google oauth with node-express-mongodb-second-part"
date:   2018-02-21 17:49:00 +0530
categories: google-oauth
---

In previous part we were successfull in getting user profile data from google. Now our job is to save data in database(mongodb) and use that data in follow up request. 

We are using mongoose and mongodb here. Mongoose is javascript library used for making model(for example user). In mongodb this model becomes collection and every instance of user model become item in mongodb's collection. 

Either we set up mongodb on our local computer or online. We will set our mongodb. For this we will log in on mlab.  

Login on `https://mlab.com/` and create database(suppose 'mydb') and also dont select `read only user`. After creating database create at least one user(as admin to this database). Now install mongoose. 

{% highlight ruby %} 
  npm  install --save mongoose
{% endhighlight %}  

Now we will connect mongodb to mongoose. If we open a our database at mlab we can see a mangoDB URI at top.Copy this and place in config/keys.js file as mongodbURI key. In this uri replace username and password with yours. Dont use `@` in your password because it is used as seperator here. Now I suppose you have a mongodbURI key and it has value(copied from mlab).

In index.js   

{% highlight ruby %} 
  const mongoose = require('mongoose')
  mongoose.connect(keys.mongodbURI)
{% endhighlight %}  

Now we want to save a collection users in mongodb. And each item in users collection will have one property `googleId`. So we will define a model in mongoose. 

In index.js  after mongoose connection.

{% highlight ruby %} 
  const { Schema } = mongoose;
  const userSchema = new Schema({
    googleId: Sring
  });

  const User = mongoose.model('users', userSchema)
{% endhighlight %}  

Now inside callback function of passport in index.js, we create a user instance and using save method on it we save this in mongodb.


{% highlight ruby %} 
  (accessToken, refreshtoken, profile, done) => {
    console.log(profile);
    new User({ googleId: profile.id }).save()
  }
{% endhighlight %}  

If we go at `http://localhost:3000/auth/google/`. We can see in our online mongodb database(which we have created) have one item with `googleId`. But there is a problem if we will go again on same link another  item with same `googleId` will be created. But this is not good.
Whenever a user google profile goes to our server, we should check if any user with this profile ID is present or not. If present then it is OK nothing to do but if not present then create a user and tell passport that we have finished authentication process. 


in Index.js

{% highlight ruby %} 
  (accessToken, refreshtoken, profile, done) => {
    User.findOne({ googleId: profile.id}).then((existingUser) => {
      if (existingUser) {
         done(null, existingUser);
      } else {
         new User({ googleId: profile.id }).save()
         .then((user) => {
           done(null, user);
         });
      }
    });
  }
{% endhighlight %}  

Now we are creating user only when there is no user with google profile Id. But after craeting new user or getting an old user, if we make a new request (suppose to get all of my post) our server could not recognise us. This because browser talk with server using HTTP which is stateless. It means it has no idea what happened in previous request. 

Hi this is a problem, how could server to recognise us. I mean it is necessary. Now I will tell you what to do. After creating new user or getting old user, server will generate a token. To generate this token we pass a function to passport's serialiseUser. This serialiseUser also save this token in cookie. When we make any follow up request(suppose for posts of user) we take this token from cookie call passport's deserialiseUser to get User. So in this way server will have user info so it can give user's posts. 
So we will define here two functions and pass to serialiseUser and deserialiseUser.

in Index.js

{% highlight ruby %} 
  passport.serialiseUser((user, done) => {
    done(null, user.id);
  });

  passport.deserialiseUser((id, done) => {
    User.findById(id).then((user) => {
      done(null, user);
    });
  });
{% endhighlight %}  

Till this time our app is able to deal with cookie because by default express can not deal with cookie. So we will install `cookie-session`.

{% highlight ruby %} 
  npm  install --save cookie-session
{% endhighlight %}  

Now we will do two things first I will tell express app to allow cookie and second tell to passport to use cookie.

{% highlight ruby %} 
 import cookieSession from 'cookie-session';

  const app = express();
  app.use(cookieSession({
    maxAge: 7 * 24 * 60 * 60 * 1000
    keys: [keys.cookieKey]
  }))
  #=> define a cookieKey in keys.js as any string. it is used for encrypt token.
{% endhighlight %}  

{% highlight ruby %} 
  app.use(passport.initialise());
  app.use(passport.session());
{% endhighlight %}  

Now to test things out, we will define a route handler,

{% highlight ruby %} 
  app.get('/current_user', (req, res) => {
    res.send(req.user);
  });
{% endhighlight %}  


If we will go on `http://localhost:3000/current_user`. We will see some user data.

Now for logout in index.js

{% highlight ruby %} 
  app.get('/logout', (req, res) => {
    req.logout();
    res.send(req.user);
  });
{% endhighlight %} 


If we will go on `http://localhost:3000/logout` and then If we will go on `http://localhost:3000/current_user` we will not get any user data.






