<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="mobile-web-app-capable" content="yes">
<title>Stacks — a local reader</title>
<script src="vendor/pdf.min.js"></script>
<script src="vendor/jszip.min.js"></script>
<script src="vendor/epub.min.js"></script>
<style>
:root{
  --paper:#efe9dd;
  --paper-2:#e4dcc9;
  --ink:#2b2620;
  --ink-soft:#6b6255;
  --spine:#8a3b2b;
  --spine-dark:#6e2f22;
  --gold:#b6924f;
  --line:#d8cdb4;
  --shadow: 0 10px 30px rgba(30,20,10,.25);
  --radius: 2px;
  --serif: 'Iowan Old Style','Palatino Linotype', Georgia, 'Noto Serif', serif;
  --display: 'Canela', 'Georgia', serif;
  --mono: 'Courier New', monospace;
}
[data-theme="night"]{
  --paper:#1b1712;
  --paper-2:#241f18;
  --ink:#ddd3bf;
  --ink-soft:#93897490;
  --line:#3a3226;
}
*{box-sizing:border-box;}
html,body{margin:0;height:100%;font-family:var(--serif);color:var(--ink);background:var(--paper);}
body{display:flex;overflow:hidden;}

/* ===== Spine sidebar ===== */
#spine{
  width:250px;min-width:250px;
  background:linear-gradient(180deg,var(--spine),var(--spine-dark));
  color:#f2e6d8;
  display:flex;flex-direction:column;
  box-shadow:inset -6px 0 18px rgba(0,0,0,.25);
}
#spine .brand{
  padding:22px 20px 14px;
  font-family:var(--display);
  font-size:22px;letter-spacing:.5px;
  border-bottom:1px solid rgba(255,255,255,.15);
  display:flex;align-items:center;gap:10px;
}
#spine .brand small{display:block;font-family:var(--serif);font-size:11px;opacity:.65;letter-spacing:1.5px;text-transform:uppercase;margin-top:2px;}
#addBtn{
  margin:16px 18px 6px;padding:10px 14px;
  background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.35);
  color:#f6ecdd;border-radius:var(--radius);cursor:pointer;
  font-family:var(--serif);font-size:13.5px;letter-spacing:.3px;
  transition:background .15s ease;
}
#addBtn:hover{background:rgba(255,255,255,.18);}
#fileInput{display:none;}
#shelf{flex:1;overflow-y:auto;padding:6px 8px 16px;}
.book-item{
  padding:11px 12px;margin:4px 2px;border-radius:var(--radius);
  cursor:pointer;display:flex;flex-direction:column;gap:2px;
  border-left:3px solid transparent;
  transition:background .12s ease, border-color .12s ease;
}
.book-item:hover{background:rgba(255,255,255,.07);}
.book-item.active{background:rgba(255,255,255,.13);border-left-color:var(--gold);}
.book-title{font-size:14px;line-height:1.3;}
.book-meta{font-size:10.5px;opacity:.6;text-transform:uppercase;letter-spacing:1px;}
.book-progress{height:2px;background:rgba(255,255,255,.15);border-radius:2px;margin-top:5px;overflow:hidden;}
.book-progress i{display:block;height:100%;background:var(--gold);}
.book-del{opacity:0;float:right;font-size:12px;color:#f2e6d8aa;background:none;border:none;cursor:pointer;}
.book-item:hover .book-del{opacity:.7;}
.book-del:hover{opacity:1 !important;color:#fff;}
#emptyShelf{padding:30px 20px;font-size:12.5px;line-height:1.7;opacity:.55;}

/* ===== Reading pane ===== */
#stage{flex:1;display:flex;flex-direction:column;min-width:0;background:var(--paper);}
#toolbar{
  display:flex;align-items:center;gap:14px;
  padding:10px 22px;border-bottom:1px solid var(--line);
  background:var(--paper-2);
  font-size:12.5px;color:var(--ink-soft);
}
#toolbar .ttl{font-family:var(--display);font-size:15px;color:var(--ink);flex:1;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;}
.tbtn{
  background:none;border:1px solid var(--line);color:var(--ink);
  padding:5px 10px;border-radius:var(--radius);cursor:pointer;font-size:12px;
  font-family:var(--serif);
}
.tbtn:hover{background:var(--line);}
#pageInfo{min-width:70px;text-align:center;}

#reader-wrap{flex:1;overflow:auto;position:relative;}
#welcome{
  height:100%;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:16px;
  color:var(--ink-soft);text-align:center;padding:40px;
}
#welcome h1{font-family:var(--display);font-size:34px;color:var(--ink);font-weight:400;margin:0;}
#welcome .rule{width:60px;height:1px;background:var(--gold);}
#welcome p{max-width:420px;font-size:14px;line-height:1.7;margin:0;}

