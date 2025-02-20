# Ludo1
// Basic structure for a multiplayer Ludo game
// This will include player management, game logic, and a simple UI with admin panel

const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const cors = require('cors');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

app.use(cors());
app.use(express.json());

let players = {};
let adminPassword = "admin123"; // Simple admin authentication

io.on('connection', (socket) => {
    console.log('A user connected:', socket.id);
    
    // Player joins the game
    socket.on('joinGame', (playerData) => {
        players[socket.id] = playerData;
        io.emit('updatePlayers', players);
    });

    // Handle dice roll
    socket.on('rollDice', () => {
        const diceValue = Math.floor(Math.random() * 6) + 1;
        io.emit('diceRolled', { playerId: socket.id, diceValue });
    });

    // Handle player movement
    socket.on('movePiece', (data) => {
        io.emit('pieceMoved', data);
    });

    // Handle chat messages
    socket.on('sendMessage', (message) => {
        io.emit('receiveMessage', { playerId: socket.id, message });
    });

    // Player disconnects
    socket.on('disconnect', () => {
        delete players[socket.id];
        io.emit('updatePlayers', players);
    });
});

// Admin Panel Routes
app.post('/admin/login', (req, res) => {
    const { password } = req.body;
    if (password === adminPassword) {
        res.json({ success: true, message: "Admin authenticated" });
    } else {
        res.status(401).json({ success: false, message: "Invalid password" });
    }
});

app.get('/admin/players', (req, res) => {
    res.json(players);
});

app.post('/admin/kick', (req, res) => {
    const { playerId } = req.body;
    if (players[playerId]) {
        delete players[playerId];
        io.emit('updatePlayers', players);
        res.json({ success: true, message: "Player kicked" });
    } else {
        res.status(404).json({ success: false, message: "Player not found" });
    }
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
