<template>
  <div class="audio-page mb-5">
    <MediaRecorderHeader />

    <div class="shadow-sm p-5 bg-white rounded">
      <b-button variant="primary" class="mr-2" @click="start" :disabled="startDisabled">Record</b-button>
      <b-button variant="outline-primary" class="mr-2" @click="pause" :disabled="stopDisabled">Pause</b-button>
      <b-button variant="outline-success" class="mr-2" @click="resume" :disabled="startDisabled">Resume</b-button>
      <b-button variant="secondary" @click="stop" :disabled="stopDisabled">Stop</b-button>

      <hr>

      <h6 class="text-dark">Tip:</h6>
      <pre class="bg-light p-3">
State:  inactive
State:  recording
State:  paused
State:  recording</pre>

      <div v-if="hasAudios">
        <ul class="list-unstyled audio-list">
          <li v-for="(item, index) in audioList" :key="index" class="audio-item mb-3">
            <div class="d-flex justify-content-between">
              <audio :controls="true" :loop="false" :src="item.src" class="mr-2 w-100"></audio>
              <div class="d-flex align-items-center">
                <b-button variant="danger" size="md" @click="removeItem(index)"> Delete </b-button>
              </div>
            </div>
          </li>
        </ul>
      </div>

    </div>
  </div>
</template>

<script>
import MediaRecorderHeader from '@/components/HeaderMediaRecorderAPI'
export default {
  name: 'Audio',
  components: { MediaRecorderHeader },
  data () {
    return {
      supported: undefined,
      mediaRecorder: undefined,
      chunks: [],
      audioList: [],
      isRecording: false
    }
  },
  computed: {
    startDisabled () {
      return this.isRecording === true
    },
    stopDisabled () {
      return this.isRecording === false
    },
    hasAudios () {
      return this.audioList.length > 0
    }
  },
  methods: {
    start () {
      console.log('State: ', this.mediaRecorder.state)
      this.mediaRecorder.start()
      this.isRecording = true
    },
    stop () {
      console.log('State: ', this.mediaRecorder.state)
      this.mediaRecorder.stop()
      this.isRecording = false
    },
    pause () {
      console.log('State: ', this.mediaRecorder.state)
      this.mediaRecorder.pause()
      this.isRecording = false
    },
    resume () {
      console.log('State: ', this.mediaRecorder.state)
      this.mediaRecorder.resume()
      this.isRecording = true
    },
    removeItem (index) {
      this.audioList.splice(index, 1)
    }
  },
  created () {
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      this.supported = true
      navigator.mediaDevices.getUserMedia({ audio: true })
        .then((stream) => {
          console.log(stream)
          this.mediaRecorder = new MediaRecorder(stream)
          this.mediaRecorder.ondataavailable = (e) => {
            this.chunks.push(e.data)
          }
          this.mediaRecorder.onstop = (e) => {
            console.log(stream)
            console.log('recorder stopped')
            var blob = new Blob(this.chunks, { 'type': 'audio/ogg; codecs=opus' })
            this.chunks = []
            const audioURL = window.URL.createObjectURL(blob)
            this.audioList.push({
              src: audioURL
            })
            window.URL.revokeObjectURL(blob)
          }
        })
        .catch((err) => {
          console.log('Err: ' + err)
        })
    } else {
      this.supported = false
    }
  }
}
</script>
