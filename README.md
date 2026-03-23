<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ink & Paper</title>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js"></script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Georgia, serif;
  background: #fdf6e3;
  min-height: 100vh;
}

header {
  padding: 20px;
  background: white;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

header h1 {
  font-size: 2em;
  color: #2c3e50;
}

#feed {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  padding: 20px;
}

.poem-card {
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.poem-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 12px rgba(0,0,0,0.15);
}

.poem-img {
  width: 100%;
  height: 250px;
  object-fit: cover;
}

.poem-content {
  padding: 20px;
}

.poem-content h2 {
  color: #2c3e50;
  margin-bottom: 10px;
}

.poem-content p {
  color: #555;
  line-height: 1.6;
  margin-bottom: 15px;
  white-space: pre-wrap;
}

.poem-actions {
  display: flex;
  gap: 10px;
}

.poem-actions button {
  flex: 1;
  padding: 8px;
  font-size: 0.9em;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.poem-actions button:hover {
  background: #2980b9;
}

.error {
  color: #e74c3c;
  padding: 10px;
  background: #fadbd8;
  border-radius: 5px;
  margin: 10px 20px;
}

.loading {
  text-align: center;
  padding: 40px;
  color: #7f8c8d;
  grid-column: 1 / -1;
}

@media (max-width: 768px) {
  #feed {
    grid-template-columns: 1fr;
  }

  header {
    flex-direction: column;
    gap: 15px;
  }
}
</style>

</head>
<body>

<header>
  <h1>✨ Ink & Paper ✨</h1>
</header>

<!-- FEED -->
<div id="feed"><div class="loading">Loading poems...</div></div>

<script>

// 🔥 FIREBASE CONFIG - REPLACE WITH YOUR CONFIG
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_MESSAGING_ID",
  appId: "YOUR_APP_ID"
};

const app = firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// 📖 LOAD POEMS
db.collection("poems").orderBy("date", "desc")
  .onSnapshot(snapshot => {
    let feed = document.getElementById("feed");
    feed.innerHTML = "";

    if (snapshot.empty) {
      feed.innerHTML = '<div class="loading">No poems yet. Be the first to share!</div>';
      return;
    }

    snapshot.forEach(doc => {
      let p = doc.data();

      let div = document.createElement("div");
      div.className = "poem-card";

      let imgHtml = p.image ? `<img src="${p.image}" class="poem-img" alt="${p.title}">` : "";

      div.innerHTML = `
        ${imgHtml}
        <div class="poem-content">
          <h2>${escapeHtml(p.title)}</h2>
          <p>${escapeHtml(p.content).replace(/\n/g, "<br>")}</p>
          <div class="poem-actions">
            <button onclick="sharePoem('${escapeHtml(p.title)}', '${escapeHtml(p.content)}')">🔗 Share</button>
          </div>
        </div>
      `;

      feed.appendChild(div);
    });
  }, error => {
    console.error("Error loading poems:", error);
    document.getElementById("feed").innerHTML = '<div class="error">Error loading poems</div>';
  });

// 🔗 SHARE
function sharePoem(title, content) {
  let text = title + "\n\n" + content + "\n\n✨ From Ink & Paper";

  if (navigator.share) {
    navigator.share({ title, text });
  } else {
    window.open("https://wa.me/?text=" + encodeURIComponent(text));
  }
}

// 🛡️ XSS PREVENTION
function escapeHtml(text) {
  let div = document.createElement("div");
  div.textContent = text;
  return div.innerHTML;
}

</script>

</body>
</html>
