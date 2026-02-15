# Number Guessing Game

<p align="center">
  <img src="https://images.unsplash.com/photo-1516975080664-ed2fc6a32937?w=600&q=80" alt="Guessing Game" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 2-3 hours  
**Prerequisites:** Basic programming knowledge, loops and conditionals

---

## Description

Build an interactive number guessing game where the computer generates a random number and the player tries to guess it with helpful hints. This project teaches you about random number generation, loops, conditional logic, user input validation, and basic game mechanics.

---

## Core Features

### Must Have
- Generate random number within a defined range (e.g., 1-100)
- Accept user input for guesses
- Provide hints after each guess ("Too high" or "Too low")
- Track number of attempts
- Display win message when guessed correctly
- Option to play again

### Nice to Have
- Difficulty levels (Easy: 1-50, Medium: 1-100, Hard: 1-1000)
- Limited attempts based on difficulty
- High score tracking (fewest attempts)
- Timer to track how long it takes
- Hint system (reveal first/last digit, range narrowing)

---

## Learning Objectives

By building this project, you will learn:

- **Random Number Generation**: Using built-in random functions
- **Loop Structures**: While loops for repeated attempts
- **Conditional Logic**: If/else statements for game logic
- **Input Validation**: Ensuring user enters valid numbers
- **Game State Management**: Tracking attempts, score, game status
- **User Experience**: Providing clear feedback and hints

---

## Implementation Guide

### Step 1: Set Up the Project Structure

Create the basic files for your application:

**For Web (HTML/CSS/JavaScript):**
```
number-guessing-game/
├── index.html
├── style.css
└── script.js
```

**For Python:**
```
number-guessing-game/
├── main.py
└── README.md
```

### Step 2: Generate Random Number

Choose an appropriate range and generate the secret number:

```javascript
const min = 1;
const max = 100;
const secretNumber = Math.floor(Math.random() * (max - min + 1)) + min;
```

### Step 3: Implement Game Loop

Create a loop that continues until the player guesses correctly:

**Key Logic:**
1. Get user's guess
2. Validate input (is it a number? is it in range?)
3. Compare with secret number
4. Provide feedback (too high, too low, or correct)
5. Increment attempt counter
6. Check for win condition

### Step 4: Add Difficulty Levels

Create different difficulty settings:

| Difficulty | Range | Max Attempts |
|------------|-------|--------------|
| Easy | 1-50 | 10 attempts |
| Medium | 1-100 | 7 attempts |
| Hard | 1-1000 | 10 attempts |

### Step 5: Track Statistics

Keep track of:
- Current attempts
- Best score (lowest attempts)
- Win/loss ratio
- Average attempts per game

---

## Technology Suggestions

### Web Development
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: CSS animations for feedback messages
- **Optional**: Local Storage for high score persistence

### Python
- **Built-in**: `random` module
- **GUI**: Tkinter for graphical version
- **CLI**: Simple command-line with `input()`

### Other Languages
- **Java**: Scanner for input, Random class
- **C++**: `<random>` library, cin for input
- **C#**: Random class, Console.ReadLine()

---

## Example Code Snippet

### JavaScript (Complete Example)
```javascript
class GuessingGame {
  constructor(difficulty = 'medium') {
    this.difficulties = {
      easy: { min: 1, max: 50, maxAttempts: 10 },
      medium: { min: 1, max: 100, maxAttempts: 7 },
      hard: { min: 1, max: 1000, maxAttempts: 10 }
    };
    
    this.config = this.difficulties[difficulty];
    this.secretNumber = this.generateNumber();
    this.attempts = 0;
    this.guesses = [];
  }

  generateNumber() {
    const { min, max } = this.config;
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  makeGuess(guess) {
    this.attempts++;
    this.guesses.push(guess);

    if (guess < this.secretNumber) {
      return { status: 'low', message: 'Too low! Try higher.' };
    } else if (guess > this.secretNumber) {
      return { status: 'high', message: 'Too high! Try lower.' };
    } else {
      return { 
        status: 'win', 
        message: `Correct! You won in ${this.attempts} attempts!` 
      };
    }
  }

  getRemainingAttempts() {
    return this.config.maxAttempts - this.attempts;
  }

  reset(difficulty) {
    this.config = this.difficulties[difficulty];
    this.secretNumber = this.generateNumber();
    this.attempts = 0;
    this.guesses = [];
  }
}

// Usage
const game = new GuessingGame('medium');
const result = game.makeGuess(50);
console.log(result.message);
```

