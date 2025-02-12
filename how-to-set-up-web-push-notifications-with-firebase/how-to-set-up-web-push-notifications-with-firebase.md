# Set up web push notifications with Firebase Cloud Messaging

In this tutorial, we'll show you how to use [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) (FCM) with TalkJS to send web push notifications to your users' browsers when they receive a new message in their TalkJS chat:

<!-- TODO: add 1-demo.jpg -->

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="Demo of the feature. The user, Alice, selects to allow push notifications and then clicks the 'Request token' button. She then switches to another tab. The other participant in the conversation, Sebastian, then sends a message. This triggers a push notification for Alice."/>
  <figcaption>Demo of the feature. Alice selects to allow push notifications and then clicks the 'Request token' button. She then switches to another tab. Sebastian then sends a message, which triggers a push notification for Alice.</figcaption>
</figure>

TalkJS can be set up to send a variety of types of [notifications](https://talkjs.com/docs/Features/Notifications/) when you're away from your chat, including email, SMS, and browser notifications. In this tutorial we'll send [web push notifications](https://talkjs.com/docs/Features/Notifications/Mobile_Push_Notifications/) with Firebase Cloud Messaging. Firebase Cloud Messaging is a cross-platform messaging solution that comes as part of Google's [Firebase](https://firebase.google.com/) platform, which provides tools for web and mobile applications such as authentication, hosting and cloud storage.

We'll then set up a web page with a TalkJS chatbox, along with a [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) that can handle FCM notifications. Service workers only run over HTTPS for security reasons. In this tutorial we'll host our service worker and webpage with [Firebase Hosting](https://firebase.google.com/docs/hosting), which provides secure HTTPS by default, but you can use a different service if you prefer.

If you'd rather get started with a working example, you can find the full source code for this project in our [GitHub examples repo](https://github.com/talkjs/talkjs-examples/tree/master/firebase/notifications-example).

<div style="background-color: #F7F7F7; padding: 15px;">
  <p><strong>Note:</strong> This tutorial assumes you're already familiar with TalkJS basics. If you're new to TalkJS, we recommend starting with our <a href="https://talkjs.com/docs/Getting_Started/JavaScript_SDK/1_On_1_Chat/">getting started guide</a>.</p>
</div>

## Contents

- [Prerequisites](#prerequisites)
- [Set up Firebase](#set-up-firebase)
  - [Step 1: Initialize Firebase](#step-1-initialize-firebase)
  - [Step 2: Create a new Firebase app](#step-2-create-a-new-firebase-app)
  - [Step 3: Generate your public key](#step-3-generate-your-public-key)
- [Create your web application](#create-your-web-application)
  - [Step 1: Create a TalkJS chatbox](#step-1-create-a-talkjs-chatbox)
  - [Step 2: Add the Firebase event listener](#step-2-add-the-firebase-event-listener)
  - [Step 3: Create the service worker](#step-3-create-the-service-worker)
  - [Step 4: Style the app](#step-4-style-the-app)
- [Test your app](#test-your-app)
  - [Step 1: Deploy your application to Firebase hosting](#step-1-deploy-your-application-to-firebase-hosting)
  - [Step 2: Send a message from another user](#step-2-send-a-message-from-another-user)
- [Conclusion](#conclusion)

## Prerequisites

To follow along with this tutorial, you'll need:

- A [TalkJS account](https://talkjs.com/dashboard/login)
- A [Firebase account](https://console.firebase.google.com/) with a Firebase project set up. If you don't yet have a Firebase project, follow [their guides](https://firebase.google.com/docs/guides?_gl=1*qyyupu*_up*MQ..*_ga*MTQxMDg5NTA2Ni4xNzM4MjQ2OTI5*_ga_CW55HF8NVT*MTczODI0NjkyOS4xLjAuMTczODI0NjkyOS4wLjAuMA..) to get started.
- Firebase Cloud Messaging configured for your TalkJS account. For instructions on how to set this up in the TalkJS dashboard, see [Configure Firebase Cloud Messaging](https://talkjs.com/docs/Features/Notifications/Mobile_Push_Notifications/Configure_FCM/).
- The [Firebase CLI](https://firebase.google.com/docs/cli#install_the_firebase_cli)

## Set up Firebase

In this section, you'll initialize Firebase Hosting and configure the necessary settings.

### Step 1: Initialize Firebase Hosting

Log in to Firebase with the Firebase CLI if you haven't already:

```sh
firebase login
```

Next, create a new directory for your code for this project and run the following command inside it:

```sh
firebase init hosting
```

Answer the questions when prompted:

- Select to associate your new project directory with an existing Firebase project or create a new one. If you create a new one, or you haven't yet set up Firebase Cloud Messaging in the TalkJS dashboard, see [Configure Firebase Cloud Messaging](https://talkjs.com/docs/Features/Notifications/Mobile_Push_Notifications/Configure_FCM/) in our docs.
- Select to use the default (`public`) as your public directory
- Select Yes for "Configure as a single-page app?"
- Choose whether to set up automatic builds and deploys with GitHub

Once initialization is complete, you should see some config files in your directory, along with a `public` directory with an `index.html` file. We'll update this file in a later step.

### Step 2: Create a new Firebase app

Next, open the [Firebase console](http://console.firebase.google.com/) and select your Firebase project. Create a new app inside the Firebase project:

- Open the [Firebase console](http://console.firebase.google.com/) and select your project
- From the **Project Overview** page, click the **+** icon to add a new web app
- Pick a name for your app.
- After registering the app name, you will see some code samples for adding the Firebase SDK. Copy and save the `firebaseConfig` JavaScript object. It will look something like this:
  ```json
  {
    "apiKey": "<FIREBASE_API_KEY>",
    "authDomain": "<FIREBASE_APP_NAME>.firebaseapp.com",
    "projectId": "<FIREBASE_APP_NAME>",
    "storageBucket": "<FIREBASE_APP_NAME>.firebasestorage.app",
    "messagingSenderId": "<SENDER_ID>",
    "appId": "<FIREBASE_APP_ID>"
  }
  ```
  You'll use this in a later section when you initialize the Firebase SDK.

### Step 3: Generate your public key

You'll need a key to encrypt push notifications. To generate this, follow these steps:

- In the Firebase console, click the gear icon and go to **Project Settings**.
- In the **Cloud Messaging** tab, go to the **Web configuration section** and then click **Generate Key Pair**.
- Copy the public key. You'll use this in the next section.

## Create your web application

In this section, you'll create a TalkJS chat application and set up a service worker to handle Firebase Cloud Messaging.

### Step 1: Create a TalkJS chatbox

First, create a page with a TalkJS [chatbox](https://talkjs.com/docs/Features/Chat_UI_Modes/The_Chatbox/). For more detail on how this works, see our [getting started guide](https://talkjs.com/docs/Getting_Started/JavaScript_SDK/1_On_1_Chat/).

Replace the `index.html` file in your `public` directory with the following:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Example: TalkJS with Firebase Cloud Messaging notifications</title>

    <!-- prettier-ignore -->
    <script>
      (function(t,a,l,k,j,s){
      s=a.createElement('script');s.async=1;s.src='https://cdn.talkjs.com/talk.js';a.head.appendChild(s)
      ;k=t.Promise;t.Talk={v:3,ready:{then:function(f){if(k)return new k(function(r,e){l.push([f,r,e])});l
      .push([f])},catch:function(){return k&&new k()},c:l}};})(window,document,[]);
    </script>

    <script>
      Talk.ready.then(function () {
        var me = new Talk.User({
          id: "sample_user_sebastian",
          name: "Sebastian",
          role: "default",
        });
        var other = new Talk.User({
          id: "sample_user_alice",
          name: "Alice",
          role: "default",
        });

        window.talkSession = new Talk.Session({
          appId: "<APP_ID>", // update this with your TalkJS app ID
          me: me,
        });

        var conversation = talkSession.getOrCreateConversation(
          "sample_conversation"
        );

        conversation.setParticipant(me);
        conversation.setParticipant(other);

        var chatbox = talkSession.createChatbox();

        chatbox.select(conversation);
        chatbox.mount(document.getElementById("talkjs-container"));
      });
    </script>
  </head>
  <body>
    <div id="talkjs-container" style="height: 700px"></div>
  </body>
</html>
```

Replace `<APP_ID>` with your TalkJS app ID, which you can find in the **Settings** tab of the [TalkJS dashboard](https://talkjs.com/dashboard).

Now run the following command from the root directory of the project to deploy your site:

```sh
firebase deploy --only hosting
```

This will deploy the website at a URL like `https://<FIREBASE_APP_NAME>.web.app`. You should see a chatbox with a conversation between two users:

<!-- TODO: add 2-chatbox.jpg -->
<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="<ALT>"/>
</figure>

### Step 2: Add the Firebase event listener

Next, we'll load and initialize Firebase. In the `head` tag of `index.html`, add the following scripts to load the [Firebase App SDK](https://firebase.google.com/docs/reference/js/app.md?_gl=1*rwe4iq*_up*MQ..*_ga*MTEzODkxNDU1LjE3MzgzMzA0NjY.*_ga_CW55HF8NVT*MTczODMzMDQ2NS4xLjAuMTczODMzMDQ2NS4wLjAuMA..#app_package) and the [Firebase Cloud Messaging SDK](https://firebase.google.com/docs/reference/js/messaging_?_gl=1*u25wvd*_up*MQ..*_ga*MTEzODkxNDU1LjE3MzgzMzA0NjY.*_ga_CW55HF8NVT*MTczODMzMDQ2NS4xLjAuMTczODMzMDQ2NS4wLjAuMA..):

```html
<!-- update the version number as needed -->
<script defer src="/__/firebase/11.1.0/firebase-app-compat.js"></script>
<!-- include only the Firebase features as you need -->
<script defer src="/__/firebase/11.1.0/firebase-messaging-compat.js"></script>
```

Then add the following elements to the `body` of `index.html`, above the TalkJS container:

```html
<p class="load" id="load">Firebase SDK loading&hellip;</p>
<p class="load" id="token"></p>

<div style="display: flex; justify-content: center">
  <button id="requestToken">Request token</button>
</div>
```

This adds a button to your site that will request the token. Add the following script to the end of the `body` to initialize the Firebase SDK and create an event listener for the button:

```html
<script>
  document.addEventListener("DOMContentLoaded", function () {
    const loadEl = document.querySelector("#load");
    // Initialize Firebase SDK
    try {
      // Update this with the firebaseConfig object from your Firebase app
      let app = firebase.initializeApp({
        apiKey: "",
        authDomain: "",
        projectId: "",
        storageBucket: "",
        messagingSenderId: "",
        appId: "",
      });
      let features = [
        "auth",
        "database",
        "firestore",
        "functions",
        "messaging",
        "storage",
        "analytics",
        "remoteConfig",
        "performance",
      ].filter((feature) => typeof app[feature] === "function");
      loadEl.textContent = `Firebase SDK loaded with ${features.join(", ")}`;
    } catch (e) {
      console.error(e);
      loadEl.textContent = "Error loading the Firebase SDK, check the console.";
    }

    const messaging = firebase.messaging(firebase.app());

    document.getElementById("requestToken").addEventListener("click", () => {
      messaging
        .getToken({
          vapidKey: "<FIREBASE_PUBLIC_KEY>", // update this with your Firebase project public key
        })
        .then((token) => {
          window.talkSession.setPushRegistration({
            provider: "fcm",
            pushRegistrationId: token,
          });

          document.getElementById("token").textContent = token;
        });
    });
  });
</script>
```

Replace the placeholder object in `firebase.initializeApp` with the `firebaseConfig` object that you saved in Step 2 of the previous section.

Also, replace `<FIREBASE_PUBLIC_KEY>` with the public key that you saved in Step 3 of the previous section.

When you press the **Request token** button, the event listener will get your token with Firebase's [`messaging.getToken` method](https://firebase.google.com/docs/reference/js/messaging_.md?_gl=1*13rj7jn*_up*MQ..*_ga*MjA5OTY5NTk5OS4xNzM4MjM4MTc5*_ga_CW55HF8NVT*MTczODIzODE3OC4xLjAuMTczODIzODE3OC4wLjAuMA..#gettoken_b538f38). It will then call TalkJS's [`setPushRegistration` method](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/Session/?ref=talkjs.com#Session__setPushRegistration) with the token to register for FCM push notifications.

To receive these notifications, you will need a [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API). By default, Firebase will look for a service worker file called `firebase-messaging-sw.js` in the `public` directory. We'll create this file in the next step.

### Step 3: Create the service worker

Create a file called `firebase-messaging-sw.js` in your `public` directory and add the following:

```js
importScripts(
  "https://www.gstatic.com/firebasejs/11.1.0/firebase-app-compat.js"
);
importScripts(
  "https://www.gstatic.com/firebasejs/11.1.0/firebase-messaging-compat.js"
);

// Update this with the firebaseConfig object from your Firebase app
firebase.initializeApp({
  apiKey: "",
  authDomain: "",
  projectId: "",
  storageBucket: "",
  messagingSenderId: "",
  appId: "",
});

// Retrieve an instance of Firebase Messaging so that it can handle background
// messages.
const messaging = firebase.messaging();

// Handle incoming messages. Called when:
// - a message is received while the app has focus
// - the user clicks on an app notification created by a service worker
//   `messaging.onBackgroundMessage` handler.
messaging.onMessage((payload) => {
  console.log("Message received. ", payload);
  // ...
});

messaging.onBackgroundMessage((payload) => {
  console.log(
    "[firebase-messaging-sw.js] Received background message ",
    payload
  );

  const data = JSON.parse(payload.data["talkjs"]);
  console.log("TalkJS payload: ", data);

  // Customize notification here
  const notificationTitle = `Message from ${data.sender.name}`;
  const notificationOptions = {
    body: data.message.text,
    icon: "/firebase-logo.png",
  };

  self.registration.showNotification(notificationTitle, notificationOptions);
});
```

As with `index.html`, replace the placeholder object in `firebase.initializeApp` with the `firebaseConfig` object that you saved in Step 2 of the previous section.

This service worker handles incoming messages and adds logging and custom notification text.

### Step 4: Style the app

Finally, let's add some styling to the page. In the `head` tag of `index.html`, add the following script:

```html
<style media="screen">
  body {
    /* background: white; */
    color: rgba(0, 0, 0, 0.87);
    font-family: Roboto, Helvetica, Arial, sans-serif;
    margin: 0;
    padding: 0;
  }
  .load {
    color: #525252;
    text-align: center;
    font-size: 13px;
  }
  #requestToken {
    margin: 0px 0px 12px;
    padding: 8px 16px;
    border-radius: 4px;
    background: #007df9;
    color: white;
    border: none;
    cursor: pointer;
  }
  @media (max-width: 600px) {
    body {
      margin-top: 0;
      background: white;
      box-shadow: none;
    }
  }
</style>
```

This styles the **Request token** button and the loading

## Test your app

Let's verify that your push notification setup is working correctly by redeploying the application and sending a message.

### Step 1: Deploy your application to Firebase hosting

Redeploy your site to Firebase Hosting:

```sh
firebase deploy --only hosting
```

This will deploy your website to a URL like `https://<APP_NAME>.web.app`.

You should now see a TalkJS chatbox and a **Request token** button:

<!-- TODO: add 3-chatbox-with-token-button.jpg -->

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="<ALT>"/>
</figure>

### Step 2: Send a message from another user

TalkJS only sends notifications where you are currently not viewing a conversation, to save users from getting unnecessary notifications. For more information on exactly when TalkJS sends notifications, see [our notifications docs](https://talkjs.com/docs/Features/Notifications/#when-are-notifications-sent). So to test notifications, you will need to switch to another tab and then send a message from a different user:

- On the device where you want to receive push notifications, go to `https://<APP_NAME>.web.app` and click the **Request token** button. Select to allow push notifications if prompted.
- Switch to a different website in another different tab
- Add a new message to the TalkJS `sample_conversation` conversation from the other user, `sample_user_alice`. A simple way to do this is to go to the **Chat UI** tab of the TalkJS dashboard](https://talkjs.com/dashboard) and add a message from the **Preview** chatbox UI:

<!-- TODO: add 4-dashboard-test.jpg -->

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="A chatbox preview in the TalkJS dashboard, with conversation of sample_conversation and current user ID of sample_user_alice. Alice has added the message 'Hi, just testing notifications!'">
</figure>

You should now receive a push notification for the message:

<!-- TODO: add 5-push-notification.jpg -->

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="An example push notification with a subject of 'Message from Alice' and a message of 'Hi, just testing notifications!'"/>
</figure>

## Conclusion

You now have a working implementation of web push notifications using Firebase Cloud Messaging and TalkJS. Your users will receive notifications when new messages arrive, even when they're not actively using your chat application.

To recap, you have:

- Set up Firebase Hosting
- Created a new Firebase app
- Created a web application with a TalkJS chatbox
- Added a button to register for Firebase Cloud Messaging
- Added a service worker to handle messages
- Tested that notifications worked

For the full example code for this tutorial, see our [GitHub examples repo](https://github.com/talkjs/talkjs-examples/tree/master/firebase.notifications-example). Or for more ideas on how to use TalkJS, browse our [tutorials](https://talkjs.com/resources/tag/tutorials/) and [demos](https://talkjs.com/demo/).

---

If you're ready to add chat and notifications to your application, [try TalkJS for free](https://talkjs.com/dashboard/signup/premium/).
