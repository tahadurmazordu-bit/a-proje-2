# a-proje-2
#  hello world
body {
  font-family: Arial, sans-serif;
  background: #0b1220;
  color: #e6e6e6;
  margin: 0;
}

.container {
  max-width: 420px;
  margin: 60px auto;
  padding: 24px;
  background: #121a2b;
  border: 1px solid #1f2a44;
  border-radius: 14px;
}

input, button {
  width: 100%;
  padding: 12px;
  margin-top: 10px;
  border-radius: 10px;
  border: 1px solid #2a3a60;
  background: #0b1220;
  color: #fff;
  box-sizing: border-box;
}

button {
  cursor: pointer;
  background: #2d6cdf;
  border: none;
  font-weight: 600;
}

button.secondary {
  background: #223055;
}

.small {
  opacity: 0.8;
  font-size: 14px;
  margin-top: 12px;
}
.error {
  color: #ff6b6b;
  margin-top: 10px;
}





<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>AI Site</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h1>AI Site</h1>
    <p class="small">Devam etmek için giriş yap.</p>

    <a href="login.html">
      <button>Giriş / Kayıt</button>
    </a>
  </div>
</body>
</html>





<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Giriş</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h2>Giriş</h2>

    <input id="email" type="email" placeholder="Email (kullanıcı)" />
    <input id="password" type="password" placeholder="Şifre" />

    <button id="loginBtn">Giriş Yap</button>
    <button id="signupBtn" class="secondary">Kayıt Ol</button>

    <div id="msg" class="error"></div>

    <p class="small">
      Not: GitHub Pages’te güvenli giriş için Firebase kullanıyoruz.
    </p>
  </div>

  <script type="module" src="auth.js"></script>
</body>
</html>






<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Panel</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h2>Hoş geldin</h2>
    <p id="userInfo" class="small"></p>

    <button id="logoutBtn" class="secondary">Çıkış Yap</button>

    <hr style="border-color:#1f2a44; margin:16px 0">

    <h3>AI Demo</h3>
    <p class="small">Buraya yapay zeka aracını (chat, resim üretme vs.) ekleyeceğiz.</p>
  </div>

  <script type="module" src="auth.js"></script>
</body>
</html>



// auth.js (ES Module)

// 1) Firebase SDK (CDN)
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.5/firebase-app.js";
import {
  getAuth,
  onAuthStateChanged,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut
} from "https://www.gstatic.com/firebasejs/10.12.5/firebase-auth.js";

// 2) Firebase config (BURAYI KENDİ PROJENDEN DOLDUR)
const firebaseConfig = {
  apiKey: "XXXX",
  authDomain: "XXXX.firebaseapp.com",
  projectId: "XXXX",
  appId: "XXXX"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);

// Sayfa tespiti
const path = location.pathname.split("/").pop(); // login.html, app.html vs.

// Login sayfası
if (path === "login.html") {
  const emailEl = document.getElementById("email");
  const passEl = document.getElementById("password");
  const msgEl = document.getElementById("msg");

  document.getElementById("loginBtn").addEventListener("click", async () => {
    msgEl.textContent = "";
    try {
      await signInWithEmailAndPassword(auth, emailEl.value, passEl.value);
      location.href = "app.html";
    } catch (e) {
      msgEl.textContent = hataMesaji(e);
    }
  });

  document.getElementById("signupBtn").addEventListener("click", async () => {
    msgEl.textContent = "";
    try {
      await createUserWithEmailAndPassword(auth, emailEl.value, passEl.value);
      location.href = "app.html";
    } catch (e) {
      msgEl.textContent = hataMesaji(e);
    }
  });
}

// App sayfası (korumalı)
if (path === "app.html") {
  const userInfo = document.getElementById("userInfo");
  const logoutBtn = document.getElementById("logoutBtn");

  onAuthStateChanged(auth, (user) => {
    if (!user) {
      // giriş yoksa login’e at
      location.href = "login.html";
      return;
    }
    userInfo.textContent = `Giriş yapan: ${user.email}`;
  });

  logoutBtn.addEventListener("click", async () => {
    await signOut(auth);
    location.href = "login.html";
  });
}

// Firebase hata mesajlarını biraz okunur yapalım
function hataMesaji(e) {
  const code = e?.code || "";
  if (code.includes("auth/invalid-email")) return "Email formatı geçersiz.";
  if (code.includes("auth/invalid-credential")) return "Email veya şifre hatalı.";
  if (code.includes("auth/user-not-found")) return "Kullanıcı bulunamadı.";
  if (code.includes("auth/wrong-password")) return "Şifre hatalı.";
  if (code.includes("auth/email-already-in-use")) return "Bu email zaten kayıtlı.";
  if (code.includes("auth/weak-password")) return "Şifre çok zayıf (en az 6 karakter).";
  return "Hata: " + (e?.message || "Bilinmeyen hata");
}
