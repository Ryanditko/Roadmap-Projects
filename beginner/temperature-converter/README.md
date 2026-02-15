# Temperature Converter

<p align="center">
  <img src="https://images.unsplash.com/photo-1564939558297-fc396f18e5c7?w=600&q=80" alt="Temperature Converter" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 1-2 hours  
**Prerequisites:** Basic HTML, CSS, JavaScript, mathematical operations

---

## Description

Build a temperature converter that converts between Celsius, Fahrenheit, and Kelvin scales. This project teaches fundamental programming concepts including mathematical formulas, form handling, input validation, and creating practical utility tools. Perfect for beginners learning how to process user input and display calculated results.

Temperature conversion is a common real-world problem in science, cooking, travel, and weather applications. This project introduces you to handling numerical inputs, implementing conversion formulas, and creating intuitive user interfaces for quick calculations.

---

## Core Features

### Must Have
- Convert between Celsius, Fahrenheit, and Kelvin
- Input validation (numeric values only)
- Real-time conversion as user types
- Clear/reset functionality
- Display conversion formulas

### Nice to Have
- Bi-directional conversion (auto-detect input scale)
- Temperature range validation (absolute zero check)
- Conversion history
- Dark mode toggle
- Preset temperature values (freezing point, boiling point, body temperature)
- Unit symbol display
- Copy result to clipboard
- Responsive design for mobile
- Keyboard shortcuts

---

## Learning Objectives

By building this project, you will learn:

- **Mathematical Operations**: Implementing temperature conversion formulas
- **Form Handling**: Processing user input from forms
- **Input Validation**: Ensuring valid numeric input
- **Event Handling**: Real-time updates on input changes
- **DOM Manipulation**: Updating results dynamically
- **Conditional Logic**: Handling different conversion types
- **User Experience**: Creating intuitive calculation interfaces

---

## Implementation Guide

### Step 1: Create HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Temperature Converter</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>🌡️ Temperature Converter</h1>
        
        <div class="converter-card">
            <!-- Celsius Input -->
            <div class="input-group">
                <label for="celsius">Celsius (°C)</label>
                <input type="number" id="celsius" placeholder="0" step="0.01">
            </div>
            
            <!-- Fahrenheit Input -->
            <div class="input-group">
                <label for="fahrenheit">Fahrenheit (°F)</label>
                <input type="number" id="fahrenheit" placeholder="32" step="0.01">
            </div>
            
            <!-- Kelvin Input -->
            <div class="input-group">
                <label for="kelvin">Kelvin (K)</label>
                <input type="number" id="kelvin" placeholder="273.15" step="0.01">
            </div>
            
            <!-- Reset Button -->
            <button id="resetBtn" class="btn-reset">Clear All</button>
        </div>
        
        <!-- Formulas Reference -->
        <div class="formulas-card">
            <h3>📐 Conversion Formulas</h3>
            <div class="formula-grid">
                <div class="formula-item">
                    <strong>°C to °F:</strong>
                    <span>(°C × 9/5) + 32</span>
                </div>
                <div class="formula-item">
                    <strong>°C to K:</strong>
                    <span>°C + 273.15</span>
                </div>
                <div class="formula-item">
                    <strong>°F to °C:</strong>
                    <span>(°F - 32) × 5/9</span>
                </div>
                <div class="formula-item">
                    <strong>°F to K:</strong>
                    <span>(°F - 32) × 5/9 + 273.15</span>
                </div>
                <div class="formula-item">
                    <strong>K to °C:</strong>
                    <span>K - 273.15</span>
                </div>
                <div class="formula-item">
                    <strong>K to °F:</strong>
                    <span>(K - 273.15) × 9/5 + 32</span>
                </div>
            </div>
        </div>
        
        <!-- Quick Reference -->
        <div class="reference-card">
            <h3>📊 Reference Points</h3>
            <div class="reference-grid">
                <button class="ref-btn" data-celsius="-273.15">Absolute Zero</button>
                <button class="ref-btn" data-celsius="0">Water Freezing</button>
                <button class="ref-btn" data-celsius="37">Body Temperature</button>
                <button class="ref-btn" data-celsius="100">Water Boiling</button>
            </div>
        </div>
    </div>
    
    <script src="script.js"></script>
</body>
</html>
```

### Step 2: Implement Conversion Logic

```javascript
// script.js
class TemperatureConverter {
    constructor() {
        this.celsiusInput = document.getElementById('celsius');
        this.fahrenheitInput = document.getElementById('fahrenheit');
        this.kelvinInput = document.getElementById('kelvin');
        this.resetBtn = document.getElementById('resetBtn');
        
        this.init();
    }
    
