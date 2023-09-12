<template>
  <div class="flex flex-col">
    <h2>Uploader un fichier HTML ou saisir du code HTML</h2>

    <label>
      <input type="radio" v-model="uploadOption" value="file" /> Uploader un
      fichier
    </label>

    <label>
      <input type="radio" v-model="uploadOption" value="code" /> Saisir du code
      HTML
    </label>

    <div v-if="uploadOption === 'file'" class="py-2">
      <input type="file" @change="handleFileUpload" accept=".html" />
    </div>

    <div v-else class="py-2">
      <textarea v-model="htmlCode" rows="10" cols="30"></textarea>
    </div>

    <label>
      Format de l'image :
      <input type="radio" v-model="outputFormat" value="png" /> PNG
      <input type="radio" v-model="outputFormat" value="jpeg" /> JPEG
      <input type="radio" v-model="outputFormat" value="pdf" /> PDF
    </label>

    <label>
      Télécharger automatiquement l'image :
      <input type="checkbox" v-model="autoDownload" />
    </label>

    <div v-if="htmlCode" id="html-content" class="py-2">
      <div v-html="htmlCode" />
    </div>

    <button @click="generateImage">Générer l'image</button>

    <div v-if="imageUrl">
      <img :src="imageUrl" alt="Generated Image" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import * as htmlToImage from "html-to-image";
// @ts-ignore
import html2pdf from "html2pdf.js";

const uploadOption = ref("file");
const htmlCode = ref("");
const imageUrl = ref("");
const outputFormat = ref("png");
const autoDownload = ref(false);

function handleFileUpload(event: any) {
  const input = event.target;
  const file = input.files?.[0];

  if (file) {
    const reader = new FileReader();

    reader.onload = () => {
      if (reader.result) {
        // @ts-ignore
        htmlCode.value = reader.result;
      }
    };

    reader.readAsText(file);
  }
}

async function generateImage() {
  const node = document.getElementById("html-content");
  if (outputFormat.value === "pdf") {
    await generatePdf(node);
  } else {
    await generateImageFromHtml(node);
  }
}

async function generatePdf(node: HTMLElement | null) {
  try {
    console.log(node);
    if (node) {
      html2pdf(node);

      if (autoDownload.value) {
        downloadPdf(html2pdf(node));
      }
    } else {
      throw new Error("Le noeud HTML n'a pas été trouvé");
    }
  } catch (error) {
    console.error(
      "Une erreur est survenue lors de la génération du pdf :",
      error
    );
  }
}

async function generateImageFromHtml(node: HTMLElement | null) {
  try {
    console.log(node);
    if (node) {
      const width = node.offsetWidth;
      const height = node.offsetHeight * 2;
      let dataUrl;
      if (outputFormat.value === "jpeg") {
        node.style.backgroundColor = "white";
        dataUrl = await htmlToImage.toJpeg(node, { width, height });
      } else {
        dataUrl = await htmlToImage.toPng(node, { width, height });
      }

      imageUrl.value = dataUrl;

      if (autoDownload.value) {
        downloadImage(dataUrl);
      }
    } else {
      throw new Error("Le noeud HTML n'a pas été trouvé");
    }
  } catch (error) {
    console.error(
      "Une erreur est survenue lors de la génération de l'image :",
      error
    );
  }
}

function downloadPdf(url: string) {
  const link = document.createElement("a");
  link.download = "generated.pdf";
  link.href = url;
  link.click();
}

function downloadImage(dataUrl: string) {
  const link = document.createElement("a");
  link.download = "generated-image.png";
  link.href = dataUrl;
  link.click();
}
</script>