#txtView{max-width:700px;margin:0 auto;padding:60px 40px 120px;font-size:18px;line-height:1.8;white-space:pre-wrap;}
#epubView{width:100%;height:100%;}
#pdfView{display:flex;justify-content:center;padding:30px 0 100px;}
#pdfCanvas{box-shadow:var(--shadow);background:#fff;}
#dropOverlay{
  position:fixed;inset:0;background:rgba(30,20,10,.55);color:#f2e6d8;
  display:none;align-items:center;justify-content:center;font-family:var(--display);font-size:26px;
  z-index:50;pointer-events:none;
}
body.dragging #dropOverlay{display:flex;}
::-webkit-scrollbar{width:10px;}
::-webkit-scrollbar-thumb{background:var(--line);border-radius:5px;}

/* ===== Mobile drawer chrome (hidden on desktop) ===== */
#drawerToggle{
  display:none;position:fixed;top:calc(10px + env(safe-area-inset-top));
  left:calc(10px + env(safe-area-inset-left));
  z-index:60;width:42px;height:42px;border-radius:50%;
  background:var(--spine);color:#f2e6d8;border:none;font-size:18px;
  box-shadow:var(--shadow);
}
#scrim{display:none;position:fixed;inset:0;background:rgba(20,14,8,.5);z-index:39;}
body.drawerOpen #scrim{display:block;}

/* ===== Mobile layout ===== */
@media (max-width: 760px){
  #drawerToggle{display:flex;align-items:center;justify-content:center;}
  #spine{
    position:fixed;inset:0 auto 0 0;z-index:40;
    width:82vw;max-width:320px;
    transform:translateX(-100%);
    transition:transform .25s ease;
    box-shadow:12px 0 30px rgba(0,0,0,.35);
    padding-top:env(safe-area-inset-top);
  }
  body.drawerOpen #spine{transform:translateX(0);}
  #toolbar #menuBtn{display:inline-flex;}
  #stage{width:100%;}
  #toolbar{
    padding:8px 12px;padding-top:calc(8px + env(safe-area-inset-top));
    gap:8px;flex-wrap:nowrap;overflow-x:auto;
  }
  #toolbar .ttl{font-size:13px;order:2;}
  .tbtn{padding:9px 11px;font-size:13px;min-height:40px;min-width:40px;flex:0 0 auto;}
  #pageInfo{min-width:52px;font-size:11px;}
  #txtView{padding:24px 18px 100px;font-size:17px;}
  #pdfView{padding:14px 0 90px;}
  #pdfCanvas{max-width:96vw;height:auto !important;}
  #welcome h1{font-size:27px;}
  #welcome p{font-size:13.5px;padding:0 6px;}
  #reader-wrap{padding-bottom:env(safe-area-inset-bottom);}
  .book-item{padding:13px 12px;}
  #addBtn{padding:12px 14px;font-size:14px;}
}
@media (min-width:761px){
  #menuBtn{display:none;}
}
</style>
</head>
<body data-theme="day">

<button id="drawerToggle" aria-label="Open shelf">☰</button>
<div id="scrim"></div>
<div id="spine">
  <div class="brand">📚<div><div>Stacks</div><small>your shelf</small></div></div>
  <button id="addBtn">+ Add books</button>
  <input type="file" id="fileInput" multiple accept=".epub,.pdf,.txt">
  <div id="shelf"><div id="emptyShelf">Nothing on the shelf yet. Add an epub, pdf, or txt file to begin.</div></div>
</div>

<div id="stage">
  <div id="toolbar" style="display:none;">
    <button class="tbtn" id="menuBtn">☰</button>
    <div class="ttl" id="toolTitle">—</div>
    <button class="tbtn" id="prevBtn">‹ Prev</button>
    <span id="pageInfo">—</span>
    <button class="tbtn" id="nextBtn">Next ›</button>
    <button class="tbtn" id="fontDown">A−</button>
    <button class="tbtn" id="fontUp">A+</button>
    <button class="tbtn" id="themeBtn">☾ Night</button>
  </div>
  <div id="reader-wrap">
    <div id="welcome">
      <h1>Stacks</h1>
      <div class="rule"></div>
      <p>A quiet place to read. Drop an EPUB, PDF, or TXT file anywhere on this window, or use “Add books” on the left. Everything stays on this device.</p>
    </div>
    <div id="txtView" style="display:none;"></div>
    <div id="epubView" style="display:none;"></div>
    <div id="pdfView" style="display:none;"><canvas id="pdfCanvas"></canvas></div>
  </div>
