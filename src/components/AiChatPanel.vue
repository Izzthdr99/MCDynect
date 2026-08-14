<script setup>
/**
 * AiChatPanel.vue
 * Figma source (collapsed tab): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1419-32822
 * Figma source (expanded panel, welcome state): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1430-41424
 * Figma source (expanded panel, active thread state): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1419-39256
 *
 * Global AI chat entry point, mounted once at the app root and shown on every
 * page except login/workspace-selection (see App.vue). The expanded state is
 * a full-viewport takeover (chat history sidebar + navbar + welcome/thread
 * view), per the "New Chat - Default" Figma frame — not a docked side drawer.
 */
import { nextTick, ref, watch } from 'vue'
import iconSparkle from '../assets/dashboard/icons/icon-ai-chat-sparkle.svg'
import iconArrowRight from '../assets/dashboard/icons/icon-ai-chat-arrow-right.svg'
import iconPlus from '../assets/dashboard/icons/icon-ai-chat-plus.svg'
import iconClose from '../assets/dashboard/icons/icon-ai-chat-close.svg'
import iconSendArrow from '../assets/dashboard/icons/icon-ai-chat-send-arrow.svg'
import iconMsgEdit from '../assets/dashboard/icons/icon-ai-chat-msg-edit.svg'
import iconMsgCopy from '../assets/dashboard/icons/icon-ai-chat-msg-copy.svg'

const isOpen = ref(false)
const draft = ref('')
const isTyping = ref(false)
const messagesEl = ref(null)
const inputEl = ref(null)
const messages = ref([])
const editingId = ref(null)

let messageId = 0

let historyId = 0
function toHistoryItems(texts) {
  return texts.map((text) => ({ id: historyId++, text }))
}

const chatHistoryGroups = ref([
  {
    label: 'Today',
    items: toHistoryItems([
      'Who have a higher sale',
      'Which marketing channel drives most sales?',
      'Monthly sales growth comparison',
      'Highest grossing month in the year',
      'Top-performing sales region this quarter',
      'Customer segment with highest lifetime value',
      'Which product category leads in revenue?',
      'Comparing online vs in-store sales',
      'Best-selling product in the last campaign',
      'Sales trends by demographic groups',
      'Who have a higher sale',
      'Which salesperson closed the most deals?',
    ]),
  },
  {
    label: 'Yesterday',
    items: toHistoryItems([
      'Who’s got the higher sales?',
      'Which product showed the fastest growth this quarter?',
      'Who leads in customer satisfaction ratings?',
      'Which region has the highest revenue this year?',
    ]),
  },
])

function removeHistoryItem(id) {
  for (const group of chatHistoryGroups.value) {
    group.items = group.items.filter((item) => item.id !== id)
  }
}

function startNewChat() {
  messages.value = []
  draft.value = ''
  editingId.value = null
}

async function scrollToBottom() {
  await nextTick()
  if (messagesEl.value) {
    messagesEl.value.scrollTop = messagesEl.value.scrollHeight
  }
}

function sendMessage() {
  const text = draft.value.trim()
  if (!text) return

  if (editingId.value !== null) {
    const index = messages.value.findIndex((m) => m.id === editingId.value)
    if (index !== -1) {
      messages.value[index].text = text
      messages.value = messages.value.slice(0, index + 1)
    }
    editingId.value = null
  } else {
    messages.value.push({ id: messageId++, role: 'user', text })
  }

  draft.value = ''
  scrollToBottom()

  // Placeholder reply — swap this block for a real API call.
  isTyping.value = true
  setTimeout(() => {
    isTyping.value = false
    messages.value.push({
      id: messageId++,
      role: 'assistant',
      text: 'Thanks for the message! I’m not wired up to live data yet, but this is where my answer will appear.',
      copied: false,
    })
    scrollToBottom()
  }, 900)
}

async function editMessage(id) {
  const message = messages.value.find((m) => m.id === id)
  if (!message) return

  editingId.value = id
  draft.value = message.text
  await nextTick()
  inputEl.value?.focus()
}

