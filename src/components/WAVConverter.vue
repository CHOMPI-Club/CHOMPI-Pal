<template>
    <div>
        <div style="color: black" class="font-bold">
            <small>Drag and drop or upload your samples, audition from within, and when you're set:
                click the "MAKE MY FILES" button to get a zip of CHOMPI-friendly samples.</small>
        </div>
        <hr class="divider" />
        <div class="selection-container">
            <label class="dropDownLabels">
                Engine Selector:
                <select v-model="selectedEngine" style="border: 2px solid blue; border-radius:100px">
                    <option value="jammi">JAMMI</option>
                    <option value="cubbi">CUBBI</option>
                </select>
            </label>
            <label class="dropDownLabels">
                Bank Selector:
                <select v-model="selectedBank" class="select font-black"
                    style="border: 2px solid blue; border-radius:100px">
                    <option value="a">A - Purple</option>
                    <option value="b">B - Gold</option>
                    <option value="c">C - Teal Blue</option>
                    <option value="d">D - Dark Orange</option>
                    <option value="e">E - Yellow-Green</option>
                </select>
            </label>
        </div>
        <div class="file-slots-container grid grid-cols-2 gap-2">
            <div class="file-slot" :class="{ 'file-slot-full': slot.file }" v-for="(slot, index) in fileSlots"
                :key="index" @dragover.prevent="handleDragOver" @drop.prevent="handleFileDrop(index, $event)"
                @click="handleSlotClick(index)" :style="{ width: slotWidth + 'px', height: slotHeight + 'px' }">
                <div class="slot-content">
                    <button v-if="slot.audioBuffer" @click.stop="playPreview(index)" class="play-button">
                        ▶
                    </button>
                    <input type="file" @change="handleFileSelect(index, $event)" accept=".wav" style="display: none" />
                    <label v-if="!slot.audioBuffer && !slot.file" class="font-black slot-label">{{ 'SELECT OR DRAG A FILE' }}</label>
                    <label v-if="!slot.audioBuffer && slot.file">{{ shortenFileName(slot.file.name) }}</label>
                    <div class="slot-number font-black">{{ selectedEngine + '_' + selectedBank + (index + 1) }}</div>
                </div>
            </div>

        </div>
        <div class="flex items-center justify-center my-4">
            <label class="checkbox-container flex items-center space-x-2">
                <span class="dropDownLabels">Normalize (-6db):</span>
                <input type="checkbox" v-model="normalize" class="hidden peer">
                <div
                    class="w-6 h-6 border-2 border-blue-500 bg-white rounded cursor-pointer flex items-center justify-center peer-checked:bg-blue-500 peer-checked:border-white transition-colors duration-300">
                    <Checkmark v-if="normalize" style="width: 50%;" />
                </div>
            </label>
        </div>
        <button @click="processFiles" class="px-4 py-2 makeButton font-heavy">MAKE MY FILES</button>
    </div>
</template>

<script>
import { ref } from 'vue';
import JSZip from 'jszip';
import { saveAs } from 'file-saver';
import audiobufferToWav from 'audiobuffer-to-wav';
import Checkmark from '../assets/logos/checkmark.svg'

