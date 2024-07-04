<template>
  <div v-show="isMasked" class="mask" @click="startEmsemble">
    <transition name="fade" mode="out-in">
      <div :key="showText">
        <div class="loading" v-show="!isLoaded"></div>
        <img class="ready" src="/click.png" v-show="isLoaded" alt="click" />
        {{ showText }}
      </div>
    </transition>
  </div>
  <div class="solar-system">
    <div class="sun">
      <img
        src="https://i0.hdslb.com/bfs/article/70514e9dee9c0954416e8893db540c961402305269.png@1e_1c.webp"
        alt="Sun"
      />
    </div>
    <div
      v-for="(planet, i) in planetList"
      :key="planet.id"
      :class="['planet', `planet-${planet.id}`, { paused: isPaused }]"
      :style="{ animationDuration: planet.orbitDuration + 's' }"
      @mouseover="isPaused = true"
      @mouseleave="isPaused = false"
    >
      <img
        :src="planet.image"
        :style="{ opacity: planet.volume }"
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
          v-model="planet.volume"
          @input="changeVolume(i)"
        />
      </div>

      <!-- todo 每个单独的文本框, 悬停时显示 -->
      <!-- <div class="text" v-show="isPaused">
        <div class="loading" v-show="!isLoaded"></div>

        <img src="" alt="" />
      </div> -->
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

const isPaused = ref(false);
const isMasked = ref(true);
const isLoaded = ref(false);
const showText = ref("Preparing Instruments......");

interface Planet {
  id: number;
  image: string;
  outline: string;
  orbitDuration: number;
  src: string;
  volume: number;
  buffer: AudioBuffer | null;
  sourceNode: AudioBufferSourceNode | null;
}

const planetList = ref<Planet[]>([
  {
    id: 1,
    image:
      "https://i0.hdslb.com/bfs/article/5070bf8512ce1c308fbf2bd832d919e51402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/3be14c053f206c2a12880db8503eb8d61402305269.png@1e_1c.webp",
    orbitDuration: 10,
    src: "/1.mp3",
    volume: 0.4,
    buffer: null,
    sourceNode: null,
  },
  {
    id: 2,
    image:
      "https://i0.hdslb.com/bfs/article/0efb6937678de02bdc1ba2132069bf541402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/9fe31d82efb736179f5336b45fc3b6591402305269.png@1e_1c.webp",
    orbitDuration: 20,
    src: "/2.mp3",
    volume: 0.15,
    buffer: null,
    sourceNode: null,
  },
  {
    id: 3,
    image:
      "https://i0.hdslb.com/bfs/article/236623c6a6f527f620aa05fca381ae581402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/10d5133031b805e5b4aae44d3f58e8fb1402305269.png@1e_1c.webp",
    orbitDuration: 30,
    src: "/3.mp3",
    volume: 0.5,
    buffer: null,
    sourceNode: null,
  },
  {
    id: 4,
    image:
      "https://i0.hdslb.com/bfs/article/0d0f8a1d1a599ac635d72c1a4d0ec5181402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/e4b078c28deb0120be29be9f024070a71402305269.png@1e_1c.webp",
    orbitDuration: 10000000,
    src: "/4.mp3",
    volume: 0.5,
    buffer: null,
    sourceNode: null,
  },
  {
    id: 5,
    image:
      "https://i0.hdslb.com/bfs/article/6fc016f64c490e3a3c478fb5e32b2cdf1402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/f8511547e065b28aef4e4c19b3bd36681402305269.png@1e_1c.webp",
    orbitDuration: 45,
    src: "/5.mp3",
    volume: 0.7,
    buffer: null,
    sourceNode: null,
  },
  {
    id: 6,
    image:
      "https://i0.hdslb.com/bfs/article/1904c9a5e5f2d0fc7d60b49bb1b73d9d1402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/969599e2d5c70e1568fd7e6068c5d2e71402305269.png@1e_1c.webp",
    orbitDuration: 55,
    src: "/6.mp3",
    volume: 0.9,
    buffer: null,
    sourceNode: null,
  },
  {
    id: 7,
    image:
      "https://i0.hdslb.com/bfs/article/fb3e5d5b42bbd74cf01da36a91d3f3e71402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/85340e698712cfaf3ccb620dd4f39d1c1402305269.png@1e_1c.webp",
    orbitDuration: 70,
    src: "/7.mp3",
    volume: 0.5,
    buffer: null,
    sourceNode: null,
  },
]);

