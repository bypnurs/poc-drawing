<template>
  <div id="drawing">
    <button @click="setMode('line')">Draw Line</button>
    <button @click="zoomIn">Zoom In</button>
    <button @click="zoomOut">Zoom Out</button>
    <button @click="setMode('moveCanvas')">Move Canvas</button>
    <button @click="setMode('select')">Select</button>
    <button @click="deleteSelected">Delete Selected</button>
    <button @click="resetCanvas">Reset Canvas</button>
  </div>
  <div id="canvas-container">
    <canvas id="canvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { Canvas, Line } from 'fabric';

const canvas = ref(null);
const mode = ref(null);
let currentZoom = 1;
let isMoving = false;
let currentLine = null;

const setMode = (newMode) => {
  mode.value = newMode;

  if (mode.value !== 'line') {
    canvas.value.isDrawingMode = false;
    canvas.value.off('mouse:down', startDrawing);
    canvas.value.off('mouse:move', draw);
    canvas.value.off('mouse:up', endDrawing);
  }

  if (mode.value === 'moveCanvas') {
    isMoving = true;
    canvas.value.selection = false;
    canvas.value.forEachObject((obj) => obj.set({ selectable: false }));

    canvas.value.on('mouse:down', startMove);
    canvas.value.on('mouse:move', moveCanvas);
    canvas.value.on('mouse:up', endMove);
  } else if (mode.value === 'select') {
    isMoving = false;
    canvas.value.selection = true;
    canvas.value.forEachObject((obj) => obj.set({ selectable: true }));

    canvas.value.off('mouse:down', startMove);
    canvas.value.off('mouse:move', moveCanvas);
    canvas.value.off('mouse:up', endMove);
  } else {
    isMoving = false;
    canvas.value.off('mouse:down', startMove);
    canvas.value.off('mouse:move', moveCanvas);
    canvas.value.off('mouse:up', endMove);
  }

  if (mode.value === 'line') {
    canvas.value.isDrawingMode = false;
    canvas.value.on('mouse:down', startDrawing);
    canvas.value.on('mouse:move', draw);
    canvas.value.on('mouse:up', endDrawing);
  }
};

const startDrawing = (options) => {
  if (options.target || mode.value !== 'line') return;

  const pointer = canvas.value.getPointer(options.e);
  currentLine = new Line([pointer.x, pointer.y, pointer.x, pointer.y], {
    strokeWidth: 2,
    fill: 'black',
    stroke: 'black',
    selectable: true,
    evented: true,
  });

  canvas.value.add(currentLine);
};

const draw = (options) => {
  if (!currentLine || mode.value !== 'line') return;

  const pointer = canvas.value.getPointer(options.e);
  currentLine.set({ x2: pointer.x, y2: pointer.y });
  canvas.value.renderAll();
};

const endDrawing = () => {
  currentLine = null;
};

const startMove = (options) => {
  if (!isMoving) return;

  canvas.value.isDragging = true;
  canvas.value.selection = false;
  canvas.value.lastPosX = options.e.clientX;
  canvas.value.lastPosY = options.e.clientY;
};

const moveCanvas = (options) => {
  if (!canvas.value.isDragging || !isMoving) return;

  const deltaX = options.e.clientX - canvas.value.lastPosX;
  const deltaY = options.e.clientY - canvas.value.lastPosY;

  canvas.value.viewportTransform[4] += deltaX / currentZoom;
  canvas.value.viewportTransform[5] += deltaY / currentZoom;

  canvas.value.renderAll();
  canvas.value.lastPosX = options.e.clientX;
  canvas.value.lastPosY = options.e.clientY;
};

const endMove = () => {
  canvas.value.isDragging = false;
};

const deleteSelected = () => {
  const activeObject = canvas.value.getActiveObject();
  if (activeObject) {
    console.log('Deleting object:', activeObject);
    canvas.value.remove(activeObject);
    canvas.value.discardActiveObject();
    canvas.value.renderAll();
  } else {
    console.log('No active object to delete');
  }
};

const resetCanvas = () => {
  canvas.value.clear();
};

const zoomIn = () => {
  currentZoom += 0.1;
  canvas.value.setZoom(currentZoom);
};

const zoomOut = () => {
  currentZoom -= 0.1;
  canvas.value.setZoom(currentZoom);
};

const resizeCanvas = () => {
  canvas.value.setWidth(window.innerWidth);
  canvas.value.setHeight(window.innerHeight);
  canvas.value.renderAll();
};

onMounted(() => {
  canvas.value = new Canvas('canvas', {
    selection: false,
    width: window.innerWidth,
    height: window.innerHeight,
  });

  window.addEventListener('resize', resizeCanvas);
});

onUnmounted(() => {
  window.removeEventListener('resize', resizeCanvas);
});
</script>

<style scoped>
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden; /* ป้องกันการเลื่อนทั้งหน้า */
}

#drawing {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-bottom: 1px solid #ddd;
  width: 100vw;
  box-sizing: border-box;
}

#canvas-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  margin: 0;
  padding: 0;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
}
</style>


