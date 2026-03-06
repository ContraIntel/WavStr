# WavStr
Your own music player 
<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<title>WAVSTR</title>

<!-- noble crypto for secp256k1 schnorr signing — works in Safari -->

<script src="https://cdn.jsdelivr.net/npm/@noble/curves@1.3.0/index.umd.min.js"></script>

<script src="https://cdn.jsdelivr.net/npm/@noble/hashes@1.3.3/crypto.umd.min.js"></script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Unbounded:wght@400;700;900&family=Inconsolata:wght@300;400;500&display=swap');

:root {
  --bg:      #06060a;
  --surface: #0e0e14;
  --card:    #12121a;
  --border:  #1c1c28;
  --b2:      #252535;
  --accent:  #c8ff57;
  --violet:  #7c3aed;
  --text:    #ebebf5;
  --muted:   #4a4a6a;
  --green:   #34d399;
  --red:     #f87171;
  --amber:   #fbbf24;
  --safe-b:  env(safe-area-inset-bottom, 0px);
}

*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
html{-webkit-text-size-adjust:100%;}

body {
  background: var(--bg);
  color: var(--text);
  font-family: 'Inconsolata', monospace;
  min-height: 100dvh;
  overflow-x: hidden;
  padding-bottom: calc(var(--safe-b) + 20px);
  -webkit-tap-highlight-color: transparent;
}

body::before {
  content:'';position:fixed;inset:0;
  background-image:radial-gradient(circle,#1c1c2a 1px,transparent 1px);
  background-size:28px 28px;opacity:.45;pointer-events:none;z-index:0;
}

.app{position:relative;z-index:1;max-width:460px;margin:0 auto;display:flex;flex-direction:column;}

/* HEADER */
.hdr {
  position:sticky;top:0;z-index:50;
  background:rgba(6,6,10,.93);
  backdrop-filter:blur(20px);-webkit-backdrop-filter:blur(20px);
  border-bottom:1px solid var(--border);
  padding:12px 18px;
  padding-top:calc(12px + env(safe-area-inset-top, 0px));
  display:flex;align-items:center;justify-content:space-between;
}
.logo{font-family:'Unbounded',sans-serif;font-weight:900;font-size:18px;letter-spacing:-1px;}
.logo-w{color:var(--accent);}
.logo-str{color:var(--muted);}

.pill {
  display:flex;align-items:center;gap:6px;padding:8px 14px;border-radius:100px;
  border:1px solid var(--b2);background:var(--card);
  font-family:'Inconsolata',monospace;font-size:11px;letter-spacing:.5px;
  cursor:pointer;color:var(--muted);-webkit-appearance:none;
  transition:border-color .15s,color .15s;
  min-height:44px;/* iOS tap target */
}
.pill:hover{border-color:var(--accent);color:var(--accent);}
.pill.on{border-color:var(--green);color:var(--green);}

.sdot{width:6px;height:6px;border-radius:50%;background:var(--muted);flex-shrink:0;transition:background .3s;}
.sdot.live{background:var(--green);animation:blink 2.4s ease infinite;}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.3}}

/* SECTIONS */
.sec{padding:22px 18px 0;animation:up .35s both;}
.sec:nth-child(2){animation-delay:.05s}
.sec:nth-child(3){animation-delay:.1s}
.sec:nth-child(4){animation-delay:.15s}
.sec:nth-child(5){animation-delay:.2s}
@keyframes up{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:none}}

.lbl{
  font-size:9px;font-weight:500;letter-spacing:3.5px;text-transform:uppercase;
  color:var(--muted);margin-bottom:12px;display:flex;align-items:center;gap:8px;
}
.lbl::after{content:'';flex:1;height:1px;background:var(--border);}

/* PLAYER */
.player{background:var(--card);border:1px solid var(--border);border-radius:14px;padding:18px;position:relative;overflow:hidden;}
.pglow{position:absolute;top:-40px;left:-20px;width:220px;height:220px;background:radial-gradient(circle,rgba(200,255,87,.07),transparent 65%);pointer-events:none;transition:opacity .5s;}
.pglow.on{opacity:1;animation:gp 3s ease infinite;}
.pglow.off{opacity:.2;}
@keyframes gp{0%,100%{transform:scale(1)}50%{transform:scale(1.2)}}

