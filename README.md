
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Kasir RM JOYO - Monitor Firebase</title>

<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

<script>
  // Konfigurasi sesuai gambar konsol Anda
  const firebaseConfig = {
    apiKey: "AIzaSyArr8xf-sOxRw7qAQMbi49...", 
    authDomain: "monitoring-rm-joyo.firebaseapp.com",
    projectId: "monitoring-rm-joyo",
    storageBucket: "monitoring-rm-joyo.appspot.com",
    messagingSenderId: "461131074192",
    appId: "1:461131074192:web:2d71007c6...",
    measurementId: "G-87L5CZGF2E",
    databaseURL: "https://monitoring-rm-joyo-default-rtdb.firebaseio.com" // Sesuaikan dengan URL Database Anda
  };

  // Inisialisasi Firebase
  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  // Fungsi Monitoring: Kirim data transaksi ke Cloud
  function kirimKeCloud(data) {
    const trxId = "TRX-" + Date.now();
    db.ref('penjualan/' + trxId).set(data)
      .then(() => console.log("Berhasil Terkirim ke Monitor Pusat"))
      .catch((err) => console.error("Gagal Monitor:", err));
  }
</script>

<style>
/* CSS: Tampilan Dioptimalkan */
:root { --biru: #42a5f5; --gelap: #121212; --hijau: #2e7d32; }
body { margin:0; font-family:'Segoe UI', Arial, sans-serif; background:#f4f7f6; padding-top:30px; transition:0.3s; }
body.dark { background: var(--gelap); color: #eee; }

.page { display:none; padding:10px; }
.page.active { display:block; animation: fadeIn 0.3s; }
@keyframes fadeIn { from {opacity:0} to {opacity:1} }

.card { width:95%; max-width:500px; margin:auto; background:#fff; padding:20px; border-radius:15px; box-shadow:0 8px 20px rgba(0,0,0,0.1); border-top:6px solid var(--biru); box-sizing:border-box; }
body.dark .card { background: #1e1e1e; border-top-color: #1e88e5; }

button { width:100%; padding:14px; margin:8px 0; border:none; border-radius:10px; background: var(--biru); color:#fff; font-size:16px; font-weight:bold; cursor:pointer; }
input { width:100%; padding:12px; margin:8px 0; border-radius:8px; border:1px solid #ddd; box-sizing:border-box; }

.menu-grid { display:grid; grid-template-columns: 1fr 1fr; gap:10px; margin:15px 0; max-height: 250px; overflow-y: auto; }
.btn-item { background:#e3f2fd; border:2px solid var(--biru); padding:12px; border-radius:12px; text-align:center; font-weight:bold; cursor:pointer; font-size:14px; }
body.dark .btn-item { background:#262626; color:#90caf9; }

.list-box { background:#f9f9f9; padding:10px; border-radius:10px; min-height:50px; margin:10px 0; font-size:14px; }
body.dark .list-box { background:#2a2a2a; }
.item-row { display:flex; justify-content:space-between; padding:5px 0; border-bottom:1px solid #eee; }

.marquee { position:fixed; top:0; left:0; width:100%; background:#000; color:#ffeb3b; padding:5px 0; font-size:11px; z-index:1000; overflow:hidden; }
.marquee span { display:inline-block; white-space:nowrap; animation:jalan 15s linear infinite; }
@keyframes jalan { 0% {transform:translateX(100%)} 100% {transform:translateX(-100%)} }
</style>
</head>
<body>

<div class="marquee"><span id="runText">SISTEM MONITORING CLOUD RM JOYO AKTIF...</span></div>

<div id="pageLogin" class="page active">
  <div class="card">
    <h2 style="text-align:center">LOGIN KASIR</h2>
    <input id="pin" type="password" placeholder="Masukkan PIN (55555)">
    <button onclick="login()">MASUK</button>
  </div>
</div>

<div id="pageKasir" class="page">
  <div class="card">
    <div style="text-align:center">
        <h2 id="dispToko" style="margin:0; color:var(--biru)"></h2>
        <small id="dispJam"></small>
    </div>

    <div id="menuContainer" class="menu-grid"></div>

    <div class="list-box" id="orderList"></div>

    <div style="text-align:right; font-size:22px; font-weight:bold; margin:15px 0; color:var(--hijau)">
        Total: Rp <span id="totalHarga">0</span>
    </div>

    <button onclick="prosesBayar()" style="background:var(--hijau)">SIMPAN & MONITOR PUSAT</button>
    <div style="display:flex; gap:10px">
        <button style="background:#555" onclick="showPage('pageAdmin')">⚙ Menu</button>
        <button style="background:#555" onclick="showPage('pageRekap')">📋 Rekap</button>
    </div>
  </div>
</div>

<div id="pageAdmin" class="page">
  <div class="card">
    <button onclick="showPage('pageKasir')">⬅ Kembali</button>
    <h3>Tambah Menu Baru</h3>
    <input id="nBaru" placeholder="Nama Menu">
    <input id="hBaru" type="number" placeholder="Harga (Rp)">
    <button onclick="addMenu()">TAMBAHKAN</button>
    <hr>
    <button onclick="toggleDark()" style="background:#444">Mode Gelap/Terang</button>
    <button onclick="resetApp()" style="background:#d32f2f">RESET DATA</button>
  </div>
</div>

<div id="pageRekap" class="page">
  <div class="card">
    <button onclick="showPage('pageKasir')">⬅ Kembali</button>
    <h3>Riwayat Lokal Hari Ini</h3>
    <div id="rekapDiv" style="max-height:350px; overflow-y:auto; font-size:13px"></div>
  </div>
</div>

<script>
let menuData = JSON.parse(localStorage.getItem("menuRM")) || [
    {nama:"Nasi Goreng", harga:15000}, {nama:"Teh Manis", harga:5000}
];
let keranjang = [], darkMode = localStorage.getItem("dark") === "true";
let iden = JSON.parse(localStorage.getItem("identitasRM")) || {toko:"RM JOYO", kasir:"KOES"};

function login(){
    if(document.getElementById("pin").value === "55555"){
        showPage('pageKasir');
        renderMenu();
    } else { alert("PIN Salah!"); }
}

function renderMenu(){
    const c = document.getElementById("menuContainer");
    c.innerHTML = "";
    menuData.forEach(m => {
        const d = document.createElement("div");
        d.className = "btn-item";
        d.innerHTML = `${m.nama}<br>Rp ${m.harga.toLocaleString()}`;
        d.onclick = () => {
            let ada = keranjang.find(x => x.nama === m.nama);
            ada ? ada.qty++ : keranjang.push({nama:m.nama, harga:m.harga, qty:1});
            renderOrder();
        };
        c.appendChild(d);
    });
    document.getElementById("dispToko").innerText = iden.toko;
}

function renderOrder(){
    const list = document.getElementById("orderList");
    list.innerHTML = "";
    let total = 0;
    keranjang.forEach((item, i) => {
        total += item.harga * item.qty;
        list.innerHTML += `<div class="item-row">
            <span><b onclick="hapusItem(${i})" style="color:red; cursor:pointer;">[✕]</b> ${item.nama} x${item.qty}</span>
            <span>Rp ${(item.harga * item.qty).toLocaleString()}</span>
        </div>`;
    });
    document.getElementById("totalHarga").innerText = total.toLocaleString();
}

function hapusItem(i){ keranjang.splice(i,1); renderOrder(); }

function prosesBayar(){
    if(keranjang.length === 0) return alert("Keranjang kosong!");
    let total = keranjang.reduce((a,b) => a + (b.harga * b.qty), 0);
    
    let nota = {
        waktu: new Date().toLocaleString('id-ID'),
        kasir: iden.kasir,
        toko: iden.toko,
        total: total,
        items: [...keranjang]
    };

    // 1. Simpan Lokal
    let riwayat = JSON.parse(localStorage.getItem("riwayatRM") || "[]");
    riwayat.push(nota);
    localStorage.setItem("riwayatRM", JSON.stringify(riwayat));

    // 2. MONITORING CLOUD (FIREBASE)
    kirimKeCloud(nota);

    alert("Transaksi Berhasil & Terpantau Pusat!");
    keranjang = []; renderOrder();
}

function addMenu(){
    let n = document.getElementById("nBaru").value;
    let h = parseInt(document.getElementById("hBaru").value);
    if(n && h){
        menuData.push({nama:n, harga:h});
        localStorage.setItem("menuRM", JSON.stringify(menuData));
        renderMenu();
        alert("Menu Ditambah!");
    }
}

function showPage(id){
    document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
    document.getElementById(id).classList.add('active');
    if(id==='pageRekap') renderRekap();
}

function renderRekap(){
    let r = JSON.parse(localStorage.getItem("riwayatRM") || "[]");
    document.getElementById("rekapDiv").innerHTML = r.reverse().map(n => `
        <div style="border-bottom:1px solid #ddd; padding:8px 0;">
            <b>${n.waktu}</b> - Rp ${n.total.toLocaleString()}<br>
            <small>${n.items.map(i=>i.nama+'x'+i.qty).join(', ')}</small>
        </div>
    `).join('');
}

function toggleDark(){
    darkMode = !darkMode;
    document.body.classList.toggle("dark", darkMode);
    localStorage.setItem("dark", darkMode);
}

function resetApp(){ if(confirm("Hapus semua data?")){ localStorage.clear(); location.reload(); } }

setInterval(() => {
    document.getElementById("dispJam").innerText = new Date().toLocaleString('id-ID');
}, 1000);

if(darkMode) document.body.classList.add("dark");
</script>
</body>
</html>
