# Coin Flip Simulator

<p align="center">
  <img src="https://images.unsplash.com/photo-1609429019995-8c40f49535a5?w=600&q=80" alt="Coin Flip" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 2-4 hours  
**Prerequisites:** Basic programming knowledge, HTML/CSS basics (for web version)

---

## Description

Build a coin flip simulator that generates random heads or tails results with smooth animations. This project teaches you about random number generation, animations, state management, and basic statistics tracking.

---

## Core Features

### Must Have
- Button to initiate coin flip
- Generate random result (heads or tails)
- Display result clearly after flip
- Keep track of flip history
- Show statistics (total flips, heads count, tails count)

### Nice to Have
- Smooth flip animation
- Sound effects for coin flip
- Visual coin image that rotates
- Reset statistics button
- Percentage display for heads/tails ratio

---

## Learning Objectives

By building this project, you will learn:

- **Random Number Generation**: Using Math.random() or equivalent
- **Animations**: CSS transitions or JavaScript animations
- **State Management**: Tracking application data over time
- **Event Handling**: Responding to user button clicks
- **Statistics**: Calculating percentages and ratios
- **DOM Manipulation**: Updating UI elements dynamically

---

## Implementation Guide

### Step 1: Set Up the Project Structure

Create the basic files for your application:

**For Web (HTML/CSS/JavaScript):**
```
coin-flip-simulator/
├── index.html
├── style.css
└── script.js
```

**For Python (GUI):**
```
coin-flip-simulator/
├── main.py
└── README.md
```

### Step 2: Design the User Interface

**Key UI Elements:**
1. Coin display area (image or div)
2. "Flip Coin" button
3. Result display (Heads or Tails)
4. Statistics panel showing:
   - Total flips
   - Heads count and percentage
   - Tails count and percentage
5. Reset button

### Step 3: Implement Random Logic

**Core Algorithm:**
```javascript
function flipCoin() {
  // Generate random number: 0 or 1
  const result = Math.random() < 0.5 ? 'heads' : 'tails';
  return result;
}
```

### Step 4: Add Animation

**CSS Animation Example:**
```css
@keyframes flip {
  0% { transform: rotateY(0); }
  100% { transform: rotateY(1800deg); }
}

.coin.flipping {
  animation: flip 1s ease-in-out;
}
```

### Step 5: Track Statistics

Keep an array or object to store all flip results:
- Count total flips
- Calculate heads percentage: (heads / total) × 100
- Calculate tails percentage: (tails / total) × 100

---

## Technology Suggestions

### Web Development
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: CSS animations, Flexbox for layout
- **Optional**: Canvas API for custom coin graphics

### Python
- **GUI Library**: Tkinter or PyQt5
- **Graphics**: PIL/Pillow for coin images
- **Alternative**: Pygame for animated version

### Mobile
- **React Native**: For cross-platform mobile app
- **Flutter**: For smooth animations

---

## Example Code Snippet

### JavaScript (Complete Example)
```javascript
let stats = { heads: 0, tails: 0 };

function flipCoin() {
  const coin = document.querySelector('.coin');
  const result = Math.random() < 0.5 ? 'heads' : 'tails';
  
  // Add flipping animation
  coin.classList.add('flipping');
  
  setTimeout(() => {
    // Update result after animation
    coin.classList.remove('flipping');
    coin.textContent = result.toUpperCase();
    
    // Update statistics
    stats[result]++;
    updateStats();
  }, 1000);
}

function updateStats() {
  const total = stats.heads + stats.tails;
  const headsPercent = ((stats.heads / total) * 100).toFixed(1);
  const tailsPercent = ((stats.tails / total) * 100).toFixed(1);
  
  document.getElementById('total').textContent = total;
  document.getElementById('heads').textContent = `${stats.heads} (${headsPercent}%)`;
  document.getElementById('tails').textContent = `${stats.tails} (${tailsPercent}%)`;
}
```

### Python (Tkinter)
```python
import tkinter as tk
import random

class CoinFlipSimulator:
    def __init__(self, root):
        self.root = root
        self.stats = {'heads': 0, 'tails': 0}
        
        self.result_label = tk.Label(root, text="Click to Flip", font=('Arial', 24))
        self.result_label.pack(pady=20)
        
        flip_button = tk.Button(root, text="Flip Coin", command=self.flip_coin)
        flip_button.pack()
        
        self.stats_label = tk.Label(root, text="Statistics: 0 flips")
        self.stats_label.pack(pady=10)
    
    def flip_coin(self):
        result = random.choice(['heads', 'tails'])
        self.stats[result] += 1
        
        self.result_label.config(text=result.upper())
        self.update_stats()
    
    def update_stats(self):
        total = sum(self.stats.values())
        text = f"Total: {total} | Heads: {self.stats['heads']} | Tails: {self.stats['tails']}"
        self.stats_label.config(text=text)

root = tk.Tk()
root.title("Coin Flip Simulator")
app = CoinFlipSimulator(root)
root.mainloop()
```

---

## Testing Checklist

- [ ] Coin flip generates random results
- [ ] Statistics update correctly after each flip
- [ ] Percentages calculate accurately
- [ ] Animation runs smoothly without glitches
- [ ] Reset button clears all statistics
- [ ] Works consistently over 100+ flips
- [ ] Results are truly random (approximately 50/50 distribution)

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
- Add sound effects when coin is flipped
- Use actual coin images (heads/tails sides)
- Add a "Flip 10 times" button for multiple flips
- Show last 10 flip results in a list

### Medium
- Create 3D coin flip animation using CSS transforms
- Add a probability chart showing expected vs actual distribution
- Implement different coin types (penny, quarter, euro, etc.)
- Add streak counter (longest heads/tails streak)

### Hard
- Build a betting game where user predicts the outcome
- Create a "fairness test" that checks for true randomness
- Add multiplayer mode (compete to predict results)
- Implement physics-based coin flip simulation
- Create slow-motion replay feature

---

## Common Pitfalls

**Poor randomness implementation**: Avoid using simple modulo operations; use proper random functions  
**Animation timing issues**: Ensure result displays after animation completes  
**State management bugs**: Reset statistics properly when clearing  
**Division by zero**: Handle the case when no flips have been made yet  

---

## Resources

### Documentation
- [Math.random() (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)
- [CSS Animations Guide](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations)
- [Python Random Module](https://docs.python.org/3/library/random.html)

### Tutorials
- [Creating Flip Animations with CSS](https://css-tricks.com/almanac/properties/a/animation/)
- [JavaScript Event Handling](https://javascript.info/introduction-browser-events)
- [Understanding Probability in Programming](https://www.khanacademy.org/computing/computer-science/algorithms/random-algorithms/a/randomized-algorithms)

### Design Inspiration
- [Coin Flip Games on CodePen](https://codepen.io/search/pens?q=coin+flip)
- [Google Coin Flip](https://www.google.com/search?q=flip+a+coin)

---

## Submission Checklist

Before considering your project complete:

- [ ] Code is well-commented and organized
- [ ] Animation runs smoothly
- [ ] Statistics are accurate and update in real-time
- [ ] UI is clean and intuitive
- [ ] Project has been tested with multiple flips
- [ ] README documentation is complete

---

## Portfolio Tips

When showcasing this project:

1. **Demo Video**: Record the coin flip animation in action
2. **Live Demo**: Deploy to GitHub Pages or Netlify
3. **Statistics Focus**: Show the probability distribution over 1000+ flips
4. **Code Quality**: Highlight clean, modular code structure

---

## License

This project idea is part of the [Project Ideas Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
