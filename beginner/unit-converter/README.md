# Unit Converter

<p align="center">
  <img src="https://images.unsplash.com/photo-1580859789928-f45fc0c1a50c?w=600&q=80" alt="Unit Converter" width="600">
</p>

**Level:** Beginner | **Time:** 2-3 hours | **Prerequisites:** HTML, CSS, JavaScript, Math

## Description

Build a universal unit converter supporting multiple categories: length, weight, volume, temperature, speed, area, time, and more. Perfect for learning category management, conversion formulas, and creating practical calculation tools.

## Core Features

### Must Have
- Multiple conversion categories (length, weight, volume, temperature)
- Real-time bidirectional conversion
- Dropdown selectors for units
- Clear and swap functionality
- Mobile responsive design

### Nice to Have
- 10+ conversion categories
- Favorite conversions
- Conversion history
- Copy result to clipboard
- Custom unit creation
- Batch conversion
- Formula display
- Dark mode

## Implementation Guide

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Unit Converter</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>🔄 Unit Converter</h1>
        
        <div class="converter-card">
            <select id="category" class="category-select">
                <option value="length">Length</option>
                <option value="weight">Weight</option>
                <option value="volume">Volume</option>
                <option value="temperature">Temperature</option>
                <option value="speed">Speed</option>
                <option value="area">Area</option>
            </select>
            
            <div class="conversion-group">
                <div class="input-block">
                    <input type="number" id="fromValue" placeholder="0">
                    <select id="fromUnit"></select>
                </div>
                
                <button id="swapBtn" class="swap-btn">⇄</button>
                
                <div class="input-block">
                    <input type="number" id="toValue" placeholder="0" readonly>
                    <select id="toUnit"></select>
                </div>
            </div>
            
            <div class="actions">
                <button id="clearBtn">Clear</button>
                <button id="copyBtn">Copy Result</button>
            </div>
        </div>
        
        <div class="formula-display" id="formulaDisplay"></div>
    </div>
    
    <script src="script.js"></script>
</body>
</html>
```

```javascript
class UnitConverter {
    constructor() {
        this.conversions = {
            length: {
                units: ['meter', 'kilometer', 'centimeter', 'millimeter', 'mile', 'yard', 'foot', 'inch'],
                formulas: {
                    meter: { meter: 1, kilometer: 0.001, centimeter: 100, millimeter: 1000, 
                            mile: 0.000621371, yard: 1.09361, foot: 3.28084, inch: 39.3701 }
                }
            },
            weight: {
                units: ['kilogram', 'gram', 'milligram', 'pound', 'ounce', 'ton'],
                formulas: {
                    kilogram: { kilogram: 1, gram: 1000, milligram: 1000000, 
                               pound: 2.20462, ounce: 35.274, ton: 0.001 }
                }
            },
            volume: {
                units: ['liter', 'milliliter', 'gallon', 'quart', 'pint', 'cup', 'fluid_ounce'],
                formulas: {
                    liter: { liter: 1, milliliter: 1000, gallon: 0.264172, 
                            quart: 1.05669, pint: 2.11338, cup: 4.22675, fluid_ounce: 33.814 }
                }
            }
        };
        
        this.init();
    }
    
    init() {
        this.category = document.getElementById('category');
        this.fromValue = document.getElementById('fromValue');
        this.toValue = document.getElementById('toValue');
        this.fromUnit = document.getElementById('fromUnit');
        this.toUnit = document.getElementById('toUnit');
        
        this.category.addEventListener('change', () => this.updateUnits());
        this.fromValue.addEventListener('input', () => this.convert());
        this.fromUnit.addEventListener('change', () => this.convert());
        this.toUnit.addEventListener('change', () => this.convert());
        
        document.getElementById('swapBtn').addEventListener('click', () => this.swap());
        document.getElementById('clearBtn').addEventListener('click', () => this.clear());
        document.getElementById('copyBtn').addEventListener('click', () => this.copy());
        
        this.updateUnits();
    }
    
    updateUnits() {
        const category = this.category.value;
        const units = this.conversions[category].units;
        
        this.fromUnit.innerHTML = units.map(u => `<option value="${u}">${u}</option>`).join('');
        this.toUnit.innerHTML = units.map(u => `<option value="${u}">${u}</option>`).join('');
        
        this.toUnit.selectedIndex = 1;
        this.convert();
    }
    
    convert() {
        const value = parseFloat(this.fromValue.value);
        if (isNaN(value)) {
            this.toValue.value = '';
            return;
        }
        
        const category = this.category.value;
        const from = this.fromUnit.value;
        const to = this.toUnit.value;
        
        const baseUnit = Object.keys(this.conversions[category].formulas)[0];
        const toBase = this.conversions[category].formulas[baseUnit][from];
        const fromBase = this.conversions[category].formulas[baseUnit][to];
        
        const result = (value / toBase) * fromBase;
        this.toValue.value = result.toFixed(6);
    }
    
    swap() {
        const tempValue = this.fromValue.value;
        const tempUnit = this.fromUnit.value;
        
        this.fromValue.value = this.toValue.value;
        this.fromUnit.value = this.toUnit.value;
        this.toUnit.value = tempUnit;
        
        this.convert();
    }
    
    clear() {
        this.fromValue.value = '';
        this.toValue.value = '';
    }
    
    copy() {
        navigator.clipboard.writeText(this.toValue.value);
        alert('Result copied!');
    }
}

new UnitConverter();
```

## Technology Stack
- HTML5, CSS3, Vanilla JavaScript
- Math.js for advanced calculations
- Chart.js for visualization

## Testing Checklist
- [ ] All conversions accurate
- [ ] Swap function works
- [ ] Clear resets fields
- [ ] Copy to clipboard works
- [ ] Responsive design

## Resources
- [Unit Conversion - Wikipedia](https://en.wikipedia.org/wiki/Conversion_of_units)
- [Math.js Documentation](https://mathjs.org/)

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
