# A conversation UI needs one scroll owner and a cancellable pending reply

Still is a small HTML/CSS/JavaScript conversation prototype. This note explains two choices in its implementation: keeping the message list scrollable without moving the composer, and ensuring a cancelled reply cannot appear in a different conversation.

**Disclosure:** This note and the prototype were created with Codex. Replies are simulated locally; there is no AI service, account system or persistent storage behind the demo.

[Try the live interface](https://moyuziya.github.io/still-conversation-ui-preview/).

## Give the message list permission to shrink

The central column is a vertical flex container. Its header and composer take their natural space; the message area takes the rest:

```css
.conversation {
  display: flex;
  flex-direction: column;
  min-width: 0;
  overflow: hidden;
}

.chat-scroll {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  overscroll-behavior: contain;
}
```

The explicit `min-height: 0` makes the intended shrink behavior clear. Automatic minimum sizing depends on the item and its overflow behavior; setting zero avoids relying on a content-based minimum. The composer stays outside the message list’s overflow region. See [MDN’s flex sizing reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/flex).

This is not a complete mobile-keyboard solution. Still uses `100dvh` and a 620px minimum height below its 800px breakpoint. On a shorter visible viewport, the whole page may need to scroll. Test your intended device, landscape orientation, zoom level and on-screen keyboard before treating a prototype as production-ready.

## Cancel pending work before changing context

The demo delays its simulated response with a timer. Resetting or switching conversations must cancel that timer first. Otherwise a late reply could arrive after the interface has moved on.

The implementation keeps one pending handle, cancelled through [clearTimeout](https://developer.mozilla.org/en-US/docs/Web/API/Window/clearTimeout):

```js
let pending = null;

function cancel() {
  if (pending !== null) {
    clearTimeout(pending);
    pending = null;
    document.getElementById('messages')
      .querySelector('.pending')?.remove();
    busy(false);
  }
}
```

`busy(false)` is the demo's helper that restores its send button, textarea and suggestion controls. This excerpt relies on that helper; it is not a standalone application.

When scheduling a reply, capture the conversation ID and mode rather than reading mutable global values later. Still also calls `cancel()` before new-conversation, reset, conversation-switch and style-switch operations. Capturing the ID and cancelling the work solve different problems: correct ownership and correct lifecycle.

For a real network-backed chat, replace the timer with request cancellation and an identity check before committing the result. Clearing a timer alone does not abort a network request.

## Review behavior before visual polish

A practical review sequence for a conversation prototype is:

1. Send a message, cancel it, and confirm no delayed assistant message appears.
2. Send again, switch conversations, and check that neither thread receives an orphaned reply.
3. Reset while a reply is pending.
4. Enter a long unbroken string and check horizontal overflow.
5. Use keyboard navigation and inspect visible focus.
6. Check narrow and short viewports separately; a narrow screenshot does not prove keyboard behavior.

The prototype uses `textContent` for message rendering. Keep that boundary if messages are plain text. Introducing rendered HTML or Markdown needs a separate sanitization decision.

## Download scope

The live demo is available to inspect before purchasing. The [editable Still package is $12](https://moyuziya.gumroad.com/l/xfjci?utm_source=github&utm_medium=technical_note&utm_campaign=still_layout_note). It includes source, setup notes, the fictional AI-generated portrait and a commercial end-product license. It excludes an AI backend, authentication, database, hosting and standalone template resale rights. No exclusive rights in the generated portrait are promised.

For a small customization, [open a scoped inquiry](https://github.com/moyuziya/still-conversation-ui-preview/issues). Describe the desired screen or behavior; price and timing must be agreed before work begins. Do not include credentials or private customer data.
