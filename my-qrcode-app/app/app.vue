<template>
  <div class="container">
    <h1>QR Code Generator + Logo</h1>

    <div class="input-group">
      <label>1. ใส่ URL เว็บไซต์:</label>
      <input v-model="url" type="text" placeholder="https://example.com" class="input-field" />
    </div>

    <div class="input-group">
      <label>2. อัปโหลด Logo (แนะนำไฟล์สี่เหลี่ยมจัตุรัส):</label>
      <input type="file" accept="image/*" @change="handleFileUpload" class="file-input" />
    </div>

    <button @click="generateQRCode" class="gen-btn">สร้าง QR Code</button>

    <hr />

    <div class="canvas-wrapper">
      <canvas ref="qrcodeCanvas"></canvas>
    </div>

    <p v-if="!url" class="hint">กรุณาใส่ URL เพื่อเริ่มใช้งาน</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import QRCode from 'qrcode'

// State
const url = ref<string>('https://gemini.google.com')
const logoFile = ref<File | null>(null)
const logoImage = ref<HTMLImageElement | null>(null)
const qrcodeCanvas = ref<HTMLCanvasElement | null>(null)

// ฟังก์ชันจัดการเมื่อมีการอัปโหลดไฟล์
const handleFileUpload = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (target.files && target.files[0]) {
    const file = target.files[0]
    const reader = new FileReader()

    reader.onload = (e) => {
      const img = new Image()
      img.src = e.target?.result as string
      img.onload = () => {
        logoImage.value = img
      }
    }
    reader.readAsDataURL(file)
  }
}

// ฟังก์ชันหลักในการสร้าง QR Code
const generateQRCode = async () => {
  if (!url.value || !qrcodeCanvas.value) return

  const canvas = qrcodeCanvas.value
  const ctx = canvas.getContext('2d')

  if (!ctx) return

  try {
    // 1. สร้าง QR Code ลงบน Canvas ก่อน
    // errorCorrectionLevel: 'H' (High) สำคัญมาก! เพื่อให้สแกนผ่านแม้มีโลโก้บัง
    await QRCode.toCanvas(canvas, url.value, {
      width: 300,
      margin: 2,
      errorCorrectionLevel: 'H',
      color: {
        dark: '#000000',
        light: '#ffffff'
      }
    })

    // 2. ถ้ามี Logo ให้วาดทับลงไปตรงกลาง
    if (logoImage.value) {
      const logoSize = 60 // กำหนดขนาด Logo (อย่าใหญ่เกิน 30% ของ QR)
      const x = (canvas.width - logoSize) / 2
      const y = (canvas.height - logoSize) / 2

      // (Optional) วาดพื้นหลังสีขาวรองหลัง Logo เพื่อความชัดเจน
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(x - 5, y - 5, logoSize + 10, logoSize + 10)

      // วาดรูป Logo
      ctx.drawImage(logoImage.value, x, y, logoSize, logoSize)
    }

  } catch (err) {
    console.error('Error generating QR code', err)
  }
}
</script>

<style scoped>
.container {
  max-width: 500px;
  margin: 50px auto;
  padding: 20px;
  font-family: sans-serif;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 15px;
  text-align: left;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.input-field {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}

.file-input {
  margin-top: 5px;
}

.gen-btn {
  background-color: #00dc82;
  /* Nuxt Green */
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 4px;
  margin-bottom: 20px;
}

.gen-btn:hover {
  background-color: #00c16e;
}

.canvas-wrapper {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.hint {
  color: #888;
  font-size: 0.9em;
}
</style>