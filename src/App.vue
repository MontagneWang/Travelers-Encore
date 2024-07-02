<template>
  <div class="solar-system">
    <div class="sun">
      <img src="/8.png" alt="Sun" />
    </div>
    <div v-for="planet in planets" :key="planet.id" :class="['planet', `planet-${planet.id}`, { paused: isPaused }]"
      :style="{ animationDuration: planet.orbitDuration + 's' }" @mouseover="pauseAnimation"
      @mouseleave="resumeAnimation">
      <img :src="planet.image" alt="Planet" />

      <div class="controls">
        <button @click="playAudio">播放</button>
        <button @click="pauseAudio">暂停</button>

        <label for="volume">音量:</label>
        <input type="range" id="volume" min="0" max="1" step="0.01" v-model="volume" @input="changeVolume">

        <label for="loop">循环播放:</label>
        <input type="checkbox" id="loop" v-model="loop">
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';

const audioSrc = './所莱内姆.flac'; // 替换为你的音频文件路径
const volume = ref(0.5);
const loop = ref(true); // 是否启用循环播放

let audioContext: AudioContext | null = null;
let audioBuffer: AudioBuffer | null = null;
let sourceNode: AudioBufferSourceNode | null = null;
let gainNode: GainNode | null = null;

const fetchAudio = async (url: string) => {
  const response = await fetch(url);
  const arrayBuffer = await response.arrayBuffer();
  if (audioContext) {
    return audioContext.decodeAudioData(arrayBuffer);
  }
  return null;
};

const playAudio = () => {
  if (audioContext && audioBuffer) {
    if (sourceNode) {
      sourceNode.stop();
    }
    sourceNode = audioContext.createBufferSource();
    sourceNode.buffer = audioBuffer;
    sourceNode.loop = loop.value;
    sourceNode.connect(gainNode as GainNode).connect(audioContext.destination);
    sourceNode.start(0);
  }
};

const pauseAudio = () => {
  if (sourceNode) {
    sourceNode.stop();
    sourceNode = null;
  }
};

const changeVolume = () => {
  if (gainNode) {
    gainNode.gain.value = volume.value;
  }
};

onMounted(async () => {
  audioContext = new (window.AudioContext || window.webkitAudioContext)();
  gainNode = audioContext.createGain();
  gainNode.gain.value = volume.value;
  audioBuffer = await fetchAudio(audioSrc);
});

watch(loop, (newLoopValue) => {
  if (sourceNode) {
    sourceNode.loop = newLoopValue;
  }
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
