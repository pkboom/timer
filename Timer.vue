<template>
  <div class="flex min-h-screen flex-col items-center justify-center space-y-4">
    <div class="flex items-center space-x-2 text-6xl">
      <span>{{ displayMinutes }}</span>
      <span class="mb-2">:</span>
      <span>{{ displaySeconds }}</span>
    </div>
    <div class="space-x-4 text-3xl">
      <button
        type="button"
        class="border rounded-lg border-gray-400 px-8 py-4 text-3xl hover:border-indigo-500 hover:text-indigo-500"
        @click="start(25)"
      >
        25
      </button>
      <button
        type="button"
        class="border rounded-lg border-gray-400 px-8 py-4 text-3xl hover:border-indigo-500 hover:text-indigo-500"
        @click="start(10)"
      >
        10
      </button>
      <button
        type="button"
        class="border rounded-lg border-gray-400 px-8 py-4 text-3xl hover:border-indigo-500 hover:text-indigo-500"
        @click="start(5)"
      >
        5
      </button>
      <button
        type="button"
        class="border rounded-lg border-gray-400 px-8 py-4 text-3xl hover:border-indigo-500 hover:text-indigo-500"
        @click="stop"
      >
        Stop
      </button>
    </div>
    <audio id="new-order" src="/sound/new-order.ogg" preload="auto" />
  </div>
</template>

<script>
export default {
  metaInfo: { title: 'Timer' },
  data() {
    return {
      timer: null,
      minutes: 0,
      seconds: 0,
    }
  },
  computed: {
    displayMinutes() {
      if (this.minutes < 0) return 0

      return this.minutes
    },
    displaySeconds() {
      if (this.seconds < 0) return '00'

      return this.seconds.toString().padStart(2, '0')
    },
  },
  mounted() {
    document.addEventListener('keydown', this.useHotKeys)
  },
  beforeDestroy() {
    document.removeEventListener('keydown', this.useHotKeys)
  },
  methods: {
    start(duration = 0) {
      if (this.timer) return

      this.changeFaviconToTimer()

      let seconds = duration ? 60 * duration : 60 * this.minutes
      let milliseconds = seconds * 1000

      let now = new Date().getTime()
      let end = now + milliseconds

      this.minutes = Math.floor((end - now) / 60 / 1000)
      this.seconds = Math.floor(((end - now) % (60 * 1000)) / 1000)

      this.timer = setInterval(() => {
        now = new Date().getTime()

        if (end - now <= 0) {
          this.stop()

          document
            .getElementById('new-order')
            .play()
            .catch(() => alert("Can't play sound."))

          return
        }

        this.minutes = Math.floor((end - now) / 60 / 1000)
        this.seconds = Math.floor(((end - now) % (60 * 1000)) / 1000)
      }, 1000)
    },
    stop() {
      this.revertFavicon()

      clearInterval(this.timer)

      this.timer = null
      this.minutes = 0
      this.seconds = 0
    },
    useHotKeys(e) {
      if (e.key === 's') {
        e.preventDefault()

        this.timer ? this.stop() : this.start()
      }

      if (e.key === 'i') {
        e.preventDefault()

        this.minutes += 10
      }

      if (e.key === 'd') {
        e.preventDefault()

        this.minutes -= 10
      }
    },
    revertFavicon() {
      this.favicon('/favicon-32x32.png')
    },
    changeFaviconToTimer() {
      this.favicon('/favicon-timer.png')
    },
    favicon(favicon = null) {
      let links = [...document.querySelectorAll('link[rel="icon"]')]
      console.log(links)

      links.forEach(link => (link.href = favicon))
    },
  },
}
</script>
