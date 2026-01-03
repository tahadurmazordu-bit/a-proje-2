import { useState, useEffect } from "react";
import { initializeApp } from "firebase/app";
import {
  getAuth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut,
} from "firebase/auth";
import { getFirestore, collection, addDoc } from "firebase/firestore";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

// 🔥 Firebase Config (KENDİ BİLGİLERİNLE DEĞİŞTİR)
const firebaseConfig = {
  apiKey: "FIREBASE_API_KEY",
  authDomain: "PROJECT.firebaseapp.com",
  projectId: "PROJECT_ID",
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

const ADMIN_USERNAME = "tahadurmaz";

// 🔐 Şifre kuralı: en az 6 karakter + özel karakter
const isPasswordValid = (pw: string) => pw.length >= 6 && /[.!@#$%^&*]/.test(pw);

export default function AIPlatformUltimate() {
  const [page, setPage] = useState<
    "login" | "register" | "dashboard" | "settings" | "adminSettings" | "logs" | "adminUsers"
  >("login");

  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [newPassword, setNewPassword] = useState("");
  const [newAdmin, setNewAdmin] = useState("");

  const [themeColor, setThemeColor] = useState(() => localStorage.getItem("themeColor") || "gray");
  const [aiProvider, setAiProvider] = useState(() => localStorage.getItem("aiProvider") || "openai");
  const [maintenance, setMaintenance] = useState(() => localStorage.getItem("maintenance") === "true");
  const [maintenanceMessage, setMaintenanceMessage] = useState(() =>
    localStorage.getItem("maintenanceMessage") || "Site güncelleniyor"
  );

  const [logs, setLogs] = useState<string[]>(() => JSON.parse(localStorage.getItem("logs") || "[]"));

  const [users, setUsers] = useState<{ username: string; password: string }[]>(() =>
    JSON.parse(localStorage.getItem("users") || "[]")
  );

  const [admins, setAdmins] = useState<string[]>(() =>
    JSON.parse(localStorage.getItem("admins") || `["${ADMIN_USERNAME}"]`)
  );

  const [currentUser, setCurrentUser] = useState<string | null>(() => localStorage.getItem("currentUser"));

  const isAdmin = currentUser ? admins.includes(currentUser) : false;

  // 🧩 Admin paneli ana siteyle entegre (ayrı URL yok)
  // Admin paneline sadece dashboard içinden girilir

  const addLog = (msg: string) => {
    const entry = `[${new Date().toLocaleString()}] ${msg}`;
    setLogs(prev => [entry, ...prev]);
  };

  useEffect(() => {
    localStorage.setItem("themeColor", themeColor);
    localStorage.setItem("aiProvider", aiProvider);
    localStorage.setItem("maintenance", String(maintenance));
    localStorage.setItem("maintenanceMessage", maintenanceMessage);
    localStorage.setItem("logs", JSON.stringify(logs));
    localStorage.setItem("admins", JSON.stringify(admins));
    localStorage.setItem("currentUser", currentUser || "");
    localStorage.setItem("users", JSON.stringify(users));
  }, [themeColor, aiProvider, maintenance, maintenanceMessage, logs, admins, currentUser, users]);

  const register = async () => {
    if (!username || !password) return alert("Bilgileri doldur");
    if (!isPasswordValid(password)) return alert("Şifre geçersiz");
    try {
      await createUserWithEmailAndPassword(auth, `${username}@site.com`, password);
      await addDoc(collection(db, "users"), { username, role: "user" });
      setUsers([...users, { username, password }]);
      setPage("login");
    } catch (e: any) {
      alert(e.message);
    }
  };

  const login = async () => {
    if (maintenance && !isAdmin) return alert("Site bakımda");
    try {
      await signInWithEmailAndPassword(auth, `${username}@site.com`, password);
      setCurrentUser(username);
      addLog(`${username} giriş yaptı`);
      setPage("dashboard");
    } catch {
      alert("Giriş hatalı");
    }
  };

  const logout = async () => {
    await signOut(auth);
    setCurrentUser(null);
    setPage("login");
  };

  const addAdmin = () => {
    if (!newAdmin) return alert("Kullanıcı adı gir");
    if (!users.find(u => u.username === newAdmin)) return alert("Kullanıcı yok");
    if (admins.includes(newAdmin)) return alert("Zaten admin");
    setAdmins([...admins, newAdmin]);
    addLog(`${newAdmin} admin yapıldı`);
    setNewAdmin("");
  };

  const removeAdmin = (u: string) => {
    if (u === ADMIN_USERNAME) return alert("Ana admin kaldırılamaz");
    setAdmins(admins.filter(a => a !== u));
    addLog(`${u} adminlikten çıkarıldı`);
  };

  const changePassword = () => {
    if (!isPasswordValid(newPassword)) return alert("Şifre geçersiz");
    setUsers(users.map(u => (u.username === currentUser ? { ...u, password: newPassword } : u)));
    setNewPassword("");
  };

  return (
    <div className="min-h-screen flex items-center justify-center p-4">
      {page === "login" && (
        <Card className="max-w-sm w-full">
          <CardContent className="p-4 flex flex-col gap-4">
            <Input placeholder="Kullanıcı Adı" onChange={e => setUsername(e.target.value)} />
            <Input type="password" placeholder="Şifre" onChange={e => setPassword(e.target.value)} />
            <Button onClick={login}>Giriş</Button>
            <Button variant="outline" onClick={() => setPage("register")}>Kaydol</Button>
          </CardContent>
        </Card>
      )}

      {page === "register" && (
        <Card className="max-w-sm w-full">
          <CardContent className="p-4 flex flex-col gap-4">
            <Input placeholder="Kullanıcı Adı" onChange={e => setUsername(e.target.value)} />
            <Input type="password" placeholder="Şifre" onChange={e => setPassword(e.target.value)} />
            <Button onClick={register}>Kayıt Ol</Button>
            <Button variant="outline" onClick={() => setPage("login")}>Geri</Button>
          </CardContent>
        </Card>
      )}

      {page === "dashboard" && currentUser && (
        <Card className="max-w-md w-full">
          <CardContent className="p-4 flex flex-col gap-3">
            <h1>Hoşgeldin {currentUser}</h1>
            <Button onClick={() => setPage("settings")}>Ayarlar</Button>
            {currentUser === ADMIN_USERNAME && (
              <Button
                onClick={() => {
                  
                  setPage("adminUsers");
                }}
              >
                Admin Yönetimi
              </Button>
            )}
            <Button variant="destructive" onClick={logout}>Çıkış</Button>
          </CardContent>
        </Card>
      )}

      {page === "settings" && (
        <Card className="max-w-md w-full">
          <CardContent className="p-4 flex flex-col gap-3">
            <Input type="password" placeholder="Yeni Şifre" value={newPassword} onChange={e => setNewPassword(e.target.value)} />
            <Button onClick={changePassword}>Şifre Değiştir</Button>
            <Button variant="outline" onClick={() => setPage("dashboard")}>Geri</Button>
          </CardContent>
        </Card>
      )}

      {page === "adminUsers" && currentUser === ADMIN_USERNAME && (
        <Card className="max-w-md w-full">
          <CardContent className="p-4 flex flex-col gap-3">
            <Input placeholder="Yeni admin" value={newAdmin} onChange={e => setNewAdmin(e.target.value)} />
            <Button onClick={addAdmin}>Admin Ekle</Button>
            {admins.map(a => (
              <div key={a} className="flex justify-between">
                <span>{a}</span>
                {a !== ADMIN_USERNAME && <Button size="sm" onClick={() => removeAdmin(a)}>Sil</Button>}
              </div>
            ))}
            <Button variant="outline" onClick={() => setPage("dashboard")}>Geri</Button>
          </CardContent>
        </Card>
      )}
    </div>
  );
}
