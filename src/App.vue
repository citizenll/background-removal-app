<script setup lang="ts">
import imglyRemoveBackground, { Config } from "@imgly/background-removal"
import { ref, watch } from "vue";

const quality = ref("medium")
let overlay = ref(false)
let src = ref("")
const files = ref([])
const caption = ref('Click me to remove background');
const progressing = ref(false)
const config: Config = {
    publicPath: "./assets/",
    model: "medium",
    progress: (key: string, current: any, total: number) => {
        const [type, subtype] = key.split(':');
        caption.value = `${type} ${subtype} ${(
            (current / total) *
            100
        ).toFixed(0)}%`;
    },
}
function start_work() {
    progressing.value = true
    if (files.value.length == 0) {
        progressing.value = false
        return
    }
    const file = files.value[0]
    config.model = quality.value
    imglyRemoveBackground(file, config).then((blob: Blob) => {
        const url = URL.createObjectURL(blob);
        src.value = url
        overlay.value = true
        progressing.value = false
    })
}
watch(files, (newVal, oldVal) => {
    if (newVal) {
        caption.value = 'Click me to remove background';
    } else {
        caption.value = 'Select a photo';
    }
}, { immediate: true })
</script>

<template>
    <div class="container">
        <section class="section">
            <v-file-input label="Select a photo" variant="outlined" v-model="files">
            </v-file-input>
            <v-select label="Model" v-model="quality" :items="['medium', 'small']" variant="solo"></v-select>
            <div style="padding: 20px;">
                <v-btn @click="start_work" :disabled="progressing">{{ caption }}</v-btn>
            </div>
        </section>
        <v-overlay v-model="overlay" class="overlay">
            <p>Click the right mouse button to save the image</p>
            <img :src="src" class="cleared-img" />
        </v-overlay>
        <v-progress-circular model-value="20" class="progress" indeterminate v-if="progressing"></v-progress-circular>
    </div>
</template>

<style>
.container {
    padding: 20px;
}

.progress {
    position: fixed;
    right: 10px;
    bottom: 10px;
}

.section {
    text-align: center;
    height: 100%;
}

.overlay {
    justify-content: center !important;
    align-items: center !important;
}

.cleared-img {
    width: 70%;
    object-fit: contain;
}

.v-overlay__content {
    text-align: center;
}
</style>
