# Sriracha-Food-Hunt
/server.js
/public
   index.html
   client.js
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static("public"));

// ===== GAME STATE =====
let players = {};
let checkpoints = [
  { id: 1, name: "ร้านข้าวมันไก่", clue: "ร้านไก่ดังในเมือง", lat: 13.75, lng: 100.50 },
  { id: 2, name: "ร้านก๋วยเตี๋ยว", clue: "เส้นเหนียวน้ำซุปเข้ม", lat: 13.751, lng: 100.501 },
  { id: 3, name: "คาเฟ่น่ารัก", clue: "ร้านกาแฟแมวน่ารัก", lat: 13.752, lng: 100.502 },
  { id: 4, name: "ตลาดของกิน", clue: "รวมสตรีทฟู้ด", lat: 13.753, lng: 100.503 },
  { id: 5, name: "ร้านชานม", clue: "หวานๆเย็นๆ", lat: 13.754, lng: 100.504 },
  { id: 6, name: "ร้านพิซซ่า", clue: "ชีสเยิ้ม", lat: 13.755, lng: 100.505 },
  { id: 7, name: "ร้านราเมง", clue: "เส้นญี่ปุ่น", lat: 13.756, lng: 100.506 },
  { id: 8, name: "ร้านไอศกรีม", clue: "ของหวานเย็น", lat: 13.757, lng: 100.507 },
  { id: 9, name: "ร้านส้มตำ", clue: "เผ็ดแซ่บ", lat: 13.758, lng: 100.508 },
  { id: 10, name: "ร้านสุดท้าย", clue: "จุดชนะเกม!", lat: 13.759, lng: 100.509 },
];

let winner = null;

io.on("connection", (socket) => {
  console.log("User connected:", socket.id);

  players[socket.id] = {
    id: socket.id,
    x: 0,
    y: 0,
    score: 0,
    finished: false,
  };

  socket.emit("init", { players, checkpoints, winner });

  socket.broadcast.emit("player-joined", players[socket.id]);

  socket.on("move", (data) => {
    if (!players[socket.id]) return;

    players[socket.id].x = data.x;
    players[socket.id].y = data.y;

    io.emit("players-update", players);
  });

  socket.on("checkin", (checkpointId) => {
    let p = players[socket.id];
    if (!p || p.finished) return;

    if (checkpointId === p.score + 1) {
      p.score += 1;

      if (p.score >= 10 && !winner) {
        winner = socket.id;
        io.emit("game-winner", winner);
      }
    }

    io.emit("players-update", players);
  });

  socket.on("disconnect", () => {
    delete players[socket.id];
    io.emit("players-update", players);
  });
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
