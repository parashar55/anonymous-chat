<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Anonymous Chat App</title>
<style>
  body { font-family: Arial, sans-serif; max-width: 600px; margin: auto; padding: 10px; }
  #chat, #messageForm { display: none; margin-top: 20px; }
  #messages { border: 1px solid #ccc; height: 300px; overflow-y: scroll; padding: 10px; }
  .message { margin-bottom: 10px; }
  .username { font-weight: bold; margin-right: 5px; }
  #roomInput, #usernameDisplay { margin-top: 10px; }
</style>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
</head>
<body>

<h2>Join a Chat Room</h2>
<label for="roomId">Enter Room ID:</label>
<input type="text" id="roomId" placeholder="e.g. room123" />
<button id="joinBtn">Join Room</button>

<div id="usernameDisplay"></div>

<div id="chat">
  <div id="messages"></div>
  <form id="messageForm">
    <input type="text" id="messageInput" placeholder="Type your message..." autocomplete="off" required />
    <button type="submit">Send</button>
  </form>
</div>

<script>
// === Firebase config ===
// Replace with your own Firebase project config from Firebase Console
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// Generate random username from arrays
const adjectives = ["Blue", "Red", "Green", "Fast", "Clever", "Quiet", "Brave"];
const animals = ["Tiger", "Bear", "Eagle", "Fox", "Wolf", "Lion", "Hawk"];
function randomUsername() {
  return adjectives[Math.floor(Math.random() * adjectives.length)] +
    animals[Math.floor(Math.random() * animals.length)] +
    Math.floor(Math.random() * 1000);
}

let username = "";
let roomId = "";
const joinBtn = document.getElementById("joinBtn");
const roomInput = document.getElementById("roomId");
const chatDiv = document.getElementById("chat");
const messagesDiv = document.getElementById("messages");
const messageForm = document.getElementById("messageForm");
const messageInput = document.getElementById("messageInput");
const usernameDisplay = document.getElementById("usernameDisplay");

joinBtn.onclick = () => {
  roomId = roomInput.value.trim();
  if (!roomId) {
    alert("Please enter a room ID.");
    return;
  }
  username = randomUsername();
  usernameDisplay.textContent = `Your username: ${username}`;
  roomInput.disabled = true;
  joinBtn.disabled = true;
  chatDiv.style.display = "block";

  loadMessages();
};

function loadMessages() {
  const messagesRef = db.ref("rooms/" + roomId + "/messages");
  messagesRef.off(); // Remove previous listeners

  messagesRef.on("child_added", (snapshot) => {
    const msg = snapshot.val();
    displayMessage(msg.username, msg.text);
  });
}

function displayMessage(user, text) {
  const msgDiv = document.createElement("div");
  msgDiv.className = "message";
  const userSpan = document.createElement("span");
  userSpan.className = "username";
  userSpan.textContent = user + ":";
  const textSpan = document.createElement("span");
  textSpan.textContent = " " + text;
  msgDiv.appendChild(userSpan);
  msgDiv.appendChild(textSpan);
  messagesDiv.appendChild(msgDiv);
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}

messageForm.addEventListener("submit", (e) => {
  e.preventDefault();
  const text = messageInput.value.trim();
  if (!text) return;
  const messagesRef = db.ref("rooms/" + roomId + "/messages");
  messagesRef.push({
    username: username,
    text: text,
    timestamp: Date.now()
  });
  messageInput.value = "";
});
</script>

</body>
</html>
