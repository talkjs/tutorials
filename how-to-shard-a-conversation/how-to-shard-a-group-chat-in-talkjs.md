# How to shard a group chat in TalkJS

Sharding is conventionally used in distributed systems to spread users across services to ensure consistent availability. Sharding involves distributing users between available services so that no one service experiences unnecessary loads at any given time. Using TalkJS, you can have group chats with up to 1250 users. But, if you require more users, you can shard a conversation and assign users to the sharded conversations.

<figure class="kg-image-card">
  <img class="kg-image" src="" alt="Users are randomly assigned to conversations."/>
  <figcaption>After entering their information, users are randomly assigned to conversations.</figcaption>
</figure>

To follow along, you’ll need:

- A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
- An existing TalkJS project using the [JavaScript Chat SDK](https://talkjs.com/docs/Reference/JavaScript_Chat_SDK/). See our [Getting Started](https://talkjs.com/docs/Getting_Started/) guide for an example of how to set this up.

This project uses a script to add three conversations with 2 users each. This is not required in a production scenario where you already have users and conversations set up. 

We’ll build up the feature step by step in the following sections. If you would rather see the complete example code, see the [GitHub repo]() for this tutorial.

## Executing the script to add users and conversations

The sample script contains four functions. The `createUsers()` method creates six users. The other three methods, `setupConversation1()`, `setupConversation2()`, and `setupConversation3()`, creates three conversations with two users each. Ensure that you enter your app ID in the `appId` field and your secret key in the `secretKey` field. You can find them both on your TalkJS dashboard. Note that each conversation has the same subject so that every user assigned feels like they're being added to the same conversation. 

## Setting the theme for your chat

Go to your TalkJS dashboard, and navigate to the *Chat UI* tab. Then, select the default role and select the *team_chat* theme. Click *Publish to live* to apply this theme to your default role.

## Adding a modal for users to enter the chat

After following the getting started guide, you should have a working chat. We must make some changes to it to allow users to enter their information to enter a group chat. Edit the `index.html` file to add a modal that appears when the page loads. We are using Bootstrap here, but you can use any library or framework you prefer. Below the `talkjs-container` div, add the following code:

```html
<div class="modal fade" id="userEntryFormModal" tabindex="-1" data-bs-backdrop="static">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="userEntryFormModalLabel">Enter your info</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <form id="myForm">
                <div class="modal-body">

                    <div class="mb-3">
                        <label for="userName" class="form-label">Name</label>
                        <input type="text" class="userName form-control">
                    </div>
                    <div class="mb-3">
                        <label for="userEmailAddress" class="form-label">Email address</label>
                        <input type="email" class="userEmailAddress form-control">
                    </div>
                    <div class="mb-3">
                        <label for="userProfilePicture" class="form-label">Profile Picture URL</label>
                        <input type="text" class="userProfilePicture form-control">
                    </div>
            </form>
            <button class="btn btn-primary" type="submit">Enter chat</button>
        </div>
        </form>
    </div>
</div>
```

This modal contains three fields to accept the user's name, email address, and a URL to a profile picture. Once the user submits this information, they enter the group chat. Go [here](https://getbootstrap.com/docs/5.0/getting-started/introduction/), to learn how to use Bootstrap in your project.

## Adding a new user to a chat

When the page loads first, the user must enter their name, email address, and a URL to their profile picture to enter the chat. On clicking *Enter Chat*, they are randomly added to one of the created group chats. Let's see how this works behind the scenes. If you are using the `index.js` file from the getting started guide, you must clear its contents to replace it with the following code explained below.

```javascript
let talkSession;
const userEntryFormModal = new bootstrap.Modal(document.getElementById('userEntryFormModal'), {})
const talkJSContainer = document.getElementById("talkjs-container");

if(sessionStorage.getItem("showModal") == "false")
  setMainConversation();
else{
  userEntryFormModal.show();
  talkJSContainer.style.display = "none";
}
```

Inside the `index.js` file, we have several functions. The first line creates a new variable called `talkSession`. This is to store the TalkJS session after creating it. Then, we create another `const` called `userEntryFormModal` to store the Bootstrap modal. Lastly, we create another `const` called `talkJSContainer` to retrieve the `div` to mount our chatbox to.

```javascript
document.getElementById("myForm").addEventListener("submit", async (event) => {
  event.preventDefault();
  const name = document.querySelector('.userName').value;
  const email = document.querySelector('.userEmailAddress').value;
  const profilePictureURL = document.querySelector('.userProfilePicture').value;
  const user = await createUser(name, email, profilePictureURL);
  const conversation = await getRandomConversation();  
  await assignUserToConversation(user, conversation);
  sessionStorage.setItem("currentUser", user.id);
  sessionStorage.setItem("currentConversation", conversation.id);
  sessionStorage.setItem("showModal", "false");
  userEntryFormModal.hide();  
});
```

The next few lines checks if the `sessionStorage` contains a key called `showModal`. If this is set to false, it means that the user has already entered a group chat. Otherwise, we must hide the talkJSContainer and display the modal for the user to enter their information.

Once the user enters their information and clicks submit, we retrieve the values from the form and do the following:

* Create a TalkJS user
* Retrieve a random conversation
* Assign the user to the random conversation
* Set the current user and conversation to `sessionStorage`

After setting the current user and conversation to the sessionStorage, we also set a key called `showModal` to the session and set it to false. Finally, we hide the modal.

### Creating a TalkJS User

Once the user enters their details and submits it, we create a TalkJS user using that information. Inside the `index.js` file, there's an asynchronous function called `createUser()` that does this. It accepts the name, email address, and URL to the profile picture and uses that to create a new user in TalkJS. The `id` field is assigned a random UUID using the `crypto` package in JavaScript. After creating the user, we create a new Talk session using the `appID` and the created user. Finally, we return the user to the calling function.

```javascript
async function createUser(name, email, profilePictureURL){
  await Talk.ready;
  const user = new Talk.User({
    id: crypto.randomUUID(),
    name: name,
    email: email,
    photoUrl: profilePictureURL,
    role: "default"
  });
  talkSession = window.talkSession = new Talk.Session({
    appId: "<YOUR_APP_ID>",
    me: user,
  });
  return user;
}
```

### Getting a random conversation

Inside the `index.js` file, there's an asynchronous function called `getRandomConversation()`. This function retrieves the three conversations we have in TalkJS and adds them into an array. Then, it uses the `Math.floor()` and `Math.random()` methods to return a random conversation from the array.

```javascript
async function getRandomConversation(){
  let conversationArray = [];
  let conversation1, conversation2, conversation3;
  conversation1 = await talkSession.getOrCreateConversation("sharded-conversation-1");
  conversation2 = await talkSession.getOrCreateConversation("sharded-conversation-2");
  conversation3 = await talkSession.getOrCreateConversation("sharded-conversation-3");
  conversationArray.push(conversation1, conversation2, conversation3);
  const randomIndex = Math.floor(Math.random() * conversationArray.length);
  return conversationArray[randomIndex];
}
```

### Assign user to the conversation

After creating a user and retrieving a random conversation, we must assign the user to that conversation. This is done inside the `assignUserToConversation()` method inside `index.js`. 

```javascript
async function assignUserToConversation(user, conversation){
  await Talk.ready;
  conversation.setParticipant(user);
  const chatbox = talkSession.createChatbox();
  chatbox.select(conversation);
  talkJSContainer.style.display = "block";
  chatbox.mount(talkJSContainer);
}
```

We first wait for TalkJS to load and then set the user to the conversation. Using the session created earlier, we create a chatbox and select the retrieved conversation. Finally, we set the `display` property of the `talkJSContainer` to `block` and mount the chatbox inside it.

## Setting the main conversation

Whatever you've done so far works well the first time you execute it. But as soon as you refresh the page, the modal appears for the user to enter their information and enter a group chat again. This shouldn't be the case. Remember adding the current user's and conversation's `id` to the session storage, along with a key called `showModal`? These are used to retrieve the created conversation and user from the user's session and display them rather than showing the modal on page refresh.

```javascript
async function setMainConversation(){
  await Talk.ready;
  const user = new Talk.User(sessionStorage.getItem("currentUser"));
  talkSession = window.talkSession = new Talk.Session({
    appId: "<YOUR_APP_ID>",
    me: user,
  });
  const chatbox = talkSession.createChatbox();
  const currentConversation = await talkSession.getOrCreateConversation(sessionStorage.getItem("currentConversation"));
  chatbox.select(currentConversation);
  chatbox.mount(talkJSContainer);
}
```

## Conclusion

You now have a working demonstration of how to shard a conversation! To recap, in this tutorial you have:

- Added a modal for the user to enter their details
- Create a TalkJS user based on the information entered
- Randomly assign the user to one of the existing conversations

For the full example code for this tutorial, see our [GitHub repo]().

If you want to learn more about TalkJS, here are some good places to start:

- The [TalkJS Docs](https://talkjs.com/docs/) help you get started with TalkJS.
- [TalkJS tutorials](https://talkjs.com/resources/tag/tutorials/) provide how-to guides for many common TalkJS use cases.
- The [talkjs-examples Github repo](https://github.com/talkjs/talkjs-examples) has larger complete examples that demonstrate how to integrate with other libraries and frameworks.