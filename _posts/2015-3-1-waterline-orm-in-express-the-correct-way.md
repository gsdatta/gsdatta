---
layout:     post
title:      "Set up Waterline ORM with ExpressJS"
subtitle:   "Easily set up an ORM for relational databases in ExpressJS."
date:       2015-03-1 8:59:00
author:     "Ganesh Datta"
header-img:     "img/waterline.png"
description: "Setting up the Waterline ORM in express is easy if you put everything in one file, but I like to keep my apps organized. Here's how I setup Waterline in a clean, readable manner for ExpressJS."
ogimage: "img/waterline.png"
comments:   true
---

I've been working with Node.js and the ExpressJS framework recently for a project of mine, and I wanted to use Postgres as my database due to the relational nature of my data. I looked for an ORM to speed up my development, and came across multiple, including Sequelize and Bookshelf, among others. Looking at their docs, I found that Waterline, the default ORM in SailsJS, was much friendlier to use from my last encounter with Sails. So, I decided to figure out how to use Waterline in an Express application in a more modular format. 

*Disclaimer: I'm in no way an expert at this stuff, so please do correct me in the comment section if I'm wrong!*

#Folder Setup
I like to keep my project organized, so my folder layout for a REST API looks like the following (a modification of the built in express-generator):

~~~
myapp
    - bin
        + www
    - models
        + index.js
        + all other models
    - router
        + routes (containing all routes)
        + index.js
    - services
~~~

Setting up Waterline with Express is quite simple if you have all the models in the same file. However, keeping things modular makes it a little more complicated. First off is my models folder. In my models folder, I have an index.js file, along with a file for each individual model. For example, `models/User.js` would look like:

{% highlight javascript %}
var Waterline = require('Waterline');
var bcrypt = require('bcrypt');

var User = Waterline.Collection.extend({
  identity: 'user',
  connection: 'myPostgres',

  attributes: {

    email: {
      type: 'string',
      email: true,
      required: true,
      unique: true
    },

    name: {
      type: 'string',
      required: true,
    },

    password: {
      type: 'string',
      required: true
    },

    classes: {
      type: 'array',
      required: true
    },

    toJSON: function() {
      var obj = this.toObject();
      delete obj.password;
      return obj;
    }   
  },

  beforeCreate: function(values, next){
    var bcrypt = require('bcrypt');

    bcrypt.genSalt(10, function(err, salt) {
      if (err) return next(err);

      bcrypt.hash(values.password, salt, function(err, hash) {
        if (err) return next(err);

        values.password = hash;
        next();
      });
    });
  }
});

module.exports = User;
{% endhighlight %}
Notice the connection field in the model. This will be defined in the index.js file. Speaking of which, let's move on to that file now!

#index.js - Why?
Waterline needs to be told about all models so that it can create the correct tables, etc. in your database, and allow you to access these models elsewhere in your app. However, we've defined all these models in different files, and we want to make it easy for these models to be added to the ORM automatically. So, I use an index.js file to automate this process for me. 

Additionally, I use it to set up the actual connection to the database (though this should probably be in an external config file).

{% highlight javascript %}
var postgresAdapter = require('sails-postgresql');
var Waterline = require('waterline');

var orm = new Waterline();

var config = {
  adapters: {
    postgresql: postgresAdapter
  },

  connections: {
    myPostgres: {
      adapter: 'postgresql',
      host: 'localhost',
      user: 'ganeshdatta',
      database: 'studygroup-express'
    }
  }
};

var fs = require('fs');
var path      = require("path");

fs
  .readdirSync(__dirname)
  .filter(function(file) {
    return (file.indexOf(".") !== 0) && (file !== "index.js");
  })
  .forEach(function(file) {
    var model = require(path.join(__dirname, file));
    orm.loadCollection(model);
  });

module.exports = {waterline: orm, config: config};
{% endhighlight %}

There are two things going on here. First, we create a waterline object, which will be used to access models throughout the application.

Second, we're going through all the files in the models folder, and added them to waterline with the `loadCollection()` method. The best part about doing it this way is that we can continue adding model files to the models directory, and they'll automatically be added to Waterline!

Also, notice how the module exports two things - the orm object itself, and the config we defined here. This will be used in our application startup file (`bin/www` in my case), so that we can get the orm running correctly when the server is started. 

#Server Startup
Wherever you're starting your server, you would now need to add this little snippet of code:

~~~
var models = require('../models');

models.waterline.initialize(models.config, function(err, models) {
  if(err) throw err;
  // console.log(models.collections);
  app.models = models.collections;
  app.connections = models.connections;
 
  // Start Server
  server.listen(port);
});
~~~

What this does is initializes the server with the defined config in index.js, and then start the server only if waterline is up and running correctly. Additionally, it adds the collections (models) to your app object (assuming your app is called `app`, obviously), so you can access them around the application by calling it like `app.models.user.findOne({})`.

With this, waterline is set up in a modular way in express! Shoutout to the guys at SailsJS and Waterline working on this awesome ORM!