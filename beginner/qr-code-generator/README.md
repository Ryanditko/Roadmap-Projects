# QR Code Generator

<p align="center">
  <img src="https://images.unsplash.com/photo-1614680376573-df3480f0c6ff?w=600&q=80" alt="QR Code Generator" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 1-2 hours  
**Prerequisites:** Basic HTML, CSS, JavaScript, using external libraries

---

## Description

Build a QR Code Generator that converts text, URLs, contact information, or WiFi credentials into scannable QR codes. This project teaches you about working with external JavaScript libraries, data encoding, canvas manipulation, and creating practical utility tools that bridge physical and digital worlds.

QR codes are everywhere - from restaurant menus to payment systems to marketing materials. By building this generator, you will learn how to integrate third-party libraries, handle different data types, customize visual appearance, and provide download functionality for generated images.

---

## Core Features

### Must Have
- Text input field for data to encode
- Generate QR code button
- Display generated QR code image
- Download QR code as PNG image
- Support for URLs and plain text

### Nice to Have
- Customizable QR code size
- Color customization (foreground/background)
- Multiple data type support (WiFi, vCard, email, SMS)
- Error correction level selection
- Logo/image embedding in center
- Batch QR code generation
- QR code scanner functionality
- History of generated codes
- Export as SVG format

---

## Learning Objectives

By building this project, you will learn:

- **External Libraries**: Integrating and using third-party JavaScript libraries
- **Canvas API**: Working with HTML5 Canvas for image generation
- **File Download**: Creating and triggering file downloads from browser
- **Data Encoding**: Understanding QR code data formats and structures
- **Form Handling**: Processing different input types and validations
- **Event Handling**: Managing button clicks and form submissions
- **DOM Manipulation**: Dynamically creating and updating elements

---

## Implementation Guide

### Step 1: Set Up HTML Structure

Create the interface with input and display areas:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Code Generator</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>📱 QR Code Generator</h1>
        
        <div class="generator-section">
            <!-- Data Type Selector -->
            <div class="type-selector">
                <button class="type-btn active" data-type="text">📝 Text/URL</button>
                <button class="type-btn" data-type="wifi">📶 WiFi</button>
                <button class="type-btn" data-type="vcard">👤 Contact</button>
                <button class="type-btn" data-type="email">✉️ Email</button>
            </div>
            
            <!-- Input Forms -->
            <div class="input-section">
                <!-- Text/URL Form -->
                <div id="textForm" class="input-form active">
                    <label for="textInput">Enter Text or URL:</label>
                    <textarea id="textInput" rows="4" placeholder="https://example.com or any text..."></textarea>
                </div>
                
                <!-- WiFi Form -->
                <div id="wifiForm" class="input-form">
                    <label for="wifiSSID">Network Name (SSID):</label>
                    <input type="text" id="wifiSSID" placeholder="MyWiFi">
                    
                    <label for="wifiPassword">Password:</label>
                    <input type="password" id="wifiPassword" placeholder="password123">
                    
                    <label for="wifiEncryption">Security:</label>
                    <select id="wifiEncryption">
                        <option value="WPA">WPA/WPA2</option>
                        <option value="WEP">WEP</option>
                        <option value="nopass">None</option>
                    </select>
                </div>
                
                <!-- vCard Form -->
                <div id="vcardForm" class="input-form">
                    <label for="vcardName">Full Name:</label>
                    <input type="text" id="vcardName" placeholder="John Doe">
                    
                    <label for="vcardPhone">Phone:</label>
                    <input type="tel" id="vcardPhone" placeholder="+1234567890">
                    
                    <label for="vcardEmail">Email:</label>
                    <input type="email" id="vcardEmail" placeholder="john@example.com">
                    
                    <label for="vcardOrg">Organization:</label>
                    <input type="text" id="vcardOrg" placeholder="Company Name">
                </div>
                
                <!-- Email Form -->
                <div id="emailForm" class="input-form">
                    <label for="emailTo">To:</label>
                    <input type="email" id="emailTo" placeholder="recipient@example.com">
                    
                    <label for="emailSubject">Subject:</label>
                    <input type="text" id="emailSubject" placeholder="Hello">
                    
                    <label for="emailBody">Message:</label>
                    <textarea id="emailBody" rows="3" placeholder="Your message here..."></textarea>
                </div>
            </div>
            
            <!-- Customization Options -->
            <div class="options-section">
                <h3>⚙️ Customization</h3>
                <div class="options-grid">
                    <div class="option">
                        <label for="qrSize">Size:</label>
                        <select id="qrSize">
                            <option value="200">Small (200px)</option>
                            <option value="300" selected>Medium (300px)</option>
                            <option value="400">Large (400px)</option>
                            <option value="500">Extra Large (500px)</option>
                        </select>
                    </div>
                    
                    <div class="option">
                        <label for="qrColorDark">Foreground:</label>
                        <input type="color" id="qrColorDark" value="#000000">
                    </div>
                    
                    <div class="option">
                        <label for="qrColorLight">Background:</label>
                        <input type="color" id="qrColorLight" value="#ffffff">
                    </div>
                    
                    <div class="option">
                        <label for="errorCorrection">Error Correction:</label>
                        <select id="errorCorrection">
                            <option value="L">Low (7%)</option>
                            <option value="M" selected>Medium (15%)</option>
                            <option value="Q">Quartile (25%)</option>
                            <option value="H">High (30%)</option>
                        </select>
                    </div>
                </div>
            </div>
            
            <!-- Generate Button -->
            <button id="generateBtn" class="btn-generate">
                Generate QR Code
            </button>
        </div>
        
        <!-- QR Code Display -->
        <div id="qrResult" class="qr-result hidden">
            <h2>Your QR Code</h2>
            <div id="qrCanvas"></div>
            <div class="qr-actions">
                <button id="downloadBtn" class="btn-download">
                    💾 Download PNG
                </button>
                <button id="copyBtn" class="btn-copy">
                    📋 Copy to Clipboard
                </button>
                <button id="newBtn" class="btn-new">
                    ➕ Generate New
                </button>
            </div>
        </div>
    </div>
    
    <!-- Include QRCode.js library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="script.js"></script>