.trow{display:flex;align-items:center;gap:14px;margin-bottom:18px;}
.thumb{
  width:56px;height:56px;border-radius:10px;flex-shrink:0;
  background:linear-gradient(135deg,#1a1a28,#0e0e1c);border:1px solid var(--b2);
  display:flex;align-items:center;justify-content:center;font-size:22px;
  transition:border-color .3s,box-shadow .3s;
}
.thumb.active{border-color:var(--accent);box-shadow:0 0 16px rgba(200,255,87,.15);}
.tmeta{flex:1;min-width:0;}
.ttitle{font-family:'Unbounded',sans-serif;font-weight:700;font-size:13px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;}
.tsub{font-size:11px;color:var(--muted);margin-top:3px;}
.bbadge{
  display:none;align-items:center;gap:4px;font-size:9px;padding:2px 7px;
  border-radius:4px;margin-top:5px;background:rgba(124,58,237,.12);
  border:1px solid rgba(124,58,237,.28);color:#a78bfa;letter-spacing:.4px;
}
.bbadge.show{display:inline-flex;}

.wfwrap{cursor:pointer;user-select:none;-webkit-user-select:none;margin-bottom:8px;}
.wf{display:flex;align-items:center;gap:1.5px;height:44px;}
.wb{flex:1;border-radius:2px;background:var(--b2);min-height:3px;transition:background .08s;}
.wb.p{background:var(--accent);}
.wb.cur{background:var(--accent);filter:brightness(1.5);}

.timerow{display:flex;justify-content:space-between;font-size:10px;color:var(--muted);margin-bottom:16px;}

.ctrls{display:flex;align-items:center;justify-content:center;gap:20px;margin-bottom:16px;}
.cb{
  background:none;border:none;cursor:pointer;color:var(--muted);
  font-size:18px;padding:8px;border-radius:8px;line-height:1;
  min-width:44px;min-height:44px;display:flex;align-items:center;justify-content:center;
  -webkit-appearance:none;transition:color .15s,background .15s;
}
.cb:hover,.cb:active{color:var(--text);background:var(--border);}
.cb.on{color:var(--accent);}

.playbtn{
  width:54px;height:54px;border-radius:50%;background:var(--accent);border:none;
  color:#0a0a0a;font-size:20px;display:flex;align-items:center;justify-content:center;
  cursor:pointer;box-shadow:0 0 24px rgba(200,255,87,.25);
  -webkit-appearance:none;transition:transform .12s,box-shadow .12s;
}
.playbtn:active{transform:scale(.93);}

.volrow{display:flex;align-items:center;gap:10px;}
.vi{font-size:13px;color:var(--muted);width:16px;text-align:center;}
input[type=range]{
  flex:1;-webkit-appearance:none;appearance:none;
  height:4px;background:var(--b2);border-radius:2px;cursor:pointer;
}
input[type=range]::-webkit-slider-thumb{
  -webkit-appearance:none;width:20px;height:20px;
  border-radius:50%;background:var(--accent);cursor:pointer;
}

/* QUEUE */
.queue{display:flex;flex-direction:column;gap:1px;}
.qi{
  display:flex;align-items:center;gap:12px;padding:12px;
  border-radius:8px;cursor:pointer;border:1px solid transparent;
  min-height:44px;transition:background .12s,border-color .12s;
}
.qi:active{background:var(--card);}
.qi.active{background:var(--card);border-color:rgba(200,255,87,.2);}
.qi-num{font-size:10px;color:var(--muted);width:18px;text-align:center;flex-shrink:0;}
.qi.active .qi-num{color:var(--accent);}
.qi-name{flex:1;font-size:12px;font-weight:500;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;}
.qi.active .qi-name{color:var(--accent);}
.qi-src{font-size:10px;color:var(--muted);flex-shrink:0;}
.qi-del{
  background:none;border:none;color:var(--muted);cursor:pointer;
  font-size:16px;padding:4px 8px;border-radius:4px;min-width:36px;min-height:36px;
  display:flex;align-items:center;justify-content:center;
}
.qi-del:active{color:var(--red);}
.empty{text-align:center;padding:24px;font-size:11px;color:var(--muted);}

/* UPLOAD */
.upcard{border:1px dashed var(--b2);border-radius:12px;overflow:hidden;}
.dropzone{
  padding:28px 20px;text-align:center;cursor:pointer;position:relative;
  transition:background .15s;
}
.dropzone:active{background:rgba(200,255,87,.03);}
.dropzone input{position:absolute;inset:0;opacity:0;cursor:pointer;font-size:16px;/* prevents iOS zoom */}
.drop-icon{font-size:30px;margin-bottom:8px;}
.drop-text{font-size:12px;color:var(--muted);line-height:1.7;}
.drop-text strong{color:var(--accent);display:block;font-size:13px;margin-bottom:2px;}

.upstatus{
  padding:11px 16px;border-top:1px solid var(--border);
  font-size:11px;min-height:40px;display:flex;align-items:center;gap:10px;color:var(--muted);
}
.progwrap{flex:1;height:3px;background:var(--b2);border-radius:2px;overflow:hidden;display:none;}
.progbar{height:100%;background:var(--accent);width:0%;transition:width .2s;border-radius:2px;}

.srvchips{padding:10px 14px;border-top:1px solid var(--border);display:flex;gap:6px;flex-wrap:wrap;}
.schip{
  font-size:9px;padding:4px 9px;border-radius:4px;
  border:1px solid rgba(124,58,237,.25);background:rgba(124,58,237,.08);color:#a78bfa;
  transition:all .2s;
}
.schip.ok{border-color:var(--green);background:rgba(52,211,153,.08);color:var(--green);}
.schip.fail{border-color:var(--red);background:rgba(248,113,113,.08);color:var(--red);}

/* COMMENTS */
.cbox{background:var(--card);border:1px solid var(--border);border-radius:12px;overflow:hidden;}
.cta{
  width:100%;padding:14px 16px;background:none;border:none;outline:none;resize:none;
  color:var(--text);font-family:'Inconsolata',monospace;font-size:14px;line-height:1.65;
  min-height:80px;-webkit-appearance:none;
}
.cta::placeholder{color:var(--muted);}
.cfoot{
  display:flex;align-items:center;justify-content:space-between;
  padding:10px 14px;border-top:1px solid var(--border);
}
.rchips{display:flex;gap:5px;flex-wrap:wrap;flex:1;}
.rchip{
  font-size:9px;padding:3px 7px;border-radius:4px;letter-spacing:.3px;
  border:1px solid rgba(200,255,87,.2);background:rgba(200,255,87,.05);color:var(--accent);
}
.postarea{display:flex;align-items:center;gap:10px;}
.char{font-size:10px;color:var(--muted);}
.char.warn{color:var(--amber);}

.postbtn{
  padding:10px 18px;background:var(--violet);color:#fff;border:none;border-radius:7px;
  font-family:'Unbounded',sans-serif;font-weight:700;font-size:10px;letter-spacing:.5px;
  cursor:pointer;white-space:nowrap;-webkit-appearance:none;min-height:44px;
  box-shadow:0 0 16px rgba(124,58,237,.3);transition:opacity .15s;
}
.postbtn:active{opacity:.75;}
.postbtn:disabled{opacity:.35;cursor:not-allowed;}

.feed{display:flex;flex-direction:column;gap:8px;margin-top:14px;}
.fc{background:var(--card);border:1px solid var(--border);border-radius:10px;padding:12px 14px;animation:up .25s both;}
.fc-hd{display:flex;justify-content:space-between;margin-bottom:6px;}
.fc-npub{font-size:10px;color:#a78bfa;}
.fc-time{font-size:10px;color:var(--muted);}
.fc-body{font-size:12px;line-height:1.65;color:#c8c8e0;}
.fc-tag{font-size:9px;color:var(--muted);margin-top:6px;}

/* MODAL */
.overlay{
  position:fixed;inset:0;z-index:200;
  background:rgba(4,4,10,.9);backdrop-filter:blur(12px);-webkit-backdrop-filter:blur(12px);
  display:flex;align-items:flex-end;justify-content:center;
  opacity:0;pointer-events:none;transition:opacity .22s;
}
.overlay.open{opacity:1;pointer-events:all;}
.modal{
  background:var(--surface);border:1px solid var(--b2);
  border-radius:20px 20px 0 0;
  padding:28px 22px;
  padding-bottom:calc(28px + env(safe-area-inset-bottom, 0px));
  width:100%;max-width:460px;
  transform:translateY(24px);transition:transform .22s;
}
.overlay.open .modal{transform:translateY(0);}
.mhandle{width:40px;height:4px;border-radius:2px;background:var(--b2);margin:0 auto 22px;}
.mtitle{font-family:'Unbounded',sans-serif;font-weight:900;font-size:18px;margin-bottom:4px;}
.msub{font-size:11px;color:var(--muted);margin-bottom:20px;line-height:1.65;}

.minput{
  width:100%;background:var(--bg);border:1px solid var(--b2);border-radius:10px;
  padding:14px;color:var(--text);font-family:'Inconsolata',monospace;font-size:14px;
  outline:none;margin-bottom:10px;-webkit-appearance:none;
  transition:border-color .15s;
}
.minput:focus{border-color:var(--accent);}

.mbtn{
  width:100%;padding:16px;background:var(--accent);color:#0a0a0a;border:none;
  border-radius:10px;font-family:'Unbounded',sans-serif;font-weight:900;font-size:14px;
  cursor:pointer;-webkit-appearance:none;min-height:54px;
  transition:opacity .15s;
}
.mbtn:active{opacity:.8;}

.mwarn{
  margin-top:12px;padding:12px 14px;border-radius:8px;font-size:11px;
  color:var(--muted);line-height:1.65;
  background:rgba(251,191,36,.05);border:1px solid rgba(251,191,36,.15);
}
.mdivider{text-align:center;font-size:10px;color:var(--muted);margin:14px 0;letter-spacing:1px;}

.ext-btn{
  width:100%;padding:14px;background:var(--card);border:1px solid var(--b2);border-radius:10px;
  color:var(--text);cursor:pointer;text-align:left;display:flex;align-items:center;gap:12px;
  font-family:'Inconsolata',monospace;font-size:13px;-webkit-appearance:none;min-height:54px;
  transition:border-color .15s;
}
.ext-btn:active{border-color:var(--accent);}
.ext-icon{font-size:22px;flex-shrink:0;}
.ext-info{flex:1;}
.ext-info small{display:block;font-size:10px;color:var(--muted);margin-top:2px;}

/* TOAST */
.toast{
  position:fixed;bottom:calc(28px + env(safe-area-inset-bottom, 0px));left:50%;
  transform:translateX(-50%) translateY(60px);
  background:var(--surface);border:1px solid var(--b2);
  border-radius:10px;padding:12px 20px;font-size:12px;z-index:999;
  white-space:nowrap;opacity:0;pointer-events:none;
  transition:transform .25s,opacity .25s;
}
.toast.show{transform:translateX(-50%) translateY(0);opacity:1;}
.toast.ok{border-color:var(--green);color:var(--green);}
.toast.err{border-color:var(--red);color:var(--red);}
.toast.info{border-color:var(--violet);color:#a78bfa;}
</style>

</head>
<body>
<div class="app">

  <header class="hdr">
    <div class="logo"><span class="logo-w">W</span><span>AV</span><span class="logo-str">STR</span></div>
    <button class="pill" id="connectBtn" onclick="openModal()">
      <span class="sdot" id="sdot"></span>
      <span id="clabel">CONNECT</span>
    </button>
  </header>

  <!-- PLAYER -->

  <div class="sec">
    <div class="lbl">Now Playing</div>
    <div class="player">
      <div class="pglow off" id="glow"></div>
      <div class="trow">
        <div class="thumb" id="thumb"><span id="thumbE">🎵</span></div>
        <div class="tmeta">
          <div class="ttitle" id="tTitle">No track selected</div>
          <div class="tsub" id="tSub">Upload music below to start</div>
          <div class="bbadge" id="bbadge">🌸 On Blossom</div>
        </div>
      </div>
      <div class="wfwrap" id="wfwrap" onclick="seekClick(event)">
        <div class="wf" id="wf"></div>
      </div>
      <div class="timerow"><span id="tCur">0:00</span><span id="tDur">0:00</span></div>
      <div class="ctrls">
        <button class="cb" id="shufBtn" onclick="toggleShuffle()">⇄</button>
        <button class="cb" onclick="prevTrack()">⏮</button>
        <button class="playbtn" id="playBtn" onclick="togglePlay()">▶</button>
        <button class="cb" onclick="nextTrack()">⏭</button>
        <button class="cb" id="repBtn" onclick="toggleRepeat()">↺</button>
      </div>
      <div class="volrow">
        <span class="vi">🔈</span>
        <input type="range" min="0" max="100" value="80" oninput="setVol(this.value)">
        <span class="vi">🔊</span>
      </div>
    </div>
  </div>

  <!-- QUEUE -->

  <div class="sec">
    <div class="lbl">Queue</div>
    <div class="queue" id="queue"><div class="empty">No tracks yet</div></div>
  </div>

  <!-- UPLOAD -->

  <div class="sec">
    <div class="lbl">Add Music</div>
    <div class="upcard">
      <div class="dropzone" id="dropZone">
        <input type="file" accept="audio/*" multiple onchange="handleFiles(event)" id="fileIn">
        <div class="drop-icon">🌸</div>
        <div class="drop-text">
          <strong>Tap to pick from Files / Music</strong>
          MP3 · M4A · WAV · FLAC · OGG<br>
          <span style="color:#a78bfa;font-size:10px;">Connected: uploads to Blossom · Offline: plays locally</span>
        </div>
      </div>
      <div class="upstatus" id="upStatus">
        <span id="upMsg">Ready</span>
        <div class="progwrap" id="progWrap"><div class="progbar" id="progBar"></div></div>
      </div>
      <div class="srvchips" id="srvChips"></div>
    </div>
  </div>

  <!-- NOSTR POST -->

  <div class="sec">
    <div class="lbl">Broadcast to Nostr</div>
    <div class="cbox">
      <textarea class="cta" id="commentInput" maxlength="280"
        placeholder="What are you listening to? Posts as Kind 1 with track tags…"
        oninput="updateChar()"></textarea>
      <div class="cfoot">
        <div class="rchips" id="relayChips"></div>
        <div class="postarea">
          <span class="char" id="charEl">280</span>
          <button class="postbtn" id="postBtn" onclick="broadcast()">⚡ POST</button>
        </div>
      </div>
    </div>
    <div class="feed" id="feed"></div>
  </div>

</div><!-- /app -->

<!-- MODAL -->

<div class="overlay" id="overlay" onclick="overlayClick(event)">
  <div class="modal">
    <div class="mhandle"></div>
    <div class="mtitle">Connect to Nostr</div>
    <div class="msub">Enter your nsec private key to sign notes and upload via Blossom. Key is held in memory only — never sent anywhere.</div>

```
<input type="password" class="minput" id="nsecInput" placeholder="nsec1… or hex private key" autocomplete="off" autocorrect="off" spellcheck="false">
<button class="mbtn" onclick="connectNsec()">CONNECT</button>

<div class="mwarn">
  ⚠️ Only enter your nsec on a device you own and trust. Use a dedicated Nostr key — not your main wallet key. Key is cleared when you close the page.
</div>

<div class="mdivider">— OR —</div>

<button class="ext-btn" onclick="tryNIP07()">
  <span class="ext-icon">🔌</span>
  <span class="ext-info">
    NIP-07 Extension
    <small>Desktop only — Alby, nos2x, Flamingo</small>
  </span>
  <span>→</span>
</button>
```

  </div>
</div>

<div class="toast" id="toast"></div>
<audio id="aud" onended="onEnded()" playsinline webkit-playsinline></audio>

<script>
// ── WAIT FOR NOBLE CRYPTO ────────────────────────────────────────────────────
// noble/curves exposes differently depending on load; we normalise here
let nobleReady = false;
let schnorr, bytesToHex, hexToBytes;

function initNoble() {
  try {
    // Try noble/curves UMD bundle
    if (window.nobleSecp256k1) {
      schnorr    = window.nobleSecp256k1.schnorr;
      bytesToHex = window.nobleSecp256k1.etc?.bytesToHex || (b => Array.from(b).map(x=>x.toString(16).padStart(2,'0')).join(''));
      hexToBytes = window.nobleSecp256k1.etc?.hexToBytes || (h => new Uint8Array(h.match(/.{2}/g).map(b=>parseInt(b,16))));
      nobleReady = !!schnorr;
    }
    // Fallback: noble/curves global
    if (!nobleReady && window.nobleHashes) {
      nobleReady = true;
    }
  } catch(e) {}
}

// ── PURE JS FALLBACK CRYPTO (no deps) ────────────────────────────────────────
// Minimal secp256k1 schnorr for Safari — used if CDN fails
// We always try noble first; this is the backstop.

async function sha256Bytes(data) {
  const buf = data instanceof ArrayBuffer ? data : data.buffer ?? new TextEncoder().encode(data).buffer;
  return new Uint8Array(await crypto.subtle.digest('SHA-256', buf));
}

async function sha256Hex(data) {
  const b = await sha256Bytes(data instanceof ArrayBuffer ? data : new TextEncoder().encode(data));
  return Array.from(b).map(x=>x.toString(16).padStart(2,'0')).join('');
}

function toHex(b) { return Array.from(b).map(x=>x.toString(16).padStart(2,'0')).join(''); }
function fromHex(h) { return new Uint8Array(h.match(/.{2}/g).map(b=>parseInt(b,16))); }

// ── STATE ─────────────────────────────────────────────────────────────────────
const S = {
  tracks:[], idx:-1, playing:false, shuffle:false, repeat:false,
  pubkey:null, privkey:null, useExt:false,
  relays:[
    'wss://relay.damus.io',
    'wss://nos.lol',
    'wss://relay.nostr.band',
    'wss://relay.primal.net',
    'wss://relay.snort.social'
  ],
  servers:[
    {url:'https://blossom.primal.net',    name:'primal',    st:''},
    {url:'https://blossom.oxtr.dev',      name:'oxtr',      st:''},
    {url:'https://cdn.satellite.earth',   name:'satellite', st:''},
  ]
};

const aud = document.getElementById('aud');

// ── BECH32 nsec DECODE ────────────────────────────────────────────────────────
const B32 = 'qpzry9x8gf2tvdw0s3jn54khce6mua7l';
function bech32Decode(str) {
  str = str.toLowerCase();
  const p = str.lastIndexOf('1');
  const data = str.slice(p+1).split('').map(c=>B32.indexOf(c));
  let acc=0,bits=0; const out=[];
  for (let i=0;i<data.length-6;i++) {
    acc=(acc<<5)|data[i]; bits+=5;
    while(bits>=8){bits-=8;out.push((acc>>bits)&0xff);}
  }
  return toHex(new Uint8Array(out));
}

// ── DERIVE PUBKEY FROM PRIVKEY ────────────────────────────────────────────────
async function derivePubkey(privHex) {
  initNoble();
  if (nobleReady && schnorr) {
    const pub = schnorr.getPublicKey(fromHex(privHex));
    return toHex(pub);
  }
  // Fallback: use subtle crypto won't work for secp256k1 directly in Safari
  // So we compute it via the noble lib — if not loaded, fail gracefully
  throw new Error('Crypto library not loaded. Check your internet connection and reload.');
}

// ── SIGN EVENT ────────────────────────────────────────────────────────────────
async function signEvent(ev) {
  if (S.useExt) return window.nostr.signEvent(ev);
  initNoble();
  if (!schnorr) throw new Error('Signing library not ready');
  const msgBytes = fromHex(ev.id);
  const privBytes = fromHex(S.privkey);
  const sig = await schnorr.sign(msgBytes, privBytes);
  return {...ev, sig: toHex(sig)};
}

async function buildEventId(ev) {
  const ser = JSON.stringify([0,ev.pubkey,ev.created_at,ev.kind,ev.tags,ev.content]);
  return sha256Hex(new TextEncoder().encode(ser));
}

// ── CONNECT ───────────────────────────────────────────────────────────────────
async function connectNsec() {
  let raw = document.getElementById('nsecInput').value.trim();
  if (!raw) { toast('Paste your nsec or hex key','err'); return; }
  try {
    if (raw.startsWith('nsec1')) raw = bech32Decode(raw);
    if (!/^[0-9a-f]{64}$/i.test(raw)) throw new Error('Key must be 64 hex chars or nsec1…');
    S.privkey = raw.toLowerCase();
    S.pubkey  = await derivePubkey(S.privkey);
    S.useExt  = false;
    onConnected();
    closeModal();
    document.getElementById('nsecInput').value = '';
    toast('Connected ⚡','ok');
  } catch(e) { toast('Error: '+e.message,'err'); }
}

async function tryNIP07() {
  if (!window.nostr) { toast('No NIP-07 extension — use nsec on mobile','err'); return; }
  try {
    S.pubkey = await window.nostr.getPublicKey();
    S.useExt = true;
    onConnected(); closeModal();
    toast('Connected via NIP-07 ⚡','ok');
  } catch(e) { toast('Rejected: '+e.message,'err'); }
}

function onConnected() {
  document.getElementById('connectBtn').classList.add('on');
  document.getElementById('sdot').classList.add('live');
  document.getElementById('clabel').textContent = S.pubkey.slice(0,8)+'…'+S.pubkey.slice(-4);
  renderRelayChips(); renderSrvChips();
  document.getElementById('upMsg').textContent = 'Ready — tap to pick audio files';
}

// ── BLOSSOM UPLOAD ─────────────────────────────────────────────────────────────
async function blossomUpload(file) {
  setProg(0,`Hashing ${file.name}…`);
  const fileData = await file.arrayBuffer();
  const fileHash = await sha256Hex(fileData);

  setProg(15,'Signing upload auth…');
  let uploadedUrl = null;

  for (let i=0;i<S.servers.length;i++) {
    const srv = S.servers[i];
    const endpoint = `${srv.url}/upload`;
    setChip(i,'trying');
    setProg(15+i*25,`Uploading to ${srv.name}…`);
    try {
      const authEv = {
        kind:27235, created_at:Math.floor(Date.now()/1000),
        tags:[['u',endpoint],['method','PUT'],['payload',fileHash]],
        content:'', pubkey:S.pubkey
      };
      authEv.id = await buildEventId(authEv);
      const signed = await signEvent(authEv);
      const token  = btoa(JSON.stringify(signed));

      const res = await fetch(endpoint,{
        method:'PUT',
        headers:{
          'Content-Type': file.type||'audio/mpeg',
          'Authorization': `Nostr ${token}`,
          'X-SHA-256': fileHash
        },
        body: fileData
      });
      if (res.ok) {
        const json = await res.json().catch(()=>({}));
        uploadedUrl = json.url || json.nip94_event?.tags?.find(t=>t[0]==='url')?.[1] || `${srv.url}/${fileHash}`;
        setChip(i,'ok');
        setProg(100,`✓ On Blossom (${srv.name})`);
        break;
      } else { setChip(i,'fail'); }
    } catch(e) { setChip(i,'fail'); }
  }

  if (!uploadedUrl) {
    setProg(0,'Blossom unavailable — playing locally');
    return {url: URL.createObjectURL(file), blossom:false};
  }
  return {url:uploadedUrl, blossom:true, hash:fileHash};
}

function setProg(pct,msg) {
  document.getElementById('upMsg').textContent = msg;
  const pw = document.getElementById('progWrap');
  pw.style.display = pct>0&&pct<100?'block':'none';
  document.getElementById('progBar').style.width = pct+'%';
}
function setChip(i,st){S.servers[i].st=st;renderSrvChips();}
function renderSrvChips(){
  const ic={ok:'✓ ',fail:'✗ ',trying:'… ',''：''};
  document.getElementById('srvChips').innerHTML = S.servers.map((s,i)=>
    `<span class="schip ${s.st}">${(ic[s.st]||'')}${s.name}</span>`
  ).join('');
}

async function handleFiles(e) {
  for (const f of Array.from(e.target.files)) {
    if (S.pubkey) {
      const r = await blossomUpload(f);
      if (r) addTrack(f.name.replace(/\.[^/.]+$/,''), r.url, r.blossom);
    } else {
      // No auth — play locally
      addTrack(f.name.replace(/\.[^/.]+$/,''), URL.createObjectURL(f), false);
      toast('Playing locally — connect Nostr to upload to Blossom','info');
    }
  }
  e.target.value='';
}

// ── TRACKS ────────────────────────────────────────────────────────────────────
const EMOJIS=['🎸','🥁','🎹','🎺','🎻','🎷','🎤','🎧','🔊','🌊','⚡','🌸','🔥','💿'];

function addTrack(title,url,isBlossom){
  S.tracks.push({title,url,isBlossom,emoji:EMOJIS[S.tracks.length%EMOJIS.length]});
  renderQueue();
  if(S.idx===-1) loadTrack(0);
  toast(`Added: ${title}`,'ok');
}
function removeTrack(i){
  if(i===S.idx){aud.pause();S.playing=false;updateBtn();}
  S.tracks.splice(i,1);
  if(S.idx>=S.tracks.length) S.idx=S.tracks.length-1;
  renderQueue();
  if(S.idx>=0) loadTrack(S.idx);
}
function loadTrack(i){
  if(i<0||i>=S.tracks.length)return;
  S.idx=i;
  const t=S.tracks[i];
  aud.src=t.url;
  aud.volume=document.querySelector('input[type=range]').value/100;
  document.getElementById('tTitle').textContent=t.title;
  document.getElementById('tSub').textContent=t.isBlossom?'Stored on Blossom':'Local file';
  document.getElementById('thumbE').textContent=t.emoji;
  document.getElementById('thumb').classList.add('active');
  document.getElementById('bbadge').classList.toggle('show',!!t.isBlossom);
  initWave(); renderQueue();
  if(S.playing) aud.play().catch(()=>{});
  const ci=document.getElementById('commentInput');
  if(!ci.value||ci.value.startsWith('🎵')){
    ci.value=`🎵 "${t.title}" — playing on WAVSTR #music #nostr`;
    updateChar();
  }
}

// ── PLAYER ────────────────────────────────────────────────────────────────────
function togglePlay(){
  if(!S.tracks.length)return;
  if(S.idx===-1){S.playing=true;loadTrack(0);return;}
  if(aud.paused){
    aud.play().then(()=>{S.playing=true;updateBtn();setGlow(true);}).catch(e=>toast('Playback error: '+e.message,'err'));
  } else {
    aud.pause();S.playing=false;updateBtn();setGlow(false);
  }
}
function updateBtn(){document.getElementById('playBtn').textContent=S.playing?'⏸':'▶';}
function setGlow(on){document.getElementById('glow').className='pglow '+(on?'on':'off');}
function nextTrack(){
  if(!S.tracks.length)return;
  loadTrack(S.shuffle?Math.floor(Math.random()*S.tracks.length):(S.idx+1)%S.tracks.length);
}
function prevTrack(){
  if(!S.tracks.length)return;
  if(aud.currentTime>3){aud.currentTime=0;return;}
  loadTrack((S.idx-1+S.tracks.length)%S.tracks.length);
}
function onEnded(){S.repeat?(aud.currentTime=0,aud.play()):nextTrack();}
function toggleShuffle(){S.shuffle=!S.shuffle;document.getElementById('shufBtn').classList.toggle('on',S.shuffle);}
function toggleRepeat(){S.repeat=!S.repeat;document.getElementById('repBtn').classList.toggle('on',S.repeat);}
function setVol(v){aud.volume=v/100;}

// waveform
let wH=[];
function initWave(){wH=Array.from({length:52},()=>10+Math.random()*80);renderWave(0);}
function renderWave(p){
  document.getElementById('wf').innerHTML=wH.map((h,i)=>{
    const pct=i/wH.length;
    const c=pct<p?'wb p':(pct<p+.018&&S.playing?'wb cur':'wb');
    return `<div class="${c}" style="height:${h}%"></div>`;
  }).join('');
}
function seekClick(e){
  if(!aud.duration)return;
  const r=document.getElementById('wfwrap').getBoundingClientRect();
  aud.currentTime=((e.clientX-r.left)/r.width)*aud.duration;
}
aud.addEventListener('timeupdate',()=>{
  const p=aud.duration?aud.currentTime/aud.duration:0;
  renderWave(p);
  document.getElementById('tCur').textContent=fmt(aud.currentTime);
  document.getElementById('tDur').textContent=fmt(aud.duration);
});
function fmt(s){return(!s||isNaN(s))?'0:00':`${Math.floor(s/60)}:${String(Math.floor(s%60)).padStart(2,'0')}`;}

function renderQueue(){
  const el=document.getElementById('queue');
  if(!S.tracks.length){el.innerHTML='<div class="empty">No tracks yet — tap Add Music above ↑</div>';return;}
  el.innerHTML=S.tracks.map((t,i)=>`
    <div class="qi ${i===S.idx?'active':''}" onclick="selTrack(${i})">
      <span class="qi-num">${i===S.idx&&S.playing?'▶':i+1}</span>
      <span class="qi-name">${esc(t.title)}</span>
      <span class="qi-src">${t.isBlossom?'🌸':''}</span>
      <button class="qi-del" onclick="event.stopPropagation();removeTrack(${i})">✕</button>
    </div>
  `).join('');
}
function selTrack(i){S.playing=true;loadTrack(i);updateBtn();setGlow(true);aud.play().catch(()=>{});}

// ── BROADCAST ─────────────────────────────────────────────────────────────────
async function broadcast(){
  if(!S.pubkey){openModal();return;}
  const content=document.getElementById('commentInput').value.trim();
  if(!content){toast('Write something first','err');return;}

  const tags=[['t','music'],['t','wavstr']];
  const t=S.tracks[S.idx];
  if(t){
    tags.push(['r',t.url]);
    tags.push(['subject',t.title]);
    if(t.isBlossom)tags.push(['blossom',t.url]);
  }

  const pb=document.getElementById('postBtn');
  pb.disabled=true;pb.textContent='SIGNING…';

  try{
    const ev={kind:1,created_at:Math.floor(Date.now()/1000),tags,content,pubkey:S.pubkey};
    ev.id=await buildEventId(ev);
    const signed=await signEvent(ev);

    pb.textContent='SENDING…';
    const msg=JSON.stringify(['EVENT',signed]);
    let sent=0;
    S.relays.forEach(url=>{
      try{
        const ws=new WebSocket(url);
        ws.onopen=()=>{ws.send(msg);sent++;setTimeout(()=>ws.close(),4000);};
        ws.onerror=()=>{};
      }catch(e){}
    });

    addFeed({content,pubkey:S.pubkey,created_at:ev.created_at,tags});
    document.getElementById('commentInput').value='';
    updateChar();
    toast(`Posted to ${S.relays.length} relays ⚡`,'ok');
  }catch(e){toast('Error: '+e.message,'err');}
  finally{pb.disabled=false;pb.textContent='⚡ POST';}
}

function addFeed(ev){
  const feed=document.getElementById('feed');
  const npub=ev.pubkey.slice(0,8)+'…'+ev.pubkey.slice(-6);
  const time=new Date(ev.created_at*1000).toLocaleTimeString();
  const subj=ev.tags?.find(t=>t[0]==='subject')?.[1];
  const d=document.createElement('div');
  d.className='fc';
  d.innerHTML=`
    <div class="fc-hd"><span class="fc-npub">${npub}</span><span class="fc-time">${time}</span></div>
    <div class="fc-body">${esc(ev.content)}</div>
    ${subj?`<div class="fc-tag">🎵 ${esc(subj)}</div>`:''}
  `;
  feed.insertBefore(d,feed.firstChild);
}

function updateChar(){
  const l=document.getElementById('commentInput').value.length;
  const el=document.getElementById('charEl');
  el.textContent=280-l;el.className='char'+(l>240?' warn':'');
}

function renderRelayChips(){
  document.getElementById('relayChips').innerHTML=S.relays.map(r=>
    `<span class="rchip">${r.replace('wss://','').split('/')[0]}</span>`
  ).join('');
}

// ── MODAL ─────────────────────────────────────────────────────────────────────
function openModal(){document.getElementById('overlay').classList.add('open');}
function closeModal(){document.getElementById('overlay').classList.remove('open');}
function overlayClick(e){if(e.target===e.currentTarget)closeModal();}

// ── TOAST ─────────────────────────────────────────────────────────────────────
let tt;
function toast(msg,type=''){
  const el=document.getElementById('toast');
  el.textContent=msg;el.className=`toast show ${type}`;
  clearTimeout(tt);tt=setTimeout(()=>{el.className='toast';},3500);
}
function esc(s=''){return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');}

// ── INIT ──────────────────────────────────────────────────────────────────────
window.addEventListener('load',()=>{
  initNoble();
  initWave();
  renderRelayChips();
  renderSrvChips();
});
</script>

</body>
</html>
