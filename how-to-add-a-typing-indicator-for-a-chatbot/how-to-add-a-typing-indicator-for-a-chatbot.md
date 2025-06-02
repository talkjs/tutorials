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

Popular messaging apps, such as Slack, WhatsApp, Facebook Messenger, or Telegram, all make use of typing indicators. TalkJS themes all provide a typing indicator that displays when other users are typing, and with a bit of customization, you can add it to chatbot responses too. As an example, try out the [interactive AI chatbot demo](https://talkjs.com/demo/ai-chatbot/) on TalkJS's website.

## Why are typing indicators important?

Typing awareness indicators are an important feature of any live chat. Knowing that a person is typing can help users avoid sending messages at the same time. Moreover, such real-time [micro-interactions](https://www.interaction-design.org/literature/article/micro-interactions-ux) can make conversations feel more authentic. All in all, typing indicators give users a better, more effective chat experience.

Having a typing indicator is especially vital when you’re working with AI chatbots, which is getting increasingly common today. Statista research found that around [75% of respondents globally had used generative AI at work](https://www.statista.com/statistics/1482102/rate-of-generative-ai-utilization-globally/), and that more and more people are comfortable to [use chatbots in customer support](https://www.statista.com/statistics/1488691/engagement-with-gen-ai-chatbots-by-country-europe/). An AI bot, after receiving a prompt, might take a few seconds to compose its response. But because a typing bot doesn’t type its response inside the message field, it doesn’t trigger the built-in typing dots.

To make chatting with AI bots more effective, you can add a custom typing indicator to your chat. The next section outlines how to add an animated three dots typing indicator to your TalkJS chat.

## Add a real-time typing indicator for an AI chatbot

Here’s how to add a real-time typing indicator to your TalkJS chat.

### Step 1: Add a bot typing indicator to your theme

First, go to the **Themes** tab, and select to **Edit** the theme you currently use. (In this tutorial we'll edit the `default` theme, but it would work similarly for the other themes.)

Select the `UserMessage` component from the list of **Built-in components**. Replace the existing `MessageBody` component with the following:

```html
<div t:if="{{ custom.isTyping == 'true' }}" class="typing-indicator">
  <TypingIndicator />
</div>

<MessageBody
  t:else
  body="{{ body }}"
  timestamp="{{ timestamp }}"
  floatTimestamp="auto"
  showStatus="{{ sender.isMe }}"
  isLongEmailMessage="{{isLongEmailMessage}}"
  darkenMenuArea="{{ darkenMenuArea }}"
  hasReferencedMessage="{{ hasReferencedMessage }}"
/>
```

This code adds TalkJS's `TypingIndicator` component to the user message if an `isTyping` [custom message property](https://talkjs.com/resources/Reference/Concepts/Messages/#custom) is set on the message. Otherwise, it displays the `MessageBody` as normal.

If you wanted you could instead create a custom typing indicator component for the bot in the Theme Editor, but we'll use the standard one that comes with the theme.

Your typing indicator won't show up yet. To fix that, we'll set the `isTyping` property to `true` on messages while the bot is typing.

### Step 2: Set a custom property while the bot is typing

To set a custom `isTyping` property whenever the bot is composing its response, you can use TalkJS's REST API in your backend server code:

1.  When you receive the request to the chatbot, use the REST API to [create a new message](https://talkjs.com/docs/Reference/REST_API/Messages/#send-a-message-on-behalf-of-a-user) with some placeholder text and a custom `isTyping` message property set to true. Your theme code from the previous section will replace the placeholder text with a typing indicator.
2.  Once your AI chatbot has generated its response, again use the REST API to [edit the message](https://talkjs.com/docs/Reference/REST_API/Messages/#edit-a-message). Set the text of the message to the bot's response, and set the custom property `isTyping` to `false`, or clear it altogether. Your theme code will then replace the typing indicator with a normal `MessageBody` that contains the text of the bot's response.

The exact implementation will depend on the details of your backend code, and the specific AI service that you use. For a Node.js example that uses OpenAI's API to generate responses, see our [Create a custom chatbot with TalkJS and the OpenAI API](https://talkjs.com/resources/how-to-make-a-customizable-chatbot-frontend-with-talkjs-and-the-openai-api/#optional-add-a-typing-indicator) tutorial, which gives detailed step-by-step instructions, and the accompanying [example code](https://github.com/talkjs/talkjs-examples/tree/master/chatbot-integration/openai-chatgpt).

### Step 3: Lock the conversation to read-only when the bot is typing

There's one final tweak that we need. Currently you can send more messages before the bot replies. As well as being potentially confusing, this can lead to having typing indicators appear above other messages. To prevent this, use the REST API to [update participation details](https://talkjs.com/resources/Reference/REST_API/Participation/#modify-participation) for the current user. Set the [`access`](https://talkjs.com/resources/Reference/Concepts/Participants/#access) field to `Read` while the bot generates its response, and back to `ReadWrite` when it finishes.

As before, the exact implementation details will depend on your server code. See the OpenAI [tutorial](https://talkjs.com/resources/how-to-make-a-customizable-chatbot-frontend-with-talkjs-and-the-openai-api/#optional-add-a-typing-indicator) and [example code](https://github.com/talkjs/talkjs-examples/tree/master/chatbot-integration/openai-chatgpt) for one way to do this.

The result should look something like the following:

<!-- TODO: upload 1-typing-indicator.jpg -->

<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="A conversation with a user message that asks ‘How do I try TalkJS?’, and a reply from the AI chatbot with a typing indicator. The message field displays 'You can read, but not send messages'."/>
  <figcaption>A typing indicator for an AI chatbot</figcaption>
</figure>

The typing indicator shows up whenever the bot is generating its response, and the message field is locked to read-only.

## Next steps

You’ve now added a real-time typing indicator for an AI chatbot to your theme. Whenever your bot is composing its response, your user knows that a reply is coming.

Would you like to customize your chat further? Use the [theme editor](/Features/Themes/), or check out the wide range of [code samples](/Reference/Code_Samples/) for more customization ideas.

---

Stuck? Got more questions on typing indicators for AI chatbots? We’re here to help! [Contact a TalkJS developer.](https://talkjs.com/?chat)
