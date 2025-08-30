[10:08 pm, 29/8/2025] Rakchana .S:<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TagBack - Secure QR</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f4f6ff; text-align: center; padding: 20px; }
    .card { background: white; max-width: 420px; margin: 20px auto; padding: 20px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    input, button { width: 90%; padding: 10px; margin: 10px 0; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; }
    button { background: #0066cc; color: white; cursor: pointer; }
    button:hover { background: #004999; }
    #qrcode { margin-top: 20px; }
    .hidden { display: none; }
    .danger { background: #cc0000 !important; }
    .danger:hover { background: #990000 !important; }
  </style>
</head>
<body>

  <!-- Authentication -->
  <div class="card" id="authCard">
    <h1>TagBack Login</h1>
    <input type="email" id="email" placeholder="Enter Email" required>
    <input type="password" id="password" placeholder="Enter Password" required>
    <button onclick="register()">Register</button>
    <button onclick="login()">Login</button>
  </div>

  <!-- User Form -->
  <div class="card hidden" id="formCard">
    <h1>Your TagBack Profile</h1>
    <input type="text" id="name" placeholder="Enter Name">
    <input type="text" id="phone" placeholder="Enter Phone">
    <input type="text" id="extra" placeholder="Extra Info">
    <button onclick="saveProfile()">Save / Update</button>
    <button onclick="logout()">Logout</button>
    <button class="danger" onclick="deleteProfile()">❌ Delete Profile</button>
    <div id="qrcode"></div>
    <p id="link"></p>
  </div>

  <!-- Firebase + QR Code -->
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/qrcodejs/qrcode.min.js"></script>

  <script>
    // 🔹 Replace with YOUR Firebase config
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "YOUR_SENDER_ID",
      appId: "YOUR_APP_ID"
    };

    // Init Firebase
    const app = firebase.initializeApp(firebaseConfig);
    const auth = firebase.auth();
    const db = firebase.firestore();

    let currentUserId = null;

    // Register
    async function register() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      try {
        const userCredential = await auth.createUserWithEmailAndPassword(email, password);
        currentUserId = userCredential.user.uid;
        await db.collection("users").doc(currentUserId).set({ name: "", phone: "", extra: "" });
        showProfile();
      } catch (error) {
        alert(error.message);
      }
    }

    // Login
    async function login() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      try {
        const userCredential = await auth.signInWithEmailAndPassword(email, password);
        currentUserId = userCredential.user.uid;
        showProfile();
      } catch (error) {
        alert(error.message);
      }
    }

    // Save Profile
    async function saveProfile() {
      const name = document.getElementById("name").value;
      const phone = document.getElementById("phone").value;
      const extra = document.getElementById("extra").value;

      await db.collection("users").doc(currentUserId).set({ name, phone, extra });

      const profileUrl = window.location.origin + window.location.pathname + "?id=" + currentUserId;

      document.getElementById("qrcode").innerHTML = "";
      new QRCode(document.getElementById("qrcode"), profileUrl);

      document.getElementById("link").innerHTML =
        "🔗 Your Unique Link: <br><a href='" + profileUrl + "' target='_blank'>" + profileUrl + "</a>";
    }

    // Logout
    function logout() {
      auth.signOut();
      document.getElementById("authCard").classList.remove("hidden");
      document.getElementById("formCard").classList.add("hidden");
    }

    // Delete Profile
    async function deleteProfile() {
      if (confirm("⚠️ Are you sure you want to delete your profile? This cannot be undone.")) {
        await db.collection("users").doc(currentUserId).delete();
        const user = auth.currentUser;
        await user.delete();
        alert("Profile deleted successfully.");
        location.reload();
      }
    }

    // Show profile form
    async function showProfile() {
      document.getElementById("authCard").classList.add("hidden");
      document.getElementById("formCard").classList.remove("hidden");

      const doc = await db.collection("users").doc(currentUserId).get();
      if (doc.exists) {
        const data = doc.data();
        document.getElementById("name").value = data.name;
        document.getElementById("phone").value = data.phone;
        document.getElementById("extra").value = data.extra;

        const profileUrl = window.location.origin + window.location.pathname + "?id=" + currentUserId;
        document.getElementById("qrcode").innerHTML = "";
        new QRCode(document.getElementById("qrcode"), profileUrl);
        document.getElementById("link").innerHTML =
          "🔗 Your Unique Link: <br><a href='" + profileUrl + "' target='_blank'>" + profileUrl + "</a>";
      }
    }

    // Profile View Mode (when scanning QR)
    window.onload = async function() {
      const urlParams = new URLSearchParams(window.location.search);
      if (urlParams.has("id")) {
        let userId = urlParams.get("id");
        let doc = await db.collection("users").doc(userId).get();

        if (doc.exists) {
          let data = doc.data();
          document.body.innerHTML = `
            <div class="card">
              <h1>TagBack Profile</h1>
              <p><b>Name:</b> ${data.name}</p>
              <p><b>Phone:</b> ${data.phone}</p>
              <p><b>Extra:</b> ${data.extra}</p>
              <a href="tel:${data.phone}" style="display:inline-block;background:#0066cc;color:white;padding:10px 20px;border-radius:8px;text-decoration:none;">📞 Call</a>
            </div>
          `;
        } else {
          document.body.innerHTML = "<h2>User not found ❌</h2>";
        }
      }
    }
  </script>
</body>
</html>
