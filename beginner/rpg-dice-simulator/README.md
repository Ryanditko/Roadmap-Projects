# RPG Dice Simulator

<p align="center">
  <img src="https://images.unsplash.com/photo-1516975080664-ed2fc6a32937?w=600&q=80" alt="RPG Dice" width="600">
</p>

**Level:** Beginner | **Time:** 3-5 hours | **Prerequisites:** HTML, CSS, JavaScript basics

## Description

Build a comprehensive dice rolling simulator for tabletop RPG games that supports standard notation (1d20, 3d6+5), multiple dice types, roll history, and critical hit detection. Perfect for D&D, Pathfinder, and other RPG systems.

## Core Features

### Must Have
- Standard dice types (d4, d6, d8, d10, d12, d20, d100)
- Dice notation parser (e.g., "2d6+3", "1d20", "4d8-2")
- Roll animation with visual feedback
- Sum calculation with modifiers
- Roll history display
- Individual dice results shown
- Critical hit/fail detection (for d20)

### Nice to Have
- Advantage/disadvantage rolls (5e D&D)
- Keep highest/lowest dice (e.g., "4d6kh3")
- Exploding dice (roll again on max)
- Custom dice with any number of sides
- Save favorite roll combinations
- Roll statistics and probability calculator
- Sound effects for rolls
- 3D dice animation
- Multi-user rolling for game sessions

## Learning Objectives

- **Regular Expressions**: Parse complex dice notation strings
- **Random Number Generation**: Create fair dice rolls
- **Animation**: Implement smooth rolling effects
- **Data Structures**: Store and display roll history
- **State Management**: Track multiple dice and modifiers

## Implementation Guide

### Step 1: HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RPG Dice Simulator</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>🎲 RPG Dice Simulator</h1>
            <p>Roll dice for your tabletop adventures</p>
        </header>

        <div class="main-content">
            <div class="quick-rolls">
                <h3>Quick Rolls</h3>
                <div class="dice-buttons">
                    <button class="dice-btn" data-dice="1d4">d4</button>
                    <button class="dice-btn" data-dice="1d6">d6</button>
                    <button class="dice-btn" data-dice="1d8">d8</button>
                    <button class="dice-btn" data-dice="1d10">d10</button>
                    <button class="dice-btn" data-dice="1d12">d12</button>
                    <button class="dice-btn" data-dice="1d20">d20</button>
                    <button class="dice-btn" data-dice="1d100">d100</button>
                </div>
            </div>

            <div class="custom-roll">
                <h3>Custom Roll</h3>
                <div class="input-group">
                    <input 
                        type="text" 
                        id="diceNotation" 
                        placeholder="e.g., 2d6+3, 1d20, 4d8-2"
                        autocomplete="off"
                    >
                    <button id="rollBtn" class="btn-primary">Roll!</button>
                </div>
                <div class="examples">
                    Examples: <span class="example" data-notation="2d6+3">2d6+3</span>, 
                    <span class="example" data-notation="3d8">3d8</span>, 
                    <span class="example" data-notation="1d20+5">1d20+5</span>
                </div>
            </div>

            <div class="result-display">
                <div class="dice-animation" id="diceAnimation"></div>
                <div class="total-result" id="totalResult">-</div>
                <div class="dice-breakdown" id="diceBreakdown"></div>
                <div class="roll-details" id="rollDetails"></div>
            </div>

            <div class="history-section">
                <div class="history-header">
                    <h3>Roll History</h3>
                    <button id="clearHistory" class="btn-small">Clear</button>
                </div>
                <div id="historyList" class="history-list"></div>
            </div>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
