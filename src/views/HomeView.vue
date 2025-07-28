<template>
  <div
    class="relative min-h-screen bg-white rounded-lg shadow-xl overflow-hidden p-6 md:p-10 lg:p-12 max-w-4xl mx-auto my-8"
  >
    <div class="absolute top-0 left-0 w-full h-40 bg-blue-300 z-0"></div>
    <div
      class="absolute top-32 left-0 w-full h-20 bg-green-400 rounded-b-full z-0 transform scale-x-150"
    ></div>

    <img
      src="/src/assets/monkey.png"
      alt="Monkey"
      class="absolute top-8 left-4 w-16 md:w-20 lg:w-24 z-10"
    />
    <img
      src="/src/assets/monkey.png"
      alt="Monkey"
      class="absolute top-24 left-16 w-12 md:w-16 lg:w-20 z-10 transform scale-x-[-1]"
    />
    <img
      src="/src/assets/radish.png"
      alt="Radish"
      class="absolute bottom-4 left-1/2 -translate-x-1/2 w-12 md:w-16 lg:w-20 z-10"
    />
    <img
      src="/src/assets/sun.png"
      alt="Sun"
      class="absolute top-4 right-16 w-12 md:w-16 lg:w-20 z-10"
    />
    <img
      src="/src/assets/vegetable_leaf.png"
      alt="Leaf"
      class="absolute top-24 right-4 w-12 md:w-16 lg:w-20 z-10"
    />

    <div class="relative z-10 flex flex-col items-center justify-center h-full">
      <div
        class="flex items-center mb-8 bg-purple-600 bg-opacity-80 p-3 rounded-full shadow-lg"
      >
        <img
          src="/src/assets/owl.png"
          alt="Owl Logo"
          class="w-12 h-12 mr-3 rounded-full ring-2 ring-white object-contain"
        />
        <h1
          class="text-4xl md:text-5xl font-extrabold text-white text-shadow-lg"
        >
          Little Learners Club
        </h1>
      </div>

      <div
        @dragover.prevent="handleDragOver"
        @dragleave.prevent="handleDragLeave"
        @drop.prevent="handleDrop"
        :class="[
          'border-4 border-dashed rounded-2xl p-8 md:p-12 lg:p-16 text-center transition-all duration-300 ease-in-out',
          isDragging
            ? 'border-blue-500 bg-blue-50'
            : 'border-gray-300 bg-gray-50',
        ]"
      >
        <p class="text-xl md:text-2xl font-bold text-gray-700 mb-6">
          Upload Your Animal Photo
        </p>

        <div
          class="flex flex-col md:flex-row items-center justify-center gap-6 mb-8"
        >
          <img src="/src/assets/lion.png" alt="Lion" class="w-32 md:w-40" />
        </div>

        <p class="text-lg md:text-xl text-gray-500 mb-8">
          Drag and Drop Your Image Here
        </p>

        <input
          type="file"
          ref="fileInput"
          @change="handleFileInput"
          accept="image/*"
          class="hidden"
        />
        <button
          @click="openFileInput"
          :disabled="isLoading"
          class="bg-orange-500 hover:bg-orange-600 text-white font-bold py-3 px-8 rounded-full shadow-lg transform hover:scale-105 transition-all duration-200 ease-in-out text-lg"
        >
          {{ isLoading ? "Processing..." : "Choose Image" }}
        </button>
      </div>

      <div v-if="uploadedImage" class="mt-8">
        <h2 class="text-2xl font-bold text-gray-800 mb-4">Uploaded Image:</h2>
        <img
          :src="uploadedImage"
          alt="Uploaded"
          class="max-w-xs h-auto rounded-lg shadow-md"
        />
      </div>

      <div
        v-if="errorMessage"
        class="mt-8 p-4 bg-red-100 border border-red-400 text-red-700 rounded-lg text-center"
      >
        <p class="font-bold">Error:</p>
        <p>{{ errorMessage }}</p>
      </div>

      <button
        v-if="uploadedImage || errorMessage"
        @click="resetForm"
        class="mt-6 bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-6 rounded-full transition-all duration-200 ease-in-out"
      >
        Upload Gambar Lain
      </button>
    </div>

    <div
      v-if="showResultPopup"
      class="fixed inset-0 z-50 flex items-center justify-center bg-gray-200 bg-opacity-50 p-4"
    >
      <div
        class="bg-white p-8 md:p-10 rounded-lg shadow-2xl text-center max-w-lg w-full transform transition-all duration-300 ease-out scale-100 opacity-100"
      >
        <h2 class="text-4xl font-extrabold text-blue-800 mb-6">
          Hasil Klasifikasi!
        </h2>

        <div v-if="uploadedImage" class="mb-6 flex justify-center">
          <img
            :src="uploadedImage"
            alt="Uploaded for Result"
            class="max-h-48 rounded-lg shadow-md object-contain"
          />
        </div>

        <p class="text-6xl font-extrabold text-green-700 mb-4">
          {{ classificationResult }}
        </p>
        <p v-if="confidence" class="text-xl text-gray-600 mb-8">
          Keyakinan:
          <span class="font-semibold"
            >{{ (confidence * 100).toFixed(2) }}%</span
          >
        </p>

        <button
          @click="resetAndClosePopup"
          class="bg-orange-500 hover:bg-orange-600 text-white font-bold py-3 px-8 rounded-full shadow-lg transform hover:scale-105 transition-all duration-200 ease-in-out text-lg"
        >
          Upload Gambar Lain
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

