# Word Counter - Text Analysis Tool

<p align="center">
  <img src="https://images.unsplash.com/photo-1455390582262-044cdead277a?w=600&q=80" alt="Word Counter" width="600">
</p>

**Level:** Beginner | **Time:** 2-4 hours | **Prerequisites:** HTML, CSS, JavaScript basics

## Description

Build a comprehensive text analysis tool that counts words, characters, sentences, and paragraphs in real-time. This project teaches DOM manipulation, string processing, regex patterns, and creating responsive interfaces.

## Core Features

### Must Have
- Real-time word count
- Character count (with/without spaces)
- Sentence counter
- Paragraph counter
- Reading time estimation
- Copy/paste functionality
- Clear text button

### Nice to Have
- Most used words analysis
- Keyword density calculator
- Character frequency chart
- Export statistics as PDF/CSV
- Text case converter (uppercase/lowercase/title)
- Find and replace functionality
- Dark mode toggle
- Word frequency visualization

## Learning Objectives

- **String Manipulation**: Learn text processing techniques
- **Regular Expressions**: Master pattern matching for text analysis
- **Event Listeners**: Handle input events in real-time
- **DOM Updates**: Efficiently update multiple UI elements
- **Data Visualization**: Display statistics effectively

## Implementation Guide

### Step 1: HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Word Counter</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>📝 Word Counter</h1>
            <p>Analyze your text in real-time</p>
        </header>

        <div class="editor-section">
            <textarea 
                id="textInput" 
                placeholder="Start typing or paste your text here..."
                autofocus
            ></textarea>
            
            <div class="controls">
                <button id="clearBtn" class="btn btn-danger">Clear Text</button>
                <button id="copyBtn" class="btn btn-secondary">Copy Stats</button>
                <button id="pasteBtn" class="btn btn-secondary">Paste Text</button>
            </div>
        </div>

        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-value" id="wordCount">0</div>
                <div class="stat-label">Words</div>
            </div>
            <div class="stat-card">
                <div class="stat-value" id="charCount">0</div>
                <div class="stat-label">Characters</div>
            </div>
            <div class="stat-card">
                <div class="stat-value" id="charNoSpace">0</div>
                <div class="stat-label">Characters (no spaces)</div>
            </div>
            <div class="stat-card">
                <div class="stat-value" id="sentenceCount">0</div>
                <div class="stat-label">Sentences</div>
            </div>
            <div class="stat-card">
                <div class="stat-value" id="paragraphCount">0</div>
                <div class="stat-label">Paragraphs</div>
            </div>
            <div class="stat-card">
                <div class="stat-value" id="readingTime">0 min</div>
                <div class="stat-label">Reading Time</div>
            </div>
        </div>

        <div class="details-section">
            <h3>Detailed Analysis</h3>
            <div class="details-grid">
                <div class="detail-item">
                    <span class="detail-label">Average Word Length:</span>
                    <span class="detail-value" id="avgWordLength">0</span>
                </div>
                <div class="detail-item">
                    <span class="detail-label">Longest Word:</span>
                    <span class="detail-value" id="longestWord">-</span>
                </div>
                <div class="detail-item">
                    <span class="detail-label">Speaking Time (slow):</span>
                    <span class="detail-value" id="speakTimeSlow">0 min</span>
                </div>
                <div class="detail-item">
                    <span class="detail-label">Speaking Time (fast):</span>
                    <span class="detail-value" id="speakTimeFast">0 min</span>
                </div>
            </div>
        </div>

        <div class="top-words-section">
            <h3>Most Frequent Words</h3>
            <div id="topWords" class="word-list"></div>
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
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    background: white;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

header {
    text-align: center;
    margin-bottom: 40px;
}

header h1 {
    font-size: 2.5rem;
    color: #333;
    margin-bottom: 10px;
}

header p {
    color: #666;
    font-size: 1.1rem;
}

.editor-section {
    margin-bottom: 30px;
}

#textInput {
    width: 100%;
    min-height: 300px;
    padding: 20px;
    border: 2px solid #e0e0e0;
    border-radius: 10px;
    font-size: 16px;
    font-family: 'Courier New', monospace;
    resize: vertical;
    transition: border-color 0.3s;
}

#textInput:focus {
    outline: none;
    border-color: #667eea;
}

.controls {
    display: flex;
    gap: 10px;
    margin-top: 15px;
}

