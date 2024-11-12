<template>
  <div class="flex w-full h-full">
    <SvgEditorToolbar v-model="mode" />

    <div ref="containerRef" class="grow bg-gray-100">
      <canvas ref="canvasRef" />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, onMounted, watch } from "vue";
  import { Canvas, Circle, Rect } from "fabric";
  import SvgEditorToolbar from "@/components/SvgEditorToolbar.vue";

  const containerRef = ref<HTMLDivElement>();
  const canvasRef = ref<HTMLCanvasElement>();
  const canvas = ref<Canvas | null>(null);

  const mode = ref("selection"); // Model from the toolbar

  // Watch the mode prop and update the canvas mode
  watch(mode, (newMode) => {
    disableCurrentMode();

    if (newMode === "rectangle") {
      enableRectangleDrawingMode();
    } else if (newMode === "circle") {
      enableCircleDrawingMode();
    } else if (newMode === "selection") {
      enableSelectionMode();
    }
    canvas.value?.requestRenderAll(); // Ensure canvas refresh on mode change
  });

  function disableCurrentMode() {
    canvas.value?.off("mouse:down", onMouseDown);
    canvas.value?.off("mouse:move", onMouseMove);
    canvas.value?.off("mouse:up", onMouseUp);
    canvas.value!.selection = false;
    canvas.value?.forEachObject((obj) => {
      obj.selectable = false;
      obj.evented = false;
    });
  }

  function enableRectangleDrawingMode() {
    canvas.value?.on("mouse:down", onMouseDown);
    canvas.value?.on("mouse:move", onMouseMove);
    canvas.value?.on("mouse:up", onMouseUp);
  }

  function enableCircleDrawingMode() {
    canvas.value?.on("mouse:down", onMouseDown);
    canvas.value?.on("mouse:move", onMouseMove);
    canvas.value?.on("mouse:up", onMouseUp);
  }

  function enableSelectionMode() {
    canvas.value!.selection = true;
    canvas.value?.forEachObject((obj) => {
      obj.selectable = true;
      obj.evented = true; // Ensure objects are draggable and resizable
      obj.setCoords(); // Refreshes controls for Fabric.js 6
    });
    canvas.value?.requestRenderAll();
  }

  let isDrawing = false;
  let shape: Rect | Circle | null = null;
  let startX = 0;
  let startY = 0;

  // Event handlers
  function onMouseDown(event: any) {
    if (!isDrawing) {
      isDrawing = true;
      const pointer = canvas.value!.getPointer(event.e);
      startX = pointer.x;
      startY = pointer.y;

      if (mode.value === "rectangle") {
        shape = new Rect({
          left: startX,
          top: startY,
          width: 0,
          height: 0,
          fill: "rgba(0, 0, 255, 0.3)",
          stroke: "blue",
          strokeWidth: 2,
          selectable: false,
          evented: false,
        });
      } else if (mode.value === "circle") {
        shape = new Circle({
          left: startX,
          top: startY,
          radius: 0,
          fill: "rgba(0, 255, 0, 0.3)",
          stroke: "green",
          strokeWidth: 2,
          selectable: false,
          evented: false,
        });
      }

      if (shape) canvas.value?.add(shape);
    }
  }

  function onMouseMove(event: any) {
    if (isDrawing && shape) {
      const pointer = canvas.value!.getPointer(event.e);
      const width = Math.abs(pointer.x - startX);
      const height = Math.abs(pointer.y - startY);

      if (mode.value === "rectangle" && shape instanceof Rect) {
        shape.set({
          width: width,
          height: height,
          left: Math.min(pointer.x, startX),
          top: Math.min(pointer.y, startY),
        });
      } else if (mode.value === "circle" && shape instanceof Circle) {
        const radius = Math.sqrt(width * width + height * height) / 2;
        shape.set({
          radius: radius,
          left: startX - radius,
          top: startY - radius,
        });
      }

      canvas.value?.requestRenderAll();
    }
  }

  function onMouseUp() {
    isDrawing = false;

    if (shape) {
      shape.set({
        selectable: true,
        evented: true, // Ensure object is interactive after drawing
      });
      shape.setCoords(); // Required for Fabric.js 6 to update controls
    }

    shape = null;
  }

  // Initialize the Fabric.js canvas on component mount
  onMounted(() => {
    canvas.value = new Canvas(canvasRef.value!, {
      width: containerRef.value?.getBoundingClientRect().width,
      height: containerRef.value?.getBoundingClientRect().height,
      selection: true,
    });
  });
</script>
