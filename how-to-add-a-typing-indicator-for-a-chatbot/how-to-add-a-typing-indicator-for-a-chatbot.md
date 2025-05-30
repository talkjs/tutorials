---
title: "How to add a typing indicator for an AI chatbot"
---

Typing indicators are a key feature of any chat user interface, and can enhance the user experience. This guide covers what typing indicators are, why they’re important, and how to add a typing indicator for an AI chatbot to your TalkJS chat.

To follow along with this guide, you need:

- A TalkJS account. TalkJS provides a ready-to-use chat client for your application. [Get started for free.](https://talkjs.com/dashboard/signup)
- An existing project with an AI chatbot, to which you’d like to add a typing indicator.

<div class="kg-card kg-callout-card kg-callout-card-grey"><div class="kg-callout-emoji">💡</div><div class="kg-callout-text">Are you getting started with <a href="https://talkjs.com/use-cases/ai-chatbots/" target="_blank" rel="noopener noreferrer"><u>AI chatbots</u></a> in your chat? Check out the guides to create a custom AI chatbot using OpenAI’s <a href="https://talkjs.com/resources/how-to-make-a-customizable-chatbot-frontend-with-talkjs-and-the-openai-api/" target="_blank" rel="noopener noreferrer"><u>ChatGPT</u></a>, Google’s <a href="https://talkjs.com/resources/create-a-chatbot-with-talkjs-and-gemini/" target="_blank" rel="noopener noreferrer"><u>Gemini</u></a>, or Anthropic’s <a href="https://talkjs.com/resources/how-to-integrate-claude-into-your-talkjs-chat-with-the-anthropic-api/" target="_blank" rel="noopener noreferrer"><u>Claude</u></a>.</div></div>

## What are typing indicators?

A typing indicator is the visual cue in a chat that informs you that another user is writing a message.

<figure class="kg-image-card">
  <img class="kg-image" src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdoC8syXl8poOsJthAWvLVS8gc_OW1kn1u57YFnbip6i_HOVX2g_0fr6n_qBVQo5aUmR_JIqip_Gx7inlUGhesWnCRj8IJOv969bpJHmsT-5wE2A0tgIRh6FVz1PwL9zw8jV3F8Tg?key=OYb6h5wE6xMJmnrFx6QHtZwJ" alt="A conversation in which one user asks: ‘Thanks! How long would it be?’. Three dots in a chat bubble show that the other user is typing."/>
  <figcaption>A graphical typing indicator using three dots</figcaption>
</figure>

Typing indicators can come in many different forms. A common form of typing icon are three dots (also called ‘ellipsis’) animated inside a chat bubble when someone is typing. But a typing indicator can also just display the user’s name with text such as ‘is typing…’.

<figure class="kg-image-card">
  <img class="kg-image" src="https://talkjs.com/resources/content/images/2025/02/typing-indicator-text-1.png" alt="A panel with the text ‘AI chatbot is typing…’."/>
  <figcaption>A text-based typing indicator</figcaption>
</figure>

Popular messaging apps, such as Slack, WhatsApp, Facebook Messenger, or Telegram, all make use of typing indicators.

## Why are typing indicators important?

Typing awareness indicators are an important feature of any live chat. Knowing that a person is typing can help users avoid sending messages at the same time. Moreover, such real-time [micro-interactions](https://www.interaction-design.org/literature/article/micro-interactions-ux) can make conversations feel more authentic. All in all, typing indicators give users a better, more effective chat experience.

Having a typing indicator is especially vital when you’re working with AI chatbots, which is getting increasingly common today. Statista research found that around [75% of respondents globally had used generative AI at work](https://www.statista.com/statistics/1482102/rate-of-generative-ai-utilization-globally/), and that more and more people are comfortable to [use chatbots in customer support](https://www.statista.com/statistics/1488691/engagement-with-gen-ai-chatbots-by-country-europe/). An AI bot, after receiving a prompt, might take a few seconds to compose its response. But because a typing bot doesn’t type its response inside the message field, it doesn’t trigger the built-in typing dots.

To make chatting with AI bots more effective, you can add a custom typing indicator to your chat. The next section outlines how to add an animated three dots typing indicator to your TalkJS chat.

## Add a real-time typing indicator for an AI chatbot

Here’s how to add a real-time typing indicator to your TalkJS chat.

### Step 1: Create a typing indicator theme component

Begin by creating a custom typing indicator component:

1.  In your [TalkJS dashboard](https://talkjs.com/dashboard/), go to the **Themes** page.
2.  In the sidebar panel, from the section **Custom Components**, select the **+** (plus) icon to add a new component.
3.  Enter a name for your component. This guide uses the name `BotTypingIndicator`. Select **OK** to save your new custom component.
4.  Add the following inside the `BotTypingIndicator` component:

```HTML
<!-- A typing indicator for a bot -->

<template>
    <div class="message-row by-other UserMessage">
        <Avatar photoUrl="https://talkjs.com/new-web/avatar-1.jpg" />

        <t:set showAuthor="{{ true }}" />
        <div class="message {{ showAuthor | then: 'has-author' }}">
            <!-- in group chats, show the message sender name in a random color -->
            <div t:if="{{ showAuthor }}" class="message-author" style="color: black">
                AI chatbot
            </div>

            <div class="text timestamp-float-{{ floatTimestamp }}">
                <div class="typing-indicator">
                    <TypingIndicator />
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.message-row {
    margin-bottom: 16px;
    margin-left: 16px;
    margin-top: 0;
    display: flex;
    align-items: center;
}

.message {
    white-space: normal;
    overflow: hidden;
    border-radius: 0.75rem;
    border-width: 1px;
    border-style: solid;
    word-wrap: break-word;
    position: relative;
    display: inline-block;
    max-width: calc(100% - 6rem - 0.25rem - 0.25rem);
    border-color: #ececec;
    background-color: #f7f7f7;
    color: #111;
}

.by-other .message {
    margin-left: 0.50rem;
}

.message-author {
    font-size: 75%;
    font-weight: bold;
    padding: 0.75rem 1rem 0 1rem;
    margin-bottom: -0.75rem;
}

.text {
    padding: 0.75rem 1rem;
    white-space: pre-wrap;
}

</style>
```

Your changes are saved automatically.

This code does the following:

- Adds a user avatar and the display name `AI chatbot` to the component.
- Uses the built-in [typing indicator](/Features/Themes/Components/TypingIndicator/) component.
- Styles the typing indicator as belonging to the other user in the chat. You can adjust the styling to fit your own needs.

You now have a custom bot typing indicator component in your theme. But your typing indicator currently doesn’t show up yet. Let’s change that.

### Step 2: Detect when the bot is typing

To make the typing indicator show up whenever the bot is typing, you can use [custom conversation properties](/Reference/Concepts/Conversations/#custom). Custom conversation properties allow your theme to detect when the bot is typing.

How does this work? Let your theme check for a specific custom conversation property that’s present whenever the AI chatbot is typing. (You’ll set the custom property itself in [step 3](#step-3-set-a-custom-property-while-the-bot-is-typing).) If the custom property is present, then your theme shows the typing indicator. If the custom property isn’t present, then your theme doesn’t show the typing indicator.

Take the following steps:

1.  On the **Themes** page of your TalkJS dashboard, in the section **Built-in Components**, go to the `MessageField` built-in theme component.
2.  Add the following code to the top of the `MessageField` component, right after the opening of the `<template>` tag:

```HTML
<BotTypingIndicator t:if="{{ conversation.custom.typing == 'true' }}"/>

<!--- rest of the MessageField code --->
```

Your changes are saved automatically.

This code does the following:

- Detects if, for a given conversation, the custom property `typing` is `true`.
- If `typing` is `true`, then it shows your `BotTypingIndicator` custom theme component.
- If `typing` is either missing or set to `false`, then your theme shows no typing indicator.

Your theme is now ready to show the bot typing indicator whenever the custom property `typing` is true for a conversation.

But currently your conversations don’t have any custom property typing set to them yet. Let’s change that too, so that your theme can detect when to show the bot typing indicator.

### Step 3: Set a custom property while the bot is typing

To set a custom property whenever the bot is composing its response, you can use the REST API:

1.  When you receive the request to the chatbot, use the [REST API to set a custom property](/Reference/REST_API/Conversations/#setting-conversation-data) `typing` to `true` for your conversation. Your theme now automatically shows the custom typing indicator in the chat.
2.  Once your AI chatbot has generated its response, again use the REST API to set the custom property `typing` to `false`, or clear it altogether. The bot typing indicator then disappears from the chat.

The result should look something like the following:

<figure class="kg-image-card">
  <img class="kg-image" src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdyccJ2QIFlnJ3kI7S-OpwoKxwx6xY0BsI1PtjMtzMunOaDfrEkH7R7d_hi-jSQd-FGWHIrOnTGAhfkuUoun0DXHwvT8xjJ-w2YAGXIs2bzgX2418_uwVOtplSenFmMMpK964im0g?key=OYb6h5wE6xMJmnrFx6QHtZwJ" alt="A conversation with one message that states ‘Can you help? 🙂’, and another message from the AI chatbot with a typing indicator."/>
  <figcaption>A typing indicator for an AI chatbot</figcaption>
</figure>

This way, the bot’s typing indicator shows up whenever the bot is generating its response.

## Next steps

You’ve now added a real-time typing indicator for an AI chatbot to your theme. Whenever your bot is composing its response, your user knows that a reply is coming.

Would you like to customize your chat further? Use the [theme editor](/Features/Themes/), or check out the wide range of [code samples](/Reference/Code_Samples/) for more customization ideas.

---

Stuck? Got more questions on typing indicators for AI chatbots? We’re here to help! [Contact a TalkJS developer.](https://talkjs.com/?chat)
