---
layout: page
title: Lab5 p1
published: true
---



## Blog: Server

![](img/enm.jpg){: .small}


For this assignment we are going to build an [express](https://expressjs.com/) and [mongodb](https://www.mongodb.com/) CRUD api server for our react+redux blog frontend. This will finally bring our stack all the way down to the database.

Our server will be a pure api server, returning only JSON format data to our Lab4 frontend.

## Assignment At a Glance

* Part 1:
  * Intro to Express and Mongo. [SA7](../sa/server-side) got you started with Express.js and Mongo, we'll now use that knowledge to build up an API server.  The difference here is that we won't be returning HTML but rather JSON. Easier! We'll add in the `post` schema, get going with express routes, and get set up for adding in the other stuff.
  * Basic CRUD API: Building from the intro, we'll implement the full create, update, delete api for our blog.
* Part 2:
  * Authentication: We'll extend both our frontend and our backend to support authentication and users and more!


## Some Setup


First we should do some basic setup steps.  

🚀 Do what you did in [SA4](http://cs52.me/assignments/sa/react-videos/) when pulling from your own starterpack but in this case we'll pull from a different starter — create your own repo with the classroom link, add a starter remote to [express-babel-starter](https://github.com/dartmouth-cs52/express-babel-starter), and pull from it. Then run these following commands to start our new node+express app in dev reloading mode.

```bash
npm install
npm run dev
```

This will start our new node+express app on [http://localhost:9090](http://localhost:9090) in dev reloading mode.

Ok, now that we got that out of the way. Let's dig in!  


### React App Debugging

The whole point here is to get your blog app working with this new server, so we will point your blog frontend to this new server.

🚀 Change your server api url:

```javascript
const ROOT_URL = 'http://localhost:9090/api';
// const ROOT_URL = 'https://cs52-blog.herokuapp.com/api';
```

And start up your react+redux blog app.  It'll be broken for now (should display no blog entries), but hopefully soon we'll have it all working!


## Intro Express

[Express](https://expressjs.com/) is a web framework for Node.js.  What it does for us is provide a way to listen for and respond to incoming web requests.

Each of the http API requests we were making in Part 1 will need to be provided in this assignment.

To recap the API has the following endpoints:

* GET  `/api/posts/`
  returns only title and tags for all posts
  `[[{"id":"",title":"","tags":""},...]`
* POST `/api/posts/` with post parameters `{'title', 'tags', 'content'}`
  creates a new post
* PUT `/api/posts/:postID` with parameters `{'title', 'tags', 'content'}`
  will update an entry
* GET `/api/posts/:postID`
  returns the post found at `postID`
* DELETE `/api/posts/:postID`
  deletes the post found at `postID`

Let's implement these and more!

Take a look through the current `app/server.js` file. This is the entry point for the app. Just like `index.js` has been in our frontend app (the names of these are arbitrary). Note how we are setting the route:

```javascript
// default index route
app.get('/', (req, res) => {
  res.send('hi');
});
```

This routing concept is identical to the routing we used in our react app.  It maps URL paths to functions.  These functions take 2 arguments:  request and response.  

Request is an express object that contains, among other things, any data that was part of the request. For instance, the JSON parameters we would POST or PUT in our asynchronous `axios` calls would be available as `req.body.title`, etc.  

Response is another special express object that contains, among other things, a method named `send` that allows us to send back a response to the client.  When your api call gets back JSON data this is how it is returned.  Consider `res.send()` the equivalent of a network based `return` statement.

We'll add more routing in shortly, but first let's set up our database!


## Mongo Database Server

Mongo is the database that we are going to use.  We will run a local mongo daemon to connect to for testing.


🚀 On OSX to install:

```bash
brew install mongodb
```

Then follow further [installation instructions here](https://docs.mongodb.com/manual/installation/#mongodb-community-edition).

You will need to run the `mongod &` process, which your node app will connect to.  This is a background server process.

There is a commmandline client you can use to connect to the database: `mongo`. You can also play around with a more graphical client [robomongo](https://robomongo.org/).

```javascript
// mongoshell is a commandline interface to your local mongo db

show dbs
// will show your current databases

use blog
// will make blog the current database

db.posts.insert(
   {
     "title": "first post",
     "tags": "words",
     "content":  "this is a test post",
     "cover_url": "https://media.giphy.com/media/uscuTAPrWqmqI/giphy.gif"
   }
)
// will insert an object into the database
// into a collection called posts

db.posts.find()
// returns everything in this collection

db.posts.find({"title": "first post"})
// finds a entry in database by nested key:value
```

Ok, so now you've played a little bit with mongo directly, let's build something on top of it.

## Mongoose

![](img/mongoose.jpg){: .small .fancy }

Don't forget to install  Mongoose! Just repeat the Mongoose section from [SA7](../sa/server-side/#mongoose)




## Model

We're going to create a `Post` data model to work with. You've already made something similar in [SA7](../sa/server-side/#models)

🚀 Create a directory `app/models` and a file inside this directory named `post_model.js`.


```javascript
import mongoose, { Schema } from 'mongoose';

// create a PostSchema with a title field


// create PostModel class from schema


export default PostModel;
```

This will do for now, let's see about how to use this.

## Controller

🚀 Create a directory `app/controllers` and a file inside this named `post_controller.js`.

What might our controller do?

Well it should have methods that perform all the main functionality of our API.  In short those methods would be:

```javascript
import Post from '../models/post_model';

export const createPost = (req, res) => {
  res.send('post should be created here');
};
export const getPosts = (req, res) => {
  res.send('posts should be returned');
};
export const getPost = (req, res) => {
  res.send('single post looked up');
};
export const deletePost = (req, res) => {
  res.send('delete a post here');
};
export const updatePost = (req, res) => {
  res.send('update a post here');
};
```

Let's just have them empty for now and then deal with the details later.  ⚠️ Note now this is a bit different from what we did in SA7.  There we played with promises, but to simplify things here we'll just pass in `req, res` from our routes directly to the controller. Up to you how you prefer it, but this way is a bit terser.


### Routing

Now we are ready to wire it all together with routes!  The way we were doing it earlier is not very extensible. Lets create a separate file for routes.

🚀 Create:  `app/router.js`.

Here's some boilerplate to get us going:

```javascript
import { Router } from 'express';
import * as Posts from './controllers/post_controller';


const router = Router();

router.get('/', (req, res) => {
  res.json({ message: 'welcome to our blog api!' });
});

///your routes will go here

export default router;
```

Express provides a nice [route interface](https://expressjs.com/en/4x/api.html#router) that we will use to define all of our routes.

The chaining method simplifies how our routes look. For instance here is how we could define our `posts/:id` routes.

```javascript
// example!
// on routes ending in /someroute/:someID
// ----------------------------------------------------
router.route('/someroute/:someID')
  .post(/*someMethod*/)
  .get(/*someMethod*/)
  .delete(/*someMethod*/);
```
{: .example}

Note `/*someMethod*/` is just a comment, you would call a method there in a module that we will call the controller — more on that shortly!


Ok, remember how we defined all our API endpoints?   Let's map them in our router.

🚀 Use the syntax above to make routes to map the following:

* POST `/posts`:  Posts.createPost
* GET `/posts`:  Posts.getPosts

and also for:

* GET `/posts/:id`: Posts.getPost
* PUT `/posts/:id`: Posts.updatePost
* DELETE `/posts/:id`: Posts.deletePost

You will have 2 `router.route()` definitions with separate chains of HTTP verb methods, one for `/posts` and one for `/posts/:id`.


### Import New Routes

🚀 Now in your `app/server.js` file import our new routes and assign them to handle all `/api/*` routes!

```javascript
//at top of server.js
import apiRouter from './router';

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
// this should go AFTER body parser
app.use('/api', apiRouter);
```

Neat!

Note: the `app.use('api` line should go towards the bottom of the file, in particular after bodyparser.  The `server.js` file is read top to bottom every connection request, so when it hits the route definition if the body of the request hasn't been parsed you will get errors when you try to use  `req.body`.

### First Pass Test

Let's test what we have done so far! Test out each of the routes using these curl commands (just modified version with localhost from hw4):

```bash
# all posts get:
curl -X GET "http://localhost:9090/api/posts"

# create new post
curl -X POST -H "Content-Type: application/json" -d '{
    "title": "first post",
    "tags": "words",
    "content":  "this is a test post",
    "cover_url": "https://media.giphy.com/media/uscuTAPrWqmqI/giphy.gif"
}' "http://localhost:9090/api/posts"

# update by POSTID
curl -X PUT -H "Content-Type: application/json" -d '{
    "title": "new title"
}' "http://localhost:9090/api/posts/POSTID"

# fetch 1 by POSTID
curl -X GET "http://localhost:9090/api/posts/POSTID"

# delete by POSTID
curl -X DELETE -H "Content-Type: application/json" "http://localhost:9090/api/posts/POSTID"

```


## Controller Continued

Ok but our controller `controllers/post_controller.js` is fairly useless.  We have everything wired, but we need to actually store stuff.

Let's walk through making one of those endpoints not useless!

The most important might be the `createPost` endpoint.  If you recall from Lab4 this gets called with the fields of our new post `{title: '', tags: '', contents: '', cover_url: ''}`.  These end up in our `req` (request) object, specifically in `req.body`.

Let's fill out the contents of the `createPost` method:

🚀 create a new Post object:

```javascript
const post = new Post();
```

🚀 All our fields are available in `req.body`, so let's set them on the new Post object. You know how to do this.

Now we just have to save the object (so far we've been working with a new instance purely in memory).  Most save and query methods in Mongoose can return promises — so let's stretch our promise muscles a little.

```javascript
post.save()
.then(result => {
  res.json({ message: 'Post created!' });
})
.catch(error => {
  res.status(500).json({ error });
});
```

It is common practice to return the modified object in a API call as confirmation, but often API's will just return various success codes also to indicate success or failure. We're going to return JSON though for now, but for the error case we will set an error status code and also return the error.


🚀 Let's test that this worked:

```bash
curl -X POST -H "Content-Type: application/json" -d '{
    "title": "first post"
}' "http://localhost:9090/api/posts"
```

And then in mongo shell:  

```javascript
use blog
db.posts.find()
```

## Get All Endpoints Working


Your mission is to now implement the rest of the endpoints!  You have the wiring ready, all you need is to use the [mongoose docs](http://mongoosejs.com/docs/queries.html) to implement `getPost`, `getPosts`, `updatePost`, and `deletePost`.  You may find mongoose methods such as: `.find()`, `.findById()`, `.remove()` helpful.  You might want to look into sorting the results for `getPosts` by `created_at`.

Note!  In the above we only saved `title`, none of the other fields were defined in our Post Schema!  You should now go back and add the other fields we need to the model as well as to the all the controller methods.

One more caveat! Mongo happens to use `_id` instead of `id` as we have been so you could use a method like the following to format the list of posts to be cleaner.

```javascript
// this cleans the posts because we use id instead of dangling _id
// and we purposefully don't return content here either
const cleanPosts = (posts) => {
  return posts.map((post) => {
    return { id: post._id, title: post.title, tags: post.tags, cover_url: post.cover_url };
  });
};
```

This would be before you `res.json()` them back to the client.

And finally, you'll need to get the router id that is passed in when we hit `/posts/:id`.  This is accessible as `req.params.id` inside each of our controller functions that had this `:id` path variable.



## What about APIKEY

Note, unlike the blog api we've been using, nothing in the above relies on the query parameter `?key=foobar`. This is because this is your personal database for which we're shortly going to implement authentication, so you don't really need the APIKEY sandboxing.  If you were curious and wanted to implement it it would be available to you in `req.query.key` and easiest would be to store it in each document and then query on it.


## Deploy

We will need to host this new server component so your blog can use it instead of the `cs52-blog.herokuapp.com` one.  

Great! We have everything working now. We will need to host this new server component!

1. Head over to [Heroku](https://www.heroku.com/) and login/sign up. Then, make a new app.
1. Now you need to connect to a mongo database.  Go to *Resources* and search for "mLab" under *Add-Ons*. Provision the *Sandbox* version of mLab for your app. This will automatically set a `MONGODB_URI` config variable so once you push your code to Heroku it will connect to this new mongo database. You'll need to enter in a credit card but it is free so it won't be charged.
1. Follow the steps under "Deploy Using Heroku Git".  But really all you need is to add a new git remote - find your heroku git URL by going to "Settings" and then do `git remote add heroku https://git.heroku.com/cs52-blog.git`.   
1. To host on heroku all you need to do is `git push heroku master`, this will push your code and run the npm command that is listed in your `Procfile` to launch your app.  COOL!

Note: Don't forget to push master to **both** heroku and origin.


## P1 Complete

Once you have all the api endpoints complete, test it out using your blog frontend, make sure all the parts still work!  IE. Change your Lab4 `ROOT_URL` to point to your Heroku hosted server instance.  Commit and push your very slightly altered Lab4.


## To Turn In

1. github url to your repo
1. url to your new heroku app instance for testing
1. working url for Lab4 on surge that points to your new API server. You can just change it to post to your own.

## Extra Credit

* change your tags store to be an array rather than a string, can just split by whitespace
* add commenting to posts (either an array or another model) / change both api and frontend to support this.
* really at this point you can start modifying your blog to be whatever you want. Add in new fields to your posts.
* add in search support. Here's an [article](https://www.compose.com/articles/full-text-search-with-mongodb-and-node-js/) that might help you get started.
* in part 2 we'll introduce User and Authentication so don't implement those here though.