export default {
    setup() {
        const selectedEngine = ref('jammi');
        const selectedBank = ref('a');
        const fileSlots = ref(Array.from({ length: 14 }, () => ({ file: null, audioBuffer: null })));
        const slotWidth = 120;
        const slotHeight = 80;
        const playingSlots = ref([]);
        const normalize = ref(false);

        async function normalizeAudioBuffer(audioBuffer) {
            const targetDb = -6;
            const currentDb = calculatePeakAmplitudeDb(audioBuffer);
            const gain = calculateGain(targetDb, currentDb);
            const targetSampleRate = 48000;
            const normalizedBuffer = applyGain(audioBuffer, gain, targetSampleRate);

            return normalizedBuffer;
        }

        function calculatePeakAmplitudeDb(audioBuffer) {
            let peakAmplitude = 0;

            for (let channel = 0; channel < audioBuffer.numberOfChannels; channel++) {
                const data = audioBuffer.getChannelData(channel);

                for (let i = 0; i < data.length; i++) {
                    const absSample = Math.abs(data[i]);
                    if (absSample > peakAmplitude) {
                        peakAmplitude = absSample;
                    }
                }
            }
            return 20 * Math.log10(peakAmplitude);
        }

        function calculateGain(targetDb, currentDb) {
            return Math.pow(10, (targetDb - currentDb) / 20);
        }

        function applyGain(audioBuffer, gain, targetSampleRate) {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const numberOfChannels = audioBuffer.numberOfChannels;
            const length = audioBuffer.length;
            const newBuffer = audioContext.createBuffer(numberOfChannels, length, targetSampleRate);

            for (let channel = 0; channel < numberOfChannels; channel++) {
                const oldData = audioBuffer.getChannelData(channel);
                const newData = newBuffer.getChannelData(channel);

                for (let i = 0; i < length; i++) {
                    newData[i] = oldData[i] * gain;
                }
            }

            return newBuffer;
        }


        async function convertAudioBufferToWav(audioBuffer) {
            const options = {
                float32: false,
                stereo: true,
            };

            return new Promise((resolve) => {
                const wavBuffer = audiobufferToWav(audioBuffer, options);
                resolve(new Uint8Array(wavBuffer));
            });
        }

        function handleFileSelect(index, event) {
            const fileInput = event.target;
            const file = fileInput.files[0];
            updateSlotFile(index, file);
        }

        function handleDragOver(event) {
            event.dataTransfer.dropEffect = 'copy';
        }

        function handleFileDrop(index, event) {
            const file = event.dataTransfer.files[0];
            updateSlotFile(index, file);
        }

        function updateSlotFile(index, file) {
            if (file) {
                readFile(file)
                    .then((arrayBuffer) => convertToAudioBuffer(arrayBuffer))
                    .then((audioBuffer) => {
                        fileSlots.value[index].file = file;
                        fileSlots.value[index].audioBuffer = audioBuffer;
                    })
                    .catch((error) => {
                        console.error('Error handling file:', error);
                    });
            }
        }


        async function createDoubleSpeedBuffer(audioBuffer) {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();

            const offlineContext = new OfflineAudioContext(
                audioBuffer.numberOfChannels,
                audioBuffer.duration * audioBuffer.sampleRate,
                audioBuffer.sampleRate
            );

            const source = offlineContext.createBufferSource();
            source.buffer = audioBuffer;
            source.playbackRate.value = 2;
            source.connect(offlineContext.destination);
            source.start(0);

            const renderedBuffer = await offlineContext.startRendering();
            return renderedBuffer;
        }


        async function processFiles() {
            const zip = new JSZip();

            for (let i = 0; i < fileSlots.value.length; i++) {
                const slot = fileSlots.value[i];
                const { file, audioBuffer } = slot;

                if (file && audioBuffer) {
                    try {
                        const normalizedAudioBuffer = normalize.value ? await normalizeAudioBuffer(audioBuffer) : audioBuffer;
                        const wavBuffer = await convertAudioBufferToWav(normalizedAudioBuffer);
                        zip.file(`${selectedEngine.value}_${selectedBank.value}${i + 1}.wav`, wavBuffer);

                        const doubleSpeedBuffer = await createDoubleSpeedBuffer(normalizedAudioBuffer); 
                        const doubleWavBuffer = await convertAudioBufferToWav(doubleSpeedBuffer);
                        zip.file(`${selectedEngine.value}_${selectedBank.value}${i + 1}_double.wav`, doubleWavBuffer);
                    } catch (error) {
                        console.error('Error processing file:', error);
                    }
                } else {
                    console.log(`No file or audio buffer selected for slot ${i + 1}`);
                }
            }

            zip.generateAsync({ type: 'blob' })
                .then((blob) => {
                    console.log('Zip file size:', blob.size, 'bytes');
                    saveAs(blob, 'yum_chompi_samples.zip');
                });
        }


        function readFile(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = (event) => resolve(event.target.result);
                reader.onerror = (error) => reject(error);
                reader.readAsArrayBuffer(file);
            });
        }

        function convertToAudioBuffer(arrayBuffer) {
            return new Promise((resolve, reject) => {
                const audioContext = new (window.AudioContext || window.webkitAudioContext)();
                const sampleRate = 48000;

                audioContext.decodeAudioData(arrayBuffer, (decodedData) => {
                    resolve(decodedData);
                }, (error) => {
                    reject(error);
                }).then((decodedData) => {
                    const offlineContext = new OfflineAudioContext(2, decodedData.length, sampleRate);
                    const source = offlineContext.createBufferSource();
                    source.buffer = decodedData;

                    source.connect(offlineContext.destination);
                    source.start(0);
                    offlineContext.startRendering().then((renderedBuffer) => {
                        resolve(renderedBuffer);
                    }).catch((error) => {
                        reject(error);
                    });
                });
            });
        }

        function playPreview(index) {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const source = audioContext.createBufferSource();
            const audioBuffer = fileSlots.value[index].audioBuffer;

            playingSlots.value.forEach((source) => source.stop());

            const newSource = audioContext.createBufferSource();
            newSource.buffer = audioBuffer;
            newSource.connect(audioContext.destination);
            newSource.start();

            playingSlots.value = [newSource];
        }

        function handleSlotClick(index) {
            const fileInput = document.createElement('input');
            fileInput.type = 'file';
            fileInput.accept = '.wav';
            fileInput.style.display = 'none';
            fileInput.addEventListener('change', (event) => handleFileSelect(index, event));
            document.body.appendChild(fileInput);
            fileInput.click();
            document.body.removeChild(fileInput);
        }

        function shortenFileName(fileName) {
            return fileName;
        }

        return {
            selectedEngine,
            selectedBank,
            fileSlots,
            slotWidth,
            slotHeight,
            playingSlots,
            normalize,
            handleSlotClick,
            handleFileSelect,
            handleDragOver,
            handleFileDrop,
            processFiles,
            playPreview,
            shortenFileName,
        };
    },
    components: {
        Checkmark
    }
};
</script>

