<template>
  <div>
    <h1 class="text-xl font-bold mb-4">Cartoon Or Not Classifier</h1>

    <div v-if="imageUrl" class="items-center">
      <img :src="imageUrl" alt="Selected Image" class="w-1/2 mx-auto p-4 rounded-lg" />
    </div>

    <div class="col-span-full">
      <div @dragover.prevent="dragOver" @dragleave.prevent="dragLeave" @drop.prevent="handleDrop"
        class="mt-2 flex justify-center rounded-lg border border-dashed border-white/25 py-10 max-w-xl mx-auto">
        <div class="text-center">
          <div class="mx-auto w-12 text-white" aria-hidden="true"></div>
          <div class="text-sm leading-6 text-white">
            <label for="file-upload"
              class="relative cursor-pointer rounded-md font-semibold text-blue-400 focus-within:outline-none focus-within:ring-2 focus-within:ring-blue-600 focus-within:ring-offset-2 hover:text-blue-500">
              <span>Upload a file</span>
              <input id="file-upload" name="file-upload" type="file" class="sr-only" @change="handleImageChange"
                accept="image/*" required style="min-width: 400px;" />
            </label>
            <p class="pl-1">or drag and drop</p>
          </div>
          <p class="text-xs leading-5 text-white">PNG, JPG, GIF up to 10MB</p>
          <br>
          <p v-if="selectedFileName" class="text-xs leading-5 text-white">{{ selectedFileName }}</p>
        </div>
      </div>
    </div>

    <div v-if="isProcessing" class="py-4">
      <p>Loading...</p>
      <!-- You can replace this with any loading indicator or spinner -->
    </div>

    <div v-else-if="predictionData" class="py-4">
      <p>Prediction: {{ predictionData.prediction }}</p>
      <p>Accuracy: {{ predictionData.accuracy }}</p>
    </div>

    <button @click="submitImage"
      class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600">
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
      selectedFileName: '', // To store the selected file name
    };
  },
  methods: {
    dragOver(e) {
      e.preventDefault();
      console.log(process.env.VUE_APP_TEST);
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
        this.imageFile = file; // Store the file in your data model
      } else {
        this.imageUrl = null;
        this.imageFile = null; // Clear the stored file if not an image
      }
    },
    submitImage() {
      if (this.imageFile) {
        this.isProcessing = true; // Set isProcessing to true when submitting
        const formData = new FormData();
        formData.append('input_image', this.imageFile);
        const authToken = process.env.VUE_APP_AUTH_TOKEN;
        fetch('https://cartoonornot-muf7kziviq-as.a.run.app/predict', {
          method: 'POST',
          body: formData,
          headers: {
            'Authorization': `Bearer ${authToken}`, // Replace 'your_token_here' with your actual token
          },

        })
          .then(response => response.json())
          .then(data => {
            console.log(data);
            this.predictionData = data.data; // Update predictionData with response data
          })
          .catch(error => {
            console.error('Error:', error);
            // Handle error here
          })
          .finally(() => {
            this.isProcessing = false; // Reset isProcessing to false after processing
          });
      } else {
        console.error('No image selected');
        // Handle no image selected error here
      }
    }
  }
};
</script>
  
<style>
/* You can add custom styles specific to this component here */
</style>
  