const startEmsemble = () => {
  if (isLoaded.value) {
    isMasked.value = false;
    for (let i = 0; i < planetList.value.length; i++) {
      playAudio(i);
    }
    isLoaded.value = false; // 禁止二次触发
  }
};

const audioContext = new window.AudioContext();
const gainNodes = planetList.value.map(() => audioContext.createGain());

const fetchAudio = async (planet: Planet, i: number) => {
  const response = await fetch(planet.src);
  const arrayBuffer = await response.arrayBuffer();
  planet.buffer = await audioContext.decodeAudioData(arrayBuffer);

  // todo 兼容
  if (i === planetList.value.length - 1) {
    isLoaded.value = true;
    showText.value = "Let's Play Together";
  }
};

const playAudio = (i: number) => {
  const planet = planetList.value[i];
  if (planet.buffer) {
    if (planet.sourceNode) {
      planet.sourceNode.disconnect();
    }
    planet.sourceNode = audioContext.createBufferSource();
    planet.sourceNode.buffer = planet.buffer;
    planet.sourceNode.connect(gainNodes[i]);
    gainNodes[i].connect(audioContext.destination);
    planet.sourceNode.loop = true;
    changeVolume(i);
    planet.sourceNode.start(0);
  }
};

const changeVolume = (i: number) => {
  gainNodes[i].gain.value = planetList.value[i].volume;
  // todo 写入 localStorage
  localStorage.setItem(
    "Planet",
    JSON.stringify([
      // fixme 写入的会有重复
      ...JSON.parse(localStorage.getItem("Planet") || "[]"),
      {
        id: planetList.value[i].id,
        volume: planetList.value[i].volume,
      },
    ])
  );
};

onMounted(() => {
  // todo 读取赋值
  JSON.parse(localStorage.getItem("Planet") || "[]")?.forEach(
    (item: { id: number; volume: number }) => {
      planetList.value[item.id - 1].volume = item.volume;
    }
  );
  planetList.value.forEach((planet: Planet, i: number) => {
    fetchAudio(planet, i);
  });
});

onBeforeUnmount(() => {
  audioContext.close();
});
</script>

<style scoped lang="scss">
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.75s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.ready {
  width: 4vw;
  height: 4vw;
  animation: 1.8s 1s infinite blink;
  margin-top: 0.6vw;
  vertical-align: text-top;

  img {
    width: 100%;
    height: 100%;
  }
}

@keyframes blink {
  50% {
    opacity: 0;
  }
}
// todo 加载
.loading {
  display: inline-block;
  vertical-align: text-top;
  width: 6vw;
  height: 6vw;
  margin-right: 0.5vw;
  margin-top: -0.3vw;
  animation: rotate 4s infinite linear;
  border: 0.2vw solid #fff;
  border-radius: 100%;

  // 卫星
  &::before,
  &::after {
    content: "";
    border-radius: 100%;
    position: absolute;
    left: 0.05vw;
    top: 0.05vw;
    width: 1.5vw;
    height: 1.5vw;
    background: radial-gradient(circle, #66ccff 20%, #fff);
    box-shadow: 0 0 1vw #fff;
  }

  // 中心天体fff
  &::after {
    width: 3vw;
    height: 3vw;
    margin: 1.5vw;
    background: radial-gradient(circle, #fff700 20%, #ee0000);
    box-shadow: 0 0 2vw #ee0000;
  }
}

// todo 文本框样式
.text {
  width: 300px;
  height: 100px;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 10px;
  position: relative;

  &:before {
    content: "";
    position: absolute;
    top: 20px;
    left: -20px;
    border: 10px solid transparent;
    border-right: 10px solid rgba(0, 0, 0, 0.7);
  }
}

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

  // &.paused {
  //   animation-play-state: paused !important;

  //   img {
  //     animation-play-state: paused !important; // 确保自转动画也暂停
  //   }
  // }
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

    // todo 单独暂停
    &.paused {
      animation-play-state: paused !important;
      img {
        animation-play-state: paused !important;
      }
    }
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