</body>
</html>
```

### Step 2: Implement QR Code Generation Logic

Create JavaScript to handle generation:

```javascript
// script.js
class QRCodeGenerator {
    constructor() {
        this.currentType = 'text';
        this.qrCode = null;
        
        this.elements = {
            typeBtns: document.querySelectorAll('.type-btn'),
            inputForms: document.querySelectorAll('.input-form'),
            generateBtn: document.getElementById('generateBtn'),
            qrResult: document.getElementById('qrResult'),
            qrCanvas: document.getElementById('qrCanvas'),
            downloadBtn: document.getElementById('downloadBtn'),
            copyBtn: document.getElementById('copyBtn'),
            newBtn: document.getElementById('newBtn'),
            qrSize: document.getElementById('qrSize'),
            qrColorDark: document.getElementById('qrColorDark'),
            qrColorLight: document.getElementById('qrColorLight'),
            errorCorrection: document.getElementById('errorCorrection')
        };
        
        this.init();
    }
    
    init() {
        // Type selector buttons
        this.elements.typeBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.switchType(e.target.dataset.type);
            });
        });
        
        // Generate button
        this.elements.generateBtn.addEventListener('click', () => {
            this.generateQRCode();
        });
        
        // Action buttons
        this.elements.downloadBtn.addEventListener('click', () => {
            this.downloadQRCode();
        });
        
        this.elements.copyBtn.addEventListener('click', () => {
            this.copyToClipboard();
        });
        
        this.elements.newBtn.addEventListener('click', () => {
            this.reset();
        });
    }
    
    switchType(type) {
        this.currentType = type;
        
        // Update active button
        this.elements.typeBtns.forEach(btn => {
            btn.classList.toggle('active', btn.dataset.type === type);
        });
        
        // Show corresponding form
        this.elements.inputForms.forEach(form => {
            form.classList.remove('active');
        });
        document.getElementById(`${type}Form`).classList.add('active');
    }
    
    getData() {
        switch (this.currentType) {
            case 'text':
                return document.getElementById('textInput').value.trim();
            
            case 'wifi':
                const ssid = document.getElementById('wifiSSID').value;
                const password = document.getElementById('wifiPassword').value;
                const encryption = document.getElementById('wifiEncryption').value;
                return `WIFI:T:${encryption};S:${ssid};P:${password};;`;
            
            case 'vcard':
                const name = document.getElementById('vcardName').value;
                const phone = document.getElementById('vcardPhone').value;
                const email = document.getElementById('vcardEmail').value;
                const org = document.getElementById('vcardOrg').value;
                return `BEGIN:VCARD\nVERSION:3.0\nFN:${name}\nTEL:${phone}\nEMAIL:${email}\nORG:${org}\nEND:VCARD`;
            
            case 'email':
                const to = document.getElementById('emailTo').value;
                const subject = document.getElementById('emailSubject').value;
                const body = document.getElementById('emailBody').value;
                return `mailto:${to}?subject=${encodeURIComponent(subject)}&body=${encodeURIComponent(body)}`;
            
            default:
                return '';
        }
    }
    
    generateQRCode() {
        const data = this.getData();
        
        if (!data) {
            alert('Please enter data to generate QR code!');
            return;
        }
        
        // Clear previous QR code
        this.elements.qrCanvas.innerHTML = '';
        
        // Get options
        const size = parseInt(this.elements.qrSize.value);
        const colorDark = this.elements.qrColorDark.value;
        const colorLight = this.elements.qrColorLight.value;
        const correctLevel = this.elements.errorCorrection.value;
        
        // Generate new QR code
        this.qrCode = new QRCode(this.elements.qrCanvas, {
            text: data,
            width: size,
            height: size,
            colorDark: colorDark,
            colorLight: colorLight,
            correctLevel: QRCode.CorrectLevel[correctLevel]
        });
        
        // Show result section
        this.elements.qrResult.classList.remove('hidden');
        
        // Scroll to result
        this.elements.qrResult.scrollIntoView({ behavior: 'smooth' });
    }
    
    downloadQRCode() {
        const canvas = this.elements.qrCanvas.querySelector('canvas');
        if (!canvas) return;
        
        // Convert canvas to blob
        canvas.toBlob((blob) => {
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `qrcode-${Date.now()}.png`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        });
    }
    
    async copyToClipboard() {
        const canvas = this.elements.qrCanvas.querySelector('canvas');
        if (!canvas) return;
        
        try {
            canvas.toBlob(async (blob) => {
                await navigator.clipboard.write([
                    new ClipboardItem({ 'image/png': blob })
                ]);
                
                // Show feedback
                const originalText = this.elements.copyBtn.textContent;
                this.elements.copyBtn.textContent = '✅ Copied!';
                setTimeout(() => {
                    this.elements.copyBtn.textContent = originalText;
                }, 2000);
            });
        } catch (error) {
            console.error('Failed to copy:', error);
            alert('Failed to copy to clipboard');
        }
    }
    
    reset() {
        // Hide result
        this.elements.qrResult.classList.add('hidden');
        
        // Clear inputs
        document.getElementById('textInput').value = '';
        document.getElementById('wifiSSID').value = '';
        document.getElementById('wifiPassword').value = '';
        document.getElementById('vcardName').value = '';
        document.getElementById('vcardPhone').value = '';
        document.getElementById('vcardEmail').value = '';
        document.getElementById('vcardOrg').value = '';
        document.getElementById('emailTo').value = '';
        document.getElementById('emailSubject').value = '';
        document.getElementById('emailBody').value = '';
        
        // Clear QR code
        this.elements.qrCanvas.innerHTML = '';
        
        // Scroll back to top
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }
}

