<template>
  <div id="app">
    <div class="container">
      <div class="video">
        <div class="off-screen" v-show="isButtonVisible"></div>
        <img
          v-show="!isButtonVisible && isNoiseVisible"
          class="noise"
          src="https://media.giphy.com/media/YRcXl6VfNhCorklI0R/giphy.gif"
          alt="tv snow and noise"
        />
        <youtube
          :video-id="currentVideoId"
          ref="youtube"
          :player-vars="playerVars"
        ></youtube>
      </div>
      <div class="controls">
        <button
          class="start-button"
          v-if="!isLoading && isButtonVisible"
          @click="showPlayer"
        >
          <img class="icon" :src="power" alt="power icon" />
        </button>
      </div>
      <img class="tv" :src="tv" alt="old tv frame" />
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { getIdFromUrl } from 'vue-youtube';
import tv from '@/assets/tv.png';
import power from '@/assets/power.svg';

export default {
  name: 'App',
  data() {
    return {
      videos: [],
      playerVars: {
        autoplay: 1,
        controls: 0,
      },
      isButtonVisible: true,
      isNoiseVisible: false,
      isLoading: true,
      currentIndex: -1,
      tv,
      power,
    };
  },
  computed: {
    currentVideoId() {
      return this.videos.length && this.currentIndex !== -1
        ? getIdFromUrl(this.videos[this.currentIndex].data.url)
        : '';
    },
    player() {
      return this.$refs.youtube && this.$refs.youtube.player;
    },
  },
  methods: {
    async showPlayer() {
      this.isButtonVisible = false;
      this.isNoiseVisible = true;
      await new Promise((resolve) => setTimeout(resolve, 2000));
      this.isNoiseVisible = false;
      this.currentIndex = Math.floor(Math.random() * this.videos.length);
      this.player.addEventListener('onStateChange', this.youtubeStateChange);
    },
    youtubeStateChange(stateChange) {
      if (stateChange.data === 0) {
        this.currentIndex = Math.floor(Math.random() * this.videos.length);
      }
    },
  },
  async created() {
    let videos = [];
    let page = await axios.get(
      'https://api.reddit.com/r/interdimensionalcable/top?t=all'
    );
    videos = videos.concat(page.data.data.children);
    for (let i = 0; i < 4; i++) {
      const after = page.data.data.after;
      if (!after) break;
      page = await axios.get(
        `https://api.reddit.com/r/interdimensionalcable/top?t=all&after=${after}`
      );
      videos = videos.concat(page.data.data.children);
    }
    this.videos = videos.filter(
      (vid) => vid.data.media_embed && !vid.data.over_18
    );
    this.isLoading = false;
  },
};
</script>

<style>
.container {
  display: flex;
  height: 100vh;
  width: 100vw;
  justify-content: center;
  position: relative;
}

.tv {
  height: 90vmin;
  width: auto;
  z-index: 99;
}

.video {
  z-index: 1;
  position: absolute;
  width: 85vmin;
  height: 50vmin;
  top: 8vmin;
}

.video iframe,
.noise {
  width: 100%;
  height: 100%;
}

.noise {
  position: absolute;
  top: 0;
  left: 0;
}

.off-screen {
  background-color: black;
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

.controls {
  position: absolute;
  width: 85vmin;
  height: 10vmin;
  top: 78vmin;
  z-index: 1000;
}

.start-button {
  left: 13vmin;
  position: absolute;
  border-radius: 50%;
  width: 5vmin;
  height: 5vmin;
  background-color: black;
  box-shadow: 0px 0px 35px 10px #ff000086;
  cursor: pointer;
}

.start-button:hover {
  background-color: grey;
}

.icon {
  height: 100%;
  width: 100%;
}
</style>
