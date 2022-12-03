<template>
    <div class="relative">
        <div class="controls absolute p-3 flex flex-col gap-2">
            <button
                class="rounded-md bg-blue-500 hover:bg-blue-600 transition duration-200 p-2 flex-none"
                @click="toggle()"
            >
                {{ video?.paused ? 'Play' : 'Pause' }}
            </button>
            <button
                class="rounded-md bg-blue-500 hover:bg-blue-600 transition duration-200 p-2 flex-none"
                @click="transparency = !transparency"
            >Transparency {{ transparency ? 'Off' : 'On' }}</button>
            <button
                class="rounded-md bg-blue-500 hover:bg-blue-600 transition duration-200 p-2 flex-none"
                @click="negative = !negative"
            >Negative {{ negative ? 'Off' : 'On' }}</button>
            <label for="volume">Volume - {{ volume }}</label>
            <input type="range" name="volume" min="0" max="0.5" step="0.025" v-model="volume">
            <div class="flex flex-row gap-2 items-center">
                <input class="w-5 h-5" type="checkbox" name="textMode" id="textMode" v-model="textMode">
                <label for="textMode">Text Mode</label>
            </div>
            <div v-if="textMode">
                <div class="flex flex-row gap-2">
                    <input class="rounded-md p-2 bg-gray-700 inline w-10" type="text" v-model="filledChar" name="filledChar" id="filledChar">
                    <input class="rounded-md p-2 bg-gray-700 inline w-10" type="text" v-model="emptyChar" name="emptyChar" id="emptyChar">
                </div>
            </div>
        </div>
        <div v-if="(arr[0].length > 0)" class="flex flex-col items-center justify-center h-screen">
            <div v-if="!textMode" class="flex flex-row leading-none" v-for="(i1, k1) in sizeH" :key="k1">
                <input
                    v-for="(i2, k2) in sizeW"
                    :key="k2"
                    class="w-4 h-4"
                    :class="{ 'invisible': (negative ? (arr[k1][k2]) : (!arr[k1][k2])) && transparency }"
                    type="checkbox"
                    :checked="!negative ? (arr[k1][k2]) : (!arr[k1][k2])"
                >
            </div>
            <div v-else class="flex flex-row leading-none" v-for="(i1, k1) in sizeH" :key="k1 + 1000">
                <span
                    v-for="(i2, k2) in sizeW"
                    :key="k2"
                    class="font-mono mr-2 text-green-400"
                    :class="{ 'invisible': (negative ? (arr[k1][k2]) : (!arr[k1][k2])) && transparency }"
                    type="checkbox"
                    :checked="!negative ? (arr[k1][k2]) : (!arr[k1][k2])"
                >{{ (!negative ? (arr[k1][k2]) : (!arr[k1][k2])) ? filledChar : emptyChar }}</span>
            </div>
        </div>
        <video class="hidden" ref="video" controls src="/bad_apple.webm" :volume="volume"></video>
        <canvas class="hidden" ref="canvas" :width="sizeW" :height="sizeH"></canvas>
    </div>
</template>

<script setup>
import { onMounted } from 'vue';
import { deltaE } from "./utils/CanvasUtils"

let sizeW = 60;
let sizeH = 45;
let canvas = $ref();
let video = $ref();
let transparency = $ref(false);
let negative = $ref(false);
let volume = $ref(0);
let textMode = $ref(false);
let filledChar = $ref('1');
let emptyChar = $ref('0');


// We have to do this terribleness, because
// otherwise Vue will treat all arrays as
// references, so changing [0][0] will also
// change [1][0] ... [n][0]
let arr = $ref([])
for (let i = 0; i < sizeH; i++) {
    arr.push(new Array(sizeW).fill(false))
} 

function toggle() {
    if (video.paused) {
        video.play();
    } else {
        video.pause();
    }
}

onMounted(() => {
    console.log(canvas, video)
    /**
     * @type {CanvasRenderingContext2D}
     */
    const ctx = canvas.getContext('2d', { alpha: false, willReadFrequently: true });

    let shouldScan = false;
    const desiredRGB = [0, 0, 0];
    video.addEventListener('play', () => {
        shouldScan = true;
        function step() {
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height)
            if (shouldScan) {
                for (let i = 0; i < sizeH; i++) {
                    let res = [];
                    for (let k = 0; k < sizeW; k++) {
                        const pixel = ctx.getImageData(k, i, 1, 1).data;
                        const rgb = [pixel[0], pixel[1], pixel[2]];
                        res.push(deltaE(rgb, desiredRGB) < 3);
                        //res.push(rgb[0] < 250)
                    }
                    arr[i] = res;
                }
                requestAnimationFrame(step);
            }
        }
        requestAnimationFrame(step);
    })

    video.addEventListener('pause', () => {
        console.log("pause")
        shouldScan = false;
    })
})
</script>

<style>
body {
    @apply bg-gray-800 text-white;
}
</style>