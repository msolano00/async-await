# Async/Await

Async/Await is a JavaScript **pattern** that, as it name implies, allows asynchronous operations to be structured in a syntactical synchronous way

## Callbacks and Promises

In order to understand the Async/Await pattern (or at least why it was implemented in ES2017).

### Callbacks

A callback is a function that is passed into another function as an argument. This callback function will be invoked once a certain condition is met.

As an example for this blog we are going to simulate an HTTP request that has a 1 second response time and then some code being executed once that response is fulfilled:

```javascript
/**
* greet
* Greeter function, prints a string
*/

function  greet(name)  {
  console.log(`Hi  ${name}, I am a callback function!`);
}

/**
* demo
* Function that triggers a callback with a setTimeout
* @param {object}  callback  - The callback function to be executed by the setTimeout
*/

function  demo(callback)  {
  setTimeout(()  =>  {
    callback('Nebula user');
  },  1000);
}

/**
* main
* Main function, executes the program
*/
function  main()  {
  demo(callback);  // Executes the callback after 1 second
}

main();
```

The problem with this pattern is that, due to JavaScript asynchronous nature, we might end up having nested callbacks, lots of them (what we call *callback hell*):

```javascript
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```
So, how do we fix nested callbacks? Let's introduce Promises.

### Promises

Promises are objects that act as a proxy for a value that is not known when the promise is created. A Promise has different states that represent the status of the asynchronous operation, these states are:
-   **Pending**: initial state, neither fulfilled nor rejected
-   **Fulfilled**: meaning that the operation was completed successfully
-   **Rejected**: meaning that the operation failed

A pending Promise will be settled to either of the other states. Fulfilled with a value or rejected with an error.

The Promise object contains three different methods `promise.then()`, `promise.catch()` and `promise.finally()`. The **then()** method allows developers to pass a callback function that will be executed when the Promise is fulfilled. The **catch()** method allows developers to pass a callback function that will handle errors. The **finally()** method allows the developers to pass a callback function that gets executed once the Promise is settled regardless of its status (fulfilled or rejected). The main idea behind the **finally()** method is to avoid duplicating code between both of the previous methods. All three methods return another Promise that allows the developer to chain Promises to each other, making the code more organized and readable.

Let's refactor our example using promises:

```javascript
/**
* greet
* Greeter function, prints a string
*/
function greet(name) {
  console.log(`Hi ${name}, I am a callback function!`);
}
 
/**
* demo
* Function that triggers a callback with a setTimeout
* @return {Promise} A promise.
*/
function demo() {
  return new Promise((resolve, reject) => {
    try {
      setTimeout(() => {
        resolve('Nebula');
      }, 1000);
    } catch (error) {
      reject(new Error(error));
    }
  });
}
 
/**
* main
* Main function, executes the program
*/
function main() {
  demo()
    .then(resolved => console.log(callback(resolved)))
    .catch(error => console.error(error));
}
 
main();
```

If you are thinking that the code has become more verbose, indeed it has, but now the code is more structured, clean and readable. For some people the chaining syntax might look a little unnatural. Some people are used to the synchronous way of writing code and because of this is that we introduce **Async/Await**.

## Async/Await

ES2017 introduces the Async/Await pattern to the JavaScript language. Async/Await removes the need of chaining operations by introducing two new keywords, **async** and **await**. Let's take a look into each keyword and how they work together.  

### Await

The **await** operator is used to wait for an asynchronous operation to be done. 

### Async

The **async** keyword declares that a function is asynchronous and allows the usage of the **await** operator inside itself. An asynchronous function will wait until the awaited variable (a **Promise**) is fulfilled or rejected and then executes the next line of code, avoiding the **Promise** chaining. 

Let's refactor our **Promise** example using **Async/Await**:

```javascript
/**
* greet
* Greeter function, prints a string
*/
function greet(name) {
  console.log(`Hi ${name}, I am a callback function!`);
}
 
/**
* demo
* Function that triggers a callback with a setTimeout
* @return {Promise} A promise.
*/
function demo() {
  return new Promise((resolve, reject) => {
    try {
      setTimeout(() => {
        resolve('Nebula');
      }, 1000);
    } catch (error) {
      reject(new Error(error));
    }
  });
}
 
/**
* main
* Main function, executes the program
*/
async function main() {
  try {
    const name = await demo();
    greet(name);
  } catch (error) {
    console.error(error)
  }
}
 
main();
```
Our main function is not only clean and readable but also much more intuitive!

This concludes the post, I hope this helps you understand the nature behind **Async/Await**.

## Additional Examples

```javascript
const axios = require('axios');
 
// No nested callbacks but still a little verbose
const chainedGetData = (req, res) => {
  axios.get(`https://some/url.com`)
  .then(response => {
    const user = response.data.user;
    return axios.get(`https://some/other/url/${user}`);
  })
  .then(response => {
    res.status(200).send(response.data);
  })
  .catch(error => {
    console.error(error);
    res.status(500).send(error);
  })
};
 
// Much cleaner and more intuitive to read
const cleanerGetData = async (req, res) => {
  try {
    const user = await axios.get(`https://some/url.com`);
    const response = await axios.get(`https://some/other/url/${user.data.username}`);
    res.status(200).send(response.data);
  } catch (error) {
    console.error(error);
    res.status(500).send(error);
  }
};
```