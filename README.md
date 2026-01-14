<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kasir RM JOYO - Monitor Aktif</title>

<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');

  // Fungsi otomatis lapor ke dashboard Anda
  function kirimLaporan(aksi, info) {
    gtag('event', aksi, {
      'event_label': info,
      'nama_kasir': identitas.kasir,
      'nama_toko': identitas.toko
    });
    console.log("Memantau: " + aksi + " - " + info);
  }
</script>

<style>
body{margin:0;font-family:Arial,sans-serif;background:#f2f2f2;color:#000;padding-top:24px;transition:.3s}
body.dark{background:#121212;color:#eee}
.page{display:none;padding:10px}
.page.active{display:block}
button{width:100%;padding:10px;margin:5px 0;border:none;border-radius:10px;background:#42a5f5;color:#fff;font-size:16px;cursor:pointer;transition:0.2s}
button:hover{opacity:.85}
body.dark button{background:#1e88e5}
input,select{width:100%;padding:8px;margin:5px 0;border-radius:6px;border:1px solid #ccc;font-size:14px;transition:.3s}
body.dark input,body.dark select{background:#333;color:#eee;border-color:#888}

/* CARD CONTAINER */
.card{width:95%;max-width:700px;margin:15px auto;background:#fff;padding:15px;border-radius:14px;box-shadow:0 6px 20px rgba(0,0,0,.15);border:2px solid #42a5f5}
body.dark .card{background:#1e1e1e;border:2px solid #1e88e5}

/* HEADER */
.header{text-align:center;margin-bottom:10px}
.header-content{display:flex;align-items:center;justify-content:center;gap:15px;margin-bottom:10px}
#logoImg{max-height:60px;width:auto;border-radius:8px;display:none}

#displayNamaToko{
    font-weight:bold;
    font-size:26px;
    background: linear-gradient(90deg, #ff0000, #ffeb3b);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin:0;
}

.kategori{margin:15px 0 5px;font-weight:bold;color:#42a5f5;border-bottom:2px solid #42a5f5;display:inline-block}
.menu-item{display:flex;justify-content:space-between;align-items:center;padding:8px;background:#f9f9f9;border-radius:8px;margin-bottom:5px;}
.del{color:#aaa;font-size:14px;padding:0 8px;cursor:pointer;}
.order{display:flex;justify-content:space-between;align-items:center;padding:4px 0;border-bottom:1px solid #ddd;font-size:14px}
.scroll-box{max-height:300px;overflow-y:auto;padding-right:5px}
.marquee{position:fixed;top:0;left:0;width:100%;overflow:hidden;white-space:nowrap;font-size:12px;font-weight:bold;background:#000;color:#ffeb3b;z-index:1000}
.marquee span{display:inline-block;padding-left:100%;animation:jalan 15s linear infinite}
@keyframes jalan{100%{transform:translateX(-100%)}} 
</style>
</head>
<body>

<div class="marquee"><span id="runText">Loading Monitor...</span></div>

<div id="pageLogin" class="page active">
  <div class="card">
    <div class="header"><h2>LOGIN KASIR</h2></div>
    <input id="username" placeholder="Username (Gunakan '55555')">
    <input id="password" type="password" placeholder="Password">
    <button onclick="login()">MASUK</button>
  </div>
</div>

<div id="pageKasir" class="page">
  <div class="card">
    <div class="header">
      <div class="header-content">
        <img id="logoImg" src="" alt="Logo">
        <h2 id="displayNamaToko"></h2>
      </div>
      <small id="waktu"></small>
    </div>
    <div id="menuContainer" class="scroll-box"></div>
    <hr>
    <h3 style="text-align:center;">Daftar Pesanan</h3>
    <div id="list" class="scroll-box"></div>
    <div style="text-align:center;font-weight:bold;font-size:20px;margin:15px 0;">
      Total: Rp <span id="total">0</span>
    </div>
    <button onclick="bayar()">BAYAR & SIMPAN NOTA</button>
    <button style="background:#444;" onclick="showPage('pageRiwayat')">⚙ Pengaturan</button>
    <button style="background:#444;" onclick="showPage('pageRekap')">📋 Rekap Nota</button>
  </div>
</div>

<div id="pageRiwayat" class="page">
  <div class="card">
    <button onclick="showPage('pageKasir')">⬅ Kembali</button>
    <hr>
    <div class="header"><h3>⚙ Pengaturan Identitas</h3></div>
    <label>Nama Toko:</label>
    <input type="text" id="inputNamaToko" placeholder="Nama Toko">
    <label>Nama Kasir:</label>
    <input type="text" id="inputNamaKasir" placeholder="Nama Kasir">
    <button onclick="updateIdentitas()" style="background:#0288d1;">SIMPAN PERUBAHAN</button>
    <hr>
    <h3>Tambah Menu Baru</h3>
    <input type="text" id="namaBaru" placeholder="Nama Menu">
    <input type="number" id="hargaBaru" placeholder="Harga">
    <button onclick="tambahMenuBaru()" style="background:#2e7d32;">Tambah Menu</button>
    <button id="darkModeBtn" onclick="toggleDark()">Dark Mode</button>
    <div id="riwayat" style="margin-top:15px;"></div>
  </div>
</div>

<div id="pageRekap" class="page">
  <div class="card">
    <div class="header"><h2>NOTA PESAN</h2></div>
    <button onclick="showPage('pageKasir')">Kembali ke Kasir</button>
    <div id="rekapDiv"></div>
  </div>
</div>

<script>
let menuData = JSON.parse(localStorage.getItem("menuRM")) || [
  {nama:"Nasi Goreng",harga:15000,kategori:"Makanan"},
  {nama:"Teh Manis",harga:5000,kategori:"Minuman"}
];
let identitas = JSON.parse(localStorage.getItem("identitasRM")) || { toko:"RM JOYO", kasir:"KOES" };
let pesananData = [], darkMode = JSON.parse(localStorage.getItem("darkMode")) || false;

function login(){
  const u=document.getElementById("username").value;
  if(u==="55555" || u==="kasir"){ 
    showPage("pageKasir"); 
    renderMenu(); 
    applyIdentitas();
    // 2. MEMANTAU LOGIN
    kirimLaporan('Login', 'Kasir ' + identitas.kasir + ' mulai bekerja');
  } else {
    alert("Login Gagal!");
  }
}

function bayar(){ 
  if(pesananData.length===0) return alert("Pesanan kosong!"); 
  const r=JSON.parse(localStorage.getItem("riwayatRM")||"[]"); 
  const t=pesananData.reduce((a,b)=>a+b.harga*b.jumlah,0); 
  
  // Tetap Simpan Lokal di HP masing-masing
  r.push({waktu:new Date().toISOString(),pesanan:[...pesananData],total:t,kasir:identitas.kasir}); 
  localStorage.setItem("riwayatRM",JSON.stringify(r)); 
  
  // 3. MEMANTAU TRANSAKSI (Muncul di HP Anda)
  kirimLaporan('Transaksi_Baru', 'Total Rp ' + t + ' oleh ' + identitas.kasir);

  pesananData=[]; renderPesanan(); 
  alert("Tersimpan di HP & Terpantau Pusat!");
}

function applyIdentitas(){
  document.getElementById("displayNamaToko").innerText = identitas.toko;
  document.getElementById("inputNamaToko").value = identitas.toko;
  document.getElementById("inputNamaKasir").value = identitas.kasir;
}

function updateIdentitas(){
  identitas.toko=document.getElementById("inputNamaToko").value||"RM JOYO";
  identitas.kasir=document.getElementById("inputNamaKasir").value||"KOES";
  localStorage.setItem("identitasRM", JSON.stringify(identitas));
  applyIdentitas();
  kirimLaporan('Ganti_Nama', 'Identitas diubah jadi: ' + identitas.kasir);
  alert("Tersimpan!");
}

function renderMenu(){
  const c=document.getElementById("menuContainer"); c.innerHTML="";
  ["Makanan","Minuman","Jajanan"].forEach(k=>{
    const l = menuData.filter(m=>m.kategori===k);
    if(l.length){ c.innerHTML+=`<div class="kategori">${k}</div>`;
      l.forEach(m=>{
        const item=document.createElement("div"); item.className="menu-item";
        item.innerHTML=`<div style="flex:2; cursor:pointer;" onclick="tambahItem('${m.nama}',${m.harga})"><span>${m.nama}</span> <br> <small>Rp ${m.harga.toLocaleString()}</small></div>`;
        c.appendChild(item);
      });
    }
  });
}

function updateWaktu(){
  const d=new Date();
  document.getElementById("waktu").innerText=d.toLocaleString('id-ID');
  document.getElementById("runText").innerText=`MONITORING AKTIF: ${identitas.kasir.toUpperCase()} @ ${identitas.toko.toUpperCase()}... Hub: 087850876841 jika ada kendala`;
}
setInterval(updateWaktu,1000);

function showPage(id){
  document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
}

function tambahItem(n,h){ let a=pesananData.find(x=>x.nama===n); a? a.jumlah++: pesananData.push({nama:n,harga:h,jumlah:1}); renderPesanan();}
function renderPesanan(){ 
  const l=document.getElementById("list"); l.innerHTML=""; 
  let t=0; 
  pesananData.forEach((p,i)=>{ 
    t+=p.harga*p.jumlah; 
    l.innerHTML+=`<div class="order"><span><b>${p.nama}</b> (x${p.jumlah})</span><span>Rp ${(p.harga*p.jumlah).toLocaleString()}</span></div>`; 
  }); 
  document.getElementById("total").innerText=t.toLocaleString();
}

function toggleDark(){
  darkMode = !darkMode;
  localStorage.setItem("darkMode", darkMode);
  document.body.classList.toggle("dark", darkMode);
}

// Inisialisasi
applyIdentitas();
updateWaktu();
if(darkMode) document.body.classList.add("dark");
</script>
</body>
</html>
