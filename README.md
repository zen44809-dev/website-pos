<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Web Kasir - Simple POS</title>
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --muted:#94a3b8; --accent:#06b6d4; --accent-2:#06d6a0;
    --glass: rgba(255,255,255,0.03);
    font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  *{box-sizing:border-box}
  body{
    margin:0; min-height:100vh; background:
    linear-gradient(180deg, rgba(6,30,50,0.6), rgba(2,6,23,0.9)), linear-gradient(90deg,#062b3b33,#0000);
    color:#e6eef6; font-size:15px;
    display:flex; align-items:stretch; padding:18px;
    gap:18px;
  }
  .app{
    width:100%; max-width:1200px; margin:auto; display:grid; grid-template-columns:1fr 420px; gap:18px;
  }
  .panel{background:var(--card); border-radius:12px; padding:14px; box-shadow:0 6px 24px rgba(2,6,23,0.6);}
  header{display:flex; justify-content:space-between; align-items:center; margin-bottom:8px}
  h1{font-size:18px; margin:0}
  .top-actions{display:flex; gap:8px; align-items:center}
  .btn{background:linear-gradient(90deg,var(--accent),var(--accent-2)); border:none; color:#022; padding:8px 12px; border-radius:8px; cursor:pointer; font-weight:600}
  .btn.ghost{background:transparent; border:1px solid rgba(255,255,255,0.06); color:var(--muted)}
  .layout{display:flex; gap:14px; align-items:flex-start}
  /* left: categories + menu grid */
  .left{display:flex; flex-direction:column; gap:12px}
  .categories{display:flex; gap:8px; flex-wrap:wrap}
  .cat{background:var(--glass); padding:8px 10px; border-radius:999px; cursor:pointer; color:var(--muted); border:1px solid rgba(255,255,255,0.02)}
  .cat.active{background:linear-gradient(90deg,var(--accent),var(--accent-2)); color:#022; font-weight:700}
  .search{display:flex; gap:8px}
  .search input{flex:1; padding:8px 10px; border-radius:8px; border:1px solid rgba(255,255,255,0.04); background:transparent; color:inherit}
  .grid{display:grid; grid-template-columns:repeat(auto-fill,minmax(140px,1fr)); gap:10px; max-height:65vh; overflow:auto; padding-right:6px}
  .item{
    background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    padding:10px; border-radius:10px; cursor:pointer; display:flex; flex-direction:column; gap:8px; min-height:90px;
  }
  .item .title{font-weight:700}
  .item .price{color:var(--muted); font-size:13px}
  .item .meta{display:flex; justify-content:space-between; align-items:center}
  .item .add{background:rgba(255,255,255,0.06); padding:6px 8px; border-radius:8px; font-weight:700; cursor:pointer}
  /* right: cart */
  .right{display:flex; flex-direction:column; gap:12px}
  .cart-list{max-height:46vh; overflow:auto; padding:6px; display:flex; flex-direction:column; gap:8px}
  .cart-row{display:flex; gap:8px; align-items:center; background:rgba(255,255,255,0.02); padding:8px; border-radius:8px}
  .cart-row .name{flex:1}
  .qty{display:flex; gap:6px; align-items:center}
  .qty button{background:transparent; border:1px solid rgba(255,255,255,0.04); padding:6px 8px; border-radius:8px; cursor:pointer}
  .summary{padding:10px; background:rgba(255,255,255,0.02); border-radius:8px}
  .summary-row{display:flex; justify-content:space-between; margin-bottom:8px}
  .pay-row{display:flex; gap:8px; align-items:center}
  input[type="number"], input[type="text"]{padding:8px; border-radius:8px; border:1px solid rgba(255,255,255,0.04); background:transparent; color:inherit}
  .receipt{white-space:pre-wrap; background:#071426; padding:10px; border-radius:8px; max-height:45vh; overflow:auto; font-family:monospace; font-size:13px}
  footer{display:flex; justify-content:space-between; gap:8px}
  .small{font-size:13px; color:var(--muted)}
  /* modal */
  .modal{
    position:fixed; inset:0; display:none; align-items:center; justify-content:center; background:rgba(2,6,23,0.6); z-index:40;
  }
  .modal.show{display:flex}
  .modal .card{width:480px; max-width:96%; padding:16px}
  .form-row{display:flex; gap:8px; margin-bottom:8px}
  .badge{background:rgba(255,255,255,0.03); padding:6px 8px; border-radius:8px; font-weight:700; color:var(--muted)}
  /* responsif */
  @media (max-width:920px){
    .app{grid-template-columns:1fr}
  }
</style>
</head>
<body>
<div class="app">
  <div class="panel left">
    <header>
      <div>
        <h1>Web Kasir</h1>
        <div class="small">Simple POS — klik item untuk tambah</div>
      </div>
      <div class="top-actions">
        <button class="btn" id="adminBtn">Admin</button>
        <button class="btn ghost" id="clearBtn">Clear Cart</button>
      </div>
    </header>

    <div class="search">
      <div class="categories" id="categories"></div>
      <input id="search" placeholder="Cari menu..." />
    </div>

    <div class="grid" id="menuGrid" aria-live="polite"></div>
  </div>

  <div class="panel right">
    <header style="display:flex; justify-content:space-between; align-items:center">
      <div>
        <strong>Keranjang</strong>
        <div class="small" id="orderIdArea"></div>
      </div>
      <div class="badge" id="totalItems">0 item</div>
    </header>

    <div class="cart-list" id="cartList"></div>

    <div class="summary">
      <div class="summary-row"><div>Subtotal</div><div id="subtotal">Rp 0</div></div>
      <div class="summary-row"><div>Pajak (0%)</div><div id="tax">Rp 0</div></div>
      <div class="summary-row" style="font-weight:800; font-size:18px"><div>Total</div><div id="total">Rp 0</div></div>

      <div style="margin-top:8px" class="pay-row">
        <input type="number" id="paid" placeholder="Bayar (Rp)" />
        <button class="btn" id="payBtn">Proses Bayar</button>
      </div>
      <div style="margin-top:8px" class="summary-row"><div>Kembalian</div><div id="change">Rp 0</div></div>
    </div>

    <div style="display:flex; gap:8px; margin-top:8px">
      <button class="btn" id="checkoutBtn">Checkout (Cetak Struk)</button>
      <button class="btn ghost" id="saveOrderBtn">Simpan Order</button>
    </div>

    <div style="margin-top:12px">
      <strong>Struk</strong>
      <div class="receipt" id="receipt">Belum ada transaksi</div>
    </div>

  </div>
</div>

<!-- modal admin -->
<div class="modal" id="modal">
  <div class="card panel">
    <header style="display:flex; justify-content:space-between; align-items:center">
      <strong>Admin Panel</strong>
      <button class="btn ghost" id="closeModal">Tutup</button>
    </header>

    <div id="adminArea" style="margin-top:10px">
      <div>
        <label class="small">Masuk (Password Admin)</label>
        <div style="display:flex; gap:8px; margin-bottom:10px">
          <input type="password" id="adminPass" placeholder="Password" />
          <button class="btn" id="adminLogin">Login</button>
        </div>
      </div>

      <div id="adminControls" style="display:none; margin-top:8px">
        <div style="margin-bottom:8px">
          <div class="small">Tambah Kategori</div>
          <div style="display:flex; gap:8px; margin-top:6px">
            <input id="newCat" placeholder="Nama kategori" />
            <button class="btn" id="addCat">Tambah</button>
          </div>
        </div>

        <div style="margin-bottom:8px">
          <div class="small">Tambah Menu</div>
          <div class="form-row">
            <input id="newName" placeholder="Nama menu" />
            <input id="newPrice" type="number" placeholder="Harga" />
            <select id="newCatSelect"></select>
            <button class="btn" id="addItem">Tambah Menu</button>
          </div>
        </div>

        <div style="margin-top:10px">
          <div class="small">Daftar menu (klik untuk edit/hapus)</div>
          <div id="adminList" style="max-height:160px; overflow:auto; margin-top:8px"></div>
        </div>

        <div style="margin-top:10px">
          <button class="btn ghost" id="exportMenu">Export Menu (JSON)</button>
          <button class="btn ghost" id="importMenuBtn">Import Menu</button>
          <input type="file" id="importFile" style="display:none" accept=".json" />
        </div>
      </div>
    </div>

  </div>
</div>

<script>
/* ---------------------------
 Simple POS logic (single file)
 Persist menu to localStorage under key 'pos_menu'
 Persist orders optionally under 'pos_orders'
 Admin password default: "admin123" (change in code)
 ---------------------------*/
const LS_MENU='pos_menu_v1';
const LS_ORDERS='pos_orders_v1';
const ADMIN_PW='admin123'; // change if needed

// default sample menu (used on first load)
const SAMPLE_MENU = {
  categories: ['Minuman','Makanan','Snack'],
  items: [
    {id: genId(), name:'Es Teh', price:5000, category:'Minuman'},
    {id: genId(), name:'Kopi Hitam', price:8000, category:'Minuman'},
    {id: genId(), name:'Nasi Goreng', price:20000, category:'Makanan'},
    {id: genId(), name:'Mie Ayam', price:18000, category:'Makanan'},
    {id: genId(), name:'Pisang Goreng', price:7000, category:'Snack'},
    {id: genId(), name:'Keripik', price:6000, category:'Snack'}
  ]
};

// state
let menu = loadMenu();
let cart = [];
let activeCategory = 'ALL';
let searchTerm = '';

// ui refs
const categoriesEl = document.getElementById('categories');
const menuGrid = document.getElementById('menuGrid');
const cartList = document.getElementById('cartList');
const subtotalEl = document.getElementById('subtotal');
const totalEl = document.getElementById('total');
const taxEl = document.getElementById('tax');
const totalItemsEl = document.getElementById('totalItems');
const paidInput = document.getElementById('paid');
const changeEl = document.getElementById('change');
const receiptEl = document.getElementById('receipt');
const orderIdArea = document.getElementById('orderIdArea');

const modal = document.getElementById('modal');
const adminBtn = document.getElementById('adminBtn');
const closeModal = document.getElementById('closeModal');
const adminLogin = document.getElementById('adminLogin');
const adminPass = document.getElementById('adminPass');
const adminControls = document.getElementById('adminControls');
const newCat = document.getElementById('newCat');
const addCat = document.getElementById('addCat');
const newName = document.getElementById('newName');
const newPrice = document.getElementById('newPrice');
const newCatSelect = document.getElementById('newCatSelect');
const addItem = document.getElementById('addItem');
const adminList = document.getElementById('adminList');
const exportMenu = document.getElementById('exportMenu');
const importMenuBtn = document.getElementById('importMenuBtn');
const importFile = document.getElementById('importFile');

document.getElementById('clearBtn').addEventListener('click', ()=>{ if(confirm('Kosongkan keranjang?')) { cart=[]; renderCart(); }});
document.getElementById('payBtn').addEventListener('click', processPayment);
document.getElementById('checkoutBtn').addEventListener('click', checkout);
document.getElementById('saveOrderBtn').addEventListener('click', saveOrder);

// admin modal
adminBtn.addEventListener('click', ()=> modal.classList.add('show'));
closeModal.addEventListener('click', ()=> modal.classList.remove('show'));
adminLogin.addEventListener('click', () => {
  if(adminPass.value === ADMIN_PW){ adminControls.style.display='block'; adminPass.value=''; renderAdmin(); } else alert('Password salah');
});

// admin controls
addCat.addEventListener('click', ()=>{
  const v=newCat.value.trim(); if(!v) return alert('Nama kategori kosong');
  if(!menu.categories.includes(v)) menu.categories.push(v);
  saveMenu(); newCat.value=''; renderCategories(); renderAdmin();
});
addItem.addEventListener('click', ()=>{
  const name=newName.value.trim(); const price=Number(newPrice.value); const cat=newCatSelect.value;
  if(!name || !price || !cat) return alert('Lengkapi nama, harga, kategori');
  menu.items.push({id: genId(), name, price, category:cat});
  saveMenu(); newName.value=''; newPrice.value=''; renderMenu(); renderAdmin();
});
exportMenu.addEventListener('click', ()=>{
  const data=JSON.stringify(menu, null, 2);
  const blob=new Blob([data], {type:'application/json'}); const url=URL.createObjectURL(blob);
  const a=document.createElement('a'); a.href=url; a.download='menu-pos.json'; a.click(); URL.revokeObjectURL(url);
});
importMenuBtn.addEventListener('click', ()=> importFile.click());
importFile.addEventListener('change', async (e)=>{
  const f=e.target.files[0]; if(!f) return;
  try{
    const txt = await f.text();
    const parsed = JSON.parse(txt);
    if(parsed.categories && parsed.items){ menu = parsed; saveMenu(); renderAll(); modal.classList.remove('show'); alert('Menu berhasil diimport'); }
    else alert('File tidak valid');
  } catch(err){ alert('Gagal baca file: '+err.message); }
});

// search
document.getElementById('search').addEventListener('input', (e)=>{ searchTerm=e.target.value.trim().toLowerCase(); renderMenu(); });

// initial render
renderAll();
generateOrderId();

// ---------- functions ----------
function genId(){
  return 'i'+Math.random().toString(36).slice(2,9);
}
function loadMenu(){
  try{
    const raw = localStorage.getItem(LS_MENU);
    if(!raw){ localStorage.setItem(LS_MENU, JSON.stringify(SAMPLE_MENU)); return JSON.parse(JSON.stringify(SAMPLE_MENU)); }
    return JSON.parse(raw);
  } catch(e){
    console.error(e); localStorage.setItem(LS_MENU, JSON.stringify(SAMPLE_MENU)); return JSON.parse(JSON.stringify(SAMPLE_MENU));
  }
}
function saveMenu(){ localStorage.setItem(LS_MENU, JSON.stringify(menu)); }
function loadOrders(){ try{ return JSON.parse(localStorage.getItem(LS_ORDERS) || '[]'); }catch{ return []; } }
function saveOrders(orders){ localStorage.setItem(LS_ORDERS, JSON.stringify(orders)); }

function renderAll(){ renderCategories(); renderMenu(); renderCart(); }
function renderCategories(){
  categoriesEl.innerHTML='';
  const allBtn = el('div','ALL','cat'+(activeCategory==='ALL'?' active':'')); allBtn.addEventListener('click', ()=>{ activeCategory='ALL'; renderMenu(); renderCategories(); });
  categoriesEl.appendChild(allBtn);
  menu.categories.forEach(c=>{
    const d=el('div', c, 'cat'+(activeCategory===c?' active':'')); d.addEventListener('click', ()=>{ activeCategory=c; renderMenu(); renderCategories(); });
    categoriesEl.appendChild(d);
  });
}
function renderMenu(){
  menuGrid.innerHTML='';
  const filter = menu.items.filter(it=>{
    if(activeCategory!=='ALL' && it.category!==activeCategory) return false;
    if(searchTerm && !it.name.toLowerCase().includes(searchTerm)) return false;
    return true;
  });
  if(filter.length===0){ menuGrid.innerHTML='<div class="small" style="padding:8px">Tidak ada menu</div>'; return; }
  filter.forEach(it=>{
    const card = document.createElement('div'); card.className='item';
    card.innerHTML = `<div style="display:flex; justify-content:space-between; align-items:center">
                        <div class="title">${escapeHtml(it.name)}</div>
                        <div class="price">Rp ${formatRp(it.price)}</div>
                      </div>
                      <div class="meta">
                        <div class="small">${escapeHtml(it.category)}</div>
                        <div class="add">Tambah</div>
                      </div>`;
    card.querySelector('.add').addEventListener('click', (e)=>{ e.stopPropagation(); addToCart(it.id); });
    card.addEventListener('click', ()=> addToCart(it.id));
    menuGrid.appendChild(card);
  });
}

function addToCart(itemId){
  const item = menu.items.find(x=>x.id===itemId); if(!item) return;
  const existing = cart.find(c=>c.id===itemId);
  if(existing) existing.qty++;
  else cart.push({id:itemId, name:item.name, price:item.price, qty:1});
  renderCart();
}

function renderCart(){
  cartList.innerHTML='';
  if(cart.length===0){ cartList.innerHTML='<div class="small">Keranjang kosong</div>'; updateSummary(); return; }
  cart.forEach(row=>{
    const r = document.createElement('div'); r.className='cart-row';
    r.innerHTML = `<div class="name">${escapeHtml(row.name)}</div>
                   <div style="min-width:120px; display:flex; align-items:center; gap:8px">
                     <div class="qty">
                       <button class="tiny" data-act="dec">-</button>
                       <div>${row.qty}</div>
                       <button class="tiny" data-act="inc">+</button>
                     </div>
                     <div style="min-width:70px;text-align:right">Rp ${formatRp(row.price*row.qty)}</div>
                     <button data-act="del" title="Hapus">x</button>
                   </div>`;
    r.querySelector('[data-act="dec"]').addEventListener('click', ()=>{ if(row.qty>1) row.qty--; else if(confirm('Hapus item?')) cart = cart.filter(c=>c!==row); renderCart(); });
    r.querySelector('[data-act="inc"]').addEventListener('click', ()=>{ row.qty++; renderCart(); });
    r.querySelector('[data-act="del"]').addEventListener('click', ()=>{ if(confirm('Hapus item?')) { cart = cart.filter(c=>c!==row); renderCart(); }});
    cartList.appendChild(r);
  });
  updateSummary();
}

function updateSummary(){
  const subtotal = cart.reduce((s,i)=> s + i.price * i.qty, 0);
  const taxRate = 0; // if you want VAT, set here
  const tax = Math.round(subtotal * taxRate);
  const total = subtotal + tax;
  subtotalEl.textContent = 'Rp '+formatRp(subtotal);
  taxEl.textContent = 'Rp '+formatRp(tax);
  totalEl.textContent = 'Rp '+formatRp(total);
  totalItemsEl.textContent = `${cart.reduce((s,i)=>s+i.qty,0)} item`;
  // update change if paid
  const paid = Number(paidInput.value || 0);
  changeEl.textContent = 'Rp '+formatRp(Math.max(0, paid - total));
}

// payment
function processPayment(){
  const subtotal = cart.reduce((s,i)=> s + i.price * i.qty, 0);
  const total = subtotal;
  const paid = Number(paidInput.value || 0);
  if(total===0){ alert('Keranjang kosong'); return; }
  if(paid < total){ alert('Uang yang dibayarkan kurang'); return; }
  const change = paid - total;
  changeEl.textContent = 'Rp '+formatRp(change);
  alert('Pembayaran berhasil. Kembalian: Rp '+formatRp(change));
}

// checkout -> generate receipt and optionally save
function checkout(){
  const subtotal = cart.reduce((s,i)=> s + i.price * i.qty, 0);
  const total = subtotal;
  const paid = Number(paidInput.value || 0);
  if(total===0){ alert('Keranjang kosong'); return; }
  if(paid < total){ if(!confirm('Bayar kurang atau 0. Lanjutkan checkout?')) return; }
  const change = Math.max(0, paid - total);
  const order = {
    id: generateOrderId(),
    datetime: new Date().toLocaleString(),
    items: JSON.parse(JSON.stringify(cart)),
    subtotal, total, paid, change
  };
  // append to orders
  const orders = loadOrders();
  orders.push(order);
  saveOrders(orders);
  // show receipt
  const lines = [];
  lines.push('--- STRUK PEMBELIAN ---');
  lines.push('Order ID: '+order.id);
  lines.push('Waktu: '+order.datetime);
  lines.push('');
  order.items.forEach(it => lines.push(`${it.qty} x ${it.name} @ Rp ${formatRp(it.price)} = Rp ${formatRp(it.qty*it.price)}`));
  lines.push('');
  lines.push('Subtotal: Rp '+formatRp(order.subtotal));
  lines.push('Total:    Rp '+formatRp(order.total));
  lines.push('Bayar:    Rp '+formatRp(order.paid));
  lines.push('Kembali:  Rp '+formatRp(order.change));
  lines.push('-----------------------');
  receiptEl.textContent = lines.join('\\n');
  // reset cart (but keep menu)
  cart = [];
  paidInput.value = '';
  renderCart();
  generateOrderId();
}

// save order (without checkout)
function saveOrder(){
  if(cart.length===0) return alert('Keranjang kosong');
  const orders = loadOrders();
  orders.push({
    id: generateOrderId(),
    datetime: new Date().toLocaleString(),
    items: JSON.parse(JSON.stringify(cart)),
    subtotal: cart.reduce((s,i)=> s+i.price*i.qty,0),
    saved:true
  });
  saveOrders(orders);
  alert('Order disimpan (draft).');
}

/* admin render */
function renderAdmin(){
  // fill category select
  newCatSelect.innerHTML = menu.categories.map(c=>`<option value="${escapeHtml(c)}">${escapeHtml(c)}</option>`).join('');
  // list items
  adminList.innerHTML = menu.items.map(it=>{
    return `<div style="display:flex; justify-content:space-between; align-items:center; padding:6px; border-radius:6px; background:rgba(255,255,255,0.01); margin-bottom:6px">
      <div>
        <div style="font-weight:700">${escapeHtml(it.name)}</div>
        <div class="small">${escapeHtml(it.category)} — Rp ${formatRp(it.price)}</div>
      </div>
      <div style="display:flex; gap:6px">
        <button data-id="${it.id}" class="editBtn">Edit</button>
        <button data-id="${it.id}" class="delBtn">Hapus</button>
      </div>
    </div>`;
  }).join('');
  // wire up edit/delete
  adminList.querySelectorAll('.delBtn').forEach(b=>{
    b.addEventListener('click', ()=> {
      if(!confirm('Hapus menu ini?')) return;
      const id=b.dataset.id; menu.items = menu.items.filter(x=>x.id!==id); saveMenu(); renderMenu(); renderAdmin();
    });
  });
  adminList.querySelectorAll('.editBtn').forEach(b=>{
    b.addEventListener('click', ()=> {
      const id=b.dataset.id; const it = menu.items.find(x=>x.id===id);
      const newNameVal = prompt('Nama baru', it.name); if(newNameVal===null) return;
      const newPriceVal = prompt('Harga (angka)', String(it.price)); if(newPriceVal===null) return;
      const catVal = prompt('Kategori', it.category); if(catVal===null) return;
      it.name = newNameVal.trim(); it.price = Number(newPriceVal) || it.price; it.category = catVal.trim();
      if(!menu.categories.includes(it.category)) menu.categories.push(it.category);
      saveMenu(); renderMenu(); renderAdmin();
    });
  });
}

/* helpers */
function el(tag, text, className){
  const d = document.createElement('div'); d.className = className || ''; d.textContent = text; return d;
}
function formatRp(num){ return Number(num).toLocaleString('id-ID'); }
function escapeHtml(s){ return String(s).replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;'); }

function generateOrderId(){ orderIdArea.textContent = 'Order: #' + (Math.random().toString(36).slice(2,8)).toUpperCase(); }
function genOrderIdVal(){ return 'ORD-'+Math.random().toString(36).slice(2,9).toUpperCase(); }
function generateOrderId(){ orderIdArea.textContent = 'Order: #' + genOrderIdVal(); return orderIdArea.textContent; }
function loadMenuAndRender(){ menu = loadMenu(); renderAll(); }

// small keyboard/accessibility: Enter in paid field triggers pay
paidInput.addEventListener('keydown', (e)=>{ if(e.key==='Enter'){ processPayment(); }});
paidInput.addEventListener('input', updateSummary);

// initial
renderAll();
updateSummary();

</script>
</body>
</html>
[kasir.html](https://github.com/user-attachments/files/23774119/kasir.html)
