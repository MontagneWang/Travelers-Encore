<template>
  <div v-if="showMask" class="mask" @click="concert">{{ showText }}</div>
  <div class="solar-system">
    <div class="sun">
      <img src="https://i0.hdslb.com/bfs/article/70514e9dee9c0954416e8893db540c961402305269.png@1e_1c.webp" alt="Sun" />
    </div>
    <div
      v-for="(planet, index) in planets"
      :key="planet.id"
      :class="['planet', `planet-${planet.id}`, { paused: isPaused }]"
      :style="{ animationDuration: planet.orbitDuration + 's' }"
      @mouseover="pauseAnimation"
      @mouseleave="resumeAnimation"
    >
      <img
        :src="planet.image"
        :style="{ opacity: playlist[index].volume }"
        alt="Planet"
      />
      <img :src="planet.outline" alt="Planet-Outline" />

      <div class="controls">
        <input
          type="range"
          id="volume"
          min="0"
          max="1"
          step="0.01"
          v-model="playlist[index].volume"
          @input="changeVolume(index)"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

let showMask = ref(true);
let loaded = ref(false);
let showText = ref("Loading......");

const concert = () => {
  if (loaded.value) {
    showMask.value = false;
    for (let index = 0; index < 7; index++) {
      playAudio(index);
    }
    loaded.value = false; // 禁止二次触发
  }
};

interface Track {
  src: string;
  volume: number;
  buffer: AudioBuffer | null;
  sourceNode: AudioBufferSourceNode | null;
}

const playlist = ref<Track[]>([
  { src: "/1.mp3", volume: 0.4, buffer: null, sourceNode: null }, // 余烬双星
  { src: "/2.mp3", volume: 0.15, buffer: null, sourceNode: null }, // 废岩星
  { src: "/3.mp3", volume: 0.5, buffer: null, sourceNode: null }, // 碎空星
  { src: "/4.mp3", volume: 0.5, buffer: null, sourceNode: null }, // 外星站
  { src: "/5.mp3", volume: 0.75, buffer: null, sourceNode: null }, // 量子卫星
  { src: "/6.mp3", volume: 0.9, buffer: null, sourceNode: null }, // 深巨星
  { src: "/7.mp3", volume: 0.4, buffer: null, sourceNode: null }, // 黑棘星
]);

const audioContext = new window.AudioContext();
const gainNodes = playlist.value.map(() => audioContext.createGain());

const fetchAudio = async (track: Track, index: number) => {
  const response = await fetch(track.src);
  const arrayBuffer = await response.arrayBuffer();
  track.buffer = await audioContext.decodeAudioData(arrayBuffer);
  if (index === 6) {
    setTimeout(() => {
      showText.value = "Click to Start";
      loaded.value = true;
    }, 0);
  }
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
    track.sourceNode.loop = true;
    track.sourceNode.start(0);
    gainNodes[index].gain.value = playlist.value[index].volume;
  }
};

const changeVolume = (index: number) => {
  gainNodes[index].gain.value = playlist.value[index].volume;
};

onMounted(() => {
  playlist.value.forEach((track, index) => {
    fetchAudio(track, index);
  });
});

onBeforeUnmount(() => {
  audioContext.close();
});

const planets = ref([
  {
    id: 1,
    image:
      "https://i0.hdslb.com/bfs/article/5070bf8512ce1c308fbf2bd832d919e51402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/3be14c053f206c2a12880db8503eb8d61402305269.png@1e_1c.webp",
    orbitDuration: 10,
  }, // 余烬双星
  {
    id: 2,
    image:
      "https://i0.hdslb.com/bfs/article/0efb6937678de02bdc1ba2132069bf541402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/9fe31d82efb736179f5336b45fc3b6591402305269.png@1e_1c.webp",
    orbitDuration: 20,
  }, // 废岩星
  {
    id: 3,
    image:
      "https://i0.hdslb.com/bfs/article/236623c6a6f527f620aa05fca381ae581402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/10d5133031b805e5b4aae44d3f58e8fb1402305269.png@1e_1c.webp",
    orbitDuration: 30,
  }, // 碎空星
  {
    id: 4,
    image:
      "https://i0.hdslb.com/bfs/article/0d0f8a1d1a599ac635d72c1a4d0ec5181402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/e4b078c28deb0120be29be9f024070a71402305269.png@1e_1c.webp",
    orbitDuration: 10000000,
  }, // 外星站与太阳保持相对静止
  {
    id: 5,
    image:
      "https://i0.hdslb.com/bfs/article/6fc016f64c490e3a3c478fb5e32b2cdf1402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/f8511547e065b28aef4e4c19b3bd36681402305269.png@1e_1c.webp",
    orbitDuration: 45,
  }, // 量子卫星应该绕指定的三颗行星旋转（之后切换网页显隐时移动到其他行星上）
  {
    id: 6,
    image:
      "https://i0.hdslb.com/bfs/article/1904c9a5e5f2d0fc7d60b49bb1b73d9d1402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/969599e2d5c70e1568fd7e6068c5d2e71402305269.png@1e_1c.webp",
    orbitDuration: 55,
  }, // 深巨星
  {
    id: 7,
    image:
      "https://i0.hdslb.com/bfs/article/fb3e5d5b42bbd74cf01da36a91d3f3e71402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/85340e698712cfaf3ccb620dd4f39d1c1402305269.png@1e_1c.webp",
    orbitDuration: 70,
  }, // 黑棘星
  // { id: 1, image: "./1.png", outline: "./11.png", orbitDuration: 10 }, // 余烬双星
  // { id: 2, image: "./2.png", outline: "./22.png", orbitDuration: 20 }, // 废岩星
  // { id: 3, image: "./3.png", outline: "./33.png", orbitDuration: 30 }, // 碎空星
  // { id: 4, image: "./4.png", outline: "./44.png", orbitDuration: 10000000 }, // 外星站与太阳保持相对静止
  // { id: 5, image: "./5.png", outline: "./55.png", orbitDuration: 45 }, // 量子卫星应该绕指定的三颗行星旋转（之后切换网页显隐时移动到其他行星上）
  // { id: 6, image: "./6.png", outline: "./66.png", orbitDuration: 55 }, // 深巨星
  // { id: 7, image: "./7.png", outline: "./77.png", orbitDuration: 70 }, // 黑棘星
]);

// 动画控制
const isPaused = ref(false);
const pauseAnimation = () => {
  isPaused.value = true;
};
const resumeAnimation = () => {
  isPaused.value = false;
};
</script>

<style scoped lang="scss">
.mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  color: white;
  text-align: center;
  line-height: 100vh;
  font-size: 4vw;
  user-select: none;
  background-color: rgba(0, 0, 0, 0.75);
  z-index: 2;
}

// 调整公转半径
$planet-sizes: (
  1: 150px,
  2: 250px,
  3: 400px,
  4: 550px,
  5: 700px,
  6: 850px,
  7: 1000px,
);

$orbit-durations: (
  1: 5s,
  2: 7s,
  3: 10s,
  4: 12s,
  5: 15s,
  6: 18s,
  7: 20s,
);

.solar-system {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  // background: url("@/assets/bg.jpeg") no-repeat center / cover;
  background: url("@/assets/bg.webp") no-repeat center / cover;
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
    position: absolute;
    animation: rotate 15s linear infinite; // 添加自转动画
  }

  .controls {
    transform: translate(-15%, 550%);
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
