# How to use TalkJS to create a team chat with channels

When your team is spread out across multiple locations, it's great to have a central place to collaborate, with discussion channels and private messaging all in one place. In this tutorial, we'll walk you through how to customize TalkJS to create a team chat app, with a similar layout to Slack or Microsoft Teams:

!! gif

You can see it in action in our [interactive demo](https://talkjs.com/demo/team-chat/).

Our example will use [React](https://react.dev/) along with the [`talkjs-react` library](https://github.com/talkjs/talkjs-react). To follow along, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing React app
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.

For the complete example code, see the [GitHub repo](https://github.com/talkjs/talkjs-examples/tree/master/react/remote-work-demo) for this tutorial.

## Summary of React components

As well as the TalkJS chatbox, we'll need some other React components to make up our team chat UI, including lists of channels and DMs and a custom chat header. The exact details of styling and placement will depend on your app and the libraries that you're already using. In this section we'll list the components that we're using in the example repo.

You'll see the following components in the `src/components` directory:

- `CategoryCollapse`: displays a collapsible list of conversations with a given `title`. We'll use this component twice: once with a title of "Channels" and once with a title of "DMs"
- `ConversationListItem`: displays a single conversation within each collapsible list
- `ChatHeader`: displays a custom chat header
- `ConversationImage`: displays a conversation image in the chat header and next to each conversation list item.

In the example repo we're using [Tailwind CSS](https://tailwindcss.com/) to style our components, so you'll see layout code inside the `className` of each component. The layout leaves an empty space for the TalkJS chatbox (we'll add this in the next section):

!! image: 2-layout.jpg

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Layout of the team chat UI. There's an empty space that we'll fill with the TalkJS chatbox in later sections. The chat header is above it and the collapsible lists are to the left."/>
  <figcaption>Layout of the team chat UI</figcaption>
</figure>

## Set up the TalkJS chatbox

In this section we'll set up a basic chatbox with the `<Session>` and `<Chatbox>` components, and sync some user and conversation data.

### Install the talkjs-react library

If you aren't already using the `talkjs-react` library, install it with `npm` or `yarn`:

```
npm install talkjs @talkjs/react
```

or

```
yarn add talkjs @talkjs/react
```

To use it in the component that you'll be adding a chatbox to, import TalkJS using the following statement:

```jsx
import Talk from "talkjs";
```

### Add the session component and sync user data

Next, import the TalkJS [`Session` component](https://github.com/talkjs/talkjs-react#1-create-a-session) and add it to your app in the place where you want the chatbox to appear in your UI:

```jsx
import { Session } from "@talkjs/react";
import { useCallback } from "react";

// ...

const syncUser = useCallback(
  () =>
    new Talk.User({
      id: talkJsConfig.userId,
      name: "Eulalia Van Helgen",
      photoUrl: "https://talkjs.com/new-web/avatar-7.jpg",
      role: "default",
    }),
  []
);

// ...

<Session appId={talkJsConfig.appId} syncUser={syncUser}>
  {/* The chatbox will go here */}
</Session>;
```

This sets up the basic information we need for a TalkJS [session](https://talkjs.com/docs/Reference/Concepts/Sessions/): an app ID, and a user to log in as.

You can find your `appId` in the **Settings** tab of the TalkJS dashboard. In the example above, we're storing it in a separate `talkJsConfig` file.

To sync data for the user we want to log in as, we'll use the [`syncUser` prop](https://github.com/talkjs/talkjs-react#1-synchronize-user-data). This prop uses the [`useCallback`](https://react.dev/reference/react/useCallback) React Hook and expects a callback function that creates a `Talk.User` object. In this example, the `syncUser` callback creates a user with a user ID that we define in `talkJsConfig`.

### Add the chatbox component and sync conversation data

Similarly, we'll import the [`Chatbox` component](https://github.com/talkjs/talkjs-react#2-create-a-chatbox) and add it inside the `Session` component:

```jsx
import { Chatbox } from "@talkjs/react";

// ... //

const syncConversation = useCallback((session) => {
  const other = new Talk.User({
    id: "remoteWorkOther",
    name: "TalkJS",
    photoUrl: "https://talkjs.com/new-web/avatar-talkjs.jpg",
    welcomeMessage:
      "Hi there 👋 \nThis is our chat demo and you can test it out in any way you like. Play with some of the chat features, kick the tyres a little, and experience what you could easily build with TalkJS. Also consider checking out our Docs: https://talkjs.com/docs/",
    role: "default",
  });

  const defaultConv = session.getOrCreateConversation("remoteWorkWelcome");
  defaultConv.setParticipant(session.me);
  defaultConv.setParticipant(other);
  return defaultConv;
}, []);

// ... //

<Session appId={talkJsConfig.appId} syncUser={syncUser}>
  <Chatbox
    syncConversation={syncConversation}
    className="h-full w-full overflow-hidden rounded-b-xl lg:rounded-none lg:rounded-br-xl"
    showChatHeader={false}
    theme="team_chat"
  />
</Session>;
```

Here we've set `showChatHeader` to `false`, because we've already replaced it with a custom `ChatHeader` component. We'll use the `team_chat` [theme](https://talkjs.com/docs/Features/Themes/). We use `className` to style the component with Tailwind.

We'll sync conversation data with the [`syncConversation` prop](https://github.com/talkjs/talkjs-react#2-create-and-join-a-conversation), which takes a callback that creates a [`ConversationBuilder`](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/ConversationBuilder/#ConversationBuilder) object. In this example, the `syncConversation` callback creates a conversation with a conversation id of `remoteWorkWelcome`, and adds a couple of users. This will be the default "welcome" channel that you see when you first load the app:

!! add screenshot

## Change the conversation

Currently, our chatbox displays a single conversation. Next, we want to switch to a different conversation when we click on another `ConversationListItem` (channel or direct message). The `ConversationListItem` component is outside our `Session` component, so we'll use the [`useRef`](https://react.dev/reference/react/useRef) React Hook to [make references to the session and chatbox](https://github.com/talkjs/talkjs-react#using-refs), and pass these in to the `sessionRef` and `chatboxRef` props in our components:

```jsx
import { useRef } from "react";

const sessionRef = useRef(null);
const chatboxRef = useRef(null);

// Added sessionRef and chatboxRef
<Session appId={talkJsConfig.appId} syncUser={syncUser} sessionRef={sessionRef}>
  <Chatbox
    syncConversation={syncConversation}
    chatboxRef={chatboxRef}
    className="h-full w-full overflow-hidden rounded-b-xl lg:rounded-none lg:rounded-br-xl"
    showChatHeader={false}
    theme="team_chat"
  />
</Session>;
```

We then use these references inside a `changeConversation` function that we pass to the `ConversationListItem`:

```jsx
import { useState } from "react";

const initialConversation = talkJsConfig.conversations.channels[0];
const [currentConversation, setCurrentConversation] =
  useState(initialConversation);

// ... //

const changeConversation = (conversation) => {
  if (sessionRef.current?.isAlive) {
    const talkJsConversation = sessionRef.current.getOrCreateConversation(
      conversation.id
    );

    const me = new Talk.User({
      id: talkJsConfig.userId,
      name: "Eulalia Van Helgen",
      photoUrl: "https://talkjs.com/new-web/avatar-7.jpg",
      role: "default",
    });

    const other = new Talk.User({
      id: "remoteWorkOther",
      name: "TalkJS",
      photoUrl: "https://talkjs.com/new-web/avatar-talkjs.jpg",
      welcomeMessage:
        "Hi there 👋 \nThis is our chat demo and you can test it out in any way you like. Play with some of the chat features, kick the tyres a little, and experience what you could easily build with TalkJS. Also consider checking out our Docs: https://talkjs.com/docs/",
      role: "default",
    });

    talkJsConversation.setParticipant(me);
    talkJsConversation.setParticipant(other);
    talkJsConversation.setAttributes(conversation);
    setMobileChannelSelected(true);
    setCurrentConversation(conversation);
    if (chatboxRef.current?.isAlive) {
      chatboxRef.current.select(talkJsConversation);
    }
  }
};
```

Inside the `changeConversation` function we set up the relevant TalkJS conversation and add participants, in the same way that we did for the default conversation.

We'll then call the `changeConversation` function inside the `ConversationListItem` component:

```jsx
<button
  onClick={() => changeConversation(conversation)}
  // ...
>
  // ...
</button>
```

Clicking on a different channel or DM thread now takes you to the relevant conversation.

## Highlight unread channels

We also need a way to see that there are unread messages. In this section we'll update our code so that channel or DM thread names are displayed in bold when there's an unread message.

To do this, we'll use [the `onUnreadsChange` prop](https://github.com/talkjs/talkjs-react#events) in the `Session` component to listen for changes in the number of unread messages:

```jsx
const [unreadMessages, setUnreadMessages] = useState([]);

// ...

<Session
  appId={talkJsConfig.appId}
  syncUser={syncUser}
  sessionRef={sessionRef}
  onUnreadsChange={(unreads) => setUnreadMessages(unreads)}
>
  // Chatbox code here...
</Session>;
```

We'll then pass the `unreadMessages` variable into the `ConversationListItem` component and use it to set the conversation name to bold:

```jsx
const unread = unreadMessages.find(
  (item) => item.lastMessage.conversation.id === conversation.id
);

return (
  <button
    onClick={() => changeConversation(conversation)}
    // ...
  >
    // ...
    <div className="header">
      <div
        className={`conversation-name unread text-sm ${
          unread ? "font-bold text-white" : ""
        }`}
      >
        {conversation.subject}
      </div>
    </div>
  </button>
);
```

!! gif

## Conclusion

You now have a working demonstration of a team chat with channels! To recap, in this tutorial you have:

- Added a TalkJS chatbox to your React app with the `talkjs-react` library
- Added a function to change the conversation when you click on a channel name
- Listened for unread messages and highlighted channels with unread messages

For the full example code for this tutorial, see our [GitHub repo](https://github.com/talkjs/talkjs-examples/tree/master/react/remote-work-demo).

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS Docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.
