<script setup lang="ts">
import { ref, onUnmounted } from 'vue'

const faceProfile = ref<File | null>(null)
const groupImage = ref<File | null>(null)

const loading = ref(false)
const errorMessage = ref('')
const successMessage = ref('')
const uploadResult = ref<{
  face_profile: { filename: string; path: string }
  group_image: { filename: string; path: string }
} | null>(null)

const handleFileChange = (
  event: Event,
  target: 'profile' | 'group'
) => {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0] ?? null

  errorMessage.value = ''
  successMessage.value = ''
  uploadResult.value = null

  if (file && !file.type.startsWith('image/')) {
    errorMessage.value = 'Please select an image file.'
    input.value = ''

    if (target === 'profile') {
      faceProfile.value = null
    } else {
      groupImage.value = null
    }

    return
  }

  if (target === 'profile') {
    faceProfile.value = file
  } else {
    groupImage.value = file
  }
}

const sendImages = async () => {
  errorMessage.value = ''
  successMessage.value = ''
  uploadResult.value = null

  if (!faceProfile.value || !groupImage.value) {
    errorMessage.value = 'Please select both images before submitting.'
    return
  }

  const formData = new FormData()

  // These field names must match the FastAPI parameters.
  formData.append('face_profile', faceProfile.value)
  formData.append('group_image', groupImage.value)

  loading.value = true

  try {
    const response = await fetch(
      'http://127.0.0.1:8000/process-images',
      {
        method: 'POST',
        body: formData,
      }
    )

    const data = await response.json()

    if (!response.ok) {
      throw new Error(data.detail || 'Image upload failed.')
    }

    uploadResult.value = data
    successMessage.value = 'Both images uploaded successfully.'

    console.log('Upload result:', data)
  } catch (error) {
    errorMessage.value =
      error instanceof Error
        ? error.message
        : 'An unexpected error occurred.'

    console.error('Image upload error:', error)
  } finally {
    loading.value = false
  }
}


const downloading = ref(false)
const downloadError = ref('')
const processedImageUrl = ref('')
const processedFilename = ref('')

const getResults = async () => {
  downloading.value = true
  downloadError.value = ''

  // Clear the previous image URL to avoid memory leaks.
  if (processedImageUrl.value) {
    URL.revokeObjectURL(processedImageUrl.value)
    processedImageUrl.value = ''
  }

  try {
    const response = await fetch('http://127.0.0.1:8000/download')

    if (!response.ok) {
      const errorData = await response.json().catch(() => null)
      throw new Error(
        errorData?.detail || 'Could not retrieve the processed photo.'
      )
    }

    const blob = await response.blob()

    if (!blob.type.startsWith('image/')) {
      throw new Error('The server did not return an image.')
    }

    processedImageUrl.value = URL.createObjectURL(blob)

    const disposition = response.headers.get('Content-Disposition')
    const filenameMatch = disposition?.match(/filename="?([^";]+)"?/i)

    processedFilename.value =
      filenameMatch?.[1] || 'processed-photo.jpg'
  } catch (error) {
    downloadError.value =
      error instanceof Error
        ? error.message
        : 'An unexpected error occurred.'

    console.error('Download error:', error)
  } finally {
    downloading.value = false
  }
}

const downloadPhoto = () => {
  if (!processedImageUrl.value) return

  const link = document.createElement('a')
  link.href = processedImageUrl.value
  link.download = processedFilename.value
  document.body.appendChild(link)
  link.click()
  link.remove()
}

onUnmounted(() => {
  if (processedImageUrl.value) {
    URL.revokeObjectURL(processedImageUrl.value)
  }
})

</script>

<template>
  <main class="blur-page">
    <h1>Welcome to Blur</h1>

    <p>
      Upload a reference face and an image containing multiple people.
    </p>

    <form @submit.prevent="sendImages" enctype="multipart/form-data">
      <section class="upload-section">
        <h2>1. Reference face</h2>
        <p>
          Choose an image containing exactly one face you want to keep clear.
        </p>

        <input
          type="file"
          name="face_profile"
          accept="image/*"
          required
          @change="handleFileChange($event, 'profile')"
        />

        <p v-if="faceProfile" class="file-name">
          Selected: {{ faceProfile.name }}
        </p>
      </section>

      <section class="upload-section">
        <h2>2. Group image</h2>
        <p>
          Choose an image containing multiple faces to process.
        </p>

        <input
          type="file"
          name="group_image"
          accept="image/*"
          required
          @change="handleFileChange($event, 'group')"
        />

        <p v-if="groupImage" class="file-name">
          Selected: {{ groupImage.name }}
        </p>
      </section>

      <button type="submit" :disabled="loading">
        {{ loading ? 'Uploading...' : 'Upload Both Images' }}
      </button>
    </form>

    <p v-if="errorMessage" class="error" role="alert">
      {{ errorMessage }}
    </p>

    <p v-if="successMessage" class="success" role="status">
      {{ successMessage }}
    </p>

    <section v-if="uploadResult" class="results">
      <h2>Upload results</h2>

      <p>
        <strong>Reference image:</strong>
        {{ uploadResult.face_profile.filename }}
      </p>

      <p>
        <strong>Group image:</strong>
        {{ uploadResult.group_image.filename }}
      </p>
    </section>


    <section class="download-section">
      <h2>3. Download Processed Photo</h2>

      <p>Retrieve the image after face recognition and blurring.</p>

      <button
        type="button"
        @click="getResults"
        :disabled="downloading"
      >
        {{ downloading ? 'Getting Photo...' : 'Get Processed Photo' }}
      </button>

      <p v-if="downloadError" class="error" role="alert">
        {{ downloadError }}
      </p>

      <div v-if="processedImageUrl" class="photo-result">
        <h3>Processed image</h3>

        <img
          :src="processedImageUrl"
          alt="Processed image with unrecognized faces blurred"
          class="processed-image"
        />

        <p>{{ processedFilename }}</p>

        <button type="button" @click="downloadPhoto">
          Download Photo
        </button>
      </div>
    </section>

  </main>
</template>

<style scoped>
.blur-page {
  max-width: 700px;
  margin: 40px auto;
  padding: 24px;
  font-family: Arial, sans-serif;
}

.upload-section {
  margin: 24px 0;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 10px;
}

.upload-section h2 {
  margin-top: 0;
}

input[type='file'] {
  display: block;
  width: 100%;
  margin-top: 12px;
}

button {
  padding: 12px 20px;
  border: none;
  border-radius: 6px;
  background: #501a10;
  color: white;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.file-name {
  overflow-wrap: anywhere;
  color: #555;
}

.error {
  color: #b00020;
}

.success {
  color: #16733a;
}

.results {
  margin-top: 24px;
  overflow-wrap: anywhere;
}
img.processed-image {
  max-width: 100%;
  height: auto;
  border: 1px solid #ddd;
  border-radius: 10px;
  margin-top: 12px;
}
</style>

