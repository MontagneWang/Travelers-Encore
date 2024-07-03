<template>
  <div class="solar-system">
    <div class="sun">
      <img src="/8.png" alt="Sun" />
    </div>
    <div v-for="planet in planets" :key="planet.id" :class="['planet', `planet-${planet.id}`, { paused: isPaused }]"
      :style="{ animationDuration: planet.orbitDuration + 's' }" @mouseover="pauseAnimation"
      @mouseleave="resumeAnimation">
      <img :src="planet.image" alt="Planet" />

      <!-- todo 把下面的音乐播放器和上面的合并在一起 通过一个 v-for 实现 -->
      <div v-for="(track, index) in playlist" :key="index" class="player">
      <h2>{{ track.title }}</h2>
      <div class="controls">
        <button @click="playAudio(index)">播放</button>
        <button @click="pauseAudio(index)">暂停</button>

        <label for="volume">音量:</label>
        <input type="range" id="volume" min="0" max="1" step="0.01" v-model="track.volume" @input="changeVolume(index)">

        <label for="loop">循环播放:</label>
        <input type="checkbox" id="loop" v-model="track.loop">
      </div>

      <div class="progress-container">
        <input type="range" min="0" max="100" step="0.01" v-model="track.progress" @input="seekAudio(index)">
      </div>
    </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, onBeforeUnmount } from 'vue';

interface Track {
  title: string;
  src: string;
  volume: number;
  loop: boolean;
  progress: number;
  buffer: AudioBuffer | null;
  sourceNode: AudioBufferSourceNode | null;
}

const playlist = ref<Track[]>([
  { title: 'Track 1', src: '/1.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null },
  { title: 'Track 2', src: '/2.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null },
  { title: 'Track 3', src: '/3.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null },
  { title: 'Track 4', src: '/4.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null },
  { title: 'Track 5', src: '/5.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null },
  { title: 'Track 6', src: '/6.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null },
  { title: 'Track 7', src: '/7.flac', volume: 0.5, loop: true, progress: 0, buffer: null, sourceNode: null }
]);

const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const gainNodes = playlist.value.map(() => audioContext.createGain());

const fetchAudio = async (track: Track, index: number) => {
  const response = await fetch(track.src);
  const arrayBuffer = await response.arrayBuffer();
  track.buffer = await audioContext.decodeAudioData(arrayBuffer);
};

const playAudio = (index: number) => {
  const track = playlist.value[index];
  if (track.buffer) {
    if (track.sourceNode) {
      track.sourceNode.disconnect();
    }
    track.sourceNode = audioContext.createBufferSource();
    track.sourceNode.buffer = track.buffer;
    track.sourceNode.connect(gainNodes[index]);
    gainNodes[index].connect(audioContext.destination);
    track.sourceNode.loop = track.loop;
    track.sourceNode.start(0, (track.progress / 100) * track.buffer.duration);
  }
};

const pauseAudio = (index: number) => {
  const track = playlist.value[index];
  if (track.sourceNode) {
    track.sourceNode.stop();
  }
};

const changeVolume = (index: number) => {
  gainNodes[index].gain.value = playlist.value[index].volume;
};

const updateProgress = (index: number) => {
  const track = playlist.value[index];
  if (track.sourceNode && track.buffer) {
    const currentTime = audioContext.currentTime;
    const startTime = parseFloat(track.sourceNode.start.toString());
    track.progress = ((currentTime - startTime) / track.buffer.duration) * 100;
  }
};

const seekAudio = (index: number) => {
  const track = playlist.value[index];
  if (track.buffer) {
    const seekTime = (track.progress / 100) * track.buffer.duration;
    if (track.sourceNode) {
      track.sourceNode.stop();
      track.sourceNode.disconnect();
    }
    track.sourceNode = audioContext.createBufferSource();
    track.sourceNode.buffer = track.buffer;
    track.sourceNode.connect(gainNodes[index]);
    gainNodes[index].connect(audioContext.destination);
    track.sourceNode.loop = track.loop;
    track.sourceNode.start(0, seekTime);
  }
};

const onTrackEnd = (index: number) => {
  const track = playlist.value[index];
  if (track.loop) {
    playAudio(index);
  } else {
    track.progress = 0;
  }
};

onMounted(() => {
  playlist.value.forEach((track, index) => {
    fetchAudio(track, index);
  });

  // 定时更新播放进度
  setInterval(() => {
    playlist.value.forEach((track, index) => {
      updateProgress(index);
    });
  }, 1000);
});

onBeforeUnmount(() => {
  audioContext.close();
});

const planets = ref([
  { id: 1, image: './1.png', orbitDuration: 10 }, // 余烬双星
  { id: 2, image: './2.png', orbitDuration: 20 }, // 废岩星
  { id: 3, image: './3.png', orbitDuration: 30 }, // 碎空星
  { id: 4, image: './4.png', orbitDuration: 10000000 }, // 外星站与太阳保持相对静止
  { id: 5, image: './5.png', orbitDuration: 45 }, // 量子卫星应该绕指定的三颗行星旋转（之后切换网页显隐时移动到其他行星上）
  { id: 6, image: './6.png', orbitDuration: 55 }, // 深巨星
  { id: 7, image: './7.png', orbitDuration: 70 } // 黑棘星
]);

const isPaused = ref(false);

const pauseAnimation = () => {
  isPaused.value = true;
};

const resumeAnimation = () => {
  isPaused.value = false;
};
</script>

<style scoped lang="scss">
// 调整公转半径
$planet-sizes: (
  1: 150px,
  2: 250px,
  3: 400px,
  4: 550px,
  5: 700px,
  6: 850px,
  7: 1000px
);

$orbit-durations: (
  1: 5s,
  2: 7s,
  3: 10s,
  4: 12s,
  5: 15s,
  6: 18s,
  7: 20s
);

.solar-system {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  background: url('@/assets/bg.jpeg') no-repeat center / cover
}

.sun {
  width: 100px;
  height: 100px;

  img {
    width: 100%;
    height: 100%;
    animation: rotate 25s linear infinite; // 添加自转动画
  }
}

.planet {
  position: absolute;
  width: 100px;
  height: 100px;
  transform-origin: center;
  animation: rotate 20s linear infinite;

  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    animation: rotate 15s linear infinite; // 添加自转动画
  }

  &.paused {
    animation-play-state: paused !important;

    img {
      animation-play-state: paused !important; // 确保自转动画也暂停
    }
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

@each $id, $size in $planet-sizes {
  .planet-#{$id} {
    animation: orbit-#{$id} map-get($orbit-durations, $id) linear infinite;
  }

  @keyframes orbit-#{$id} {
    from {
      transform: rotate(0deg) translateX($size) rotate(0deg);
    }

    to {
      transform: rotate(360deg) translateX($size) rotate(-360deg);
    }
  }
}
</style>
