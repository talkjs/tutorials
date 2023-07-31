# How to archive a conversation

When you're dealing with a cluttered inbox, it can be useful to have a way to archive old messages. This tutorial walks you through how to add an "Archive this conversation" menu option to your TalkJS inbox with [Conversation Actions](https://talkjs.com/docs/Features/Customizations/Conversation_Actions/), [webhooks](https://talkjs.com/docs/Reference/Webhooks/) and the [REST API](https://docs.google.com/document/d/1DGUW4zIyWSx_F128tXPfFheOkuJcGpAXyKFcN9K7n1k/edit#:~:text=09%3A37%20Today-,https%3A//talkjs.com/docs/Reference/REST_API/Getting_Started/Introduction/,-30).

To follow along, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing TalkJS project using the [JavaScript Chat SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/). See our [Getting Started guide](https://talkjs.com/docs/Getting_Started/) for an example of how to set this up.
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.

We’ll build up the feature step by step in the following sections. If you would rather see the complete example code,

!! see the [GitHub repo]() for this tutorial.

## Get notified about new messages

First we’ll enable TalkJS [webhooks](https://talkjs.com/docs/Reference/Webhooks/), which allow the TalkJS server to notify your server when a message is sent. Webhooks let you use an event-driven architecture, where you get told about events when they happen rather than having to constantly check for new messages. There are lots of events you can listen for, but we’re only interested in new messages being sent.

Webhooks are server-side only, so we’ll need a web server. We’ll be using [Express](https://expressjs.com/) in this tutorial, but feel free to use your favorite web server library instead:

```js
import express from "express";
const app = express().use(express.json()); // creates http server

app.listen(3000, () => console.log("Server is up"));

app.post("/addCustomData", (req, res) => {
  console.log(req.body);
  res.status(200).end();
});
```

This sets up a POST endpoint at `/addCustomData` that monitors incoming events from the TalkJS server. In the next section, we'll use this to add a custom `archived` field to the conversation.

For TalkJS to communicate with your server, you must expose it to the internet. This can be very difficult when developing locally, with endless firewalls and port forwarding to set up. Instead, we’ll use [ngrok](https://ngrok.com/) to create a secure tunnel to your local server. See our tutorial on [How to integrate ngrok with TalkJS](https://talkjs.com/resources/how-to-integrate-ngrok-with-talkjs-to-receive-webhooks-locally/#setting-up-ngrok) for instructions on how to install ngrok.

Once you have installed ngrok, run the following command:

```sh
ngrok http 3000
```

This command starts a secure tunnel to your local port 3000. The output should include the URL that ngrok exposes:

```sh
Forwarding                    https://<YOUR_SITE>.ngrok-free.app -> http://localhost:3000
```

You’re now ready to enable webhooks. You can do this in the TalkJS dashboard under **Webhooks**. Paste the URL above into the **Webhook URL** field, including the `/addCustomData` path: `https://<YOUR_SITE>.ngrok-free.app/addCustomData`.

Select the **message.sent** option:

!! 3-webhooks-ui.jpg - The list of webhook options in the TalkJS dashboard.

TalkJS will now send a web request to your server when a message is sent. To test this, write another message in your chat UI. You should see the event in your server’s console:

```sh
{
  createdAt: 1683276915840,
  data: {
    conversation: {
      createdAt: 1683275536667,
      custom: [Object],
      id: '15966c817cb1473d9b0a',
      // ... more fields here ...
    },
    message: {
      // ... message fields here ...
    },
    sender: {
      // ... sender fields here ...
    }
  },
  id: 'evt_AVL7tDG9V7CPSXBfG4',
  type: 'message.sent'
}
```

## Add a custom `archived` field

Next, we’ll react to the events by calling the [TalkJS REST API](https://talkjs.com/docs/Reference/REST_API/Getting_Started/Introduction/) to add custom data to conversations. We’ll use this to set an `archived: false` custom property on the conversation every time we receive a new message. (Later, we'll add an **Archive this conversation** menu option that sets this property to `true`.)

In this tutorial we will use the [node-fetch](https://github.com/node-fetch/node-fetch) module to send the HTTP requests, but you can use another library if you prefer:

```js
import fetch from "node-fetch";

const appId = "<APP_ID>";
const secretKey = "<SECRET_KEY>";

const basePath = "https://api.talkjs.com";
const path = basePath + "/v1/" + appId + "/conversations/";
```

This imports the `node-fetch` module and sets up the URLs you will need to call. Fill in the app ID and secret key with the values from your TalkJS dashboard. Make sure you protect that secret key and never expose it in frontend code – it has full admin access to your TalkJS account.

Next, we’ll use a PUT request to update the conversation data when a sent message event arrives at your local server. The conversation data includes a custom field which you can use to add an extra `archived` property. We'll set this to `false`, because we don't want conversations with new messages to be archived. Update your call to `app.post` to include the following:

```js
async function setArchived(conversationId, isOn) {
  console.log("Setting archived on", conversationId, "to", isOn);
  return fetch(`${basePath}/v1/${appId}/conversations/${conversationId}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${secretKey}`,
    },
    body: JSON.stringify({
      custom: {
        archived: isOn ? "true" : "false",
      },
    }),
  });
}

app.post("/addCustomData", async (req, res) => {
  const data = req.body.data;
  const conversationId = data.conversation.id;

  await setArchived(conversationId, false);
  res.status(200).end();
});
```

This now calls the `setArchived` function, which calls the REST API to add the custom `archived` property.

## Add an "Archive conversation" option to the menu

Next, we need add a way to trigger the archive action in the frontend. You can configure which actions a user is able to take based on their [user role](https://talkjs.com/docs/Reference/Concepts/Roles/). In this example we'll add a custom conversation action to the "default" role.

In the **Roles** tab of the TalkJS dashboard, select the "default" role option. In the **Custom conversation actions** section, add a new custom conversation action with a **Name** of "archive" and a **Label** of "Archive conversation". The name will be used in our code, while the label is what appears in the UI:

!! add 1-conversation-actions-dashboard.jpg

If you're using a preset theme, you should now see a new **Archive conversation** option in the menu at the top of your conversation:

!! add 2-archive-conversation-ui.jpg

If you're using a legacy theme, or customized a theme before the custom conversation actions feature was released, you'll need to add this menu to your theme before it will show up. See [our docs](https://talkjs.com/docs/Features/Customizations/Conversation_Actions/#the-action-menu-does-not-show-up) for details on how to edit the theme.

## Mark the conversation as archived

You can now listen for your new conversation action using the [`onCustomConversationAction` method](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/Chatbox/#Chatbox__onCustomConversationAction) in TalkJS's JavaScript SDK.

As a test, add the following to your TalkJS code:

```js
inbox.onCustomConversationAction("archive", (event) => {
  console.log("Archived conversation with id:", event.conversation.id);
});
```

You should now see the conversation ID logged to your browser console when you click the **Mute conversation** menu option.

Next, create a `/archiveConversation` endpoint in your server code to receive the conversation ID. You'll need to set up CORS support on your server. In this case we'll use the [`cors`](https://expressjs.com/en/resources/middleware/cors.html) package with Express:

```js
import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

app.listen(3000, () => console.log("Server is up"));

app.post("/archiveConversation", async (req, res) => {
  const conversationId = req.body.conversationId;

  await setArchived(conversationId, true);
  res.status(200).end();
});
```

Next, update

```js
inbox.onCustomConversationAction("archive", async (event) => {
  async function postConversationId() {
    // Send conversation id to your backend server
    const response = await fetch("http://localhost:3000/archiveConversation", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ conversationId: event.conversation.id }),
    });
  }

  await postConversationId();

  console.log("Archived conversation with id:", event.conversation.id);
});
```

then

```js
app.post("/archiveConversation", async (req, res) => {
  const conversationId = req.body.conversationId;

  await setArchived(conversationId, true);
  res.status(200).end();
});
```

!! more explanation...

## Filter the inbox

Finally, we want to filter conversations so that only unarchived ones appear in the inbox. To do this, use the setFeedFilter method.

Add the following line before mounting your inbox, to only show conversations where `custom.answered` is not `true`. This means that any pre-existing conversations that are missing the `archived` property will still appear:

```js
inbox.setFeedFilter({ custom: { answered: ["!=", "true"] } });
```

Now, test your archive feature by selecting **Archive a conversation** from the menu. The conversation should be removed from your inbox.

## Conclusion

You now have a working demonstration of how to archive a conversation! To recap, in this tutorial you have:

!!summarise

For the full example code for this tutorial,

!!see our [GitHub repo]().

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS Docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.
