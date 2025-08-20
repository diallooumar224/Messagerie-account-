# Guinea Messager 
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

const server = http.createServer(app);
const io = socketIo(server, {
  cors: {
    origin: "*"
  }
});

let users = {};

io.on("connection", (socket) => {
  socket.on("login", (username) => {
    users[socket.id] = username;
    io.emit("users", Object.values(users));
  });

  socket.on("send_message", (data) => {
    io.emit("receive_message", data);
  });

  socket.on("disconnect", () => {
    delete users[socket.id];
    io.emit("users", Object.values(users));
  });
});

app.get("/", (req, res) => {
  res.send("API is running");
});

server.listen(5000, () => {
  console.log("Server started on http://localhost:5000");
});
npx create-react-app frontend
cd frontend
npm install socket.io-client
import React, { useEffect, useState } from "react";
import io from "socket.io-client";

const socket = io("http://localhost:5000");

function App() {
  const [username, setUsername] = useState("");
  const [message, setMessage] = useState("");
  const [messages, setMessages] = useState([]);
  const [users, setUsers] = useState([]);

  useEffect(() => {
    socket.on("receive_message", (data) => {
      setMessages((msgs) => [...msgs, data]);
    });
    socket.on("users", (users) => {
      setUsers(users);
    });
    return () => {
      socket.off("receive_message");
      socket.off("users");
    };
  }, []);

  const handleLogin = () => {
    socket.emit("login", username);
  };

  const sendMessage = () => {
    socket.emit("send_message", { username, message, time: new Date().toLocaleTimeString() });
    setMessage("");
  };

  return (
    <div>
      {!users.includes(username) ? (
        <div>
          <input placeholder="Nom d'utilisateur" value={username} onChange={e => setUsername(e.target.value)} />
          <button onClick={handleLogin}>Se connecter</button>
        </div>
      ) : (
        <div>
          <h3>Utilisateurs en ligne :</h3>
          <ul>
            {users.map(u => <li key={u}>{u}</li>)}
          </ul>
          <div style={{ border: "1px solid #ddd", height: 300, overflowY: "scroll" }}>
            {messages.map((msg, idx) => (
              <div key={idx}><b>{msg.username}</b> [{msg.time}]: {msg.message}</div>
            ))}
          </div>
          <input value={message} onChange={e => setMessage(e.target.value)} onKeyDown={e => e.key === "Enter" && sendMessage()} />
          <button onClick={sendMessage}>Envoyer</button>
        </div>
      )}
    </div>
  );
}

export default App;
