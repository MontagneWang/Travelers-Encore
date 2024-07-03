<template>
  <div v-if="showMask" class="mask" @click="concert">{{ showText }}</div>
  <div class="solar-system">
    <div class="sun">
      <img src="/8.png" alt="Sun" />
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
  title: string;
  src: string;
  volume: number;
  loop: boolean;
  progress: number;
  buffer: AudioBuffer | null;
  sourceNode: AudioBufferSourceNode | null;
}

const playlist = ref<Track[]>([
  {
    title: "Track 1",
    src: "/1.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
  {
    title: "Track 2",
    src: "/2.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
  {
    title: "Track 3",
    src: "/3.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
  {
    title: "Track 4",
    src: "/4.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
  {
    title: "Track 5",
    src: "/5.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
  {
    title: "Track 6",
    src: "/6.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
  {
    title: "Track 7",
    src: "/7.mp3",
    volume: 0.5,
    loop: true,
    progress: 0,
    buffer: null,
    sourceNode: null,
  },
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
    track.sourceNode.loop = track.loop;
    track.sourceNode.start(0, (track.progress / 100) * track.buffer.duration);
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
  { id: 1, image: "./1.png", outline: "./11.png", orbitDuration: 10 }, // 余烬双星
  { id: 2, image: "./2.png", outline: "./22.png", orbitDuration: 20 }, // 废岩星
  { id: 3, image: "./3.png", outline: "./33.png", orbitDuration: 30 }, // 碎空星
  { id: 4, image: "./4.png", outline: "./44.png", orbitDuration: 10000000 }, // 外星站与太阳保持相对静止
  { id: 5, image: "./5.png", outline: "./55.png", orbitDuration: 45 }, // 量子卫星应该绕指定的三颗行星旋转（之后切换网页显隐时移动到其他行星上）
  { id: 6, image: "./6.png", outline: "./66.png", orbitDuration: 55 }, // 深巨星
  { id: 7, image: "./7.png", outline: "./77.png", orbitDuration: 70 }, // 黑棘星
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
  background: url("@/assets/bg.png") no-repeat center / cover;
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
