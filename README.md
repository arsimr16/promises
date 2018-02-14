# promises
This is a project I completed as a student at [hackreactor](http://hackreactor.com). This project was worked on with a pair.

Promises are a very powerful abstraction on top of callbacks and many asynchronous libraries are starting to use them by default. They are included as a language standard in ES6, and with future versions building them up even further, now is a great time to learn.

This sprint is designed to ease you into using implementing and consuming promises by comparing them to the callback pattern you are already familiar with.

## Setup
* npm install

## Bare Minimum Requirements
**Review the Node style callback pattern**
Asynchronous functions in JavaScript should follow the **node style callback pattern**. There are two conditions for this pattern:
  * The function expects a callback as the last argument
  * The callback is invoked with (err, results)
Here's an example of consuming Node's built-in fs.readFile. Notice that the callback we pass into it meets the two conditions above:
```javascript
fs.readFile(__dirname + '/README.md', 'utf8', function (err, content) {
  console.log('Example from callbackReview.js')
  if (err) {
    console.log('fs.readFile failed :(\n', err)
  } else {
    console.log('fs.readFile successfully completed :)\n', content)
  }
});
```
* Complete the exercises in bare_minimum/callbackReview.js and make the Callback review tests pass

**Use the Promise constructor**
There are five steps to writing a promise-returning function:
* Create a promise with the new Promise constructor
* Do something async, then...
  * Pass the successful value into the resolve function
    * this value will be made available in the next then block
    * only 1 value can ever be passed into resolve
  * Pass any errors into the reject function
    * this error will be made available in the catch block
* return the promise instance. This should be a synchronous step

* Complete the exercises in bare_minimum/promiseConstructor.js and make the Promise constructor tests pass

**Promisify existing functions**
There's an easier way to create a promise returning version of a function that exactly follows the node style callback pattern used previously. This technique is called **promisification**, and implemented different in each library

All of the work done in promiseConstructor.js can be done in these three lines:
```javascript
var nodeStyle = require('./callbackReview.js');
var pluckFirstLineFromFileAsync = Promise.promisify(nodeStyle.pluckFirstLineFromFile)
var getStatusCodeAsync = Promise.promisify(nodeStyle.getStatusCode)
```
Assuming all functions in a library precisely follow the node style callback pattern, you can even promisify an entire library!
```javascript
Promise.promisifyAll(fs);
// Promisifies all 'fs' functions and gives us an `Async` suffixed version
// For example - `fs.readFileAsync`, `fs.writeFileAsync`
```
Read more [here](http://bluebirdjs.com/docs/api/promisification.html)
* Complete the exercises in bare_minimum/promisification.js and make the Promisification tests pass

**Chain promises together**
Remember the pyramid of doom?
```javascript
var addNewUserToDatabase = function(user, callback) {
  // (1) See if the user already exists
  db.findUserInDatabase(user, function(err, existingUser) {
    if (err) {
      callback(err, null)
    } else if (existingUser) {
      callback('User already exists!', null)
    } else {
      // (2) then, secure the user by hashing the pw
      db.hashPassword(user, function(err, securedUser) {
        if (err) {
          callback(err, null)
        } else {
          // (3) then, create and save the new secured user
          db.createAndSaveUser(securedUser, function(err, savedUser) {
            if (err) {
              callback(err, null)
            } else {
              // (4) We're finally done! Pass our new saved user along
              callback(null, savedUser)
            }
          });
        }
      });
    }
  });
};
```
Promises can be chained to flatten the pyramid:
```javascript
Promise.promisifyAll(db)

var addNewUserToDatabaseAsync = function(user) {
  // The outermost `return` lets us continue the chain
  // after an invocation of `addNewUserToDatabaseAsync`
  return db.findUserInDatabaseAsync(user)
    .then(function(existingUser) {
      if (existingUser) {
        throw new Error('User already exists!') // Head straight to `catch`. Do not pass Go, do not collect $200
      } else {
        return user; // Return a synchronous value
      }
    })
    .then(function(newUser) {
      return db.hashPasswordAsync(newUser) // Return a promise
    })
    .then(function(securedUser) {
      return db.createAndSaveUserAsync(securedUser) // Return another promise
    })
}
```
* Tinker with the example inside examples/chaining.js to build intuition around promise chaining
* Complete the exercises in bare_minimum/basicChaining.js and make the Basic chaining tests pass

## Resources
[The Promise Cookbook](https://github.com/mattdesl/promise-cookbook)
