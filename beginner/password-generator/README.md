# Secure Password Generator

<p align="center">
  <img src="https://images.unsplash.com/photo-1614064641938-3bbee52942c7?w=600&q=80" alt="Password Security" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 2-4 hours  
**Prerequisites:** Basic programming knowledge, string manipulation

---

## Description

Build a secure password generator that creates random passwords with customizable options. This project teaches you about randomization, character sets, string manipulation, and security best practices for password creation.

---

## Core Features

### Must Have
- Set password length (minimum 8 characters)
- Include/exclude uppercase letters (A-Z)
- Include/exclude lowercase letters (a-z)
- Include/exclude numbers (0-9)
- Include/exclude special symbols (!@#$%^&*)
- Generate button to create password
- Display generated password clearly

### Nice to Have
- Copy to clipboard button
- Password strength indicator
- Generate multiple passwords at once
- Save password history
- Exclude ambiguous characters (0, O, l, 1, I)

---

## Learning Objectives

By building this project, you will learn:

- **Randomization**: Using random functions to generate unpredictable values
- **String Manipulation**: Building strings from character sets
- **User Input Validation**: Ensuring valid length and options
- **Arrays/Lists**: Working with character collections
- **Conditional Logic**: Including/excluding character types based on user choice
- **Clipboard API**: Copying text programmatically

---

## Implementation Guide

### Step 1: Set Up the Project Structure

Create the basic files for your application:

**For Web (HTML/CSS/JavaScript):**
```
password-generator/
├── index.html
├── style.css
└── script.js
```

**For Python:**
```
password-generator/
├── main.py
└── README.md
```

### Step 2: Define Character Sets

Create constants for different character types:

```javascript
const UPPERCASE = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
const LOWERCASE = 'abcdefghijklmnopqrstuvwxyz';
const NUMBERS = '0123456789';
const SYMBOLS = '!@#$%^&*()_+-=[]{}|;:,.<>?';
```

### Step 3: Implement Core Generation Logic

**Algorithm:**
1. Combine selected character sets into one string
2. Loop for desired password length
3. Randomly select characters from combined set
4. Return generated password

```javascript
function generatePassword(length, options) {
  let charset = '';
  if (options.uppercase) charset += UPPERCASE;
  if (options.lowercase) charset += LOWERCASE;
  if (options.numbers) charset += NUMBERS;
  if (options.symbols) charset += SYMBOLS;
  
  let password = '';
  for (let i = 0; i < length; i++) {
    const randomIndex = Math.floor(Math.random() * charset.length);
    password += charset[randomIndex];
  }
  
  return password;
}
```

### Step 4: Add Password Strength Evaluation

Evaluate password strength based on:
- Length (8-12: Weak, 13-16: Medium, 17+: Strong)
- Character diversity (more types = stronger)
- Presence of all character types

### Step 5: Implement Copy to Clipboard

```javascript
function copyToClipboard(text) {
  navigator.clipboard.writeText(text)
    .then(() => alert('Password copied!'))
    .catch(err => console.error('Failed to copy:', err));
}
```

---

## Technology Suggestions

### Web Development
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: Modern UI with gradient backgrounds
- **Clipboard**: Navigator Clipboard API

### Python
- **Libraries**: `random`, `string` (built-in)
- **GUI**: Tkinter or PyQt5
- **CLI**: Simple command-line version with argparse

### Mobile
- **React Native**: Cross-platform password generator
- **Flutter**: Beautiful native UI

---

## Example Code Snippet

### JavaScript (Complete Example)
```javascript
class PasswordGenerator {
  constructor() {
    this.uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    this.lowercase = 'abcdefghijklmnopqrstuvwxyz';
    this.numbers = '0123456789';
    this.symbols = '!@#$%^&*()_+-=[]{}|;:,.<>?';
  }

  generate(length, options) {
    let charset = '';
    if (options.useUppercase) charset += this.uppercase;
    if (options.useLowercase) charset += this.lowercase;
    if (options.useNumbers) charset += this.numbers;
    if (options.useSymbols) charset += this.symbols;

    if (charset === '') {
      throw new Error('Select at least one character type');
    }

    let password = '';
    for (let i = 0; i < length; i++) {
      const randomIndex = Math.floor(Math.random() * charset.length);
      password += charset[randomIndex];
    }

    return password;
  }

  evaluateStrength(password) {
    const length = password.length;
    const hasUpper = /[A-Z]/.test(password);
    const hasLower = /[a-z]/.test(password);
    const hasNumber = /[0-9]/.test(password);
    const hasSymbol = /[^A-Za-z0-9]/.test(password);

    const score = (hasUpper ? 1 : 0) + (hasLower ? 1 : 0) + 
                  (hasNumber ? 1 : 0) + (hasSymbol ? 1 : 0);

    if (length < 8) return 'Too Short';
    if (length < 12 || score < 3) return 'Weak';
    if (length < 16 || score < 4) return 'Medium';
    return 'Strong';
  }
}

// Usage
const generator = new PasswordGenerator();
const password = generator.generate(16, {
  useUppercase: true,
  useLowercase: true,
  useNumbers: true,
  useSymbols: true
});
console.log(password); // e.g., "aB3#xY9@kL2$mN5!"
```

### Python
```python
import random
import string

class PasswordGenerator:
    def __init__(self):
        self.uppercase = string.ascii_uppercase
        self.lowercase = string.ascii_lowercase
        self.numbers = string.digits
        self.symbols = "!@#$%^&*()_+-=[]{}|;:,.<>?"
    
    def generate(self, length, use_uppercase=True, use_lowercase=True, 
                 use_numbers=True, use_symbols=True):
        charset = ''
        if use_uppercase:
            charset += self.uppercase
        if use_lowercase:
            charset += self.lowercase
        if use_numbers:
            charset += self.numbers
        if use_symbols:
            charset += self.symbols
        
        if not charset:
            raise ValueError("Select at least one character type")
        
        password = ''.join(random.choice(charset) for _ in range(length))
        return password
    
    def evaluate_strength(self, password):
        length = len(password)
        has_upper = any(c.isupper() for c in password)
        has_lower = any(c.islower() for c in password)
        has_number = any(c.isdigit() for c in password)
        has_symbol = any(c in self.symbols for c in password)
        
        score = sum([has_upper, has_lower, has_number, has_symbol])
        
        if length < 8:
            return "Too Short"
        elif length < 12 or score < 3:
            return "Weak"
        elif length < 16 or score < 4:
            return "Medium"
        else:
            return "Strong"

# Usage
generator = PasswordGenerator()
password = generator.generate(16)
print(f"Password: {password}")
print(f"Strength: {generator.evaluate_strength(password)}")
```

---

## Testing Checklist

- [ ] Generates passwords of specified length
- [ ] Respects character type selections
- [ ] Produces truly random passwords (no patterns)
- [ ] Handles edge cases (minimum length, no options selected)
- [ ] Copy to clipboard works correctly
- [ ] Strength indicator shows accurate assessment
- [ ] Excludes ambiguous characters when selected

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
- Add "Exclude ambiguous characters" option (0, O, l, 1, I)
- Create password strength meter with visual bar
- Add preset buttons (Weak/Medium/Strong)
- Show character count as user adjusts length

### Medium
- Implement passphrase generator (e.g., "correct-horse-battery-staple")
- Add password history with ability to regenerate
- Create memorable password option (pronounceable)
- Add entropy calculation and display
- Implement custom character set input

### Hard
- Build password manager integration
- Add multi-language support for passphrases
- Implement HIBP (Have I Been Pwned) API check
- Create browser extension version
- Add password comparison tool
- Implement secure password hashing preview

---

## Common Pitfalls

**Weak randomness**: Use cryptographically secure random functions (e.g., `crypto.getRandomValues()`)  
**Predictable patterns**: Ensure true randomization without sequential patterns  
**Too short passwords**: Enforce minimum length of 8 characters  
**No character diversity**: Require at least 2 character types for security  

---

## Resources

### Documentation
- [Math.random() (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)
- [Crypto.getRandomValues()](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues)
- [Python Random Module](https://docs.python.org/3/library/random.html)
- [Python Secrets Module](https://docs.python.org/3/library/secrets.html)

### Tutorials
- [Password Security Best Practices](https://owasp.org/www-community/password-special-characters)
- [How to Generate Secure Random Numbers](https://stackoverflow.com/questions/5651789/how-to-generate-cryptographically-secure-random-numbers-in-javascript)
- [Clipboard API Guide](https://web.dev/async-clipboard/)

### Design Inspiration
- [1Password Generator](https://1password.com/password-generator/)
- [LastPass Generator](https://www.lastpass.com/features/password-generator)

---

## Submission Checklist

Before considering your project complete:

- [ ] Passwords are truly random and secure
- [ ] All character options work correctly
- [ ] UI is intuitive and easy to use
- [ ] Copy to clipboard function works
- [ ] Code is well-commented
- [ ] Project has been tested with various configurations

---

## Portfolio Tips

When showcasing this project:

1. **Security Focus**: Emphasize cryptographic randomness
2. **Demo Video**: Show generating various password types
3. **Live Demo**: Deploy interactive version online
4. **Code Quality**: Highlight clean, modular code

---

## License

This project idea is part of the [Project Ideas Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