async function copyMessage(msg) {
  try {
    await navigator.clipboard.writeText(msg.text)
    msg.copied = true
    setTimeout(() => {
      msg.copied = false
    }, 1500)
  } catch {
    // Clipboard API unavailable (e.g. insecure context) — ignore silently.
  }
}

watch(isOpen, (open) => {
  if (open) scrollToBottom()
})
</script>

<template>
  <div class="ai-chat">
    <Transition name="tab-fade">
      <button
        v-if="!isOpen"
        type="button"
        class="chat-tab"
        aria-label="Open MC Dynect AI chat"
        @click="isOpen = true"
      >
        <img class="chat-tab-icon" :src="iconSparkle" alt="" />
        <span class="chat-tab-label">Dyxus AI</span>
      </button>
    </Transition>

    <Transition name="backdrop-fade">
      <div v-if="isOpen" class="chat-backdrop" @click="isOpen = false"></div>
    </Transition>

    <Transition name="overlay-slide">
      <div v-if="isOpen" class="chat-overlay">
        <button type="button" class="collapse-tab" aria-label="Collapse MC Dynect AI chat" @click="isOpen = false">
          <img class="collapse-tab-icon" :src="iconArrowRight" alt="" />
          <span class="collapse-tab-label">Dyxus AI</span>
        </button>

        <section class="chat-panel" role="dialog" aria-modal="false" aria-label="MC Dynect AI chat">
          <aside class="chat-history">
            <div class="chat-history-list">
              <div v-for="group in chatHistoryGroups" :key="group.label" class="chat-history-group">
                <p class="chat-history-label">{{ group.label }}</p>
                <div v-for="item in group.items" :key="item.id" class="chat-history-item">
                  <span class="chat-history-item-text">{{ item.text }}</span>
                  <button
                    type="button"
                    class="chat-history-item-delete tooltip-left"
                    aria-label="Delete chat"
                    data-tooltip="Delete chat"
                    @click="removeHistoryItem(item.id)"
                  >
                    <svg viewBox="0 0 10.6667 13.3333" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M3.05719 0.390524C3.30724 0.140476 3.64638 0 4 0H6.66667C7.02029 0 7.35943 0.140476 7.60948 0.390524C7.85952 0.640573 8 0.979711 8 1.33333V2.66667H10C10.3682 2.66667 10.6667 2.96514 10.6667 3.33333C10.6667 3.70152 10.3682 4 10 4V12C10 12.3536 9.85953 12.6928 9.60948 12.9428C9.35943 13.1929 9.02029 13.3333 8.66667 13.3333H2C1.64638 13.3333 1.30724 13.1929 1.05719 12.9428C0.807142 12.6928 0.666667 12.3536 0.666667 12V4C0.298477 4 0 3.70152 0 3.33333C0 2.96514 0.298477 2.66667 0.666667 2.66667H2.66667V1.33333C2.66667 0.979711 2.80714 0.640573 3.05719 0.390524ZM4 2.66667H6.66667V1.33333H4V2.66667ZM2 4V12H8.66667V4H2ZM4 4.66667C4.36819 4.66667 4.66667 4.96514 4.66667 5.33333V10.6667C4.66667 11.0349 4.36819 11.3333 4 11.3333C3.63181 11.3333 3.33333 11.0349 3.33333 10.6667V5.33333C3.33333 4.96514 3.63181 4.66667 4 4.66667ZM6.66667 4.66667C7.03486 4.66667 7.33333 4.96514 7.33333 5.33333V10.6667C7.33333 11.0349 7.03486 11.3333 6.66667 11.3333C6.29848 11.3333 6 11.0349 6 10.6667V5.33333C6 4.96514 6.29848 4.66667 6.66667 4.66667Z"
                        fill="currentColor"
                      />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
          </aside>

          <div class="chat-main">
            <header class="chat-navbar">
              <button type="button" class="new-chat-btn" @click="startNewChat">
                <img :src="iconPlus" alt="" />
                <span>New chat</span>
              </button>
              <button
                type="button"
                class="chat-navbar-close tooltip-bottom"
                aria-label="Close chat"
                data-tooltip="Close chat"
                @click="isOpen = false"
              >
                <img :src="iconClose" alt="" />
              </button>
            </header>

            <div v-if="messages.length === 0" class="chat-welcome">
              <div class="chat-welcome-inner">
                <div class="chat-welcome-heading">
                  <p class="chat-welcome-greeting">Hey, <span class="wave">👋</span></p>
                  <p>What can I help you with today?</p>
                </div>

                <div class="ai-chatbox">
                  <input
                    ref="inputEl"
                    v-model="draft"
                    type="text"
                    placeholder="Ask anything"
                    aria-label="Message"
                    @keydown.enter="sendMessage"
                  />
                  <button
                    type="button"
                    class="chatbox-icon-btn chatbox-icon-btn-brand chatbox-icon-btn-send"
                    :disabled="!draft.trim()"
                    aria-label="Send message"
                    data-tooltip="Send message"
                    @click="sendMessage"
                  >
                    <img class="send-icon" :src="iconSendArrow" alt="" />
                  </button>
                </div>
              </div>
            </div>

            <template v-else>
              <div class="chat-thread">
                <div ref="messagesEl" class="chat-panel-messages">
                  <div class="thread-column">
                    <TransitionGroup name="message-pop" tag="div" class="messages-group">
                      <div v-for="msg in messages" :key="msg.id" class="message" :class="msg.role">
                        <template v-if="msg.role === 'user'">
                          <div class="message-bubble">
                            <p class="message-text">&ldquo;{{ msg.text }}&rdquo;</p>
                          </div>
                          <button
                            type="button"
                            class="message-icon-btn message-edit-btn"
                            aria-label="Edit message"
                            data-tooltip="Edit message"
                            @click="editMessage(msg.id)"
                          >
                            <img :src="iconMsgEdit" alt="" />
                          </button>
                        </template>
                        <template v-else>
                          <p class="message-text">{{ msg.text }}</p>
                          <button
                            type="button"
                            class="message-icon-btn message-copy-btn"
                            :class="{ 'is-active': msg.copied }"
                            aria-label="Copy response"
                            :data-tooltip="msg.copied ? 'Copied!' : 'Copy response'"
                            @click="copyMessage(msg)"
                          >
                            <img :src="iconMsgCopy" alt="" />
                          </button>
                        </template>
                      </div>
                    </TransitionGroup>

                    <div v-if="isTyping" class="typing-indicator" aria-live="polite">
                      <span></span><span></span><span></span>
                    </div>
                  </div>
                </div>

                <div class="chat-thread-input">
                  <div class="ai-chatbox ai-chatbox-docked">
                    <input
                      ref="inputEl"
                      v-model="draft"
                      type="text"
                      placeholder="Ask anything"
                      aria-label="Message"
                      @keydown.enter="sendMessage"
                    />
                    <button
                      type="button"
                      class="chatbox-icon-btn chatbox-icon-btn-brand chatbox-icon-btn-send"
                      :disabled="!draft.trim()"
                      aria-label="Send message"
                      @click="sendMessage"
                    >
                      <img class="send-icon" :src="iconSendArrow" alt="" />
                    </button>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </section>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.ai-chat {
  --brand: #b13444;
  --brand-strong: #54101a;
  --border: #e5e7eb;
  --ink: #101828;
  --heading: #24252d;
  --body-text: #4a5565;
  --body-text-subtle: #6a7282;
  --surface: #f9fafb;
}
.ai-chat * {
  box-sizing: border-box;
}