</div>
<div id="dropOverlay">Drop to add to your shelf</div>

<script>
pdfjsLib.GlobalWorkerOptions.workerSrc = "vendor/pdf.worker.min.js";

// ---------- IndexedDB storage ----------
const DB_NAME='stacksDB', STORE='books';
function openDB(){
  return new Promise((res,rej)=>{
    const req=indexedDB.open(DB_NAME,1);
    req.onupgradeneeded=()=>req.result.createObjectStore(STORE,{keyPath:'id'});
    req.onsuccess=()=>res(req.result);
    req.onerror=()=>rej(req.error);
  });
}
async function dbPut(book){
  const db=await openDB();
  return new Promise((res,rej)=>{
    const tx=db.transaction(STORE,'readwrite');
    tx.objectStore(STORE).put(book);
    tx.oncomplete=()=>res();
    tx.onerror=()=>rej(tx.error);
  });
}
async function dbGetAll(){
  const db=await openDB();
  return new Promise((res,rej)=>{
    const tx=db.transaction(STORE,'readonly');
    const req=tx.objectStore(STORE).getAll();
    req.onsuccess=()=>res(req.result);
    req.onerror=()=>rej(req.error);
  });
}
async function dbDelete(id){
  const db=await openDB();
  return new Promise((res,rej)=>{
    const tx=db.transaction(STORE,'readwrite');
    tx.objectStore(STORE).delete(id);
    tx.oncomplete=()=>res();
    tx.onerror=()=>rej(tx.error);
  });
}

// ---------- State ----------
let library=[];        // [{id,name,type,blob,progress,location}]
let activeId=null;
let currentEpubBook=null, currentEpubRendition=null;
let currentPdfDoc=null, currentPdfPage=1, currentPdfScale=1.3;
let fontSize=18;

const $=s=>document.querySelector(s);
const shelf=$('#shelf'), emptyShelf=$('#emptyShelf');

function fmtType(t){ return t.toUpperCase(); }

function renderShelf(){
  shelf.querySelectorAll('.book-item').forEach(e=>e.remove());
  emptyShelf.style.display = library.length ? 'none' : 'block';
  library.slice().sort((a,b)=>b.added-a.added).forEach(book=>{
    const div=document.createElement('div');
    div.className='book-item'+(book.id===activeId?' active':'');
    const pct = book.progressPct ? Math.round(book.progressPct*100) : 0;
    div.innerHTML = `
      <div class="book-title">${book.name.replace(/\.(epub|pdf|txt)$/i,'')} <button class="book-del" title="Remove">✕</button></div>
      <div class="book-meta">${fmtType(book.type)}${pct?` · ${pct}%`:''}</div>
      <div class="book-progress"><i style="width:${pct}%"></i></div>
    `;
    div.addEventListener('click',(e)=>{
      if(e.target.classList.contains('book-del')){
        e.stopPropagation();
        removeBook(book.id);
        return;
      }
      openBook(book.id);
    });
    shelf.appendChild(div);
  });
}

async function removeBook(id){
  library = library.filter(b=>b.id!==id);
  await dbDelete(id);
  if(activeId===id){ activeId=null; showWelcome(); }
  renderShelf();
}

function showWelcome(){
  $('#welcome').style.display='flex';
  $('#txtView').style.display='none';
  $('#epubView').style.display='none';
  $('#pdfView').style.display='none';
  $('#toolbar').style.display='none';
}

async function addFiles(fileList){
  for(const file of fileList){
    const ext = file.name.split('.').pop().toLowerCase();
    if(!['epub','pdf','txt'].includes(ext)) continue;
    const id = 'b_'+Date.now()+'_'+Math.random().toString(36).slice(2,8);
    const book = { id, name:file.name, type:ext, blob:file, added:Date.now(), progressPct:0, location:null };
    library.push(book);
    await dbPut(book);
  }
  renderShelf();
}

async function loadLibrary(){
  library = await dbGetAll();
  renderShelf();
}

