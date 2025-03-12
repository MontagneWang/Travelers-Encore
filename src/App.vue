<template>
  <transition name="fade">
    <div v-show="isMasked" class="mask" @click="startEmsemble">
      <transition name="fade" mode="out-in">
        <div :key="loadingText">
          <div class="loading" v-show="!isLoaded"></div>
          <img
            class="ready"
            v-show="isLoaded"
            alt="click"
            src="data:image/png;base64,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"
          />
          {{ loadingText }}
        </div>
      </transition>
    </div>
  </transition>
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
      :class="[
        'planet',
        `planet-${planet.id}`,
        { paused: hoveredPlanet === planet.id },
      ]"
      @mouseover="hoveredPlanet = planet.id"
      @mouseleave="hoveredPlanet = null"
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

const isMasked = ref(true);
const isLoaded = ref(false);
const hoveredPlanet = ref<null | number>(null);
const loadingText = ref("Preparing Instruments......");
const audioBufferedStatus = ref<number[]>(Array(7).fill(-1)); // 音频加载状态
let localVolume: number[] = [];

interface Planet {
  id: number;
  image: string;
  outline: string;
  src: string;
  srcOnline: string;
  volume: number; // todo 会被转换为 string，需要排查
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
    src: "/1.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731720465519/1.mp3",
    volume: 0.35,
    buffer: null,
    sourceNode: null,
  }, // 余烬双星
  {
    id: 2,
    image:
      "https://i0.hdslb.com/bfs/article/0efb6937678de02bdc1ba2132069bf541402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/9fe31d82efb736179f5336b45fc3b6591402305269.png@1e_1c.webp",
    src: "/2.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731720509341/2.mp3",
    volume: 0.1,
    buffer: null,
    sourceNode: null,
  }, // 废岩星
  {
    id: 3,
    image:
      "https://i0.hdslb.com/bfs/article/236623c6a6f527f620aa05fca381ae581402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/10d5133031b805e5b4aae44d3f58e8fb1402305269.png@1e_1c.webp",
    src: "/3.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731720509487/3.mp3",
    volume: 0.5,
    buffer: null,
    sourceNode: null,
  }, // 碎空星
  {
    id: 4,
    image:
      "https://i0.hdslb.com/bfs/article/0d0f8a1d1a599ac635d72c1a4d0ec5181402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/e4b078c28deb0120be29be9f024070a71402305269.png@1e_1c.webp",
    src: "/4.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731720509639/4.mp3",
    volume: 0.4,
    buffer: null,
    sourceNode: null,
  }, // 外星站
  {
    id: 5,
    image:
      "https://i0.hdslb.com/bfs/article/6fc016f64c490e3a3c478fb5e32b2cdf1402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/f8511547e065b28aef4e4c19b3bd36681402305269.png@1e_1c.webp",
    src: "/5.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731720509774/5.mp3",
    volume: 0.7,
    buffer: null,
    sourceNode: null,
  }, // 量子卫星
  {
    id: 6,
    image:
      "https://i0.hdslb.com/bfs/article/1904c9a5e5f2d0fc7d60b49bb1b73d9d1402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/969599e2d5c70e1568fd7e6068c5d2e71402305269.png@1e_1c.webp",
    src: "/6.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731721179091/6.mp3",
    volume: 0.9,
    buffer: null,
    sourceNode: null,
  }, // 深巨星
  {
    id: 7,
    image:
      "https://i0.hdslb.com/bfs/article/fb3e5d5b42bbd74cf01da36a91d3f3e71402305269.png@1e_1c.webp",
    outline:
      "https://i0.hdslb.com/bfs/article/85340e698712cfaf3ccb620dd4f39d1c1402305269.png@1e_1c.webp",
    src: "/7.mp3",
    srcOnline:
      "https://fs-im-kefu.7moor-fs1.com/ly/4d2c3f00-7d4c-11e5-af15-41bf63ae4ea0/1731721179356/7.mp3",
    volume: 0.55,
    buffer: null,
    sourceNode: null,
  }, // 黑棘星
]);

const startEmsemble = () => {
  // 两者都要确保，只确保 isLoaded 会导致 mask 存在时可快速点击多次产生重播问题
  if (isLoaded.value && isMasked.value) {
    isMasked.value = false;
    for (let i = 0; i < planetList.value.length; i++) {
      playAudio(i);
    }
  }
};

const audioContext = new window.AudioContext();
const gainNodes = planetList.value.map(() => audioContext.createGain());

// const fetchAudio = async (planet: Planet, i: number) => {
//   const response = await fetch(planet.src);
//   const arrayBuffer = await response.arrayBuffer();
//   planet.buffer = await audioContext.decodeAudioData(arrayBuffer);

//   if (i === planetList.value.length - 1) {
//     isLoaded.value = true;
//     loadingText.value = "Let's Play Together";
//   }
// };

