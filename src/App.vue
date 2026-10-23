
<script setup lang="ts">
import { ref } from 'vue'

const faceProfile = ref<File[]>([])
const upload = ref<File[]>([])

const SendFaceProfile = async () => {
  if (!faceProfile.value.length) {
    alert('Please select a face profile image')
    return
  }

  const formData = new FormData()
  formData.append('file', faceProfile.value[0]!)

  try {
    const response = await fetch('http://127.0.0.1:8000/faceProfile', {
      method: 'POST',
      body: formData,
    })

    if (!response.ok) {
      throw new Error(`Upload failed: ${response.status}`)
    }

    const data = await response.json()
    console.log(data)
  } catch (error) {
    console.error('Face profile upload error:', error)
  }
}

const SendFilesToBlur = async () => {
  if (!upload.value.length) {
    alert('Please select a file to blur')
    return
  }

  const formData = new FormData()
  formData.append('file', upload.value[0]!)

  try {
    const response = await fetch('http://127.0.0.1:8000/upload', {
      method: 'POST',
      body: formData,
    })

    if (!response.ok) {
      throw new Error(`Upload failed: ${response.status}`)
    }

    const data = await response.json()
    console.log(data)
  } catch (error) {
    console.error('File upload error:', error)
  }
}

const getResults = async () => {
  try {
    const response = await fetch('http://127.0.0.1:8000/download', {
      method: 'GET',
    })
    const data = await response.json()
    console.log(data)
  } catch (error) {
    console.error('Error fetching results:', error)
  }
}

</script>

<template>
  <h1>Welcome to Blur</h1>

  <h2>Input the faces you do not want to blur</h2>

  <form @submit.prevent="SendFaceProfile" enctype="multipart/form-data">
    <input
      type="file"
      name="faceProfile"
      accept="image/*"
      @change="faceProfile = Array.from(($event.target as HTMLInputElement).files ?? [])"
    />
    <button type="submit">Submit Face Profile</button>
  </form>

  <h2>Upload files to blur</h2>

  <form @submit.prevent="SendFilesToBlur" enctype="multipart/form-data">
    <input
      type="file"
      name="upload"
      accept="image/*,video/*"
      @change="upload = Array.from(($event.target as HTMLInputElement).files ?? [])"
    />
    <button type="submit">Upload Files</button>
  </form>

  <h2>Output</h2>
  <button @click="getResults">Get Results</button>
  <p>The blurred files will be displayed here.</p>

</template>
