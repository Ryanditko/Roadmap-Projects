# Simple Calculator

<p align="center">
  <img src="https://images.unsplash.com/photo-1587145820266-a5951ee6f620?w=600&q=80" alt="Calculator" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 3-5 hours  
**Prerequisites:** Basic programming knowledge, understanding of arithmetic operations

---

## Description

Build a functional calculator application that performs fundamental mathematical operations. This project teaches you about event handling, input validation, and creating interactive user interfaces. Perfect for understanding how to process user input and display calculated results.

---

## Core Features

### Must Have
- Basic operations: addition (+), subtraction (-), multiplication (×), division (÷)
- Number input buttons (0-9)
- Decimal point support
- Clear/Reset button
- Display screen showing current input and result
- Equals button to calculate result

### Nice to Have
- Backspace/Delete button
- Keyboard input support
- Calculation history
- Memory functions (M+, M-, MR, MC)
- Percentage calculation

---

## Learning Objectives

By building this project, you will learn:

- **Event Handling**: Responding to button clicks and keyboard input
- **Input Validation**: Ensuring valid mathematical expressions
- **Error Handling**: Managing division by zero and invalid operations
- **State Management**: Tracking current operation and operands
- **UI/UX Design**: Creating an intuitive calculator layout
- **String Manipulation**: Parsing and evaluating expressions

---

## Implementation Guide

### Step 1: Set Up the Project Structure

Create the basic files for your application:

**For Web (HTML/CSS/JavaScript):**
```
simple-calculator/
├── index.html
├── style.css
└── script.js
```

**For Python:**
```
simple-calculator/
├── main.py
└── README.md
```

### Step 2: Design the Calculator Layout

**Standard Calculator Layout:**
```
┌─────────────────┐
│   Display       │
├───┬───┬───┬─────┤
│ 7 │ 8 │ 9 │  ÷  │
├───┼───┼───┼─────┤
│ 4 │ 5 │ 6 │  ×  │
├───┼───┼───┼─────┤
│ 1 │ 2 │ 3 │  -  │
├───┼───┼───┼─────┤
│ 0 │ . │ = │  +  │
└───┴───┴───┴─────┘
```

### Step 3: Implement Core Logic

**Key Functions:**
1. **appendNumber(num)**: Add digit to display
2. **setOperation(op)**: Store the operation
3. **calculate()**: Perform the calculation
4. **clear()**: Reset calculator state
5. **delete()**: Remove last digit

**Calculation Logic:**
```javascript
function calculate(num1, operator, num2) {
  switch(operator) {
    case '+': return num1 + num2;
    case '-': return num1 - num2;
    case '*': return num1 * num2;
    case '/': return num2 !== 0 ? num1 / num2 : 'Error';
  }
}
```

### Step 4: Handle Edge Cases

- Division by zero → Display "Error"
- Multiple decimal points → Prevent second decimal
- Multiple operators in sequence → Use last operator
- Leading zeros → Remove unnecessary zeros

### Step 5: Add Polish

- Format large numbers with commas
- Limit decimal places to 8-10 digits
- Add smooth animations for button presses
- Implement keyboard shortcuts

---

## Technology Suggestions

### Web Development
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: CSS Grid for button layout
- **Optional**: React/Vue for component-based approach

### Python
- **GUI**: Tkinter (built-in), PyQt5, or Kivy
- **CLI**: Simple command-line version with input prompts

### Mobile
- **React Native**: Cross-platform calculator app
- **Flutter**: Native performance with beautiful UI

---

## Example Code Snippet

### JavaScript (Complete Example)
```javascript
class Calculator {
  constructor() {
    this.currentOperand = '';
    this.previousOperand = '';
    this.operation = null;
  }

  appendNumber(number) {
    if (number === '.' && this.currentOperand.includes('.')) return;
    this.currentOperand += number.toString();
  }

  chooseOperation(operation) {
    if (this.currentOperand === '') return;
    if (this.previousOperand !== '') {
      this.calculate();
    }
    this.operation = operation;
    this.previousOperand = this.currentOperand;
    this.currentOperand = '';
  }

  calculate() {
    let result;
    const prev = parseFloat(this.previousOperand);
    const current = parseFloat(this.currentOperand);
    
    if (isNaN(prev) || isNaN(current)) return;
    
    switch (this.operation) {
      case '+':
        result = prev + current;
        break;
      case '-':
        result = prev - current;
        break;
      case '*':
        result = prev * current;
        break;
      case '/':
        result = current === 0 ? 'Error' : prev / current;
        break;
      default:
        return;
    }
    
    this.currentOperand = result;
    this.operation = null;
    this.previousOperand = '';
  }

  clear() {
    this.currentOperand = '';
    this.previousOperand = '';
    this.operation = null;
  }

  delete() {
    this.currentOperand = this.currentOperand.toString().slice(0, -1);
  }
}
```