// 资源竞速加载
const fetchAudio = async (planet: Planet, i: number) => {
  const fetchResource = async (
    url: string
  ): Promise<{ url: string; buffer: ArrayBuffer }> => {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`获取资源失败 ${url}`);
    }
    const buffer = await response.arrayBuffer();
    return { url, buffer };
  };

  try {
    const result = await Promise.race([
      fetchResource(planet.src),
      fetchResource(planet.srcOnline),
    ]);

    planet.buffer = await audioContext.decodeAudioData(result.buffer);

    console.log(
      `第${i + 1}星球音频竞速加载结果: ${
        result.url.startsWith("http") ? "Online" : "Local"
      }`
    );

    audioBufferedStatus.value[i] = 0; // 已就绪
    if (audioBufferedStatus.value.every(v => v === 0)) {
      isLoaded.value = true;
      loadingText.value = "Let's Play Together";
    }
  } catch (error) {
    console.error("获取资源失败:", error);
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
    crescendoVolume(i);
    planet.sourceNode.start(0);
  }
};

const changeVolume = (i: number) => {
  gainNodes[i].gain.value = planetList.value[i].volume;
  localVolume[i] = Number(planetList.value[i].volume);
  localStorage.setItem("Planet", JSON.stringify(localVolume));
};

const crescendoVolume = (i: number) => {
  const fadeDuration = 2.7; // 渐变持续时间（秒）
  const currentTime = audioContext.currentTime;
  gainNodes[i].gain.cancelScheduledValues(currentTime);
  gainNodes[i].gain.setValueAtTime(0, currentTime); // 从 0 音量与当前时间开始
  gainNodes[i].gain.linearRampToValueAtTime(
    planetList.value[i].volume,
    currentTime + fadeDuration
  );
  localVolume[i] = Number(planetList.value[i].volume);
  localStorage.setItem("Planet", JSON.stringify(localVolume));
};

onMounted(() => {
  try {
    localVolume = JSON.parse(localStorage.getItem("Planet") || "[]");
  } catch (error) {
    localVolume = [];
    console.warn("请不要手动修改您的 localStorage");
  }
  planetList.value.forEach((planet: Planet, i: number) => {
    // 保证本地读取的音量在 0-1 之间，防止用户手动修改导致音量过大
    // planetList.value[i].volume = Math.max(0, Math.min(1, localVolume[i])) ?? planetList.value[i].volume; // Math.min(1, undefined) => NaN
    planetList.value[i].volume =
      localVolume[i] !== undefined
        ? Math.max(0, Math.min(1, localVolume[i]))
        : planetList.value[i].volume;
    fetchAudio(planet, i);
  });

  window.addEventListener("keydown", handleKeydown);
});

const handleKeydown = (event: KeyboardEvent) => {
  switch (event.key) {
    case " ":
      // 空格键重置 localStorage
      localStorage.clear();
      break;
    case "ArrowUp":
      planetList.value.forEach((planet: Planet, i: number) => {
        // 确保音量不会超出 1
        planet.volume = Math.min(
          parseFloat((Number(planet.volume) + 0.1).toFixed(2)),
          1
        );
        changeVolume(i);
      });
      break;
    case "ArrowDown":
      planetList.value.forEach((planet: Planet, i: number) => {
        // 确保音量不会跌出 0
        planet.volume = Math.max(
          parseFloat((Number(planet.volume) - 0.1).toFixed(2)),
          0
        );
        changeVolume(i);
      });
      break;
    case "ArrowLeft":
      console.log("Left, 切换音质");
      break;
    case "ArrowRight":
      console.log("Right, 切换音质");
      break;
  }
};

onBeforeUnmount(() => {
  window.removeEventListener("keydown", handleKeydown);
  audioContext.close();
});
</script>

<style scoped lang="scss">
* {
  user-select: none;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
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
  1: 150,
  // 余烬双星
  2: 250,
  // 废岩星
  3: 350,
  // 碎空星
  4: 450,
  // 外星站
  5: 550,
  // 量子卫星
  6: 650,
  // 深巨星
  7: 750,
  // 黑棘星
);

$orbit-durations: (
  1: 15,
  // 余烬双星
  2: 25,
  // 废岩星
  3: 35,
  // 碎空星
  4: 100000,
  // 外星站相对太阳静止
  5: 45,
  // 量子卫星
  6: 55,
  // 深巨星
  7: 75,
  // 黑棘星
);

.solar-system {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  background: url("https://i0.hdslb.com/bfs/article/a3dfab455801de0e8dc5bdd9067476391402305269.png@1e_1c.webp") no-repeat center / cover;
  // background: url("@/assets/bg.webp") no-repeat center / cover;
}

.sun {
  width: 120px;
  height: 120px;

  img {
    width: 100%;
    height: 100%;
    animation: rotate 25s linear infinite; // 添加自转动画
  }
}

.planet {
  position: absolute;
  width: 80px;
  height: 80px;
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
    width: 80px;
    transform: translate(0, 425%);

    #volume {
      width: 100%;
    }
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
// todo 先做量子卫星环绕行星公转的动画，之后量子态只需要切换绑定的环绕行星的 ID 即可
@each $id, $size in $planet-sizes {
  .planet-#{$id} {
    animation: orbit-#{$id} #{map-get($orbit-durations, $id)}s linear infinite;
  }

  @keyframes orbit-#{$id} {
    from {
      transform: rotate(0deg) translateX(#{$size}px) rotate(0deg);
    }

    to {
      transform: rotate(360deg) translateX(#{$size}px) rotate(-360deg);
    }
  }
}
</style>
