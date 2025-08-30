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
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js";
    import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, deleteUser } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-auth.js";
    import { getFirestore, doc, setDoc, getDoc, deleteDoc } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore.js";
    import QRCode from "https://cdn.jsdelivr.net/npm/qrcodejs/qrcode.min.js";

    // 🔹 Replace with YOUR Firebase config
    const firebaseConfig = {
      apiKey: "AIzaSyDBoBZj-1bwW-H8WxDzWJK1EFFGenA-2Yg",
      authDomain: "tagback-50f4e.firebaseapp.com",
      projectId: "tagback-50f4e",
      storageBucket: "tagback-50f4e.appspot.com",
      messagingSenderId: "1029603108645",
      appId: "1:1029603108645:web:46f2811764a677716a1ccc"
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    let currentUserId = null;

    // Register
    async function register() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      try {
        const userCredential = await createUserWithEmailAndPassword(auth, email, password);
        currentUserId = userCredential.user.uid;
        await setDoc(doc(db, "users", currentUserId), { name: "", phone: "", extra: "" });
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
        const userCredential = await signInWithEmailAndPassword(auth, email, password);
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

      await setDoc(doc(db, "users", currentUserId), { name, phone, extra });

      const profileUrl = window.location.origin + window.location.pathname + "?id=" + currentUserId;

      document.getElementById("qrcode").innerHTML = "";
      new QRCode(document.getElementById("qrcode"), profileUrl);

      document.getElementById("link").innerHTML =
        "🔗 Your Unique Link: <br><a href='" + profileUrl + "' target='_blank'>" + profileUrl + "</a>";
    }

    // Logout
    async function logout() {
      await signOut(auth);
      document.getElementById("authCard").classList.remove("hidden");
      document.getElementById("formCard").classList.add("hidden");
    }

    // Delete Profile
    async function deleteProfile() {
      if (confirm("⚠️ Are you sure you want to delete your profile? This cannot be undone.")) {
        await deleteDoc(doc(db, "users", currentUserId));
        await deleteUser(auth.currentUser);
        alert("Profile deleted successfully.");
        location.reload();
      }
    }

    // Show profile form
    async function showProfile() {
      document.getElementById("authCard").classList.add("hidden");
      document.getElementById("formCard").classList.remove("hidden");

      const docSnap = await getDoc(doc(db, "users", currentUserId));
      if (docSnap.exists()) {
        const data = docSnap.data();
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
        let docSnap = await getDoc(doc(db, "users", userId));

        if (docSnap.exists()) {
          let data = docSnap.data();
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
