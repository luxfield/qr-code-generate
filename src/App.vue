<template>
  <el-space wrap direction="horizontal" alignment="baseline" spacer=" ">
    <el-card class="box-card">
      <template #header>
        <div class="card-header">
          <span>Pengaturan</span>
        </div>
      </template>
      <el-form label-width="auto" label="right">
        <el-form-item label="Tautan atau teks dalam QR">
          <el-tooltip effect="dark" content="Your link or text" placement="top-start">
            <el-input v-model="message" clearable :placeholder="message" />
          </el-tooltip>
        </el-form-item>
        <el-form-item label="Warna QR">
          <el-color-picker v-model="color"></el-color-picker>
        </el-form-item>
        <el-form-item label="Warna latar belakang QR">
          <el-color-picker v-model="backgroundColor" />
        </el-form-item>
        <!-- <el-form-item>
          <el-checkbox v-model="inverseLogoColor" label="Inverse logo colors" size="large" border />
        </el-form-item> -->
        <el-form-item>
          <el-checkbox v-model="padding" label="tambahkan jarak luar" size="large" border />
        </el-form-item>

        <el-form-item label="Ukuran Ekspor (dalam px)">
          <el-input-number v-model="exportSize" :step="50" :min="100" :max="10000" />
        </el-form-item>
        <el-form-item label="Simpan sebagai">
          <el-radio-group v-model="exportType" size="small">
            <el-radio-button label="SVG" />
            <el-radio-button label="PNG" />
            <el-radio-button label="JPG" />
            <el-radio-button label="WEBP" />
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="large" :loading="loading" @click="saveToFile">Simpan<el-icon
              class="el-icon--right">
              <Download />
            </el-icon></el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <el-card class="box-card" body-style="
        background: repeating-linear-gradient(
          -45deg,
          #eee,
          #eee 10px,
          #ccc 10px,
          #ccc 20px
        );
      ">
      <template #header>
        <div class="card-header">
          <span>Pratinjau</span>
        </div>
      </template>
      <div v-loading="loading">
        <svg :width="size" :height="size" id="svgcontainer" ref="svgcontainer">

          <svg :width="size" :height="size" version="1.1" xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink">
            <svg v-html="preview"></svg>
            <rect fill="white" stroke-width="6px" :stroke="[this.color === null ? 'black' : this.color]" width="125"
              height="80" :x="size / 2 - (size * logoScale) / 2" :y="size / 2 - (size * logoScale) / 2 + 10" />
            <svg :x="size / 2 - (size * logoScale) / 2" :y="size / 2 - (size * logoScale) / 2 + 8" width="125"
              height="100" viewBox="0 0 24 24" fill="red">
              <!-- logo ubah dulu gambar dalam bentuk base64 agar bisa tersimpan dengan qrcode nya -->
              <image
                href="data:image/png;base64,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"
                height="20" width="25" x="1"></image>
              <!-- logo -->
            </svg>
          </svg>
        </svg>
      </div>
    </el-card>
  </el-space>
</template>

<script>
import {
  ElForm,
  ElFormItem,
  ElCard,
  ElSpace,
  ElInput,
  ElInputNumber,
  ElColorPicker,
  ElCheckbox,
  ElRadioGroup,
  ElRadioButton,
  ElButton,
  ElIcon,
  ElTooltip,
  ElLoading,
} from "element-plus";

import { Download } from "@element-plus/icons-vue";
import { saveAs } from "file-saver";
import QRCode from "qrcode-svg";

export default {
  name: "App",
  components: {
    ElForm,
    ElFormItem,
    ElCard,
    ElSpace,
    ElInput,
    ElInputNumber,
    ElColorPicker,
    ElCheckbox,
    ElRadioGroup,
    ElRadioButton,
    ElButton,
    ElIcon,
    Download,
    ElTooltip,
  },
  directives: {
    loading: ElLoading.directive,
  },
  data() {
    return {
      message: "insert url or text here",
      logoScale: 0.33,
      size: 350,
      color: "#000",
      backgroundColor: "#fff",
      padding: false,
      inverseLogoColor: false,
      exportType: "PNG",
      exportSize: 1200,
      loading: false,
    };
  },
  methods: {
    export() {
      const exportType = this.exportType;
      const fileName = this.fileName;

      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      const imgSize = this.size;
      const tempImg = new Image();

      tempImg.onload = function () {
        tempImg.width = imgSize;
        tempImg.height = imgSize;
        canvas.width = imgSize;
        canvas.height = imgSize;
        ctx.drawImage(tempImg, 0, 0, tempImg.width, tempImg.height);

        switch (exportType) {
          case "PNG":
            canvas.toBlob(
              function (blob) {
                saveAs(blob, `${fileName}.png`);
              },
              "image/png",
              1
            );
            break;

          case "JPG":
            canvas.toBlob(
              function (blob) {
                saveAs(blob, `${fileName}.jpg`);
              },
              "image/jpeg",
              1
            );
            break;
          case "WEBP":
            canvas.toBlob(
              function (blob) {
                saveAs(blob, `${fileName}.webp`);
              },
              "image/webp",
              1
            );
            break;
        }
      };

      tempImg.src =
        "data:image/svg+xml;charset=utf-8," +
        encodeURIComponent(this.$refs.svgcontainer.innerHTML);
    },
    saveToFile() {
      this.loading = true;
      [this.size, this.exportSize] = [this.exportSize, this.size];

      if (this.exportType === "SVG") {
        var blob = new Blob([this.$refs.svgcontainer.innerHTML], {
          type: "image/svg+xml;charset=utf-8",
        });
        saveAs(blob, `${this.fileName}.svg`);
      } else {
        this.export();
      }

      [this.size, this.exportSize] = [this.exportSize, this.size];
      setTimeout(() => (this.loading = false), 1500);
    },
  },
  computed: {
    logoColor() {
      return this.inverseLogoColor ? this.backgroundColor : this.color;
    },
    logoBackgroundColor() {
      return this.inverseLogoColor ? this.color : this.backgroundColor;
    },
    fileName() {
      try {
        const url = new URL(this.content);
        return `QR-for-${url.hostname}`;
      } catch (error) {
        const dashMessage = this.message.replace(/ /g, "-");
        return `QR-for-${dashMessage}`;

      }
    },
    content() {
      return this.message;
    },
    preview() {
      var svg = new QRCode({
        content: this.content,
        padding: this.padding ? 1 : 0,
        width: this.size,
        height: this.size,
        color: this.color,
        background: this.backgroundColor,
        ecl: "Q",
        xmlDeclaration: true,
        container: "none",
        join: true,
        pretty: false,
      });
      return svg.svg();
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
}

.card-header {
  text-align: left;
}

.box-card {
  max-width: 480px;
  margin: 0 auto;
}

.el-space .el-space__item {
  margin: 8px auto !important;
}
</style>
