<script setup>
import { ref } from 'vue';
import { usePrimeVue } from 'primevue/config';
import { useToast } from "primevue/usetoast";

const $primevue = usePrimeVue();
const toast = useToast();

const model_chosen = ref('');
const confidence = ref(70);
const files = ref([]);
const detections = ref([]); // Store API results

const totalSize = ref(0);
const totalSizePercent = ref(0);

const isUploading = ref(false);

const formatSize = (bytes) => {
    const k = 1024;
    const dm = 3;
    const sizes = $primevue.config.locale.fileSizeTypes;

    if (bytes === 0) return `0 ${sizes[0]}`;
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return `${(bytes / Math.pow(k, i)).toFixed(dm)} ${sizes[i]}`;
};

const onSelectedFiles = (event) => {
    files.value = event.files;
    totalSize.value = files.value.reduce((sum, file) => sum + file.size, 0);
    totalSizePercent.value = totalSize.value / 10;
};

const onRemoveTemplatingFile = (file, removeFileCallback, index) => {
    removeFileCallback(index);
    totalSize.value -= file.size;
    totalSizePercent.value = totalSize.value / 10;
};

const onClearTemplatingUpload = (clear) => {
    clear();
    totalSize.value = 0;
    totalSizePercent.value = 0;
};

const uploadEvent = async (callback) => {
    await handleImageUploads();
    //callback(); // Update UI
};

const onTemplatedUpload = () => {
    toast.add({ severity: "success", summary: "Uploaded", detail: "Files successfully uploaded", life: 3000 });
};

/** 🧠 API upload logic */
const handleImageUploads = async () => {
    isUploading.value = true;
    detections.value = []; // Reset
    for (const file of files.value) {
        const formData = new FormData();
        formData.append("file", file);
        formData.append("confidence", (confidence.value / 100).toFixed(2));
        // formData.append("model", model_chosen.value);

        try {
            const response = await fetch("https://lung-cancer-detecton-backend.onrender.com/detect/", {
                method: "POST",
                body: formData,
            });

            if (!response.ok) {
                throw new Error(`HTTP Error! Status: ${response.status}`);
            }

            const result = await response.json();

            // Push response along with the original file name
            detections.value.push({
                // filename: file.name,
                result: result,
                annotatedImage: `data:image/png;base64,${result.image}`
            });

        } catch (error) {
            console.error("Error uploading file:", error);
        }
    }
    isUploading.value = false;
};
</script>