    init() {
        // Add event listeners for real-time conversion
        this.celsiusInput.addEventListener('input', () => {
            this.convertFromCelsius();
        });
        
        this.fahrenheitInput.addEventListener('input', () => {
            this.convertFromFahrenheit();
        });
        
        this.kelvinInput.addEventListener('input', () => {
            this.convertFromKelvin();
        });
        
        // Reset button
        this.resetBtn.addEventListener('click', () => {
            this.reset();
        });
        
        // Reference buttons
        document.querySelectorAll('.ref-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                const celsius = parseFloat(e.target.dataset.celsius);
                this.celsiusInput.value = celsius;
                this.convertFromCelsius();
            });
        });
    }
    
    convertFromCelsius() {
        const celsius = parseFloat(this.celsiusInput.value);
        
        if (isNaN(celsius)) {
            this.fahrenheitInput.value = '';
            this.kelvinInput.value = '';
            return;
        }
        
        // Validate minimum (absolute zero)
        if (celsius < -273.15) {
            alert('Temperature cannot be below absolute zero (-273.15°C)');
            this.celsiusInput.value = -273.15;
            return;
        }
        
        // Convert to Fahrenheit: (°C × 9/5) + 32
        const fahrenheit = (celsius * 9/5) + 32;
        this.fahrenheitInput.value = fahrenheit.toFixed(2);
        
        // Convert to Kelvin: °C + 273.15
        const kelvin = celsius + 273.15;
        this.kelvinInput.value = kelvin.toFixed(2);
    }
    
    convertFromFahrenheit() {
        const fahrenheit = parseFloat(this.fahrenheitInput.value);
        
        if (isNaN(fahrenheit)) {
            this.celsiusInput.value = '';
            this.kelvinInput.value = '';
            return;
        }
        
        // Convert to Celsius: (°F - 32) × 5/9
        const celsius = (fahrenheit - 32) * 5/9;
        
        // Validate minimum
        if (celsius < -273.15) {
            alert('Temperature cannot be below absolute zero (-459.67°F)');
            this.fahrenheitInput.value = -459.67;
            return;
        }
        
        this.celsiusInput.value = celsius.toFixed(2);
        
        // Convert to Kelvin
        const kelvin = celsius + 273.15;
        this.kelvinInput.value = kelvin.toFixed(2);
    }
    
    convertFromKelvin() {
        const kelvin = parseFloat(this.kelvinInput.value);
        
        if (isNaN(kelvin)) {
            this.celsiusInput.value = '';
            this.fahrenheitInput.value = '';
            return;
        }
        
        // Validate minimum (0 K is absolute zero)
        if (kelvin < 0) {
            alert('Temperature cannot be below absolute zero (0 K)');
            this.kelvinInput.value = 0;
            return;
        }
        
        // Convert to Celsius: K - 273.15
        const celsius = kelvin - 273.15;
        this.celsiusInput.value = celsius.toFixed(2);
        
        // Convert to Fahrenheit
        const fahrenheit = (celsius * 9/5) + 32;
        this.fahrenheitInput.value = fahrenheit.toFixed(2);
    }
    
    reset() {
        this.celsiusInput.value = '';
        this.fahrenheitInput.value = '';
        this.kelvinInput.value = '';
        this.celsiusInput.focus();
    }
}

// Initialize
new TemperatureConverter();
```

### Step 3: Style the Converter

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
    display: flex;
    align-items: center;
    justify-content: center;
}

.container {
    max-width: 600px;
    width: 100%;
}

h1 {
    text-align: center;
    color: white;
    font-size: 42px;
    margin-bottom: 30px;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}

.converter-card,
.formulas-card,
.reference-card {
    background: white;
    border-radius: 20px;
    padding: 30px;
    margin-bottom: 20px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.input-group {
    margin-bottom: 20px;
}

.input-group label {
    display: block;
    font-weight: 600;
    margin-bottom: 8px;
    color: #333;
    font-size: 16px;
}

.input-group input {
    width: 100%;
    padding: 15px;
    border: 2px solid #e0e0e0;
    border-radius: 10px;
    font-size: 18px;
    transition: all 0.3s;
}

.input-group input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.btn-reset {
    width: 100%;
    padding: 15px;
    background: #f44336;
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-reset:hover {
    background: #d32f2f;
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(244, 67, 54, 0.3);
}

.formulas-card h3,
.reference-card h3 {
    margin-bottom: 20px;
    color: #333;
    text-align: center;
}

.formula-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
}

.formula-item {
    padding: 15px;
    background: #f8f9fa;
    border-radius: 10px;
    border-left: 4px solid #667eea;
}

.formula-item strong {
    display: block;
    color: #667eea;
    margin-bottom: 5px;
}

.formula-item span {
    color: #666;
    font-size: 14px;
}

.reference-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
}

.ref-btn {
    padding: 12px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 10px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.ref-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
}

@media (max-width: 768px) {
    h1 {
        font-size: 32px;
    }
    
    .converter-card,
    .formulas-card,
    .reference-card {
        padding: 20px;
    }
    
    .formula-grid {
        grid-template-columns: 1fr;
    }
}
```

---

## Technology Stack

- **HTML5**: Form inputs
- **CSS3**: Grid layouts and animations
- **Vanilla JavaScript**: Conversion logic
- **Mathematics**: Temperature conversion formulas

---

## Testing Checklist

- [ ] Celsius converts correctly to F and K
- [ ] Fahrenheit converts correctly to C and K
- [ ] Kelvin converts correctly to C and F
- [ ] Absolute zero validation works
- [ ] Reference buttons populate values
- [ ] Reset button clears all fields
- [ ] Decimal numbers work correctly
- [ ] Real-time conversion updates
- [ ] Mobile responsive

---

## Additional Challenges

### Easy
- Add Rankine and Réaumur scales
- Show conversion steps
- Add temperature color coding
- Implement keyboard shortcuts

### Medium
- Add conversion history
- Create comparison charts
- Add weather API integration
- Implement unit tests

### Hard
- Build scientific calculator mode
- Add thermodynamics calculations
- Create mobile app version
- Implement voice input

---

## Resources

- [Temperature Conversion - Wikipedia](https://en.wikipedia.org/wiki/Conversion_of_units_of_temperature)
- [MDN - Number.toFixed()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed)

---

## License

MIT License - Part of [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
