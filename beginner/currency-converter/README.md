# Currency Converter

<p align="center">
  <img src="https://images.unsplash.com/photo-1621761191319-c6fb62004040?w=600&q=80" alt="Currency Exchange" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 3-5 hours  
**Prerequisites:** Basic programming knowledge, understanding of API calls

---

## Description

Build a currency converter application that allows users to convert monetary values between different currencies using real-time exchange rates from a public API. This project teaches you how to work with external APIs, handle asynchronous requests, and format numerical data.

---

## Core Features

### Must Have
-  Select source currency from dropdown list
-  Select target currency from dropdown list
-  Input field to enter the amount to convert
-  Display converted value with proper formatting
-  Fetch real-time exchange rates from an API
-  Error handling for API failures

### Nice to Have
-  Currency swap button (switch source and target)
-  Popular currency pairs as quick options
-  Last updated timestamp for exchange rates
-  Loading indicator while fetching rates

---

## Learning Objectives

By building this project, you will learn:

- **API Integration**: How to consume RESTful APIs
- **Async Programming**: Handling promises and async/await
- **Data Transformation**: Parsing and manipulating JSON data
- **Number Formatting**: Working with decimal precision and currency symbols
- **Error Handling**: Managing network failures gracefully
- **User Input Validation**: Ensuring valid numerical inputs

---

## Implementation Guide

### Step 1: Set Up the Project Structure

### Step 1: Set Up the Project Structure

Create the basic files for your application:

**For Web (HTML/CSS/JavaScript):**
```
currency-converter/
 index.html
 style.css
 script.js
```

**For Python:**
```
currency-converter/
 main.py
 requirements.txt
 README.md
```

### Step 2: Choose an Exchange Rate API

Select a free API to fetch exchange rates:

| API | Free Tier | Features | Registration |
|-----|-----------|----------|--------------|
| [ExchangeRate-API](https://www.exchangerate-api.com/) | 1,500 requests/month | Simple JSON response | Required |
| [Fixer.io](https://fixer.io/) | 100 requests/month | Extensive currency list | Required |
| [CurrencyAPI](https://currencyapi.com/) | 300 requests/month | Real-time rates | Required |
| [Open Exchange Rates](https://openexchangerates.org/) | 1,000 requests/month | Historical data | Required |

### Step 3: Implement Core Functionality

**Key Functions to Build:**

1. **fetchExchangeRates()**: Get current rates from API
2. **convertCurrency(amount, fromCurrency, toCurrency)**: Perform conversion calculation
3. **updateUI()**: Display results to user
4. **validateInput()**: Ensure amount is a valid number

**Conversion Formula:**
```
convertedAmount = amount × (targetRate / sourceRate)
```

### Step 4: Handle API Response

Example API response structure:
```json
{
  "base": "USD",
  "rates": {
    "EUR": 0.85,
    "GBP": 0.73,
    "JPY": 110.15,
    "BRL": 5.25
  }
}
```

### Step 5: Format Output

Display results with:
- 2 decimal places for most currencies
- Currency symbol or code
- Thousand separators for large numbers
- Example: `$1,000.00 USD = €850.00 EUR`

---

## Technology Suggestions

### Web Development
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: Bootstrap or Tailwind CSS (optional)
- **HTTP Requests**: Fetch API or Axios

### Python
- **Libraries**: `requests` for API calls, `tkinter` for GUI (optional)
- **Alternative**: Flask for web-based version

### JavaScript (Node.js)
- **Libraries**: `axios` or `node-fetch`
- **Framework**: Express.js for web server (optional)

---

## Example Code Snippet

### JavaScript (Fetch API)
```javascript
async function convertCurrency(amount, from, to) {
  const API_KEY = 'your_api_key_here';
  const url = `https://api.exchangerate-api.com/v4/latest/${from}`;
  
  try {
    const response = await fetch(url);
    const data = await response.json();
    const rate = data.rates[to];
    return (amount * rate).toFixed(2);
  } catch (error) {
    console.error('Error fetching exchange rates:', error);
    return null;
  }
}
```

### Python
```python
import requests

def convert_currency(amount, from_currency, to_currency):
    url = f"https://api.exchangerate-api.com/v4/latest/{from_currency}"
    try:
        response = requests.get(url)
        data = response.json()
        rate = data['rates'][to_currency]
        return round(amount * rate, 2)
    except Exception as e:
        print(f"Error: {e}")
        return None
```

---

## Testing Checklist

- [ ] Conversion works with different currency pairs
- [ ] Handles invalid input (negative numbers, letters)
- [ ] Shows error message when API is unavailable
- [ ] Displays loading state while fetching data
- [ ] Formats large numbers correctly (1,000,000)
- [ ] Works with decimal amounts (0.50, 10.99)
- [ ] Updates rates when page refreshes

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
-  Add a currency swap button to reverse source and target
-  Show the exchange rate being used (1 USD = 0.85 EUR)
-  Add more currency options (cryptocurrencies like BTC, ETH)

### Medium
-  Cache exchange rates locally to reduce API calls
-  Add a favorites system to save frequently used currency pairs
-  Implement offline mode with last known rates
-  Create a mobile-responsive design

### Hard
-  Display a chart showing exchange rate trends over time
-  Convert multiple currencies simultaneously (matrix view)
-  Add country flags next to currency codes
-  Implement voice input for amounts
-  Build a browser extension version

---

## Common Pitfalls

 **Not handling API rate limits**: Implement caching to avoid hitting API limits  
 **Ignoring floating-point precision**: Use proper rounding for currency calculations  
 **Missing error states**: Always handle network failures gracefully  
 **Hardcoding API keys**: Use environment variables for sensitive data  

---

## Resources

### Documentation
- [Fetch API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [Working with JSON](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)
- [Python Requests Library](https://requests.readthedocs.io/)

### Tutorials
- [REST API Tutorial](https://www.restapitutorial.com/)
- [Async/Await in JavaScript](https://javascript.info/async-await)
- [Currency Formatting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat)

### Design Inspiration
- [Dribbble: Currency Converters](https://dribbble.com/search/currency-converter)
- [Google Currency Converter](https://www.google.com/search?q=currency+converter)

---

## Submission Checklist

Before considering your project complete:

- [ ] Code is well-commented and organized
- [ ] Error handling is implemented
- [ ] UI is user-friendly and responsive
- [ ] README documentation is complete
- [ ] Project has been tested with various inputs
- [ ] API key is not exposed in public code

---

## Portfolio Tips

When showcasing this project:

1. **Demo Video**: Record a short video showing the conversion in action
2. **Live Demo**: Deploy to GitHub Pages, Netlify, or Vercel
3. **Write-Up**: Explain challenges you faced and how you solved them
4. **Code Quality**: Ensure code is clean and follows best practices

---

## License

This project idea is part of the [Project Ideas Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
