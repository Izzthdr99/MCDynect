<script setup>
/**
 * McDynectLogin.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=756-15010
 *
 * Built with Vue 3 <script setup> (Composition API) + flowbite-vue components.
 * Requires: `flowbite` + `flowbite-vue` installed, and Tailwind configured to scan
 * `./node_modules/flowbite-vue/**\/*.{js,vue}` in its `content` array.
 *
 * NOTE ON ASSETS: the icon/logo/background image constants below point at
 * Figma's temporary asset host. Those URLs expire ~7 days after export. Before
 * shipping to production, export the same layers from Figma (Dev Mode > this
 * frame > Assets) and swap each constant for a local `import` from your own
 * `assets/` folder — the geometry/classes around each <img> already match the
 * source exactly, only the `src` needs to move local.
 */
import { ref } from 'vue'
import { FwbInput, FwbCheckbox, FwbButton, FwbA } from 'flowbite-vue'

// --- Figma-exported assets (temporary URLs, see note above) ---
const imgPatternBackground = 'https://www.figma.com/api/mcp/asset/45bb196c-41ce-4a10-a06c-824c6d237d12.png'
const imgLogoMark = 'https://www.figma.com/api/mcp/asset/c42434bd-f431-48f5-b900-51274c6dd3c0.svg'
const imgLogoWordmark = 'https://www.figma.com/api/mcp/asset/135fd143-1324-4dcc-9a5a-1e2f116518d0.svg'
const imgOrbitRings = 'https://www.figma.com/api/mcp/asset/6d99546a-9b51-4765-8b4b-448c524477cd.svg'
const imgRetailLight = 'https://www.figma.com/api/mcp/asset/b8b70b9b-6410-4363-bc70-1f59ddeaa289.svg'
const imgRetailBase = 'https://www.figma.com/api/mcp/asset/e813069c-6301-450a-b8b7-f311a513c10b.svg'
const imgRetailDark = 'https://www.figma.com/api/mcp/asset/e56e5f80-8c88-4b2f-a80a-e6eeb833d241.svg'
const imgErpBase = 'https://www.figma.com/api/mcp/asset/78b858da-0b17-40cc-94da-3925a9fa872c.svg'
const imgErpDark = 'https://www.figma.com/api/mcp/asset/1baae35c-f69b-4486-b8f5-0b990e6a8c71.svg'
const imgErpLight = 'https://www.figma.com/api/mcp/asset/8bedaeb7-d373-4200-aeb9-5eb2ef8a486e.svg'
const imgProjectBase = 'https://www.figma.com/api/mcp/asset/b0d66229-9fd5-498b-a25a-23aa3e8060b1.svg'
const imgProjectDark = 'https://www.figma.com/api/mcp/asset/6fa8b85e-119c-4928-8cdc-96bdd4989415.svg'
const imgCrmDark = 'https://www.figma.com/api/mcp/asset/b34eb6bc-ab5f-41be-933e-28ab35f2337e.svg'
const imgPosLight = 'https://www.figma.com/api/mcp/asset/3fd6a39e-7903-4ba8-9170-db476779dbd6.svg'
const imgPosDark = 'https://www.figma.com/api/mcp/asset/9504694f-784a-43ec-b87c-a8d3157e8e26.svg'
const imgPosBase = 'https://www.figma.com/api/mcp/asset/96e61c31-9f3f-4d00-b5bc-af5545c6d0d2.svg'
const imgAssetLight = 'https://www.figma.com/api/mcp/asset/29b89976-d2f5-428c-af50-4f8b0db2f5c8.svg'
const imgAssetBase = 'https://www.figma.com/api/mcp/asset/9bcd4e59-b954-4ead-b3f8-2e1e621de9f8.svg'
const imgAssetDark = 'https://www.figma.com/api/mcp/asset/fce8003e-7a5c-4fa4-a414-0d272af4f4d4.svg'
const imgInvestmentBase = 'https://www.figma.com/api/mcp/asset/2500df59-69db-4d75-85d1-b75245a931a3.svg'
const imgInvestmentLight = 'https://www.figma.com/api/mcp/asset/548ca0c3-8ce1-4e8d-838c-642df75073ae.svg'
const imgInvestmentDark = 'https://www.figma.com/api/mcp/asset/0c2d1d19-9dba-498b-a85f-396c00761759.svg'
const imgHrmsActive = 'https://www.figma.com/api/mcp/asset/f13f70e7-268a-4a56-af62-d6816b143c40.svg'

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
  <div class="flex flex-col items-center bg-white size-full min-h-screen">
    <div class="flex flex-[1_0_0] min-h-px w-full items-stretch">
      <!-- Left content -->
      <div class="flex w-full max-w-[628px] flex-col items-center justify-between p-10">
        <!-- Logo -->
        <div class="flex w-full items-center gap-3">
          <div class="flex h-6 items-center gap-0.5">
            <img :src="imgLogoMark" alt="MC" class="h-[21px] w-[42px]" />
            <img :src="imgLogoWordmark" alt="Dynect" class="h-[22px] w-[103px]" />
          </div>
        </div>

        <!-- Inputs -->
        <div class="flex w-full max-w-[394px] flex-col items-start gap-9 rounded-lg">
          <p class="w-full max-w-[310px] font-['Geist:SemiBold',_sans-serif] text-4xl font-semibold leading-normal text-[#54101a]">
            Welcome!
          </p>

          <form class="flex w-full flex-col items-center justify-center gap-9 rounded-lg" @submit.prevent="handleSubmit">
            <div class="flex w-full flex-col items-start gap-6">
              <!-- Email -->
              <div class="flex w-full flex-col items-start gap-2.5">
                <label class="flex items-center gap-0.5 text-sm font-medium font-['Geist:Medium',_sans-serif] leading-5">
                  <span class="text-[#911b2d]">Email</span>
                  <span class="text-[#c70036]">*</span>
                </label>
                <fwb-input
                  v-model="email"
                  type="email"
                  placeholder="Enter your email address"
                  wrapper-class="w-full"
                  input-class="w-full rounded-xl border border-[#e5e7eb] bg-[#f9fafb] px-3 py-2.5 text-sm text-[#6a7282] placeholder:text-[#6a7282] shadow-[0px_1px_0.25px_0px_rgba(29,41,61,0.02)] focus:border-[#911b2d] focus:ring-[#911b2d]"
                />
              </div>

              <!-- Password -->
              <div class="flex w-full flex-col items-start gap-2.5">
                <label class="flex items-center gap-0.5 text-sm font-medium font-['Geist:Medium',_sans-serif] leading-5">
                  <span class="text-[#911b2d]">Password</span>
                  <span class="text-[#c70036]">*</span>
                </label>
                <fwb-input
                  v-model="password"
                  type="password"
                  placeholder="Enter your password"
                  wrapper-class="w-full"
                  input-class="w-full rounded-lg border border-[#e5e7eb] bg-[#f9fafb] px-3 py-2.5 text-sm text-[#6a7282] placeholder:text-[#6a7282] shadow-[0px_1px_0.25px_0px_rgba(29,41,61,0.02)] focus:border-[#911b2d] focus:ring-[#911b2d]"
                />
              </div>

              <!-- Remember me / Forgot password -->
              <div class="flex w-full items-center justify-between">
                <fwb-checkbox
                  v-model="rememberMe"
                  label="Remember me"
                  input-class="size-4 rounded-[4px] border border-[#e5e7eb] bg-[#f9fafb] text-[#911b2d] focus:ring-[#911b2d]"
                  label-class="text-sm font-medium font-['Geist:Medium',_sans-serif] leading-4 text-[#4a5565]"
                />
                <fwb-a href="#" class="text-right text-sm font-medium font-['Geist:Medium',_sans-serif] leading-[14px] text-[#911b2d] no-underline hover:underline">
                  Forgot password?
                </fwb-a>
              </div>
            </div>

            <!-- Submit + helper text -->
            <div class="flex w-full flex-col items-start gap-4">
              <fwb-button
                type="submit"
                :loading="isSubmitting"
                class="w-full"
                square
                pill
              >
                <span class="w-full text-center text-base font-medium leading-6 font-['Geist:Medium',_sans-serif]">Log In</span>
              </fwb-button>

              <p class="text-sm font-['Geist:Medium',_sans-serif] font-medium">
                <span class="leading-5 text-[#4a5565]">Don&rsquo;t have an account?</span>
                <fwb-a href="#" class="leading-5 text-[#911b2d] no-underline hover:underline"> Sign up here</fwb-a>
              </p>
            </div>
          </form>
        </div>

        <!-- Footer -->
        <div class="flex w-full items-center">
          <p class="text-sm font-['Geist:Regular',_sans-serif] font-normal leading-5 text-[#4a5565]">© 2025 Dynect.</p>
        </div>
      </div>

      <!-- Right content -->
      <div class="relative hidden h-full w-[812px] shrink-0 flex-col items-center justify-center gap-8 overflow-hidden bg-[#911b2d] lg:flex">
        <!-- Decorative pattern + glow (absolutely centered, oversized, clipped by parent overflow-hidden) -->
        <div class="pointer-events-none absolute left-1/2 top-1/2 flex h-[1804px] w-[1725px] -translate-x-1/2 -translate-y-1/2 items-center justify-center">
          <div class="h-[1402px] w-[1221px] -rotate-[27.01deg]">
            <img :src="imgPatternBackground" alt="" class="size-full object-cover opacity-30" />
          </div>
        </div>
        <div
          class="pointer-events-none absolute left-1/2 top-1/2 size-[808px] -translate-x-1/2 -translate-y-1/2"
          :style="{
            backgroundImage:
              'radial-gradient(circle at center, rgba(145,27,45,1) 0%, rgba(145,27,45,1) 40%, rgba(145,27,45,0) 100%)',
          }"
        />

        <!-- Content -->
        <div class="relative flex w-full flex-col items-center justify-center gap-8 px-20">
          <div class="relative flex h-[340px] w-[540px] flex-col items-center justify-end gap-4 overflow-hidden rounded-3xl">
            <!-- Orbit rings -->
            <img :src="imgOrbitRings" alt="" class="absolute left-[2px] top-[41px] w-[536px]" />

            <!-- Retail -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[67.5px] -translate-x-[calc(50%+215.5px)] -translate-y-[calc(50%+1.38px)] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 500ms">
              <div class="relative size-[45px] overflow-hidden">
                <img :src="imgRetailLight" alt="" class="absolute inset-[34.38%_13.54%_13.54%_13.54%] block size-full max-w-none" />
                <img :src="imgRetailBase" alt="" class="absolute inset-[13.54%_13.54%_65.63%_13.54%] block size-full max-w-none" />
                <img :src="imgRetailDark" alt="" class="absolute inset-[34.38%_13.54%_40.63%_13.54%] block size-full max-w-none" />
              </div>
            </div>

            <!-- ERP -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[67.5px] -translate-x-[calc(50%+132.66px)] -translate-y-[calc(50%-107.13px)] scale-y-[-1] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 700ms">
              <div class="relative size-[45px] scale-y-[-1] overflow-hidden">
                <img :src="imgErpBase" alt="" class="absolute inset-[8.33%_10.25%_8.33%_8.33%] block size-full max-w-none" />
                <img :src="imgErpDark" alt="" class="absolute bottom-1/2 left-1/2 right-[8.33%] top-[8.33%] block size-full max-w-none" />
                <img :src="imgErpLight" alt="" class="absolute bottom-1/2 left-1/2 right-[27.09%] top-[27.09%] block size-full max-w-none" />
              </div>
            </div>

            <!-- Project -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[45px] -translate-x-[calc(50%+87.45px)] -translate-y-[calc(50%+17.67px)] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 400ms">
              <div class="relative size-[28px] overflow-hidden">
                <img :src="imgProjectBase" alt="" class="absolute inset-[13.35%_32.29%_13.73%_13.54%] block size-full max-w-none" />
                <img :src="imgProjectDark" alt="" class="absolute inset-[13.35%_13.54%_40.82%_67.71%] block size-full max-w-none" />
                <div class="absolute inset-[32.1%_13.54%_59.57%_67.71%] bg-[#eeba5d]" />
              </div>
            </div>

            <!-- CRM -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[67.5px] -translate-x-[calc(50%-199.6px)] -translate-y-[calc(50%+22.29px)] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 600ms">
              <div class="relative size-[45px] overflow-hidden">
                <div class="absolute inset-[13.35%_13.54%_55.4%_44.79%] bg-[#c10007]" />
                <img :src="imgCrmDark" alt="" class="absolute inset-[13.35%_13.54%_13.73%_13.54%] block size-full max-w-none" />
                <div class="absolute inset-[44.6%_55.21%_13.73%_13.54%] bg-[#eeba5d]" />
              </div>
            </div>

            <!-- POS -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[56.25px] -translate-x-[calc(50%-79.79px)] -translate-y-[calc(50%+20.58px)] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 300ms">
              <div class="relative size-[39px] overflow-hidden">
                <img :src="imgPosLight" alt="" class="absolute inset-[29.19%_13.93%_8.33%_49.99%] block size-full max-w-none" />
                <img :src="imgPosDark" alt="" class="absolute inset-[8.33%_13.93%_59.11%_13.93%] block size-full max-w-none" />
                <img :src="imgPosBase" alt="" class="absolute inset-[29.19%_65.86%_17.5%_13.93%] block size-full max-w-none" />
              </div>
            </div>

            <!-- Asset -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[67.5px] -translate-x-[calc(50%+1.88px)] -translate-y-[calc(50%+114.27px)] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 200ms">
              <div class="relative size-[45px] overflow-hidden">
                <img :src="imgAssetLight" alt="" class="absolute bottom-[32.86%] left-1/2 right-[32.86%] top-[32.86%] block size-full max-w-none" />
                <img :src="imgAssetBase" alt="" class="absolute inset-[13.54%] block size-full max-w-none" />
                <img :src="imgAssetDark" alt="" class="absolute inset-[13.55%_13.54%_13.54%_67.13%] block size-full max-w-none" />
              </div>
            </div>

            <!-- Investment -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[67.5px] -translate-x-[calc(50%+1.88px)] -translate-y-[calc(50%-55.25px)] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]" style="animation-delay: 100ms">
              <div class="relative size-[45px] overflow-hidden">
                <img :src="imgInvestmentBase" alt="" class="absolute inset-[13.35%_13.54%_13.73%_13.54%] block size-full max-w-none" />
                <img :src="imgInvestmentLight" alt="" class="absolute inset-[55.37%_32.3%_13.73%_32.3%] block size-full max-w-none" />
                <img :src="imgInvestmentDark" alt="" class="absolute inset-[34.18%_13.54%_13.73%_13.54%] block size-full max-w-none" />
              </div>
            </div>

            <!-- HRMS -->
            <div class="orbit-icon absolute left-1/2 top-1/2 flex size-[67.5px] -translate-x-[calc(50%-162.65px)] -translate-y-[calc(50%-100.25px)] items-center justify-center rounded-full bg-white/40" style="animation-delay: 800ms">
              <div class="flex size-[67.5px] items-center justify-center rounded-full border-[1.1px] border-[#f9fafb] bg-gradient-to-b from-white to-white/80 shadow-[0px_2.8px_2.8px_0px_rgba(29,41,61,0.1),0px_5.6px_4.2px_0px_rgba(29,41,61,0.1)]">
                <img :src="imgHrmsActive" alt="" class="h-[45px] w-[46px]" />
              </div>
            </div>
          </div>

          <!-- Heading & paragraph -->
          <div class="flex w-full flex-col items-center gap-3 px-14 pb-6 text-center text-white">
            <p class="fade-up min-w-full font-['Geist:Bold',_sans-serif] text-[36px] font-bold leading-[44px] tracking-[-0.72px]" style="animation-delay: 900ms">
              Over 5+ technologies
            </p>
            <div class="fade-up w-[389px] font-['Geist:Regular',_sans-serif] text-base font-normal leading-7" style="animation-delay: 1100ms">
              <p class="mb-0">Bizcentral brings all MCDynect applications</p>
              <p>together—giving you secure, unified access to manage and grow your business.</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.orbit-icon {
  animation-name: orbit-pop;
  animation-duration: 2s;
  animation-iteration-count: infinite;
  animation-timing-function: cubic-bezier(0.34, 1.4, 0.64, 1);
  animation-fill-mode: backwards;
}

@keyframes orbit-pop {
  0%,
  11.25% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0);
  }
  22.5%,
  100% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}

.fade-up {
  animation-name: fade-up;
  animation-duration: 2s;
  animation-iteration-count: infinite;
  animation-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1);
  animation-fill-mode: backwards;
}

@keyframes fade-up {
  0% {
    opacity: 0;
    transform: translateY(16px);
  }
  25%,
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
