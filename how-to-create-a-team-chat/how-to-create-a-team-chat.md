# How to use TalkJS to create a team chat with channels

When your team is spread out across multiple locations, it's great to have a central place to collaborate, with discussion channels and private messaging all in one place. In this tutorial, we'll walk you through how to customize TalkJS to create a team chat app, with a similar layout to Slack or Microsoft Teams:

!! gif

You can see it in action in our [interactive demo](https://talkjs.com/demo/team-chat/).

Our example will use [React](https://react.dev/) along with the [`talkjs-react` library](https://github.com/talkjs/talkjs-react). To follow along, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing React app
- An installation of [Node.js](https://nodejs.org/) along with the [npm](https://www.npmjs.com/) package manager. We’ll use this to create our backend server.

For the complete example code, see the [GitHub repo](https://github.com/talkjs/talkjs-examples/tree/master/react/remote-work-demo) for this tutorial.

## Add talkjs-react to your project

If you aren't already using the `talkjs-react` library, install it with `npm` or `yarn`:

```
npm install talkjs @talkjs/react
```

or

```
yarn add talkjs @talkjs/react
```

To use it in your components, import TalkJS using the following statement:

```jsx
import Talk from "talkjs";
```

## Add the session component and sync user data

Next, import the TalkJS [`Session` component](https://github.com/talkjs/talkjs-react#1-create-a-session) and add it to your app. In our example, we'll import the `Session` component at the top level of our app, and put the rest of the team chat code inside it as a child component, so that the whole team chat app has access to the session:

```jsx
import { useCallback } from "react";
import { Session } from "@talkjs/react";
import talkJsConfig from "./talkJsConfig";

function App() {
  const syncUser = useCallback(
    () =>
      new Talk.User({
        id: talkJsConfig.userId,
        name: "Eulalia van Helgen",
        photoUrl: "https://talkjs.com/new-web/avatar-7.jpg",
        role: "default",
      }),
    []
  );

  return (
    <Session appId={talkJsConfig.appId} syncUser={syncUser}>
      <TeamChat />
    </Session>
  );
}

export default App;
```

The `Session` component takes the basic information we need for a TalkJS [session](https://talkjs.com/docs/Reference/Concepts/Sessions/): an app ID, and a user to log in as.

You can find your `appId` in the **Settings** tab of the TalkJS dashboard. In the example above, we're storing it in a separate `talkJsConfig` file.

To sync data for the user we want to log in as, we'll use the [`syncUser` prop](https://github.com/talkjs/talkjs-react#1-synchronize-user-data). This prop uses the [`useCallback`](https://react.dev/reference/react/useCallback) React Hook and expects a callback function that creates a `Talk.User` object. In this example, the `syncUser` callback creates a user with a user ID that we define in `talkJsConfig`.

## Add UI components for the team chat

As well as the TalkJS chatbox, we'll need some other React components to make up our team chat UI, including lists of channels and DMs and a custom chat header. The exact details of styling and placement will depend on your app and the libraries that you're already using. In this section we'll list the components that we're using in the example repo.

You'll see the following components in the `src/components` directory:

- `TeamChat`: the main component that displays our team chat.
- `CategoryCollapse`: displays a collapsible list of conversations with a given `title`. We'll use this component twice: once to display a list of channels and once to display a list of DMs.
- `ConversationListItem`: displays a single conversation within each collapsible list.
- `ChatHeader`: displays a custom chat header.
- `ConversationImage`: displays a conversation image in the chat header and next to each conversation list item.

In the example repo we're using [Tailwind CSS](https://tailwindcss.com/) to style our components, so you'll see layout code inside the `className` of each component. The layout leaves a space for the TalkJS chatbox (we'll add this in the next section):

!! image: 2-layout.jpg

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Layout of the team chat UI. There's an empty space that we'll fill with the TalkJS chatbox in later sections. The chat header is above it and the collapsible lists are to the left."/>
  <figcaption>Layout of the team chat UI</figcaption>
</figure>

## Add the chatbox component and sync conversation data

To add the TalkJS chatbox, import the [`Chatbox` component](https://github.com/talkjs/talkjs-react#2-create-a-chatbox) and add it in the place you want it in your layout:

```jsx
import { Chatbox } from "@talkjs/react";

// ... //

const syncConversation = useCallback((session) => {
  const defaultConv = session.getOrCreateConversation("remoteWorkMeetup");
  defaultConv.setParticipant(session.me);
  return defaultConv;
}, []);

// ... //

<Chatbox
  syncConversation={syncConversation}
  className="h-full w-full overflow-hidden rounded-b-xl lg:rounded-none lg:rounded-br-xl"
  showChatHeader={false}
  theme="team_chat"
/>;
```

The `Chatbox` component has a [`syncConversation` prop](https://github.com/talkjs/talkjs-react#2-create-and-join-a-conversation), which takes a callback that creates a [`ConversationBuilder`](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/ConversationBuilder/#ConversationBuilder) object. In this example, the `syncConversation` callback creates a conversation with a conversation id of `remoteWorkMeetup`, and adds you to the conversation. This will be the default "meetup" channel that you see when you first load the app:

!! add screenshot

The other props style the chatbox. We set `showChatHeader` to `false`, because we've already replaced it with a custom `ChatHeader` component, and select the `team_chat` [theme](https://talkjs.com/docs/Features/Themes/). We use `className` to style the component further with Tailwind.

## Change the conversation

Currently, our chatbox displays a single conversation. Next, we want to switch to a different TalkJS conversation when we click on a channel or direct message thread. In our UI implementation, this is handled inside the `ConversationListItem` component, where we have a button that calls a `changeConversation` function when you click on the item:

```jsx
<button
  onClick={() => changeConversation(conversation)}
  // ...
>
  // ...
</button>
```

To change the conversation, we'll need to be able to access the TalkJS `Session` object inside the `changeConversation` function. To do this, we'll use with the [`useSession` custom React Hook](https://github.com/talkjs/talkjs-react#using-hooks). We'll also need to access the `Chatbox` object, which we'll [create a reference to](https://github.com/talkjs/talkjs-react#using-refs) with the [`useRef` React Hook](https://react.dev/reference/react/useRef):

```jsx
import { useRef } from "react";
import { useSession } from "@talkjs/react";

const chatboxRef = useRef(null);

// ... //

<Chatbox
  syncConversation={syncConversation}
  chatboxRef={chatboxRef} // add chatbox ref
  className="h-full w-full overflow-hidden rounded-b-xl lg:rounded-none lg:rounded-br-xl"
  showChatHeader={false}
  theme="team_chat"
/>;
```

We then use these references inside the `changeConversation` function, where we set up the relevant TalkJS conversation and add the user as a participant, in the same way that we did for the default conversation:

```jsx
import { useState } from "react";

const initialConversation = talkJsConfig.conversations.channels[0];
const [currentConversation, setCurrentConversation] =
  useState(initialConversation);

// ... //

const changeConversation = (conversation) => {
  if (session?.isAlive) {
    const talkJsConversation = session.getOrCreateConversation(conversation.id);
    const me = new Talk.User(talkJsConfig.userId);

    talkJsConversation.setParticipant(me);
    talkJsConversation.setAttributes(conversation);
    setCurrentConversation(conversation);

    if (chatboxRef.current?.isAlive) {
      chatboxRef.current.select(talkJsConversation);
    }
  }
};
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
  (item) => item.conversation.id === conversation.id
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