/* ---------- Collapsed idle tab ---------- */
.chat-tab {
  position: fixed;
  top: 50%;
  right: 0;
  z-index: 90;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  width: 44px;
  height: 143px;
  padding: 16px 10px;
  background: var(--brand);
  border: none;
  border-radius: 12px 0 0 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.05);
  cursor: pointer;
  transition: transform 200ms ease, box-shadow 200ms ease, background 150ms ease;
}
.chat-tab:hover,
.chat-tab:focus-visible {
  transform: translateY(-50%) scale(1.04);
  box-shadow: -6px 0 20px rgba(16, 24, 40, 0.18);
  background: var(--brand-strong);
  outline: none;
}
.chat-tab-icon {
  width: 16px;
  height: 19px;
  flex-shrink: 0;
}
.chat-tab-label {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: #ffffff;
  white-space: nowrap;
}
.tab-fade-enter-active,
.tab-fade-leave-active {
  transition: opacity 180ms ease, transform 180ms ease;
}
.tab-fade-enter-from,
.tab-fade-leave-to {
  opacity: 0;
  transform: translateY(-50%) translateX(12px);
}

/* ---------- Expanded overlay ---------- */
.chat-backdrop {
  position: fixed;
  inset: 0;
  z-index: 99;
  background: rgba(10, 13, 18, 0.7);
  cursor: pointer;
}
.backdrop-fade-enter-active,
.backdrop-fade-leave-active {
  transition: opacity 220ms ease;
}
.backdrop-fade-enter-from,
.backdrop-fade-leave-to {
  opacity: 0;
}

