<script setup>
/**
 * AiChatPanel.vue
 * Figma source (collapsed tab): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1419-32822
 * Figma source (expanded panel): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1419-38282
 *
 * Global AI chat entry point, mounted once at the app root and shown on every
 * page except login/workspace-selection (see App.vue). The expanded state is
 * a full-viewport takeover (chat history sidebar + navbar + welcome/thread
 * view), per the "New Chat - Default" Figma frame — not a docked side drawer.
 */
import { nextTick, ref, watch } from 'vue'
import iconSparkle from '../assets/dashboard/icons/icon-ai-chat-sparkle.svg'
import iconArrowRight from '../assets/dashboard/icons/icon-ai-chat-arrow-right.svg'
import iconNewChat from '../assets/dashboard/icons/icon-ai-chat-new-chat.svg'
import iconTrash from '../assets/dashboard/icons/icon-ai-chat-trash.svg'
import iconClose from '../assets/dashboard/icons/icon-ai-chat-close.svg'
import iconPlus from '../assets/dashboard/icons/icon-ai-chat-plus.svg'
import iconMic from '../assets/dashboard/icons/icon-ai-chat-mic.svg'
import iconSendArrow from '../assets/dashboard/icons/icon-ai-chat-send-arrow.svg'
import iconNavbarLogo from '../assets/dashboard/icons/icon-ai-chat-navbar-logo.svg'
import imgPatternBg from '../assets/dashboard/pattern-ai-chat-bg.png'

const isOpen = ref(false)
const draft = ref('')
const isTyping = ref(false)
const messagesEl = ref(null)
const messages = ref([])

let historyId = 0
const chatHistory = ref([
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
].map((text) => ({ id: historyId++, text })))

function removeHistoryItem(id) {
  chatHistory.value = chatHistory.value.filter((item) => item.id !== id)
}

function startNewChat() {
  messages.value = []
  draft.value = ''
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

  messages.value.push({ role: 'user', text })
  draft.value = ''
  scrollToBottom()

  // Placeholder reply — swap this block for a real API call.
  isTyping.value = true
  setTimeout(() => {
    isTyping.value = false
    messages.value.push({
      role: 'assistant',
      text: 'Thanks for the message! I’m not wired up to live data yet, but this is where my answer will appear.',
    })
    scrollToBottom()
  }, 900)
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
        <span class="chat-tab-label">MC Dynect AI</span>
      </button>
    </Transition>

    <Transition name="backdrop-fade">
      <div v-if="isOpen" class="chat-backdrop" @click="isOpen = false"></div>
    </Transition>

    <Transition name="overlay-slide">
      <div v-if="isOpen" class="chat-overlay">
        <button type="button" class="collapse-tab" aria-label="Collapse MC Dynect AI chat" @click="isOpen = false">
          <img class="collapse-tab-icon" :src="iconArrowRight" alt="" />
          <span class="collapse-tab-label">MC Dynect AI</span>
        </button>

        <section class="chat-panel" role="dialog" aria-modal="false" aria-label="MC Dynect AI chat">
          <div class="chat-panel-pattern" aria-hidden="true">
            <img :src="imgPatternBg" alt="" />
          </div>

          <aside class="chat-history">
            <button type="button" class="new-chat-btn" @click="startNewChat">
              <img :src="iconNewChat" alt="" />
              <span>New Chat</span>
            </button>

            <div class="chat-history-list">
              <p class="chat-history-label">Chats</p>
              <div v-for="item in chatHistory" :key="item.id" class="chat-history-item">
                <span class="chat-history-item-text">{{ item.text }}</span>
                <button
                  type="button"
                  class="chat-history-item-delete"
                  aria-label="Delete chat"
                  @click="removeHistoryItem(item.id)"
                >
                  <img :src="iconTrash" alt="" />
                </button>
              </div>
            </div>
          </aside>

          <div class="chat-main">
            <header class="chat-navbar">
              <img class="chat-navbar-logo" :src="iconNavbarLogo" alt="MC Dynect" />
              <button type="button" class="chat-navbar-close" aria-label="Close chat" @click="isOpen = false">
                <img :src="iconClose" alt="" />
              </button>
            </header>

            <div v-if="messages.length === 0" class="chat-welcome">
              <div class="chat-welcome-inner">
                <div class="chat-welcome-heading">
                  <p>Hey, <span class="wave">👋</span></p>
                  <p>What can I help you with today?</p>
                </div>

                <div class="ai-chatbox">
                  <div class="ai-chatbox-body">
                    <input
                      v-model="draft"
                      type="text"
                      placeholder="Ask anything"
                      aria-label="Message"
                      @keydown.enter="sendMessage"
                    />
                    <div class="ai-chatbox-actions">
                      <button type="button" class="chatbox-icon-btn chatbox-icon-btn-brand" aria-label="Attach">
                        <img :src="iconPlus" alt="" />
                      </button>
                      <div class="ai-chatbox-actions-right">
                        <button type="button" class="chatbox-icon-btn chatbox-icon-btn-mic" aria-label="Voice input">
                          <img :src="iconMic" alt="" />
                        </button>
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
                </div>
              </div>
            </div>

            <template v-else>
              <div ref="messagesEl" class="chat-panel-messages">
                <TransitionGroup name="message-pop" tag="div" class="messages-inner">
                  <div v-for="(msg, i) in messages" :key="i" class="message" :class="msg.role">
                    <p>{{ msg.text }}</p>
                  </div>
                </TransitionGroup>

                <div v-if="isTyping" class="typing-indicator" aria-live="polite">
                  <span></span><span></span><span></span>
                </div>
              </div>

              <div class="chat-panel-input-row">
                <div class="ai-chatbox ai-chatbox-docked">
                  <div class="ai-chatbox-body">
                    <input
                      v-model="draft"
                      type="text"
                      placeholder="Ask anything"
                      aria-label="Message"
                      @keydown.enter="sendMessage"
                    />
                    <div class="ai-chatbox-actions">
                      <button type="button" class="chatbox-icon-btn chatbox-icon-btn-brand" aria-label="Attach">
                        <img :src="iconPlus" alt="" />
                      </button>
                      <div class="ai-chatbox-actions-right">
                        <button type="button" class="chatbox-icon-btn chatbox-icon-btn-mic" aria-label="Voice input">
                          <img :src="iconMic" alt="" />
                        </button>
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
  --brand: #911b2d;
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
  background: var(--surface);
  box-shadow: 0px 25px 50px -12px rgba(29, 41, 61, 0.25);
}