// Initialize
new QRCodeGenerator();
```

### Step 3: Style the Generator

Create modern, user-friendly styling:

```css
/* styles.css */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}

.container {
    max-width: 800px;
    margin: 0 auto;
}

h1 {
    text-align: center;
    color: white;
    font-size: 48px;
    margin-bottom: 30px;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}

.generator-section {
    background: white;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    margin-bottom: 30px;
}

/* Type Selector */
.type-selector {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 10px;
    margin-bottom: 30px;
}

.type-btn {
    padding: 15px;
    border: 2px solid #e0e0e0;
    background: white;
    border-radius: 12px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.type-btn:hover {
    border-color: #667eea;
    transform: translateY(-2px);
}

.type-btn.active {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border-color: transparent;
}

/* Input Forms */
.input-form {
    display: none;
}

.input-form.active {
    display: block;
}

.input-form label {
    display: block;
    font-weight: 600;
    margin-bottom: 8px;
    color: #333;
}

.input-form input,
.input-form textarea,
.input-form select {
    width: 100%;
    padding: 12px;
    border: 2px solid #e0e0e0;
    border-radius: 8px;
    font-size: 14px;
    margin-bottom: 15px;
    font-family: inherit;
    transition: border-color 0.3s;
}

.input-form input:focus,
.input-form textarea:focus,
.input-form select:focus {
    outline: none;
    border-color: #667eea;
}

/* Options Section */
.options-section {
    margin: 30px 0;
    padding: 20px;
    background: #f8f9fa;
    border-radius: 12px;
}

.options-section h3 {
    margin-bottom: 15px;
    color: #333;
}

.options-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
}