.chat-overlay {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  z-index: 100;
  display: flex;
  align-items: center;
  width: calc(100vw - 131px);
}

.overlay-slide-enter-active {
  transition: transform 300ms cubic-bezier(0.16, 1, 0.3, 1), opacity 300ms ease;
}
.overlay-slide-leave-active {
  transition: transform 220ms cubic-bezier(0.4, 0, 1, 1), opacity 220ms ease;
}
.overlay-slide-enter-from,
.overlay-slide-leave-to {
  transform: translateX(100%);
  opacity: 0;
}

/* Re-collapse tab attached to the left edge of the expanded overlay */
.collapse-tab {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  flex-shrink: 0;
  width: 44px;
  height: 147px;
  padding: 16px 10px;
  background: var(--brand);
  border: none;
  border-radius: 12px 0 0 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  cursor: pointer;
  transition: background 150ms ease;
}
.collapse-tab:hover,
.collapse-tab:focus-visible {
  background: var(--brand-strong);
  outline: none;
}
.collapse-tab-icon {
  width: 13.33px;
  height: 8.33px;
  flex-shrink: 0;
  transform: rotate(180deg);
}
.collapse-tab-label {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: #ffffff;
  white-space: nowrap;
}

.chat-panel {
  position: relative;
  align-self: stretch;
  flex: 1;
  min-width: 0;
  display: flex;
  overflow: hidden;
  background: #f9fafb;
  box-shadow: 0px 25px 50px -12px rgba(29, 41, 61, 0.25);
}

/* ---------- Chat history sidebar ---------- */
.chat-history {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  width: 296px;
  height: 100%;
  padding: 16px;
  background: #ffffff;
  border-right: 1px solid var(--border);
}

.new-chat-btn {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 8px 12px;
  border: none;
  border-radius: 12px;
  background: var(--brand);
  color: #ffffff;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  transition: background 150ms ease;
}
.new-chat-btn:hover {
  background: var(--brand-strong);
}
.new-chat-btn img {
  width: 16px;
  height: 16px;
}

.chat-history-list {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
  overflow-y: auto;
}
.chat-history-group {
  display: flex;
  flex-direction: column;
}

.chat-history-label {
  flex-shrink: 0;
  margin: 0;
  padding: 10px 12px;
  font-size: 12px;
  font-weight: 500;
  color: var(--body-text);
}

.chat-history-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 8px;
  cursor: default;
  transition: background 150ms ease;
}
.chat-history-item:hover {
  background: rgba(249, 223, 227, 0.5);
}
.chat-history-item-text {
  min-width: 0;
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-size: 14px;
  color: var(--ink);
}
.chat-history-item-delete {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 16px;
  height: 16px;
  padding: 0;
  border: none;
  background: transparent;
  color: #99a1af;
  cursor: pointer;
  opacity: 0.7;
  transition: color 150ms ease, opacity 150ms ease;
}
.chat-history-item:hover .chat-history-item-delete,
.chat-history-item-delete:hover {
  opacity: 1;
  color: var(--brand);
}
.chat-history-item-delete svg {
  width: 10.67px;
  height: 13.33px;
}