<template>
    <div class="flex justify-center">
        <div class="w-[75%] flex flex-col">
            <!-- Title -->
            <div class="pt-10">
                <p class="text-5xl font-bold text-surface-700 pb-5 text-center">
                LungVision Demo
            </p>
            </div>
            <!-- Desc -->
            <p class="text-xl text-surface-700 pt-5 pb-5 ">
                Upload one or more lung CT scan images, and the app will detect potential cancerous regions.
            </p>
            <p class="text-sm text-surface-600 ">Disclaimer: This model is not perfect and may produce false positives or false negatives. Please consult a medical professional for a thorough diagnosis.</p>
            <!-- Upload Dialog -->
            <div class="card w-full pt-5 pb-5">
                <FileUpload pt:content:class="overflow-y-auto h-96" @upload="onTemplatedUpload($event)" :multiple="true" accept="image/*" :maxFileSize="500000000" @select="onSelectedFiles">
                    <template #header="{ chooseCallback, uploadCallback, clearCallback, files }">
                        <div class="flex flex-wrap justify-between items-center flex-1 gap-4">
                            <div class="flex gap-2">
                                <Button @click="chooseCallback()" icon="pi pi-cloud-upload" rounded outlined severity="success"></Button>
                                <Button @click="clearCallback()" icon="pi pi-times" rounded outlined severity="danger" :disabled="!files || files.length === 0"></Button>
                            </div>
                            <ProgressBar :value="totalSizePercent" :showValue="false" class="md:w-20rem h-1 w-full md:ml-auto">
                                <span class="whitespace-nowrap">{{ totalSize }}B / 1Mb</span>
                            </ProgressBar>
                        </div>
                    </template>
                    <template #content="{ files, uploadedFiles, removeUploadedFileCallback, removeFileCallback }">
                        <div class="flex flex-col gap-8 pt-4">
                            <div v-if="files.length > 0">
                                <div class="flex flex-wrap gap-4">
                                    <div v-for="(file, index) of files" :key="file.name + file.type + file.size" class="p-8 rounded-border flex flex-col border border-surface items-center gap-4">
                                        <div>
                                            <img role="presentation" :alt="file.name" :src="file.objectURL" width="100" height="50" />
                                        </div>
                                        <span class="font-semibold text-ellipsis max-w-60 whitespace-nowrap overflow-hidden">{{ file.name }}</span>
                                        <div>{{ formatSize(file.size) }}</div>
                                        <Badge value="Pending" severity="warn" />
                                        <Button icon="pi pi-times" @click="onRemoveTemplatingFile(file, removeFileCallback, index)" outlined rounded severity="danger" />
                                    </div>
                                </div>
                            </div>
                        </div>
                    </template>
                    <template #empty>
                        <div class="flex items-center justify-center flex-col">
                            <i class="pi pi-cloud-upload !border-2 !rounded-full !p-8 !text-4xl !text-muted-color" />
                            <p class="mt-6 mb-0">Drag and drop files to here to upload.</p>
                        </div>
                    </template>
                </FileUpload>
            </div>
            <!-- Slider -->
            <div class="w-full">
                <p class="text-xl text-surface-700 pt-5 pb-5">Confidnce Threshold <span class="text-primary-600">{{ confidence }}%</span></p>
                <Slider v-model="confidence" class="w-full" pt:handle:class="before:bg-primary-500 bg-primary-500"/>
            </div>
            <!-- Radio Buttons -->
            <p class="text-xl text-surface-700 pt-5 pb-5">Choose the YOLO Model for predictions</p>
            <div class="flex flex-col gap-4 pb-5 text-surface-700">
                <div class="flex items-center gap-2">
                    <RadioButton v-model="model_chosen" inputId="model_chosen1" name="model_chosen" value="YOLOv5mu (v0.0.2b)"/>
                    <label for="model_chosen1">YOLOv5mu (v0.0.2b)</label>
                </div>
                <div class="flex items-center gap-2">
                    <RadioButton v-model="model_chosen" inputId="model_chosen2" name="model_chosen" value="YOLOv5lu (v0.0.2c)" />
                    <label for="model_chosen2">YOLOv5lu (v0.0.2c)</label>
                </div>
                <div class="flex items-center gap-2">
                    <RadioButton v-model="model_chosen" inputId="model_chosen3" name="model_chosen" value="PlaceHolder" />
                    <label for="model_chosen3">PlaceHolder</label>
                </div>
            </div>
            <!-- Scan Button -->
            <div class="flex justify-center pb-10">
                <Button @click="uploadEvent(uploadCallback)" :disabled="!files || files.length === 0" label="Scan" class="p-2 w-32"/>
            </div>
            <p v-if="isUploading" class="text-center text-primary-500 font-medium pb-4">Processing images, please wait...</p>
            <div v-if="detections.length > 0" class="pt-10">
                <h3 class="text-2xl font-bold text-surface-700 pb-5">Detection Results</h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div v-for="(detection, index) in detections" :key="index" class="p-4 border border-surface rounded-lg shadow-sm">
                        <h4 class="text-lg font-semibold mb-2 text-surface-800">{{ detection.filename }}</h4>
                        <img :src="detection.annotatedImage" alt="Detection result" class="w-full rounded border" />
                        <div class="mt-2 text-sm text-surface-600">
                            <p><strong>Raw Result:</strong></p>
                            <pre class="bg-surface-100 p-2 rounded overflow-x-auto">{{ detection.result }}</pre>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.p-slider {
    @apply bg-surface-400
}

.custom-slider-handle::before {
    background: #22c55e;
}
</style>