<!-- FROM: https://github.com/Redlnn/Fake-QQ-Chat-Window -->

<template>
  <div class="chat-item" :class="[onright ? 'right-chat' : 'left-chat']">
    <div
      v-if="avatar"
      :style="{ 'background-image': `url(${avatar})` }"
      class="chat-avatar"
    ></div>
    <div v-else class="chat-avatar chat-avatar-text">{{ name[0] }}</div>
    <div class="chat-content">
      <div class="chat-name">{{ name }}</div>
      <div class="bubble" :onclick="playVoice" style="cursor: pointer">
        <div class="bubble-arrow"></div>
        <div class="chat-audio">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 52 52">
            <circle cx="26" cy="26" r="26" />
            <path
              d="M35.5,25.324,20.512,14.575a1,1,0,0,0-1.589.815v21.5a1,1,0,0,0,1.589.816L35.5,26.955a1,1,0,0,0,.237-1.394A.988.988,0,0,0,35.5,25.324Z"
            />
          </svg>
          <audio
            ref="audio"
            :src="audio"
            @ended="reset"
            @loadedmetadata="onLoadedmetadata"
          ></audio>
          <span class="voice-bar">
            <span
              v-for="line in getLineCount(duration)"
              :key="line.id"
              ref="voice-line"
              class="line"
            ></span>
          </span>
          {{ formatedDuration }}
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'ChatVoice',
  props: {
    name: { type: String, required: true },
    avatar: String,
    audio: { type: String, required: true },
    onright: Boolean,
  },
  data() {
    return {
      playFlag: false,
      duration: 10,
      formatedDuration: '',
    }
  },
  methods: {
    getLineCount: function (num: number) {
      const lineArray = []
      num = num / 1.5
      if (num < 5)
        return [{ id: 1 }, { id: 2 }, { id: 3 }, { id: 4 }, { id: 5 }]
      for (let i = 0; i <= num; i++) {
        if (i >= 25) {
          break
        }
        lineArray.push({ id: i })
      }
      return lineArray
    },
    reset: function () {
      this.playFlag = false
    },
    onLoadedmetadata: function () {
      const audioElem = this.$refs.audio as HTMLAudioElement
      this.duration = Math.round(audioElem.duration)
      const m = Math.floor(audioElem.duration / 60)
      const s = Math.round(audioElem.duration % 60)
      this.formatedDuration = m > 0 ? `${m}'${s}"` : `${s}"`
    },
    sleep: (ms: number) => {
      return new Promise((resolve) => setTimeout(resolve, ms))
    },
    async playVoice() {
      const audioElem = this.$refs.audio as HTMLAudioElement
      const lines = this.$refs['voice-line'] as NodeListOf<HTMLElement>
      if (this.playFlag) {
        audioElem.pause()
        audioElem.currentTime = 0
        lines.forEach((line) => {
          line.style.backgroundColor = 'var(--c-text)'
        })
        this.playFlag = false
      } else {
        audioElem.play()
        this.playFlag = true
        lines.forEach((line) => {
          line.style.backgroundColor = '#999999'
        })
        for (let index = 0; index < lines.length; index++) {
          if (audioElem.paused) return
          await this.sleep((this.duration * 1000) / lines.length).then(() => {
            lines[index].style.backgroundColor = 'var(--c-text)'
          })
        }
      }
    },
  },
})
</script>

<style scoped lang="scss">
.chat-audio {
  display: flex;
  align-items: center;
}

.chat-audio svg {
  height: 1em;
  width: 1em;
}

.chat-audio svg circle {
  fill: var(--c-text);
}

.chat-audio svg path {
  fill: var(--c-bg);
}

html.dark .chat-audio svg circle {
  fill: var(--c-text);
}

html.dark .chat-audio svg path {
  fill: var(--c-bg);
}

.voice-bar {
  display: inline-flex;
  height: 1em;
  margin-left: 5px;
  margin-right: 5px;
  align-items: center;
}

.voice-bar .line {
  width: 3px;
  height: 100%;
  margin: 0 1px;
  border-radius: 2px;
  background-color: var(--c-text);
}

.voice-bar .line:nth-child(1n) {
  height: 73%;
}

.voice-bar .line:nth-child(2n) {
  height: 78%;
}

.voice-bar .line:nth-child(3n) {
  height: 84%;
}

.voice-bar .line:nth-child(4n) {
  height: 66%;
}

.voice-bar .line:nth-child(5n) {
  height: 58%;
}

.voice-bar .line:nth-child(6n) {
  height: 75%;
}

.voice-bar .line:nth-child(7n) {
  height: 95%;
}

.voice-bar .line:nth-child(8n) {
  height: 100%;
}
</style>