### Python
```python
import random

class GuessingGame:
    def __init__(self, difficulty='medium'):
        self.difficulties = {
            'easy': {'min': 1, 'max': 50, 'max_attempts': 10},
            'medium': {'min': 1, 'max': 100, 'max_attempts': 7},
            'hard': {'min': 1, 'max': 1000, 'max_attempts': 10}
        }
        
        self.config = self.difficulties[difficulty]
        self.secret_number = self.generate_number()
        self.attempts = 0
        self.guesses = []
    
    def generate_number(self):
        return random.randint(self.config['min'], self.config['max'])
    
    def make_guess(self, guess):
        self.attempts += 1
        self.guesses.append(guess)
        
        if guess < self.secret_number:
            return {'status': 'low', 'message': 'Too low! Try higher.'}
        elif guess > self.secret_number:
            return {'status': 'high', 'message': 'Too high! Try lower.'}
        else:
            return {
                'status': 'win',
                'message': f'Correct! You won in {self.attempts} attempts!'
            }
    
    def get_remaining_attempts(self):
        return self.config['max_attempts'] - self.attempts
    
    def reset(self, difficulty='medium'):
        self.config = self.difficulties[difficulty]
        self.secret_number = self.generate_number()
        self.attempts = 0
        self.guesses = []

# Usage
def play_game():
    game = GuessingGame('medium')
    print(f"Guess a number between {game.config['min']} and {game.config['max']}")
    
    while game.get_remaining_attempts() > 0:
        try:
            guess = int(input("Your guess: "))
            result = game.make_guess(guess)
            print(result['message'])
            
            if result['status'] == 'win':
                break
                
            print(f"Remaining attempts: {game.get_remaining_attempts()}")
        except ValueError:
            print("Please enter a valid number")
    
    if game.get_remaining_attempts() == 0:
        print(f"Game over! The number was {game.secret_number}")

if __name__ == '__main__':
    play_game()
```

---

## Testing Checklist

- [ ] Random number generates within correct range
- [ ] "Too high" hint works correctly
- [ ] "Too low" hint works correctly
- [ ] Win condition triggers on correct guess
- [ ] Attempt counter increments properly
- [ ] Invalid input is handled gracefully
- [ ] Play again functionality works
- [ ] High score saves correctly (if implemented)

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
- Add sound effects for correct/incorrect guesses
- Show guess history in a list
- Display a progress bar showing range narrowing
- Add "Give Up" button that reveals the number

### Medium
- Implement hint system (costs attempts but reveals digit)
- Create two-player mode (players alternate)
- Add timer and show how long each game took
- Implement binary search hint system
- Show statistical probability after each guess

### Hard
- Create AI opponent that plays optimally using binary search
- Implement reverse mode (computer guesses your number)
- Add leaderboard with player names and scores
- Create tournament mode with multiple rounds
- Implement adaptive difficulty (adjusts based on performance)

---

## Common Pitfalls

**Not validating input**: Always check if input is a number and within range  
**Off-by-one errors**: Ensure random range includes both min and max values  
**Infinite loops**: Make sure loop has a clear exit condition  
**Poor user feedback**: Always tell the user what happened and what to do next  

---

## Resources

### Documentation
- [Math.random() (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)
- [Python Random Module](https://docs.python.org/3/library/random.html)
- [Input Validation Best Practices](https://owasp.org/www-community/vulnerabilities/Improper_Input_Validation)

### Tutorials
- [JavaScript Loops Explained](https://javascript.info/while-for)
- [Random Number Generation](https://www.freecodecamp.org/news/random-number-generator/)
- [Game Logic Patterns](https://gameprogrammingpatterns.com/)

### Design Inspiration
- [Google Number Game](https://www.google.com/search?q=number+guessing+game)
- [Code.org Number Guessing](https://studio.code.org/)

---

## Submission Checklist

Before considering your project complete:

- [ ] Game logic works correctly for all scenarios
- [ ] Input validation prevents crashes
- [ ] User feedback is clear and helpful
- [ ] Code is well-commented and organized
- [ ] Project has been tested with edge cases
- [ ] README documentation is complete

---

## Portfolio Tips

When showcasing this project:

1. **Demo Video**: Show a complete game from start to finish
2. **Live Demo**: Deploy an interactive version online
3. **Statistics**: Show win rate and average attempts data
4. **Code Quality**: Highlight clean, modular code structure

---

## License

This project idea is part of the [Project Ideas Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
