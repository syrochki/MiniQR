<template>
  <main class="min-h-dvh bg-zinc-50 text-zinc-900 selection:bg-zinc-900 selection:text-white">
    <div class="mx-auto flex min-h-dvh w-full max-w-3xl flex-col px-5 py-10">
      <header class="no-print mb-10">
        <h1 class="text-balance text-2xl font-semibold tracking-tight">
          MiniQR
        </h1>
        <p class="mt-2 text-sm text-zinc-600">
          МиниКВАР - генератор QR-кодов, просто так.
        </p>
      </header>

      <section class="flex flex-1 flex-col items-center justify-center">
        <div class="w-full max-w-xl">
          <div class="no-print mb-4 flex items-center justify-center gap-2">
            <button
              type="button"
              class="rounded-xl border border-zinc-200 bg-white px-3 py-2 text-sm font-medium shadow-sm transition hover:bg-zinc-50 active:translate-y-px disabled:opacity-50"
              :disabled="!hasText"
              @click="share()"
            >
              Поделиться
            </button>
            <button
              type="button"
              class="rounded-xl border border-zinc-200 bg-white px-3 py-2 text-sm font-medium shadow-sm transition hover:bg-zinc-50 active:translate-y-px disabled:opacity-50"
              :disabled="!hasText"
              @click="print()"
            >
              Печать
            </button>
            <button
              type="button"
              class="rounded-xl border border-zinc-200 bg-white px-3 py-2 text-sm font-medium shadow-sm transition hover:bg-zinc-50 active:translate-y-px disabled:opacity-50"
              :disabled="!hasText"
              @click="download()"
            >
              Скачать
            </button>
          </div>

          <div class="rounded-3xl border border-zinc-200 bg-white p-5 shadow-sm">
            <div class="flex flex-col items-center">
              <div
                class="grid place-items-center rounded-2xl bg-zinc-50 p-4"
                :class="hasText ? 'w-full' : 'w-full opacity-60'"
              >
                <img
                  v-if="hasText"
                  :src="qrUrl"
                  :alt="`QR: ${normalizedText}`"
                  class="aspect-square w-full max-w-130 rounded-xl bg-white p-3"
                  decoding="async"
                  loading="eager"
                  referrerpolicy="no-referrer"
                >
                <div v-else class="aspect-square w-full max-w-130 rounded-xl border border-dashed border-zinc-300 bg-white p-6">
                  <div class="flex h-full flex-col items-center justify-center text-center">
                    <div class="text-sm font-medium text-zinc-700">
                      Введите текст или ссылку
                    </div>
                    <div class="mt-1 text-xs text-zinc-500">
                      QR появится здесь
                    </div>
                  </div>
                </div>
              </div>

              <button
                v-if="hasText"
                type="button"
                class="no-print mt-4 max-w-full truncate rounded-xl bg-zinc-900 px-4 py-2 text-sm font-medium text-white shadow-sm transition hover:bg-black active:translate-y-px"
                :title="normalizedText"
                @click="copyText(normalizedText)"
              >
                {{ normalizedText }}
              </button>
              <div v-if="toast" class="no-print mt-3 text-xs text-zinc-600">
                {{ toast }}
              </div>
            </div>
          </div>

          <form class="no-print mt-5" @submit.prevent="applyQuery()">
            <label class="block text-sm font-medium text-zinc-700" for="qr-input">
              Текст для QR
            </label>
            <div class="mt-2 flex gap-2">
              <input
                id="qr-input"
                v-model="draft"
                name="s"
                maxlength="1024"
                autocomplete="off"
                inputmode="url"
                placeholder="https://example.com"
                class="min-w-0 flex-1 rounded-xl border border-zinc-200 bg-white px-4 py-3 text-sm shadow-sm outline-none ring-zinc-900/10 focus:ring-4"
              >
              <button
                type="submit"
                class="rounded-xl bg-zinc-900 px-4 py-3 text-sm font-medium text-white shadow-sm transition hover:bg-black active:translate-y-px disabled:opacity-50"
                :disabled="!draft.trim()"
              >
                Сохранить УРЛ
              </button>
            </div>
          </form>
        </div>
      </section>

      <footer class="no-print mt-10 text-center text-xs text-zinc-500">
        made by <a href="https://github.com/syrochki" target="_blank" rel="noopener noreferrer">@syrochki</a>
      </footer>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

const route = useRoute()
const router = useRouter()

const coerceQueryString = (value: unknown): string => {
  if (typeof value === 'string') return value
  if (Array.isArray(value)) return value.join(' ')
  return ''
}

const draft = ref<string>(coerceQueryString(route.query.s))
const toast = ref<string>('')
let toastTimer: ReturnType<typeof setTimeout> | undefined

const normalizedText = computed(() => draft.value.trim())
const hasText = computed(() => normalizedText.value.length > 0)

const qrUrl = computed(() => {
  if (!hasText.value) return ''
  const encoded = encodeURIComponent(normalizedText.value)
  return `https://quickchart.io/qr?size=700&margin=1&text=${encoded}`
})

watch(
  () => route.query.s,
  (value) => {
    const next = coerceQueryString(value)
    if (next !== draft.value) draft.value = next
  }
)

const showToast = (message: string) => {
  toast.value = message
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => {
    toast.value = ''
  }, 1800)
}

const applyQuery = async () => {
  const text = normalizedText.value
  await router.replace({
    query: {
      ...route.query,
      s: text || undefined
    }
  })
}

const copyText = async (text: string) => {
  if (!import.meta.client) return
  try {
    await navigator.clipboard.writeText(text)
    showToast('Скопировано')
  } catch {
    showToast('Не удалось скопировать')
  }
}

const share = async () => {
  if (!import.meta.client || !hasText.value) return

  const url = new URL(window.location.href)
  url.searchParams.set('s', normalizedText.value)

  try {
    if (navigator.share) {
      await navigator.share({ title: 'MiniQR', text: normalizedText.value, url: url.toString() })
      return
    }
  } catch {
    // ignore
  }

  await copyText(url.toString())
}

const print = () => {
  if (!import.meta.client || !hasText.value) return
  window.print()
}

const download = async () => {
  if (!import.meta.client || !hasText.value) return

  const filename = 'miniqr.png'
  try {
    const response = await fetch(qrUrl.value, { mode: 'cors' })
    if (!response.ok) throw new Error('bad response')
    const blob = await response.blob()

    const objectUrl = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = objectUrl
    a.download = filename
    document.body.appendChild(a)
    a.click()
    a.remove()
    URL.revokeObjectURL(objectUrl)
  } catch {
    window.open(qrUrl.value, '_blank', 'noopener,noreferrer')
    showToast('Открыл в новой вкладке для сохранения')
  }
}
</script>