.chat-panel-pattern {
  position: absolute;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
}
.chat-panel-pattern img {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 160%;
  height: 160%;
  min-width: 900px;
  min-height: 900px;
  transform: translate(-50%, -50%) rotate(-18deg);
  object-fit: cover;
  opacity: 0.18;
}
.chat-panel-pattern::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(249, 250, 251, 0.9), rgba(249, 250, 251, 0.5) 45%, rgba(255, 253, 248, 0.9));
}

/* ---------- Chat history sidebar ---------- */
.chat-history {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  gap: 24px;
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
  gap: 4px;
  width: 100%;
  height: 36px;
  border: none;
  border-radius: 8px;
  background: var(--brand);
  color: #ffffff;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 150ms ease;
}
.new-chat-btn:hover {
  background: var(--brand-strong);
}
.new-chat-btn img {
  width: 13.66px;
  height: 13.66px;
}

.chat-history-list {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
  overflow-y: auto;
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
  cursor: pointer;
  opacity: 0.7;
  transition: opacity 150ms ease;
}
.chat-history-item-delete:hover {
  opacity: 1;
}
.chat-history-item-delete img {
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
.chat-navbar-logo {
  height: 26px;
  width: auto;
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
.ai-chatbox-body {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 87px;
}
.ai-chatbox-docked .ai-chatbox-body {
  height: auto;
  gap: 10px;
}
.ai-chatbox input {
  width: 100%;
  border: none;
  outline: none;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  color: var(--ink);
}
.ai-chatbox input::placeholder {
  color: var(--body-text);
}
.ai-chatbox-actions {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.ai-chatbox-actions-right {
  display: flex;
  align-items: center;
  gap: 12px;
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
.chatbox-icon-btn img {
  width: 10.33px;
  height: 10.33px;
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
.chatbox-icon-btn-mic img {
  width: 10.33px;
  height: 14.33px;
}
.chatbox-icon-btn-send {
  padding: 4px;
}
.send-icon {
  width: 8.75px;
  height: 10px;
  transform: rotate(180deg) scaleX(-1);
}

/* ---------- Active thread state ---------- */
.chat-panel-messages {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  padding: 24px;
  background: var(--surface);
}
.messages-inner {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-width: 720px;
  margin: 0 auto;
}
.message {
  display: flex;
  max-width: 85%;
}
.message p {
  margin: 0;
  padding: 10px 14px;
  border-radius: 14px;
  font-size: 14px;
  line-height: 1.5;
}
.message.assistant {
  align-self: flex-start;
}
.message.assistant p {
  background: #ffffff;
  border: 1px solid var(--border);
  color: var(--body-text);
  border-bottom-left-radius: 4px;
}
.message.user {
  align-self: flex-end;
}
.message.user p {
  background: var(--brand);
  color: #ffffff;
  border-bottom-right-radius: 4px;
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

.chat-panel-input-row {
  flex-shrink: 0;
  padding: 16px 24px;
  background: #ffffff;
  border-top: 1px solid var(--border);
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
