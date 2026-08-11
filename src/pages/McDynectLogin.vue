<script setup>
/**
 * McDynectLogin.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=756-15035
 *
 * Built with Vue 3 <script setup> (Composition API). Plain HTML/CSS — no flowbite-vue
 * dependency for this page.
 *
 * NOTE ON ASSETS: the icon/logo/background image constants below point at
 * Figma's temporary asset host. Those URLs expire ~7 days after export. Before
 * shipping to production, export the same layers from Figma (Dev Mode > this
 * frame > Assets) and swap each constant for a local `import` from your own
 * `assets/` folder — the geometry/classes below already match the source
 * exactly, only the `src` needs to move local.
 *
 * RIGHT-PANEL ANIMATION: only the heading/subtext still animate (fade-up, transcribed
 * from Figma's motion data on node 756:15035). The orbit rings + 8 icons were
 * originally built as individually-animated layered DOM (per-icon pop-in), but are
 * now baked into one flattened `orbit-icons.png` — captured from that implementation
 * at rest, transparent background — for a simpler static illustration.
 */
import { ref } from 'vue'
import imgOrbitIcons from '../assets/orbit-icons.png'

// --- Left panel assets ---
const imgLogoMark = 'https://www.figma.com/api/mcp/asset/7cc852c0-3d0b-489d-a1fd-e9373bf746cf.svg'
const imgLogoWordmark = 'https://www.figma.com/api/mcp/asset/5f05cc29-d15a-448e-9f3d-28a8c21fc45d.svg'

// --- Right panel assets ---
const imgPatternBackground = 'https://www.figma.com/api/mcp/asset/282ac419-1705-41de-b15b-af1755e4cac4.png'

// Radial glow behind the orbit graphic — reproduced from the exact gradient Figma exports
// (an off-center ellipse, not a perfectly centered circle) rather than approximated.
const radialGlowSvg = `<svg viewBox="0 0 808 808" xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="none"><rect x="0" y="0" height="100%" width="100%" fill="url(#grad)" opacity="1"/><defs><radialGradient id="grad" gradientUnits="userSpaceOnUse" cx="0" cy="0" r="10" gradientTransform="matrix(0.0000036908 35.84 -35.84 0.0000026146 404 368.73)"><stop stop-color="rgba(145,27,45,1)" offset="0.4"/><stop stop-color="rgba(145,27,45,0)" offset="1"/></radialGradient></defs></svg>`
const imgRadialGlow = `data:image/svg+xml;utf8,${encodeURIComponent(radialGlowSvg)}`

const emit = defineEmits(['submit'])

const email = ref('')
const password = ref('')
const rememberMe = ref(false)
const isSubmitting = ref(false)

async function handleSubmit() {
  isSubmitting.value = true
  try {
    emit('submit', { email: email.value, password: password.value, rememberMe: rememberMe.value })
  } finally {
    isSubmitting.value = false
  }
}
</script>

<template>
  <div class="page">
    <div class="left">
      <div class="logo-row">
        <img class="logo-mark" :src="imgLogoMark" alt="MC" />
        <img class="logo-text" :src="imgLogoWordmark" alt="dynect" />
      </div>

      <div class="form-wrap">
        <h1>Welcome!</h1>
        <form @submit.prevent="handleSubmit">
          <div class="fields">
            <div class="field">
              <label for="email">Email<span class="req">*</span></label>
              <input id="email" v-model="email" type="email" placeholder="Enter your email address" required />
            </div>
            <div class="field">
              <label for="password">Password<span class="req">*</span></label>
              <input id="password" v-model="password" type="password" placeholder="Enter your password" required />
            </div>
            <div class="row-between">
              <label class="checkbox-label">
                <input v-model="rememberMe" type="checkbox" />
                <span>Remember me</span>
              </label>
              <a class="forgot-link" href="#">Forgot password?</a>
            </div>
          </div>

          <div class="button-group">
            <button class="btn-primary" type="submit" :disabled="isSubmitting">
              {{ isSubmitting ? 'Logging in…' : 'Log In' }}
            </button>
            <p class="signup"><span>Don&rsquo;t have an account?</span> <a href="#">Sign up here</a></p>
          </div>
        </form>
      </div>

      <div class="footer">© 2026 MCdynect.</div>
    </div>

    <div class="right">
      <div class="pattern-bg-wrap">
        <div class="pattern-bg-rotate">
          <img :src="imgPatternBackground" alt="" />
        </div>
      </div>
      <div class="radial-glow" :style="{ backgroundImage: 'url(' + imgRadialGlow + ')' }"></div>

      <div class="right-content">
        <div class="orbit-panel">
          <div class="orbit-frame" aria-hidden="true">
            <img class="orbit-illustration" :src="imgOrbitIcons" alt="" />
          </div>

          <div class="right-copy-block">
            <p class="right-heading fade-block" style="animation-delay: 900ms">Over 5+ technologies</p>
            <div class="right-subtext fade-block" style="animation-delay: 1100ms">
              <p>Bizcentral brings all MCDynect applications</p>
              <p>together&mdash;giving you secure, unified access to manage and grow your business.</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.page {
  --brand: #911b2d;
  --brand-strong: #54101a;
  --danger: #c70036;
  --border: #e5e7eb;
  --bg-secondary: #f9fafb;
  --text-body: #4a5565;
  --text-body-subtle: #6a7282;
  --white: #ffffff;

  display: flex;
  min-height: 100vh;
  width: 100%;
  color: var(--text-body);
}

