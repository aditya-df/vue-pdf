<template>
  <div
    class="absolute left-0 top-0 select-none"
    :style="{
      width: width + dw + 'px',
      height: height + dh + 'px',
      transform: 'translate(' + (x + dx) + 'px, ' + (y + dy) + 'px)',
    }"
    ref="pannable"
  >
    <div
      class="absolute w-full h-full cursor-grab"
      :class="{ 'cursor-grabbing': operation === 'move', operation: operation }"
    >
      <div
        data-direction="left"
        class="resize-border h-full w-1 left-0 top-0 border-l cursor-ew-resize"
      ></div>
      <div
        data-direction="top"
        class="resize-border w-full h-1 left-0 top-0 border-t cursor-ns-resize"
      ></div>
      <div
        data-direction="bottom"
        class="resize-border w-full h-1 left-0 bottom-0 border-b cursor-ns-resize"
      ></div>
      <div
        data-direction="right"
        class="resize-border h-full w-1 right-0 top-0 border-r cursor-ew-resize"
      ></div>
      <div
        data-direction="left-top"
        class="resize-corner left-0 top-0 cursor-nwse-resize transform -translate-x-1/2 -translate-y-1/2 md:scale-25"
      ></div>
      <div
        data-direction="right-top"
        class="resize-corner right-0 top-0 cursor-nesw-resize transform translate-x-1/2 -translate-y-1/2 md:scale-25"
      ></div>
      <div
        data-direction="left-bottom"
        class="resize-corner left-0 bottom-0 cursor-nesw-resize transform -translate-x-1/2 translate-y-1/2 md:scale-25"
      ></div>
      <div
        data-direction="right-bottom"
        class="resize-corner right-0 bottom-0 cursor-nwse-resize transform translate-x-1/2 translate-y-1/2 md:scale-25"
      ></div>
    </div>
    <div
      @click="onDelete"
      class="absolute left-0 top-0 right-0 w-12 h-12 m-auto rounded-full bg-white cursor-pointer transform -translate-y-1/2 md:scale-25"
    >
      <!-- <img class="w-full h-full" src="/delete.svg" alt="delete object" /> -->
      <p>delete</p>
    </div>
    <canvas class="w-full h-full" ref="canvas"></canvas>
  </div>
</template>

<script>
import { pannable } from "@/utils/pannable.js";
import { onMounted, ref, nextTick, createApp } from "vue";

export default {
  props: {
    payload: {},
    file: {},
    width: {},
    height: {},
    x: {},
    y: {},
    pageScale: {
      default: 1,
    },
  },
  setup(props, { emit }) {
    let startX;
    let startY;
    let canvas = ref(null);
    let operation = "";
    let directions = [];
    let dx = 0;
    let dy = 0;
    let dw = 0;
    let dh = 0;
    const pannable = ref(null);

    async function render() {
      // use canvas to prevent img tag's auto resize
      canvas.value.width = props.width;
      canvas.value.height = props.height;
      canvas.value.getContext("2d").drawImage(props.payload, 0, 0);
      let scale = 1;
      const limit = 500;
      if (props.width > limit) {
        scale = limit / props.width;
      }
      if (props.height > limit) {
        scale = Math.min(scale, limit / props.height);
      }
      emit("update", {
        width: props.width * scale,
        height: props.height * scale,
      });
      if (!["image/jpeg", "image/png"].includes(props.file.type)) {
        canvas.value.toBlob((blob) => {
          dispatch("update", {
            file: blob,
          });
        });
      }
    }

    function onDelete() {
      emit("delete");
    }

    onMounted(() => {
    //   pannable(pannable);
      render();
    });

    return {
      canvas,
      onDelete,
      operation,
      dy,
      dx,
      dw,
      dh,
      startX,
      startY,
      directions,
    };
  },
};
</script>
