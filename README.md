# t-n-j-cx-c-
nói cái j cx đc=)
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Đàn Piano — Xin chào</title>
  <style>
    :root{
      --bg1: #0f172a;
      --bg2: #0b1220;
      --accent: linear-gradient(90deg,#7c3aed,#06b6d4);
      --glass: rgba(255,255,255,0.06);
    }
    html,body{height:100%;margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial}
    body{
      background: radial-gradient(1200px 600px at 10% 10%, rgba(99,102,241,0.12), transparent),
                  radial-gradient(900px 500px at 90% 90%, rgba(6,182,212,0.08), transparent),
                  linear-gradient(180deg,var(--bg1),var(--bg2));
      color:#e6eef8;
      display:flex;align-items:center;justify-content:center;padding:40px;box-sizing:border-box
    }
    .card{
      width:960px;max-width:96%;background:linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
      border-radius:18px;padding:26px;box-shadow:0 10px 30px rgba(2,6,23,0.6);backdrop-filter: blur(6px);
    }
    .header{display:flex;align-items:center;gap:16px;margin-bottom:18px}
    .logo{
      width:64px;height:64px;border-radius:12px;background:var(--accent);display:flex;align-items:center;justify-content:center;font-weight:700;font-size:22px;color:white;box-shadow:0 6px 20px rgba(124,58,237,0.25);
    }
    h1{font-size:20px;margin:0}
    p.lead{margin:0;color:rgba(230,238,248,0.8)}

    /* Piano area */
    .piano-wrap{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:18px;border-radius:12px}
    .keys{position:relative;height:260px;display:flex;align-items:flex-end;user-select:none}

    .key{position:relative;flex:1;border-radius:6px 6px 8px 8px;margin:0 2px;height:100%;display:flex;align-items:flex-end;justify-content:center;padding-bottom:18px;cursor:pointer;box-shadow:0 6px 16px rgba(2,6,23,0.5);transition:transform .08s ease, box-shadow .08s}
    .key.white{background:linear-gradient(180deg,#ffffff,#f3f4f6);height:220px}
    .key.black{position:absolute;width:48px;height:135px;background:linear-gradient(180deg,#0f172a,#071322);top:0;z-index:2;border-radius:6px;box-shadow:0 10px 18px rgba(2,6,23,0.6)}

    .key-label{font-size:12px;color:#0b1220;font-weight:600;background:rgba(255,255,255,0.9);padding:4px 8px;border-radius:999px}

    .key:active, .key.playing{transform:translateY(6px);box-shadow:0 3px 8px rgba(2,6,23,0.4)}
    .key.white:hover{filter:brightness(0.98)}
    .black.key:active,.black.key.playing{transform:translateY(6px)}

    /* positioning black keys */
    .black.pos-1{left:64px}
    .black.pos-2{left:150px}
    .black.pos-3{left:322px}
    .black.pos-4{left:408px}
    .black.pos-5{left:494px}

    /* Responsive */
    @media (max-width:700px){
      .keys{height:180px}
      .key.white{height:150px}
      .black{height:95px;width:38px}
    }

    /* controls */
    .controls{display:flex;gap:12px;align-items:center;margin-top:14px}
    .btn{background:var(--glass);padding:10px 14px;border-radius:10px;border:1px solid rgba(255,255,255,0.03);cursor:pointer}
    .range{appearance:none;height:6px;border-radius:8px;background:linear-gradient(90deg,#7c3aed,#06b6d4);width:160px}

    .hint{margin-top:12px;color:rgba(230,238,248,0.75);font-size:13px}
  </style>
</head>
<body>
  <div class="card">
    <div class="header">
      <div class="logo">P</div>
      <div>
        <h1>Đàn Piano đẹp — tương tác</h1>
        <p class="lead">Click hoặc dùng phím để chơi. Lưu file thành <code>index.html</code> và đưa lên GitHub Pages.</p>
      </div>
    </div>

    <div class="piano-wrap">
      <div id="piano" class="keys" aria-label="Piano keyboard"></div>

      <div class="controls">
        <div class="btn" id="octaveDown">Octave -</div>
        <div class="btn" id="octaveUp">Octave +</div>
        <label class="btn" style="display:flex;gap:8px;align-items:center">Sustain<input id="sustain" type="checkbox" style="transform:scale(1.1)"></label>
        <div style="margin-left:auto;display:flex;gap:12px;align-items:center">
          <div>Vol</div>
          <input id="volume" class="range" type="range" min="0" max="1" step="0.01" value="0.6">
        </div>
      </div>
      <div class="hint">Phím: Z S X D C V G B H N J M  — tương đương các nốt (C — B). Nhấn và giữ để tạo tiếng dài.</div>
    </div>
  </div>

  <script>
    // NOTE: This script uses Web Audio API — works in modern browsers.
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    let masterGain = audioCtx.createGain(); masterGain.gain.value = 0.6; masterGain.connect(audioCtx.destination);

    const noteFreq = (function(){
      // A4 = 440Hz
      const A4 = 440;
      const A4Index = 57; // MIDI number for A4
      const freqs = {};
      for(let i=0;i<128;i++){
        freqs[i] = Math.pow(2, (i - A4Index)/12) * A4;
      }
      return freqs;
    })();

    // Map UI keys and keyboard letters to MIDI numbers (C4 = 60)
    const whiteNotes = ['C','D','E','F','G','A','B'];
    const keyMap = [
      {name:'C', midi:60, key:'z'}, {name:'C#', midi:61, key:'s'}, {name:'D', midi:62, key:'x'}, {name:'D#', midi:63, key:'d'}, {name:'E', midi:64, key:'c'},
      {name:'F', midi:65, key:'v'}, {name:'F#', midi:66, key:'g'}, {name:'G', midi:67, key:'b'}, {name:'G#', midi:68, key:'h'}, {name:'A', midi:69, key:'n'},
      {name:'A#', midi:70, key:'j'}, {name:'B', midi:71, key:'m'}, {name:'C2', midi:72, key:','}
    ];

    // Build UI
    const piano = document.getElementById('piano');
    const baseOctaveOffset = 0;
    let octaveShift = 0;
    const activeOsc = new Map();
    const sustainCheckbox = document.getElementById('sustain');
    const volumeControl = document.getElementById('volume');
    volumeControl.addEventListener('input', e=>masterGain.gain.value = +e.target.value);

    // Create white keys container (7 white keys * 2 octaves for a nicer look)
    const WHITE_ORDER = ['C','D','E','F','G','A','B'];
    // We'll create 8 white keys across (C to C)
    const whiteCount = 8;
    for(let i=0;i<whiteCount;i++){
      const keyEl = document.createElement('div');
      keyEl.className = 'key white';
      const noteName = WHITE_ORDER[i%7];
      const label = document.createElement('div'); label.className = 'key-label'; label.textContent = noteName;
      keyEl.dataset.note = noteName;
      keyEl.appendChild(label);
      piano.appendChild(keyEl);

      keyEl.addEventListener('mousedown', ()=>noteOnByName(noteName, 0));
      keyEl.addEventListener('mouseup', ()=>noteOffByName(noteName, 0));
      keyEl.addEventListener('mouseleave', ()=>noteOffByName(noteName, 0));
      keyEl.addEventListener('touchstart', e=>{e.preventDefault(); noteOnByName(noteName,0)}, {passive:false});
      keyEl.addEventListener('touchend', e=>{e.preventDefault(); noteOffByName(noteName,0)});
    }

    // Add black keys absolute positioned above whites
    const blacks = [ {pos:1,name:'C#'},{pos:2,name:'D#'},{pos:3,name:'F#'},{pos:4,name:'G#'},{pos:5,name:'A#'} ];
    blacks.forEach(b=>{
      const el = document.createElement('div'); el.className = 'key black pos-'+b.pos; el.dataset.note = b.name; el.textContent = '';
      piano.appendChild(el);
      el.addEventListener('mousedown', ()=>noteOnByName(b.name,0));
      el.addEventListener('mouseup', ()=>noteOffByName(b.name,0));
      el.addEventListener('mouseleave', ()=>noteOffByName(b.name,0));
      el.addEventListener('touchstart', e=>{e.preventDefault(); noteOnByName(b.name,0)}, {passive:false});
      el.addEventListener('touchend', e=>{e.preventDefault(); noteOffByName(b.name,0)});
    });

    function midiFromName(name){
      // Basic mapping relative to C4=60
      const base = {'C':60,'C#':61,'D':62,'D#':63,'E':64,'F':65,'F#':66,'G':67,'G#':68,'A':69,'A#':70,'B':71};
      if(name.endsWith('2')) return 72; // C5
      return base[name] || 60;
    }

    function noteOn(midi){
      if(audioCtx.state === 'suspended') audioCtx.resume();
      if(activeOsc.has(midi)) return; // already playing
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = 'sine';
      osc.frequency.value = noteFreq[midi];
      gain.gain.value = 0;
      // Attack
      gain.gain.linearRampToValueAtTime(1, audioCtx.currentTime + 0.02);
      // connect through master
      osc.connect(gain); gain.connect(masterGain);
      osc.start();
      activeOsc.set(midi, {osc,gain});

      // visual
      const keyEls = [...document.querySelectorAll('[data-note]')].filter(k=>midiFromName(k.dataset.note) === midi || (k.dataset.note + '2' && midi===72 && k.dataset.note==='C'));
      keyEls.forEach(el=>el.classList.add('playing'));
    }

    function noteOff(midi){
      if(!activeOsc.has(midi)) return;
      const {osc,gain} = activeOsc.get(midi);
      if(sustainCheckbox.checked){
        // do not stop yet
        gain.gain.cancelScheduledValues(audioCtx.currentTime);
        gain.gain.setValueAtTime(gain.gain.value, audioCtx.currentTime);
        // keep until sustain unchecked
        // store as sustained
        activeOsc.set(midi, {osc,gain, sustained:true});
        return;
      }
      // Release
      gain.gain.linearRampToValueAtTime(0.0001, audioCtx.currentTime + 0.5);
      setTimeout(()=>{
        try{osc.stop()}catch(e){}
        osc.disconnect(); gain.disconnect();
      },600);
      activeOsc.delete(midi);
      const keyEls = [...document.querySelectorAll('[data-note]')].filter(k=>midiFromName(k.dataset.note) === midi || (k.dataset.note + '2' && midi===72 && k.dataset.note==='C'));
      keyEls.forEach(el=>el.classList.remove('playing'));
    }

    function noteOnByName(name, offsetOct){
      const baseMidi = midiFromName(name);
      const midi = baseMidi + octaveShift*12;
      noteOn(midi);
    }
    function noteOffByName(name, offsetOct){
      const baseMidi = midiFromName(name);
      const midi = baseMidi + octaveShift*12;
      noteOff(midi);
    }

    // Keyboard handling
    const keyLookup = {};
    keyMap.forEach(k=>{ keyLookup[k.key] = k.midi; });

    window.addEventListener('keydown', e=>{
      if(e.repeat) return;
      const k = e.key.toLowerCase();
      if(k in keyLookup){
        const midi = keyLookup[k] + octaveShift*12 - 60 + 60; // just add octave shift
        noteOn(keyLookup[k] + octaveShift*12);
      }
    });
    window.addEventListener('keyup', e=>{
      const k = e.key.toLowerCase();
      if(k in keyLookup){ noteOff(keyLookup[k] + octaveShift*12); }
    });

    // Octave controls
    document.getElementById('octaveUp').addEventListener('click', ()=>{ octaveShift = Math.min(3, octaveShift+1);} );
    document.getElementById('octaveDown').addEventListener('click', ()=>{ octaveShift = Math.max(-2, octaveShift-1);} );

    // Sustain toggle: when unchecked, release all sustained notes
    sustainCheckbox.addEventListener('change', ()=>{
      if(!sustainCheckbox.checked){
        // release all sustained
        for(const [m, obj] of Array.from(activeOsc.entries())){
          if(obj.sustained){
            noteOff(m);
          }
        }
      }
    });

    // allow clicking anywhere to resume audio context on certain mobile browsers
    document.body.addEventListener('touchstart', function resume(){ if(audioCtx.state === 'suspended') audioCtx.resume(); document.body.removeEventListener('touchstart', resume); }, {passive:true});

  </script>
</body>
</html>