// ---------- Opening books ----------
async function openBook(id){
  const book = library.find(b=>b.id===id);
  if(!book) return;
  activeId = id;
  renderShelf();
  $('#toolbar').style.display='flex';
  $('#toolTitle').textContent = book.name.replace(/\.(epub|pdf|txt)$/i,'');
  $('#welcome').style.display='none';

  if(book.type==='txt') await openTxt(book);
  else if(book.type==='pdf') await openPdf(book);
  else if(book.type==='epub') await openEpub(book);
}

async function openTxt(book){
  $('#epubView').style.display='none'; $('#pdfView').style.display='none';
  const view=$('#txtView'); view.style.display='block';
  view.style.fontSize=fontSize+'px';
  const text = await book.blob.text();
  view.textContent = text;
  $('#pageInfo').textContent='';
  $('#prevBtn').style.visibility='hidden'; $('#nextBtn').style.visibility='hidden';
  // restore scroll progress
  requestAnimationFrame(()=>{
    const wrap=$('#reader-wrap');
    if(book.progressPct) wrap.scrollTop = book.progressPct*(wrap.scrollHeight-wrap.clientHeight);
  });
  const wrap=$('#reader-wrap');
  wrap.onscroll = ()=>{
    const pct = wrap.scrollTop/(wrap.scrollHeight-wrap.clientHeight||1);
    book.progressPct = Math.max(0,Math.min(1,pct));
    dbPut(book);
    updateShelfProgressOnly(book);
  };
}

function updateShelfProgressOnly(book){
  const item = [...shelf.querySelectorAll('.book-item')].find(el=>el.querySelector('.book-title').textContent.startsWith(book.name.replace(/\.(epub|pdf|txt)$/i,'')));
  if(!item) return;
  const pct=Math.round((book.progressPct||0)*100);
  item.querySelector('.book-meta').textContent = `${fmtType(book.type)}${pct?` · ${pct}%`:''}`;
  item.querySelector('.book-progress i').style.width=pct+'%';
}

async function openPdf(book){
  $('#txtView').style.display='none'; $('#epubView').style.display='none';
  $('#pdfView').style.display='flex';
  $('#reader-wrap').onscroll=null;
  const buf = await book.blob.arrayBuffer();
  currentPdfDoc = await pdfjsLib.getDocument({data:buf}).promise;
  currentPdfPage = book.location ? parseInt(book.location) : 1;
  $('#prevBtn').style.visibility='visible'; $('#nextBtn').style.visibility='visible';
  await renderPdfPage();
}

async function renderPdfPage(){
  const book = library.find(b=>b.id===activeId);
  const page = await currentPdfDoc.getPage(currentPdfPage);
  const viewport = page.getViewport({scale:currentPdfScale});
  const canvas = $('#pdfCanvas');
  canvas.width=viewport.width; canvas.height=viewport.height;
  const ctx=canvas.getContext('2d');
  await page.render({canvasContext:ctx, viewport}).promise;
  $('#pageInfo').textContent = `${currentPdfPage} / ${currentPdfDoc.numPages}`;
  if(book){
    book.location = String(currentPdfPage);
    book.progressPct = currentPdfPage/currentPdfDoc.numPages;
    dbPut(book);
    updateShelfProgressOnly(book);
  }
}

async function openEpub(book){
  $('#txtView').style.display='none'; $('#pdfView').style.display='none';
  const view=$('#epubView'); view.style.display='block';
  view.innerHTML='';
  $('#reader-wrap').onscroll=null;
  const buf = await book.blob.arrayBuffer();
  currentEpubBook = ePub(buf);
  currentEpubRendition = currentEpubBook.renderTo(view, {width:'100%', height:'100%', flow:'paginated'});
  currentEpubRendition.themes.fontSize(fontSize+'px');
  applyEpubTheme();
  await currentEpubRendition.display(book.location || undefined);
  $('#prevBtn').style.visibility='visible'; $('#nextBtn').style.visibility='visible';

  currentEpubRendition.on('relocated', (loc)=>{
    book.location = loc.start.cfi;
    const pct = currentEpubBook.locations && currentEpubBook.locations.length()
      ? currentEpubBook.locations.percentageFromCfi(loc.start.cfi) : 0;
    book.progressPct = pct;
    dbPut(book);
    updateShelfProgressOnly(book);
    $('#pageInfo').textContent = pct ? Math.round(pct*100)+'%' : '';
  });
  currentEpubBook.ready.then(()=>currentEpubBook.locations.generate(1600));
}