.btn {
    padding: 12px 24px;
    border: none;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.btn-danger {
    background: #e74c3c;
    color: white;
}

.btn-danger:hover {
    background: #c0392b;
    transform: translateY(-2px);
}

.btn-secondary {
    background: #95a5a6;
    color: white;
}

.btn-secondary:hover {
    background: #7f8c8d;
    transform: translateY(-2px);
}

.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
}

.stat-card {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 25px;
    border-radius: 15px;
    text-align: center;
    transition: transform 0.3s;
}

.stat-card:hover {
    transform: translateY(-5px);
}

.stat-value {
    font-size: 2.5rem;
    font-weight: bold;
    margin-bottom: 8px;
}

.stat-label {
    font-size: 0.9rem;
    opacity: 0.9;
}

.details-section, .top-words-section {
    background: #f8f9fa;
    padding: 25px;
    border-radius: 15px;
    margin-bottom: 20px;
}

.details-section h3, .top-words-section h3 {
    margin-bottom: 20px;
    color: #333;
}

.details-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 15px;
}

.detail-item {
    display: flex;
    justify-content: space-between;
    padding: 10px;
    background: white;
    border-radius: 8px;
}

.detail-label {
    font-weight: 600;
    color: #555;
}

.detail-value {
    color: #667eea;
    font-weight: bold;
}

.word-list {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}

.word-item {
    background: white;
    padding: 8px 15px;
    border-radius: 20px;
    font-size: 14px;
    border: 2px solid #667eea;
    color: #667eea;
}

@media (max-width: 768px) {
    .container {
        padding: 20px;
    }
    
    header h1 {
        font-size: 2rem;
    }
    
    .stats-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}
```

### Step 3: JavaScript Implementation

```javascript
class WordCounter {
    constructor() {
        this.textInput = document.getElementById('textInput');
        this.initializeElements();
        this.attachEventListeners();
    }

    initializeElements() {
        this.elements = {
            wordCount: document.getElementById('wordCount'),
            charCount: document.getElementById('charCount'),
            charNoSpace: document.getElementById('charNoSpace'),
            sentenceCount: document.getElementById('sentenceCount'),
            paragraphCount: document.getElementById('paragraphCount'),
            readingTime: document.getElementById('readingTime'),
            avgWordLength: document.getElementById('avgWordLength'),
            longestWord: document.getElementById('longestWord'),
            speakTimeSlow: document.getElementById('speakTimeSlow'),
            speakTimeFast: document.getElementById('speakTimeFast'),
            topWords: document.getElementById('topWords')
        };
    }

    attachEventListeners() {
        this.textInput.addEventListener('input', () => this.analyzeText());
        document.getElementById('clearBtn').addEventListener('click', () => this.clearText());
        document.getElementById('copyBtn').addEventListener('click', () => this.copyStats());
        document.getElementById('pasteBtn').addEventListener('click', () => this.pasteText());
    }

    analyzeText() {
        const text = this.textInput.value;
        
        // Basic counts
        const words = this.countWords(text);
        const characters = text.length;
        const charactersNoSpace = text.replace(/\s/g, '').length;
        const sentences = this.countSentences(text);
        const paragraphs = this.countParagraphs(text);
        
        // Time calculations (average reading speed: 200 words/min)
        const readingTime = Math.ceil(words / 200);
        const speakTimeSlow = Math.ceil(words / 100); // 100 words/min
        const speakTimeFast = Math.ceil(words / 160); // 160 words/min
        
        // Word analysis
        const wordArray = this.getWordArray(text);
        const avgLength = wordArray.length > 0 
            ? (wordArray.reduce((sum, word) => sum + word.length, 0) / wordArray.length).toFixed(1)
            : 0;
        const longestWord = this.getLongestWord(wordArray);
        
        // Update UI
        this.updateStats({
            words,
            characters,
            charactersNoSpace,
            sentences,
            paragraphs,
            readingTime,
            avgLength,
            longestWord,
            speakTimeSlow,
            speakTimeFast
        });
        
        // Update top words
        this.updateTopWords(wordArray);
    }

    countWords(text) {
        const trimmed = text.trim();
        if (trimmed === '') return 0;
        return trimmed.split(/\s+/).length;
    }

    countSentences(text) {
        if (text.trim() === '') return 0;
        const sentences = text.match(/[.!?]+/g);
        return sentences ? sentences.length : 0;
    }

    countParagraphs(text) {
        if (text.trim() === '') return 0;
        const paragraphs = text.split(/\n\n+/).filter(p => p.trim() !== '');
        return paragraphs.length;
    }