/* ---------- Main chat column ---------- */
.chat-main {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
  height: 100%;
}

.chat-navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-shrink: 0;
  height: 66px;
  padding: 12px;
  background: #ffffff;
  border-bottom: 1px solid var(--border);
}
.chat-navbar-close {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: 1px solid var(--border);
  border-radius: 12px;
  background: var(--surface);
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  cursor: pointer;
  transition: background 150ms ease, border-color 150ms ease;
}
.chat-navbar-close:hover,
.chat-navbar-close:focus-visible {
  background: #ffffff;
  border-color: var(--brand);
  outline: none;
}
.chat-navbar-close img {
  width: 8.17px;
  height: 8.17px;
}

/* ---------- Welcome / empty state ---------- */
.chat-welcome {
  display: flex;
  flex: 1;
  min-height: 0;
  align-items: center;
  justify-content: center;
  overflow-y: auto;
  padding: 40px 24px;
}
.chat-welcome-inner {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  width: 100%;
}
.chat-welcome-heading {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}
.chat-welcome-heading p {
  margin: 0;
  font-size: clamp(24px, 4vw, 36px);
  font-weight: 400;
  letter-spacing: -0.72px;
  line-height: 1.25;
  color: var(--heading);
  text-align: center;
}
.chat-welcome-greeting {
  display: flex;
  align-items: center;
  gap: 10px;
}
.wave {
  display: inline-block;
  font-size: 1.25em;
  transform-origin: 70% 70%;
  animation: wave 2.4s ease-in-out 1;
}
@keyframes wave {
  0%,
  100% {
    transform: rotate(0deg);
  }
  15% {
    transform: rotate(14deg);
  }
  30% {
    transform: rotate(-8deg);
  }
  45% {
    transform: rotate(14deg);
  }
  60% {
    transform: rotate(-4deg);
  }
  75% {
    transform: rotate(10deg);
  }
}

/* ---------- Shared "Ask anything" input box ---------- */
.ai-chatbox {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 608px;
  max-width: 100%;
  padding: 16px 20px;
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0px 4px 24px 0px rgba(0, 0, 0, 0.04);
}
.ai-chatbox-docked {
  margin: 0 auto;
}
.ai-chatbox input {
  flex: 1;
  min-width: 0;
  border: none;
  outline: none;
  box-shadow: none;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  color: var(--ink);
}
.ai-chatbox input:focus {
  outline: none;
  box-shadow: none;
}
.ai-chatbox input::placeholder {
  color: var(--body-text);
}

.chatbox-icon-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  padding: 6px;
  border: none;
  border-radius: 6px;
  background: transparent;
  cursor: pointer;
  transition: background 150ms ease, opacity 150ms ease, transform 120ms ease;
}
.chatbox-icon-btn:hover:not(:disabled) {
  background: var(--surface);
}
.chatbox-icon-btn:active:not(:disabled) {
  transform: scale(0.94);
}
.chatbox-icon-btn-brand {
  background: var(--brand);
}
.chatbox-icon-btn-brand:hover:not(:disabled) {
  background: var(--brand-strong);
}
.chatbox-icon-btn-brand:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
.chatbox-icon-btn-send {
  flex-shrink: 0;
  padding: 4px;
}
.chatbox-icon-btn .send-icon {
  width: 8.75px;
  height: 10px;
  transform: rotate(180deg) scaleX(-1);
}

/* ---------- Active thread state ---------- */
.chat-thread {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
}
.chat-panel-messages {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow-y: auto;
  padding: 24px 24px 0;
  background: var(--surface);
}
.thread-column {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 32px;
  width: 100%;
  max-width: 600px;
  padding-bottom: 24px;
  margin-top: auto;
}
.chat-thread-input {
  flex-shrink: 0;
  display: flex;
  justify-content: center;
  padding: 0 24px 24px;
  background: var(--surface);
}
.messages-group {
  display: contents;
}
.message {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
  max-width: 608px;
}
.message-text {
  margin: 0;
  font-size: 14px;
  line-height: 24px;
  color: var(--body-text);
  overflow-wrap: break-word;
}

