<script setup lang="ts">
import { ref, computed } from 'vue';
import { Cropper } from 'vue-advanced-cropper';
import 'vue-advanced-cropper/dist/style.css';
import { useImageStore } from '../stores/imageStore';
import type { AspectRatioPreset } from '../types';

const imageStore = useImageStore();
const currentImage = computed(() => imageStore.currentImage);

const presets: AspectRatioPreset[] = [
  { name: 'Square', value: 1, platform: 'instagram', type: 'square' },
  { name: 'Portrait', value: 0.8, platform: 'instagram', type: 'portrait' },
  { name: 'Landscape', value: 1.91, platform: 'instagram', type: 'landscape' },
  { name: 'Stories', value: 0.5625, platform: 'instagram', type: 'stories' },
];

const selectedPreset = ref<AspectRatioPreset>(presets[0]);
const cropperRef = ref();

const onCrop = ({ coordinates }: any) => {
  if (!currentImage.value) return;

  const crop = {
    id: crypto.randomUUID(),
    platform: selectedPreset.value.platform,
    type: selectedPreset.value.type,
    aspectRatio: selectedPreset.value.value,
    coordinates,
  };

  currentImage.value.crops = [
    ...currentImage.value.crops.filter(
      (c) =>
        c.platform !== selectedPreset.value.platform ||
        c.type !== selectedPreset.value.type
    ),
    crop,
  ];
};
</script>

<template>
  <div v-if="currentImage" class="space-y-4">
    <div class="flex space-x-4 mb-4">
      <button
        v-for="preset in presets"
        :key="preset.name"
        class="px-4 py-2 rounded"
        :class="{
          'bg-blue-500 text-white': selectedPreset === preset,
          'bg-gray-200': selectedPreset !== preset,
        }"
        @click="selectedPreset = preset"
      >
        {{ preset.name }}
      </button>
    </div>

    <Cropper
      ref="cropperRef"
      class="h-[600px]"
      :src="currentImage.previewUrl"
      :stencil-props="{
        aspectRatio: selectedPreset.value,
      }"
      @change="onCrop"
    />
  </div>
  <div v-else class="text-center py-8 text-gray-500">Není nahrána fotka</div>
</template>
