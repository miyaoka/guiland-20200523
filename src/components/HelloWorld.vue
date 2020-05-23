<template>
  <div>
    <h1>#土曜GUI</h1>
    <h2>WebSpeechAPIでなんかやりたい</h2>

    <section class="control">
      <button @click="startRecognition">start</button>
      <button @click="stopRecognition">stop</button>
      <button @click="clearLog">clear</button>
      <label>
        show system log
        <input v-model="showSystemLog" type="checkbox" />
      </label>
    </section>

    <section class="info">
      <p v-show="interimTranscript" class="interimTranscript">
        {{ interimTranscript }}
      </p>
      <p v-for="(item, i) in filteredLog" :key="i" :class="item.type">
        <span>
          {{ item.value }}
        </span>
        <span class="time">
          {{ item.time.toLocaleTimeString('ja-JP') }}
        </span>
      </p>
    </section>

    <section class="canvas"></section>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

// eslint-disable-next-line @typescript-eslint/no-explicit-any
const { webkitSpeechRecognition } = window as any

const recognition = new webkitSpeechRecognition() as SpeechRecognition
recognition.continuous = true
recognition.lang = 'ja-JP'
recognition.interimResults = true

type LogType = 'transcript' | 'log'
type Log = {
  type: LogType
  value: string
  time: Date
}
export default Vue.extend({
  data() {
    return {
      interimTranscript: '',
      logs: [] as Log[],
      showSystemLog: true,
    }
  },
  computed: {
    filteredLog(): Log[] {
      return this.showSystemLog
        ? this.logs
        : this.logs.filter((log) => log.type === 'transcript')
    },
  },
  mounted() {
    recognition.onresult = (event) => {
      let finalTranscript = ''
      let interimTranscript = ''
      if (typeof event.results == 'undefined') {
        recognition.onend = null
        recognition.stop()
        return
      }

      for (let i = event.resultIndex; i < event.results.length; ++i) {
        if (event.results[i].isFinal) {
          finalTranscript += event.results[i][0].transcript
        } else {
          interimTranscript += event.results[i][0].transcript
        }
      }
      console.log(interimTranscript)

      if (finalTranscript != '') {
        this.addLog(finalTranscript, 'transcript')
      }

      this.interimTranscript = interimTranscript
    }
    recognition.onspeechstart = () => {
      this.addLog('speech start')
    }
    recognition.onspeechend = () => {
      this.addLog('speech end')
    }
    recognition.onsoundstart = () => {
      this.addLog('sound start')
    }
    recognition.onsoundend = () => {
      this.addLog('sound end')
    }
    recognition.onstart = () => {
      this.addLog('start')
    }
    recognition.onend = () => {
      this.addLog('end')
      // continuousが一定時間で切れるので再起動する
      recognition.start()
    }

    // this.startRecognition()
  },
  methods: {
    clearLog() {
      this.logs = []
    },
    addLog(value: string, type: LogType = 'log') {
      this.logs.unshift({
        type,
        value,
        time: new Date(),
      })
    },
    startRecognition() {
      recognition.start()
    },
    stopRecognition() {
      recognition.stop()
    },
  },
})
</script>

<style scoped lang="scss">
.info {
  display: flex;
  flex-direction: column;
}
.interimTranscript {
  color: #999;
  font-size: 16px;
  // border: 1px solid rgb(73, 66, 66);
  // border-radius: 24px;
  padding: 4px 12px;
}
.transcript {
  color: #333;
  // font-weight: 700;
  font-size: 24px;

  border: 1px solid #666;
  border-radius: 24px;
  padding: 4px 12px;
}
.time {
  font-size: 12px;
}
.log {
  color: #696;
  font-size: 12px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