```

### Step 2: CSS Styling

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
    min-height: 100vh;
    padding: 20px;
    color: #333;
}

.container {
    max-width: 900px;
    margin: 0 auto;
}

header {
    text-align: center;
    color: white;
    margin-bottom: 40px;
}

header h1 {
    font-size: 3rem;
    margin-bottom: 10px;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

header p {
    font-size: 1.2rem;
    opacity: 0.9;
}

.main-content {
    background: white;
    border-radius: 20px;
    padding: 30px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}

.quick-rolls, .custom-roll, .result-display, .history-section {
    margin-bottom: 30px;
}

h3 {
    color: #1e3c72;
    margin-bottom: 15px;
    font-size: 1.3rem;
}

.dice-buttons {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
    gap: 10px;
}

.dice-btn {
    padding: 20px;
    font-size: 1.5rem;
    font-weight: bold;
    border: 3px solid #1e3c72;
    background: white;
    color: #1e3c72;
    border-radius: 15px;
    cursor: pointer;
    transition: all 0.3s;
}

.dice-btn:hover {
    background: #1e3c72;
    color: white;
    transform: translateY(-3px);
    box-shadow: 0 5px 15px rgba(30, 60, 114, 0.3);
}

.dice-btn:active {
    transform: translateY(0);
}

.input-group {
    display: flex;
    gap: 10px;
}

#diceNotation {
    flex: 1;
    padding: 15px;
    font-size: 1.1rem;
    border: 2px solid #ddd;
    border-radius: 10px;
    transition: border-color 0.3s;
}

#diceNotation:focus {
    outline: none;
    border-color: #1e3c72;
}

.btn-primary {
    padding: 15px 40px;
    font-size: 1.1rem;
    font-weight: bold;
    background: linear-gradient(135deg, #1e3c72, #2a5298);
    color: white;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(30, 60, 114, 0.4);
}

.examples {
    margin-top: 10px;
    color: #666;
    font-size: 0.9rem;
}

.example {
    color: #1e3c72;
    cursor: pointer;
    text-decoration: underline;
    margin: 0 5px;
}

.example:hover {
    color: #2a5298;
}

.result-display {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 30px;
    border-radius: 15px;
    text-align: center;
    color: white;
}

.dice-animation {
    font-size: 4rem;
    margin-bottom: 20px;
    min-height: 80px;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-wrap: wrap;
    gap: 15px;
}

.die {
    display: inline-block;
    animation: roll 0.5s ease-in-out;
}

@keyframes roll {
    0%, 100% { transform: rotate(0deg); }
    25% { transform: rotate(90deg); }
    50% { transform: rotate(180deg); }
    75% { transform: rotate(270deg); }
}

.total-result {
    font-size: 4rem;
    font-weight: bold;
    margin: 20px 0;
}

.dice-breakdown {
    font-size: 1.2rem;
    opacity: 0.9;
    margin-bottom: 10px;
}

.roll-details {
    font-size: 1rem;
    opacity: 0.8;
}

.critical {
    animation: pulse 0.5s ease-in-out 3;
}

@keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.1); }
}

.history-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.btn-small {
    padding: 8px 16px;
    background: #e74c3c;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-small:hover {
    background: #c0392b;
}

.history-list {
    max-height: 300px;
    overflow-y: auto;
}

.history-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px;
    margin-bottom: 10px;
    background: #f8f9fa;
    border-radius: 10px;
    border-left: 4px solid #1e3c72;
}

.history-item:hover {
    background: #e9ecef;
}

.history-notation {
    font-weight: bold;
    color: #1e3c72;
}

.history-result {
    font-size: 1.5rem;
    font-weight: bold;
    color: #2a5298;
}

.history-time {
    font-size: 0.85rem;
    color: #999;
}

@media (max-width: 768px) {
    header h1 {
        font-size: 2rem;
    }
    
    .dice-buttons {
        grid-template-columns: repeat(4, 1fr);
    }
    
    .input-group {
        flex-direction: column;
    }
    
    .total-result {
        font-size: 3rem;
    }
}
```

### Step 3: JavaScript Implementation

