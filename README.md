<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>TagBack – Secure QR</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    :root { --bg:#0b1020; --card:#121936; --ink:#e6e9ff; --muted:#9aa3c7; --accent:#6ea8ff; --ok:#2ecc71; --err:#ff6b6b; }
    *{box-sizing:border-box;font-family:Inter,system-ui,Segoe UI,Roboto,Arial}
    body{margin:0;background:linear-gradient(180deg,#0b1020,#0e1440 60%,#0b1020);color:var(--ink)}
    .wrap{max-width:980px;margin:24px auto;padding:16px}
    .card{background:var(--card);border:1px solid #1b2552;border-radius:20px;box-shadow:0 20px 40px rgba(0,0,0,.3);padding:24px}
    h1{font-size:28px;margin:0 0 16px}
    h2{font-size:18px;color:var(--muted);font-weight:600;margin:0 0 12px}
    label{display:block;font-size:13px;color:var(--muted);margin:14px 0 6px}
    input{width:100%;padding:12px 14px;border-radius:12px;border:1px solid #2a376f;background:#0e1440;color:var(--ink);outline:none}
    input:focus{border-color:var(--accent);box-shadow:0 0 0 4px rgba(110,168,255,.15)}
    .row{display:grid;grid-template-columns:1fr 1fr;gap:16px}
    .btn{appearance:none;display:inline-flex;align-items:center;gap:10px;padding:12px 16px;border-radius:12px;border:1px solid #2a376f;background:#18214a;color:#fff;font-weight:600;cursor:pointer}
    .btn.primary{background:linear-gradient(180deg,#4f86ff,#2f65f5);border-color:#2f65f5}
    .btn.ghost{background:transparent}
    .btn:disabled{opacity:.6;cursor:not-allowed}
    .step{display:none}
    .step.active{display:block}
    .qr-wrap{display:flex;align-items:center;justify-content:center;padding:20px;background:#0e1440;border:1px dashed #33407a;border-radius:16px}
    .muted{color:var(--muted)}
    .pill{display:inline-block;padding:6px 10px;border-radius:999px;border:1px solid #33407a;color:#9fb0ff;font-size:12px}
    .success{color:var(--ok)}
    .error{color:var(--err)}
    .center{text-align:center}
    .footer{margin-top:18px;color:#6f7bb3;font-size:12px}
    a{color:#a9c5ff}
    .hidden{display:none}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <div id="mode-create" class="step">
        <h1>TagBack – Create Your QR Tag</h1>
        <p class="muted">Register once, enter the contact to show when someone scans, and get a printable QR tag for your valuables.</p><h2>1) Register</h2>
    <div class="row">
      <div>
        <label>Your Name</label>
        <input id="regName" type="text" placeholder="e.g., Priya K" />
      </div>
      <div>
        <label>Email</label>
        <input id="regEmail" type="email" placeholder="you@example.com" />
      </div>
    </div>

    <h2 style="margin-top:18px">2) Tag Info (Shown to Finder)</h2>
    <div class="row">
      <div>
        <label>Name to Display</label>
        <input id="tagName" type="text" placeholder="Owner's name on tag" />
      </div>
      <div>
        <label>Phone Number</label>
        <input id="tagPhone" type="tel" placeholder="e.g., +91 98765 43210" />
      </div>
    </div>

    <div style="margin-top:18px;display:flex;gap:10px;align-items:center">
      <button id="btnCreate" class="btn primary">Generate QR</button>
      <span id="status" class="muted"></span>
    </div>

    <div id="qrSection" class="hidden" style="margin-top:22px">
      <h2>3) Your QR Code</h2>
      <div class="qr-wrap"><div id="qrcode"></div></div>
      <div class="center" style="margin-top:14px">
        <button id="btnPrint" class="btn">Print QR</button>
        <button id="btnDownload" class="btn ghost">Download PNG</button>
      </div>
      <p class="footer center">This QR opens a secure page with your chosen contact only. You can create multiple tags—one per item.</p>
      <div id="shareLinkWrap" class="center muted"></div>
    </div>
  </div>

  <div id="mode-view" class="step">
    <div class="pill">Found Item</div>
    <h1>Contact the Owner</h1>
    <p class="muted">This item has a TagBack code. Please use the details below to reach the owner.</p>
    <div id="viewBox" class="card" style="background:#0f1640;border-radius:16px;border:1px solid #2a376f">
      <h2 id="vName">Loading…</h2>
      <p id="vPhone" style="font-size:22px;margin:6px 0 0">&nbsp;</p>
      <div id="callBtns" style="margin-top:12px"></div>
    </div>
    <p class="footer">If you prefer not to call, you can also message the owner using your dialer or SMS app.</p>
  </div>
</div>

<p class="footer center">Made with ❤ for easy returns. – TagBack</p>

  </div>  <!-- QRCode library -->  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js" integrity="sha512-Bc5sZ3H0S6+QKneCkaoMBm3f2uC6rU4GQ3q3y2kzXv0l9G8zB+QH8wUEZ9q3nZ9cp1A2vS6H4k+F7G2LrC5z+Q==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>  <!-- Firebase SDKs (modular) -->  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
    import { getFirestore, collection, addDoc, doc, getDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

    const firebaseConfig = {
    apiKey: "AIzaSyAND6PPkKxns0ebU71h7TAKTI3RGFtmql4",
    authDomain: "tagback-quick.firebaseapp.com",
    projectId: "tagback-quick",
    storageBucket: "tagback-quick.firebasestorage.app",
    messagingSenderId: "422442680722",
    appId: "1:422442680722:web:72756331a788f87e9f014b",
    measurementId: "G-XJWH5K3F3C"
  };

  // Initialize Firebase
  const app = initializeApp(firebaseConfig);
  const analytics = getAnalytics(app);
</script>

    // --- Elements ---
    const modeCreate = document.getElementById('mode-create');
    const modeView = document.getElementById('mode-view');
    const statusEl = document.getElementById('status');
    const qrSection = document.getElementById('qrSection');
    const regName = document.getElementById('regName');
    const regEmail = document.getElementById('regEmail');
    const tagName = document.getElementById('tagName');
    const tagPhone = document.getElementById('tagPhone');
    const btnCreate = document.getElementById('btnCreate');
    const btnPrint = document.getElementById('btnPrint');
    const btnDownload = document.getElementById('btnDownload');
    const qrcodeBox = document.getElementById('qrcode');
    const shareLinkWrap = document.getElementById('shareLinkWrap');

    const vName = document.getElementById('vName');
    const vPhone = document.getElementById('vPhone');
    const callBtns = document.getElementById('callBtns');

    // --- Routing: create mode vs view mode ---
    const params = new URLSearchParams(location.search);
    const tagId = params.get('id');

    function showCreate(){ modeCreate.classList.add('active'); modeView.classList.remove('active'); }
    function showView(){ modeView.classList.add('active'); modeCreate.classList.remove('active'); }

    if (tagId) {
      showView();
      renderView(tagId).catch(err => {
        vName.textContent = 'Tag not found';
        vPhone.textContent = '';
        callBtns.innerHTML = '<p class="error">This tag may have been deleted or never existed.</p>';
        console.error(err);
      });
    } else {
      showCreate();
    }

    // --- Create Tag Flow ---
    btnCreate?.addEventListener('click', async () => {
      const ownerName = regName.value.trim();
      const email = regEmail.value.trim();
      const displayName = tagName.value.trim();
      const phone = tagPhone.value.trim();

      if (!ownerName || !email || !displayName || !phone) {
        flash('Please fill all fields to continue.', true); return;
      }

      try {
        btnCreate.disabled = true;
        flash('Saving…');
        const docRef = await addDoc(collection(db, 'tags'), {
          ownerName, email, displayName, phone,
          createdAt: serverTimestamp()
        });

        const link = `${location.origin}${location.pathname}?id=${docRef.id}`;
        makeQR(link);
        qrSection.classList.remove('hidden');
        flash('QR ready! Print or download below.');
        shareLinkWrap.innerHTML = `Link encoded in QR: <br><a href="${link}">${link}</a>`;
      } catch (e) {
        console.error(e);
        flash('Failed to save. Check your Firebase config & Firestore rules.', true);
      } finally {
        btnCreate.disabled = false;
      }
    });

    // --- View Mode ---
    async function renderView(id){
      const snap = await getDoc(doc(db, 'tags', id));
      if (!snap.exists()) throw new Error('not-found');
      const data = snap.data();
      vName.textContent = data.displayName || 'Owner';
      vPhone.textContent = data.phone || '';
      callBtns.innerHTML = '';
      if (data.phone) {
        const tel = document.createElement('a');
        tel.href = `tel:${data.phone.replace(/\s+/g,'')}`;
        tel.className = 'btn primary';
        tel.textContent = 'Call Now';
        callBtns.appendChild(tel);
        const sms = document.createElement('a');
        sms.href = `sms:${data.phone.replace(/\s+/g,'')}`;
        sms.className = 'btn';
        sms.textContent = 'Send SMS';
        callBtns.appendChild(sms);
      }
    }

    // --- Helpers ---
    function flash(msg, isErr=false){
      statusEl.textContent = msg;
      statusEl.className = isErr ? 'error' : 'success';
    }

    function makeQR(text){
      qrcodeBox.innerHTML = '';
      const qr = new QRCode(qrcodeBox, { text, width: 220, height: 220, correctLevel: QRCode.CorrectLevel.H });
      // Download PNG
      btnDownload.onclick = () => {
        const img = qrcodeBox.querySelector('img') || qrcodeBox.querySelector('canvas');
        const link = document.createElement('a');
        link.download = 'tagback-qr.png';
        link.href = (img.tagName === 'IMG') ? img.src : img.toDataURL('image/png');
        link.click();
      };
      // Print QR only
      btnPrint.onclick = () => {
        const w = window.open('', 'PRINT', 'height=600,width=480');
        w.document.write(`<!doctype html><html><head><title>Print QR</title></head><body style="display:flex;align-items:center;justify-content:center;height:100vh">${qrcodeBox.innerHTML}</body></html>`);
        w.document.close();
        w.focus();
        w.print();
        w.close();
      };
    }
  </script></body>
</html>
