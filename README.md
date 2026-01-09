<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kasir RM JOYO</title>
<style>
body{margin:0;font-family:Arial,sans-serif;background:#f2f2f2;color:#000;padding-top:24px;transition:.3s}
body.dark{background:#121212;color:#eee}
.page{display:none;padding:15px}
.page.active{display:block}
button{width:100%;padding:10px;margin:5px 0;border:none;border-radius:10px;background:#42a5f5;color:#fff;font-size:16px;cursor:pointer;transition:0.2s}
button:hover{opacity:0.85}
body.dark button{background:#1e88e5}
input,select{width:100%;padding:8px;margin:5px 0;border-radius:6px;border:1px solid #ccc;font-size:14px;transition:0.3s}
body.dark input,body.dark select{background:#333;color:#eee;border-color:#888}

/* CARD / CONTAINER */
.card{width:90%;max-width:700px;margin:15px auto;background:#fff;padding:15px;border-radius:14px;box-shadow:0 6px 20px rgba(0,0,0,.15);border:2px solid #42a5f5}
body.dark .card{background:#1e1e1e;border:2px solid #1e88e5}

.header{text-align:center;margin-bottom:10px}

/* Judul gradient */
#displayNamaToko{
    font-weight:bold;
    font-size:28px;
    background: linear-gradient(90deg, #ff0000, #ffeb3b); /* merah → kuning */
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-align:center;
    margin-bottom:10px;
    transition: background 1s ease;
}

/* Kategori menu */
.kategori{margin:15px 0 5px;font-weight:bold;color:#42a5f5;border-bottom:2px solid #42a5f5;display:inline-block}
body.dark .kategori{color:#1e88e5;border-bottom-color:#1e88e5}

/* Item menu */
.menu-item{display:flex;justify-content:space-between;align-items:center;padding:6px;background:#f9f9f9;border-radius:8px;margin-bottom:5px;transition:0.2s}
.menu-item:hover{background:#e0f7fa}
body.dark .menu-item{background:#2a2a2a}
body.dark .menu-item:hover{background:#333}
.menu-info{flex:2;display:flex;justify-content:space-between;margin-right:15px;cursor:pointer}

/* Tombol hapus */
.del{color:#aaa;font-size:14px;padding:0 8px;cursor:pointer;transition:0.2s}
.del:hover{color:red}

/* Pesanan rapat */
.order{display:flex;justify-content:space-between;align-items:center;padding:4px 0;border-bottom:1px solid #ddd;font-size:14px}
body.dark .order{border-bottom-color:#555}

/* Scroll container */
.scroll-box{max-height:300px;overflow-y:auto;padding-right:5px}
.scroll-box::-webkit-scrollbar{width:6px}
.scroll-box::-webkit-scrollbar-thumb{background:#888;border-radius:3px}
body.dark .scroll-box::-webkit-scrollbar-thumb{background:#555}

/* MARQUEE / RUNNING TEXT kuning */
.marquee{position:fixed;top:0;left:0;width:100%;overflow:hidden;white-space:nowrap;font-size:12px;font-weight:bold;background:#000;color:#ffeb3b;z-index:1000}
.marquee span{display:inline-block;padding-left:100%;animation:jalan 15s linear infinite}
@keyframes jalan{100%{transform:translateX(-100%)}} 

/* REKAP NOTA BORDER */
div[style*="border-left:4px solid"]{border-left:4px solid #42a5f5}
</style>
</head>
<body>

<div class="marquee"><span id="runText">Loading...</span></div>

<!-- LOGIN -->
<div id="pageLogin" class="page active">
  <div class="card">
    <div class="header"><h2>LOGIN KASIR</h2></div>
    <input id="username" placeholder="Username">
    <input id="password" type="password" placeholder="Password (kosong untuk 55555)">
    <button onclick="login()">MASUK</button>
  </div>
</div>

<!-- KASIR -->
<div id="pageKasir" class="page">
  <div class="card">
    <div class="header">
      <h2 id="displayNamaToko"></h2>
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

<!-- RIWAYAT / PENGATURAN -->
<div id="pageRiwayat" class="page">
  <div class="card">
    <div class="header"><h3>⚙ Pengaturan Identitas</h3></div>
    <input type="text" id="inputNamaToko" placeholder="Nama Toko">
    <input type="text" id="inputNamaKasir" placeholder="Nama Kasir">
    <button onclick="updateIdentitas()" style="background:#0288d1;">SIMPAN PERUBAHAN</button>
    <hr>
    <h3>Tambah Menu Baru</h3>
    <input type="text" id="namaBaru" placeholder="Nama Menu">
    <input type="number" id="hargaBaru" placeholder="Harga">
    <select id="kategoriBaru">
      <option value="Makanan">Makanan</option>
      <option value="Minuman">Minuman</option>
      <option value="Jajanan">Jajanan</option>
    </select>
    <button onclick="tambahMenuBaru()" style="background:#2e7d32;">Tambah Menu</button>
    <button id="darkModeBtn" onclick="toggleDark()">Dark Mode</button>
    <button onclick="resetRiwayat()" style="background:#d32f2f;">Hapus Semua Transaksi</button>
    <button onclick="showPage('pageKasir')">Kembali ke Kasir</button>
    <div id="riwayat" style="margin-top:15px;border-top:1px solid #ccc;"></div>
  </div>
</div>

<!-- REKAP -->
<div id="pageRekap" class="page">
  <div class="card">
    <div class="header"><h2>NOTA PESAN</h2></div>
    <button onclick="showPage('pageKasir')">Kembali ke Kasir</button>
    <div id="rekapDiv"></div>
  </div>
</div>

<script>
// DATA
let menuData = JSON.parse(localStorage.getItem("menuRM")) || [
  {nama:"Nasi Goreng",harga:15000,kategori:"Makanan"},
  {nama:"Teh Manis",harga:5000,kategori:"Minuman"},
  {nama:"Krupuk",harga:2000,kategori:"Jajanan"}
];
let identitas = JSON.parse(localStorage.getItem("identitasRM")) || { toko:"RM JOYO", kasir:"KOES" };
let pesananData = [], darkMode = JSON.parse(localStorage.getItem("darkMode")) || false;

// APPLY IDENTITAS
function applyIdentitas(){
  document.title = identitas.toko;
  document.getElementById("displayNamaToko").innerText = identitas.toko;
  document.getElementById("inputNamaToko").value = identitas.toko;
  document.getElementById("inputNamaKasir").value = identitas.kasir;
}
applyIdentitas();
if(darkMode) document.body.classList.add("dark");
document.getElementById("darkModeBtn").innerText = darkMode?"Dark Mode: ON":"Dark Mode: OFF";

// TOGGLE DARK MODE
function toggleDark(){
  darkMode=!darkMode;
  localStorage.setItem("darkMode",darkMode);
  document.body.classList.toggle("dark");
  document.getElementById("darkModeBtn").innerText = darkMode?"Dark Mode: ON":"Dark Mode: OFF";
}

// WAKTU & RUN TEXT
function updateWaktu(){
  const d=new Date();
  document.getElementById("waktu").innerText=d.toLocaleString('id-ID');
  const jam=d.getHours();
  let salam=jam<11?"PAGI":jam<15?"SIANG":jam<18?"SORE":"MALAM";
  document.getElementById("runText").innerText=`${salam}, ${identitas.kasir.toUpperCase()} @ ${identitas.toko.toUpperCase()}... SEMANGAT!   [ 087850876841 ]  DISIMPAN hub kami kalau bermasalah atau ERROR. sebisa akan kami bantu        SEMOGA LANCAR USAHANYA MENJADI LARIS MANIS`;
}
setInterval(updateWaktu,1000);

// NAVIGASI HALAMAN
function showPage(id){
  document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  if(id==='pageRiwayat') loadRiwayat();
  if(id==='pageRekap') loadRekap();
}

// LOGIN
function login(){
  const u=document.getElementById("username").value;
  const p=document.getElementById("password").value;
  if(u==="55555"){ showPage("pageKasir"); renderMenu(); return; }
  if((u==="kasir" && p==="000")||(u==="bos" && p==="777")){ showPage("pageKasir"); renderMenu(); }
  else alert("Login Gagal!");
}

// UPDATE IDENTITAS
function updateIdentitas(){
  identitas.toko=document.getElementById("inputNamaToko").value||"RM JOYO";
  identitas.kasir=document.getElementById("inputNamaKasir").value||"KOES";
  localStorage.setItem("identitasRM",JSON.stringify(identitas));
  applyIdentitas();
  alert("Identitas berhasil diperbarui!");
}

// RENDER MENU
function renderMenu(){
  const container=document.getElementById("menuContainer");
  container.innerHTML="";
  ["Makanan","Minuman","Jajanan"].forEach(kat=>{
    const list=menuData.filter(m=>m.kategori===kat);
    if(list.length){container.innerHTML+=`<div class="kategori">${kat}</div>`;
      list.forEach((m)=>{
        const item=document.createElement("div");
        item.className="menu-item";
        item.innerHTML=`<div class="menu-info" onclick="tambahItem('${m.nama}',${m.harga})"><span>${m.nama}</span><span>Rp ${m.harga.toLocaleString()}</span></div><div class="del" onclick="hapusMenuMaster(event,'${m.nama}')">✖</div>`;
        container.appendChild(item);
      });
    }
  });
}

// MENU TAMBAH / HAPUS
function tambahMenuBaru(){
  const n=document.getElementById("namaBaru").value;
  const h=parseInt(document.getElementById("hargaBaru").value);
  if(!n||!h) return alert("Nama & Harga harus diisi!");
  menuData.push({nama:n,harga:h,kategori:document.getElementById("kategoriBaru").value});
  localStorage.setItem("menuRM",JSON.stringify(menuData));
  renderMenu();
  document.getElementById("namaBaru").value="";
  document.getElementById("hargaBaru").value="";
}
function hapusMenuMaster(e,nama){e.stopPropagation();if(confirm(`Hapus menu "${nama}"?`)){menuData=menuData.filter(m=>m.nama!==nama);localStorage.setItem("menuRM",JSON.stringify(menuData));renderMenu();}}

// PESANAN
function tambahItem(n,h){let ada=pesananData.find(x=>x.nama===n);if(ada)ada.jumlah++;else pesananData.push({nama:n,harga:h,jumlah:1});renderPesanan();}
function renderPesanan(){
  const list=document.getElementById("list");
  list.innerHTML="";
  let total=0;
  pesananData.forEach((p,i)=>{
    total+=p.harga*p.jumlah;
    list.innerHTML+=`<div class="order"><div style="flex:2;"><b>${p.nama}</b> (${p.jumlah} x Rp ${p.harga.toLocaleString()})</div><div style="flex:1;text-align:right;">Rp ${(p.harga*p.jumlah).toLocaleString()}</div><div class="del" onclick="hapusSatu(${i})">✖</div></div>`;
  });
  document.getElementById("total").innerText=total.toLocaleString();
}
function hapusSatu(i){pesananData.splice(i,1);renderPesanan();}
function bayar(){
  if(pesananData.length===0) return alert("Pilih menu dulu!");
  const riwayat=JSON.parse(localStorage.getItem("riwayatRM")||"[]");
  const total=pesananData.reduce((a,b)=>a+b.harga*b.jumlah,0);
  riwayat.push({waktu:new Date().toISOString(),pesanan:[...pesananData],total,kasir:identitas.kasir});
  localStorage.setItem("riwayatRM",JSON.stringify(riwayat));
  pesananData=[];
  renderPesanan();
  alert("Nota berhasil disimpan!");
}

// GRADIENT NAMA TOKO BERUBAH TIAP 10 MENIT
const gradients = [
  "linear-gradient(90deg, #ff0000, #ffeb3b)", // merah → kuning
  "linear-gradient(90deg, #ffeb3b, #ffa500)", // kuning → oranye
  "linear-gradient(90deg, #ff4500, #ffd700)"  // oranye → kuning terang
];
let gradIndex=0;
function ubahGradient(){gradIndex=(gradIndex+1)%gradients.length;document.getElementById("displayNamaToko").style.background=gradients[gradIndex];}
setInterval(ubahGradient,10*60*1000); // tiap 10 menit

// RIWAYAT & REKAP
function loadRiwayat(){
  const riwayat=JSON.parse(localStorage.getItem("riwayatRM")||"[]");
  const div=document.getElementById("riwayat");div.innerHTML="<h4>Laporan Terjual Per Item per Tanggal:</h4>";
  let laporan={};
  riwayat.forEach(nota=>{
    const tgl=new Date(nota.waktu).toLocaleDateString('id-ID');
    if(!laporan[tgl]) laporan[tgl]={};
    nota.pesanan.forEach(p=>{laporan[tgl][p.nama]=(laporan[tgl][p.nama]||0)+p.jumlah;});
  });
  for(let tgl in laporan){
    div.innerHTML+=`<h5>${tgl}</h5>`;
    for(let item in laporan[tgl]){
      div.innerHTML+=`<div>${item}: <b>${laporan[tgl][item]} terjual</b></div>`;
    }
    div.innerHTML+="<hr>";
  }
}
function resetRiwayat(){if(confirm("Hapus semua riwayat?")){localStorage.removeItem("riwayatRM"); loadRiwayat(); loadRekap();}}
function loadRekap(){
  const riwayat=JSON.parse(localStorage.getItem("riwayatRM")||"[]");
  const div=document.getElementById("rekapDiv");div.innerHTML="";
  let gt=0;
  riwayat.slice().reverse().forEach((n,i)=>{
    gt+=n.total;
    let items=n.pesanan.map(p=>`<li>${p.nama} x${p.jumlah}</li>`).join("");
    div.innerHTML+=`<div style="background:#fff;color:#000;padding:10px;margin-bottom:10px;border-radius:8px;border-left:4px solid #42a5f5;"><b>Nota #${riwayat.length-i}</b> - ${new Date(n.waktu).toLocaleTimeString()}<br><small>Kasir: ${n.kasir}</small><ul style="margin:5px 0;">${items}</ul><b>Total: Rp ${n.total.toLocaleString()}</b></div>`;
  });
  div.innerHTML=`<h3 style="text-align:center;">TOTAL OMZET: Rp ${gt.toLocaleString()}</h3>`+div.innerHTML;
}
</script>
</body>
</html>