### Python (Tkinter)
```python
import tkinter as tk

class Calculator:
    def __init__(self, root):
        self.root = root
        self.root.title("Simple Calculator")
        self.expression = ""
        
        self.display = tk.Entry(root, font=('Arial', 20), bd=10, 
                               insertwidth=2, width=14, justify='right')
        self.display.grid(row=0, column=0, columnspan=4)
        
        self.create_buttons()
    
    def create_buttons(self):
        buttons = [
            '7', '8', '9', '/',
            '4', '5', '6', '*',
            '1', '2', '3', '-',
            'C', '0', '=', '+'
        ]
        
        row = 1
        col = 0
        
        for button in buttons:
            cmd = lambda x=button: self.on_button_click(x)
            tk.Button(self.root, text=button, width=5, height=2,
                     font=('Arial', 14), command=cmd).grid(row=row, column=col)
            col += 1
            if col > 3:
                col = 0
                row += 1
    
    def on_button_click(self, char):
        if char == 'C':
            self.expression = ""
        elif char == '=':
            try:
                self.expression = str(eval(self.expression))
            except:
                self.expression = "Error"
        else:
            self.expression += str(char)
        
        self.display.delete(0, tk.END)
        self.display.insert(0, self.expression)

root = tk.Tk()
calc = Calculator(root)
root.mainloop()
```

---

## Testing Checklist

- [ ] All basic operations work correctly (+, -, *, /)
- [ ] Division by zero shows error message
- [ ] Decimal numbers calculate properly
- [ ] Clear button resets calculator state
- [ ] Multiple operations in sequence work correctly
- [ ] Keyboard input is supported (if implemented)
- [ ] Large numbers display correctly
- [ ] Negative numbers are handled properly

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
- Add backspace button to delete last digit
- Implement percentage calculation
- Add keyboard support for all operations
- Show calculation history below calculator

### Medium
- Add memory functions (M+, M-, MR, MC)
- Implement order of operations (PEMDAS)
- Create scientific calculator mode (sin, cos, tan, log)
- Add themes (light/dark mode)

### Hard
- Support parentheses in expressions
- Create graphing calculator functionality
- Implement unit conversions (length, weight, temperature)
- Build a programmer calculator (binary, hex, oct)
- Add expression history with ability to reuse past calculations

---

## Common Pitfalls

**Floating-point precision errors**: Use proper rounding (e.g., `toFixed()`)  
**Handling multiple operators**: Clear previous operation when new one is selected  
**String vs Number operations**: Ensure proper type conversion  
**Empty input handling**: Validate before attempting calculations  

---

## Resources

### Documentation
- [JavaScript Math Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math)
- [CSS Grid Layout](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Python Tkinter Documentation](https://docs.python.org/3/library/tkinter.html)

### Tutorials
- [Build a Calculator with JavaScript](https://www.freecodecamp.org/news/how-to-build-an-html-calculator-app-from-scratch-using-javascript-4454b8714b98/)
- [Calculator Logic Explained](https://freshman.tech/calculator/)
- [Handling Keyboard Events](https://javascript.info/keyboard-events)

### Design Inspiration
- [iOS Calculator](https://www.apple.com/ios/calculator/)
- [Google Calculator](https://www.google.com/search?q=calculator)
- [Calculator Designs on Dribbble](https://dribbble.com/search/calculator)

---

## Submission Checklist

Before considering your project complete:

- [ ] All operations produce correct results
- [ ] Error states are handled gracefully
- [ ] UI is clean and easy to use
- [ ] Code is well-organized and commented
- [ ] Project has been tested with various inputs
- [ ] README documentation is complete

---

## Portfolio Tips

When showcasing this project:

1. **Demo Video**: Show complex calculations being performed
2. **Live Demo**: Deploy interactive version online
3. **Code Quality**: Demonstrate clean, modular code structure
4. **Feature Highlight**: Emphasize any advanced features (keyboard support, history, etc.)

---

## License

This project idea is part of the [Project Ideas Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
