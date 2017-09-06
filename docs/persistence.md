# Persistence

Generally speaking, adding database storage or persistence to your Probot App will greatly complicate your life. It's perfectly doable, but for most use-cases you'll be able to manage relevant data within GitHub issues, pull requests and your app's configuration file.

For when you absolutely do need external data storage, here are some examples using a few popular database solutions. Your needs will be slightly different from what any example can help with. Below are some code suggestions to help you get started.

## MongoDB (with Mongoose)

[MongoDB](https://mongodb.com) is a popular NoSQL database, and [Mongoose](http://mongoosejs.com) is the de facto Node.js wrapper for it.

```js
// PeopleSchema.js

const mongoose = require('mongoose');

const Schema = mongoose.Schema;

const PeopleSchema = new Schema({
  name: {
    type: String,
    required: true,
  },
});

module.exports = mongoose.model('People', PeopleSchema);
```

```js
// index.js

const mongoose = require('mongoose');

// Connect to the Mongo database using credentials
// in your environment variables
const mongoUri = `mongodb://${process.env.DB_HOST}`;

mongoose.connect(mongoUri, {
  user: process.env.DB_USER,
  pass: process.env.DB_PASS,
  useMongoClient: true,
});

// Register the mongoose model
const People = require('./PeopleSchema');

module.exports = robot => {
  robot.on('issues.opened', async context => {
    // Find all the people in the database
    const people = await People.find().exec();

    // Generate a string using all the peoples' names.
    // It would look like: 'Jason, Jane, James, Jennifer'
    const peoplesNames = people.map(person => person.name).join(', ');

    // `context` extracts information from the event, which can be passed to
    // GitHub API calls. This will return:
    //   {owner: 'yourname', repo: 'yourrepo', number: 123, body: 'The following people are in the database: Jason, Jane, James, Jennifer'}
    const params = context.issue({body: `The following people are in the database: ${peoplesNames}`})

    // Post a comment on the issue
    return context.github.issues.createComment(params);
  });
};
```

## MySQL

## Redis

## Firebase

[Firebase](https://firebase.google.com/) is Google's services-as-a-service that includes a simple JSON database. You can learn more about dealing with the Javascript API [here](https://firebase.google.com/docs/database/web/start). Note that for security purposes, you may also want to look into the [Admin API](https://firebase.google.com/docs/database/admin/start).

```js
// index.js

const firebase = require('firebase');
// Set the configuration for your app
// TODO: Replace with your project's config object
const config = {
  apiKey: "apiKey",
  authDomain: "projectId.firebaseapp.com",
  databaseURL: "https://databaseName.firebaseio.com",
};
firebase.initializeApp(config);

const database = firebase.database();

module.exports = robot => {
  robot.on('issues.opened', async context => {
    // Find all the people in the database
    const people = await database.ref('/people').once('value').then((snapshot) => {
      return snapshot.val();
    });

    // Generate a string using all the peoples' names.
    // It would look like: 'Jason, Jane, James, Jennifer'
    const peoplesNames = Object.keys(people).map(key => people[key].name).join(', ');

    // `context` extracts information from the event, which can be passed to
    // GitHub API calls. This will return:
    //   {owner: 'yourname', repo: 'yourrepo', number: 123, body: 'The following people are in the database: Jason, Jane, James, Jennifer'}
    const params = context.issue({body: `The following people are in the database: ${peoplesNames}`})

    // Post a comment on the issue
    return context.github.issues.createComment(params);
  });
};
```
