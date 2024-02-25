<template>
    <div>
      <h1 class="text-xl font-bold mb-4">Cartoon Or Not Classifier</h1>
      
      <div v-if="imageUrl" class="items-center">
        <img :src="imageUrl" alt="Selected Image" class="w-1/2 mx-auto p-4" />
      </div>
      
      <div class="relative">
        <input
          @change="handleImageChange"
          class="pl-10 pr-4 py-2 w-full sm:w-1/2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:border-blue-300"
          type="file"
          accept="image/*"
          placeholder="Select an image..."
          style="min-width: 400px;"
          required
        />
      </div>
  
      <div v-if="isProcessing" class="py-4">
        <p>Loading...</p>
        <!-- You can replace this with any loading indicator or spinner -->
      </div>
      
      <div v-else-if="predictionData" class="py-4">
        <p>Prediction: {{ predictionData.prediction }}</p>
        <p>Accuracy: {{ predictionData.accuracy }}</p>
      </div>
      
      <button @click="submitImage" class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600">
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
        isProcessing: false // Initially set to false
      };
    },
    methods: {
      handleImageChange(event) {
        const file = event.target.files[0];
        if (file) {
          this.imageUrl = URL.createObjectURL(file);
        } else {
          this.imageUrl = null;
        }
      },
      submitImage() {
        if (this.imageUrl) {
          this.isProcessing = true; // Set isProcessing to true when submitting
          const fileInput = document.querySelector('input[type="file"]');
          const file = fileInput.files[0];  // Get the file from the file input
          const formData = new FormData();
          formData.append('input_image', file);  // Append the file directly
          fetch('http://localhost:5000/predict', {
            method: 'POST',
            body: formData
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
  