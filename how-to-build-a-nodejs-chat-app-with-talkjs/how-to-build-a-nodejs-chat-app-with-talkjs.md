# How to build a Node.js chat app with TalkJS

In this tutorial, we'll show you how to integrate the [TalkJS](https://talkjs.com/) chat API with a Node.js backend server to build a real-time chat application:

<!-- TODO: Add 1-demo.jpg -->
<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Chatbox with a conversation between two users"/>
</figure>

Node.js lets you run JavaScript outside of web browsers, for example when building web servers and other backend tools. It's a popular choice for web developers because it lets you use the same language for both frontend and backend development.

In this tutorial, you'll use an [Express](https://expressjs.com/) web server with Node.js to serve user data from a REST endpoint. You'll then use this data in your frontend to create TalkJS users with the [JavaScript SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/) and start a chat between them. To keep things simple, this tutorial will use a lightweight [LowDB database](https://github.com/typicode/lowdb), but you can swap this out for other database options.

If you'd rather get started with a working example, you can find the full source code for this project in our [GitHub examples repo](https://github.com/talkjs/talkjs-examples/tree/master/nodejs).

<div style="background-color: #EFF; padding: 15px;">

  <p><strong>Note:</strong> This is a simple example to help you get started. In a production setting, you will need to ensure that users are authenticated and authorized to use your service. You will also want to use a full-featured database, rather than a lightweight embedded one. </p>

</div>

## Contents

- [Prerequisites](#prerequisites)
- [Set up the backend server](#set-up-the-backend-server)
- [Create the API endpoints](#create-the-api-endpoints)
- [Create a frontend with a TalkJS chatbox](#create-a-frontend-with-a-talkjs-chatbox)

## Prerequisites

To follow along with this tutorial, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. You’ll use this to create your backend server.

## Set up the backend server

In this section, you'll install and initialize the dependencies you need for your backend server.

### Step 1: Create the project structure

Create a top-level directory (called for example `nodejs-example`), and add a `nodejs-backend` directory inside it.

Inside `talkjs-backend`, create the following `package.json` file:

```json
{
  "name": "integrating-a-nodejs-app-with-talkjs",
  "version": "1.0.0",
  "type": "module",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

This provides `npm` with the information it will need to start the application.

### Step 2: Install dependencies

Run the following command inside `talkjs-backend`:

```sh
npm install express cors lowdb
```

This installs three dependencies that you will need for this project:

- [Express](https://expressjs.com/) is the Node.js web application framework that you will use to create your server
- [LowDB](https://github.com/typicode/lowdb) is a local JSON database that will store your TalkJS user data
- The [`cors` package](https://www.npmjs.com/package/cors) enables [cross-origin resource sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), so that your backend server can handle requests from your frontend

Next, create a file called `server.js` and import your dependencies:

```js
import express from "express";
import cors from "cors";
import { LowSync, JSONFileSync } from "lowdb";
```

### Step 3: Set up LowDB

To get started with LowDB, it will be useful to have some example data ready to go. Inside `talkjs-backend`, create the following `users.json` file with some initial user data for your app:

```json
{
  "users": [
    {
      "id": "alice",
      "name": "Alice",
      "email": "alice@example.com",
      "photoUrl": "https://talkjs.com/new-web/avatar-7.jpg",
      "role": "default",
      "welcomeMessage": "Hi 👋"
    },
    {
      "id": "sebastian",
      "name": "Sebastian",
      "email": "sebastian@example.com",
      "photoUrl": "https://talkjs.com/new-web/avatar-2.jpg",
      "role": "default",
      "welcomeMessage": "Hello there!"
    }
  ]
}
```

Then add the following code to `server.js` to load this data in to a new LowDB database:

```js
const adapter = new JSONFileSync("users.json");
const db = new LowSync(adapter);
db.read();
// Initialize with an empty array if users.json is empty or doesn't exist
db.data ||= { users: [] };
```

### Step 4: Set up Express

To create and initialize your Express server, add the following code to `server.js`:

```js
const app = express();
const port = 3000;
app.use(cors());
app.use(express.json());

app.listen(port, () => console.log(`Server up and running on port ${port}`));
```

This code enables CORS so that your server will be able to handle requests from the frontend you create later, and sets up JSON request parsing to make it easier to work with data sent in requests from the frontend.

Now start your server by running the following command from `talkjs-backend`:

```sh
npm start
```

You should see the message "Server up and running on port 3000" in your console window.

## Create the API endpoints

You're now ready to create the server endpoints. In this section, you'll add a `/createUser` endpoint to create a user and add them to the database, and a `/getUser` endpoint to retrieve a user's data from the database given an ID.

### Step 5: add the "Create user" endpoint

Add the following code to your `server.js` file:

```js
app.post("/createUser", (req, res) => {
  const id = req.body.id;
  const name = req.body.name;
  const email = req.body.email;
  const photoUrl = req.body.photoUrl;
  const role = req.body.role;
  db.data.users.push({
    id: id,
    name: name,
    email: email,
    photoUrl: photoUrl,
    role: role,
  });
  db.write();
  res.status(200).send("User created successfully");
});
```

When this endpoint receives a POST request with user data, it writes it to the database file.

To test this out, restart your server. Then, from a terminal run the following [curl](https://curl.se/) command:

```sh
curl -X POST http://localhost:3000/createUser \
-H "Content-Type: application/json" \
-d '{
  "id": "nina",
  "name": "Nina",
  "email": "nina@example.com",
  "photoUrl": "https://talkjs.com/new-web/avatar-6.jpg",
  "role": "default",
  "welcomeMessage": "Hi, how can I help?"
}'
```

Alternatively, use an API client like [Postman](https://www.postman.com/) to call the endpoint.

You should see a third user added to `users.json`:

```json
{
  "users": [
    // ... existing users ...
    {
      "id": "nina",
      "name": "Nina",
      "email": "nina@example.com",
      "photoUrl": "https://talkjs.com/new-web/avatar-6.jpg",
      "role": "default",
      "welcomeMessage": "Hi, how can I help?"
    }
  ]
}
```

### Step 6: Add the "Get user" endpoint

Next, add the following code to `server.js`:

```js
app.get("/getUser/:id", (req, res) => {
  const id = req.params.id;
  let record = db.data.users.find((p) => p.id == id);
  if (!record) {
    console.log("No data found!");
  } else {
    console.log("== Record found ==");
    console.log(record);
    res.status(200).send(record);
  }
});
```

When, for example, the `/getUser/sebastian` endpoint receives a GET request, it retrieves the data for a user with an id of `sebastian` from the database, and sends a JSON response with the user data to the frontend.

You can test this with another `curl` command:

```sh
curl http://localhost:3000/getUser/sebastian
```

You should receive a JSON response with the following user data:

```json
{
  "id": "sebastian",
  "name": "Sebastian",
  "email": "sebastian@example.com",
  "photoUrl": "https://talkjs.com/new-web/avatar-2.jpg",
  "role": "default",
  "welcomeMessage": "Hello there!"
}
```

## Create a frontend with a TalkJS chatbox

In this section, you'll create a frontend chat application that displays a TalkJS [chatbox](/Features/Chat_UI_Modes/The_Chatbox/) with a [1-to-1 chat](https://talkjs.com/use-cases/1-on-1-chat/) between two users, similar to the one in our [getting started guide](/Getting_Started/JavaScript_SDK/1_On_1_Chat/). The main difference is that you'll call the `/getUser` server endpoint that you created in the previous section to get the data for the two users.

### Step 7: Create the chatbox

In your top-level project directory, create a new `talkjs-frontend` directory and add the following `index.html` file:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>TalkJS with NodeJS example</title>
    <meta name="description" content="" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>

  <body>
    <!-- prettier-ignore -->
    <script>
      (function(t,a,l,k,j,s){
      s=a.createElement('script');s.async=1;s.src='https://cdn.talkjs.com/talk.js';a.head.appendChild(s)
      ;k=t.Promise;t.Talk={v:3,ready:{then:function(f){if(k)return new k(function(r,e){l.push([f,r,e])});l
      .push([f])},catch:function(){return k&&new k()},c:l}};})(window,document,[]);
    </script>
    <script src="script.js"></script>
    <!-- container element in which TalkJS will display a chat UI -->
    <div id="talkjs-container" style="width: 90%; margin: 30px; height: 500px">
      <i>Loading chat...</i>
    </div>
  </body>
</html>
```

This HTML file loads TalkJS along with a `script.js` file (which you'll add below), and creates a container element that TalkJS will display the chatbox in.

Next, add the `script.js` file inside `talkjs-frontend` and add the following:

```js
const getUser = async (id) => {
  const response = await fetch(`http://127.0.0.1:3000/getUser/${id}`);
  const data = await response.json();
  let user = new Talk.User({
    id: data.id,
    name: data.name,
    photoUrl: data.photoUrl,
    email: data.email,
    role: data.role,
  });
  return user;
};
```

This function calls the `/getUser` server endpoint for a given id, and uses the returned data to create a [TalkJS user](https://talkjs.com/docs/Reference/Concepts/Users/).

Now add the following code to `script.js`:

```js
(async function () {
  await Talk.ready;
  const alice = await getUser("alice");
  const sebastian = await getUser("sebastian");
  const session = new Talk.Session({
    appId: "<APP_ID>", // replace with your app ID
    me: sebastian,
  });
  const conversation = session.getOrCreateConversation(
    "nodeJSExampleConversation"
  );
  conversation.setAttributes({
    welcomeMessages: [
      "Example chat for our Node.js tutorial. Try sending a message!",
    ],
  });
  conversation.setParticipant(alice);
  conversation.setParticipant(sebastian);

  const chatbox = session.createChatbox(conversation);
  chatbox.select(conversation);
  chatbox.mount(document.getElementById("talkjs-container"));
})();
```

This code uses the `getUser` function to create two TalkJS users with data it retrieves from the backend server. It then creates a new [conversation](/Reference/Concepts/Conversations/) and displays it in a chatbox.

You'll need to replace your app ID with the value found in the **Settings** tab of your [TalkJS dashboard](https://talkjs.com/dashboard/login).

Open the `index.html` file in your browser. You should now see a chatbox with a welcome message from Alice:

<!-- TODO: Add 2-chatbox.jpg -->
<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Chatbox with Sebastian's view of the conversation with Alice"/>
</figure>

Try sending a message! You can also try replacing `me: sebastian` with `me: alice` in `script.js` to see Alice's side of the conversation. Or add your third user, `nina`, to create a [group chat](https://talkjs.com/use-cases/group-chat/).

## Conclusion

You now have a working demonstration of how to use TalkJS together with a Node.js backend to create a real-time JavaScript chat application. To recap, you've

- Set up a backend server and lightweight database
- Created server endpoints for getting and creating user data
- Created a frontend application with a TalkJS chatbox that gets user data from the backend server

There are many ways you could extend this project. You could [add more users](/Features/Group_Chats/) to the chat, try an [inbox](/Features/Chat_UI_Modes/The_Inbox/) or [popup](/Features/Chat_UI_Modes/The_Popup_Widget/) chat UI, edit your [theme](/Features/Themes/The_Theme_Editor/) or experiment with other TalkJS features like [action buttons](https://talkjs.com/docs/Features/Customizations/Action_Buttons_Links/) or [message filters](/Features/Customizations/Message_Filters/).

You may also want to extend the frontend logic to call the `/createUser` server endpoint as well – for example, you could add a form that creates a new user, and then add them to the conversation.

If you want to learn more about TalkJS, our [docs](https://talkjs.com/docs/) and [tutorials](https://talkjs.com/resources/tag/tutorials/) have you covered. Or try our [demos](https://talkjs.com/demo/) to see TalkJS features in action.

If you're ready to get started with building your chat app, [try TalkJS for free](https://talkjs.com/dashboard/signup/premium/).
