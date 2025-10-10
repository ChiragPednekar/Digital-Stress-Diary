<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Digital Stress Diary</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
  <style>
    :root {
      /* CHANGED: Updated background color to a soft mint green */
      --bg: #f0fff4; /* A very light, calming green */
      --card: #ffffff;
      --muted: #6b7280;
      --accent: #4f46e5;

      /* New 7-level Mood Palette */
      --mood-7: #059669; /* Emerald - Excellent */
      --mood-6: #10b981; /* Green - Great */
      --mood-5: #3b82f6; /* Blue - Good */
      --mood-4: #f59e0b; /* Amber - Okay */
      --mood-3: #f97316; /* Orange - Subpar */
      --mood-2: #ef4444; /* Red - Bad */
      --mood-1: #b91c1c; /* Dark Red - Very Bad */
    }
    * { box-sizing: border-box; }
    body { font-family: 'Inter', sans-serif; margin: 0; background: var(--bg); color: #0f172a; }
    /* CHANGED: Updated header background gradient color to complement the new --bg */
    header { background: linear-gradient(90deg, #e6ffe6, transparent); padding: 20px 16px; border-bottom: 1px solid rgba(15,23,42,0.04); }
    header h1 { margin: 0; font-size: 24px; }
    .container { max-width: 1100px; margin: 20px auto; padding: 0 16px; }
    .grid { display: grid; grid-template-columns: 1fr 420px; gap: 20px; }
    @media (max-width: 900px) { .grid { grid-template-columns: 1fr; } }
    .card { background: var(--card); border-radius: 12px; padding: 20px; box-shadow: 0 6px 20px rgba(2,6,23,0.04); }
    .mood-row { display: flex; gap: 8px; flex-wrap: wrap; margin: 8px 0; }
    /* Updated button size and styling for the new mood count */
    .mood-btn { border: 1px solid rgba(15,23,42,0.06); background: transparent; padding: 10px 12px; font-size: 18px; border-radius: 12px; cursor: pointer; transition: transform 0.15s, box-shadow 0.15s, background-color 0.2s; white-space: nowrap;}
    .mood-btn:hover { transform: translateY(-2px); box-shadow: 0 6px 16px rgba(79,70,229,0.12); }
    .mood-btn.selected { background: var(--accent); color: #fff; border-color: var(--accent); }
    textarea { width: 100%; min-height: 90px; padding: 12px; border-radius: 10px; border: 1px solid rgba(15,23,42,0.06); resize: vertical; }
    .actions { display: flex; gap: 8px; flex-wrap: wrap; margin-top: 16px; }
    .btn { padding: 10px 14px; border-radius: 10px; border: none; cursor: pointer; font-weight: 600; transition: all 0.2s; }
    .btn-primary { background: var(--accent); color: white; }
    .btn-primary:hover { background: #4338ca; }
    .btn-ghost { background: transparent; border: 1px solid rgba(15,23,42,0.06); }
    .btn-ghost:hover { background: #f0f4f8; }
    .btn-clear { background: #fee2e2; border: 1px solid #fca5a5; color: #b91c1c; }
    .btn-clear:hover { background: #fecaca; }
    .muted { color: var(--muted); font-size: 13px; }
    /* Changed background of journal entries to match the new overall theme better */
    .entry { display: flex; gap: 12px; align-items: flex-start; padding: 14px; border-radius: 12px; border: 1px solid rgba(15,23,42,0.03); margin-bottom: 10px; background: #ffffff; transition: transform 0.15s; }
    .entry:hover { transform: translateY(-2px); box-shadow: 0 4px 10px rgba(2,6,23,0.04); }
    .entry .emoji { font-size: 26px; line-height: 1; }
    .entry .meta { flex: 1; }
    .entry .date { font-size: 12px; color: var(--muted); }
    .entry .note { margin-top: 6px; white-space: pre-wrap; }
    .top-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
    footer { padding: 14px 16px; text-align: center; color: var(--muted); font-size: 13px; }
  </style>
</head>
<body>
  <header>
    <div class="container">
      <h1>Digital Stress Diary 📝</h1>
      <p class="muted">Track your mood daily and visualize your stress patterns.</p>
    </div>
  </header>

  <main class="container">
    <div class="grid">
      <section>
        <div class="card">
          <div class="top-row">
            <strong>New Mood Entry ✨</strong>
            <div class="muted" id="todayDate"></div>
          </div>
          <div>
            <label class="muted">Choose mood level (1=Worst, 7=Best)</label>
            <div class="mood-row" id="moodButtons"></div>
            <div style="margin-top:10px">
              <label class="muted">Or choose from dropdown</label>
              <select id="moodSelect" style="padding:10px;border-radius:10px;border:1px solid rgba(15,23,42,0.06);width:100%">
                <option value="">-- select mood --</option>
              </select>
            </div>
            <div style="margin-top:14px">
              <label class="muted">Notes (optional)</label>
              <textarea id="noteInput" placeholder="Describe what's going on..."></textarea>
            </div>
            <div class="actions">
              <button id="saveBtn" class="btn btn-primary">Save Entry</button>
              <button id="exportJsonBtn" class="btn btn-ghost">Export JSON</button>
              <button id="exportCsvBtn" class="btn btn-ghost">Export CSV</button>
              <button id="clearBtn" class="btn btn-clear">Clear All Entries</button>
            </div>
          </div>
        </div>

        <div class="card" style="margin-top:16px">
          <div class="top-row"><strong>Mood Journal 📔</strong><span class="muted" id="countBadge"></span></div>
          <div id="journalList"></div>
        </div>
      </section>

      <aside>
        <div class="card">
          <div class="top-row">
            <strong>Mood Chart 📈</strong>
            <div>
              <label class="muted">Type</label>
              <select id="chartType" style="padding:6px;border-radius:8px;border:1px solid rgba(15,23,42,0.06)">
                <option value="line">Trend (line)</option>
                <option value="bar">Counts (bar)</option>
              </select>
            </div>
          </div>
          <canvas id="moodChart" height="280"></canvas>
          <div style="margin-top:12px" class="muted">Legend: <span id="legendInline"></span></div>
        </div>
      </aside>
    </div>
  </main>

  <footer>
    Your entries are stored locally on this device. Use Export to back up your data.
  </footer>

  <script>
    // Updated MOODS array for 7 levels (1=Worst, 7=Best)
    const MOODS = [
  {emoji:'🤩', label:'Euphoric', score:10, color:'#047857'},  // deep green
  {emoji:'😄', label:'Joyful', score:9, color:'#10b981'},
  {emoji:'😊', label:'Happy', score:8, color:'#34d399'},
  {emoji:'😌', label:'Calm', score:7, color:'#60a5fa'},
  {emoji:'🙂', label:'Content', score:6, color:'#93c5fd'},
  {emoji:'😐', label:'Neutral', score:5, color:'#facc15'},
  {emoji:'😕', label:'Tired', score:4, color:'#f59e0b'},
  {emoji:'😟', label:'Anxious', score:3, color:'#f97316'},
  {emoji:'😫', label:'Frustrated', score:2, color:'#ef4444'},
  {emoji:'😭', label:'Depressed', score:1, color:'#b91c1c'}
].reverse(); // Reverse so UI goes from 10→1

    const STORAGE_KEY = 'stress_diary_entries_v2';

    const moodButtonsEl = document.getElementById('moodButtons');
    const moodSelectEl = document.getElementById('moodSelect');
    const noteInput = document.getElementById('noteInput');
    const saveBtn = document.getElementById('saveBtn');
    const journalList = document.getElementById('journalList');
    const countBadge = document.getElementById('countBadge');
    const todayDate = document.getElementById('todayDate');
    const clearBtn = document.getElementById('clearBtn');
    const exportJsonBtn = document.getElementById('exportJsonBtn');
    const exportCsvBtn = document.getElementById('exportCsvBtn');
    const chartTypeEl = document.getElementById('chartType');
    const legendInline = document.getElementById('legendInline');

    function initUI(){
      todayDate.textContent = new Date().toLocaleDateString();
      MOODS.forEach(m=>{
        const btn = document.createElement('button');
        // Include the score in the button for clarity (e.g., "7 🤩 Excellent")
        btn.className='mood-btn'; btn.textContent=`${m.score} ${m.emoji} ${m.label}`;
        btn.dataset.emoji=m.emoji; btn.dataset.score=m.score;
        btn.addEventListener('click',()=>selectMood(m.emoji));
        moodButtonsEl.appendChild(btn);
        
        const opt = document.createElement('option'); opt.value=m.emoji; opt.textContent=`${m.score} ${m.emoji} ${m.label}`;
        moodSelectEl.appendChild(opt);
      });
      moodSelectEl.addEventListener('change',()=>selectMood(moodSelectEl.value,{fromSelect:true}));
      // Updated legend to show all 7 options
      legendInline.innerHTML = MOODS.slice().reverse().map(m=>`${m.score}: ${m.emoji} ${m.label}`).join(' · ');
    }

    function selectMood(emoji,opts={}){
      document.querySelectorAll('.mood-btn').forEach(b=>b.classList.remove('selected'));
      if(!emoji){ moodSelectEl.value=''; return; }
      const btn=Array.from(document.querySelectorAll('.mood-btn')).find(b=>b.dataset.emoji===emoji);
      if(btn) btn.classList.add('selected');
      if(!opts.fromSelect) moodSelectEl.value=emoji;
    }

    function loadEntries(){ try{ const raw=localStorage.getItem(STORAGE_KEY); return raw?JSON.parse(raw):[] }catch(e){return[]}}
    function saveEntries(list){ localStorage.setItem(STORAGE_KEY,JSON.stringify(list)); }

    function addEntry(entry){ const list=loadEntries(); list.push(entry); saveEntries(list); renderJournal(); updateChart(); }

    function renderJournal(){
      const list=loadEntries(); journalList.innerHTML='';
      if(list.length===0){ journalList.innerHTML='<div class="muted">No entries yet — add one above.</div>'; countBadge.textContent=''; return; }
      countBadge.textContent=`${list.length} entries`;
      list.slice().sort((a,b)=>new Date(b.date)-new Date(a.date)).forEach(item=>{
        const wrap=document.createElement('div'); wrap.className='entry';
        const emoji=document.createElement('div'); emoji.className='emoji'; emoji.textContent=item.mood;
        const meta=document.createElement('div'); meta.className='meta';
        const d=document.createElement('div'); d.className='date'; d.textContent=new Date(item.date).toLocaleString();
        const note=document.createElement('div'); note.className='note'; note.textContent=item.note||'';
        meta.appendChild(d); meta.appendChild(note); wrap.appendChild(emoji); wrap.appendChild(meta);
        journalList.appendChild(wrap);
      });
    }

    clearBtn.addEventListener('click',()=>{if(confirm('Clear ALL saved entries? This action cannot be undone.')){localStorage.removeItem(STORAGE_KEY); renderJournal(); updateChart();}});

    saveBtn.addEventListener('click',()=>{
      const selected=document.querySelector('.mood-btn.selected');
      const mood=(selected && selected.dataset.emoji) || moodSelectEl.value;
      if(!mood){ alert('Please select a mood.'); return; }
      const note=noteInput.value.trim();
      addEntry({date:new Date().toISOString(),mood,note});
      selectMood(''); noteInput.value=''; moodSelectEl.value='';
    });

    let chart=null;
    // Updated function to get score from the expanded MOODS array
    function moodToScore(emoji){ const found=MOODS.find(m=>m.emoji===emoji); return found?found.score:0; }

    function updateChart(){
      const entries=loadEntries(); const type=chartTypeEl.value;
      // Max score is now 7
      const MAX_SCORE = 7; 
      
      if(type==='line'){
        const sorted=entries.slice().sort((a,b)=>new Date(a.date)-new Date(b.date));
        const labels=sorted.map(e=>new Date(e.date).toLocaleDateString());
        const data=sorted.map(e=>moodToScore(e.mood));
        
        // Use a consistent line color for the trend chart, like the accent color
        const cfg={ 
            type:'line', 
            data:{ 
                labels,
                datasets:[{
                    label:'Mood score',
                    data,
                    tension:0.3,
                    fill:false,
                    borderWidth:3,
                    pointRadius:6,
                    // Use the accent color for the line and points
                    borderColor: getComputedStyle(document.documentElement).getPropertyValue('--accent'), 
                    backgroundColor: getComputedStyle(document.documentElement).getPropertyValue('--accent'), 
                }] 
            }, 
            options:{ 
                scales:{
                    y:{
                        min:1, // Start y-axis at 1 (lowest score)
                        max:MAX_SCORE, // End y-axis at 7 (highest score)
                        ticks:{stepSize:1}
                    }
                }, 
                plugins:{legend:{display:false}}
            } 
        };
        if(chart) chart.destroy(); chart=new Chart(document.getElementById('moodChart'),cfg);
      } else {
        const counts={}; MOODS.forEach(m=>counts[m.emoji]=0); entries.forEach(e=>{if(counts[e.mood]!==undefined) counts[e.mood]++});
        // Ensure MOODS is in ascending score order for bar chart visualization
        const sortedMoods = MOODS.slice().sort((a,b)=>a.score-b.score);
        const labels=sortedMoods.map(m=>`${m.score} ${m.emoji}`); 
        const data=sortedMoods.map(m=>counts[m.emoji]); 
        const colors=sortedMoods.map(m=>getComputedStyle(document.documentElement).getPropertyValue(m.color));
        
        const cfg={
            type:'bar',
            data:{
                labels,
                datasets:[{
                    label:'Count',
                    data,
                    backgroundColor:colors, // Use the new colors for the bars
                    borderRadius:8
                }]
            },
            options:{
                plugins:{legend:{display:false}},
                scales:{y:{beginAtZero:true,ticks:{precision:0}}},
                // Adjust x-axis to be less cluttered if needed
            }
        };
        if(chart) chart.destroy(); chart=new Chart(document.getElementById('moodChart'),cfg);
      }
    }
    chartTypeEl.addEventListener('change',updateChart);

    function download(filename,content){ const blob=new Blob([content],{type:'application/octet-stream'}); const url=URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download=filename; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url); }
    exportJsonBtn.addEventListener('click',()=>{ const list=loadEntries(); download('stress-diary.json',JSON.stringify(list,null,2)); });
    exportCsvBtn.addEventListener('click',()=>{ const list=loadEntries(); if(list.length===0){alert('No entries to export'); return;} const rows=[['date','mood','note']]; list.forEach(r=>rows.push([r.date,r.mood,'"'+(r.note||'').replace(/"/g,'""')+'"'])); download('stress-diary.csv',rows.map(r=>r.join(',')).join('\n')); });

    initUI(); renderJournal(); updateChart();
    window.addEventListener('storage',(e)=>{if(e.key===STORAGE_KEY){renderJournal();updateChart();}});
  </script>
</body>
</html>
