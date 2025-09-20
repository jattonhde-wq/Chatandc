<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Real-Time Chat Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
<style>
  body {
    margin:0; padding:0;
    font-family: 'Roboto', sans-serif;
    background:#f0f4ff;
    display:flex; height:100vh;
  }

  /* Sidebar */
  .sidebar {
    width: 220px;
    background:#4f46e5;
    color:white;
    display:flex; flex-direction:column;
    padding:20px;
  }
  .sidebar h2 { margin-bottom:20px; font-size:20px; }
  .user-list { flex:1; overflow-y:auto; }
  .user { padding:10px; border-radius:8px; margin-bottom:5px; background:#6366f1; transition:0.2s; }
  .user:hover { background:#818cf8; }

  /* Main chat */
  .chat-container {
    flex:1;
    display:flex;
    flex-direction:column;
    padding:20px;
  }
  .chat-box {
    flex:1;
    background:white;
    border-radius:15px;
    padding:15px;
    overflow-y:auto;
    box-shadow:0 2px 10px rgba(0,0,0,0.05);
    display:flex;
    flex-direction:column;
    gap:10px;
  }
  .message {
    max-width:70%;
    padding:10px 14px;
    border-radius:15px;
    animation: fadeIn 0.3s ease;
    position:relative;
  }
  .message.user { background:#dbeafe; align-self:flex-end; }
  .message.other { background:#eef2ff; align-self:flex-start; }
  .timestamp {
    font-size:10px;
    color:#555;
    margin-top:4px;
  }

  /* Input box */
  .input-area { display:flex; gap:10px; margin-top:15px; flex-wrap:wrap; }
  .input-area input { flex:1; padding:12px; border-radius:10px; border:1px solid #ccc; }
  .input-area button { padding:12px 20px; border:none; border-radius:10px; background:#4f46e5; color:white; cursor:pointer; transition:0.2s; }
  .input-area button:hover { background:#3730a3; }

  @keyframes fadeIn { from {opacity:0;} to {opacity:1;} }

  @media(max-width:768px){
    body{ flex-direction:column; }
    .sidebar{ width:100%; flex-direction:row; overflow-x:auto; padding:10px; }
    .chat-container{ flex:1; }
  }
</style>
</head>
<body>

<div class="sidebar">
  <h2>Active Users</h2>
  <div class="user-list" id="user-list"></div>
</div>

<div class="chat-container">
  <div class="chat-box" id="chat-box"></div>
  <div class="input-area">
    <input type="text" id="username" placeholder="Your name">
    <input type="text" id="message" placeholder="Type a message...">
    <button onclick="sendMessage()">Send</button>
  </div>
</div>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database.js"></script>
<script>
  // 🔹 Replace with your Firebase config
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT.firebaseio.com",
    projectId: "YOUR_PROJECT",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };

  const app = firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  const chatBox = document.getElementById("chat-box");
  const userListDiv = document.getElementById("user-list");

  // Send message
  function sendMessage(){
    const user = document.getElementById("username").value || "Anonymous";
    const msg = document.getElementById("message").value;
    if(msg.trim()==="") return;

    const timestamp = new Date().toLocaleTimeString();
    db.ref("messages").push({user,msg,timestamp});
    document.getElementById("message").value="";
  }

  // Listen for new messages
  db.ref("messages").on("child_added",(snapshot)=>{
    const data = snapshot.val();
    const div = document.createElement("div");
    div.className = "message " + ((data.user === (document.getElementById("username").value || "Anonymous")) ? "user" : "other");
    div.innerHTML = `<strong>${data.user}</strong>: ${data.msg} <div class="timestamp">${data.timestamp}</div>`;
    chatBox.appendChild(div);
    chatBox.scrollTop = chatBox.scrollHeight;
    updateUserList(data.user);
  });

  // Keep track of active users
  let users = new Set();
  function updateUserList(user){
    users.add(user);
    userListDiv.innerHTML = "";
    users.forEach(u=>{
      const div = document.createElement("div");
      div.className="user";
      div.textContent=u;
      userListDiv.appendChild(div);
    });
  }
</script>

</body>
</html>


