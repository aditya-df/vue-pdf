<script>
import * as PDFJS from "pdfjs-dist";
import { saveAs } from "file-saver";
import { PDFDocument } from "pdf-lib";
export default {
  methods: {
    handleFileUpload() {
      const file = this.$refs.fileInput.files[0];
      const fileReader = new FileReader();
      fileReader.onload = () => {
        const buffer = fileReader.result;

        PDFJS.getDocument(buffer).promise.then((pdf) => {
          const container = this.$refs.pdfContainer;

          for (let i = 1; i <= pdf.numPages; i++) {
            pdf.getPage(i).then((page) => {
              const canvas = document.createElement("canvas");
              const scale = 1;
              const viewport = page.getViewport({ scale });
              const context = canvas.getContext("2d");

              canvas.height = viewport.height;
              canvas.width = viewport.width;

              const renderContext = {
                canvasContext: context,
                viewport,
              };

              page.render(renderContext).promise.then(() => {
                container.appendChild(canvas);
              });
            });
          }
        });
      };

      fileReader.readAsArrayBuffer(file);
    },
    async generatePDF() {
      const doc = await PDFDocument.create();

      // Mendapatkan canvas dari setiap halaman PDF yang ditampilkan
      const canvases = Array.from(
        this.$refs.pdfContainer.getElementsByTagName("canvas")
      );

      for (let i = 0; i < canvases.length; i++) {
        const canvas = canvases[i];
        const imgData = canvas.toDataURL("image/jpeg");
        const image = await doc.embedJpg(imgData);
        const page = doc.addPage();
        page.drawImage(image, {
          x: 0,
          y: 0,
          width: image.width,
          height: image.height,
        });

        // Menambahkan objek tambahan pada setiap halaman
        page.drawText(`Halaman ${i + 1}`, { x: 50, y: 50 });
      }

      // Mengakhiri dan menyimpan PDF
      const pdfBytes = await doc.save();
      const blob = new Blob([pdfBytes], { type: "application/pdf" });
      saveAs(blob, "contoh.pdf");
    },
  },
};
</script>

<template>
  <div>
    <input type="file" ref="fileInput" @change="handleFileUpload" />
    <button @click="generatePDF">Unduh PDF</button>
    <div ref="pdfContainer"></div>
  </div>
</template>
