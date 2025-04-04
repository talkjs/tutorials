# Create a custom chatbot with TalkJS and Gemini

Google's [Gemini](https://gemini.google.com/) is an AI chat service that can automatically provide context-aware responses to all kinds of queries. In this tutorial, we'll show you how to use the [Gemini API](https://ai.google.dev/gemini-api) along with TalkJS to create a customizable chatbot that integrates Gemini's responses seamlessly into your user chat.

We'll first show you how to use TalkJS [webhooks](https://talkjs.com/docs/Reference/Webhooks/) and Gemini's API to generate replies to user messages. We'll then use TalkJS's [REST API](https://talkjs.com/docs/Reference/REST_API/Getting_Started/Introduction/ to add Gemini's responses directly to your chat:

!! 1-demo.gif

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Chatbot demo. The user asks questions about TalkJS and gets replies generated by the Gemini API."/>
  <figcaption>Chatbot demo. The user asks questions about TalkJS and gets replies generated by the Gemini API.</figcaption>
</figure>

To follow along with this tutorial, you will need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing TalkJS project using the [JavaScript Chat SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/). See our [Getting Started guide](https://talkjs.com/docs/Getting_Started/) for an example of how to set this up.
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.
- A [Gemini API account](https://ai.google.dev/gemini-api) and [API key](https://aistudio.google.com/app/apikey).

We’ll build up the feature step by step in the following sections. If you would rather see the complete example code, see the [Github repo](https://github.com/talkjs/talkjs-examples/tree/master/chatbot-integration/google-gemini) for this tutorial.

## Set up your chat

First, set up a TalkJS chat in your frontend code. The details will depend on exactly how you integrate TalkJS into your current application, but for our example we'll use TalkJS's [JavaScript SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/) to create a conversation between a user and a bot, and display it with our [prebuilt chatbox UI](https://talkjs.com/docs/Features/Chat_UI_Modes/The_Chatbox/):

```js
Talk.ready.then(function () {
  const me = new Talk.User({
    id: "geminiExampleUser",
    name: "Alice",
    email: "alice@example.com",
    role: "default",
    photoUrl: "https://talkjs.com/images/avatar-1.jpg",
  });
  const talkSession = new Talk.Session({
    appId: "<APP_ID>", // replace with your TalkJS app ID
    me: me,
  });

  const bot = new Talk.User({
    id: "geminiExampleBot",
    name: "Bot 🤖",
    email: "bot@example.com",
    role: "default",
    photoUrl: "https://talkjs.com/new-web/talkjs-logo.svg",
    welcomeMessage:
      "Hi, I'm a friendly chatbot! I'll use the Gemini API to assist with your queries. How can I help?",
  });

  var conversation = talkSession.getOrCreateConversation(
    "geminiExampleConversation"
  );
  conversation.setParticipant(me);
  conversation.setParticipant(bot);

  const chatbox = talkSession.createChatbox();
  chatbox.select(conversation);
  chatbox.mount(document.getElementById("talkjs-container"));
});
```

We've included a `welcomeMessage` for the bot to start the conversation off.

## Get notified about new messages

We'll eventually want the bot to reply to any messages that the user sends in the conversation. To do this, we'll first enable TalkJS webhooks, which allow the TalkJS server to notify your server when a message is sent.

Webhooks let you use an event-driven architecture, where you get told about events when they happen rather than having to constantly check for new messages. There are lots of events you can listen for, but we’re only interested in new messages being sent.

Webhooks are server-side only, so you’ll need a web server. We’ll be using [Express](https://expressjs.com/) in this tutorial, but feel free to use your favorite web server library instead.

Add the following lines to your `package.json` directory (run `npm init` to create one if you don't have one already):

```json
{
  "type": "module",
  "scripts": {
    "start": "node server.js"
  }
}
```

Then install Express:

```sh
npm install express
```

Next, create a `server.js` file with the following code:

```js
import express from "express";
const app = express().use(express.json()); // creates http server

app.listen(3000, () => console.log("Server is up"));

app.post("/onMessageSent", (req, res) => {
  console.log(req.body);
  res.status(200).end();
});
```

This sets up an Express server with a POST endpoint at `/onMessageSent` that logs incoming events from the TalkJS server to the terminal. To start the server, run:

```sh
npm start
```

For TalkJS to communicate with your server, you must expose it to the internet. This can be very difficult when developing locally, with endless firewalls and port forwarding to set up. Instead, we’ll use [ngrok](https://ngrok.com/) to create a secure tunnel to your local server. See our tutorial on [How to integrate ngrok with TalkJS](https://talkjs.com/resources/how-to-integrate-ngrok-with-talkjs-to-receive-webhooks-locally/#setting-up-ngrok) for instructions on how to install ngrok.

Once you have installed ngrok, run the following command:

```sh
ngrok http 3000
```

This command starts a secure tunnel to your local port 3000. The output should include the URL that ngrok exposes:

```sh
Forwarding                    https://<YOUR_SITE>.ngrok-free.app -> http://localhost:3000
```

You’re now ready to enable webhooks. You can do this in the **Settings** section of the TalkJS dashboard, under **Webhooks**. Paste the ngrok URL into the **Webhook URL** field, including the `/onMessageSent` path: https://<YOUR_SITE>.ngrok-free.app/talkjs.

Select the **message.sent** option:

!! 2-webhooks-ui.jpg

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="The list of webhook options in the TalkJS dashboard."/>
  <figcaption>The list of webhook options in the TalkJS dashboard.</figcaption>
</figure>

TalkJS will now send a web request to your server when a message is sent. To test this, write a message in your chat UI. You should see the event in your server’s console.

## Call the Gemini API

Next, we'll call the Gemini API to [create an interactive chat](https://ai.google.dev/gemini-api/docs/text-generation?lang=node#chat).

The API call uses a secret key, so you'll need to call it from your backend server to avoid exposing the key.

First, install the `@google/generative-ai` package:

```shell
npm install @google/generative-ai
```

Then, as a first test of the API, add the following to your server code:

```js
import { GoogleGenerativeAI } from "@google/generative-ai";

const geminiApiKey = "<GEMINI_SECRET_KEY>";

const genAI = new GoogleGenerativeAI(geminiApiKey);
const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

async function getReply(messageHistory, messageText) {
  const chat = model.startChat({
    history: messageHistory,
    generationConfig: {
      maxOutputTokens: 1024,
    },
  });

  const result = await chat.sendMessage(messageText);
  const response = await result.response;
  return response.text();
}

const messageHistory = [
  {
    role: "user",
    parts: [
      {
        text: "You are a helpful assistant. Please provide short, concise answers.",
      },
    ],
  },
];
const messageText = "What is TalkJS?";

const reply = await getReply(messageHistory, messageText);

console.log(reply);
```

This code calls Gemini's [`startChat`](https://cloud.google.com/vertex-ai/generative-ai/docs/reference/nodejs/latest/vertexai/generativemodel#_google_cloud_vertexai_GenerativeModel_startChat_member_1_) method to start a chat with the given message history. It then calls the [`sendMessage`](https://cloud.google.com/vertex-ai/generative-ai/docs/reference/nodejs/latest/vertexai/chatsession#_google_cloud_vertexai_ChatSession_sendMessage_member_1_) method on the chat with the text of the message we want Gemini to reply to.

In this tutorial, our initial message history will consist of one user message that gives basic instructions on the type of replies we'd like to receive. You can edit this message to give Gemini any context you want to provide for the chat.

We also need to select an AI model to generate the reply. In this example, we use the Gemini 1.5 Flash model, `gemini-1.5-flash`. You can switch this out for a different model if you like. See Gemini's [list of models](https://ai.google.dev/gemini-api/docs/models/gemini) for more choices.

Replace `<GEMINI_SECRET_KEY>` with your API key and run the above code with `npm start`. You should see a reply (something like `TalkJS is a service that makes it easy to add real-time chat to websites and apps. `) logged to the console.

Next, we want Gemini to reply to user messages typed in the chat, instead of a fixed `messageText`. Delete the code below the `getReply` function in the above snippet and update your `app.post` endpoint to call `getReply`:

```js
const botId = "chatbotExampleBot";

app.post("/onMessageSent", async (req, res) => {
  const convId = req.body.data.conversation.id;
  const messageText = req.body.data.message.text;
  const senderId = req.body.data.sender.id;

  if (!(convId in allMessageHistory)) {
    allMessageHistory[convId] = [
      {
        role: "user",
        parts: [
          {
            text: "You are a helpful assistant. Please provide short, concise answers.",
          },
        ],
      },
    ];
  }
  const messageHistory = allMessageHistory[convId];

  if (senderId != botId) {
    const reply = await getReply(messageHistory, messageText);
  }

  res.status(200).end();
});
```

Here we've created a message history for each conversation, and passed it to `getReply` along with the current message text. We do not call the Gemini API for messages from the bot, or it will get stuck in a loop of responding to itself!

Restart your server, and send another message. You should see a relevant reply from Gemini logged to your terminal.

## Call the TalkJS API

The final step is to display the bot's replies as messages in the chatbox. To do this, create a function to call the TalkJS REST API to [send messages](https://talkjs.com/docs/Reference/REST_API/Messages/#sending-on-behalf-of-a-user) from the bot:

```js
const appId = "<APP_ID>";
const talkJSSecretKey = "<TALKJS_SECRET_KEY>";
const basePath = "https://api.talkjs.com";
const conversationId = "geminiExampleConversation";

async function sendTalkJSMessage(text) {
  return fetch(
    `${basePath}/v1/${appId}/conversations/${conversationId}/messages`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${talkJSSecretKey}`,
      },
      body: JSON.stringify([
        {
          text: text,
          sender: botId,
          type: "UserMessage",
        },
      ]),
    }
  );
}
```

As with the Gemini API, you should call the TalkJS API from your backend server to avoid exposing your secret key. You can find your secret key along with your app ID in the [TalkJS dashboard](https://talkjs.com/dashboard/login) under **Settings**.

Now you can call the `sendTalkJSMessage` function after `getReply` in your `app.post` endpoint:

```js
app.post("/onMessageSent", async (req, res) => {
  // ...

  if (senderId != botId) {
    const reply = await getReply(messageHistory, messageText);
    await sendTalkJSMessage(convId, reply);
  }

  res.status(200).end();
});
```

The responses from Gemini now appear in the chatbox as messages from the bot:

!! 3-chat.jpg

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Chatbox with an example question and reply."/>
  <figcaption>Chatbox with an example question and reply.</figcaption>
</figure>

## Customize your chat

You now have a working demonstration of how to integrate a Gemini-powered chatbot into your TalkJS chat!

You may want to customize your chat further. For example, you could include an indicator in your chat that the bot is generating a response. See our blog post on [How to add a custom typing indicator for a chatbot](https://talkjs.com/resources/how-to-add-a-custom-typing-indicator-for-a-chatbot/) for information on ways to do this.

You can also use all our other features, such as [custom themes](https://talkjs.com/docs/Features/Themes/), [action buttons](https://talkjs.com/docs/Features/Customizations/Action_Buttons_Links/) and [HTML Panels](https://talkjs.com/docs/Features/Customizations/HTML_Panels/), to control exactly how you want the chat to look and behave.

## Summary

To recap, in this tutorial we have:

- Created a chat between a user and a bot
- Set up a web server to receive webhook events from the TalkJS server when new messages are sent
- Called the Gemini API to generate replies to new user messages
- Called the TalkJS API to send the replies as messages in the chat

For the full example code for this tutorial, see [our Github repo](https://github.com/talkjs/talkjs-examples/tree/master/chatbot-integration/google-gemini).

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.
