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

    <section class="logs">
      <p v-show="interimTranscript" class="interimTranscript balloon4">
        {{ interimTranscript }}
      </p>
      <p
        v-for="(item, i) in filteredLog"
        :key="i"
        class="balloon3-left"
        :class="item.type"
      >
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
      showSystemLog: false,
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
.logs {
  display: flex;
  flex-direction: column;
  align-items: center;
}
p {
  margin: 10px;
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

  // border: 1px solid #666;
  // border-radius: 24px;
  // padding: 4px 12px;
}

.balloon3-left {
  position: relative;
  padding: 20px;
  // line-height: 90px;
  color: #fff;
  font-size: 20px;
  font-weight: bold;
  background: #ffcc75;
  border-radius: 20px;
  box-sizing: border-box;
}

.balloon3-left:before {
  content: '';
  position: absolute;
  top: 50%;
  left: -25px;
  margin-top: -15px;
  border: 15px solid transparent;
  border-right: 15px solid #ffcc75;
  z-index: 0;
}

.balloon4 {
  position: relative;
  margin: 2em 0 2em 40px;
  padding: 15px;
  background: #fff0c6;
  border-radius: 30px;
}

.balloon4:before {
  content: '';
  position: absolute;
  left: -38px;
  width: 13px;
  height: 12px;
  bottom: 0;
  background: #fff0c6;
  border-radius: 50%;
}

.balloon4:after {
  content: '';
  position: absolute;
  left: -24px;
  width: 20px;
  height: 18px;
  bottom: 3px;
  background: #fff0c6;
  border-radius: 50%;
}
.balloon4 p {
  margin: 0;
  padding: 0;
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
