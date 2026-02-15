# Pomodoro Timer

<p align="center">
  <img src="https://images.unsplash.com/photo-1484480974693-6ca0a78fb36b?w=600&q=80" alt="Pomodoro Timer" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 2-3 hours  
**Prerequisites:** Basic HTML, CSS, JavaScript, setInterval/setTimeout

---

## Description

Build a Pomodoro Timer based on the famous Pomodoro Technique - a time management method that uses 25-minute focused work sessions followed by short breaks. This project teaches you about timers, intervals, state management, audio notifications, and creating productivity tools that help users manage their time effectively.

The Pomodoro Technique improves focus and productivity by breaking work into manageable intervals. Your timer will count down from 25 minutes (work), followed by 5-minute breaks, with longer 15-minute breaks after every 4 sessions. This project introduces concepts used in time-tracking apps, meditation timers, and workout interval trainers.

---

## Core Features

### Must Have
- 25-minute work timer countdown
- 5-minute short break timer
- Start, pause, and reset controls
- Visual countdown display (MM:SS)
- Audio notification when timer ends
- Session counter tracking completed pomodoros

### Nice to Have
- 15-minute long break after 4 pomodoros
- Customizable timer durations
- Progress ring/circle visual
- Browser notifications
- Task list integration
- Statistics dashboard (total time, sessions completed)
- Dark mode
- Sound selection (different alert tones)
- Auto-start next session option
- Background ambient sounds during work

---

## Learning Objectives

By building this project, you will learn:

- **Timer Functions**: Using setInterval() and setTimeout()
- **Time Calculation**: Converting seconds to MM:SS format
- **State Management**: Tracking timer state (running/paused/stopped)
- **Audio API**: Playing sounds for notifications
- **Notifications API**: Browser desktop notifications
- **localStorage**: Saving settings and statistics
- **UI Updates**: Updating display every second
- **Event Handling**: Start/pause/reset button interactions

---

## Implementation Guide

### Step 1: Create the HTML Structure

Build the timer interface:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pomodoro Timer</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>🍅 Pomodoro Timer</h1>
        
        <!-- Mode Selector -->
        <div class="mode-selector">
            <button class="mode-btn active" data-mode="work">Work</button>
            <button class="mode-btn" data-mode="shortBreak">Short Break</button>
            <button class="mode-btn" data-mode="longBreak">Long Break</button>
        </div>
        
        <!-- Timer Display -->
        <div class="timer-display">
            <svg class="progress-ring" width="300" height="300">
                <circle class="progress-ring__circle" 
                        stroke="#e0e0e0" 
                        stroke-width="8" 
                        fill="transparent" 
                        r="140" 
                        cx="150" 
                        cy="150"/>
                <circle class="progress-ring__progress" 
                        stroke="#4caf50" 
                        stroke-width="8" 
                        fill="transparent" 
                        r="140" 
                        cx="150" 
                        cy="150"
                        stroke-dasharray="879.645 879.645"
                        stroke-dashoffset="0"/>
            </svg>
            
            <div class="timer-text">
                <div id="timeDisplay" class="time">25:00</div>
                <div id="modeLabel" class="mode-label">Focus Time</div>
            </div>
        </div>
        
        <!-- Controls -->
        <div class="controls">
            <button id="startBtn" class="control-btn start">
                <span class="btn-icon">▶️</span>
                <span class="btn-text">Start</span>
            </button>
            <button id="pauseBtn" class="control-btn pause hidden">
                <span class="btn-icon">⏸️</span>
                <span class="btn-text">Pause</span>
            </button>
            <button id="resetBtn" class="control-btn reset">
                <span class="btn-icon">↻</span>
                <span class="btn-text">Reset</span>
            </button>
        </div>
        
        <!-- Statistics -->
        <div class="stats">
            <div class="stat-card">
                <div class="stat-value" id="pomodorosCompleted">0</div>
                <div class="stat-label">Pomodoros Today</div>
            </div>
            <div class="stat-card">
                <div class="stat-value" id="totalTime">0h 0m</div>
                <div class="stat-label">Total Focus Time</div>
            </div>
        </div>
        
        <!-- Settings Panel -->
        <div class="settings">
            <h3>⚙️ Settings</h3>
            <div class="setting-item">
                <label for="workDuration">Work Duration (minutes):</label>
                <input type="number" id="workDuration" min="1" max="60" value="25">
            </div>
            <div class="setting-item">
                <label for="shortBreakDuration">Short Break (minutes):</label>
                <input type="number" id="shortBreakDuration" min="1" max="30" value="5">
            </div>
            <div class="setting-item">
                <label for="longBreakDuration">Long Break (minutes):</label>
                <input type="number" id="longBreakDuration" min="1" max="60" value="15">
            </div>
            <div class="setting-item">
                <label for="autoStart">
                    <input type="checkbox" id="autoStart">
                    Auto-start next session
                </label>
            </div>
            <div class="setting-item">
                <label for="notifications">
                    <input type="checkbox" id="notifications" checked>
                    Enable notifications
                </label>
            </div>
        </div>
    </div>
    
    <audio id="timerEndSound" src="https://assets.mixkit.co/active_storage/sfx/2869/2869-preview.mp3"></audio>
    
    <script src="script.js"></script>
