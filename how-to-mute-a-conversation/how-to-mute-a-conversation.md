# How to mute a conversation

When you're keeping track of a busy inbox, it can be useful to turn off [notifications](https://talkjs.com/docs/Features/Notifications/) for individual conversations. TalkJS's [Conversation Actions](https://talkjs.com/docs/Features/Customizations/Conversation_Actions/) feature makes it easy to add a new custom "Mute this conversation" option to turn off email, SMS and push notifications.

In this tutorial, we'll demonstrate how to add a new action that sends a web request to your backend server. The server then calls TalkJS's REST API to turn off notifications for the conversation.

To follow along, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing TalkJS project using the [JavaScript Chat SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/). See our [Getting Started guide](https://talkjs.com/docs/Getting_Started/) for an example of how to set this up.
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.

We’ll build up the feature step by step in the following sections. If you would rather see the complete example code, see the

!! [GitHub repo]() for this tutorial.

## Add a mute option to the menu

First, we need add a way to trigger the mute action in the frontend. You can configure which actions a user is able to take based on their [user role](https://talkjs.com/docs/Reference/Concepts/Roles/). In this example we'll apply a custom conversation action to the "default" role.

In the **Roles** tab of the TalkJS dashboard, select the "default" role option. In the **Custom conversation actions** section, add a new custom conversation action with a **Name** of "mute" and a **Label** of "Mute conversation". The name will be used in our code, while the label is what appears in the UI:

!! add 1-conversation-actions-dashboard.jpg

If you're using a preset theme, you should now see a new **Delete conversation** option in the menu at the top of your conversation:

!! add 2-mute-conversation-ui.jpg

If you're using a legacy theme, or customized a theme before the custom conversation actions feature was released, you'll need to add this menu to your theme before it will show up. See [our docs](https://talkjs.com/docs/Features/Customizations/Conversation_Actions/#the-action-menu-does-not-show-up) for details on how to edit the theme.

## Listen for the "Mute conversation" event

You can now listen for your new conversation action using the [`onCustomConversationAction` method](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/Chatbox/#Chatbox__onCustomConversationAction) in TalkJS's JavaScript SDK.

As a test, add the following to your TalkJS code:

```js
inbox.onCustomConversationAction("mute", (event) => {
  console.log("Muted conversation with id:", event.conversation.id);
});
```

You should now see the conversation ID logged to your browser console when you click the **Mute conversation** menu option.

## Mute notifications

Next, we'll mute notifications for the conversation with the TalkJS JavaScript SDK. In this tutorial, we'll use an example conversation between `me`, the user in the current TalkJS session, and another user:

```js
const me = new Talk.User({
  id: "muteConversationExampleSupportAgent",
  name: "Alice",
  email: "alice@example.com",
  role: "default",
  photoUrl: "https://talkjs.com/images/avatar-1.jpg",
  welcomeMessage: "Hey there! How can I help?",
});
const talkSession = new Talk.Session({
  appId: "<APP_ID>", // replace with your app ID
  me: me,
});

const other = new Talk.User({
  id: "muteConversationExampleUser",
  name: "Sebastian",
  email: "sebastian@example.com",
  role: "default",
  photoUrl: "https://talkjs.com/images/avatar-5.jpg",
  welcomeMessage: "Hey, how can I help?",
});

const conversation = talkSession.getOrCreateConversation(
  "muteConversationExample"
);
conversation.setParticipant(me);
conversation.setParticipant(other);
```

Now, update your `onCustomConversationAction` call with the following:

```js
inbox.onCustomConversationAction("mute", (event) => {
  console.log("Muted conversation with id:", event.conversation.id);
  let conversation = talkSession.getOrCreateConversation(event.conversation.id);
  conversation.setParticipant(me, { notify: false });
  inbox.select(conversation);
});
```

This code calls [`setParticipant`](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/ConversationBuilder/#ConversationBuilder__setParticipant) on the conversation and sets the participant to `me` again, this time with the [`notify`](https://talkjs.com/docs/Reference/Concepts/Participants/#notify) field set to `false`. After this, it makes the updated conversation the active one by calling [`select`](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/Inbox/#Inbox__select) on the inbox.

## Test your mute feature

To test that the mute feature is working, you'll need to set up notifications for a test user. In this tutorial, we'll test [email notifications](https://talkjs.com/docs/Features/Notifications/Email_Notifications/), but you may want to test [SMS notifications](https://talkjs.com/docs/Features/Notifications/SMS_Notifications/) and [mobile push notifications](https://talkjs.com/docs/Features/Notifications/Mobile_Push_Notifications/) as well.

First, set an email address where you can receive test notifications:

```js
const me = new Talk.User({
  id: "muteConversationExampleSupportAgent",
  name: "Alice",
  email: "<EMAIL_ADDRESS>", // replace with your test email address
  role: "default",
  photoUrl: "https://talkjs.com/images/avatar-1.jpg",
  welcomeMessage: "Hey there! How can I help?",
});
```

Now switch focus from the tab with your chat conversation, to trigger TalkJS's [rules for sending a notification](https://talkjs.com/docs/Features/Notifications/#when-are-notifications-sent), and send a test message from the other user in the conversation. In this example we'll do this with the TalkJS REST API using the `node-fetch` library, but you can use another approach if you prefer:

```js
import fetch from "node-fetch";

const appId = "<APP_ID>";
const secretKey = "<SECRET_KEY>";

const basePath = "https://api.talkjs.com";
const conversationId = "muteConversationExample";

// Add a new message from Sebastian

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
        text: "Hi, there's a problem with my order",
        sender: "muteConversationExampleUser",
        type: "UserMessage",
      },
    ]),
  }
);
```

You should receive a notification email for this message.

Now click your **Mute conversation** menu option, and then send another test email. This time, you should not receive a notification email.

## Conclusion

You now have a working demonstration of how to mute a conversation! To recap, in this tutorial you have:

- Added a new "Mute conversation" custom conversation action.
- Used TalkJS's JavaScript SDK to mute notifications for the conversation.
- Tested that you no longer receive email notifications after you mute the conversation.

For the full example code for this tutorial,

!!see our [GitHub repo]().

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS Docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.
