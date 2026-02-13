
# Complete Guide to Blobs in JavaScript

## Table of Contents
1. [What is a Blob?](#What is a Blob)
2. [Creating Blobs](#creating-blobs)
3. [Blob Properties](#blob-properties)
4. [Blob Methods](#blob-methods)
5. [Reading Blobs](#reading-blobs)
6. [Blob URLs](#blob-urls)
7. [Practical Use Cases](#practical-use-cases)
8. [File vs Blob](#file-vs-blob)
9. [Advanced Techniques](#advanced-techniques)

---

## What is a Blob?

**Blob** stands for **Binary Large Object**. It's a JavaScript object that represents immutable raw binary data. Think of it as a file-like container for binary data.

### Key Characteristics:
- Immutable (cannot be changed once created)
- Can represent data that isn't necessarily in JavaScript-native format
- Can store images, audio, video, or any binary data
- Has a size and MIME type

```javascript
// Basic Blob structure
const blob = new Blob(['Hello World'], { type: 'text/plain' });
console.log(blob.size); // 11
console.log(blob.type); // 'text/plain'
```

---

## Creating Blobs

### 1. **Basic Constructor**

```javascript
new Blob(array, options)
```

**Parameters:**
- `array`: An array of ArrayBuffer, ArrayBufferView, Blob, or String objects
- `options`: Optional object with properties:
  - `type`: MIME type (default: '')
  - `endings`: How to handle newlines ('transparent' or 'native')

### 2. **Creating from Text**

```javascript
// Simple text blob
const textBlob = new Blob(['Hello, World!'], { type: 'text/plain' });

// Multiple parts concatenated
const multiPartBlob = new Blob(
  ['Hello', ' ', 'World', '!'], 
  { type: 'text/plain' }
);

// With HTML
const htmlBlob = new Blob(
  ['<html><body><h1>Hello</h1></body></html>'], 
  { type: 'text/html' }
);
```

### 3. **Creating from JSON**

```javascript
const data = { name: 'John', age: 30 };
const jsonBlob = new Blob(
  [JSON.stringify(data)], 
  { type: 'application/json' }
);
```

### 4. **Creating from ArrayBuffer**

```javascript
const buffer = new ArrayBuffer(8);
const view = new Uint8Array(buffer);
view[0] = 72; // 'H'
view[1] = 101; // 'e'
view[2] = 108; // 'l'
view[3] = 108; // 'l'
view[4] = 111; // 'o'

const binaryBlob = new Blob([buffer], { type: 'application/octet-stream' });
```

### 5. **Creating from Canvas**

```javascript
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');

// Draw something
ctx.fillStyle = 'red';
ctx.fillRect(10, 10, 100, 100);

// Convert to blob
canvas.toBlob((blob) => {
  console.log('Canvas blob created:', blob);
}, 'image/png');
```

---

## Blob Properties

### 1. **size**
Returns the size of the blob in bytes.

```javascript
const blob = new Blob(['Hello World']);
console.log(blob.size); // 11
```

### 2. **type**
Returns the MIME type of the blob.

```javascript
const blob = new Blob(['Hello'], { type: 'text/plain' });
console.log(blob.type); // 'text/plain'
```

---

## Blob Methods

### 1. **slice()**
Creates a new blob containing a subset of the original blob's data.

```javascript
blob.slice(start, end, contentType)
```

```javascript
const originalBlob = new Blob(['Hello World'], { type: 'text/plain' });

// Get first 5 bytes
const slicedBlob = originalBlob.slice(0, 5);
console.log(slicedBlob.size); // 5

// Read the sliced content
slicedBlob.text().then(text => {
  console.log(text); // 'Hello'
});

// Change content type while slicing
const newTypeBlob = originalBlob.slice(0, 5, 'text/html');
console.log(newTypeBlob.type); // 'text/html'
```

### 2. **text()** (Promise-based)
Returns blob content as text.

```javascript
const blob = new Blob(['Hello World']);

blob.text().then(text => {
  console.log(text); // 'Hello World'
});

// Or with async/await
async function readBlobText() {
  const text = await blob.text();
  console.log(text);
}
```

### 3. **arrayBuffer()** (Promise-based)
Returns blob content as ArrayBuffer.

```javascript
const blob = new Blob(['Hello']);

blob.arrayBuffer().then(buffer => {
  console.log(buffer); // ArrayBuffer(5)
  const view = new Uint8Array(buffer);
  console.log(view); // Uint8Array(5) [72, 101, 108, 108, 111]
});
```

### 4. **stream()**
Returns a ReadableStream to read blob content.

```javascript
const blob = new Blob(['Hello World']);
const stream = blob.stream();

const reader = stream.getReader();

reader.read().then(({ done, value }) => {
  if (!done) {
    console.log(value); // Uint8Array
  }
});

// Better example - reading entire stream
async function readStream(blob) {
  const stream = blob.stream();
  const reader = stream.getReader();
  const chunks = [];
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    chunks.push(value);
  }
  
  return chunks;
}
```

---

## Reading Blobs

### Using FileReader (Traditional Method)

The `FileReader` API is the classic way to read blob content.

#### 1. **Reading as Text**

```javascript
const blob = new Blob(['Hello World'], { type: 'text/plain' });
const reader = new FileReader();

reader.onload = function(event) {
  console.log(event.target.result); // 'Hello World'
};

reader.onerror = function(event) {
  console.error('Error reading blob:', event);
};

reader.readAsText(blob);
```

#### 2. **Reading as Data URL**

```javascript
const blob = new Blob(['Hello World'], { type: 'text/plain' });
const reader = new FileReader();

reader.onload = function(event) {
  console.log(event.target.result); 
  // data:text/plain;base64,SGVsbG8gV29ybGQ=
};

reader.readAsDataURL(blob);
```

#### 3. **Reading as ArrayBuffer**

```javascript
const blob = new Blob(['Hello'], { type: 'text/plain' });
const reader = new FileReader();

reader.onload = function(event) {
  const buffer = event.target.result;
  console.log(new Uint8Array(buffer));
  // Uint8Array(5) [72, 101, 108, 108, 111]
};

reader.readAsArrayBuffer(blob);
```

#### 4. **Reading as Binary String**

```javascript
const blob = new Blob(['Hello'], { type: 'text/plain' });
const reader = new FileReader();

reader.onload = function(event) {
  console.log(event.target.result);
  // Binary string representation
};

reader.readAsBinaryString(blob);
```

### Modern Async Methods

```javascript
// Much cleaner with async/await
async function readBlob(blob) {
  try {
    // As text
    const text = await blob.text();
    console.log('Text:', text);
    
    // As array buffer
    const buffer = await blob.arrayBuffer();
    console.log('Buffer:', buffer);
    
    // As stream
    const stream = blob.stream();
    console.log('Stream:', stream);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

---

## Blob URLs

Blob URLs (Object URLs) are temporary URLs that reference blob data.

### 1. **Creating Blob URLs**

```javascript
const blob = new Blob(['Hello World'], { type: 'text/plain' });
const blobURL = URL.createObjectURL(blob);

console.log(blobURL); 
// blob:http://localhost:3000/5c5e5e5e-5e5e-5e5e-5e5e-5e5e5e5e5e5e
```

### 2. **Using Blob URLs**

```javascript
// For images
const imageBlob = new Blob([imageData], { type: 'image/png' });
const imageURL = URL.createObjectURL(imageBlob);

const img = document.createElement('img');
img.src = imageURL;
document.body.appendChild(img);

// For downloads
const downloadBlob = new Blob(['Download this!'], { type: 'text/plain' });
const downloadURL = URL.createObjectURL(downloadBlob);

const a = document.createElement('a');
a.href = downloadURL;
a.download = 'file.txt';
a.click();

// For video/audio
const videoBlob = new Blob([videoData], { type: 'video/mp4' });
const videoURL = URL.createObjectURL(videoBlob);

const video = document.createElement('video');
video.src = videoURL;
video.controls = true;
document.body.appendChild(video);
```

### 3. **Revoking Blob URLs**

**Important:** Always revoke blob URLs to free memory!

```javascript
const blob = new Blob(['Hello'], { type: 'text/plain' });
const url = URL.createObjectURL(blob);

// Use the URL...

// Clean up when done
URL.revokeObjectURL(url);

// Better pattern with image loading
const img = document.createElement('img');
img.onload = function() {
  URL.revokeObjectURL(this.src); // Free memory after load
};
img.src = URL.createObjectURL(blob);
```

---

## Practical Use Cases

### 1. **Downloading Files**

```javascript
function downloadFile(content, filename, contentType) {
  const blob = new Blob([content], { type: contentType });
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  
  URL.revokeObjectURL(url);
}

// Usage
downloadFile('Hello World', 'hello.txt', 'text/plain');
downloadFile(
  JSON.stringify({ name: 'John' }, null, 2), 
  'data.json', 
  'application/json'
);
```

### 2. **Image Preview Before Upload**

```javascript
function previewImage(input) {
  const file = input.files[0];
  
  if (file) {
    const url = URL.createObjectURL(file);
    
    const img = document.getElementById('preview');
    img.src = url;
    
    // Clean up when image loads
    img.onload = function() {
      URL.revokeObjectURL(url);
    };
  }
}

// HTML: <input type="file" onchange="previewImage(this)" accept="image/*">
// HTML: <img id="preview">
```

### 3. **Canvas to Blob to Download**

```javascript
const canvas = document.getElementById('myCanvas');

// Draw something on canvas...
const ctx = canvas.getContext('2d');
ctx.fillStyle = 'blue';
ctx.fillRect(0, 0, 200, 200);

// Convert to blob and download
canvas.toBlob(function(blob) {
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = 'canvas-image.png';
  a.click();
  
  URL.revokeObjectURL(url);
}, 'image/png');
```

### 4. **Fetch API with Blobs**

```javascript
// Download image as blob
async function downloadImage(imageUrl) {
  try {
    const response = await fetch(imageUrl);
    const blob = await response.blob();
    
    console.log('Blob size:', blob.size);
    console.log('Blob type:', blob.type);
    
    // Create image element
    const url = URL.createObjectURL(blob);
    const img = document.createElement('img');
    img.src = url;
    img.onload = () => URL.revokeObjectURL(url);
    document.body.appendChild(img);
    
    return blob;
  } catch (error) {
    console.error('Error:', error);
  }
}

// Upload blob
async function uploadBlob(blob, uploadUrl) {
  const formData = new FormData();
  formData.append('file', blob, 'filename.png');
  
  const response = await fetch(uploadUrl, {
    method: 'POST',
    body: formData
  });
  
  return response.json();
}
```

### 5. **Recording Audio/Video**

```javascript
let mediaRecorder;
let chunks = [];

async function startRecording() {
  const stream = await navigator.mediaDevices.getUserMedia({ 
    audio: true, 
    video: true 
  });
  
  mediaRecorder = new MediaRecorder(stream);
  
  mediaRecorder.ondataavailable = (event) => {
    if (event.data.size > 0) {
      chunks.push(event.data);
    }
  };
  
  mediaRecorder.onstop = () => {
    const blob = new Blob(chunks, { type: 'video/webm' });
    chunks = [];
    
    // Create video element
    const url = URL.createObjectURL(blob);
    const video = document.createElement('video');
    video.src = url;
    video.controls = true;
    document.body.appendChild(video);
    
    // Or download it
    const a = document.createElement('a');
    a.href = url;
    a.download = 'recording.webm';
    a.click();
    
    URL.revokeObjectURL(url);
  };
  
  mediaRecorder.start();
}

function stopRecording() {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop();
  }
}
```

### 6. **Chunked File Upload**

```javascript
async function uploadLargeFile(file, chunkSize = 1024 * 1024) {
  const totalChunks = Math.ceil(file.size / chunkSize);
  
  for (let i = 0; i < totalChunks; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);
    
    const formData = new FormData();
    formData.append('chunk', chunk);
    formData.append('chunkIndex', i);
    formData.append('totalChunks', totalChunks);
    formData.append('filename', file.name);
    
    await fetch('/upload-chunk', {
      method: 'POST',
      body: formData
    });
    
    console.log(`Uploaded chunk ${i + 1}/${totalChunks}`);
  }
}
```

### 7. **Creating PDF from HTML**

```javascript
async function createPDF() {
  // Using a library like jsPDF
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  
  doc.text('Hello World!', 10, 10);
  
  // Get blob
  const blob = doc.output('blob');
  
  // Download
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'document.pdf';
  a.click();
  
  URL.revokeObjectURL(url);
}
```

---

## File vs Blob

`File` is a subclass of `Blob` with additional properties.

### Differences:

```javascript
// Blob - just binary data
const blob = new Blob(['Hello'], { type: 'text/plain' });
console.log(blob.name); // undefined
console.log(blob.lastModified); // undefined

// File - blob with metadata
const file = new File(['Hello'], 'hello.txt', { 
  type: 'text/plain',
  lastModified: Date.now()
});
console.log(file.name); // 'hello.txt'
console.log(file.lastModified); // timestamp
console.log(file.size); // 5
console.log(file.type); // 'text/plain'

// File inherits from Blob
console.log(file instanceof Blob); // true
console.log(file instanceof File); // true
```

### Creating Files:

```javascript
// Constructor
const file = new File(
  ['content'], 
  'filename.txt', 
  { 
    type: 'text/plain',
    lastModified: new Date().getTime()
  }
);

// From blob
const blob = new Blob(['content'], { type: 'text/plain' });
const fileFromBlob = new File([blob], 'filename.txt');

// All blob methods work on files
file.text().then(console.log);
file.arrayBuffer().then(console.log);
```

---

## Advanced Techniques

### 1. **Blob Concatenation**

```javascript
function concatenateBlobs(blobs, type) {
  return new Blob(blobs, { type });
}

const blob1 = new Blob(['Hello ']);
const blob2 = new Blob(['World']);
const combined = concatenateBlobs([blob1, blob2], 'text/plain');

combined.text().then(console.log); // 'Hello World'
```

### 2. **Blob Comparison**

```javascript
async function blobsAreEqual(blob1, blob2) {
  if (blob1.size !== blob2.size || blob1.type !== blob2.type) {
    return false;
  }
  
  const buffer1 = await blob1.arrayBuffer();
  const buffer2 = await blob2.arrayBuffer();
  
  const view1 = new Uint8Array(buffer1);
  const view2 = new Uint8Array(buffer2);
  
  return view1.every((byte, index) => byte === view2[index]);
}
```

### 3. **Blob Encryption/Decryption**

```javascript
async function encryptBlob(blob, password) {
  const buffer = await blob.arrayBuffer();
  const key = await deriveKey(password);
  const iv = crypto.getRandomValues(new Uint8Array(12));
  
  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    buffer
  );
  
  // Combine IV and encrypted data
  const combined = new Uint8Array(iv.length + encrypted.byteLength);
  combined.set(iv);
  combined.set(new Uint8Array(encrypted), iv.length);
  
  return new Blob([combined], { type: 'application/octet-stream' });
}

async function deriveKey(password) {
  const encoder = new TextEncoder();
  const keyMaterial = await crypto.subtle.importKey(
    'raw',
    encoder.encode(password),
    'PBKDF2',
    false,
    ['deriveKey']
  );
  
  return crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt: encoder.encode('salt'),
      iterations: 100000,
      hash: 'SHA-256'
    },
    keyMaterial,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt', 'decrypt']
  );
}
```

### 4. **Blob Caching**

```javascript
class BlobCache {
  constructor() {
    this.cache = new Map();
  }
  
  set(key, blob) {
    this.cache.set(key, {
      blob,
      url: URL.createObjectURL(blob),
      timestamp: Date.now()
    });
  }
  
  get(key) {
    return this.cache.get(key);
  }
  
  getURL(key) {
    const item = this.cache.get(key);
    return item ? item.url : null;
  }
  
  has(key) {
    return this.cache.has(key);
  }
  
  delete(key) {
    const item = this.cache.get(key);
    if (item) {
      URL.revokeObjectURL(item.url);
      this.cache.delete(key);
    }
  }
  
  clear() {
    this.cache.forEach((item) => {
      URL.revokeObjectURL(item.url);
    });
    this.cache.clear();
  }
  
  // Clean old entries
  cleanup(maxAge = 3600000) { // 1 hour default
    const now = Date.now();
    this.cache.forEach((item, key) => {
      if (now - item.timestamp > maxAge) {
        this.delete(key);
      }
    });
  }
}

// Usage
const cache = new BlobCache();
const blob = new Blob(['Hello'], { type: 'text/plain' });
cache.set('greeting', blob);

const url = cache.getURL('greeting');
console.log(url);
```

### 5. **Progress Tracking**

```javascript
async function readBlobWithProgress(blob, onProgress) {
  const stream = blob.stream();
  const reader = stream.getReader();
  const totalSize = blob.size;
  let receivedSize = 0;
  const chunks = [];
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) break;
    
    chunks.push(value);
    receivedSize += value.length;
    
    if (onProgress) {
      onProgress({
        loaded: receivedSize,
        total: totalSize,
        percentage: (receivedSize / totalSize) * 100
      });
    }
  }
  
  const allChunks = new Uint8Array(receivedSize);
  let position = 0;
  for (const chunk of chunks) {
    allChunks.set(chunk, position);
    position += chunk.length;
  }
  
  return allChunks;
}

// Usage
const largeBlob = new Blob([new ArrayBuffer(10000000)]);
readBlobWithProgress(largeBlob, (progress) => {
  console.log(`${progress.percentage.toFixed(2)}% loaded`);
});
```

### 6. **Blob to Base64**

```javascript
function blobToBase64(blob) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onloadend = () => resolve(reader.result);
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
}

// Or modern way
async function blobToBase64Modern(blob) {
  const buffer = await blob.arrayBuffer();
  const bytes = new Uint8Array(buffer);
  let binary = '';
  for (let i = 0; i < bytes.length; i++) {
    binary += String.fromCharCode(bytes[i]);
  }
  return 'data:' + blob.type + ';base64,' + btoa(binary);
}

// Usage
const blob = new Blob(['Hello World'], { type: 'text/plain' });
blobToBase64(blob).then(console.log);
```

### 7. **Base64 to Blob**

```javascript
function base64ToBlob(base64, contentType = '') {
  const byteCharacters = atob(base64.split(',')[1]);
  const byteArrays = [];
  
  for (let offset = 0; offset < byteCharacters.length; offset += 512) {
    const slice = byteCharacters.slice(offset, offset + 512);
    const byteNumbers = new Array(slice.length);
    
    for (let i = 0; i < slice.length; i++) {
      byteNumbers[i] = slice.charCodeAt(i);
    }
    
    const byteArray = new Uint8Array(byteNumbers);
    byteArrays.push(byteArray);
  }
  
  return new Blob(byteArrays, { type: contentType });
}

// Usage
const base64 = 'data:text/plain;base64,SGVsbG8gV29ybGQ=';
const blob = base64ToBlob(base64, 'text/plain');
```

---

## Common MIME Types

```javascript
const mimeTypes = {
  // Text
  txt: 'text/plain',
  html: 'text/html',
  css: 'text/css',
  js: 'text/javascript',
  json: 'application/json',
  xml: 'application/xml',
  
  // Images
  png: 'image/png',
  jpg: 'image/jpeg',
  jpeg: 'image/jpeg',
  gif: 'image/gif',
  svg: 'image/svg+xml',
  webp: 'image/webp',
  
  // Audio
  mp3: 'audio/mpeg',
  wav: 'audio/wav',
  ogg: 'audio/ogg',
  
  // Video
  mp4: 'video/mp4',
  webm: 'video/webm',
  
  // Documents
  pdf: 'application/pdf',
  doc: 'application/msword',
  docx: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
  
  // Archives
  zip: 'application/zip',
  
  // Binary
  bin: 'application/octet-stream'
};
```

---

## Best Practices

1. **Always revoke blob URLs** to prevent memory leaks:
```javascript
const url = URL.createObjectURL(blob);
// ... use url ...
URL.revokeObjectURL(url);
```

2. **Use appropriate MIME types** for proper browser handling

3. **Handle errors** when reading blobs:
```javascript
try {
  const text = await blob.text();
} catch (error) {
  console.error('Failed to read blob:', error);
}
```

4. **Consider blob size** for performance:
```javascript
if (blob.size > 10 * 1024 * 1024) { // 10MB
  console.warn('Large blob detected');
  // Use chunked processing
}
```

5. **Clean up resources** in component unmount/cleanup:
```javascript
// React example
useEffect(() => {
  const url = URL.createObjectURL(blob);
  
  return () => {
    URL.revokeObjectURL(url); // Cleanup
  };
}, [blob]);
```

---

