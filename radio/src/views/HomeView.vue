<template>
  <v-container class="container-background">
    <h1 class="text-center">Radio</h1>
    <v-row>
      <v-col v-for="(radio, index) in radios" :key="index" cols="12" sm="4" md="4" lg="4">
        <v-card class="radio-card punk-card">
          <div class="btn-container">
            <v-btn @click="toggleFavorite(radio)" class="ma-2" color="transparent">
              <v-icon :color="radio.isFavorite ? 'red' : 'white'" size="24">mdi-heart</v-icon>
            </v-btn>
          </div>

          <v-row justify="center">
            <v-col cols="12" class="text-center">
              <v-img :src="radio.favicon ? radio.favicon : require('@/assets/no immage.png')" height="200" contain></v-img>
            </v-col>
          </v-row>

          <v-card-title class="text-center radio-title">{{ radio.name }}</v-card-title>
          <v-card-actions class="d-flex justify-center">
            <v-btn @click="playRadio(radio)" color="transparent" class="ma-2">
              <div class="play-button-container">
                <font-awesome-icon :icon="radio.isPlaying ? 'pause' : 'play'" style="color: white;" />
              </div>
            </v-btn>
            <img v-if="isPlaying(radio)" src="@/assets/soundwave.gif" class="loading-gif" alt="Caricamento GIF">
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { faPlay, faPause } from '@fortawesome/free-solid-svg-icons'
import Hls from 'hls.js';

library.add(faPlay, faPause);

export default {
  name: 'RadioView',
  data() {
    return {
      radios: [],
      currentPlayingRadio: null,
      audio: new Audio(),
      isAudioReady: false,
    }
  },
  methods: {
    getRadios() {
      return fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => {
          if (!response.ok) {
            throw new Error('Errore durante il recupero dei dati delle stazioni radio');
          }
          return response.json();
        })
        .then(data => {
          this.radios = data.map(radio => ({
            ...radio,
            isFavorite: false
          }));
        })
        .catch(error => {
          console.error('Si è verificato un errore durante il recupero dei dati:', error);
        });
    },
    playRadio(radio) {
  if (!this.audio) {
    console.error('L\'oggetto audio non è stato inizializzato correttamente.');
    return;
  }
  if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
    if (this.audio.hls) {
      this.audio.hls.destroy();
    }
    this.audio.pause();
    // Reset the icon of the previously playing radio to play
    this.currentPlayingRadio.isPlaying = false;
  }
  if (this.isPlaying(radio)) {
    // If the same radio is clicked again, pause it
    this.audio.pause();
    radio.isPlaying = false; // Set isPlaying to false
    this.currentPlayingRadio = null;
  } else {
    // Start playing the selected radio
    if (radio.url.endsWith('.m3u8')) {
      if (Hls.isSupported()) {
        this.audio.hls = new Hls();
        this.audio.hls.loadSource(radio.url);
        this.audio.hls.attachMedia(this.audio);
        this.audio.addEventListener('canplaythrough', () => {
          this.audio.play();
          radio.isPlaying = true; // Set isPlaying to true
          this.currentPlayingRadio = radio;
        }, false);
      } else if (this.audio.canPlayType('application/x-mpegURL')) {
        this.audio.src = radio.url;
        this.audio.addEventListener('canplaythrough', () => {
          this.audio.play();
          radio.isPlaying = true; // Set isPlaying to true
          this.currentPlayingRadio = radio;
        }, false);
      } else {
        console.error('Il browser non supporta la riproduzione di file m3u8.');
      }
    } else {
      this.audio.src = radio.url;
      this.audio.addEventListener('canplaythrough', () => {
        this.audio.play();
        radio.isPlaying = true; // Set isPlaying to true
        this.currentPlayingRadio = radio;
      }, false);
    }
  }
},
    isPlaying(radio) {
      return this.currentPlayingRadio === radio;
    },
    toggleFavorite(radio) {
      radio.isFavorite = !radio.isFavorite;
    }
  },
  created() {
    this.getRadios();
  },
  components: {
    FontAwesomeIcon
  },
  mounted() {
    library.add(faPlay, faPause);
  }
}
</script>

<style scoped>
.radio-card {
  margin-bottom: 20px;
  position: relative;
  border: 2px solid #1976D2;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 350px; /* Aumenta l'altezza di 10 pixel */
}


.loading-gif {
  width: 170px;
  height: auto;
}

.punk-card {
  background: linear-gradient(45deg, #ff69b4, #8a2be2);
}

.radio-title {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  text-transform: uppercase;
}

.btn-container {
  position: absolute;
  top: 5px;
  right: 5px;
  z-index: 1;
}

.play-button-container {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: transparent;
  transition: background-color 0.3s;
  cursor: pointer;
}

.play-button-container:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.play-button-container:active {
  background-color: rgba(255, 255, 255, 0.4);
}
</style>
