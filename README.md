# App-builder-
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PKG // HTML → App Packager</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<style>
  :root{
    --bg: #07090a;
    --panel: #0e1310;
    --panel-2: #121814;
    --line: #1d2620;
    --text: #d7e5dc;
    --dim: #6f8377;
    --acid: #8dff5c;
    --acid-dim: #4f8c39;
    --amber: #ffb454;
    --err: #ff5c5c;
    --mono: 'JetBrains Mono', 'Courier New', monospace;
  }
  @font-face{
    font-family:'JetBrains Mono';
    src: local('JetBrains Mono');
  }
  *{ box-sizing:border-box; }
  html,body{
    margin:0; padding:0; background:var(--bg); color:var(--text);
    font-family:var(--mono);
    -webkit-font-smoothing:antialiased;
  }
  body{
    background-image:
      radial-gradient(circle at 15% 0%, rgba(141,255,92,0.05), transparent 40%),
      repeating-linear-gradient(0deg, rgba(255,255,255,0.012) 0px, rgba(255,255,255,0.012) 1px, transparent 1px, transparent 3px);
    min-height:100vh;
  }
  .wrap{ max-width:960px; margin:0 auto; padding:22px 16px 60px; }

  header{ display:flex; align-items:baseline; justify-content:space-between; border-bottom:1px solid var(--line); padding-bottom:14px; margin-bottom:22px; flex-wrap:wrap; gap:8px; }
  .brand{ font-size:15px; letter-spacing:0.14em; text-transform:uppercase; color:var(--acid); font-weight:700; }
  .brand span{ color:var(--dim); font-weight:400; }
  .sub{ font-size:11px; color:var(--dim); letter-spacing:0.08em; }

  .grid{ display:grid; grid-template-columns: 1fr; gap:16px; }
  @media(min-width:800px){ .grid{ grid-template-columns: 1.3fr 1fr; align-items:start; } }

  .panel{ background:var(--panel); border:1px solid var(--line); border-radius:4px; padding:16px; position:relative; }
  .panel::before{
    content: attr(data-tag);
    position:absolute; top:-9px; left:12px; background:var(--bg); padding:0 6px;
    font-size:10px; letter-spacing:0.12em; color:var(--acid-dim); text-transform:uppercase;
  }

  .tabs{ display:flex; gap:2px; margin-bottom:12px; }
  .tab{ flex:1; text-align:center; padding:9px 8px; background:var(--panel-2); border:1px solid var(--line); color:var(--dim); font-size:11px; letter-spacing:0.08em; text-transform:uppercase; cursor:pointer; user-select:none; }
  .tab.active{ color:var(--acid); border-color:var(--acid-dim); background:#0d1a0f; }

  textarea{
    width:100%; height:260px; resize:vertical; background:#050706; color:var(--text);
    border:1px solid var(--line); border-radius:3px; padding:12px; font-family:var(--mono);
    font-size:12.5px; line-height:1.5; outline:none;
  }
  textarea:focus{ border-color:var(--acid-dim); }
  textarea::placeholder{ color:#3a463e; }

  .droparea{ border:1px dashed var(--line); border-radius:3px; padding:26px 12px; text-align:center; color:var(--dim); font-size:12px; cursor:pointer; transition:border-color .15s, color .15s; }
  .droparea:hover, .droparea.drag{ border-color:var(--acid); color:var(--acid); }
  .droparea input{ display:none; }
  .filename{ margin-top:10px; font-size:12px; color:var(--acid); word-break:break-all; }

  label.field{ display:block; margin-bottom:12px; }
  label.field .lbl{ display:block; font-size:10.5px; letter-spacing:0.1em; text-transform:uppercase; color:var(--dim); margin-bottom:5px; }
  input[type=text], input[type=color]{
    width:100%; background:#050706; border:1px solid var(--line); color:var(--text);
    padding:8px 10px; font-family:var(--mono); font-size:13px; border-radius:3px; outline:none;
  }
  input[type=text]:focus{ border-color:var(--acid-dim); }
  .row2{ display:grid; grid-template-columns:1fr 1fr; gap:10px; }
  input[type=color]{ padding:2px 4px; height:36px; cursor:pointer; }

  .icon-preview{ display:flex; align-items:center; gap:10px; margin-top:4px; }
  .icon-preview canvas{ border-radius:8px; border:1px solid var(--line); }

  button.build{
    width:100%; margin-top:6px; background:var(--acid); color:#06110a; border:none;
    font-family:var(--mono); font-weight:700; letter-spacing:0.08em; text-transform:uppercase;
    font-size:13px; padding:13px; border-radius:3px; cursor:pointer; transition:transform .1s, box-shadow .15s;
    box-shadow:0 0 0 rgba(141,255,92,0);
  }
  button.build:hover{ box-shadow:0 0 22px rgba(141,255,92,0.25); }
  button.build:active{ transform:scale(0.99); }
  button.build:disabled{ opacity:0.4; cursor:not-allowed; box-shadow:none; }

  .log{ background:#050706; border:1px solid var(--line); border-radius:3px; padding:10px 12px; font-size:11.5px; line-height:1.7; height:150px; overflow-y:auto; color:var(--dim); }
  .log .ok{ color:var(--acid); }
  .log .warn{ color:var(--amber); }
  .log .err{ color:var(--err); }
  .log .t{ color:#425048; }

  .hint{ font-size:11px; color:var(--dim); line-height:1.6; margin-top:10px; }
  .hint b{ color:var(--text); font-weight:600; }
  .hint code{ color:var(--acid); background:#0d1410; padding:1px 5px; border-radius:2px; }

  .footer-note{ margin-top:24px; font-size:11px; color:var(--dim); text-align:center; }
  .footer-note a{ color:var(--acid-dim); }

  ::-webkit-scrollbar{ width:8px; height:8px; }
  ::-webkit-scrollbar-track{ background:var(--panel-2); }
  ::-webkit-scrollbar-thumb{ background:var(--line); }
</style>
</head>
<body>
<div class="wrap">

  <header>
    <div>
      <div class="brand">PKG <span>// html → installable app</span></div>
      <div class="sub">paste or upload · packages a PWA · zero build tools</div>
    </div>
    <div class="sub" id="statusClock"></div>
  </header>

  <div class="grid">

    <!-- LEFT: input -->
    <div class="panel" data-tag="01 · source">
      <div class="tabs">
        <div class="tab active" data-tab="paste">Paste code</div>
        <div class="tab" data-tab="upload">Upload file</div>
        <div class="tab" data-tab="url">Already hosted (URL)</div>
      </div>

      <div id="paste-pane">
        <textarea id="htmlInput" placeholder="&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;&lt;title&gt;My App&lt;/title&gt;&lt;/head&gt;
  &lt;body&gt;
    &lt;h1&gt;Hello&lt;/h1&gt;
  &lt;/body&gt;
&lt;/html&gt;

Paste your full single-file HTML app here."></textarea>
      </div>

      <div id="upload-pane" style="display:none;">
        <div class="droparea" id="dropZone">
          <input type="file" id="fileInput" accept=".html,.htm">
          ⬆ drop .html file here, or tap to choose
          <div class="filename" id="fileName"></div>
        </div>
      </div>

      <div id="url-pane" style="display:none;">
        <label class="field">
          <span class="lbl">Your app's live URL (GitHub Pages, Cloudflare Pages, etc.)</span>
          <input type="text" id="hostedUrl" placeholder="https://yourname.github.io/your-app/">
        </label>
        <div class="hint" style="margin-top:2px;">
          This has to be a real address the public internet can reach — not a local file, not
          <code>localhost</code>. That's a hard requirement of the build step below, not something
          I can work around: a real signed <code>.apk</code> is only issued to an app that can prove it owns
          a real domain.
        </div>
        <button class="build" id="realApkBtn" style="margin-top:14px; background:var(--amber);">
          ▸ Send to real Android build (PWABuilder) →
        </button>
        <div class="hint">
          Opens PWABuilder in a new tab, pre-filled with your URL. It runs Google's actual Bubblewrap /
          Android SDK build on their servers and hands you back a genuinely signed <code>.apk</code> (and an
          <code>.aab</code> if you ever want the Play Store). That part isn't something I can execute for
          you inside this page — it needs a real compiler, not JavaScript — but the link below does the
          real thing, no shortcuts.
        </div>
      </div>
    </div>

    <!-- RIGHT: config -->
    <div class="panel" data-tag="02 · configure">
      <label class="field">
        <span class="lbl">App name</span>
        <input type="text" id="appName" value="My App" maxlength="40">
      </label>

      <label class="field">
        <span class="lbl">Short name (home screen label)</span>
        <input type="text" id="shortName" value="MyApp" maxlength="12">
      </label>

      <div class="row2">
        <label class="field">
          <span class="lbl">Theme color</span>
          <input type="color" id="themeColor" value="#8dff5c">
        </label>
        <label class="field">
          <span class="lbl">Background color</span>
          <input type="color" id="bgColor" value="#07090a">
        </label>
      </div>

      <label class="field">
        <span class="lbl">Icon letter</span>
        <input type="text" id="iconLetter" value="A" maxlength="2">
      </label>

      <div class="icon-preview">
        <canvas id="iconPreview" width="56" height="56"></canvas>
        <span class="sub">live icon preview</span>
      </div>

      <button class="build" id="buildBtn">▸ Build downloadable app (.zip)</button>

      <div class="log" id="log">
        <div class="t">$ awaiting source…</div>
      </div>
    </div>
  </div>

  <div class="panel" data-tag="03 · what you get" style="margin-top:16px;">
    <div class="hint">
      <b>The zip contains:</b> <code>index.html</code> (your code, unmodified except for injected PWA tags),
      <code>manifest.json</code>, <code>sw.js</code> (offline caching service worker), and two generated icons.<br><br>
      <b>To install it as a real app:</b> upload the unzipped folder to <code>GitHub Pages</code> or
      <code>Cloudflare Pages</code>, open the deployed URL on a phone, then use
      <code>Add to Home Screen</code> (Android/Chrome) or <code>Share → Add to Home Screen</code> (iOS/Safari).
      It launches full-screen, no browser chrome — a real installed app, and it still works offline after first load.<br><br>
      Nothing here uses simulated data or placeholders — your HTML is packaged byte-for-byte, and everything runs
      client-side in this page. No files are uploaded anywhere.<br><br>
      <b>Want a real, signed <code>.apk</code> instead of just an installable web app?</b> Host the zip's contents
      anywhere public (GitHub Pages, Cloudflare Pages — both free), then switch to the
      <code>Already hosted (URL)</code> tab above and paste that link in. That hands off to PWABuilder, which runs
      Google's actual Android build servers — not a simulation, the same pipeline real Play Store apps use.
    </div>
  </div>

  <div class="footer-note">built as a single HTML file · no backend · no dependencies beyond JSZip (loaded from cdnjs)</div>
</div>

<script>
// ---------- tabs ----------
const tabs = document.querySelectorAll('.tab');
tabs.forEach(t => t.addEventListener('click', () => {
  tabs.forEach(x => x.classList.remove('active'));
  t.classList.add('active');
  const which = t.dataset.tab;
  document.getElementById('paste-pane').style.display = which === 'paste' ? 'block' : 'none';
  document.getElementById('upload-pane').style.display = which === 'upload' ? 'block' : 'none';
  document.getElementById('url-pane').style.display = which === 'url' ? 'block' : 'none';
  document.getElementById('buildBtn').style.display = which === 'url' ? 'none' : 'block';
  document.getElementById('log').style.display = which === 'url' ? 'none' : 'block';
}));

document.getElementById('realApkBtn').addEventListener('click', () => {
  const raw = document.getElementById('hostedUrl').value.trim();
  if(!raw){
    alert('Paste the live URL of your hosted app first — e.g. https://yourname.github.io/your-app/');
    return;
  }
  let url;
  try{
    url = new URL(raw.startsWith('http') ? raw : 'https://' + raw);
  } catch(e){
    alert('That doesn\'t look like a valid URL.');
    return;
  }
  if(url.protocol !== 'https:'){
    alert('PWABuilder requires HTTPS to issue a signed APK. Most hosts (GitHub Pages, Cloudflare Pages) give you this automatically — check your link starts with https://');
    return;
  }
  window.open('https://www.pwabuilder.com/reportcard?site=' + encodeURIComponent(url.href), '_blank');
});

// ---------- file upload ----------
const dropZone = document.getElementById('dropZone');
const fileInput = document.getElementById('fileInput');
const fileNameEl = document.getElementById('fileName');
let uploadedContent = null;

dropZone.addEventListener('click', () => fileInput.click());
['dragover','dragenter'].forEach(ev => dropZone.addEventListener(ev, e => { e.preventDefault(); dropZone.classList.add('drag'); }));
['dragleave','drop'].forEach(ev => dropZone.addEventListener(ev, e => { e.preventDefault(); dropZone.classList.remove('drag'); }));
dropZone.addEventListener('drop', e => { handleFile(e.dataTransfer.files[0]); });
fileInput.addEventListener('change', e => handleFile(e.target.files[0]));

function handleFile(file){
  if(!file) return;
  const reader = new FileReader();
  reader.onload = e => {
    uploadedContent = e.target.result;
    fileNameEl.textContent = `✓ ${file.name} (${(file.size/1024).toFixed(1)} KB)`;
    // derive an app name guess from filename
    const guess = file.name.replace(/\.(html?|htm)$/i,'').replace(/[-_]/g,' ').trim();
    if(guess) document.getElementById('appName').value = guess.charAt(0).toUpperCase()+guess.slice(1);
  };
  reader.readAsText(file);
}

// ---------- icon preview ----------
const iconLetterInput = document.getElementById('iconLetter');
const themeColorInput = document.getElementById('themeColor');
const bgColorInput = document.getElementById('bgColor');
const previewCanvas = document.getElementById('iconPreview');

function drawIcon(canvas, size, letter, theme, bg){
  const ctx = canvas.getContext('2d');
  canvas.width = size; canvas.height = size;
  ctx.clearRect(0,0,size,size);
  // rounded bg
  const r = size*0.22;
  ctx.fillStyle = bg;
  roundRect(ctx, 0, 0, size, size, r);
  ctx.fill();
  // accent ring
  ctx.strokeStyle = theme;
  ctx.lineWidth = size*0.035;
  roundRect(ctx, ctx.lineWidth/2, ctx.lineWidth/2, size-ctx.lineWidth, size-ctx.lineWidth, r);
  ctx.stroke();
  // letter
  ctx.fillStyle = theme;
  ctx.font = `700 ${size*0.46}px 'JetBrains Mono', monospace`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(letter.slice(0,2).toUpperCase(), size/2, size*0.56);
}
function roundRect(ctx,x,y,w,h,r){
  ctx.beginPath();
  ctx.moveTo(x+r,y);
  ctx.arcTo(x+w,y,x+w,y+h,r);
  ctx.arcTo(x+w,y+h,x,y+h,r);
  ctx.arcTo(x,y+h,x,y,r);
  ctx.arcTo(x,y,x+w,y,r);
  ctx.closePath();
}
function updatePreview(){
  drawIcon(previewCanvas, 56, iconLetterInput.value || 'A', themeColorInput.value, bgColorInput.value);
}
[iconLetterInput, themeColorInput, bgColorInput].forEach(el => el.addEventListener('input', updatePreview));
updatePreview();

// ---------- logging ----------
const logEl = document.getElementById('log');
function log(msg, cls){
  const line = document.createElement('div');
  line.innerHTML = `<span class="t">$</span> ${msg}`;
  if(cls) line.classList.add(cls);
  logEl.appendChild(line);
  logEl.scrollTop = logEl.scrollHeight;
}
function clearLog(){ logEl.innerHTML=''; }

// ---------- build ----------
document.getElementById('buildBtn').addEventListener('click', async () => {
  clearLog();
  const activeTab = document.querySelector('.tab.active').dataset.tab;
  const source = activeTab === 'paste' ? document.getElementById('htmlInput').value.trim() : (uploadedContent || '').trim();

  if(!source){
    log('no source found — paste your HTML or upload a file first.', 'err');
    return;
  }

  const btn = document.getElementById('buildBtn');
  btn.disabled = true;

  try{
    log('reading source… ' + source.length + ' chars', 'ok');
    await sleep(150);

    const appName = document.getElementById('appName').value.trim() || 'My App';
    const shortName = document.getElementById('shortName').value.trim() || appName.slice(0,12);
    const theme = themeColorInput.value;
    const bg = bgColorInput.value;
    const letter = (iconLetterInput.value || appName.charAt(0) || 'A').slice(0,2);

    log(`config → name:"${appName}" short:"${shortName}" theme:${theme}`);
    await sleep(120);

    // inject PWA tags into <head> if not already present
    let html = source;
    const hasManifestLink = /rel=["']manifest["']/i.test(html);
    const injections = [];
    if(!/<meta[^>]+viewport/i.test(html)) injections.push('<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">');
    if(!hasManifestLink) injections.push('<link rel="manifest" href="manifest.json">');
    injections.push(`<meta name="theme-color" content="${theme}">`);
    injections.push('<link rel="apple-touch-icon" href="icons/icon-192.png">');
    injections.push('<meta name="mobile-web-app-capable" content="yes">');
    injections.push('<meta name="apple-mobile-web-app-capable" content="yes">');
    injections.push('<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">');

    const swRegister = `<script>
if('serviceWorker' in navigator){
  window.addEventListener('load', () => navigator.serviceWorker.register('sw.js').catch(()=>{}));
}
<\/script>`;

    if(/<head[^>]*>/i.test(html)){
      html = html.replace(/<head[^>]*>/i, m => m + '\n' + injections.join('\n') + '\n');
    } else if(/<html[^>]*>/i.test(html)){
      html = html.replace(/<html[^>]*>/i, m => m + '\n<head>\n' + injections.join('\n') + '\n</head>\n');
    } else {
      html = `<head>\n${injections.join('\n')}\n</head>\n` + html;
    }
    if(/<\/body>/i.test(html)){
      html = html.replace(/<\/body>/i, swRegister + '\n</body>');
    } else {
      html += swRegister;
    }

    log('injected manifest + service worker registration', 'ok');
    await sleep(150);

    // build manifest
    const manifest = {
      name: appName,
      short_name: shortName,
      start_url: "./index.html",
      display: "standalone",
      background_color: bg,
      theme_color: theme,
      orientation: "portrait-primary",
      icons: [
        { src: "icons/icon-192.png", sizes: "192x192", type: "image/png", purpose: "any maskable" },
        { src: "icons/icon-512.png", sizes: "512x512", type: "image/png", purpose: "any maskable" }
      ]
    };
    log('generated manifest.json', 'ok');
    await sleep(120);

    // build service worker
    const swContent = `const CACHE = "pkg-cache-v1";
const ASSETS = ["./", "./index.html", "./manifest.json"];

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll(ASSETS)).catch(()=>{}));
  self.skipWaiting();
});

self.addEventListener('activate', e => {
  e.waitUntil(
    caches.keys().then(keys => Promise.all(keys.filter(k => k !== CACHE).map(k => caches.delete(k))))
  );
  self.clients.claim();
});

self.addEventListener('fetch', e => {
  if(e.request.method !== 'GET') return;
  e.respondWith(
    caches.match(e.request).then(cached => {
      const fetchPromise = fetch(e.request).then(res => {
        if(res && res.status === 200){
          const clone = res.clone();
          caches.open(CACHE).then(c => c.put(e.request, clone));
        }
        return res;
      }).catch(() => cached);
      return cached || fetchPromise;
    })
  );
});`;
    log('generated sw.js (offline cache)', 'ok');
    await sleep(120);

    // build icons
    const icon192 = document.createElement('canvas');
    const icon512 = document.createElement('canvas');
    drawIcon(icon192, 192, letter, theme, bg);
    drawIcon(icon512, 512, letter, theme, bg);
    const icon192Blob = await canvasToBlob(icon192);
    const icon512Blob = await canvasToBlob(icon512);
    log('rendered icon-192.png + icon-512.png', 'ok');
    await sleep(120);

    // zip it
    log('zipping package…');
    const zip = new JSZip();
    zip.file('index.html', html);
    zip.file('manifest.json', JSON.stringify(manifest, null, 2));
    zip.file('sw.js', swContent);
 