.option label {
    display: block;
    font-size: 13px;
    font-weight: 600;
    margin-bottom: 5px;
    color: #666;
}

.option input,
.option select {
    width: 100%;
    padding: 8px;
    border: 2px solid #e0e0e0;
    border-radius: 6px;
}

.option input[type="color"] {
    height: 40px;
    cursor: pointer;
}

/* Generate Button */
.btn-generate {
    width: 100%;
    padding: 18px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 18px;
    font-weight: bold;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-generate:hover {
    transform: translateY(-3px);
    box-shadow: 0 10px 30px rgba(102, 126, 234, 0.4);
}

/* QR Result Section */
.qr-result {
    background: white;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    text-align: center;
}

.qr-result.hidden {
    display: none;
}

.qr-result h2 {
    margin-bottom: 30px;
    color: #333;
}

#qrCanvas {
    display: inline-block;
    padding: 20px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    margin-bottom: 30px;
}

/* Action Buttons */
.qr-actions {
    display: flex;
    gap: 10px;
    justify-content: center;
    flex-wrap: wrap;
}

.qr-actions button {
    padding: 12px 24px;
    border: none;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-download {
    background: #4caf50;
    color: white;
}

.btn-download:hover {
    background: #45a049;
    transform: translateY(-2px);
}

.btn-copy {
    background: #2196f3;
    color: white;
}

.btn-copy:hover {
    background: #0b7dda;
    transform: translateY(-2px);
}

.btn-new {
    background: #ff9800;
    color: white;
}

.btn-new:hover {
    background: #f57c00;
    transform: translateY(-2px);
}

/* Responsive */
@media (max-width: 768px) {
    h1 {
        font-size: 32px;
    }
    
    .generator-section,
    .qr-result {
        padding: 25px;
    }
    
    .type-selector {
        grid-template-columns: 1fr 1fr;
    }
    
    .options-grid {
        grid-template-columns: 1fr;
    }
    
    .qr-actions {
        flex-direction: column;
    }
    
    .qr-actions button {
        width: 100%;
    }
}
```

---

## Technology Stack

- **HTML5**: Structure and forms
- **CSS3**: Modern styling with gradients
- **JavaScript**: Logic and DOM manipulation
- **QRCode.js Library**: QR code generation
- **Canvas API**: Image rendering

---

## Testing Checklist

- [ ] QR codes generate for text/URLs
- [ ] WiFi QR codes work on mobile devices
- [ ] vCard QR codes import contacts correctly
- [ ] Email QR codes open email client
- [ ] Color customization applies
- [ ] Size options work correctly
- [ ] Download saves PNG file
- [ ] Copy to clipboard works
- [ ] Error correction levels affect quality
- [ ] Responsive on mobile devices

---

## Additional Challenges

### Easy
- Add QR code templates (common URLs)
- Implement QR code history
- Add more data types (SMS, phone)
- Create batch generation

### Medium
- Add QR code scanner using device camera
- Implement logo embedding in center
- Add SVG export option
- Create URL shortening integration
- Add analytics tracking

### Hard
- Build dynamic QR codes (editable content)
- Add custom design patterns
- Implement animated QR codes
- Create API for programmatic generation
- Build Chrome extension version

---

## Resources

- [QRCode.js Documentation](https://github.com/davidshimjs/qrcodejs)
- [QR Code Standard](https://www.qrcode.com/en/about/standards.html)
- [Canvas API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)

---

## License

MIT License - Part of [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
