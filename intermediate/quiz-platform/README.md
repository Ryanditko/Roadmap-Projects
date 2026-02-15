# Quiz Platform

<p align="center">
  <img src="https://images.unsplash.com/photo-1606326608606-aa0b62935f2b?w=600&q=80" alt="Quiz Platform" width="600">
</p>

**Level:** Intermediate | **Time:** 1 week

## Description
Interactive quiz application with multiple question types, scoring, timer, progress tracking, and results analysis. Learn state management, timer logic, and data visualization.

## Core Features
- Multiple choice questions
- True/False questions
- Timer per quiz/question
- Score calculation
- Progress bar
- Results summary with charts
- Question bank management
- Categories and difficulty levels

## Implementation Guide

```javascript
class QuizPlatform {
    constructor() {
        this.questions = [];
        this.currentQuestionIndex = 0;
        this.score = 0;
        this.answers = [];
        this.timer = null;
        this.timeLeft = 0;
    }
    
    loadQuiz(questions, timeLimit = 60) {
        this.questions = questions;
        this.timeLimit = timeLimit;
        this.timeLeft = timeLimit;
        this.currentQuestionIndex = 0;
        this.score = 0;
        this.answers = [];
        
        this.showQuestion();
        this.startTimer();
    }
    
    showQuestion() {
        const question = this.questions[this.currentQuestionIndex];
        
        document.getElementById('question-number').textContent = 
            `Question ${this.currentQuestionIndex + 1} of ${this.questions.length}`;
        
        document.getElementById('question-text').textContent = question.text;
        
        const optionsContainer = document.getElementById('options');
        optionsContainer.innerHTML = '';
        
        question.options.forEach((option, index) => {
            const button = document.createElement('button');
            button.className = 'option';
            button.textContent = option;
            button.onclick = () => this.selectAnswer(index);
            optionsContainer.appendChild(button);
        });
        
        this.updateProgress();
    }
    
    selectAnswer(answerIndex) {
        const question = this.questions[this.currentQuestionIndex];
        const isCorrect = answerIndex === question.correctAnswer;
        
        this.answers.push({
            questionIndex: this.currentQuestionIndex,
            selectedAnswer: answerIndex,
            correctAnswer: question.correctAnswer,
            isCorrect: isCorrect
        });
        
        if (isCorrect) {
            this.score++;
        }
        
        this.nextQuestion();
    }
    
    nextQuestion() {
        this.currentQuestionIndex++;
        
        if (this.currentQuestionIndex < this.questions.length) {
            this.showQuestion();
        } else {
            this.endQuiz();
        }
    }
    
    startTimer() {
        this.timer = setInterval(() => {
            this.timeLeft--;
            
            const minutes = Math.floor(this.timeLeft / 60);
            const seconds = this.timeLeft % 60;
            document.getElementById('timer').textContent = 
                `${minutes}:${seconds.toString().padStart(2, '0')}`;
            
            if (this.timeLeft <= 0) {
                this.endQuiz();
            }
        }, 1000);
    }
    
    stopTimer() {
        if (this.timer) {
            clearInterval(this.timer);
            this.timer = null;
        }
    }
    
    updateProgress() {
        const progress = ((this.currentQuestionIndex + 1) / this.questions.length) * 100;
        document.getElementById('progress-bar').style.width = progress + '%';
    }
    
    endQuiz() {
        this.stopTimer();
        this.showResults();
    }
    
    showResults() {
        const percentage = (this.score / this.questions.length) * 100;
        const timeTaken = this.timeLimit - this.timeLeft;
        
        const resultsHTML = `
            <div class="results">
                <h2>Quiz Complete!</h2>
                <div class="score">
                    <p>Your Score: ${this.score}/${this.questions.length}</p>
                    <p>Percentage: ${percentage.toFixed(1)}%</p>
                    <p>Time Taken: ${Math.floor(timeTaken / 60)}:${(timeTaken % 60).toString().padStart(2, '0')}</p>
                </div>
                <div class="grade">
                    ${this.getGrade(percentage)}
                </div>
                <button onclick="quiz.restart()">Retake Quiz</button>
                <button onclick="quiz.reviewAnswers()">Review Answers</button>
            </div>
        `;
        
        document.getElementById('quiz-container').innerHTML = resultsHTML;
    }
    
    getGrade(percentage) {
        if (percentage >= 90) return '🏆 Excellent!';
        if (percentage >= 70) return '👍 Good Job!';
        if (percentage >= 50) return '👌 Pass';
        return '📚 Keep Studying';
    }
    
    reviewAnswers() {
        let reviewHTML = '<h2>Answer Review</h2>';
        
        this.answers.forEach((answer, index) => {
            const question = this.questions[answer.questionIndex];
            reviewHTML += `
                <div class="review-item ${answer.isCorrect ? 'correct' : 'incorrect'}">
                    <h4>Question ${index + 1}</h4>
                    <p>${question.text}</p>
                    <p>Your answer: ${question.options[answer.selectedAnswer]}</p>
                    <p>Correct answer: ${question.options[answer.correctAnswer]}</p>
                </div>
            `;
        });
        
        document.getElementById('quiz-container').innerHTML = reviewHTML;
    }
    
    restart() {
        this.loadQuiz(this.questions, this.timeLimit);
    }
}

// Example questions
const sampleQuiz = [
    {
        text: "What is the capital of France?",
        options: ["London", "Berlin", "Paris", "Madrid"],
        correctAnswer: 2
    },
    {
        text: "What is 2 + 2?",
        options: ["3", "4", "5", "6"],
        correctAnswer: 1
    },
    {
        text: "Which planet is known as the Red Planet?",
        options: ["Venus", "Mars", "Jupiter", "Saturn"],
        correctAnswer: 1
    }
];

const quiz = new QuizPlatform();
quiz.loadQuiz(sampleQuiz, 120);
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quiz Platform</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        #quiz-container {
            background: white;
            border-radius: 20px;
            padding: 40px;
            width: 600px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }
        .header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 30px;
        }
        #progress-container {
            background: #e0e0e0;
            height: 10px;
            border-radius: 5px;
            overflow: hidden;
            margin-bottom: 20px;
        }
        #progress-bar {
            background: linear-gradient(90deg, #667eea, #764ba2);
            height: 100%;
            width: 0%;
            transition: width 0.3s;
        }
        .option {
            display: block;
            width: 100%;
            padding: 15px;
            margin-bottom: 10px;
            background: #f5f5f5;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s;
        }
        .option:hover {
            background: #667eea;
            color: white;
            transform: translateX(10px);
        }
        .results {
            text-align: center;
        }
        .score {
            background: #f5f5f5;
            padding: 30px;
            border-radius: 15px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div id="quiz-container">
        <div class="header">
            <span id="question-number"></span>
            <span id="timer">0:00</span>
        </div>
        <div id="progress-container">
            <div id="progress-bar"></div>
        </div>
        <h3 id="question-text"></h3>
        <div id="options"></div>
    </div>
</body>
</html>
```

## Technology Stack
- Vanilla JavaScript
- CSS animations
- localStorage for quiz history
- Chart.js for results visualization

## Testing Checklist
- [ ] Questions display correctly
- [ ] Timer counts down
- [ ] Score calculates accurately
- [ ] Progress bar updates
- [ ] Results page shows all data
- [ ] Can restart quiz
- [ ] Review answers works

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
