<template>
  <div class="max-w-xl mx-auto py-8 px-4">
    <h1 class="text-xl font-bold mb-4">Cartoon Or Not Classifier</h1>

    <div v-if="imageUrl" class="text-center">
      <img :src="imageUrl" alt="Selected Image" class="mx-auto w-1/2 max-w-lg rounded-lg mb-4" />
    </div>

    <div class="text-center">
      <div @dragover.prevent="dragOver" @dragleave.prevent="dragLeave" @drop.prevent="handleDrop"
        class="border-dashed rounded-lg border border-white/25 py-10 max-w-lg mx-auto mb-4">
        <label for="file-upload"
          class="cursor-pointer font-semibold text-blue-400 focus-within:outline-none focus-within:ring-2 focus-within:ring-blue-600 focus-within:ring-offset-2 hover:text-blue-500">
          <span>Upload a file</span>
          <input id="file-upload" name="file-upload" type="file" class="sr-only" @change="handleImageChange"
            accept="image/*" required />
        </label>
        <p class="text-xs leading-5 text-white mt-1">or drag and drop</p>
        <p v-if="selectedFileName" class="text-xs leading-5 text-white mt-1">{{ selectedFileName }}</p>
      </div>
    </div>

    <div v-if="isProcessing" class="text-center mb-4">
      <p>Loading...</p>
    </div>

    <div v-else-if="predictionData" class="text-center mb-4">
      <p>Prediction: {{ predictionData.prediction }}</p>
      <p>Accuracy: {{ predictionData.accuracy }}</p>
    </div>

    <button @click="submitImage"
      class="block mx-auto px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600">
      {{ isProcessing ? 'Processing...' : 'Submit' }}
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      imageUrl: null,
      predictionData: null,
      isProcessing: false,
      imageFile: null,
      selectedFileName: '',
    };
  },
  methods: {
    dragOver(e) {
      e.preventDefault();
    },
    dragLeave(e) {
      e.preventDefault();
    },
    handleDrop(e) {
      const file = e.dataTransfer.files[0];
      this.selectedFileName = file ? file.name : '';
      this.processFile(file);
    },
    handleImageChange(event) {
      const file = event.target.files[0];
      this.selectedFileName = file ? file.name : '';
      this.processFile(file);
    },
    processFile(file) {
      if (file && file.type.startsWith('image/')) {
        this.imageUrl = URL.createObjectURL(file);
        this.imageFile = file;
      } else {
        this.imageUrl = null;
        this.imageFile = null;
      }
    },
    submitImage() {
      if (this.imageFile) {
        this.isProcessing = true;
        const formData = new FormData();
        formData.append('input_image', this.imageFile);
        const authToken = import.meta.env.VITE_AUTH_TOKEN;
        fetch('https://cartoonornot-muf7kziviq-as.a.run.app/predict', {
          method: 'POST',
          body: formData,
          headers: {
            'Authorization': `Bearer ${authToken}`,
          },

        })
          .then(response => response.json())
          .then(data => {
            this.predictionData = data.data;
          })
          .catch(error => {
            console.error('Error:', error);
          })
          .finally(() => {
            this.isProcessing = false;
          });
      } else {
        console.error('No image selected');
      }
    }
  }
};
</script>
