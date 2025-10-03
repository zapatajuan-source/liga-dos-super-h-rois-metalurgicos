<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Machine Tamagotchi — LINGOTINHO (2P Pixel)</title>
  <style>
    :root{--bg:#000;--fg:#0f0;--accent:#0b6;--glass:rgba(15,255,15,0.06);--panel:#083;}
    html,body{height:100%;margin:0;background:var(--bg);color:var(--fg);font-family: 'Courier New',monospace}
    .wrap{display:flex;gap:18px;align-items:flex-start;justify-content:center;padding:20px;box-sizing:border-box}
    .device{width:380px;border:6px solid var(--fg);padding:12px;background:linear-gradient(180deg,#001200,#000);box-shadow:0 12px 30px rgba(0,255,0,0.03);border-radius:10px}
    .screen{background:var(--bg);height:520px;width:340px;padding:12px;box-sizing:border-box;outline:4px solid var(--fg);border-radius:8px;display:flex;flex-direction:column;justify-content:space-between}
    .topbar{display:flex;align-items:center;justify-content:space-between}
    .title{letter-spacing:1px;font-weight:700}

    .main{display:flex;gap:12px}
    .viewport{width:200px;height:260px;background:var(--glass);border:2px solid var(--fg);padding:8px;box-sizing:border-box;display:flex;align-items:center;justify-content:center;position:relative}
    .pixel-art{image-rendering:pixelated; width:192px;height:192px;}
    .pulse{animation:pulse 1.6s infinite}
    @keyframes pulse{0%{filter:brightness(1)}50%{filter:brightness(1.15)}100%{filter:brightness(1)}}

    .panel{flex:1;display:flex;flex-direction:column;gap:8px}
    .meters{display:flex;flex-direction:column;gap:8px}
    .meter{display:flex;align-items:center;gap:8px}
    .meter .bar{flex:1;height:14px;background:rgba(0,255,0,0.06);border:1px solid rgba(0,255,0,0.18);position:relative;overflow:hidden}
    .meter .fill{height:100%;background:linear-gradient(90deg,var(--fg),var(--accent));width:50%;transition:width 500ms ease}
    .meter .label{width:80px;text-align:right;font-size:13px}

    .actions{display:flex;gap:8px;flex-wrap:wrap}
    .btn{background:transparent;border:2px solid var(--fg);padding:6px 8px;border-radius:6px;cursor:pointer;min-width:80px;text-align:center}
    .btn:active{transform:translateY(1px)}

    .sidebar{width:300px;border:4px solid var(--fg);padding:12px;border-radius:8px}
    .list{display:grid;grid-template-columns:1fr 1fr;gap:8px}
    .card{padding:8px;border:2px solid rgba(0,255,0,0.12);background:rgba(0,0,0,0.2);cursor:pointer;border-radius:6px}
    .card.active{outline:3px solid var(--accent)}

    .log{height:90px;overflow:auto;background:#001200;border:1px solid var(--fg);padding:6px;font-size:12px}
    .footer{display:flex;gap:6px;align-items:center;justify-content:space-between}

    /* particle confetti */
    .particles{position:absolute;left:0;top:0;right:0;bottom:0;pointer-events:none}

    /* tooltip */
    [data-tip]{position:relative}
    [data-tip]:hover::after{content:attr(data-tip);position:absolute;left:0;bottom:calc(100% + 8px);background:#001200;border:1px solid var(--fg);padding:6px;border-radius:4px;font-size:12px}

    /* small helpers */
    .muted{opacity:0.9;font-size:13px}
  </style>
</head>
<body>
  <div class="wrap">
    <aside class="sidebar">
      <h3>Máquinas & Bichos</h3>
      <div style="margin:8px 0;display:flex;gap:8px;align-items:center">
        <select id="filterType"><option value="all">Todas</option><option value="server">Servidor</option><option value="robot">Robô</option><option value="printer">Impressora</option></select>
        <input id="searchBox" type="text" placeholder="Buscar..." />
      </div>
      <div class="list" id="machineList"></div>

      <hr style="border-color:rgba(0,255,0,0.06);margin:10px 0" />
      <h4>Seu Bicho — LINGOTINHO</h4>
      <div style="display:flex;gap:8px;align-items:center;margin-top:6px">
        <select id="petSelect"></select>
        <button class="btn" id="uploadSpriteBtn">Carregar Sprite</button>
      </div>
      <div style="margin-top:10px;font-size:12px">Salvar / Exportar</div>
      <div style="display:flex;gap:6px;margin-top:6px">
        <button class="btn" id="saveBtn">Salvar</button>
        <button class="btn" id="exportBtn">Exportar JSON</button>
      </div>
      <div style="margin-top:12px;font-size:12px" class="muted">Dica: pode colar JSON do estado no clipboard para importar.</div>
    </aside>

    <div class="device">
      <div class="screen">
        <div class="topbar">
          <div>
            <div class="title">MACHINE TAMAGOTCHI — LINGOTINHO</div>
            <div class="muted">2P Pixel — Versão Profissional</div>
          </div>
          <div class="muted">Tempo: <span id="timeLabel">00:00</span></div>
        </div>

        <div class="main">
          <div style="display:flex;flex-direction:column;align-items:center;gap:8px">
            <div class="viewport" id="viewport">
              <canvas id="pixelCanvas" class="pixel-art" width="32" height="32"></canvas>
              <div class="particles" id="particles"></div>
            </div>
            <div style="display:flex;gap:6px;justify-content:center">
              <button class="btn" id="feedBtn">Carregar</button>
              <button class="btn" id="coolBtn">Resfriar</button>
              <button class="btn" id="repairBtn">Consertar</button>
            </div>
          </div>

          <div class="panel">
            <div class="meters">
              <div class="meter"><div class="label">Energia</div><div class="bar"><div id="eFill" class="fill"></div></div></div>
              <div class="meter"><div class="label">Temp (°C)</div><div class="bar"><div id="tFill" class="fill"></div></div></div>
              <div class="meter"><div class="label">Desgaste</div><div class="bar"><div id="wFill" class="fill"></div></div></div>
              <div class="meter"><div class="label">Limpeza</div><div class="bar"><div id="cFill" class="fill"></div></div></div>
            </div>

            <div style="display:flex;justify-content:space-between;align-items:center">
              <div style="display:flex;gap:8px;align-items:center"><div class="muted">Velocidade:</div><select id="speedSelect"><option value="1">Normal</option><option value="0.5">Lento</option><option value="2">Rápido</option></select></div>
              <div class="muted">Estado: <span id="stateLabel">OK</span></div>
            </div>

            <div class="actions">
              <button class="btn" id="toggleAuto">Auto: ON</button>
              <button class="btn" id="renameBtn">Renomear Máquina</button>
              <button class="btn" id="addMachineBtn">+ Criar</button>
              <button class="btn" id="removeMachineBtn">Remover</button>
            </div>

            <div class="log" id="log"></div>
          </div>
        </div>

        <div class="footer">
          <div class="muted">Salvo automaticamente no navegador (localStorage)</div>
          <div>
            <button class="btn" id="fullscreenBtn">Tela Cheia</button>
            <button class="btn" id="helpBtn">Ajuda</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <input id="spriteFile" type="file" accept="image/*" style="display:none" />

  <script>
    // Utilities
    const $ = id => document.getElementById(id);
    const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));
    const nowStr = ()=> new Date().toLocaleTimeString();
    const rand = (a,b)=> Math.floor(Math.random()*(b-a+1))+a;

    // Default machines
    const DEFAULT_MACHINES = () => ([
      {id:uid(),name:'Servidor A',type:'server',energy:100,temp:40,wear:5,clean:90,pet:'lingotinho'},
      {id:uid(),name:'Robô-01',type:'robot',energy:88,temp:48,wear:18,clean:70,pet:'lingotinho'},
      {id:uid(),name:'Impressora',type:'printer',energy:76,temp:60,wear:30,clean:50,pet:'lingotinho'}
    ]);

    // Built-in LINGOTINHO 32x32 pixel sprite (simple approximation)
    const LINGOTINHO_SPRITE = (function(){
      const g = Array.from({length:32},()=>Array(32).fill(0));
      // helmet head body simplified
      for(let y=6;y<26;y++) for(let x=8;x<24;x++) g[y][x]=1;
      for(let y=10;y<22;y++) for(let x=10;x<22;x++) if((x-16)*(x-16)+(y-16)*(y-16)<64) g[y][x]=1;
      // eyes/mouth carve
      g[14][13]=0; g[14][18]=0; g[18][16]=0;
      return g;
    })();

    const PETS = {
      'lingotinho':{name:'Lingotinho',sprite:LINGOTINHO_SPRITE},
      'bot':{name:'Bot',sprite:makeBotSprite()},
      'cog':{name:'Cog',sprite:makeCogSprite()}
    };

    let state = {machines:DEFAULT_MACHINES(),activeIndex:0,auto:true,speed:1,lastTick:Date.now()};
    const STORAGE_KEY = 'mt_2p_v2';

    function saveState(){ localStorage.setItem(STORAGE_KEY,JSON.stringify(state)); log('Salvo'); }
    function loadState(){ const s=localStorage.getItem(STORAGE_KEY); if(!s) return false; try{ state=JSON.parse(s); return true;}catch(e){return false} }

    function uid(){return 'm'+Math.random().toString(36).slice(2,9)}

    // UI refs
    const machineListEl = $('machineList'); const petSelect = $('petSelect'); const pixelCanvas = $('pixelCanvas'); const ctx = pixelCanvas.getContext('2d');
    const eFill = $('eFill'), tFill = $('tFill'), wFill = $('wFill'), cFill = $('cFill'); const timeLabel = $('timeLabel'); const logEl = $('log'); const particlesEl = $('particles');

    function populatePetSelect(){ petSelect.innerHTML=''; Object.keys(PETS).forEach(k=>{ const o=document.createElement('option'); o.value=k; o.textContent=PETS[k].name; petSelect.appendChild(o); }); }

    function renderMachineList(filter='all',search=''){
      machineListEl.innerHTML=''; state.machines.forEach((m,i)=>{
        if(filter!=='all' && m.type!==filter) return; if(search && !m.name.toLowerCase().includes(search.toLowerCase())) return;
        const card=document.createElement('div'); card.className='card'+(i===state.activeIndex?' active':''); card.dataset.index=i;
        card.innerHTML=`<h4>${m.name}</h4><p>${m.type} • E:${Math.round(m.energy)} T:${Math.round(m.temp)} W:${Math.round(m.wear)}</p>`;
        card.onclick=()=>{ state.activeIndex=i; renderUI(); playSound('select'); }
        machineListEl.appendChild(card);
      });
    }

    function renderUI(){ const m = state.machines[state.activeIndex]; if(!m) return;
      eFill.style.width = m.energy+'%'; tFill.style.width = clamp((m.temp/120)*100,0,100)+'%'; wFill.style.width = m.wear+'%'; cFill.style.width = m.clean+'%';
      $('stateLabel').textContent = (m.energy<20||m.temp>90||m.wear>80)?'CRÍTICO':'OK'; petSelect.value = m.pet || 'lingotinho'; drawSprite(PETS[m.pet]?.sprite || PETS['lingotinho'].sprite,m);
      renderMachineList($('filterType').value,$('searchBox').value); timeLabel.textContent = nowStr(); }

    function log(txt){ const line=document.createElement('div'); line.textContent=`[${nowStr()}] ${txt}`; logEl.prepend(line); if(logEl.childNodes.length>200) logEl.removeChild(logEl.lastChild); }

    // Actions
    function actionFeed(){ interact('feed'); }
    function actionCool(){ interact('cool'); }
    function actionRepair(){ interact('repair'); }

    function interact(action){ const m = state.machines[state.activeIndex]; if(!m) return;
      if(action==='feed'){ m.energy = clamp(m.energy + 26,0,100); m.clean = clamp(m.clean - 6,0,100); log(`${m.name} carregado (+26 energia)`); playSound('feed'); spawnParticles('green'); }
      if(action==='cool'){ m.temp = clamp(m.temp - 20,0,120); m.wear = clamp(m.wear + 3,0,100); log(`${m.name} resfriado (-20°C)`); playSound('cool'); spawnParticles('blue'); }
      if(action==='repair'){ m.wear = clamp(m.wear - 30,0,100); m.clean = clamp(m.clean - 8,0,100); log(`${m.name} consertado (-30 desgaste)`); playSound('repair'); spawnConfetti(); }
      renderUI(); saveState(); }

    // Simulation tick
    function tick(dtSec){ state.machines.forEach(m=>{
      const typeFactor = m.type==='server'?1.2:(m.type==='robot'?1.0:0.9);
      m.energy = clamp(m.energy - (0.6*typeFactor*dtSec*state.speed),0,100);
      m.temp = clamp(m.temp + (0.25*typeFactor*dtSec*state.speed) + (100-m.clean)*0.004,0,130);
      m.wear = clamp(m.wear + (0.06*typeFactor*dtSec*state.speed) + (m.temp>80?0.22:0),0,100);
      m.clean = clamp(m.clean - (0.04*dtSec*state.speed),0,100);
      if(m.energy<=5 && Math.random()<0.01*dtSec) log(`${m.name} entrando em hibernação por falta de energia`);
      if(m.temp>=110 && Math.random()<0.02*dtSec) log(`${m.name} superaquecendo! risco de falha.`);
      if(m.wear>=95 && Math.random()<0.012*dtSec) log(`${m.name} falha mecânica! precisa de conserto.`);
    }); renderUI(); }

    // Pixel drawing
    function drawSprite(sprite,m){ ctx.clearRect(0,0,32,32); const imgData=ctx.createImageData(32,32);
      for(let y=0;y<32;y++) for(let x=0;x<32;x++){ const i=(y*32+x); const v = sprite[y] && sprite[y][x] ? 1 : 0; const p=i*4; if(v){ imgData.data[p]=30; imgData.data[p+1]=220; imgData.data[p+2]=20; imgData.data[p+3]=255; } else { imgData.data[p]=0; imgData.data[p+1]=0; imgData.data[p+2]=0; imgData.data[p+3]=0; } }
      ctx.putImageData(imgData,0,0); pixelCanvas.style.width='192px'; pixelCanvas.style.height='192px';
      // small mood animation: shake when critical
      if(m.energy<20||m.temp>100||m.wear>85){ pixelCanvas.style.transform='translateX(0)'; pixelCanvas.animate([{transform:'translateX(-4px)'},{transform:'translateX(4px)'},{transform:'translateX(0)'}],{duration:400,iterations:1}); }
    }

    // Sprites generators
    function makeCogSprite(){ const g=Array.from({length:32},()=>Array(32).fill(0)); for(let y=8;y<24;y++) for(let x=8;x<24;x++){ const dx=x-16,dy=y-16; if(dx*dx+dy*dy<64) g[y][x]=1; } for(let i=0;i<8;i++){g[7][12+i]=1; g[24][12+i]=1; g[12+i][7]=1; g[12+i][24]=1} return g; }
    function makeBotSprite(){ const g=Array.from({length:32},()=>Array(32).fill(0)); for(let y=6;y<26;y++) for(let x=10;x<22;x++) g[y][x]=1; g[12][13]=0; g[12][18]=0; return g; }

    // Particles
    function spawnParticles(color){ const count=12; for(let i=0;i<count;i++){ const el=document.createElement('div'); el.style.position='absolute'; el.style.left=(96)+'px'; el.style.top=(96)+'px'; el.style.width='6px'; el.style.height='6px'; el.style.background = color==='blue' ? 'rgba(50,150,255,0.9)' : 'rgba(50,255,100,0.95)'; el.style.borderRadius='2px'; particlesEl.appendChild(el); const dx=(Math.random()-0.5)*160; const dy=(Math.random()-0.5)*160; el.animate([{transform:'translate(0,0)',opacity:1},{transform:`translate(${dx}px,${dy}px)`,opacity:0}],{duration:700,fill:'forwards'}).onfinish=()=>el.remove(); } }

    function spawnConfetti(){ const colors=['#ffd400','#ff6b6b','#6bffb8','#6b9bff']; for(let i=0;i<30;i++){ const el=document.createElement('div'); el.style.position='absolute'; el.style.left='96px'; el.style.top='96px'; el.style.width='6px'; el.style.height='8px'; el.style.background = colors[Math.floor(Math.random()*colors.length)]; el.style.borderRadius='2px'; particlesEl.appendChild(el); const dx=(Math.random()-0.5)*300; const dy=(Math.random()-1)*300; const rot= Math.random()*720; el.animate([{transform:'translate(0,0) rotate(0deg)',opacity:1},{transform:`translate(${dx}px,${dy}px) rotate(${rot}deg)`,opacity:0}],{duration:1200,fill:'forwards'}).onfinish=()=>el.remove(); } }

    // Sounds: WebAudio simple synth
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playTone(freq,duration, type='sine'){ const now=audioCtx.currentTime; const o=audioCtx.createOscillator(); const g=audioCtx.createGain(); o.type=type; o.frequency.value=freq; g.gain.value=0.0001; o.connect(g); g.connect(audioCtx.destination); o.start(now); g.gain.exponentialRampToValueAtTime(0.08, now+0.01); g.gain.exponentialRampToValueAtTime(0.0001, now+duration);
      o.stop(now+duration+0.02);
    }
    function playSound(kind){ if(!audioCtx) return; try{ if(audioCtx.state==='suspended') audioCtx.resume(); }catch(e){}
      if(kind==='feed'){ playTone(440,0.18,'sawtooth'); playTone(660,0.12,'square'); }
      if(kind==='cool'){ playTone(300,0.22,'sine'); playTone(220,0.12,'sine'); }
      if(kind==='repair'){ playTone(880,0.26,'triangle'); playTone(660,0.14,'sine'); }
      if(kind==='select'){ playTone(660,0.08,'sine'); }
    }

    // Create/remove machines
    function createMachine(name,type){ const m={id:uid(),name:name||('Máquina '+(state.machines.length+1)),type:type||'server',energy:rand(60,100),temp:rand(30,60),wear:rand(0,30),clean:rand(50,100),pet:'lingotinho'}; state.machines.push(m); state.activeIndex=state.machines.length-1; renderUI(); saveState(); log('Criada '+m.name); }
    function removeActiveMachine(){ const m=state.machines[state.activeIndex]; if(!m) return; if(!confirm('Remover '+m.name+'?')) return; state.machines.splice(state.activeIndex,1); state.activeIndex=Math.max(0,state.activeIndex-1); renderUI(); saveState(); log('Removida '+m.name); }

    // Wiring
    $('feedBtn').onclick=()=>{ actionFeed(); };
    $('coolBtn').onclick=()=>{ actionCool(); };
    $('repairBtn').onclick=()=>{ actionRepair(); };
    $('toggleAuto').onclick=()=>{ state.auto=!state.auto; $('toggleAuto').textContent='Auto: '+(state.auto?'ON':'OFF'); saveState(); };
    $('speedSelect').onchange=()=>{ state.speed=parseFloat($('speedSelect').value); saveState(); };
    $('addMachineBtn').onclick=()=>{ const name=prompt('Nome da máquina:','Máquina Nova'); const type=prompt('Tipo (server/robot/printer):','server'); if(name) createMachine(name,type); };
    $('removeMachineBtn').onclick=removeActiveMachine;
    $('saveBtn').onclick=saveState;
    $('exportBtn').onclick=()=>{ const blob=new Blob([JSON.stringify(state,null,2)],{type:'application/json'}); const url=URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='machine_tamagotchi_export.json'; a.click(); URL.revokeObjectURL(url); };
    $('renameBtn').onclick=()=>{ const m=state.machines[state.activeIndex]; if(!m) return; const nn=prompt('Novo nome:',m.name); if(nn){ m.name=nn; renderUI(); saveState(); } };
    petSelect.onchange=()=>{ const m=state.machines[state.activeIndex]; if(m){ m.pet=petSelect.value; renderUI(); saveState(); } };
    $('filterType').onchange=()=>renderMachineList($('filterType').value,$('searchBox').value);
    $('searchBox').oninput=()=>renderMachineList($('filterType').value,$('searchBox').value);
    $('fullscreenBtn').onclick=()=>{ if(!document.fullscreenElement) document.documentElement.requestFullscreen(); else document.exitFullscreen(); };
    $('helpBtn').onclick=()=>{ alert('Controles:
1: Carregar  2: Resfriar  3: Consertar
Salvar/Export: estado em JSON.
Você pode carregar um sprite PNG 32x32 para substituir Lingotinho.'); };

    // Upload sprite to replace Lingotinho
    $('uploadSpriteBtn').onclick = ()=>{ $('spriteFile').click(); };
    $('spriteFile').onchange = async (e)=>{
      const f = e.target.files[0]; if(!f) return; const img = new Image(); img.onload = ()=>{
        // draw into temp canvas scaled to 32x32 and read pixels
        const tmp = document.createElement('canvas'); tmp.width=32; tmp.height=32; const tctx=tmp.getContext('2d'); tctx.drawImage(img,0,0,32,32); const d = tctx.getImageData(0,0,32,32).data; const g = Array.from({length:32},()=>Array(32).fill(0)); for(let y=0;y<32;y++) for(let x=0;x<32;x++){ const i=(y*32+x)*4; const alpha=d[i+3]; const lum=(d[i]+d[i+1]+d[i+2])/3; if(alpha>60 && lum>20) g[y][x]=1; }
        PETS['lingotinho'].sprite = g; renderUI(); saveState(); log('Sprite de Lingotinho atualizado.'); playSound('repair');
      };
      const url=URL.createObjectURL(f); img.src=url;
    };

    // Keyboard shortcuts
    window.addEventListener('keydown',e=>{ if(e.key==='1') actionFeed(); if(e.key==='2') actionCool(); if(e.key==='3') actionRepair(); if(e.key==='n') $('addMachineBtn').click(); });

    // Load or init
    if(!loadState()){ saveState(); }
    populatePetSelect(); renderUI();

    // Main loop
    let last = performance.now(); function loop(t){ const dt=(t-last)/1000; last=t; if(state.auto) tick(dt*1); if(Date.now()-state.lastTick>15000){ saveState(); state.lastTick=Date.now(); } requestAnimationFrame(loop); } requestAnimationFrame(loop);

    // Paste import
    window.addEventListener('paste',async(ev)=>{ const text = ev.clipboardData.getData('text'); if(!text) return; try{ const obj=JSON.parse(text); if(obj.machines){ state=obj; saveState(); renderUI(); alert('Importado do clipboard!'); } }catch(e){} });

    // Tiny helpers end
  </script>
</body>
</html>

