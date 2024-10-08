# How to add threads to your team chat with TalkJS

This tutorial builds on [How to use TalkJS to create a team chat with channels](https://talkjs.com/resources/how-to-use-talkjs-to-create-a-team-chat-with-channels/), which shows you how to use TalkJS's [React SDK](https://talkjs.com/docs/Getting_Started/React/) to make a collaborative chat with DMs and discussion channels, similar to Slack or Teams.

In this tutorial we'll show you how to add reply threads:

!! 1-demo.gif

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Demo of threads in the team chat, where the user views and replies in two threads."/>
  <figcaption>Demo of threads in the team chat</figcaption>
</figure>

By default, TalkJS instead supports replying to messages within a conversation, in a similar style to replies in a messaging app like WhatsApp. However, TalkJS is flexible enough to let you build reply threads as a feature yourself.

To try it out, see the [interactive chat demo](https://talkjs.com/demo/team-chat/) on our website.

We'll take you through the following steps:

- Adding a **Reply in thread** message action
- Adding a **Back** button
- Adding a link to the thread with a reply count

To do this, we'll make full use of TalkJS's customization options, including [custom themes](https://talkjs.com/docs/Features/Themes/The_Theme_Editor/), [custom message actions](https://talkjs.com/docs/Features/Message_Features/Message_Actions/), [custom data](https://talkjs.com/docs/Reference/Concepts/Conversations/#custom), [webhooks](https://talkjs.com/docs/Reference/Webhooks/) and the [REST API](https://talkjs.com/docs/Reference/REST_API/Getting_Started/Introduction/).

You can see the complete example code in our [GitHub repo](https://github.com/talkjs/talkjs-examples/tree/master/react/team-chat-with-threads) for this tutorial.

To follow along with this tutorial, you'll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing team chat to add reply threads to. In this tutorial we'll be using our React-based [team chat example](https://github.com/talkjs/talkjs-examples/tree/master/react/remote-work-demo) as the starting point, but you could adapt the same ideas to a different framework.
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.

## Set up the project

In this section, we'll get the basic structure of the project set up.

### Set up the team chat

If you haven't already worked through the [How to use TalkJS to create a team chat with channels](https://talkjs.com/resources/how-to-use-talkjs-to-create-a-team-chat-with-channels/) tutorial, follow the instructions in the [example repo README](https://github.com/talkjs/talkjs-examples/tree/master/react/remote-work-demo) to clone the project, install dependencies and run the app.

### Create the backend server

To create new threads, you'll need to call TalkJS's REST API. You'll need to do this from a backend server to avoid exposing your TalkJS secret key. In this tutorial we'll use [Express](https://expressjs.com/), but feel free to use your favorite web server library instead.

Add a new `backend` directory to your project. Inside `backend`, initialize the project:

```sh
npm init
```

This will add a `package.json` file to your directory. Add the following lines:

```json
  "type": "module",
  "scripts": {
    "start": "node server.js"
  }
```

Then install the packages that we'll need:

```sh
npm install express cors
```

As well as Express, this installs the `cors` package, which we use to set up CORS support on the server.

Next, create a new `server.js` file inside `backend` and add the following:

```js
import express from "express";
import cors from "cors";

const appId = "<APP_ID>";
const secretKey = "<SECRET_KEY>";

const basePath = "https://api.talkjs.com";

const app = express();
app.use(cors());
app.use(express.json());

app.listen(3001, () => console.log("Server is up"));
```

Replace `<APP_ID>` and `<SECRET_KEY>` with the values in the **Settings** page of your [TalkJS dashboard](https://talkjs.com/dashboard).

Now, run the server with:

```sh
npm start
```

We'll add calls to the REST API in later sections.

## Add the "Reply in thread" feature

In this section, we'll create a **Reply in thread** [message action](/Features/Message_Features/Message_Actions/) that creates a new thread for that message, or takes you to the thread if it already exists.

The thread will be a standard TalkJS [conversation](/Reference/Concepts/Conversations/), with the message you're replying to duplicated at the top as a [system message](/Reference/Concepts/System_Messages/).

### Replace the "Reply" action with a custom "Reply in thread" action

First we'll remove the standard **Reply** message action and create a custom **Reply in thread** action instead. To do this:

1.  Go to the **Chat UI** tab of the TalkJS dashboard.
2.  Select the "default" role.
3.  In **Actions and permissions** > **Built-in message actions**, set **Reply** to **None**.
4.  In **Actions and permissions** > **Custom message actions**, create a new action with a name of "replyInThread" and a label of "Reply in thread", available to all messages for users with write permission.

### Add an event handler for the "Reply in thread" action

To listen for the new `replyInThread` event in your frontend code, pass the [`onCustomMessageAction` prop](https://talkjs.com/docs/Reference/React_SDK/Components/Chatbox/#event-props) to your `Chatbox`component in `TeamChat.jsx`:

```jsx
const replyInThread = (event) => {
  console.log(event);
};

<Chatbox
  // ... other props ...
  onCustomMessageAction={replyInThread}
/>;
```

Now when you select the **Reply in thread** message action, you should see the event data logged to your browser console.

### Create the new thread conversation in the frontend

Next we'll update the `replyInThread` function to create a new conversation which we'll use as our thread. We'll give the conversation an ID of `replyto_<MESSAGE_ID>`, where `<MESSAGE_ID>` is the ID of the message it's replying to.

<div style="background-color: #EFF; padding: 15px;">

  <p><strong>Note:</strong> In the rest of this tutorial we'll refer to the message we're replying to as the <strong>parent message</strong>, and its associated conversation as the <strong>parent conversation</strong>. </p>

</div>

Update your `replyInThread` function in `TeamChat.jsx` with:

```js
const replyInThread = (event) => {
  console.log(event);

  if (session?.isAlive) {
    let thread = session.getOrCreateConversation("replyto_" + event.message.id);
    const me = new Talk.User(talkJsConfig.userId);
    thread.setParticipant(me);

    if (chatboxRef.current?.isAlive) {
      chatboxRef.current.select(thread);
    }

    setCurrentConversation({
      id: "replyto_" + event.message.id,
      avatar: "",
      subject: "Replies",
    });
  }
};
```

We also set the current conversation, which is used by the `ConversationListItem` component to highlight the relevant conversation in the channel list.

### Pass message action data to the server

We can create the conversation for the thread on the frontend, but we'll need a backend server to deal with other parts of creating the new thread, like copying across the message that the thread is replying to.

First, we'll pass event data from the **Reply in thread** message action to the backend server. Create a new `postMessageData` function to call a `/new-thread` endpoint (which we'll create below):

```jsx
async function postMessageData(
  messageId,
  conversationId,
  messageText,
  participants
) {
  const response = await fetch("http://localhost:3001/new-thread", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      messageId,
      conversationId,
      messageText,
      participants,
    }),
  });
}
```

Now update your `replyInThread` function to replace your `console.log(event)` call with a call to `postMessageData`:

```js
const replyInThread = (event) => {
  postMessageData(
    event.message.id,
    event.message.conversation.id,
    event.message.body,
    Object.keys(event.message.conversation.participants)
  );

  // ...
};
```

This posts the message ID, conversation ID, message text and a list of participants to a `/newThread` endpoint in the backend server you created earlier. Let's create the endpoint next in `server.js`:

```js
// Endpoint to create new sub-conversation (thread)
app.post("/new-thread", async (req, res) => {
  // Get details of the message we'll reply to
  const parentMessageId = req.body.messageId;
  const parentConvId = req.body.conversationId;
  const parentMessageText = req.body.messageText;
  const parentParticipants = req.body.participants;

  console.log("Parent message id: " + parentMessageId);
  console.log("Parent conversation id: " + parentConvId);
  console.log("Parent message text: " + parentMessageText);
  console.log("Parent message participants: " + parentParticipants);
});
```

Restart your server and try clicking the **Reply in thread** message action again. You should see the event data logged to your server console. We'll update this endpoint later to create the new thread.

### Create the new thread

Next we'll extend this endpoint to use the data to create a new thread if one doesn't already exist, instead of just logging it to the console. Update the `/new-thread` endpoint as follows:

```js
// Endpoint to create new sub-conversation (thread)
app.post("/new-thread", async (req, res) => {
  // Get details of the message we'll reply to
    const parentMessageId = req.body.messageId;
    const parentConvId = req.body.conversationId;
    const parentMessageText = req.body.messageText;
    const parentParticipants = req.body.participants;

    const response = await getMessages(parentMessageId);
    const messages = await response.data;

    // Create a message with the text of the parent message if one doesn't already exist
    if (!messages.data?.length) {
      await handleThreadCreation(
        parentMessageId,
        parentConvId,
        parentParticipants,
        parentMessageText
      );
    }
  }
});
```

Here we're calling new `getMessages` and `handleThreadCreation` functions, which we'll create next.

We use the `getMessages` function to check if there's already an existing conversation with messages:

```js
// Get messages in a given thread (sub-conversation)
async function getMessages(messageId) {
  // Sometimes getOrCreateConversation gets called slightly out of sync with this backend,
  // which causes the thread functionality to break, so we make a "put" call
  // to create the conversation if it doesn't already exist

  await fetch(`${basePath}/v1/${appId}/conversations/replyto_${messageId}/`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${secretKey}`,
    },
  });

  const response = await fetch(
    `${basePath}/v1/${appId}/conversations/replyto_${messageId}/messages`,
    {
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${secretKey}`,
      },
    }
  );

  const data = await response.json();
  if (!data) {
    return { data: [] };
  } else {
    return { data };
  }
}
```

This function first calls the [create or update conversation](https://talkjs.com/docs/Reference/REST_API/Conversations/#setting-conversation-data) endpoint of the REST API to ensure that the conversation exists, in case there are syncing issues with the frontend. It then calls the [list messages from a conversation](https://docs-add-components-getting-started-react.talkjsonprem.com/docs/Reference/REST_API/Messages/#list-messages-from-a-conversation) REST endpoint.

Then add the `handleThreadCreation` function, along with the `createThread` and `duplicateParentMessageText` functions that it calls:

```js
async function createThread(parentMessageId, parentConvId, participants) {
  const response = await fetch(
    `${basePath}/v1/${appId}/conversations/replyto_${parentMessageId}`,
    {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${secretKey}`,
      },

      body: JSON.stringify({
        participants: participants,
        subject: "Replies",
        custom: {
          parentConvId: parentConvId,
          parentMessageId: parentMessageId,
        },
      }),
    }
  );
  return response.json();
}

// Send message to the new thread with text of parent message
async function duplicateParentMessageText(parentMessageId, messageText) {
  const response = await fetch(
    `${basePath}/v1/${appId}/conversations/replyto_${parentMessageId}/messages`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${secretKey}`,
      },

      body: JSON.stringify([
        {
          text: messageText,
          type: "SystemMessage",
        },
      ]),
    }
  );
  return response.json();
}

async function handleThreadCreation(
  parentMessageId,
  parentConvId,
  participants,
  messageText
) {
  const createThreadResponse = await createThread(
    parentMessageId,
    parentConvId,
    participants
  );

  const duplicateMessageResponse = await duplicateParentMessageText(
    parentMessageId,
    messageText
  );

  return {
    thread: createThreadResponse.data,
    duplicateMessage: duplicateMessageResponse.data,
  };
}
```

The `createThread` function creates or updates the thread on the backend, and adds the message ID and conversation ID of the parent message as [custom conversation properties](/Reference/Concepts/Conversations/#custom). We'll use these in later sections.

The `duplicateParentMessageText` function then calls the [send message](/Reference/REST_API/Messages/#send-a-message) REST API endpoint to send a system message that duplicates the text of the parent message, so that readers of the thread can see the message they're replying to.

## Add a back button

Currently, there's no way of getting back to the parent conversation from a thread. In this section, we'll fix this by updating the behavior of the left arrow icon button in the chat header.

Some steps are similar to the steps for setting up the **Reply** message action in the last section, so we'll move more quickly through these this time.

### Track the conversation history

Our **Reply in thread** feature allows nested threads: you can reply to a message in one of the original conversations in the team chat (a channel or DM conversation), and then reply to a message in that thread, and so on. We'll use a stack to keep track of the history of previous conversations you've viewed at your current depth in the nesting.

For example, if you're currently in a thread that's a reply to a message in another thread, your `historyStack` would look something like this:

```js
[
  {
    id: "remoteWorkMeetup",
    subject: "design",
    // ...
  },
  {
    id: "replyto_msg_<MESSAGE_ID>",
    subject: "Replies",
    // ...
  },
];
```

First we initialise the history stack in `TeamChat.jsx` as an empty array:

```jsx
const [historyStack, setHistoryStack] = useState([]);
```

We'll also add a `findConversation` helper function to allow us to find a conversation given its ID:

```jsx
const findConversation = (convId) => {
  const allConversations = [
    ...talkJsConfig.conversations.dms,
    ...talkJsConfig.conversations.channels,
  ];

  const foundConversation = allConversations.find((conv) => conv.id === convId);

  if (!foundConversation) {
    return {
      id: convId,
      subject: "Replies",
      avatar: "",
    };
  } else {
    return foundConversation;
  }
};
```

This function retrieves the conversation data in `talkJsConfig` for channels and DMs, and an object in the same format for threads, with a subject of "Replies".

Update the `replyInThread` to add the parent conversation to the stack when you select the thread:

```jsx
const replyInThread = (event) => {
  // ...

  if (session?.isAlive) {
    // ...

    setHistoryStack((prevStack) => [
      ...prevStack,
      findConversation(event.message.conversation.id),
    ]);
  }
};
```

Then, add a `goBack` function that switches you to the previous conversation and removes it from the stack:

```jsx
const goBack = () => {
  const conversation = historyStack[historyStack.length - 1];
  setHistoryStack((prevStack) => prevStack.slice(0, -1));

  if (session?.isAlive) {
    const talkJsConversation = session.getOrCreateConversation(conversation.id);
    const me = new Talk.User({
      id: talkJsConfig.userId,
      name: "Eulalia Van Helgen",
      photoUrl: "https://talkjs.com/new-web/avatar-7.jpg",
      role: "default",
    });
    talkJsConversation.setParticipant(me);

    setMobileChannelSelected(true);
    setCurrentConversation(conversation);

    if (chatboxRef.current?.isAlive) {
      chatboxRef.current.select(talkJsConversation);
    }
  }
};
```

We'll pass this through to the `ChatHeader` as a prop, which we'll use in the next section:

```jsx
<ChatHeader
  // ... other props ...
  goBack={goBack}
/>
```

### Update the chat header

In the chat header of our current team chat, we're already using a back button on smaller screens to go back to the conversation list:

!! 2-back-button.jpg

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="The back button, a left arrow icon in the chat header."/>
  <figcaption>The back button</figcaption>
</figure>

We'll extend this behavior to also show the left arrow icon on all screens if an `isThread` flag (which we'll add below) is set:

```jsx
<button
  onClick={() => clickBackButton()}
  className={`mr-4 ${isThread ? "" : "lg:hidden"}`}
>
  <Icon.ChevronLeft size={25} />
</button>
```

We'll then call the `goBack` function from the previous section if the conversation is a thread:

```jsx
import { useEffect, useState } from "react";

const ChatHeader = ({
  conversation,
  goBack,
  mobileChannelSelected,
  setMobileChannelSelected,
}) => {
  const [isThread, setIsThread] = useState(false);

  useEffect(() => {
    const threadPattern = "replyto_";
    if (conversation.id.startsWith(threadPattern)) {
      setIsThread(true);
    } else {
      setIsThread(false);
    }
  }, [conversation]);

  const clickBackButton = () => {
    if (isThread) {
      goBack();
    } else {
      setMobileChannelSelected(false);
    }
  };

  return (
    // ...
  );
};

```

## Add a reply count

We can now create threads and navigate through them. The final feature we need is a way to tell whether messages have replies without clicking through. To complete our threaded chat demo, we'll add a link below messages with replies that's labelled, say, **Replies (3)** if there are three replies.

### Enable the "message.sent" webhook

We'll need a way of updating the reply link when users add new messages to threads. To do this, we’ll first enable TalkJS [webhooks](https://talkjs.com/docs/Reference/Webhooks/), which allow the TalkJS server to notify your server when it sends a new message. Webhooks let you use an event-driven architecture, where you get told about events when they happen rather than having to constantly check for new messages.

Webhooks are server-side only, so you'll need to add a new `/updateReplyCount` endpoint to your existing server code to monitor incoming events from the TalkJS server:

```js
app.post("/update-reply-count", (req, res) => {
  console.log(req.body);
  res.status(200).end();
});
```

Currently this endpoint just logs the event data when it receives a new message sent event.

For TalkJS to communicate with your server, you must expose it to the internet. To make this easy, we’ll use [ngrok](https://ngrok.com/) to create a secure tunnel to your local server. See our tutorial on [How to integrate ngrok with TalkJS](https://talkjs.com/resources/how-to-integrate-ngrok-with-talkjs-to-receive-webhooks-locally/#setting-up-ngrok) for instructions on how to install ngrok.

Once you have installed ngrok, run the following command:

```sh
ngrok http 3001
```

This command starts a secure tunnel to your local port 3001. The output should include the URL that ngrok exposes:

```sh
Forwarding                    https://<YOUR_SITE>.ngrok.io -> http://localhost:3001
```

Then enable the webhook with the following steps:

1.  Go to the **Settings** tab of the TalkJS dashboard.
2.  Enable the `message.sent` option in the **Webhooks** section of the TalkJS dashboard.
3.  Add the ngrok URL to **Webhook URLs** in the TalkJS dashboard, including the `update-reply-count` path: `https://<YOUR-URL>.ngrok.io/update-reply-count`

TalkJS will now send a web request to your server when a message is sent. To test this, write another message in your chat UI. You should see the event logged to your server's console.

### Add a custom reply count property

Next, we'll add a reply count as a [custom message property](https://talkjs.com/docs/Reference/Concepts/Messages/#custom) on the parent message that the thread replies to.

Update your `/update-reply-count` endpoint code with the following:

```js
// Update parent message with a reply count custom field
async function updateReplyCount(messageId, conversationId, count) {
  const response = await fetch(
    `${basePath}/v1/${appId}/conversations/${conversationId}/messages/${messageId}`,
    {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${secretKey}`,
      },

      body: JSON.stringify({
        custom: { replyCount: count.toString() },
      }),
    }
  );
  return response.json();
}

// Endpoint for webhook listener for reply counts
app.post("/update-reply-count", async (req, res) => {
  const data = req.body.data;
  const conversationId = data.conversation.id;
  const messageType = data.message.type;

  if (conversationId.startsWith("replyto_") && messageType === "UserMessage") {
    const { parentMessageId, parentConvId } = data.conversation.custom;

    const response = await getMessages(parentMessageId);
    const messages = await response.data;

    const messageCount = messages.data.length;

    // Ignore the first message in thread (it's a repeat of the parent message)
    if (messageCount > 1) {
      await updateReplyCount(parentMessageId, parentConvId, messageCount - 1);
    }
  }
});
```

Now, when the webhook listener receives new user messages from a thread (a conversation with an ID starting with "replyto\_"), it will call `getMessages`, which we introduced earlier, to get the count of messages in the thread. It then calls the `updateReplyCount` function to update the parent message with a `replyCount` custom message property with the current number of replies, ignoring the duplicated parent message at the top of the thread.

### Update your theme to display replies

The final step is to display the reply count in the chat UI. TalkJS allows you to [use conditionals](https://talkjs.com/docs/Features/Themes/Editing_Component_Templates/#rendering-conditionally) and [access custom properties](https://talkjs.com/docs/Features/Themes/Passing_Data_to_Themes/#storing-custom-data-in-users-conversations-and-messages) in your templates. Combining these ideas, we'll create an [action link](/Features/Customizations/Action_Buttons_Links/#actions-in-the-theme-editor) that says, for example, **1 reply** if the `replyCount` custom property has a value of 1 or **3 replies** if it has a value of 3. If there are no replies or the message is a welcome message, we won't include the link.

In the **UserMessage** section of your `team_chat` theme, add the following to the bottom of the template, below the emoji reactions div:

```html
<div t:if="{{ isWelcomeMessage }}"></div>
<div t:else>
  <ActionLink
    t:if="{{ custom.replyCount > 1 }}"
    class="reply-box-link"
    action="replyInThread"
  >
    <div class="reply-box">
      <a class="reply-box-link">{{ custom.replyCount }} replies</a>

      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="64"
        height="64"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
        viewBox="0 0 64 64"
        class="Icon icon-chevronLeft view-thread"
        aria-hidden="true"
      >
        <path
          d="M41.4157 12.5837C41.6019 12.7695 41.7497 12.9902 41.8505 13.2331C41.9513 13.4761 42.0032 13.7366 42.0032 13.9997C42.0032 14.2627 41.9513 14.5232 41.8505 14.7662C41.7497 15.0092 41.6019 15.2299 41.4157 15.4157L24.8277 31.9997L41.4157 48.5837C41.7912 48.9592 42.0022 49.4686 42.0022 49.9997C42.0022 50.5308 41.7912 51.0401 41.4157 51.4157C41.0401 51.7912 40.5308 52.0022 39.9997 52.0022C39.4686 52.0022 38.9592 51.7912 38.5837 51.4157L20.5837 33.4157C20.3974 33.2299 20.2496 33.0092 20.1488 32.7662C20.048 32.5232 19.9961 32.2627 19.9961 31.9997C19.9961 31.7366 20.048 31.4761 20.1488 31.2331C20.2496 30.9902 20.3974 30.7695 20.5837 30.5837L38.5837 12.5837C38.7695 12.3974 38.9902 12.2496 39.2331 12.1488C39.4761 12.048 39.7366 11.9961 39.9997 11.9961C40.2627 11.9961 40.5232 12.048 40.7662 12.1488C41.0092 12.2496 41.2299 12.3974 41.4157 12.5837Z"
          fill="currentColor"
        ></path>
      </svg>
    </div>
  </ActionLink>

  <ActionLink
    t:else-if="{{ custom.replyCount > 0 }}"
    class="reply-box-link"
    action="replyInThread"
  >
    <div class="reply-box">
      <a class="reply-box-link">{{ custom.replyCount }} reply</a>

      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="64"
        height="64"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
        viewBox="0 0 64 64"
        class="Icon icon-chevronLeft view-thread"
        aria-hidden="true"
      >
        <path
          d="M41.4157 12.5837C41.6019 12.7695 41.7497 12.9902 41.8505 13.2331C41.9513 13.4761 42.0032 13.7366 42.0032 13.9997C42.0032 14.2627 41.9513 14.5232 41.8505 14.7662C41.7497 15.0092 41.6019 15.2299 41.4157 15.4157L24.8277 31.9997L41.4157 48.5837C41.7912 48.9592 42.0022 49.4686 42.0022 49.9997C42.0022 50.5308 41.7912 51.0401 41.4157 51.4157C41.0401 51.7912 40.5308 52.0022 39.9997 52.0022C39.4686 52.0022 38.9592 51.7912 38.5837 51.4157L20.5837 33.4157C20.3974 33.2299 20.2496 33.0092 20.1488 32.7662C20.048 32.5232 19.9961 32.2627 19.9961 31.9997C19.9961 31.7366 20.048 31.4761 20.1488 31.2331C20.2496 30.9902 20.3974 30.7695 20.5837 30.5837L38.5837 12.5837C38.7695 12.3974 38.9902 12.2496 39.2331 12.1488C39.4761 12.048 39.7366 11.9961 39.9997 11.9961C40.2627 11.9961 40.5232 12.048 40.7662 12.1488C41.0092 12.2496 41.2299 12.3974 41.4157 12.5837Z"
          fill="currentColor"
        ></path>
      </svg>
    </div>
  </ActionLink>
</div>
```

Finally, we'll add some styling. See [our GitHub repo](https://github.com/talkjs/talkjs-examples/tree/master/react/team-chat-with-threads/theme/UserMessage.txt) for the full **UserMessage** template with added styles.

You should now have something that looks like this:

!! 3-team-chat.jpg

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="The finished chat, showing a message with three replies"/>
  <figcaption>The finished chat</figcaption>
</figure>

## Conclusion

You now have a working demonstration of a team chat with channels and reply threads! To recap, in this tutorial you have:

- added a **Reply in thread** message action
- used the REST API to create new threads
- updated the **Back** button to take you back to the parent conversation
- used webhooks and custom data to add a reply count

For the full example code for this tutorial, see our [GitHub repo](https://github.com/talkjs/talkjs-examples/tree/master/react/team-chat-with-threads).

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.