    getWordArray(text) {
        if (text.trim() === '') return [];
        return text.toLowerCase()
            .replace(/[^\w\s]/g, '')
            .split(/\s+/)
            .filter(word => word.length > 0);
    }

    getLongestWord(wordArray) {
        if (wordArray.length === 0) return '-';
        return wordArray.reduce((longest, word) => 
            word.length > longest.length ? word : longest
        );
    }

    updateStats(stats) {
        this.elements.wordCount.textContent = stats.words;
        this.elements.charCount.textContent = stats.characters;
        this.elements.charNoSpace.textContent = stats.charactersNoSpace;
        this.elements.sentenceCount.textContent = stats.sentences;
        this.elements.paragraphCount.textContent = stats.paragraphs;
        this.elements.readingTime.textContent = `${stats.readingTime} min`;
        this.elements.avgWordLength.textContent = stats.avgLength;
        this.elements.longestWord.textContent = stats.longestWord;
        this.elements.speakTimeSlow.textContent = `${stats.speakTimeSlow} min`;
        this.elements.speakTimeFast.textContent = `${stats.speakTimeFast} min`;
    }

    updateTopWords(wordArray) {
        const frequency = {};
        wordArray.forEach(word => {
            if (word.length > 3) { // Only words with more than 3 characters
                frequency[word] = (frequency[word] || 0) + 1;
            }
        });

        const sorted = Object.entries(frequency)
            .sort((a, b) => b[1] - a[1])
            .slice(0, 10);

        this.elements.topWords.innerHTML = sorted.length > 0
            ? sorted.map(([word, count]) => 
                `<div class="word-item">${word} (${count})</div>`
              ).join('')
            : '<p style="color: #999;">Type some text to see word frequency</p>';
    }

    clearText() {
        if (confirm('Are you sure you want to clear all text?')) {
            this.textInput.value = '';
            this.analyzeText();
        }
    }

    async copyStats() {
        const text = this.textInput.value;
        const words = this.countWords(text);
        const stats = `
Word Counter Statistics:
━━━━━━━━━━━━━━━━━━━━━━
Words: ${words}
Characters: ${text.length}
Characters (no spaces): ${text.replace(/\s/g, '').length}
Sentences: ${this.countSentences(text)}
Paragraphs: ${this.countParagraphs(text)}
Reading Time: ${Math.ceil(words / 200)} min
        `.trim();

        try {
            await navigator.clipboard.writeText(stats);
            alert('Statistics copied to clipboard!');
        } catch (err) {
            alert('Failed to copy statistics');
        }
    }

    async pasteText() {
        try {
            const text = await navigator.clipboard.readText();
            this.textInput.value = text;
            this.analyzeText();
        } catch (err) {
            alert('Failed to paste text. Please paste manually.');
        }
    }
}

// Initialize the word counter
const counter = new WordCounter();
```

## Technology Stack
- HTML5
- CSS3 (Grid, Flexbox, Gradients)
- Vanilla JavaScript
- Clipboard API
- Regular Expressions

## Testing Checklist
- [ ] Word count accurate for various texts
- [ ] Character count includes/excludes spaces correctly
- [ ] Sentence counter works with different punctuation
- [ ] Paragraph counter handles line breaks
- [ ] Reading time calculation reasonable
- [ ] Clear button works with confirmation
- [ ] Copy stats to clipboard functional
- [ ] Paste text from clipboard works
- [ ] Top words display correctly
- [ ] Responsive on mobile devices

## Additional Challenges

### Easy
- Add a character limit indicator
- Include a word goal tracker
- Add export to text file functionality

### Medium
- Implement text-to-speech for reading
- Add grammar checker integration
- Create keyword density analysis
- Add text comparison tool

### Hard
- Build sentiment analysis feature
- Implement readability score (Flesch-Kincaid)
- Add plagiarism checker
- Create SEO keyword analysis tool

## Common Pitfalls
- Not handling empty strings correctly
- Incorrect word counting with multiple spaces
- Not trimming text before analysis
- Performance issues with very large texts
- Not considering different types of punctuation

## Resources
- [MDN Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
- [Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API)
- [String Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)

## Portfolio Tips
1. Show live demo with sample text
2. Highlight real-time analysis feature
3. Demonstrate mobile responsiveness
4. Show code organization and clean architecture
5. Include performance metrics for large texts

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