// --- State Variables ---
const fileInput = ref(null);
const uploadedImage = ref(null);
const isDragging = ref(false);
const isLoading = ref(false);
const classificationResult = ref(null);
const confidence = ref(null);
const errorMessage = ref(null);
const showResultPopup = ref(false); // New: Untuk mengontrol visibilitas pop-up

// --- Constants ---
const API_URL = "https://82a34c8ec55f.ngrok-free.app/predict"; // MODIFIKASI INI: SESUAIKAN DENGAN URL BACKEND FLASK ANDA

// --- Functions ---
const openFileInput = () => {
  fileInput.value.click();
};

const handleFileInput = (event) => {
  const file = event.target.files[0];
  if (file) {
    processImage(file);
  }
};

const handleDragOver = () => {
  isDragging.value = true;
};

const handleDragLeave = () => {
  isDragging.value = false;
};

const handleDrop = (event) => {
  isDragging.value = false;
  const file = event.dataTransfer.files[0];
  if (file) {
    processImage(file);
  }
};

const processImage = (file) => {
  resetAllStates(); // Reset semua state termasuk pop-up
  uploadedImage.value = null; // Reset gambar yang ditampilkan sementara

  if (file.type.startsWith("image/")) {
    const reader = new FileReader();
    reader.onload = (e) => {
      uploadedImage.value = e.target.result; // Tampilkan gambar di frontend
      classifyImage(file); // Kirim gambar ke backend
    };
    reader.readAsDataURL(file);
  } else {
    errorMessage.value = "Mohon unggah file gambar (misal: .jpg, .png).";
    uploadedImage.value = null;
  }
};

const classifyImage = async (file) => {
  isLoading.value = true;
  errorMessage.value = null;
  classificationResult.value = null;
  confidence.value = null;
  showResultPopup.value = false; // Pastikan pop-up tertutup saat proses baru dimulai

  const formData = new FormData();
  formData.append("file", file);

  try {
    const response = await fetch(API_URL, {
      method: "POST",
      body: formData,
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(
        errorData.error || `HTTP error! status: ${response.status}`
      );
    }

    const data = await response.json();
    console.log("Classification result from backend:", data);

    let resultText = "Bukan Anjing, Kucing, atau Ular";
    if (data.predicted_class) {
      const predictedClass = data.predicted_class.toLowerCase();
      if (predictedClass === "dogs") {
        resultText = "Ini Anjing!";
      } else if (predictedClass === "cats") {
        resultText = "Ini Kucing!";
      } else if (predictedClass === "snakes") {
        resultText = "Ini Ular!";
      } else {
        resultText = `Ini ${predictedClass} (Bukan Anjing, Kucing, atau Ular)`;
      }
    }

    classificationResult.value = resultText;
    confidence.value = data.confidence;
    showResultPopup.value = true; // Tampilkan pop-up setelah hasil didapat
  } catch (error) {
    console.error("Error classifying image:", error);
    errorMessage.value = `Gagal mengklasifikasi gambar: ${error.message}. Pastikan backend berjalan dengan benar.`;
  } finally {
    isLoading.value = false;
  }
};

// --- Reset Functions ---
const resetAllStates = () => {
  classificationResult.value = null;
  confidence.value = null;
  errorMessage.value = null;
  showResultPopup.value = false; // Pastikan pop-up tertutup
};

const resetForm = () => {
  uploadedImage.value = null;
  resetAllStates(); // Reset semua state
  if (fileInput.value) {
    fileInput.value.value = ""; // Mengatur ulang input file
  }
};

const resetAndClosePopup = () => {
  resetForm(); // Reset semua form dan state
  showResultPopup.value = false; // Pastikan pop-up tertutup
};
</script>

<style scoped>
.text-shadow-lg {
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}
</style>