</body>
</html>
```

### Step 2: Implement Timer Logic

Create the JavaScript functionality:

```javascript
// script.js
class PomodoroTimer {
    constructor() {
        this.durations = {
            work: 25 * 60, // 25 minutes in seconds
            shortBreak: 5 * 60,
            longBreak: 15 * 60
        };
        
        this.currentMode = 'work';
        this.timeLeft = this.durations.work;
        this.isRunning = false;
        this.intervalId = null;
        this.pomodorosCompleted = 0;
        this.totalFocusTime = 0;
        
        this.elements = {
            timeDisplay: document.getElementById('timeDisplay'),
            modeLabel: document.getElementById('modeLabel'),
            startBtn: document.getElementById('startBtn'),
            pauseBtn: document.getElementById('pauseBtn'),
            resetBtn: document.getElementById('resetBtn'),
            pomodorosCompleted: document.getElementById('pomodorosCompleted'),
            totalTime: document.getElementById('totalTime'),
            progressCircle: document.querySelector('.progress-ring__progress'),
            timerEndSound: document.getElementById('timerEndSound')
        };
        
        this.progressCircleCircumference = 2 * Math.PI * 140; // r=140
        
        this.init();
    }
    
    init() {
        // Mode buttons
        document.querySelectorAll('.mode-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.switchMode(e.target.dataset.mode);
            });
        });
        
        // Control buttons
        this.elements.startBtn.addEventListener('click', () => this.start());
        this.elements.pauseBtn.addEventListener('click', () => this.pause());
        this.elements.resetBtn.addEventListener('click', () => this.reset());
        
        // Settings
        document.getElementById('workDuration').addEventListener('change', (e) => {
            this.durations.work = parseInt(e.target.value) * 60;
            if (this.currentMode === 'work' && !this.isRunning) {
                this.timeLeft = this.durations.work;
                this.updateDisplay();
            }
        });
        
        document.getElementById('shortBreakDuration').addEventListener('change', (e) => {
            this.durations.shortBreak = parseInt(e.target.value) * 60;
            if (this.currentMode === 'shortBreak' && !this.isRunning) {
                this.timeLeft = this.durations.shortBreak;
                this.updateDisplay();
            }
        });
        
        document.getElementById('longBreakDuration').addEventListener('change', (e) => {
            this.durations.longBreak = parseInt(e.target.value) * 60;
            if (this.currentMode === 'longBreak' && !this.isRunning) {
                this.timeLeft = this.durations.longBreak;
                this.updateDisplay();
            }
        });
        
        // Request notification permission
        if ('Notification' in window && Notification.permission === 'default') {
            Notification.requestPermission();
        }
        
        // Load saved data
        this.loadData();
        
        // Initial display
        this.updateDisplay();
        this.updateProgress();
    }
    
    start() {
        if (this.isRunning) return;
        
        this.isRunning = true;
        this.elements.startBtn.classList.add('hidden');
        this.elements.pauseBtn.classList.remove('hidden');
        
        this.intervalId = setInterval(() => {
            this.timeLeft--;
            this.updateDisplay();
            this.updateProgress();
            
            if (this.timeLeft <= 0) {
                this.timerComplete();
            }
        }, 1000);
        
        // Update document title
        this.updateDocumentTitle();
    }
    
    pause() {
        if (!this.isRunning) return;
        
        this.isRunning = false;
        clearInterval(this.intervalId);
        
        this.elements.startBtn.classList.remove('hidden');
        this.elements.pauseBtn.classList.add('hidden');
        
        document.title = 'Pomodoro Timer';
    }
    
    reset() {
        this.pause();
        this.timeLeft = this.durations[this.currentMode];
        this.updateDisplay();
        this.updateProgress();
        document.title = 'Pomodoro Timer';
    }
    
    timerComplete() {
        this.pause();
        
        // Play sound
        this.elements.timerEndSound.play();
        
        // Update statistics
        if (this.currentMode === 'work') {
            this.pomodorosCompleted++;
            this.totalFocusTime += this.durations.work;
            this.updateStats();
            this.saveData();
        }
        
        // Show notification
        this.showNotification();
        
        // Auto-switch to break
        const autoStart = document.getElementById('autoStart').checked;
        
        if (this.currentMode === 'work') {
            // Switch to break
            const nextMode = this.pomodorosCompleted % 4 === 0 ? 'longBreak' : 'shortBreak';
            this.switchMode(nextMode);
            
            if (autoStart) {
                setTimeout(() => this.start(), 2000);
            }
        } else {
            // Switch to work
            this.switchMode('work');
            
            if (autoStart) {
                setTimeout(() => this.start(), 2000);
            }
        }
    }
    
    switchMode(mode) {
        this.pause();
        this.currentMode = mode;
        this.timeLeft = this.durations[mode];
        
        // Update active button
        document.querySelectorAll('.mode-btn').forEach(btn => {
            btn.classList.toggle('active', btn.dataset.mode === mode);
        });
        
        // Update label
        const labels = {
            work: 'Focus Time',
            shortBreak: 'Short Break',
            longBreak: 'Long Break'
        };
        this.elements.modeLabel.textContent = labels[mode];
        
        // Update progress color
        const colors = {
            work: '#4caf50',
            shortBreak: '#2196f3',
            longBreak: '#ff9800'
        };
        this.elements.progressCircle.setAttribute('stroke', colors[mode]);
        
        this.updateDisplay();
        this.updateProgress();
    }
    
    updateDisplay() {
        const minutes = Math.floor(this.timeLeft / 60);
        const seconds = this.timeLeft % 60;
        this.elements.timeDisplay.textContent = 
            `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
        
        if (this.isRunning) {
            this.updateDocumentTitle();
        }
    }
    
    updateDocumentTitle() {
        const minutes = Math.floor(this.timeLeft / 60);
        const seconds = this.timeLeft % 60;
        document.title = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')} - Pomodoro`;
    }
    
    updateProgress() {
        const totalTime = this.durations[this.currentMode];
        const progress = this.timeLeft / totalTime;
        const offset = this.progressCircleCircumference * (1 - progress);
        this.elements.progressCircle.style.strokeDashoffset = offset;
    }
    
    updateStats() {
        this.elements.pomodorosCompleted.textContent = this.pomodorosCompleted;
        
        const hours = Math.floor(this.totalFocusTime / 3600);
        const minutes = Math.floor((this.totalFocusTime % 3600) / 60);
        this.elements.totalTime.textContent = `${hours}h ${minutes}m`;
    }
    
    showNotification() {
        const notificationsEnabled = document.getElementById('notifications').checked;
        if (!notificationsEnabled) return;
        
        if ('Notification' in window && Notification.permission === 'granted') {
            const messages = {
                work: 'Great work! Time for a break.',
                shortBreak: 'Break is over. Ready to focus?',
                longBreak: 'Long break complete. Let\'s get back to work!'
            };
            
            new Notification('Pomodoro Timer', {
                body: messages[this.currentMode],
                icon: '🍅',
                badge: '🍅'
            });
        }
    }
    
    saveData() {
        const data = {
            pomodorosCompleted: this.pomodorosCompleted,
            totalFocusTime: this.totalFocusTime,
            date: new Date().toDateString()
        };
        localStorage.setItem('pomodoroData', JSON.stringify(data));
    }
    
    loadData() {
        try {
            const saved = localStorage.getItem('pomodoroData');
            if (saved) {
                const data = JSON.parse(saved);
                
                // Reset daily stats if it's a new day
                if (data.date !== new Date().toDateString()) {
                    this.pomodorosCompleted = 0;
                    this.totalFocusTime = 0;
                } else {
                    this.pomodorosCompleted = data.pomodorosCompleted || 0;
                    this.totalFocusTime = data.totalFocusTime || 0;
                }
                
                this.updateStats();
            }
        } catch (error) {
            console.error('Error loading data:', error);
        }
    }
}

