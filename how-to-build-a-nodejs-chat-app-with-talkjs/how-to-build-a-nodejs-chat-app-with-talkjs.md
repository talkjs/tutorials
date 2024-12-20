# How to build a Node.js chat app with TalkJS

In this tutorial, we'll show you how to integrate the [TalkJS](https://talkjs.com/) chat API with a Node.js backend to build a chat app. We'll use [Express](https://expressjs.com/) to create a web server that serves user data from a REST endpoint, and then use this data in our frontend to create TalkJS users and start a chat between them. To keep things simple, we will store data in a lightweight [LowDB database](https://github.com/typicode/lowdb), but you can expand it to other popular database options.

You can find the entire source code for this project on [GitHub](https://github.com/talkjs/talkjs-examples/tree/master/nodejs).

**NOTE:** This is a barebones example of integration and is to help you get started. When using in a production setting, ensure that users are authenticated and authorized to use the services. We are using an embedded database, which is only for the scope of this tutorial. For production scenarios, you will want to use a full-fledged database.

## **Installing the dependencies**

To create an empty npm project, use the command **npm init -y**. The argument -y sets defaults for the parameters inside `package.json`. Once created, you can start downloading the dependencies. Make sure you add the parameter “type”:”module” inside the package.json file to use ES6 style import statements.

We have 4 dependencies that we need for this demo project:

- `express` is our go-to choice for creating APIs with NodeJS
- `body-parser` automatically parses your requests so that it becomes easy to handle them in your code
- `lowDB` is a local JSON database.
- `cors` is to enable cross-origin resource sharing.

After installing them, create a file called **server.js** and import them into the application:

```code
import express from 'express';
import cors from 'cors';
import bodyParser from 'body-parser';
import { LowSync, JSONFileSync } from 'lowdb';
```

**LowDB – Easy to use, embedded database for JavaScript**

LowDB is an open-source embedded database for JavaScript, and their [GitHub page](https://github.com/typicode/lowdb) has very comprehensive documentation on how to get started with examples.

```code
const adapter = new JSONFileSync('file.json');
const db = new LowSync(adapter);
db.read();
db.data ||= { users: [] };
```

To create a simple JSON database, we use the JSONFileSync function and pass in the required filename. If it is not present, LowDB creates it for you. We then pass that to the LowSync method to get an instance of the database in memory. Note that the Sync in the functions means synchronous. There are also asynchronous variants of these functions. By default, we create an empty array of users inside the database.

## **Creating the API endpoints**

Before creating the API endpoints, we must initialize our Express application and configure `body-parser`. For that, we use the below code:

```code
const app = express();
const port = 3000;
app.use(cors());

app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());
```

Once that is set, we are good to start creating our endpoints. We'll make a `/createUser` endpoint for creating the user and a `/getUser` endpoint to retrieve a user's data given their ID.

### Create User API

Creating a user is a POST request. We receive all the parameters from the request body and set it to variables. To make it shorter, we can directly assign them when we push it to the database as well.

```
app.post('/createUser', (req, res) => {
    const id = req.body.id;
    const name = req.body.name;
    const email = req.body.email;
    const photoUrl = req.body.photoUrl;
    const role = req.body.role;
    console.log(id, name, email, photoUrl, role);
    db.data.users.push({
        id: id,
        name: name,
        email: email,
        photoUrl: photoUrl,
        role: role,
    });
    db.write();
    res.status(200).send('User created successfully');
});
```

Once all the data is ready, we use the statement, **db.data.users.push** and pass in an object with the values. This does not persist in the file, so we finally need to use the **db.write()** method.

### Get User API

This is a GET request to retrieve the user's data. We pass in the id in the URL and then retrieve the record from our JSON file with the id. We use the **find** method and pass in an arrow function to go through each entry and retrieve the record.

```code
app.get('/getUser/:id', (req, res) => {
    const id = req.params.id;
    let record = db.data.users.find(p => p.id === id);
    if (!record) {
    	console.log("No data found!");
    } else {
        console.log("== Record found ==");
        console.log(record);
        res.status(200).send(record);
    }
});
```

Once retrieved, we send the record to the frontend, where it displays the user information in the chat.

## Setting up the frontend application

Now we'll create the frontend, following the TalkJS [getting started guide](/Getting_Started/JavaScript_SDK/1_On_1_Chat/) to set up a [](/Reference/JavaScript_Chat_SDK/Inbox/)[inbox](/Features/Chat_UI_Modes/The_Inbox/). Make changes to the code as follows:

```javascript
(async function () {
  await Talk.ready;
  let agent = await getAgent();
  let user = await getUser();

  const session = new Talk.Session({
    appId: "<APP_ID>",
    me: user,
  });
  var conversation = session.getOrCreateConversation(
    Talk.oneOnOneId(user, agent)
  );

  conversation.setAttributes({
    welcomeMessages: [
      "You can start typing your message here and one of our agents will be with you shortly.",
      "Please do not divulge any of your personal information.",
    ],
  });
  conversation.setParticipant(user);
  conversation.setParticipant(agent);

  var inbox = session.createInbox(conversation);
  inbox.select(conversation);
  inbox.mount(document.getElementById("talkjs-container"));
})();
```

You will need to replace your app ID with with the value found in the **Settings** tab of your [TalkJS dashboard](https://talkjs.com/dashboard/login).

Note that we use two asynchronous functions here, `getAgent()` and `getUser()` to retrieve the agent and user data. Add the following functions above the existing code inside the script file.

```javascript
const getAgent = async () => {
  const response = await fetch("http://localhost:8080/getUser?userId=1");
  const data = await response.json();
  let agent = new Talk.User({
    id: data.id,
    name: data.name,
    photoUrl: data.dp,
    email: data.email,
    role: data.role,
  });
  return agent;
};
const getUser = async () => {
  const response = await fetch("http://localhost:8080/getUser?userId=2");
  const data = await response.json();
  let user = new Talk.User({
    id: data.id,
    name: data.name,
    photoUrl: data.dp,
    email: data.email,
    role: data.role,
  });
  return user;
};
```

Once we receive the user and agent data from our database through the API calls, we create `Talk.User` objects and map them to it. This will allow us to start a conversation with the users retrieved from the database.

## Conclusion

![](https://talkjs.com/resources/content/images/2022/03/unnamed.png)

We have successfully integrated a Node.js backend with TalkJS and retrieved user data from a LowDB database through API calls. Even though this is a very simple example that showcases the capabilities of TalkJS, you can expand it to any production scenario by implementing authentication, authorization, role-based dashboards, and more. The entire source code for this project is available on [GitHub](https://github.com/talkjs/talkjs-examples/tree/master/nodejs).
