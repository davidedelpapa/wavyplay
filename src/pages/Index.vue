<template>
  <q-page class>
    <div class="audio-container">
      <div class="row q-ma-md">
        <div class="col-12">
          <div id="waveform"></div>
        </div>
      </div>
    </div>
    <div class="controls row flex flex-center fixed-bottom q-pb-md q-pt-md shadow-10">
      <div class>
        <q-btn
          color="primary"
          flat
          round
          icon="fast_rewind"
          size="xl"
          @click="wavesurfer.skipBackward(1)"
        />
        <q-btn
          v-if="isPlaying"
          color="primary"
          round
          icon="pause"
          size="xl"
          @click="wavesurfer.playPause()"
        />
        <q-circular-progress v-if="isLoading" size="72px" indeterminate color="primary" />
        <q-btn
          v-if="!isPlaying && !isLoading"
          color="primary"
          round
          icon="play_arrow"
          size="xl"
          @click="wavesurfer.playPause()"
        />
        <q-btn
          color="primary"
          flat
          round
          icon="fast_forward"
          size="xl"
          @click="wavesurfer.skipForward(1)"
        />
      </div>
    </div>
  </q-page>
</template>

<script>
import { EventBus } from '../services/event-bus.js'
import WaveSurfer from 'wavesurfer.js'

export default {
  name: 'PageIndex',
  data: () => ({
    wavesurfer: null,
    isLoading: false
  }),
  async mounted () {
    if (!this.wavesurfer) this.createWaveSurfer()
    EventBus.$on('fileChosen', file => {
      this.loadFile(file)
    })
  },
  methods: {
    createWaveSurfer () {
      this.wavesurfer = WaveSurfer.create({
        container: '#waveform',
        hideScrollbar: true,
        waveColor: 'white',
        progressColor: 'hsla(200, 100%, 30%, 0.5)',
        cursorColor: '#fff',
        barWidth: 3
      })
      this.wavesurfer.on('error', err => {
        console.error(err)
        this.isLoading = false
        this.$q.notify({ message: err })
      })

      this.wavesurfer.on('loading', () => {
        this.isLoading = true
      })

      this.wavesurfer.on('ready', () => {
        this.isLoading = false
      })
    },
    loadFile (file) {
      if (file.target.files.length === 0) return
      this.wavesurfer.loadBlob(file.target.files[0])
    }
  },
  computed: {
    isPlaying () {
      if (!this.wavesurfer) return false

      return this.wavesurfer.isPlaying()
    }
  }
}
</script>