function applyEpubTheme(){
  if(!currentEpubRendition) return;
  const night = document.body.getAttribute('data-theme')==='night';
  currentEpubRendition.themes.default({
    'body': { background: night? '#1b1712':'#efe9dd', color: night? '#ddd3bf':'#2b2620' }
  });
}

// ---------- Navigation ----------
$('#prevBtn').addEventListener('click', ()=>{
  const book=library.find(b=>b.id===activeId); if(!book) return;
  if(book.type==='pdf'){ if(currentPdfPage>1){currentPdfPage--; renderPdfPage();} }
  else if(book.type==='epub'){ currentEpubRendition && currentEpubRendition.prev(); }
});
$('#nextBtn').addEventListener('click', ()=>{
  const book=library.find(b=>b.id===activeId); if(!book) return;
  if(book.type==='pdf'){ if(currentPdfPage<currentPdfDoc.numPages){currentPdfPage++; renderPdfPage();} }
  else if(book.type==='epub'){ currentEpubRendition && currentEpubRendition.next(); }
});
document.addEventListener('keydown',(e)=>{
  if(e.key==='ArrowRight') $('#nextBtn').click();
  if(e.key==='ArrowLeft') $('#prevBtn').click();
});

$('#fontUp').addEventListener('click', ()=>{ fontSize=Math.min(32,fontSize+2); applyFont(); });
$('#fontDown').addEventListener('click', ()=>{ fontSize=Math.max(12,fontSize-2); applyFont(); });
function applyFont(){
  const book=library.find(b=>b.id===activeId); if(!book) return;
  if(book.type==='txt') $('#txtView').style.fontSize=fontSize+'px';
  if(book.type==='epub' && currentEpubRendition) currentEpubRendition.themes.fontSize(fontSize+'px');
  if(book.type==='pdf'){ currentPdfScale = fontSize/18*1.3; renderPdfPage(); }
}

$('#themeBtn').addEventListener('click', ()=>{
  const night = document.body.getAttribute('data-theme')==='night';
  document.body.setAttribute('data-theme', night?'day':'night');
  $('#themeBtn').textContent = night ? '☾ Night' : '☀ Day';
  applyEpubTheme();
});

// ---------- File input / drag&drop ----------
$('#addBtn').addEventListener('click', ()=>$('#fileInput').click());
$('#fileInput').addEventListener('change', (e)=>{ addFiles(e.target.files); e.target.value=''; });

['dragenter','dragover'].forEach(evt=>window.addEventListener(evt, e=>{
  e.preventDefault(); document.body.classList.add('dragging');
}));
['dragleave','drop'].forEach(evt=>window.addEventListener(evt, e=>{
  e.preventDefault();
  if(evt==='drop'){ document.body.classList.remove('dragging'); addFiles(e.dataTransfer.files); }
  else if(e.target===document.documentElement || e.relatedTarget===null) document.body.classList.remove('dragging');
}));

// ---------- Mobile drawer ----------
function openDrawer(){ document.body.classList.add('drawerOpen'); }
function closeDrawer(){ document.body.classList.remove('drawerOpen'); }
$('#drawerToggle').addEventListener('click', openDrawer);
$('#menuBtn').addEventListener('click', openDrawer);
$('#scrim').addEventListener('click', closeDrawer);
function isMobile(){ return window.matchMedia('(max-width: 760px)').matches; }

// wrap openBook to auto-close drawer on mobile after picking a book
const _openBook = openBook;
openBook = async function(id){
  await _openBook(id);
  if(isMobile()) closeDrawer();
};

// ---------- Swipe gestures for page turning ----------
(function(){
  let sx=0, sy=0, tracking=false;
  const wrap = $('#reader-wrap');
  wrap.addEventListener('touchstart', e=>{
    if(e.touches.length!==1) return;
    sx=e.touches[0].clientX; sy=e.touches[0].clientY; tracking=true;
  }, {passive:true});
  wrap.addEventListener('touchend', e=>{
    if(!tracking) return; tracking=false;
    const book = library.find(b=>b.id===activeId);
    if(!book || book.type==='txt') return; // txt scrolls normally
    const dx = e.changedTouches[0].clientX - sx;
    const dy = e.changedTouches[0].clientY - sy;
    if(Math.abs(dx) > 60 && Math.abs(dx) > Math.abs(dy)*1.5){
      if(dx < 0) $('#nextBtn').click(); else $('#prevBtn').click();
    }
  }, {passive:true});
})();

loadLibrary();
</script>
</body>
</html>