.page * {
  box-sizing: border-box;
}

/* ---------- Left panel ---------- */
.left {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  width: 628px;
  flex-shrink: 0;
  padding: 40px;
  min-height: 100vh;
}

.logo-row {
  display: flex;
  align-items: center;
  gap: 2px;
  height: 24px;
  width: 100%;
}
.logo-row img {
  display: block;
  height: 100%;
  width: auto;
}
.logo-mark {
  height: 21px;
}
.logo-text {
  height: 22px;
}

.form-wrap {
  width: 394px;
  max-width: 100%;
  display: flex;
  flex-direction: column;
  gap: 36px;
}

.form-wrap h1 {
  margin: 0;
  font-size: 36px;
  font-weight: 600;
  color: var(--brand-strong);
  line-height: normal;
}

form {
  display: flex;
  flex-direction: column;
  gap: 36px;
  width: 100%;
}

.fields {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
}

.field label {
  display: flex;
  align-items: center;
  gap: 2px;
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--brand);
}
.field label .req {
  color: var(--danger);
}

.field input {
  width: 100%;
  font-family: inherit;
  font-size: 14px;
  font-weight: 400;
  color: var(--text-body);
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  padding: 10px 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  outline: none;
}
.field input::placeholder {
  color: var(--text-body-subtle);
}
.field input:focus {
  border-color: var(--brand);
}

#email {
  border-radius: 12px;
}
#password {
  border-radius: 8px;
}

.row-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  color: var(--text-body);
}
.checkbox-label input[type='checkbox'] {
  width: 16px;
  height: 16px;
  margin: 0;
  border-radius: 4px;
  border: 1px solid var(--border);
  background: var(--bg-secondary);
  accent-color: var(--brand);
}

.forgot-link {
  font-size: 14px;
  font-weight: 500;
  color: var(--brand);
  text-decoration: none;
}
.forgot-link:hover {
  text-decoration: underline;
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
}

.btn-primary {
  width: 100%;
  background: var(--brand);
  color: var(--white);
  border: none;
  border-radius: 12px;
  padding: 14px 24px;
  font-family: inherit;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  transition: background 0.15s ease;
}
.btn-primary:hover {
  background: #7a1625;
}
.btn-primary:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.signup {
  margin: 0;
  font-size: 14px;
}
.signup span {
  font-weight: 500;
  color: var(--text-body);
}
.signup a {
  font-weight: 500;
  color: var(--brand);
  text-decoration: none;
}
.signup a:hover {
  text-decoration: underline;
}

.footer {
  width: 100%;
  font-size: 14px;
  font-weight: 400;
  color: var(--text-body);
}

/* ---------- Right panel ---------- */
.right {
  position: relative;
  flex: 1 1 auto;
  min-width: 0;
  background: var(--brand);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.pattern-bg-wrap {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 1725px;
  height: 1804px;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}
.pattern-bg-rotate {
  width: 2000px;
  height: 1402px;
  transform: rotate(-27.01deg);
}
.pattern-bg-rotate img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.3;
}

.radial-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 808px;
  height: 808px;
  transform: translate(-50%, -50%);
  background-repeat: no-repeat;
  pointer-events: none;
}

.right-content {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 32px;
  width: 100%;
  padding: 0 80px;
}

.orbit-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  gap: 16px;
  width: 804px;
  max-width: 100%;
}

.orbit-frame {
  position: relative;
  width: 540px;
  height: 340px;
  max-width: 100%;
  overflow: hidden;
}

.orbit-illustration {
  position: absolute;
  top: 0;
  left: 0;
  width: 540px;
  height: 340px;
  display: block;
  pointer-events: none;
}

/* ---------- Heading & paragraph ---------- */
.right-copy-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 0 56px 24px;
  width: 100%;
  text-align: center;
  color: var(--white);
}

.right-heading {
  margin: 0;
  font-size: 36px;
  font-weight: 700;
  letter-spacing: -0.72px;
  line-height: 44px;
}

.right-subtext {
  font-size: 16px;
  font-weight: 400;
  line-height: 28px;
  max-width: 389px;
}
.right-subtext p {
  margin: 0;
}

.fade-block {
  opacity: 0;
  transform: translateY(16px);
  animation: fade-up-opacity 2s linear infinite, fade-up-move 2s linear infinite;
}

@keyframes fade-up-opacity {
  0% {
    opacity: 0;
    animation-timing-function: ease;
  }
  25% {
    opacity: 1;
  }
  100% {
    opacity: 1;
  }
}
@keyframes fade-up-move {
  0% {
    transform: translateY(16px);
    animation-timing-function: ease;
  }
  25% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(0);
  }
}

@media (prefers-reduced-motion: reduce) {
  .fade-block {
    animation: none !important;
    opacity: 1 !important;
    transform: translateY(0) !important;
  }
}

/* ---------- Responsive ---------- */
@media (max-width: 1100px) {
  .right {
    display: none;
  }
  .left {
    width: 100%;
  }
}
@media (max-width: 480px) {
  .left {
    padding: 24px;
  }
  .form-wrap {
    width: 100%;
  }
}
</style>