```javascript
class DiceRoller {
    constructor() {
        this.history = JSON.parse(localStorage.getItem('diceHistory')) || [];
        this.initializeElements();
        this.attachEventListeners();
        this.renderHistory();
    }

    initializeElements() {
        this.diceNotation = document.getElementById('diceNotation');
        this.rollBtn = document.getElementById('rollBtn');
        this.totalResult = document.getElementById('totalResult');
        this.diceBreakdown = document.getElementById('diceBreakdown');
        this.rollDetails = document.getElementById('rollDetails');
        this.diceAnimation = document.getElementById('diceAnimation');
        this.historyList = document.getElementById('historyList');
    }

    attachEventListeners() {
        // Quick roll buttons
        document.querySelectorAll('.dice-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                const notation = e.target.dataset.dice;
                this.roll(notation);
            });
        });

        // Custom roll button
        this.rollBtn.addEventListener('click', () => {
            const notation = this.diceNotation.value.trim();
            if (notation) {
                this.roll(notation);
            }
        });

        // Enter key to roll
        this.diceNotation.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                this.rollBtn.click();
            }
        });

        // Example notations
        document.querySelectorAll('.example').forEach(ex => {
            ex.addEventListener('click', (e) => {
                const notation = e.target.dataset.notation;
                this.diceNotation.value = notation;
                this.roll(notation);
            });
        });

        // Clear history
        document.getElementById('clearHistory').addEventListener('click', () => {
            if (confirm('Clear all roll history?')) {
                this.history = [];
                localStorage.removeItem('diceHistory');
                this.renderHistory();
            }
        });
    }

    roll(notation) {
        const parsed = this.parseNotation(notation);
        
        if (!parsed) {
            alert('Invalid dice notation! Use format like: 2d6+3, 1d20, 3d8-2');
            return;
        }

        const result = this.performRoll(parsed);
        this.displayResult(notation, result);
        this.addToHistory(notation, result);
    }

    parseNotation(notation) {
        // Regex: (number)d(sides)(+/-)(modifier) with optional keep highest/lowest
        const regex = /^(\d*)d(\d+)([+-]\d+)?(kh|kl)?(\d+)?$/i;
        const match = notation.toLowerCase().replace(/\s/g, '').match(regex);

        if (!match) return null;

        return {
            count: parseInt(match[1]) || 1,
            sides: parseInt(match[2]),
            modifier: match[3] ? parseInt(match[3]) : 0,
            keepType: match[4] || null,
            keepCount: match[5] ? parseInt(match[5]) : null
        };
    }

    performRoll(parsed) {
        let rolls = [];
        
        // Roll all dice
        for (let i = 0; i < parsed.count; i++) {
            rolls.push(Math.floor(Math.random() * parsed.sides) + 1);
        }

        let keptRolls = [...rolls];
        let droppedRolls = [];

        // Handle keep highest/lowest
        if (parsed.keepType && parsed.keepCount) {
            rolls.sort((a, b) => parsed.keepType === 'kh' ? b - a : a - b);
            keptRolls = rolls.slice(0, parsed.keepCount);
            droppedRolls = rolls.slice(parsed.keepCount);
        }

        const sum = keptRolls.reduce((a, b) => a + b, 0);
        const total = sum + parsed.modifier;

        // Check for critical (only for d20)
        let isCritical = false;
        let isCriticalFail = false;
        if (parsed.sides === 20 && parsed.count === 1) {
            if (rolls[0] === 20) isCritical = true;
            if (rolls[0] === 1) isCriticalFail = true;
        }

        return {
            rolls: keptRolls,
            droppedRolls,
            sum,
            modifier: parsed.modifier,
            total,
            isCritical,
            isCriticalFail,
            sides: parsed.sides
        };
    }

    displayResult(notation, result) {
        // Animate dice
        this.diceAnimation.innerHTML = '';
        result.rolls.forEach(roll => {
            const die = document.createElement('div');
            die.className = 'die';
            die.textContent = '🎲';
            this.diceAnimation.appendChild(die);
        });

        // Display total
        this.totalResult.textContent = result.total;
        
        if (result.isCritical) {
            this.totalResult.classList.add('critical');
            setTimeout(() => this.totalResult.classList.remove('critical'), 1500);
        }

        // Breakdown
        const rolls = result.rolls.join(' + ');
        const modifier = result.modifier !== 0 
            ? ` ${result.modifier > 0 ? '+' : ''} ${result.modifier}` 
            : '';
        
        this.diceBreakdown.textContent = `${rolls}${modifier} = ${result.total}`;

        // Details
        let details = `Individual rolls: [${result.rolls.join(', ')}]`;
        if (result.droppedRolls.length > 0) {
            details += ` (dropped: [${result.droppedRolls.join(', ')}])`;
        }
        if (result.isCritical) details += ' 🎉 CRITICAL HIT!';
        if (result.isCriticalFail) details += ' 💀 CRITICAL FAIL!';
        
        this.rollDetails.textContent = details;
    }

    addToHistory(notation, result) {
        const entry = {
            notation,
            result: result.total,
            rolls: result.rolls,
            timestamp: new Date().toLocaleTimeString()
        };

        this.history.unshift(entry);
        
        // Keep only last 50 rolls
        if (this.history.length > 50) {
            this.history = this.history.slice(0, 50);
        }

        localStorage.setItem('diceHistory', JSON.stringify(this.history));
        this.renderHistory();
    }

    renderHistory() {
        if (this.history.length === 0) {
            this.historyList.innerHTML = '<p style="text-align: center; color: #999; padding: 20px;">No rolls yet. Start rolling!</p>';
            return;
        }

        this.historyList.innerHTML = this.history.map(entry => `
            <div class="history-item">
                <div>
                    <div class="history-notation">${entry.notation}</div>
                    <div class="history-time">${entry.timestamp}</div>
                </div>
                <div class="history-result">${entry.result}</div>
            </div>
        `).join('');
    }
}

// Initialize the dice roller
const roller = new DiceRoller();
```

## Technology Stack
- HTML5
- CSS3 (Animations, Grid, Flexbox)
- Vanilla JavaScript
- localStorage
- Regular Expressions

## Testing Checklist
- [ ] All dice types roll correctly (d4-d100)
- [ ] Modifiers add/subtract properly
- [ ] Notation parser handles complex formats
- [ ] Critical hits detected on d20 rolls
- [ ] Roll history saves and persists
- [ ] Animation plays smoothly
- [ ] Mobile responsive
- [ ] Keep highest/lowest works (if implemented)

## Additional Challenges

### Easy
- Add dice roll sound effects
- Include advantage/disadvantage buttons
- Create roll templates for common checks

### Medium
- Implement exploding dice (reroll on max)
- Add dice probability calculator
- Create multi-user session mode
- Build 3D dice animation with CSS

### Hard
- Implement physics-based 3D dice with Three.js
- Add voice command "roll 2d6"
- Create macro system for complex rolls
- Build campaign dice statistics tracker

## Resources
- [Dice Notation Guide](https://en.wikipedia.org/wiki/Dice_notation)
- [D&D 5e SRD](https://dnd.wizards.com/resources/systems-reference-document)
- [JavaScript Random](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
