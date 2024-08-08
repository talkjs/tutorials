# How to add chat to a Blazor Web app with TalkJS 

Do you want to quickly set up a chat in your Blazor application in minutes? If yes is your answer, then you’re in the right place.

In this tutorial we’ll demonstrate how to add a Chat API to a Blazor Web Application using TalkJS. TalkJS is a chat API that offers lots of features and is easy to integrate. Today, we are going to talk about integrating TalkJS to a Blazor application. Without further ado, let’s begin.

This tutorial assumes that you have knowledge of Blazor, .NET and also that you’re familiar with the basics of Javascript, HTML and CSS. 

To follow along, you’ll need:
-	A [TalkJS account](https://talkjs.com/dashboard/login). TalkJS provides a ready-to-use chat client for your application. Your account gives you access to TalkJS's free development environment.
-	A Blazor Web App in which you would like to add a chat to.

This Blazor application is created using the Blazor Web App template in Visual Studio, specifically Interactive server with Per page/component interactivity. The steps of creating one can be found in this Microsoft Blazor tutorial link. 

If you already have an existing application created using the Blazor Web App template, you can still follow along and tweak the render modes depending on where your component displaying the chat will be.

We’ll build up the chat step by step in the following sections. If you would rather see the complete example code, see the [GitHub repo -NOT UPDATED](https://github.com/MagnusMutai/tutorials/edit/main/how-to-add-chat-to-a-blazor-app-with-talkjs) for this tutorial. 
This is a tutorial just to get you started. For a fine-grained explanation, please follow the comments in the code.

# Initialize TalkJS 
In this section we are going to activate TalkJS. To do this, in app.razor, add the following script tag containing the JavaScript code that activates TalkJS in your Blazor application.

```html
<body>
    <Routes />
    <script src="_framework/blazor.web.js"></script>

    @* Initialize TalkJS *@
    @* Minified snippet to load TalkJS without delaying your page *@
    @* This code needs to be below the Talk.Wrapper Code above for it work appropriately *@
    <script>
        (function (t, a, l, k, j, s) {
            s = a.createElement('script'); s.async = 1; s.src = 'https://cdn.talkjs.com/talk.js'; a.head.appendChild(s)
                ; k = t.Promise; t.Talk = {
                    v: 3, ready: {
                        then: function (f) {
                            if (k) return new k(function (r, e) { l.push([f, r, e]) }); l
                                .push([f])
                        }, catch: function () { return k && new k() }, c: l
                    }
                };
        })(window, document, []);
    </script>

</body>
```

# Create a conversation

Next, in `app.razor` we will add another script tag that creates the actual conversation. This script will consist of a JavaScript wrapper called `TalkWrapper` (A JavaScript Wrapper is a function that calls one or more other functions.)  

Inside `TalkWrapper`, we are going to define a function called `createConversation`.This function contains the logic to create a conversation. It receives four parameters which will be provided by our C# code that will call it. Here is the code:

```html
<body>
    <Routes />
    <script src="_framework/blazor.web.js"></script>

    @* The actual code that creates a conversation *@
    <script>
        window.TalkWrapper = {

            createConversation: function (appId, myUser, otherUser, conversationId) {

                // Create a Talk.User object for me
                const me = new Talk.User(myUser);
                //Create a Talk.User object for other participant
                const other = new Talk.User(otherUser);

                // Create a Talk.Session object
                const talkSession = new Talk.Session({
                    appId: appId,
                    me: me,
                });

                // Create a conversation with a unique conversation ID
                const conversation = talkSession.getOrCreateConversation(conversationId);

                // Add myself as a participant
                conversation.setParticipant(me);

                // Add another participant
                conversation.setParticipant(other);

                const chatbox = talkSession.createChatbox();
                chatbox.select(conversation);
                // Mount a conversation into a given div
                //We're getting the div by id, make sure the div has the id specified i.e talkjs-container
                chatbox.mount(document.getElementById('talkjs-container'));
            }
        }

    </script>

    @* Initialize TalkJS *@
    @* Minified snippet to load TalkJS without delaying your page *@
    @* This code needs to be below the Talk.Wrapper Code above for it work appropriately *@
    <script>
        (function (t, a, l, k, j, s) {
            s = a.createElement('script'); s.async = 1; s.src = 'https://cdn.talkjs.com/talk.js'; a.head.appendChild(s)
                ; k = t.Promise; t.Talk = {
                    v: 3, ready: {
                        then: function (f) {
                            if (k) return new k(function (r, e) { l.push([f, r, e]) }); l
                                .push([f])
                        }, catch: function () { return k && new k() }, c: l
                    }
                };
        })(window, document, []);
    </script>


</body>
```
# Create a user model

That’s enough JavaScript for now. 

For the sake of simplicity, we are going write the rest of the C# code and HTML to display the component under the default Home.razor page.

Now, add a data model of a typical participant in a TalkJS conversation. To do so, create a class named User as shown below and add the given properties.

```c#
@code {

    //Model for a typical chat participant
    public class User
    {
        public long Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string PhotoUrl { get; set; } = string.Empty;
        public string WelcomeMessage { get; set; } = string.Empty;
    }
}
```