.message.user {
  align-items: flex-end;
  align-self: flex-end;
  width: auto;
}
.message-bubble {
  background: #ffffff;
  border-bottom: 1px solid var(--border);
  border-radius: 9999px;
  padding: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
}

.message.assistant {
  align-items: flex-start;
  align-self: flex-start;
}

.message-icon-btn {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  padding: 4px;
  border: none;
  border-radius: 6px;
  background: transparent;
  opacity: 0.6;
  cursor: pointer;
  transition: background 150ms ease, opacity 150ms ease;
}
.message-edit-btn img {
  width: 13.5px;
  height: 13.5px;
}
.message-copy-btn img {
  width: 12px;
  height: 15px;
}
.message-icon-btn:hover,
.message-icon-btn.is-active {
  opacity: 1;
  background: var(--surface);
}
.message-icon-btn.is-active {
  background: rgba(145, 27, 45, 0.08);
}

.message-pop-enter-active {
  transition: opacity 220ms ease, transform 220ms cubic-bezier(0.16, 1, 0.3, 1);
}
.message-pop-enter-from {
  opacity: 0;
  transform: translateY(8px);
}

.typing-indicator {
  display: flex;
  align-items: center;
  gap: 4px;
  width: fit-content;
  max-width: 720px;
  margin: 12px auto 0;
  padding: 12px 14px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 14px;
  border-bottom-left-radius: 4px;
}
.typing-indicator span {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--body-text-subtle);
  animation: typing-bounce 1.2s infinite ease-in-out;
}
.typing-indicator span:nth-child(2) {
  animation-delay: 0.15s;
}
.typing-indicator span:nth-child(3) {
  animation-delay: 0.3s;
}
@keyframes typing-bounce {
  0%,
  80%,
  100% {
    transform: translateY(0);
    opacity: 0.4;
  }
  40% {
    transform: translateY(-4px);
    opacity: 1;
  }
}

/* ---------- Tooltips ---------- */
[data-tooltip] {
  position: relative;
}
[data-tooltip]::after,
[data-tooltip]::before {
  position: absolute;
  left: 50%;
  transform: translateX(-50%) translateY(2px);
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
  transition: opacity 150ms ease, transform 150ms ease;
  z-index: 20;
}
[data-tooltip]::after {
  content: attr(data-tooltip);
  bottom: calc(100% + 8px);
  padding: 5px 9px;
  border-radius: 6px;
  background: var(--ink);
  color: #ffffff;
  font-family: inherit;
  font-size: 12px;
  font-weight: 500;
  line-height: 1.3;
  white-space: nowrap;
  box-shadow: 0px 4px 12px rgba(16, 24, 40, 0.18);
}
[data-tooltip]::before {
  content: '';
  bottom: calc(100% + 3px);
  border: 5px solid transparent;
  border-top-color: var(--ink);
}
[data-tooltip]:hover::after,
[data-tooltip]:hover::before,
[data-tooltip]:focus-visible::after,
[data-tooltip]:focus-visible::before {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) translateY(0);
}
[data-tooltip].tooltip-bottom::after,
[data-tooltip].tooltip-bottom::before {
  top: calc(100% + 8px);
  bottom: auto;
}
[data-tooltip].tooltip-bottom::before {
  top: calc(100% + 3px);
  border-top-color: transparent;
  border-bottom-color: var(--ink);
}
[data-tooltip].tooltip-left::after,
[data-tooltip].tooltip-left::before {
  left: auto;
  right: 0;
  transform: translateY(2px);
}
[data-tooltip].tooltip-left:hover::after,
[data-tooltip].tooltip-left:hover::before,
[data-tooltip].tooltip-left:focus-visible::after,
[data-tooltip].tooltip-left:focus-visible::before {
  transform: translateY(0);
}

/* ---------- Responsive ---------- */
@media (max-width: 860px) {
  .chat-history {
    display: none;
  }
}
@media (max-width: 640px) {
  .chat-overlay {
    width: 100vw;
  }
  .collapse-tab-label {
    display: none;
  }
  .collapse-tab {
    width: 32px;
    padding: 16px 8px;
  }
}
</style>
