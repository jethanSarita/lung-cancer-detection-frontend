<script setup>
import { ref } from 'vue';
import { usePrimeVue } from 'primevue/config';
import { useToast } from "primevue/usetoast";

const model_chosen = ref('');

const $primevue = usePrimeVue();
const toast = useToast();

const totalSize = ref(0);
const totalSizePercent = ref(0);
const files = ref([]);

const onRemoveTemplatingFile = (file, removeFileCallback, index) => {
    removeFileCallback(index);
    totalSize.value -= parseInt(formatSize(file.size));
    totalSizePercent.value = totalSize.value / 10;
};

const onClearTemplatingUpload = (clear) => {
    clear();
    totalSize.value = 0;
    totalSizePercent.value = 0;
};

const onSelectedFiles = (event) => {
    files.value = event.files;
    files.value.forEach((file) => {
        totalSize.value += parseInt(formatSize(file.size));
    });
};

const uploadEvent = (callback) => {
    totalSizePercent.value = totalSize.value / 10;
    callback();
};

const onTemplatedUpload = () => {
    toast.add({ severity: "info", summary: "Success", detail: "File Uploaded", life: 3000 });
};

const formatSize = (bytes) => {
    const k = 1024;
    const dm = 3;
    const sizes = $primevue.config.locale.fileSizeTypes;

    if (bytes === 0) {
        return `0 ${sizes[0]}`;
    }

    const i = Math.floor(Math.log(bytes) / Math.log(k));
    const formattedSize = parseFloat((bytes / Math.pow(k, i)).toFixed(dm));

    return `${formattedSize} ${sizes[i]}`;
};


const confidence = ref(70);

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
                <Toast />
                <FileUpload name="demo[]" url="/api/upload" @upload="onTemplatedUpload($event)" :multiple="true" accept="image/*" :maxFileSize="1000000" @select="onSelectedFiles">
                    <template #header="{ chooseCallback, uploadCallback, clearCallback, files }">
                        <div class="flex flex-wrap justify-between items-center flex-1 gap-4">
                            <div class="flex gap-2">
                                <Button @click="chooseCallback()" icon="pi pi-images" rounded outlined severity="secondary"></Button>
                                <Button @click="uploadEvent(uploadCallback)" icon="pi pi-cloud-upload" rounded outlined severity="success" :disabled="!files || files.length === 0"></Button>
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
                                <h5>Pending</h5>
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

                            <div v-if="uploadedFiles.length > 0">
                                <h5>Completed</h5>
                                <div class="flex flex-wrap gap-4">
                                    <div v-for="(file, index) of uploadedFiles" :key="file.name + file.type + file.size" class="p-8 rounded-border flex flex-col border border-surface items-center gap-4">
                                        <div>
                                            <img role="presentation" :alt="file.name" :src="file.objectURL" width="100" height="50" />
                                        </div>
                                        <span class="font-semibold text-ellipsis max-w-60 whitespace-nowrap overflow-hidden">{{ file.name }}</span>
                                        <div>{{ formatSize(file.size) }}</div>
                                        <Badge value="Completed" class="mt-4" severity="success" />
                                        <Button icon="pi pi-times" @click="removeUploadedFileCallback(index)" outlined rounded severity="danger" />
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
                <Button label="Scan" class="p-2 w-32"/>
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