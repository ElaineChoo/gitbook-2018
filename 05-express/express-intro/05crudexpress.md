# CRUD in Express

We can now get things from the disk and display them as an HTML page to the user.

Next we want to take input from the user that will result in data being saved on our disk.

This is part of CRUD (Create, Read, Update, Destroy).

[Formal definition on wiki](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete)

---

We will need several pieces to do this:
1. an HTML form
1. a route and response callback that takes that data
1. a place to store that data
1. and, an HTML page to show the user

First, let's talk about how to send data from the browser to the server.
---

### HTTP Post

#### Demo: How a user sends data over the internet: HTTP POST
Real-life recreation with paper. (One person pretends to be the backend)
- Prepare a request in the browser. verbs we are using: POST
- Send the request to a server
- server recieves the response
- reads the headers
- looks at the contents of the request
- constructs a response
- sends the response
- browser is waiting for the response
- Response has status and status code on the envelope.
- Deal with the server response
- If it's good, display the output to the user
- If it's bad, do nothing, or display an error to the user

---

Important Points:
- the parameters for the request are in the headers, the data itself is in the body of the request (inside the envelope)
- the point of a post request isn't always to save something, but a request where something is saved is almost always a post request

---

### How to make a POST request in the browser
- one of the main default behaviors of HTML in the browser is that `<a>` tags create `GET` requests and `form` tags create POST requests.

This behavior is always a default behavior given to us by the browser.

Given:
```
<form method="POST" action="/animals">
  Animal Name:
  <input type="text" name="name">
  <input type="submit" value="Submit">
</form>
```

When the submit button is clicked, this form will produce a POST request to the server.

---


#### RESTful Routing

RESTful routing is a scheme to structure your URLS that removes duplication, and looks clean. For each type of resource, you can specify a set of routes easily.

We don't need to rename the route animals, because express will separate things out depending on the HTTP method we use.

| **URL** | **HTTP Verb** |  **Action**|
|------------|-------------|------------|
| /photos/         | GET       | index
| /photos/new      | GET       | new
| /photos          | POST      | create
| /photos/:id      | GET       | show
| /photos/:id/edit | GET       | edit
| /photos/:id      | PATCH/PUT | update
| /photos/:id      | DELETE    | destroy


[Read more on wiki](http://en.wikipedia.org/wiki/Representational_state_transfer)

---

## CRUD in action

To receive this data we need to create a `POST` route in express and the `body-parser` npm module.

---


### BodyParser

Parsing parameters from a form needs an external module called `body-parser`.

```bash
yarn add body-parser
```

---

**index.js**
```js
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

// tell your app to use the module
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: true
}));
```
---

Note that we set an attribute `extended` to `true` when telling our app to use the body parser. This attribute determines which library is used to parse data. Discussion on extended [here](http://stackoverflow.com/questions/29175465/body-parser-extended-option-qs-vs-querystring).

Now, if we try to add this backend route, calling `req.body` should contain the form input.

---

### MethodOverride

HTML `<form>`s do no yet support PUT and DELETE requests, so we need a way to circumvent the problem.

The `method-override` npm library is designed to get over this limitation.

You will need to install the method-override package using `yarn add method-override` and make sure Express uses it.

**index.js**

```js
// Set up method-override for PUT and DELETE forms
app.use(methodOverride('_method'));
```

Then, in your handlebars HTML page, you will have to specify `_method=<VERB>`, where VERB can be either PUT or DELETE. Here is an example of PUT.

**edit.handlebars**

```html
<form method="POST" action="/{{pokemon.id}}?_method=PUT">
  <div class="pokemon-attribute">
    id: <input name="id" type="text" value="{{pokemon.id}}"/>
  </div>
</form>
```

For more examples and details, as usual, read the [method-override documentation](https://www.npmjs.com/package/method-override).

---

**backend - express route**
```js
app.post('/animals', function(req, res) {
  //debug code (output request body)
  console.log(req.body);
  res.send(req.body);
});
```

---

### Pairing Exercise

create a new dir
```
mkdir post-exercise
```
cd into it
```
cd post-exercise
```
init yarn
```
yarn init
```
add express
```
yarn add express
```
add jsonfile
```
yarn add jsonfile
```
create an empty data.json file
```
echo {} >> data.json
```
start your app
```
nodemon index.js
```

use this express starter code:

```js
const jsonfile = require('jsonfile');
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

// tell your app to use the module
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: true
}));

app.post('/animals', function(request, response) {

  //debug code (output request body)
  console.log(request.body);


  // save the request body
  jsonfile.writeFile('data.json', request.body, (err) => {
    console.error(err)

    // now look inside your json file
    response.send(request.body);
  });
});

app.get('/', (request, response) => {
  // render a handlebars template form here
  response.send("hello world");
});

app.listen(3000, () => console.log('~~~ Tuning in to the waves of port 3000 ~~~'));
```

use a CURL command to make a POST request to your server
```
curl -d "monkey=banana&koala=eucalyptus" -X POST http://localhost:3000/animals
```

#### write the html form in a handlebars template to make the post request

add handlebars to your app
```
yarn add handelbars-express
```

add a views directory
```
mkdir views
```

add the handlebars config into your express app
```
app.engine('handlebars', handlebars.create().engine);
app.set('view engine', 'handlebars');
```
- create your form in a handlebars file in the views directory
- set the action of the form to the path of the POST route

- create a GET route request handler in your express app

- try out your form