// Initialize
new PomodoroTimer();
```

### Step 3: Style the Timer

Create an attractive, focused design:

```css
/* styles.css */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

:root {
    --primary-color: #4caf50;
    --secondary-color: #2196f3;
    --accent-color: #ff9800;
    --bg-primary: #1a1a2e;
    --bg-secondary: #16213e;
    --text-primary: #ffffff;
    --text-secondary: #a0a0a0;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, var(--bg-primary) 0%, var(--bg-secondary) 100%);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
    color: var(--text-primary);
}

.container {
    max-width: 600px;
    width: 100%;
    text-align: center;
}

h1 {
    font-size: 48px;
    margin-bottom: 40px;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

/* Mode Selector */
.mode-selector {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-bottom: 40px;
}

.mode-btn {
    padding: 12px 24px;
    border: 2px solid rgba(255,255,255,0.2);
    background: transparent;
    color: var(--text-secondary);
    border-radius: 25px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.mode-btn.active {
    background: var(--primary-color);
    color: white;
    border-color: var(--primary-color);
}

.mode-btn:hover {
    border-color: var(--primary-color);
}

/* Timer Display */
.timer-display {
    position: relative;
    width: 300px;
    height: 300px;
    margin: 0 auto 40px;
}

.progress-ring {
    transform: rotate(-90deg);
}

.progress-ring__progress {
    transition: stroke-dashoffset 1s linear;
}

.timer-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
}

.time {
    font-size: 64px;
    font-weight: bold;
    margin-bottom: 10px;
}

.mode-label {
    font-size: 18px;
    color: var(--text-secondary);
    text-transform: uppercase;
    letter-spacing: 2px;
}

/* Controls */
.controls {
    display: flex;
    gap: 15px;
    justify-content: center;
    margin-bottom: 40px;
}

.control-btn {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 15px 30px;
    border: none;
    border-radius: 50px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.control-btn.start {
    background: var(--primary-color);
    color: white;
}

.control-btn.pause {
    background: var(--accent-color);
    color: white;
}

.control-btn.reset {
    background: rgba(255,255,255,0.1);
    color: white;
}

.control-btn:hover {
    transform: translateY(-3px);
    box-shadow: 0 5px 15px rgba(0,0,0,0.3);
}

.control-btn.hidden {
    display: none;
}

/* Statistics */
.stats {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
    margin-bottom: 40px;
}

.stat-card {
    background: rgba(255,255,255,0.05);
    padding: 25px;
    border-radius: 15px;
    border: 1px solid rgba(255,255,255,0.1);
}

.stat-value {
    font-size: 36px;
    font-weight: bold;
    color: var(--primary-color);
    margin-bottom: 10px;
}

.stat-label {
    font-size: 14px;
    color: var(--text-secondary);
}

/* Settings */
.settings {
    background: rgba(255,255,255,0.05);
    padding: 25px;
    border-radius: 15px;
    border: 1px solid rgba(255,255,255,0.1);
    text-align: left;
}

.settings h3 {
    margin-bottom: 20px;
}

.setting-item {
    margin-bottom: 15px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.setting-item label {
    font-size: 14px;
}

.setting-item input[type="number"] {
    width: 80px;
    padding: 8px;
    border: 1px solid rgba(255,255,255,0.2);
    background: rgba(255,255,255,0.05);
    color: white;
    border-radius: 8px;
}

.setting-item input[type="checkbox"] {
    margin-right: 10px;
}

/* Responsive */
@media (max-width: 768px) {
    h1 {
        font-size: 36px;
    }
    
    .timer-display {
        width: 250px;
        height: 250px;
    }
    
    .time {
        font-size: 48px;
    }
    
    .controls {
        flex-direction: column;
        align-items: stretch;
    }
    
    .stats {
        grid-template-columns: 1fr;
    }
}
```

---

## Technology Stack

### Web Development
- **HTML5**: Structure with semantic elements
- **CSS3**: Animations and gradients
- **Vanilla JavaScript**: Timer logic and state management
- **Web Audio API**: Sound notifications
- **Notifications API**: Browser notifications
- **localStorage**: Statistics persistence

---

## Testing Checklist

- [ ] Timer counts down correctly
- [ ] Start/pause/reset buttons work
- [ ] Mode switching updates timer duration
- [ ] Progress ring animates smoothly
- [ ] Sound plays when timer ends
- [ ] Browser notifications appear (if enabled)
- [ ] Statistics update after each pomodoro
- [ ] Custom durations save and apply
- [ ] Auto-start option works
- [ ] Stats reset daily
- [ ] Responsive on mobile devices

---

## Additional Challenges

### Easy
- Add different notification sounds
- Create fullscreen mode
- Add keyboard shortcuts (Space = start/pause)
- Display motivational quotes

### Medium
- Build task list with pomodoro estimation
- Add daily/weekly statistics charts
- Create ambient background sounds
- Implement themes and color schemes
- Add browser extension version

### Hard
- Build team pomodoro sessions (multiplayer)
- Add Spotify integration for focus playlists
- Create AI-powered productivity insights
- Implement calendar integration
- Build mobile app with background timers

---

## Common Pitfalls

**Timer drift**: Use actual time comparison instead of just decrementing  
**Tab inactive**: Consider using Web Workers for background timing  
**Notification permission**: Always check permission status before showing  
**Memory leaks**: Clear intervals properly on component cleanup  

---

## Resources

### Documentation
- [setInterval - MDN](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)
- [Notifications API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
- [Web Audio API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)

### About Pomodoro Technique
- [Pomodoro Technique Official](https://francescocirillo.com/pages/pomodoro-technique)
- [Productivity Tips](https://todoist.com/productivity-methods/pomodoro-technique)

---

## Submission Checklist

- [ ] Full timer functionality (start/pause/reset)
- [ ] Multiple modes (work/breaks)
- [ ] Visual progress indicator
- [ ] Sound and browser notifications
- [ ] Statistics tracking
- [ ] Customizable settings
- [ ] Responsive design
- [ ] Clean, organized code

---

## Portfolio Tips

1. **Show Usage**: Record yourself using it for actual work
2. **Productivity Stats**: Display how it improved your focus
3. **Comparison**: Mention popular apps like Focus Keeper, Tomato Timer
4. **Personal Touch**: Add custom themes or features you use
5. **Impact**: Explain how Pomodoro Technique helps productivity

---

## License

This project idea is part of the [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
