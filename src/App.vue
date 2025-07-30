<script setup>
import { ref, onMounted } from 'vue';

const currentColor = ref('#d40000');
const sliderValue = ref(50);
const svgContainer = ref(null);
const showTools = ref(true);

const updateSvgColor = (color) => {
  const pathColor = document.getElementById('PATH_COLOR');
  const pathBorder = document.getElementById('PATH_BORDER');

  if (pathColor) 
    pathColor.style.fill = color;

  if (pathBorder) 
    pathBorder.style.fill = color;
};

const handleColorChange = (event) => {
  const newColor = event.target.value;
  currentColor.value = newColor;
  handleChanges(newColor, sliderValue.value);
};

const handleSliderChange = (event) => {
  sliderValue.value = parseInt(event.target.value);
  handleChanges(currentColor.value, sliderValue.value);
};

const handleChanges = (newColor, newSliderValue) => {
  updateSvgColor(newColor);
  updateBorderClip(newSliderValue);
};

const updateBorderClip = (percentage) => {
  const pathBorder = document.getElementById('PATH_BORDER');
  if (pathBorder) {
    //CREATE CLIP PATH FOR pathBorder
    const clipHeight = percentage / 100;
    const imageHeight = 3950;

    // at 0%       at 100%
    // height: 0   height: 3950
    // y: -690     y: -4640 

    let clipPath = document.getElementById('CLIP_PATH');

    const rect = clipPath.querySelector('rect');
    rect.setAttribute('y', `${-690 + -1 * (imageHeight * clipHeight)}`);
    rect.setAttribute('height', `${imageHeight * clipHeight}`);
  }
};

const exportAsPNG = async () => {
  showTools.value = false;
  
  // Wait for DOM update
  await new Promise(resolve => setTimeout(resolve, 100));
  
  try {
    const html2canvas = (await import('html2canvas')).default;
    const element = document.getElementById('app');
    const canvas = await html2canvas(element, {
      backgroundColor: null,
      width: element.scrollWidth,
      height: element.scrollHeight,
      backgroundColor: 'white'
    });
    
    const link = document.createElement('a');
    link.download = 'stitch-badness-level.png';
    link.href = canvas.toDataURL();
    link.click();
  } catch (error) {
    console.error('Export failed:', error);
  } finally {
    showTools.value = true;
  }
};

const loadSvg = async () => {
  try {
    const response = await fetch('stitch.svg');
    const svgContent = await response.text();
    if (svgContainer.value) {
      svgContainer.value.innerHTML = svgContent;
      handleChanges(currentColor.value, sliderValue.value);
    }
  } catch (error) {
    console.error('Error loading SVG:', error);
  }
};

onMounted(() => {
  loadSvg();

  const value = URLSearchParams(window.location.search).get('value');
  sliderValue.value = value ? parseInt(value) : 50;
});
</script>

<template>
  <div class="title-container">
    This is your badness level. <br />It's unusually high for someone your size. <br />We (don't) have to fix that.

    <div v-if="showTools" class="tool-container">
      <div class="value-container">
        <span id="current-value">{{ sliderValue }}%</span>
      </div>
      <div class="color-picker-container">
        <input 
          type="color" 
          id="colorPicker"
          v-model="currentColor"
          @input="handleColorChange"
          class="color-picker"
        />
      </div>
      <div class="slider-container">
        <input 
          type="range" 
          id="slider"
          min="0" 
          max="100" 
          v-model="sliderValue"
          @input="handleSliderChange"
          class="slider"
          
        />
      </div>
      <button @click="exportAsPNG" class="export-button">
        Export PNG
      </button>
    </div>
  </div>
  <div>
    <div ref="svgContainer" class="svg-container"></div>
    
  </div>
</template>

<style scoped>
.title-container {
  font-size: 1.5rem;
  font-family: 'BukaBird', sans-serif;
  text-align: left;
  position: relative;
}

.tool-container {
  position: absolute;
  text-align: right;
  top: 0;
  right: 0;
  padding: 0 2rem;
}

.svg-container {
  width: 512px;
  height: 512px;
  margin: 0 auto;
}

.svg-container svg {
  width: 100%;
  height: 100%;
}

.color-picker-container,
.value-container {
  display: inline-block;
  vertical-align: middle;
  margin-left: 1rem;

}

.value-container {
  font-size: 1.5rem;
  font-family: 'BukaBird', sans-serif;
}

.color-picker {
  width: 50px;
  height: 40px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.slider {
  width: 150px;
  height: 6px;
  border-radius: 3px;
  background: #ddd;

  cursor: pointer;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

.slider::-webkit-slider-thumb {

  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: v-bind(currentColor);
  cursor: pointer;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

.slider::-moz-range-thumb,
.slider::-moz-range-progress {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: v-bind(currentColor);
  cursor: pointer;
  border: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

label {
  font-weight: bold;
  color: #333;
  font-size: 14px;
}

.export-button {
  background: v-bind(currentColor);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-family: 'BukaBird', sans-serif;
  font-size: 14px;
  margin-top: 1rem;
  transition: opacity 0.2s;
}

.export-button:hover {
  opacity: 0.8;
}
</style>
