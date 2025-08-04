# Prodigy_FSD_01
# Task-01: Secure User Authentication Website

## Description

Implement a user authentication system with secure login and registration functionality. Users can sign up, log in securely, and access protected routes only after successful authentication.

Optional: Use standard mechanisms to handle password hashing, session management, and user role-based access control.

---

## Tech Stack

- **Backend:** Node.js, Express, MongoDB, bcrypt, JWT
- **Frontend:** React.js

---

## Backend Code (`backend/app.js`)

```javascript
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors());

mongoose.connect('mongodb://localhost:27017/auth-demo', { useNewUrlParser: true, useUnifiedTopology: true });

const userSchema = new mongoose.Schema({
  username: { type: String, unique: true },
  password: String,
  role: { type: String, default: 'user' }
});

const User = mongoose.model('User', userSchema);

const SECRET = 'your_jwt_secret_key';

// Register
app.post('/register', async (req, res) => {
  const { username, password } = req.body;
  const hash = await bcrypt.hash(password, 10);
  try {
    const user = await User.create({ username, password: hash });
    res.status(201).json({ message: 'User registered' });
  } catch (err) {
    res.status(400).json({ error: 'Username already exists' });
  }
});

// Login
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  if (!user) return res.status(400).json({ error: 'Invalid credentials' });
  const valid = await bcrypt.compare(password, user.password);
  if (!valid) return res.status(400).json({ error: 'Invalid credentials' });
  const token = jwt.sign({ id: user._id, username: user.username, role: user.role }, SECRET, { expiresIn: '1h' });
  res.json({ token });
});

// Middleware to protect routes
function authMiddleware(req, res, next) {
  const token = req.headers['authorization'];
  if (!token) return res.status(401).json({ error: 'No token provided' });
  try {
    const decoded = jwt.verify(token, SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Protected route
app.get('/protected', authMiddleware, (req, res) => {
  res.json({ message: `Hello ${req.user.username}, this is protected!` });
});

// Role-based route (admin only example)
app.get('/admin', authMiddleware, (req, res) => {
  if (req.user.role !== 'admin') return res.status(403).json({ error: 'Access denied' });
  res.json({ message: 'Welcome, admin!' });
});

app.listen(5000, () => console.log('Server running on port 5000'));
```

---

## Frontend Code (`frontend/src/App.js`)

```javascript
import React, { useState } from 'react';

const API = 'http://localhost:5000';

function App() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [token, setToken] = useState('');
  const [message, setMessage] = useState('');

  const register = async () => {
    const res = await fetch(`${API}/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    });
    const data = await res.json();
    setMessage(data.message || data.error);
  };

  const login = async () => {
    const res = await fetch(`${API}/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    });
    const data = await res.json();
    if (data.token) {
      setToken(data.token);
      setMessage('Login successful!');
    } else {
      setMessage(data.error);
    }
  };

  const getProtected = async () => {
    const res = await fetch(`${API}/protected`, {
      headers: { 'Authorization': token }
    });
    const data = await res.json();
    setMessage(data.message || data.error);
  };

  return (
    <div style={{ margin: 'auto', width: '300px', padding: '50px' }}>
      <h2>Secure User Authentication</h2>
      <input placeholder="Username" value={username} onChange={e => setUsername(e.target.value)} /><br /><br />
      <input type="password" placeholder="Password" value={password} onChange={e => setPassword(e.target.value)} /><br /><br />
      <button onClick={register}>Register</button>
      <button onClick={login}>Login</button>
      <button onClick={getProtected} disabled={!token}>Access Protected Route</button>
      <p>{message}</p>
    </div>
  );
}

export default App;
```

---

## Frontend Dependencies (`frontend/package.json`)

```json
{
  "name": "frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-scripts": "5.0.0"
  },
  "scripts": {
    "start": "react-scripts start"
  }
}
```

---

## Instructions

### 1. Start MongoDB

Make sure MongoDB is running on your machine (`mongod`).

### 2. Backend Setup

```bash
cd backend
npm install express mongoose bcrypt jsonwebtoken cors
node app.js
```

### 3. Frontend Setup

```bash
cd frontend
npm install
npm start
```

### 4. Access the App

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Features

- User Registration
- Secure Login (Password Hashing)
- JWT-based Session Management
- Protected Routes
- Role-Based Access Example (Admin)
