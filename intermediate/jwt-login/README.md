# JWT Authentication System

<p align="center">
  <img src="https://images.unsplash.com/photo-1555949963-aa79dcee981c?w=600&q=80" alt="JWT Login" width="600">
</p>

**Level:** Intermediate | **Time:** 1-2 weeks | **Prerequisites:** Node.js, Express, MongoDB, JWT basics

## Description

Build a complete authentication system using JSON Web Tokens (JWT) including user registration, login, password hashing, token management, protected routes, and refresh token implementation. Essential for any full-stack application requiring secure user authentication.

## Core Features

### Must Have
- User registration with validation
- Secure password hashing (bcrypt)
- Login with JWT generation
- Protected routes/middleware
- Token verification
- Logout functionality
- Input validation and sanitization

### Nice to Have
- Refresh token mechanism
- Email verification
- Password reset via email
- OAuth integration (Google, GitHub)
- Role-based access control (RBAC)
- Account lockout after failed attempts
- Two-factor authentication (2FA)
- Session management
- Activity logging

## Implementation Guide

```javascript
// server.js
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(express.json());
app.use(cors());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI);

// User Model
const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    role: { type: String, default: 'user' },
    createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);

// Middleware to verify JWT
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) return res.status(401).json({ error: 'Access denied' });
    
    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        if (err) return res.status(403).json({ error: 'Invalid token' });
        req.user = user;
        next();
    });
};

// Register Route
app.post('/api/auth/register', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        // Validation
        if (!username || !email || !password) {
            return res.status(400).json({ error: 'All fields required' });
        }
        
        // Check existing user
        const existingUser = await User.findOne({ $or: [{ email }, { username }] });
        if (existingUser) {
            return res.status(400).json({ error: 'User already exists' });
        }
        
        // Hash password
        const salt = await bcrypt.genSalt(10);
        const hashedPassword = await bcrypt.hash(password, salt);
        
        // Create user
        const user = new User({
            username,
            email,
            password: hashedPassword
        });
        
        await user.save();
        
        // Generate token
        const token = jwt.sign(
            { id: user._id, username: user.username, role: user.role },
            process.env.JWT_SECRET,
            { expiresIn: '24h' }
        );
        
        res.status(201).json({
            message: 'User registered successfully',
            token,
            user: { id: user._id, username: user.username, email: user.email }
        });
    } catch (error) {
        res.status(500).json({ error: 'Server error' });
    }
});

// Login Route
app.post('/api/auth/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Find user
        const user = await User.findOne({ email });
        if (!user) {
            return res.status(400).json({ error: 'Invalid credentials' });
        }
        
        // Verify password
        const validPassword = await bcrypt.compare(password, user.password);
        if (!validPassword) {
            return res.status(400).json({ error: 'Invalid credentials' });
        }
        
        // Generate token
        const token = jwt.sign(
            { id: user._id, username: user.username, role: user.role },
            process.env.JWT_SECRET,
            { expiresIn: '24h' }
        );
        
        res.json({
            message: 'Login successful',
            token,
            user: { id: user._id, username: user.username, email: user.email }
        });
    } catch (error) {
        res.status(500).json({ error: 'Server error' });
    }
});

// Protected Route Example
app.get('/api/profile', authenticateToken, async (req, res) => {
    try {
        const user = await User.findById(req.user.id).select('-password');
        res.json(user);
    } catch (error) {
        res.status(500).json({ error: 'Server error' });
    }
});

// Verify Token Route
app.get('/api/auth/verify', authenticateToken, (req, res) => {
    res.json({ valid: true, user: req.user });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

```html
<!-- Frontend Login Form -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JWT Login</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .container {
            background: white;
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            width: 400px;
        }
        h2 { margin-bottom: 30px; text-align: center; }
        .form-group { margin-bottom: 20px; }
        label { display: block; margin-bottom: 5px; font-weight: 600; }
        input {
            width: 100%;
            padding: 12px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 14px;
        }
        button {
            width: 100%;
            padding: 14px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
        }
        .error { color: red; font-size: 14px; margin-top: 10px; }
        .success { color: green; font-size: 14px; margin-top: 10px; }
    </style>
</head>
<body>
    <div class="container">
        <h2>Login</h2>
        <form id="loginForm">
            <div class="form-group">
                <label>Email</label>
                <input type="email" id="email" required>
            </div>
            <div class="form-group">
                <label>Password</label>
                <input type="password" id="password" required>
            </div>
            <button type="submit">Login</button>
            <div id="message"></div>
        </form>
    </div>
    
    <script>
        document.getElementById('loginForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            
            try {
                const response = await fetch('http://localhost:5000/api/auth/login', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ email, password })
                });
                
                const data = await response.json();
                
                if (response.ok) {
                    localStorage.setItem('token', data.token);
                    document.getElementById('message').innerHTML = '<p class="success">Login successful!</p>';
                    setTimeout(() => window.location.href = '/dashboard.html', 1000);
                } else {
                    document.getElementById('message').innerHTML = `<p class="error">${data.error}</p>`;
                }
            } catch (error) {
                document.getElementById('message').innerHTML = '<p class="error">Connection error</p>';
            }
        });
    </script>
</body>
</html>
```

## Technology Stack
- **Backend:** Node.js, Express.js
- **Database:** MongoDB, Mongoose
- **Authentication:** JWT, bcrypt
- **Validation:** express-validator
- **Frontend:** Vanilla JS or React

## Testing Checklist
- [ ] Registration creates user
- [ ] Passwords hashed correctly
- [ ] Login generates valid JWT
- [ ] Protected routes require token
- [ ] Invalid tokens rejected
- [ ] Token expiration works
- [ ] Logout clears token
- [ ] Input validation working

## Resources
- [JWT.io](https://jwt.io/)
- [bcrypt Documentation](https://github.com/kelektiv/node.bcrypt.js)
- [Express.js Guide](https://expressjs.com/)

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
