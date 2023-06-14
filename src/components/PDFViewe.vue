<template>
  <button @click="page++">+</button>
  <button @click="page--">-</button>
  <button @click="download">download</button>
  <input
    type="file"
    id="image"
    name="image"
    style="display: none"
    @change="onUploadImage"
  />
  <label for="image">Insert Image</label>
  <div style="position: relative">
    <canvas ref="canvas"> </canvas>

    <div class="absolute top-0 left-0 transform origin-top-left">
      <the-image
        v-for="object in allObjects[page - 1]"
        :key="object"
        :file="object.file"
        :payload="object.payload"
        :x="object.x"
        :y="object.y"
        :width="object.width"
        :height="object.height"
      ></the-image>
    </div>
  </div>
</template>

<script>
import * as pdfjsLib from "pdfjs-dist/build/pdf";
import { PDFDocument } from "pdf-lib";
import download from "downloadjs";
import axios from "axios";

import TheImage from "./TheImage.vue";

var BASE64_MARKER = ";base64,";

function convertDataURIToBinary(dataURI) {
  var base64Index = dataURI.indexOf(BASE64_MARKER) + BASE64_MARKER.length;
  var base64 = dataURI.substring(base64Index);
  var raw = window.atob(base64);
  var rawLength = raw.length;
  var array = new Uint8Array(new ArrayBuffer(rawLength));

  for (var i = 0; i < rawLength; i++) {
    array[i] = raw.charCodeAt(i);
  }
  return array;
}

export default {
  components: { TheImage },
  data() {
    return {
      page: 1,
      pages: null,
      base64: null,
      allObjects: [],
    };
  },
  mounted() {
    this.renderPdf(true);
  },
  watch: {
    page() {
      this.renderPdf();
    },
  },
  methods: {
    onUploadImage(e) {
      const file = e.target.files[0];
      if (file && this.page >= 0) {
        this.addImage(file);
      }
      e.target.value = null;
    },
    genID() {
      let id = 0;
      return function genId() {
        return id++;
      };
    },
    async addImage(file) {
      try {
        // get dataURL to prevent canvas from tainted
        const url = await new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onload = () => resolve(reader.result);
          reader.onerror = reject;
          reader.readAsDataURL(file);
        });
        const img = await new Promise((resolve, reject) => {
          const img = new Image();
          img.onload = () => resolve(img);
          img.onerror = reject;
          if (file instanceof Blob) {
            const url = window.URL.createObjectURL(file);
            img.src = url;
          } else {
            img.src = src;
          }
        });
        const id = this.genID();
        const { width, height } = img;
        const object = {
          id,
          type: "image",
          width,
          height,
          x: 0,
          y: 0,
          payload: img,
          file,
        };

        this.allObjects = this.allObjects.map((objects, pIndex) => {
          return pIndex === this.page - 1 ? [...objects, object] : objects;
        });
      } catch (e) {
        console.log(`Fail to add image.`, e);
      }
    },
    renderPdf(first = false) {
      const canvas = this.$refs.canvas;
      const context = canvas.getContext("2d");

      // Mendapatkan URL file PDF
      //   const url = "/a.pdf"; // Ganti dengan path file PDF Anda

      // Render PDF ke canvas
      axios.get("/base64.txt").then((res) => {
        this.base64 = `data:application/pdf;base64,${res.data}`;
        pdfjsLib
          .getDocument(convertDataURIToBinary(this.base64))
          .promise.then((pdf) => {
            pdf.getPage(this.page).then((page) => {
              const viewport = page.getViewport({ scale: 1 });
              canvas.height = viewport.height;
              canvas.width = viewport.width;

              page.render({
                canvasContext: context,
                viewport: viewport,
              });
              const numPages = pdf.numPages;
              if (first) {
                this.pages = Array(numPages)
                  .fill()
                  .map((_, i) => pdf.getPage(i + 1));
                this.allObjects = this.pages.map(() => []);
              }
            });
          });
      });
    },
    async download() {
      let pdfDoc = await PDFDocument.load(this.base64);
      const pdfBytes = await pdfDoc.save();
      download(pdfBytes, "p", "application/pdf");
    },
  },
};
</script>
