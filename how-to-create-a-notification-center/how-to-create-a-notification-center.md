# How to create a message center with TalkJS

To keep your customers informed about important updates, it's useful to include a message center in your application. For example, you may want customers to receive an in-app message if your terms and conditions change, or to let them know that they've received new documents.

Although TalkJS is best known as a chat API, it's a [great fit for customer updates and notifications](https://talkjs.com/use-cases/message-center/) as well. After all, a customer update is a lot like a chat conversation, except that messages are only sent in one direction, from the company to the customer.

<figure class="kg-image-card">
  <img class="kg-image" src="https://talkjs.com/resources/content/images/2023/08/1-demo.jpg" alt="An example message center"/>
  <figcaption>An example message center</figcaption>
</figure>

TalkJS has built-in support for email and SMS notifications, so users can still receive messages when they are not logged in to your application.

In this tutorial, we'll walk you through how to send messages with the TalkJS [REST API](https://talkjs.com/docs/Reference/REST_API/Getting_Started/Introduction/) and customize your inbox for read-only messages.

To follow along, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing TalkJS project using the [JavaScript Chat SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/). See our [Getting Started guide](https://talkjs.com/docs/Getting_Started/) for an example of how to set this up.
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.

We’ll build up the feature step by step in the following sections. If you would rather see the complete example code, see the [Github repo](https://github.com/talkjs/talkjs-examples/tree/master/howtos/how-to-create-a-message-center) for this tutorial.

## Create a conversation with the REST API

The first step is to create a [conversation](https://talkjs.com/docs/Reference/Concepts/Conversations/) between the message sender and the customer who will receive it. In this tutorial, we'll create a new conversation for each message. To display the conversations we'll use TalkJS's [Inbox UI mode](https://talkjs.com/docs/Features/Chat_UI_Modes/The_Inbox/), with chat history on the left and the current selected conversation on the right, as in the demo image above.

Another option would be to build a message center based on the [Chatbox UI mode](https://talkjs.com/docs/Features/Chat_UI_Modes/The_Chatbox/), which displays a single conversation:

<figure class="kg-image-card">
  <img class="kg-image" src="https://talkjs.com/resources/content/images/2023/08/2-chatbox-demo.jpg" alt="An example message center built with the Chatbox UI mode"/>
  <figcaption>An example message center built with the Chatbox UI mode</figcaption>
</figure>

In this case you would follow the same steps as for an inbox, but add all messages to a single conversation.

To create the conversation, we'll use TalkJS's [REST API](https://talkjs.com/docs/Reference/REST_API/Getting_Started/Introduction/). The REST API needs a secret API key, which has full admin access to your TalkJS account, so we'll need a backend to call it from.

In this tutorial we will use the [`node-fetch` module](https://github.com/node-fetch/node-fetch) to send the HTTP requests to the API, but you can use another library if you prefer. Import `node-fetch` at the start of your backend code:

```js
import fetch from "node-fetch";
```

Next, we'll call the [create a conversation](https://talkjs.com/docs/Reference/REST_API/Conversations/#setting-conversation-data) endpoint. We'll also set a subject for the conversation, which will be the title for the message.

```js
const appId = "<APP_ID>";
const secretKey = "<SECRET_KEY>";
const basePath = "https://api.talkjs.com";

const conversationId = `messageCenterExample`;
const senderId = `messageCenterExampleSender`;
const receiverId = `messageCenterExampleReceiver`;
const conversationSubject = "Welcome to TalkJS 👋";

// Create a new conversation
await fetch(`${basePath}/v1/${appId}/conversations/${conversationId}`, {
  method: "put",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${secretKey}`,
  },
  body: JSON.stringify({
    participants: [receiverId, senderId],
    subject: conversationSubject,
  }),
});
```

You can find your App ID and secret key under **Settings** in the TalkJS dashboard.

## Send a message

We can now test sending a message. For this, we'll [add a message](https://talkjs.com/docs/Reference/REST_API/Messages/#sending-on-behalf-of-a-user) to the conversation with the REST API:

```js
const conversationId = `messageCenterExample`;
const messageText =
  "Check out our <https://talkjs.com/docs/Getting_Started/|Getting Started guide>!";

// Send a message from the user to make sure it will show up in the conversation list
await fetch(
  `${basePath}/v1/${appId}/conversations/${conversationId}/messages`,
  {
    method: "post",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${secretKey}`,
    },
    body: JSON.stringify([
      {
        text: messageText,
        sender: senderId,
        type: "UserMessage",
      },
    ]),
  }
);
```

You should see the message in your inbox:

<figure class="kg-image-card">
  <img class="kg-image" src="https://talkjs.com/resources/content/images/2023/08/3-send-message.jpg" alt="An example message linking to the TalkJS Getting Started guide"/>
  <figcaption>An example message</figcaption>
</figure>

You can include links in messages that you send from the REST API with TalkJS's [link markup](https://talkjs.com/docs/Features/Customizations/Formatting/#links). This is useful when you want to send a message to view information elsewhere.

Often, you'll want to send the same message to multiple users at once. In this case, your backend code will need to create a conversation for each user and then send the same message text to each conversation.

## Send email and SMS notifications

TalkJS also lets you send email and SMS notifications along with your messages. To test this, you need to provide an [email address](https://talkjs.com/docs/Reference/Concepts/Users/#email) and [phone number](https://talkjs.com/docs/Reference/Concepts/Users/#phone) for the user receiving the notifications. You must also give the user a [role](https://talkjs.com/docs/Reference/Concepts/Roles/). In our example, we give the user the `"default"` role:

```js
const me = new Talk.User({
  id: "messageCenterExampleReceiver",
  name: "Alice",
  email: "alice@example.com",
  phone: "+447700900000",
  photoUrl: "https://talkjs.com/images/avatar-1.jpg",
  role: "default",
});
```

For more information on notifications, [see our docs](https://talkjs.com/docs/Features/Notifications/).

## Make the conversation read-only

In-app messages are often read-only. TalkJS lets you enforce this by setting the [access type](https://talkjs.com/docs/Reference/Concepts/Participants/#access) to `"Read"` for the message receiver. To do this with the REST API, you can use the [Modify participation](https://talkjs.com/docs/Reference/REST_API/Participation/#modify-participation) endpoint for the conversation:

```js
// Set conversation to read-only for receiver
await fetch(
  `${basePath}/v1/${appId}/conversations/${conversationId}/participants/${receiverId}`,
  {
    method: "put",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${secretKey}`,
    },
    body: JSON.stringify({
      access: "Read",
    }),
  }
);
```

You should now see the warning "You can read, but not send messages":

<figure class="kg-image-card">
  <img class="kg-image" src="https://talkjs.com/resources/content/images/2023/08/4-read-only.jpg" alt="Inbox with the read-only warning in the message field"/>
  <figcaption>Inbox with the read-only warning in the message field</figcaption>
</figure>

## Remove the message field from the inbox

As our messages are read-only, we can reflect this in the UI by removing the message field where the customer would type a response. We can do this by setting the [message field visibility](https://talkjs.com/docs/Features/Customizations/The_Message_Field/#message-field-visibility) to `false`:

```js
inbox.messageField.setVisible(false);
```

<figure class="kg-image-card">
  <img class="kg-image" src="https://talkjs.com/resources/content/images/2023/08/5-remove-message-field.jpg" alt="Inbox with the message field removed"/>
  <figcaption>Inbox with the message field removed</figcaption>
</figure>

## Summary

You now have a working demonstration of how to build a message center with TalkJS. To recap, in this tutorial we have:

- Used the TalkJS REST API to create conversations and messages
- Set up email and SMS notifications
- Removed the message field from the inbox

For the full example code for this tutorial, see [our Github repo](https://github.com/talkjs/talkjs-examples/tree/master/howtos/how-to-create-a-message-center).

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS Docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.
