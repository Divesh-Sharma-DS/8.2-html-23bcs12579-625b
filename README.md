# 8.2-html-23bcs12579-625b
Implement protected routes with JWT verification

// server.js
const express = require('express');
const jwt = require('jsonwebtoken');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors());

const SECRET_KEY = "mysecretkey123"; // for demo only; keep secure in real apps

// Login Route (issues token)
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  
  // Demo check — real app should verify with DB
  if (username === 'admin' && password === '1234') {
    const token = jwt.sign({ user: username }, SECRET_KEY, { expiresIn: '1h' });
    res.json({ token });
  } else {
    res.status(401).json({ message: 'Invalid credentials' });
  }
});

// Protected Route
app.get('/protected', verifyToken, (req, res) => {
  res.json({ message: `Welcome ${req.user.user}, you accessed a protected route!` });
});

// JWT Verification Middleware
function verifyToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  if (!token) return res.status(403).json({ message: 'Token required' });

  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) return res.status(403).json({ message: 'Invalid or expired token' });
    req.user = user;
    next();
  });
}

app.listen(4000, () => console.log('Server running on http://localhost:4000'));


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>JWT Protected Route Demo</title>
  <style>
    body {
      background: #0d1117;
      color: #e6edf3;
      font-family: "Poppins", sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
    }
    input, button {
      margin: 8px;
      padding: 10px;
      border-radius: 8px;
      border: none;
      font-size: 16px;
    }
    button {
      background-color: #238636;
      color: white;
      cursor: pointer;
    }
    #response {
      margin-top: 20px;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <h2>🔐 JWT Protected Route Demo</h2>

  <div id="login-section">
    <input type="text" id="username" placeholder="Username" />
    <input type="password" id="password" placeholder="Password" />
    <button onclick="login()">Login</button>
  </div>

  <div id="protected-section" style="display:none;">
    <button onclick="accessProtected()">Access Protected Route</button>
  </div>

  <div id="response"></div>

  <script>
    let token = null;

    async function login() {
      const username = document.getElementById('username').value;
      const password = document.getElementById('password').value;

      const res = await fetch('http://localhost:4000/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });

      const data = await res.json();
      if (res.ok) {
        token = data.token;
        document.getElementById('response').textContent = "✅ Logged in successfully!";
        document.getElementById('login-section').style.display = 'none';
        document.getElementById('protected-section').style.display = 'block';
      } else {
        document.getElementById('response').textContent = "❌ " + data.message;
      }
    }

    async function accessProtected() {
      const res = await fetch('http://localhost:4000/protected', {
        method: 'GET',
        headers: {
          'Authorization': 'Bearer ' + token
        }
      });

      const data = await res.json();
      if (res.ok) {
        document.getElementById('response').textContent = data.message;
      } else {
        document.getElementById('response').textContent = "⚠️ " + data.message;
      }
    }
  </script>
</body>
</html>
