<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Thrift RE:Wear — E-POS </title>

<!-- CSS: Bootstrap, DataTables, FontAwesome -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<link href="https://cdn.datatables.net/1.13.6/css/dataTables.bootstrap5.min.css" rel="stylesheet">
<link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">

<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<!-- SheetJS for Excel export -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js">
</script>

<style>
  :root{ --accent:#2b8cff; --accent-2:#ffb300; --muted:#6c757d; --bg:#f4f7fb; }
  *{box-sizing:border-box;font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial}
  body{background:var(--bg);margin:0}
  /* Login */
  #loginPage{height:100vh;display:flex;align-items:center;justify-content:center;background:linear-gradient(135deg,#ffd89b,#ff8a00)}
  .login-card{width:380px;background:#fff;padding:26px;border-radius:14px;box-shadow:0 10px 30px rgba(0,0,0,0.12);text-align:center}
  .logo{font-weight:800;color:var(--accent);font-size:22px}
  .logo span{color:var(--accent-2)}
  /* App layout */
  #app{display:none}
  .layout{display:flex;min-height:100vh}
  aside.sidebar{width:240px;background:#fff;padding:18px;border-right:1px solid #eceff5}
  .brand{font-weight:800;color:var(--accent);margin-bottom:8px}
  .sidebar .menu{padding:0;margin:0;list-style:none}
  .sidebar .menu li{padding:10px 12px;border-radius:8px;margin-bottom:6px;cursor:pointer;color:#333;display:flex;align-items:center;gap:10px}
  .sidebar .menu li.active,.sidebar .menu li:hover{background:linear-gradient(90deg,#fff7e6,#fff);box-shadow:inset 0 0 0 1px rgba(0,0,0,0.02)}
  main.container-fluid{padding:22px}
  header.app-top{display:flex;justify-content:space-between;align-items:center;margin-bottom:14px}
  .user-pill{background:#f1f5ff;padding:6px 12px;border-radius:999px;color:#1f2937}
  .logout-btn{background:#ef4444;border:0;color:#fff;padding:7px 12px;border-radius:8px}
  /* Cards */
  .cards{display:grid;grid-template-columns:repeat(4,1fr);gap:12px;margin-bottom:14px}
  .card{background:#fff;border-radius:10px;padding:14px;box-shadow:0 6px 18px rgba(12,13,14,0.04);border-top:6px solid var(--accent-2)}
  .card h5{margin:0;color:#6b7280;font-size:13px}
  .card p{margin:10px 0 0;font-weight:800;font-size:18px}
  /* Panels and charts */
  .panel{background:#fff;padding:12px;border-radius:10px;box-shadow:0 6px 18px rgba(12,13,14,0.04)}
  .panel .panel-title{background:var(--accent-2);Color:#fff;padding:8px;border-radius:8px;font-weight:700;margin-bottom:8px;display:inline-block}
  canvas{max-height:350px}
  /* Tables */
  table.dataTable thead th{background:var(--accent-2);color:#fff}
  /* Forms small */
  .small{font-size:13px;color:var(--muted)}
  /* Transaction layout */
  .search-box{display:flex;gap:8px;margin-bottom:10px}
  .result-list{max-height:300px;overflow:auto;border:1px solid #eef1f6;padding:8px;border-radius:8px;background:#fff}
  .result-item{display:flex;justify-content:space-between;gap:8px;padding:8px;border-bottom:1px solid #f2f4f7}
  .result-item:last-child{border-bottom:none}
  .cart-table{max-height:260px;overflow:auto;background:#fff;border:1px solid #eef1f6;padding:8px;border-radius:8px}
  /* Nota print */
  #notaPrint{display:none;font-family:monospace}
  @media print{
    body *{visibility:hidden}
    #notaPrint, #notaPrint *{visibility:visible}
    #notaPrint{position:fixed;left:0;top:0;width:80mm;padding:6px}
  }
  /* responsive */
  @media (max-width:1000px){
    .cards{grid-template-columns:repeat(2,1fr)}
    aside.sidebar{display:none}
  }
  /* extra small eye button */
  .pw-eye { position:absolute; right:10px; top:9px; cursor:pointer; user-select:none; }
</style>
</head>
<body>

 <!-- LOGIN PAGE (full-screen) -->
  <div id="loginPage">
    <div class="login-card">
      <div class="logo">Re:Wear <span>Thrift</span></div>
      <p class="small">Electronic Point of Sales</p>

      <form id="loginForm" style="margin-top:12px;">
        <!-- Note: no default values so no auto-fill -->
        <input id="loginUser" class="form-control mb-2" placeholder="User ID" autocomplete="off" required>
        <div class="position-relative mb-2">
          <input id="loginPass" type="password" class="form-control" placeholder="Password" autocomplete="off" required>
          <span id="loginToggle" class="pw-eye">👁️</span>
        </div>
        <button class="btn btn-primary w-100 mt-2" type="submit"><i class="fa fa-sign-in-alt me-2"></i>SIGN IN</button>
      </form>
    </div>
  </div>

<!-- APP -->
<div id="app">
  <div class="layout">

    <!-- SIDEBAR -->
    <aside class="sidebar">
      <div class="brand"><i class="fa fa-store"></i> E-POS SYSTEM</div>
      <ul class="menu">
        <li data-page="dashboard" class="active"><i class="fa fa-home"></i> Dashboard</li>
        <li data-page="produk"><i class="fa fa-boxes"></i> Data Barang</li>
        <li data-page="kategori"><i class="fa fa-tags"></i> Kategori</li>
        <li data-page="transaksi"><i class="fa fa-cash-register"></i> Transaksi Jual</li>
        <li data-page="laporan"><i class="fa fa-chart-line"></i> Laporan</li>
        <li data-page="pengaturan"><i class="fa fa-cog"></i> Pengaturan</li>
      </ul>
      <div style="margin-top:12px;font-size:13px;color:#6b7280">Data penyimpanan: <strong>localStorage</strong></div>
    </aside>

    <!-- MAIN -->
    <main class="container-fluid">
      <header class="app-top">
        <div><h3 style="margin:0">Re:Wear Thrift</h3><div class="small">Sungai Sahang, Palembang</div></div>
        <div style="display:flex;gap:10px;align-items:center">
          <div class="user-pill" id="kasirDisplay"><i class="fa fa-user"></i> Kasir</div>
          <button id="btnLogout" class="logout-btn"><i class="fa fa-sign-out-alt"></i> Logout</button>
        </div>
      </header>

      <!-- PAGES -->
      <!-- Dashboard -->
      <section id="page-dashboard" class="page">
        <div class="cards">
          <div class="card"><h5>Jumlah Barang</h5><p id="cardTotalProduk">0</p></div>
          <div class="card"><h5>Total Stok</h5><p id="cardTotalStok">0</p></div>
          <div class="card"><h5>Total Terjual</h5><p id="cardTotalTerjual">0</p></div>
          <div class="card"><h5>Jumlah Kategori</h5><p id="cardTotalKategori">0</p></div>
        </div>

        <div class="row g-3">
          <div class="col-12 col-lg-6">
            <div class="panel">
              <div class="panel-title">📊 Grafik Penjualan (Terjual)</div>
              <canvas id="chartSales"></canvas>
            </div>
          </div>
          <div class="col-12 col-lg-6">
            <div class="panel">
              <div class="panel-title">📊 Stok per Produk</div>
              <canvas id="chartStock"></canvas>
            </div>
          </div>
          <div class="col-12">
            <div class="panel mt-2">
              <div class="panel-title">📊 Stok Per Bulan </div>
              <canvas id="chartMonthly"></canvas>
            </div>
          </div>
        </div>
      </section>

      <!-- Produk -->
      <section id="page-produk" class="page" style="display:none">
        <div class="d-flex justify-content-between align-items-center mb-3">
          <div><h4>Data Produk</h4><div class="small">Kelola produk toko</div></div>
          <div>
            <button id="btnAddProduk" class="btn btn-primary"><i class="fa fa-plus"></i> Tambah Produk</button>
            <button id="btnRefreshProduk" class="btn btn-outline-secondary"><i class="fa fa-sync"></i> Refresh</button>
          </div>
        </div>

        <div class="panel">
          <table id="dtProduk" class="table table-striped" style="width:100%">
            <thead>
              <tr>
                <th>No</th><th>ID</th><th>Kategori</th><th>Nama</th><th>Merk</th><th>Stok</th><th>Harga Beli</th><th>Harga Jual</th><th>Satuan</th><th>Aksi</th>
              </tr>
            </thead>
            <tbody></tbody>
          </table>
        </div>
      </section>

      <!-- Kategori -->
      <section id="page-kategori" class="page" style="display:none">
        <div class="mb-3 d-flex justify-content-between align-items-center">
          <div><h4>Kategori</h4><div class="small">Tambah / edit kategori</div></div>
        </div>

        <div class="d-flex gap-2 mb-3">
          <input id="inputKategoriBaru" class="form-control" placeholder="Masukkan kategori baru">
          <button id="btnInsertKategori" class="btn btn-primary"><i class="fa fa-plus"></i> Insert</button>
        </div>

        <div class="panel">
          <table id="dtKategori" class="table table-striped" style="width:100%">
            <thead><tr><th>No</th><th>Kategori</th><th>Tanggal Input</th><th>Aksi</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
      </section>

      <!-- Transaksi -->
      <section id="page-transaksi" class="page" style="display:none">
        <div class="mb-2"><h4>Transaksi Jual</h4><div class="small">Tambahkan barang ke keranjang lalu bayar</div></div>
        <div class="row g-3">
          <div class="col-lg-7">
            <div class="panel">
              <div class="small mb-2">Cari / Pilih Produk</div>
              <div class="search-box">
                <input id="searchProduk" class="form-control" placeholder="Cari berdasarkan ID atau nama...">
                <button id="btnSearchProduk" class="btn btn-primary"><i class="fa fa-search"></i></button>
              </div>
              <div id="resultList" class="result-list"></div>
            </div>
          </div>

          <div class="col-lg-5">
            <div class="panel">
              <div class="d-flex justify-content-between align-items-center">
                <div><strong>Keranjang</strong><div class="small">Periksa, ubah jumlah, lalu bayar</div></div>
                <div><label class="small">Kasir</label><input id="inputKasir" class="form-control" value=""></div>
              </div>
              <div class="mt-2 cart-table">
                <table id="tblCart" class="table table-sm">
                  <thead><tr><th>No</th><th>Nama</th><th>Qty</th><th>Harga</th><th>Subtotal</th><th>Aksi</th></tr></thead>
                  <tbody></tbody>
                </table>
              </div>

              <div class="mt-2 d-flex gap-2 align-items-center">
                <div style="flex:1"><div class="small">Total</div><h5 id="cartTotal">Rp 0</h5></div>
                <div style="width:180px">
                  <div class="small">Bayar</div>
                  <input id="inputBayar" type="number" class="form-control" placeholder="Masukkan nominal">
                </div>
              </div>

              <div class="mt-3 d-flex gap-2">
                <button id="btnPay" class="btn btn-success w-100"><i class="fa fa-check"></i> Bayar</button>
                <button id="btnPrint" class="btn btn-outline-secondary w-100"><i class="fa fa-print"></i> Print Nota</button>
                <button id="btnResetCart" class="btn btn-light w-100"><i class="fa fa-trash"></i> Reset</button>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- Laporan -->
      <section id="page-laporan" class="page" style="display:none">
        <div class="mb-3 d-flex justify-content-between align-items-center">
          <div>
            <h4>Laporan Penjualan</h4>
            <div class="small">Riwayat transaksi — bisa diexport ke Excel dan dihapus</div>
          </div>
          <div class="d-flex gap-2">
            <button id="btnExportExcel" class="btn btn-success"><i class="fa fa-file-excel"></i> Export Excel</button>
            <button id="btnDeleteAllSales" class="btn btn-danger"><i class="fa fa-trash"></i> Hapus Semua</button>
          </div>
        </div>

        <div class="panel">
          <table id="dtSales" class="table table-striped" style="width:100%">
            <thead><tr><th>No</th><th>ID Transaksi</th><th>Tanggal</th><th>Kasir</th><th>Items</th><th>Total</th><th>Aksi</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
      </section>

      <!-- Pengaturan (placeholder) -->
      <section id="page-pengaturan" class="page" style="display:none">
        <div class="panel">
          <h4>Pengaturan</h4>
          <p class="small">Pengaturan sederhana (placeholder).</p>
          <div class="mb-2"><label class="form-label">Nama Toko</label><input id="setStoreName" class="form-control" value="Thrift Re Ware"></div>
          <button id="btnSaveSettings" class="btn btn-primary">Simpan</button>
        </div>
      </section>

    </main>
  </div>
</div>

<!-- Nota (print area) -->
<div id="notaPrint"></div>

<!-- MODAL: Produk -->
<div class="modal fade" id="modalProduk" tabindex="-1">
  <div class="modal-dialog modal-lg">
    <div class="modal-content">
      <form id="formProduk">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa fa-box"></i> Tambah / Edit Produk</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <input type="hidden" id="produkIndex">
          <div class="row g-2">
            <div class="col-md-6"><label class="form-label">Nama Produk</label><input id="inputNamaProduk" class="form-control" required></div>
            <div class="col-md-6"><label class="form-label">Kategori</label><select id="selectKategoriProduk" class="form-select"></select></div>
            <div class="col-md-4"><label class="form-label">Merk</label><input id="inputMerkProduk" class="form-control"></div>
            <div class="col-md-4"><label class="form-label">Satuan</label><input id="inputSatuanProduk" class="form-control" placeholder="PCS/KG"></div>
            <div class="col-md-4"><label class="form-label">Stok</label><input id="inputStokProduk" type="number" class="form-control" min="0" value="0"></div>
            <div class="col-md-6"><label class="form-label">Harga Beli</label><input id="inputHargaBeli" type="number" class="form-control" min="0" value="0"></div>
            <div class="col-md-6"><label class="form-label">Harga Jual</label><input id="inputHargaJual" type="number" class="form-control" min="0" value="0"></div>
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
          <button type="submit" class="btn btn-primary" id="btnSaveProduk">Simpan</button>
        </div>
      </form>
    </div>
  </div>
</div>

<!-- CONFIRM MODAL -->
<div class="modal fade" id="modalConfirm" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header bg-danger text-white"><h5 class="modal-title">Konfirmasi</h5></div>
      <div class="modal-body" id="confirmBody">Yakin?</div>
      <div class="modal-footer">
        <button class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
        <button id="confirmYes" class="btn btn-danger">Ya</button>
      </div>
    </div>
  </div>
</div>

<!-- SCRIPTS -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdn.datatables.net/1.13.6/js/jquery.dataTables.min.js"></script>
<script src="https://cdn.datatables.net/1.13.6/js/dataTables.bootstrap5.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>

<script>
/* ============================
   Keys & initial sample data
   ============================ */
const KEY_PRODUK = 'tm_produk_v1';
const KEY_KATEGORI = 'tm_kategori_v1';
const KEY_SALES = 'tm_sales_v1';

let produkData = JSON.parse(localStorage.getItem(KEY_PRODUK)) || [];
let kategoriData = JSON.parse(localStorage.getItem(KEY_KATEGORI)) || [];
let salesData = JSON.parse(localStorage.getItem(KEY_SALES)) || [];


/* ============================
   Helpers
   ============================ */
const fmt = n => 'Rp ' + (Number(n||0)).toLocaleString('id-ID');
const genId = (prefix, arr) => prefix + String(arr.length + 1).padStart(3,'0');
const nowISO = () => new Date().toISOString();
const nowDisplay = () => new Date().toLocaleString('id-ID', {day:'numeric',month:'long',year:'numeric',hour:'2-digit',minute:'2-digit'});

/* ============================
   DOM refs & state
   ============================ */
let dtProduk = null, dtKategori = null, dtSales = null;
let chartSales=null, chartStock=null, chartMonthly=null;
let cart = []; // {id,name,price,qty,subtotal}

/* ============================
   Init on ready
   ============================ */
$(function(){

  // initialize datatables
  dtProduk = $('#dtProduk').DataTable({ columnDefs:[{orderable:false, targets:9}], autoWidth:false });
  dtKategori = $('#dtKategori').DataTable({ columnDefs:[{orderable:false, targets:3}], autoWidth:false });
  dtSales = $('#dtSales').DataTable({ autoWidth:false, columnDefs:[{orderable:false, targets:6}] });

  // page navigation
  $('.sidebar .menu li').on('click', function(){
    $('.sidebar .menu li').removeClass('active');
    $(this).addClass('active');
    const page = $(this).data('page');
    $('.page').hide();
    $('#page-' + page).show();
    if (page === 'produk') renderProdukTable();
    if (page === 'kategori') renderKategoriTable();
    if (page === 'transaksi') {
      renderSearchResults(produkData.slice(0,50)); // show sample
      renderCart();
    }
    if (page === 'laporan') renderSalesTable();
    if (page === 'dashboard') updateCharts();
  });

  // LOGIN (no autofill, toggle eye)
  $('#loginForm').on('submit', function(e){
    e.preventDefault();
    const user = $('#loginUser').val().trim();
    const pass = $('#loginPass').val().trim();
    if (user === 'Rewear' && pass === '170907'){
      $('#loginPage').hide();
      $('#app').show();
      $('#kasirDisplay').text('👩 ' + (user || 'Kasir'));
      refreshAll();
    } else {
      alert('User ID atau Password salah!');
    }
  });
  $('#loginToggle').on('click', ()=> {
    const p = $('#loginPass');
    p.attr('type', p.attr('type') === 'password' ? 'text' : 'password');
  });

  // LOGOUT: kembali ke tampilan login, kosongkan input login
  $('#btnLogout').on('click', ()=> {
    if (!confirm('Logout?')) return;
    // hide app, show login; clear login inputs and other transient UI
    $('#app').hide();
    $('#loginPage').show();
    $('#loginForm')[0].reset();
    $('#loginPass').attr('type','password'); // ensure masked
    // optional: clear cart & UI (but keep data in localStorage)
    cart = [];
    renderCart();
    // return to dashboard tab highlight for next login
    $('.sidebar .menu li').removeClass('active');
    $('.sidebar .menu li[data-page="dashboard"]').addClass('active');
    $('.page').hide(); $('#page-dashboard').show();
  });

  // product modal open
  $('#btnAddProduk').on('click', ()=> openProdukModal('add'));
  $('#btnRefreshProduk').on('click', ()=> { renderProdukTable(); alert('Data diperbarui'); });

  // category insert
  $('#btnInsertKategori').on('click', ()=> {
    const v = $('#inputKategoriBaru').val().trim();
    if (!v) return alert('Masukkan nama kategori');
    const id = genId('CAT', kategoriData);
    kategoriData.push({ id, nama:v, createdAt: nowISO() });
    localStorage.setItem(KEY_KATEGORI, JSON.stringify(kategoriData));
    $('#inputKategoriBaru').val('');
    renderKategoriTable();
    populateKategoriSelect();
    updateDashboardCards();
  });

  // product save
  $('#formProduk').on('submit', function(e){
    e.preventDefault();
    const idx = $('#produkIndex').val();
    const nama = $('#inputNamaProduk').val().trim();
    const kategoriId = $('#selectKategoriProduk').val() ;
    const merk = $('#inputMerkProduk').val().trim();
    const satuan = $('#inputSatuanProduk').val().trim();
    const stok = parseInt($('#inputStokProduk').val()) || 0;
    const hargaBeli = parseInt($('#inputHargaBeli').val()) || 0;
    const hargaJual = parseInt($('#inputHargaJual').val()) || 0;
    if (!nama) return alert('Nama produk wajib diisi');
    if (idx === '') {
      const id = genId('BR', produkData);
      produkData.push({ id, nama, kategoriId, merk, satuan, stok, hargaBeli, hargaJual, terjual:0, createdAt: nowISO() });
    } else {
      const i = parseInt(idx,10);
      produkData[i] = { ...produkData[i], nama, kategoriId, merk, satuan, stok, hargaBeli, hargaJual };
    }
    localStorage.setItem(KEY_PRODUK, JSON.stringify(produkData));
    const modal = bootstrap.Modal.getInstance(document.getElementById('modalProduk'));
    modal.hide();
    renderProdukTable();
    populateKategoriSelect();
    updateDashboardCards();
    updateCharts();
  });

  // search products in transaksi
  $('#btnSearchProduk').on('click', doSearch);
  $('#searchProduk').on('keyup', (e)=> { if (e.key==='Enter') doSearch(); });

  // cart actions (delegation)
  $('#tblCart tbody').on('click','button[data-action="remove"]', function(){
    const idx = $(this).data('idx');
    cart.splice(idx,1);
    renderCart();
  });
  $('#tblCart tbody').on('change','input.qty-input', function(){
    const idx = $(this).data('idx');
    const val = parseInt($(this).val()) || 1;
    if (val <= 0) { $(this).val(1); return; }
    const prod = produkData.find(p => p.id === cart[idx].id);
    if (val > (prod?.stok || 0)) { alert('Qty melebihi stok'); $(this).val(cart[idx].qty); return; }
    cart[idx].qty = val;
    cart[idx].subtotal = cart[idx].qty * cart[idx].price;
    renderCart();
  });

  // pay & print
  $('#btnPay').on('click', payNow);
  $('#btnPrint').on('click', () => {
    if (!salesData.length) return alert('Belum ada transaksi untuk dicetak');
    printNota(salesData[salesData.length-1]);
  });
  $('#btnResetCart').on('click', ()=> { if(confirm('Reset keranjang?')){ cart=[]; renderCart(); } });

  // Export Excel (sales report)
  $('#btnExportExcel').on('click', ()=> {
    if (!salesData.length) return alert('Tidak ada data laporan untuk diexport');
    exportSalesToExcel();
  });

  // Delete all sales
  $('#btnDeleteAllSales').on('click', ()=> {
    if (!salesData.length) return alert('Tidak ada laporan untuk dihapus');
    if (!confirm('Hapus semua laporan penjualan? Tindakan ini tidak dapat dibatalkan.')) return;
    salesData = [];
    localStorage.setItem(KEY_SALES, JSON.stringify(salesData));
    renderSalesTable();
    alert('Semua laporan dihapus.');
  });

  // settings save (placeholder)
  $('#btnSaveSettings').on('click', ()=> { alert('Pengaturan disimpan (placeholder).'); });

  // initial renders
  populateKategoriSelect();
  renderKategoriTable();
  renderProdukTable();
  initCharts();
  updateDashboardCards();
});

/* ============================
   Render / UI functions
   ============================ */

function refreshAll(){
  populateKategoriSelect();
  renderKategoriTable();
  renderProdukTable();
  renderSalesTable();
  updateDashboardCards();
  updateCharts();
}

function populateKategoriSelect(){
  const sel = $('#selectKategoriProduk');
  sel.empty();
  sel.append(`<option value="UNCAT">Uncategorized</option>`);
  kategoriData.forEach(c => sel.append(`<option value="${c.id}">${c.nama}</option>`));
}

/* Produk table */
function renderProdukTable(){
  dtProduk.clear();
  const tbody = $('#dtProduk tbody');
  tbody.empty();
  produkData.forEach((p,idx) => {
    const cat = kategoriData.find(c=>c.id===p.kategoriId);
    const catName = cat ? cat.nama : (p.kategoriId === 'UNCAT' ? 'Uncategorized' : p.kategoriId);
    const row = `<tr data-idx="${idx}">
      <td>${idx+1}</td>
      <td>${p.id}</td>
      <td>${catName}</td>
      <td>${p.nama}</td>
      <td>${p.merk || '-'}</td>
      <td style="text-align:right">${p.stok}</td>
      <td style="text-align:right">${fmt(p.hargaBeli)}</td>
      <td style="text-align:right">${fmt(p.hargaJual)}</td>
      <td>${p.satuan || '-'}</td>
      <td>
        <button class="btn btn-sm btn-primary me-1" onclick="viewProduk(${idx})"><i class="fa fa-eye"></i></button>
        <button class="btn btn-sm btn-warning me-1" onclick="openProdukModal('edit',${idx})"><i class="fa fa-pen"></i></button>
        <button class="btn btn-sm btn-danger" onclick="hapusProduk(${idx})"><i class="fa fa-trash"></i></button>
      </td>
    </tr>`;
    dtProduk.row.add($(row));
  });
  dtProduk.draw();
}

/* Kategori table */
function renderKategoriTable(){
  dtKategori.clear();
  $('#dtKategori tbody').empty();
  kategoriData.forEach((c, idx) => {
    const row = `<tr data-idx="${idx}">
      <td>${idx+1}</td>
      <td>${c.nama}</td>
      <td>${new Date(c.createdAt).toLocaleString('id-ID')}</td>
      <td>
        <button class="btn btn-sm btn-warning me-1" onclick="editKategori(${idx})"><i class="fa fa-pen"></i></button>
        <button class="btn btn-sm btn-danger" onclick="hapusKategori(${idx})"><i class="fa fa-trash"></i></button>
      </td>
    </tr>`;
    dtKategori.row.add($(row));
  });
  dtKategori.draw();
}

/* Sales table (laporan) */
function renderSalesTable(){
  dtSales.clear();
  $('#dtSales tbody').empty();
  salesData.forEach((s, idx) => {
    const items = s.items.map(it=>`${it.name} x${it.qty}`).join(', ');
    const row = `<tr data-idx="${idx}">
      <td>${idx+1}</td>
      <td>${s.id}</td>
      <td>${new Date(s.createdAt).toLocaleString('id-ID')}</td>
      <td>${s.kasir}</td>
      <td>${items}</td>
      <td style="text-align:right">${fmt(s.total)}</td>
      <td style="white-space:nowrap">
        <button class="btn btn-sm btn-success me-1" onclick="printNotaFromTable(${idx})"><i class="fa fa-print"></i></button>
        <button class="btn btn-sm btn-danger" onclick="hapusSales(${idx})"><i class="fa fa-trash"></i></button>
      </td>
    </tr>`;
    dtSales.row.add($(row));
  });
  dtSales.draw();
  updateDashboardCards();
}

/* Dashboard cards */
function updateDashboardCards(){
  $('#cardTotalProduk').text(produkData.length);
  $('#cardTotalStok').text(produkData.reduce((a,b)=>a + (b.stok||0),0));
  $('#cardTotalTerjual').text(produkData.reduce((a,b)=>a + (b.terjual||0),0));
  $('#cardTotalKategori').text(kategoriData.length);
}

/* ============================
   Produk modal CRUD helpers
   ============================ */
function openProdukModal(mode, idx){
  populateKategoriSelect();
  if (mode === 'add'){
    $('#produkIndex').val('');
    $('#inputNamaProduk').val('');
    $('#selectKategoriProduk').val(kategoriData[0] ? kategoriData[0].id : 'UNCAT');
    $('#inputMerkProduk').val('');
    $('#inputSatuanProduk').val('');
    $('#inputStokProduk').val(0);
    $('#inputHargaBeli').val(0);
    $('#inputHargaJual').val(0);
    $('#modalProduk .modal-title').text('Tambah Produk');
  } else {
    const p = produkData[idx];
    $('#produkIndex').val(idx);
    $('#inputNamaProduk').val(p.nama);
    $('#selectKategoriProduk').val(p.kategoriId || 'UNCAT');
    $('#inputMerkProduk').val(p.merk || '');
    $('#inputSatuanProduk').val(p.satuan || '');
    $('#inputStokProduk').val(p.stok || 0);
    $('#inputHargaBeli').val(p.hargaBeli || 0);
    $('#inputHargaJual').val(p.hargaJual || 0);
    $('#modalProduk .modal-title').text('Edit Produk');
  }
  const modal = new bootstrap.Modal(document.getElementById('modalProduk'));
  modal.show();
}

function viewProduk(idx){
  const p = produkData[idx];
  const cat = kategoriData.find(c=>c.id===p.kategoriId);
  alert(`Nama: ${p.nama}\nID: ${p.id}\nKategori: ${cat ? cat.nama : 'Uncategorized'}\nMerk: ${p.merk}\nStok: ${p.stok}\nHarga Jual: ${fmt(p.hargaJual)}\nSatuan: ${p.satuan}`);
}

function hapusProduk(idx){
  $('#confirmBody').text(`Hapus produk "${produkData[idx].nama}"?`);
  const modal = new bootstrap.Modal(document.getElementById('modalConfirm'));
  $('#confirmYes').off('click').on('click', ()=>{
    produkData.splice(idx,1);
    localStorage.setItem(KEY_PRODUK, JSON.stringify(produkData));
    modal.hide();
    renderProdukTable();
    updateCharts(); updateDashboardCards();
  });
  modal.show();
}

/* Kategori edit & delete */
function editKategori(i){
  const newName = prompt('Edit nama kategori:', kategoriData[i].nama);
  if (newName === null) return;
  kategoriData[i].nama = newName.trim() || kategoriData[i].nama;
  localStorage.setItem(KEY_KATEGORI, JSON.stringify(kategoriData));
  renderKategoriTable();
  populateKategoriSelect();
  updateDashboardCards();
}

function hapusKategori(i){
  $('#confirmBody').text(`Hapus kategori "${kategoriData[i].nama}"? Produk dengan kategori ini akan menjadi "".`);
  const modal = new bootstrap.Modal(document.getElementById('modalConfirm'));
  $('#confirmYes').off('click').on('click', ()=>{
    const cat = kategoriData.splice(i,1)[0];
    produkData = produkData.map(p => p.kategoriId === cat.id ? {...p, kategoriId:'UNCAT'} : p);
    localStorage.setItem(KEY_KATEGORI, JSON.stringify(kategoriData));
    localStorage.setItem(KEY_PRODUK, JSON.stringify(produkData));
    modal.hide();
    renderKategoriTable();
    renderProdukTable();
    populateKategoriSelect();
    updateCharts(); updateDashboardCards();
  });
  modal.show();
}

/* ============================
   Transaksi: search / cart / pay / print
   ============================ */
function doSearch(){
  const q = $('#searchProduk').val().trim().toLowerCase();
  const list = produkData.filter(p => p.id.toLowerCase().includes(q) || p.nama.toLowerCase().includes(q));
  renderSearchResults(list);
}
function renderSearchResults(list){
  const out = $('#resultList').empty();
  if (!list.length) { out.append(`<div class="small p-2 text-muted">Tidak ada hasil</div>`); return; }
  list.forEach(p=>{
    const cat = kategoriData.find(c=>c.id===p.kategoriId);
    const html = `
      <div class="result-item">
        <div>
          <div style="font-weight:700">${p.nama} <small class="text-muted">(${p.id})</small></div>
          <div class="small text-muted">${cat ? cat.nama : 'Uncategorized'} · Stok: ${p.stok}</div>
          <div class="small mt-1">${fmt(p.hargaJual)}</div>
        </div>
        <div style="text-align:right;min-width:140px">
          <input id="qty_${p.id}" type="number" min="1" value="1" style="width:70px;padding:6px;border-radius:6px;border:1px solid #e6eaf0">
          <div style="height:6px"></div>
          <button class="btn btn-sm btn-primary" onclick="addToCart('${p.id}')"><i class="fa fa-cart-plus"></i> Tambah</button>
        </div>
      </div>`;
    out.append(html);
  });
}

function addToCart(id){
  const p = produkData.find(x=>x.id===id);
  if (!p) return alert('Produk tidak ditemukan');
  const qty = parseInt(document.getElementById('qty_'+id).value) || 1;
  if (qty > (p.stok || 0)) return alert('Qty melebihi stok tersedia');
  const existing = cart.find(c=>c.id === id);
  if (existing){
    if (existing.qty + qty > (p.stok || 0)) return alert('Total qty melebihi stok tersedia');
    existing.qty += qty; existing.subtotal = existing.qty * existing.price;
  } else {
    cart.push({ id: p.id, name: p.nama, price: p.hargaJual || 0, qty, subtotal: qty * (p.hargaJual || 0) });
  }
  renderCart();
}

function renderCart(){
  const tbody = $('#tblCart tbody').empty();
  if (!cart.length) {
    tbody.append(`<tr><td colspan="6" class="text-center text-muted">Keranjang kosong</td></tr>`);
    $('#cartTotal').text('Rp 0');
    $('#inputBayar').val('');
    return;
  }
  let total = 0;
  cart.forEach((it, idx) => {
    total += it.subtotal;
    const row = `<tr>
      <td>${idx+1}</td>
      <td>${it.name}</td>
      <td><input class="form-control qty-input" data-idx="${idx}" value="${it.qty}" min="1" style="width:70px"></td>
      <td style="text-align:right">${fmt(it.price)}</td>
      <td style="text-align:right">${fmt(it.subtotal)}</td>
      <td style="white-space:nowrap"><button class="btn btn-sm btn-danger" data-action="remove" data-idx="${idx}" onclick="removeCartItem(${idx})"><i class="fa fa-trash"></i></button></td>
    </tr>`;
    tbody.append(row);
  });
  $('#cartTotal').text(fmt(total));
  $('#inputBayar').val(total);
}

function removeCartItem(i){
  cart.splice(i,1);
  renderCart();
}

/* Pay */
function payNow(){
  if (!cart.length) return alert('Keranjang kosong');
  const bayar = parseFloat($('#inputBayar').val()) || 0;
  const total = cart.reduce((s,i)=>s + (i.subtotal||0),0);
  if (bayar < total) return alert('Pembayaran kurang!');
  const kembali = bayar - total;
  // save transaction
  const tr = {
    id: 'TR' + String(salesData.length + 1).padStart(5,'0'),
    createdAt: nowISO(),
    kasir: $('#inputKasir').val() || 'Kasir',
    items: cart.map(c => ({ id:c.id, name:c.name, qty:c.qty, price:c.price, subtotal:c.subtotal })),
    total, bayar, kembali
  };
  salesData.push(tr);
  localStorage.setItem(KEY_SALES, JSON.stringify(salesData));
  // update produk stok & terjual
  cart.forEach(it => {
    const p = produkData.find(pp => pp.id === it.id);
    if (p) {
      p.stok = (p.stok || 0) - it.qty;
      p.terjual = (p.terjual || 0) + it.qty;
      if (p.stok < 0) p.stok = 0;
    }
  });
  localStorage.setItem(KEY_PRODUK, JSON.stringify(produkData));
  // clear cart
  cart = [];
  renderCart();
  renderProdukTable();
  renderSalesTable();
  updateCharts();
  updateDashboardCards();
  alert('Transaksi berhasil. Kembalian: ' + fmt(kembali));
}

/* Print nota small (thermal style) */
function printNota(transaction){
  const itemsRows = transaction.items.map(it => `<tr><td style="padding:4px">${it.name}</td><td style="padding:4px;text-align:center">${it.qty}</td><td style="padding:4px;text-align:right">${fmt(it.subtotal)}</td></tr>`).join('');
  const html = `
    <div style="font-family:monospace;padding:8px;width:280px">
      <div style="text-align:center;font-weight:700">Thrift Re Ware</div>
      <div style="text-align:center;font-size:12px">Jl. Bukit, Palembang | 0812-7203-9749</div>
      <hr/>
      <div>ID: ${transaction.id}</div>
      <div>Tanggal: ${new Date(transaction.createdAt).toLocaleString('id-ID')}</div>
      <div>Kasir: ${transaction.kasir}</div>
      <hr/>
      <table style="width:100%;border-collapse:collapse">${itemsRows}</table>
      <hr/>
      <div style="display:flex;justify-content:space-between"><div>Total</div><div>${fmt(transaction.total)}</div></div>
      <div style="display:flex;justify-content:space-between"><div>Bayar</div><div>${fmt(transaction.bayar)}</div></div>
      <div style="display:flex;justify-content:space-between"><div>Kembali</div><div>${fmt(transaction.kembali)}</div></div>
      <hr/>
      <div style="text-align:center;font-size:12px">Terima kasih! Selamat berbelanja :)</div>
    </div>
  `;
  const w = window.open('','_blank','width=320,height=480');
  w.document.write(html);
  w.document.close();
  w.focus();
  setTimeout(()=> w.print(), 300);
}

// print from table row
function printNotaFromTable(idx){
  if (!salesData[idx]) return alert('Data transaksi tidak ditemukan');
  printNota(salesData[idx]);
}

/* Delete single sales entry */
function hapusSales(idx){
  $('#confirmBody').text(`Hapus transaksi "${salesData[idx].id}"?`);
  const modal = new bootstrap.Modal(document.getElementById('modalConfirm'));
  $('#confirmYes').off('click').on('click', ()=>{
    salesData.splice(idx,1);
    localStorage.setItem(KEY_SALES, JSON.stringify(salesData));
    modal.hide();
    renderSalesTable();
    alert('Transaksi dihapus.');
  });
  modal.show();
}

/* ============================
   Export Sales to Excel (SheetJS)
   ============================ */
function exportSalesToExcel(){
  // Prepare rows: expand items into string
  const rows = salesData.map(s => ({
    ID: s.id,
    Tanggal: new Date(s.createdAt).toLocaleString('id-ID'),
    Kasir: s.kasir,
    Items: s.items.map(it => `${it.name} x${it.qty}`).join(' | '),
    Total: s.total,
    Bayar: s.bayar,
    Kembali: s.kembali
  }));
  // Generate worksheet & workbook
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Laporan_Penjualan");
  // Auto-width columns (simple)
  const cols = Object.keys(rows[0] || {}).map(k => ({ wch: Math.min(40, Math.max(10, Math.floor(k.length + 10))) }));
  ws['!cols'] = cols;
  // Write file
  const wbout = XLSX.write(wb, { bookType:'xlsx', type:'array' });
  const blob = new Blob([wbout], {type:'application/octet-stream'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  const fname = `Laporan_Penjualan_${new Date().toISOString().slice(0,10)}.xlsx`;
  a.download = fname;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

/* ============================
   Charts
   ============================ */
function initCharts(){
  const ctxS = document.getElementById('chartSales').getContext('2d');
  chartSales = new Chart(ctxS, { type:'bar', data:{ labels: produkData.map(p=>p.nama), datasets:[{ label:'Terjual', data: produkData.map(p=>p.terjual||0), backgroundColor:'rgba(255,193,7,0.8)'}]}, options:{scales:{y:{beginAtZero:true}}} });

  const ctxSt = document.getElementById('chartStock').getContext('2d');
  chartStock = new Chart(ctxSt, { type:'bar', data:{ labels: produkData.map(p=>p.nama), datasets:[{ label:'Stok', data: produkData.map(p=>p.stok||0), backgroundColor:'rgba(54,162,235,0.8)'}]}, options:{scales:{y:{beginAtZero:true}}} });

  const ctxM = document.getElementById('chartMonthly').getContext('2d');
  const bulan = ['Jan','Feb','Mar','Apr','Mei','Jun','Jul','Agt','Sep','Okt','Nov','Des'];
  chartMonthly = new Chart(ctxM, { type:'bar', data:{ labels: bulan, datasets:[{ label:'Stok per Bulan', data: new Array(12).fill(0), backgroundColor:'rgba(75,192,192,0.8)'}]}, options:{scales:{y:{beginAtZero:true}}} });
  updateCharts();
}

function updateCharts(){
  if (!chartSales || !chartStock || !chartMonthly) return;
  chartSales.data.labels = produkData.map(p=>p.nama);
  chartSales.data.datasets[0].data = produkData.map(p=>p.terjual||0);
  chartSales.update();

  chartStock.data.labels = produkData.map(p=>p.nama);
  chartStock.data.datasets[0].data = produkData.map(p=>p.stok||0);
  chartStock.update();

  const arr = new Array(12).fill(0);
  produkData.forEach(p => {
    const m = p.createdAt ? new Date(p.createdAt).getMonth() : new Date().getMonth();
    arr[m] += (p.stok || 0);
  });
  chartMonthly.data.datasets[0].data = arr;
  chartMonthly.update();
}

/* ============================
   Utility small
   ============================ */
function renderSearchResultsOnce(){
  renderSearchResults(produkData.slice(0,50));
}
function renderSearchResultsIfEmpty(){
  if (!$('#resultList').children().length) renderSearchResults(produkData.slice(0,50));
}

/* initialize UI on script load */
(function initUI(){
  // show dashboard initially
  $('.page').hide(); $('#page-dashboard').show();
  // Ensure login inputs are empty at start
  $('#loginUser').val('');
  $('#loginPass').val('');
})();
</script>


</body>
</html>
