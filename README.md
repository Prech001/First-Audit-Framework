<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>First-Audit Framework</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Oswald:wght@400;500;600;700&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500;600&display=swap');
  :root {
    --ink: #16324A;
    --blueprint: #123A5C;
    --blueprint-dark: #0B2739;
    --blueprint-line: #6FA3C9;
    --paper: #EEF2F5;
    --paper-line: #C7D3DB;
    --stamp-red: #B23A2E;
    --stamp-green: #2F6B4F;
    --stamp-amber: #C97A1F;
    --steel: #5C6B73;
  }
  * { box-sizing: border-box; }
  html, body { margin: 0; padding: 0; background: var(--paper); color: var(--ink); font-family: 'IBM Plex Sans', sans-serif; }
  #app { min-height: 100vh; }
  .fa-btn { font-family: 'IBM Plex Sans', sans-serif; font-weight: 600; font-size: 13px; padding: 10px 16px; border-radius: 6px; border: 1.5px solid var(--blueprint); background: var(--blueprint); color: #fff; cursor: pointer; }
  .fa-btn.secondary { background: transparent; color: var(--blueprint); }
  .fa-btn.ghost { background: transparent; color: var(--ink); border-color: var(--paper-line); }
  .fa-btn:hover { opacity: 0.88; }
  .fa-btn:disabled { opacity: 0.4; cursor: not-allowed; }
  .fa-tab { font-family: 'Oswald', sans-serif; letter-spacing: 0.05em; text-transform: uppercase; font-size: 13px; padding: 12px 18px; cursor: pointer; color: rgba(255,255,255,0.6); border-bottom: 3px solid transparent; display: inline-block; }
  .fa-tab.active { color: #fff; border-bottom: 3px solid var(--stamp-amber); }
  .fa-card { background: #fff; border: 1px solid var(--paper-line); border-radius: 8px; padding: 20px; margin-bottom: 18px; }
  .fa-input, .fa-select { font-family: 'IBM Plex Mono', monospace; font-size: 13px; padding: 8px 10px; border: 1.5px solid var(--paper-line); border-radius: 5px; background: #fff; color: var(--ink); width: 100%; }
  table.fa-table { width: 100%; border-collapse: collapse; font-size: 13px; }
  table.fa-table th { text-align: left; font-family: 'Oswald', sans-serif; letter-spacing: 0.04em; text-transform: uppercase; font-size: 11px; color: var(--steel); border-bottom: 2px solid var(--ink); padding: 8px 6px; }
  table.fa-table td { padding: 7px 6px; border-bottom: 1px solid var(--paper-line); font-family: 'IBM Plex Mono', monospace; }
  .checklist-item { display: flex; gap: 10px; align-items: flex-start; padding: 8px 0; border-bottom: 1px solid var(--paper-line); font-size: 13.5px; }
  .section-title { font-family: 'Oswald', sans-serif; text-transform: uppercase; font-size: 16px; margin-bottom: 6px; }
  .muted { color: var(--steel); font-size: 13.5px; }
  .stamp { display: inline-flex; align-items: center; justify-content: center; border-radius: 10px; padding: 8px 18px; font-family: 'Oswald', sans-serif; font-weight: 600; letter-spacing: 0.12em; font-size: 15px; transform: rotate(-4deg); text-transform: uppercase; background: rgba(255,255,255,0.4); }
  .stamp.pass { border: 3px solid var(--stamp-green); color: var(--stamp-green); }
  .stamp.fail { border: 3px solid var(--stamp-red); color: var(--stamp-red); }
  .grid-wrap { overflow-x: auto; border: 1px solid var(--paper-line); border-radius: 6px; }
  .grid-wrap table { border-collapse: collapse; width: 100%; font-size: 12.5px; }
  .grid-wrap th, .grid-wrap td { border-bottom: 1px solid var(--paper-line); }
  .grid-wrap th { background: var(--paper); border-bottom: 2px solid var(--ink); padding: 4px; min-width: 130px; }
  .grid-wrap td { padding: 2px; }
  .grid-cell-input { width: 100%; border: 1px solid transparent; padding: 6px; font-family: 'IBM Plex Mono', monospace; font-size: 12.5px; background: transparent; }
  .grid-cell-input:focus { border: 1px solid var(--blueprint-line); outline: none; }
  .grid-header-input { font-family: 'IBM Plex Mono', monospace; font-size: 12px; font-weight: 600; border: none; background: transparent; width: 100%; padding: 4px; color: var(--ink); }
  .col-remove, .row-remove { border: none; background: none; color: var(--stamp-red); cursor: pointer; font-size: 13px; padding: 0 4px; }
  .pill-row { display: flex; gap: 8px; margin-bottom: 16px; flex-wrap: wrap; }
  .check-pill { display: flex; align-items: center; gap: 6px; border: 1.5px solid var(--paper-line); border-radius: 6px; padding: 6px 10px; font-size: 12.5px; cursor: pointer; background: #fff; }
  .check-pill.on { background: #E3ECF1; }
  @media print {
    .no-print { display: none !important; }
    body { background: #fff !important; }
    .fa-card { border: 1px solid #999 !important; }
  }
</style>
</head>
<body>
<div id="app"></div>

<script>
/* ---------------------------------------------------------------
   FIRST-AUDIT FRAMEWORK — standalone HTML/vanilla JS build
   Same engine as the React version: disparate-impact screening
   (four-fifths rule + chi-square), concentration risk (HHI),
   a NIST/ISO-aligned governance checklist, and a printable report.
   This is a screening instrument, not a legal determination.
------------------------------------------------------------------*/

// ---------- Statistics helpers ----------
const CHI_SQ_CRIT = {1:3.841,2:5.991,3:7.815,4:9.488,5:11.07,6:12.592,7:14.067,8:15.507,9:16.919,10:18.307};
function chiSquareCritical(df){
  if (CHI_SQ_CRIT[df]) return {value: CHI_SQ_CRIT[df], approx:false};
  const z = 1.645;
  return {value: df + Math.sqrt(2*df)*z, approx:true};
}
function isNumericValue(v){ if(v===null||v===undefined||v==='') return false; return typeof v==='number' && !Number.isNaN(v); }
function uniqueValues(rows,col){ const s=new Set(); rows.forEach(r=>s.add(r[col])); return Array.from(s).filter(v=>v!==undefined&&v!==null&&v!==''); }
function fmtPct(n){ return (n*100).toFixed(1)+'%'; }
function fmtNum(n,d=2){ if(n===null||n===undefined||Number.isNaN(n)) return '—'; return Number(n).toFixed(d); }
function esc(s){ return String(s===undefined||s===null?'':s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'); }

// ---------- Core analysis engine ----------
function analyzeGroupColumn(rows, groupCol, outcomeCol, positiveValue){
  const groups = {};
  rows.forEach(r=>{
    const g = r[groupCol];
    if (g===undefined||g===null||g==='') return;
    if (!groups[g]) groups[g] = {total:0, positive:0};
    groups[g].total += 1;
    if (String(r[outcomeCol])===String(positiveValue)) groups[g].positive += 1;
  });
  const groupNames = Object.keys(groups);
  const rates = groupNames.map(g=>({ group:g, total:groups[g].total, positive:groups[g].positive, rate: groups[g].total>0? groups[g].positive/groups[g].total:0 }));
  const maxRate = Math.max(...rates.map(r=>r.rate),0);
  const withRatio = rates.map(r=>({ ...r, impactRatio: maxRate>0? r.rate/maxRate:null, lowSample: r.total<5, flagged: maxRate>0 && (r.rate/maxRate)<0.8 && r.total>=5 }));
  const total = rates.reduce((a,r)=>a+r.total,0);
  const totalPositive = rates.reduce((a,r)=>a+r.positive,0);
  const overallRate = total>0? totalPositive/total:0;
  let chiSq = 0;
  withRatio.forEach(r=>{
    const expectedPos = r.total*overallRate;
    const expectedNeg = r.total*(1-overallRate);
    const obsPos = r.positive;
    const obsNeg = r.total - r.positive;
    if (expectedPos>0) chiSq += Math.pow(obsPos-expectedPos,2)/expectedPos;
    if (expectedNeg>0) chiSq += Math.pow(obsNeg-expectedNeg,2)/expectedNeg;
  });
  const df = Math.max(groupNames.length-1,1);
  const crit = chiSquareCritical(df);
  const significant = chiSq > crit.value;
  const worstGroup = withRatio.reduce((worst,r)=>{ if (r.lowSample) return worst; if (!worst||r.rate<worst.rate) return r; return worst; }, null);
  return {
    groupCol,
    rows: withRatio.sort((a,b)=>b.total-a.total),
    minImpactRatio: Math.min(...withRatio.filter(r=>!r.lowSample).map(r=> r.impactRatio===null?1:r.impactRatio), 1),
    anyFlagged: withRatio.some(r=>r.flagged),
    chiSq, df, crit, significant, worstGroup
  };
}
function analyzeConcentration(rows, entityCol, weightCol){
  const totals = {};
  rows.forEach(r=>{
    const e = r[entityCol];
    if (e===undefined||e===null||e==='') return;
    const w = weightCol? (Number(r[weightCol])||0) : 1;
    totals[e] = (totals[e]||0) + w;
  });
  const grand = Object.values(totals).reduce((a,b)=>a+b,0);
  const shares = Object.entries(totals).map(([e,v])=>({entity:e, value:v, share: grand>0? v/grand:0}));
  const hhi = shares.reduce((a,s)=>a+Math.pow(s.share*100,2),0);
  let level = 'Unconcentrated';
  if (hhi>2500) level='Highly Concentrated'; else if (hhi>=1500) level='Moderately Concentrated';
  shares.sort((a,b)=>b.value-a.value);
  return {entityCol, weightCol, hhi, level, top: shares.slice(0,8), n: shares.length};
}
function detectColumns(rows, headers){
  const numericCols=[], categoricalCols=[], idLikeCols=[];
  headers.forEach(h=>{
    const vals = rows.map(r=>r[h]);
    const nonEmpty = vals.filter(v=>v!==''&&v!==null&&v!==undefined);
    if (nonEmpty.length===0) return;
    const numericCount = nonEmpty.filter(isNumericValue).length;
    const uniq = new Set(nonEmpty.map(String));
    if (numericCount/nonEmpty.length > 0.85) numericCols.push(h);
    else if (uniq.size===nonEmpty.length && uniq.size>8) idLikeCols.push(h);
    else if (uniq.size<=20) categoricalCols.push(h);
  });
  return {numericCols, categoricalCols, idLikeCols};
}
function proxyFlags(rows, numericCols, worstGroupInfo, groupCol){
  if (!worstGroupInfo) return [];
  const flags = [];
  const worstGroupVal = worstGroupInfo.group;
  numericCols.forEach(col=>{
    const inGroup=[], outGroup=[];
    rows.forEach(r=>{
      const v = Number(r[col]);
      if (Number.isNaN(v)) return;
      if (String(r[groupCol])===String(worstGroupVal)) inGroup.push(v); else outGroup.push(v);
    });
    if (inGroup.length<5 || outGroup.length<5) return;
    const meanIn = inGroup.reduce((a,b)=>a+b,0)/inGroup.length;
    const meanOut = outGroup.reduce((a,b)=>a+b,0)/outGroup.length;
    const base = Math.max(Math.abs(meanOut), 1e-9);
    const relDiff = Math.abs(meanIn-meanOut)/base;
    if (relDiff>0.25) flags.push({col, meanIn, meanOut, relDiff, groupCol, worstGroupVal});
  });
  return flags;
}

// ---------- CSV template + export ----------
const TEMPLATE_CSV = `supplier_id,supplier_name,region,ownership_type,business_size_tier,contract_value_usd,decision_outcome
SUP-1001,Halden Component Works,North America,Majority-Owned,Medium,182000,Approved
SUP-1002,Ferra Metal Partners,Eastern Europe,Woman-Owned,Small,64000,Rejected
SUP-1003,Okoye Precision Tooling,Sub-Saharan Africa,Minority-Owned,Small,71000,Rejected
SUP-1004,Highline Fastener Co,North America,Majority-Owned,Large,410000,Approved
SUP-1005,Suvarna Alloy Group,Southeast Asia,Minority-Owned,Medium,158000,Approved
SUP-1006,Bergman Industrial,North America,Veteran-Owned,Medium,176000,Approved
SUP-1007,Meridian Cast Parts,Southeast Asia,Minority-Owned,Small,52000,Rejected
SUP-1008,Torrington Fasteners,North America,Majority-Owned,Medium,198000,Approved
SUP-1009,Nia Textile & Trim,Sub-Saharan Africa,Woman-Owned,Small,49000,Rejected
SUP-1010,Baltic Forge Ltd,Eastern Europe,Majority-Owned,Medium,163000,Approved
`;
function downloadBlob(content, filename, type){
  const blob = new Blob([content], {type});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename;
  document.body.appendChild(a); a.click(); document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

// ---------- Governance checklist ----------
const GOVERNANCE_SECTIONS = [
  { key:'govern', title:'Govern', subtitle:'Accountability & structure', items:[
    'A named individual (not a committee) is accountable for the outcomes of automated procurement decisions.',
    'There is a written policy describing which procurement decisions may be made or narrowed by automated systems.',
    'Suppliers or bidders have a documented path to contest or appeal an automated decision.',
    'Vendor contracts for automated systems include a right to audit and access to decision logs.'
  ]},
  { key:'map', title:'Map', subtitle:'Context & risk identification', items:[
    'Every automated system touching procurement has been inventoried, including tools embedded inside other software.',
    'Protected or sensitive attributes (or close proxies for them) used by any system have been identified in writing.',
    "The training data or historical basis for each system's logic has been reviewed for known skew or gaps.",
    'High-risk decision points (exclusion, disqualification, automatic de-prioritization) have been explicitly flagged.'
  ]},
  { key:'measure', title:'Measure', subtitle:'Testing & metrics', items:[
    'Disparate impact (four-fifths rule) testing has been run on this system within the last 12 months.',
    'A statistical significance check (not just the raw ratio) was applied to the results.',
    'Supplier or spend concentration risk has been measured, not just decision fairness.',
    'Testing is repeated on a defined schedule, not only once at deployment.'
  ]},
  { key:'manage', title:'Manage', subtitle:'Response & continuous monitoring', items:[
    'A documented remediation step exists for any system that fails a disparate-impact check.',
    'Findings from testing are escalated to someone with authority to pause or adjust the system.',
    'There is a monitoring cadence for model or data drift after initial deployment.',
    'Audit results are retained and available if a regulator or a top-tier buyer requests evidence.'
  ]}
];

// ---------- SVG widgets ----------
function ringGaugeSVG(value, label, sub, size){
  size = size || 118;
  const r = (size-16)/2;
  const c = 2*Math.PI*r;
  const pct = Math.max(0, Math.min(1, value===null||value===undefined?0:value));
  const offset = c*(1-pct);
  let color = 'var(--stamp-green)';
  if (pct<0.6) color='var(--stamp-red)'; else if (pct<0.8) color='var(--stamp-amber)';
  const valText = (value===null||value===undefined) ? '—' : (value*100).toFixed(0)+'%';
  return `
  <div style="display:flex;flex-direction:column;align-items:center;gap:6px;">
    <svg width="${size}" height="${size}" viewBox="0 0 ${size} ${size}">
      <circle cx="${size/2}" cy="${size/2}" r="${r}" fill="none" stroke="var(--paper-line)" stroke-width="10"/>
      <circle cx="${size/2}" cy="${size/2}" r="${r}" fill="none" stroke="${color}" stroke-width="10"
        stroke-dasharray="${c}" stroke-dashoffset="${offset}" stroke-linecap="round"
        transform="rotate(-90 ${size/2} ${size/2})" style="transition:stroke-dashoffset .6s ease;"/>
      <text x="50%" y="47%" text-anchor="middle" font-family="'IBM Plex Mono',monospace" font-size="22" font-weight="600" fill="var(--ink)">${valText}</text>
      <text x="50%" y="64%" text-anchor="middle" font-family="'IBM Plex Sans',sans-serif" font-size="9" fill="var(--steel)">${esc(sub||'')}</text>
    </svg>
    <div style="font-family:'Oswald',sans-serif;font-size:12px;letter-spacing:.06em;text-transform:uppercase;color:var(--ink);text-align:center;max-width:${size+30}px;">${esc(label||'')}</div>
  </div>`;
}
function stampHTML(status){
  const pass = status==='pass';
  return `<div class="stamp ${pass?'pass':'fail'}">${pass?'Cleared':'Flagged'}</div>`;
}
function barChartSVG(rows){
  const w = 560, h = 210, padL = 34, padB = 40, padT = 14, padR = 10;
  const innerW = w - padL - padR, innerH = h - padT - padB;
  const maxRate = Math.max(...rows.map(r=>r.rate), 0.01);
  const n = rows.length;
  const bw = innerW / n * 0.6;
  const gap = innerW / n;
  const refY = padT + innerH * (1 - (maxRate*0.8)/Math.max(maxRate,0.0001));
  let bars = '';
  rows.forEach((r,i)=>{
    const bh = innerH * (r.rate / Math.max(maxRate,0.0001));
    const x = padL + gap*i + (gap-bw)/2;
    const y = padT + innerH - bh;
    const color = r.flagged ? 'var(--stamp-red)' : 'var(--stamp-green)';
    const label = String(r.group).length > 12 ? String(r.group).slice(0,11)+'…' : String(r.group);
    bars += `<rect x="${x}" y="${y}" width="${bw}" height="${Math.max(bh,1)}" fill="${color}" rx="2"/>`;
    bars += `<text x="${x+bw/2}" y="${y-4}" text-anchor="middle" font-size="10" font-family="IBM Plex Mono" fill="var(--ink)">${(r.rate*100).toFixed(0)}%</text>`;
    bars += `<text x="${x+bw/2}" y="${padT+innerH+14}" text-anchor="middle" font-size="10" font-family="IBM Plex Mono" fill="var(--steel)">${esc(label)}</text>`;
  });
  return `<svg width="100%" viewBox="0 0 ${w} ${h}" style="max-width:100%;">
    <line x1="${padL}" y1="${padT}" x2="${padL}" y2="${padT+innerH}" stroke="var(--paper-line)"/>
    <line x1="${padL}" y1="${padT+innerH}" x2="${padL+innerW}" y2="${padT+innerH}" stroke="var(--ink)" stroke-width="1.5"/>
    <line x1="${padL}" y1="${refY}" x2="${padL+innerW}" y2="${refY}" stroke="var(--stamp-amber)" stroke-dasharray="4 4"/>
    <text x="${padL+innerW}" y="${refY-4}" text-anchor="end" font-size="10" fill="var(--stamp-amber)" font-family="IBM Plex Mono">80% floor</text>
    ${bars}
  </svg>`;
}

// ---------- App state ----------
const state = {
  tab: 'data',
  inputMode: 'upload',
  rows: [], headers: [], fileName: '', parseError: '',
  outcomeCol: '', positiveValue: '', selectedGroupCols: [], entityCol: '', weightCol: '',
  results: null,
  orgName: '',
  govAnswers: {},
  manual: { colDefs: [], data: [] }
};

const STARTER_MANUAL_COLUMNS = ['supplier_id','region','ownership_type','business_size_tier','contract_value_usd','decision_outcome'];
function makeCol(name){ return { id: 'c'+Math.random().toString(36).slice(2,9), name }; }
function resetManualGrid(){
  state.manual.colDefs = STARTER_MANUAL_COLUMNS.map(makeCol);
  state.manual.data = Array.from({length:4}, ()=> Object.fromEntries(state.manual.colDefs.map(c=>[c.id,''])));
}
resetManualGrid();

function detectedCols(){ return state.rows.length ? detectColumns(state.rows, state.headers) : {numericCols:[],categoricalCols:[],idLikeCols:[]}; }

function govScore(){
  const allItems = GOVERNANCE_SECTIONS.flatMap(s=>s.items.map((_,i)=>`${s.key}-${i}`));
  const answered = allItems.filter(k=>state.govAnswers[k]).length;
  return { answered, total: allItems.length, pct: allItems.length? answered/allItems.length : 0 };
}

function commitData(rowsOut, headersOut, label){
  state.rows = rowsOut;
  state.headers = headersOut;
  state.fileName = label;
  state.parseError = '';
  state.results = null;
  state.outcomeCol = ''; state.positiveValue = ''; state.selectedGroupCols = [];
  state.entityCol = ''; state.weightCol = '';
  render();
}

function handleFileInput(file){
  const reader = new FileReader();
  reader.onload = (ev)=>{
    Papa.parse(ev.target.result, {
      header:true, dynamicTyping:true, skipEmptyLines:true,
      complete: (res)=>{
        if (!res.data.length){ state.parseError = "No rows found in this file. Check that it's a comma-separated CSV with a header row."; render(); return; }
        commitData(res.data, res.meta.fields || Object.keys(res.data[0]), file.name + ' · ' + res.data.length + ' rows');
      },
      error: (err)=>{ state.parseError = 'Could not parse this file: ' + err.message; render(); }
    });
  };
  reader.readAsText(file);
}
function loadTemplateAsData(){
  Papa.parse(TEMPLATE_CSV, {
    header:true, dynamicTyping:true, skipEmptyLines:true,
    complete: (res)=>{
      state.rows = res.data;
      state.headers = res.meta.fields || Object.keys(res.data[0]);
      state.fileName = 'sample-procurement-data.csv (template) · ' + res.data.length + ' rows';
      state.results = null;
      state.outcomeCol = 'decision_outcome';
      state.positiveValue = 'Approved';
      state.selectedGroupCols = ['region','ownership_type','business_size_tier'];
      state.entityCol = 'supplier_id';
      state.weightCol = 'contract_value_usd';
      render();
    }
  });
}
function runAudit(){
  if (!state.outcomeCol || !state.positiveValue || state.selectedGroupCols.length===0) return;
  const byGroup = state.selectedGroupCols.map(gc=>analyzeGroupColumn(state.rows, gc, state.outcomeCol, state.positiveValue));
  const concentration = state.entityCol ? analyzeConcentration(state.rows, state.entityCol, state.weightCol) : null;
  const firstFlagged = byGroup.find(g=>g.anyFlagged) || byGroup[0];
  const cols = detectedCols();
  const proxies = firstFlagged ? proxyFlags(state.rows, cols.numericCols.filter(c=>c!==state.weightCol), firstFlagged.worstGroup, firstFlagged.groupCol) : [];
  state.results = { byGroup, concentration, proxies, runAt: new Date() };
  state.tab = 'analysis';
  render();
}
function exportResultsCsv(){
  if (!state.results) return;
  let out = 'group_column,group_value,n,positive_rate,impact_ratio,flagged,low_sample\n';
  state.results.byGroup.forEach(g=>{
    g.rows.forEach(r=>{
      out += `${g.groupCol},${r.group},${r.total},${fmtPct(r.rate)},${r.impactRatio===null?'':fmtNum(r.impactRatio,3)},${r.flagged},${r.lowSample}\n`;
    });
  });
  if (state.results.concentration){
    out += `\nconcentration_metric,value\n`;
    out += `entity_column,${state.results.concentration.entityCol}\n`;
    out += `weight_column,${state.results.concentration.weightCol || 'count'}\n`;
    out += `hhi,${fmtNum(state.results.concentration.hhi,1)}\n`;
    out += `level,${state.results.concentration.level}\n`;
  }
  downloadBlob(out, 'first-audit-results.csv', 'text/csv');
}

// ---------- Manual grid ----------
function manualBuildRowsAndHeaders(){
  const colDefs = state.manual.colDefs, data = state.manual.data;
  const headers = colDefs.map(c=>c.name.trim()).filter(Boolean);
  const numericCols = new Set();
  colDefs.forEach(c=>{
    const vals = data.map(r=>r[c.id]).filter(v=>v!==''&&v!==null&&v!==undefined);
    if (vals.length>0 && vals.every(v=>v!=='' && !Number.isNaN(Number(v)))) numericCols.add(c.id);
  });
  const rowsOut = data.filter(r=>colDefs.some(c=>String(r[c.id]||'').trim()!=='')).map(r=>{
    const obj = {};
    colDefs.forEach(c=>{
      const raw = r[c.id]; const name = c.name.trim();
      if (!name) return;
      obj[name] = numericCols.has(c.id) && raw!=='' ? Number(raw) : raw;
    });
    return obj;
  });
  return {rowsOut, headers};
}
function manualExportCsv(){
  const colDefs = state.manual.colDefs, data = state.manual.data;
  const headers = colDefs.map(c=>c.name);
  let out = headers.map(h=>`"${String(h).replace(/"/g,'""')}"`).join(',') + '\n';
  data.forEach(r=>{ out += colDefs.map(c=>`"${String(r[c.id]??'').replace(/"/g,'""')}"`).join(',') + '\n'; });
  downloadBlob(out, 'manual-entry-data.csv', 'text/csv');
}
function manualFilledRowCount(){
  return state.manual.data.filter(r=>state.manual.colDefs.some(c=>String(r[c.id]||'').trim()!=='')).length;
}
function renderManualGrid(){
  const container = document.getElementById('manual-grid-root');
  if (!container) return;
  const colDefs = state.manual.colDefs, data = state.manual.data;
  let thead = `<th style="width:32px;"></th>` + colDefs.map(c=>`
    <th data-col="${c.id}">
      <div style="display:flex;align-items:center;gap:4px;">
        <input class="grid-header-input" data-role="header" data-col="${c.id}" value="${esc(c.name)}"/>
        <button class="col-remove" data-action="remove-col" data-col="${c.id}" title="Remove column">×</button>
      </div>
    </th>`).join('') + `<th><button class="fa-btn ghost" data-action="add-col" style="padding:4px 8px;font-size:11px;">+ Column</button></th>`;
  let tbody = data.map((r,idx)=>{
    let cells = colDefs.map(c=>`<td><input class="grid-cell-input" data-role="cell" data-row="${idx}" data-col="${c.id}" value="${esc(r[c.id]??'')}"/></td>`).join('');
    return `<tr>
      <td style="text-align:center;color:var(--steel);font-family:'IBM Plex Mono',monospace;font-size:11px;">${idx+1}</td>
      ${cells}
      <td style="text-align:center;"><button class="row-remove" data-action="remove-row" data-row="${idx}" title="Remove row">×</button></td>
    </tr>`;
  }).join('');
  const filled = manualFilledRowCount();
  container.innerHTML = `
    <div class="muted" style="margin-bottom:12px;">Type or paste values cell by cell. Rename column headers to match your own data — anything you'd normally see in a spend report or supplier register works: region, certification status, past-performance score, whatever your process actually tracks.</div>
    <div class="grid-wrap"><table><thead><tr>${thead}</tr></thead><tbody>${tbody}</tbody></table></div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:12px;">
      <button class="fa-btn ghost" data-action="add-row-1">+ Add row</button>
      <button class="fa-btn ghost" data-action="add-row-10">+ Add 10 rows</button>
      <button class="fa-btn ghost" data-action="clear-values">Clear values</button>
      <button class="fa-btn secondary" data-action="manual-export">Save as CSV</button>
      <button class="fa-btn" data-action="manual-commit" ${filled<2?'disabled':''}>Use this data (${filled} row${filled!==1?'s':''}) →</button>
    </div>
    ${filled<2 ? `<div style="font-size:12px;color:var(--steel);margin-top:6px;">Enter at least 2 rows with values before running an audit — disparate-impact math needs more than one data point per group.</div>` : ''}
  `;

  container.addEventListener('input', function(e){
    const t = e.target;
    if (t.dataset.role === 'cell'){
      const idx = Number(t.dataset.row), colId = t.dataset.col;
      state.manual.data[idx][colId] = t.value;
    } else if (t.dataset.role === 'header'){
      const colId = t.dataset.col;
      const col = state.manual.colDefs.find(c=>c.id===colId);
      if (col) col.name = t.value;
    }
  });
  container.addEventListener('click', function(e){
    const btn = e.target.closest('[data-action]');
    if (!btn) return;
    const action = btn.dataset.action;
    if (action==='add-col'){
      const newCol = makeCol('field_' + (state.manual.colDefs.length+1));
      state.manual.colDefs.push(newCol);
      state.manual.data.forEach(r=> r[newCol.id] = '');
      renderManualGrid();
    } else if (action==='remove-col'){
      if (state.manual.colDefs.length<=1) return;
      const colId = btn.dataset.col;
      state.manual.colDefs = state.manual.colDefs.filter(c=>c.id!==colId);
      state.manual.data.forEach(r=> delete r[colId]);
      renderManualGrid();
    } else if (action==='add-row-1'){
      state.manual.data.push(Object.fromEntries(state.manual.colDefs.map(c=>[c.id,''])));
      renderManualGrid();
    } else if (action==='add-row-10'){
      for(let i=0;i<10;i++) state.manual.data.push(Object.fromEntries(state.manual.colDefs.map(c=>[c.id,''])));
      renderManualGrid();
    } else if (action==='remove-row'){
      const idx = Number(btn.dataset.row);
      state.manual.data.splice(idx,1);
      renderManualGrid();
    } else if (action==='clear-values'){
      state.manual.data = state.manual.data.map(()=>Object.fromEntries(state.manual.colDefs.map(c=>[c.id,''])));
      renderManualGrid();
    } else if (action==='manual-export'){
      manualExportCsv();
    } else if (action==='manual-commit'){
      const {rowsOut, headers} = manualBuildRowsAndHeaders();
      commitData(rowsOut, headers, `Manual entry · ${rowsOut.length} rows`);
    }
  });
}

// ---------- Tab renderers ----------
function renderHeaderNav(){
  const tabs = [['data','1 · Data'],['analysis','2 · Bias Analysis'],['governance','3 · Governance'],['report','4 · Report']];
  return `
  <div class="no-print" style="background:
      repeating-linear-gradient(0deg, var(--blueprint-dark), var(--blueprint-dark) 27px, rgba(255,255,255,0.045) 28px),
      repeating-linear-gradient(90deg, var(--blueprint-dark), var(--blueprint-dark) 27px, rgba(255,255,255,0.045) 28px);
      padding:22px 28px 0 28px;">
    <div style="display:flex;justify-content:space-between;align-items:flex-end;flex-wrap:wrap;gap:12px;">
      <div>
        <div style="font-family:'IBM Plex Mono',monospace;color:var(--blueprint-line);font-size:11px;letter-spacing:.15em;margin-bottom:4px;">FORM FA-01 · REV. A</div>
        <div style="font-family:'Oswald',sans-serif;color:#fff;font-size:30px;font-weight:600;letter-spacing:.02em;text-transform:uppercase;">First-Audit Framework</div>
        <div style="color:rgba(255,255,255,0.68);font-size:13px;margin-top:2px;max-width:560px;">A working instrument for testing procurement and supplier-selection systems for disparate impact and concentration risk — before a regulator or buyer asks you to.</div>
      </div>
    </div>
    <div style="display:flex;gap:4px;margin-top:18px;">
      ${tabs.map(([k,label])=>`<div class="fa-tab ${state.tab===k?'active':''}" data-tab="${k}">${label}</div>`).join('')}
    </div>
  </div>`;
}

function renderDataTab(){
  const cols = detectedCols();
  let out = `<div class="fa-card">
    <div class="section-title">Step 1 — Load your data</div>
    <div class="muted" style="margin-bottom:14px;">Bring in supplier or bid decisions from your procurement system, or build the table by hand if you don't have a clean export yet. One row per supplier decision. Nothing leaves this session — everything runs locally in your browser.</div>
    <div class="pill-row">
      <button class="fa-btn ${state.inputMode==='upload'?'':'ghost'}" data-action="mode-upload">Upload a file</button>
      <button class="fa-btn ${state.inputMode==='manual'?'':'ghost'}" data-action="mode-manual">Enter data manually</button>
    </div>`;

  if (state.inputMode==='upload'){
    out += `<div style="display:flex;gap:10px;flex-wrap:wrap;">
      <button class="fa-btn" data-action="upload-click">Upload CSV</button>
      <input id="file-input" type="file" accept=".csv" style="display:none;"/>
      <button class="fa-btn secondary" data-action="download-template">Download CSV template</button>
      <button class="fa-btn ghost" data-action="load-sample">Try it with sample data</button>
    </div>`;
  } else {
    out += `<div id="manual-grid-root"></div>`;
  }

  if (state.fileName) out += `<div style="margin-top:10px;font-size:12.5px;font-family:'IBM Plex Mono',monospace;color:var(--steel);">Loaded: ${esc(state.fileName)}</div>`;
  if (state.parseError) out += `<div style="margin-top:10px;font-size:13px;color:var(--stamp-red);">${esc(state.parseError)}</div>`;
  out += `</div>`;

  if (state.rows.length>0){
    out += `<div class="fa-card">
      <div class="section-title">Step 2 — Preview</div>
      <div style="overflow-x:auto;"><table class="fa-table"><thead><tr>${state.headers.map(h=>`<th>${esc(h)}</th>`).join('')}</tr></thead><tbody>
        ${state.rows.slice(0,5).map(r=>`<tr>${state.headers.map(h=>`<td>${esc(r[h])}</td>`).join('')}</tr>`).join('')}
      </tbody></table></div>
      <div style="font-size:12px;color:var(--steel);margin-top:6px;">Showing first 5 of ${state.rows.length} rows.</div>
    </div>`;

    out += `<div class="fa-card">
      <div class="section-title">Step 3 — Map your columns</div>
      <div style="display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-bottom:16px;">
        <div>
          <label style="font-size:12px;color:var(--steel);display:block;margin-bottom:5px;">Decision / outcome column</label>
          <select class="fa-select" data-field="outcomeCol">
            <option value="">Select column…</option>
            ${state.headers.map(h=>`<option value="${esc(h)}" ${state.outcomeCol===h?'selected':''}>${esc(h)}</option>`).join('')}
          </select>
        </div>
        <div>
          <label style="font-size:12px;color:var(--steel);display:block;margin-bottom:5px;">Which value counts as "selected"?</label>
          <select class="fa-select" data-field="positiveValue" ${!state.outcomeCol?'disabled':''}>
            <option value="">Select value…</option>
            ${state.outcomeCol ? uniqueValues(state.rows, state.outcomeCol).map(v=>`<option value="${esc(v)}" ${String(state.positiveValue)===String(v)?'selected':''}>${esc(v)}</option>`).join('') : ''}
          </select>
        </div>
      </div>

      <div style="margin-bottom:16px;">
        <label style="font-size:12px;color:var(--steel);display:block;margin-bottom:6px;">Test these columns for disparate impact (pick attributes that could carry bias risk — region, ownership type, size tier, etc.)</label>
        <div style="display:flex;flex-wrap:wrap;gap:8px;">
          ${cols.categoricalCols.filter(c=>c!==state.outcomeCol).map(c=>`
            <label class="check-pill ${state.selectedGroupCols.includes(c)?'on':''}">
              <input type="checkbox" data-action="toggle-group-col" data-col="${esc(c)}" ${state.selectedGroupCols.includes(c)?'checked':''}/> ${esc(c)}
            </label>`).join('') || `<div class="muted">No categorical columns detected — check that grouping columns (like region or ownership type) have 20 or fewer distinct values.</div>`}
        </div>
      </div>

      <div style="display:grid;grid-template-columns:1fr 1fr;gap:16px;">
        <div>
          <label style="font-size:12px;color:var(--steel);display:block;margin-bottom:5px;">Supplier / entity column (for concentration risk)</label>
          <select class="fa-select" data-field="entityCol">
            <option value="">None</option>
            ${[...cols.idLikeCols, ...cols.categoricalCols].map(h=>`<option value="${esc(h)}" ${state.entityCol===h?'selected':''}>${esc(h)}</option>`).join('')}
          </select>
        </div>
        <div>
          <label style="font-size:12px;color:var(--steel);display:block;margin-bottom:5px;">Spend / contract value column (optional weight)</label>
          <select class="fa-select" data-field="weightCol" ${!state.entityCol?'disabled':''}>
            <option value="">Use count instead</option>
            ${cols.numericCols.map(h=>`<option value="${esc(h)}" ${state.weightCol===h?'selected':''}>${esc(h)}</option>`).join('')}
          </select>
        </div>
      </div>

      <div style="margin-top:18px;">
        <button class="fa-btn" data-action="run-audit" ${(!state.outcomeCol||!state.positiveValue||state.selectedGroupCols.length===0)?'disabled':''}>Run audit →</button>
      </div>
    </div>`;
  }
  return out;
}

function renderAnalysisTab(){
  if (!state.results) return `<div class="fa-card">Load data and run an audit from the Data tab first.</div>`;
  const r = state.results;
  let out = '';
  r.byGroup.forEach(g=>{
    out += `<div class="fa-card">
      <div style="display:flex;justify-content:space-between;align-items:flex-start;flex-wrap:wrap;gap:14px;">
        <div>
          <div class="section-title">Disparate impact — ${esc(g.groupCol)}</div>
          <div class="muted">Four-fifths rule (EEOC 80% threshold), applied across ${g.rows.length} group${g.rows.length!==1?'s':''}.</div>
        </div>
        ${stampHTML(g.anyFlagged?'fail':'pass')}
      </div>
      <div style="display:flex;gap:26px;flex-wrap:wrap;margin:18px 0;align-items:center;">
        <div>${ringGaugeSVG(g.minImpactRatio, 'Lowest impact ratio', 'vs. 80% floor')}</div>
        <div style="flex:1;min-width:260px;">${barChartSVG(g.rows)}</div>
      </div>
      <div style="overflow-x:auto;"><table class="fa-table"><thead><tr><th>Group</th><th>N</th><th>Selection rate</th><th>Impact ratio</th><th>Status</th></tr></thead><tbody>
        ${g.rows.map(row=>`<tr>
          <td style="font-family:'IBM Plex Sans',sans-serif;">${esc(row.group)}</td>
          <td>${row.total}</td>
          <td>${fmtPct(row.rate)}</td>
          <td>${row.impactRatio===null?'—':fmtNum(row.impactRatio,2)}</td>
          <td style="color:${row.lowSample?'var(--steel)':row.flagged?'var(--stamp-red)':'var(--stamp-green)'};">${row.lowSample?'Sample too small':row.flagged?'Below 80% floor':'Within range'}</td>
        </tr>`).join('')}
      </tbody></table></div>
      <div style="margin-top:12px;font-size:12.5px;color:var(--steel);">
        Chi-square statistic: ${fmtNum(g.chiSq,2)} (df=${g.df}, critical value${g.crit.approx?' ≈':' ='} ${fmtNum(g.crit.value,2)}) —
        <strong style="color:${g.significant?'var(--stamp-red)':'var(--stamp-green)'};">${g.significant?'difference unlikely to be chance':'not statistically significant at this sample size'}</strong>.
      </div>
    </div>`;
  });

  if (r.concentration){
    const c = r.concentration;
    out += `<div class="fa-card">
      <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:14px;">
        <div>
          <div class="section-title">Supply concentration risk</div>
          <div class="muted">Herfindahl-Hirschman Index across ${esc(c.entityCol)}, weighted by ${esc(c.weightCol || 'row count')}.</div>
        </div>
        ${stampHTML(c.hhi>2500?'fail':'pass')}
      </div>
      <div style="display:flex;gap:26px;flex-wrap:wrap;margin:18px 0;align-items:center;">
        <div>${ringGaugeSVG(Math.min(c.hhi/10000,1), 'HHI (of 10,000 max)', c.level)}</div>
        <div style="font-size:13px;color:var(--ink);flex:1;min-width:220px;">
          <strong>${fmtNum(c.hhi,0)}</strong> — ${esc(c.level)}. ${c.level==='Highly Concentrated' ? 'A small number of suppliers account for most of your sourcing. This is a resilience risk independent of any bias finding — a disruption to one supplier disrupts the line.' : 'Sourcing is reasonably distributed across suppliers.'}
        </div>
      </div>
      <table class="fa-table"><thead><tr><th>Supplier</th><th>Share of total</th></tr></thead><tbody>
        ${c.top.map(s=>`<tr><td>${esc(s.entity)}</td><td>${fmtPct(s.share)}</td></tr>`).join('')}
      </tbody></table>
    </div>`;
  }

  if (r.proxies && r.proxies.length>0){
    out += `<div class="fa-card">
      <div class="section-title">Potential proxy signals</div>
      <div class="muted" style="margin-bottom:10px;">Numeric fields whose average value differs sharply for the lowest-scoring group. A large gap doesn't prove causation, but it's worth checking whether this field is quietly standing in for the protected attribute.</div>
      <table class="fa-table"><thead><tr><th>Field</th><th>Lowest-scoring group avg.</th><th>Everyone else avg.</th><th>Relative gap</th></tr></thead><tbody>
        ${r.proxies.map(p=>`<tr><td style="font-family:'IBM Plex Sans',sans-serif;">${esc(p.col)}</td><td>${fmtNum(p.meanIn,2)}</td><td>${fmtNum(p.meanOut,2)}</td><td style="color:var(--stamp-amber);">${fmtPct(p.relDiff)}</td></tr>`).join('')}
      </tbody></table>
    </div>`;
  }

  out += `<div style="display:flex;gap:10px;">
    <button class="fa-btn secondary" data-action="export-csv">Export findings as CSV</button>
    <button class="fa-btn" data-action="goto-governance">Continue to governance checklist →</button>
  </div>`;
  return out;
}

function renderGovernanceTab(){
  const score = govScore();
  let out = `<div class="fa-card">
    <div class="section-title">Governance self-assessment</div>
    <div class="muted">Structured around the four functions of the NIST AI Risk Management Framework — Govern, Map, Measure, Manage — the same skeleton that underlies a certifiable ISO/IEC 42001 management system. This isn't a certification. It's a fast way to see where your paper trail has gaps before someone else finds them.</div>
    <div style="margin-top:14px;">${ringGaugeSVG(score.pct, 'Governance readiness', `${score.answered}/${score.total} in place`)}</div>
  </div>`;

  GOVERNANCE_SECTIONS.forEach(s=>{
    out += `<div class="fa-card">
      <div style="font-family:'Oswald',sans-serif;text-transform:uppercase;font-size:15px;">${esc(s.title)}</div>
      <div class="muted" style="margin-bottom:8px;">${esc(s.subtitle)}</div>
      ${s.items.map((item,i)=>{
        const k = `${s.key}-${i}`;
        return `<label class="checklist-item"><input type="checkbox" data-action="gov-check" data-key="${k}" ${state.govAnswers[k]?'checked':''} style="margin-top:2px;"/><span>${esc(item)}</span></label>`;
      }).join('')}
    </div>`;
  });

  out += `<div><button class="fa-btn" data-action="goto-report">Continue to report →</button></div>`;
  return out;
}

function renderReportTab(){
  const score = govScore();
  const r = state.results;
  let out = `<div class="fa-card no-print">
    <div style="display:flex;gap:16px;flex-wrap:wrap;align-items:flex-end;">
      <div style="flex:1;min-width:220px;">
        <label style="font-size:12px;color:var(--steel);display:block;margin-bottom:5px;">Organization name (appears on report)</label>
        <input class="fa-input" id="org-name-input" value="${esc(state.orgName)}" placeholder="e.g. Torrington Fasteners Co."/>
      </div>
      <button class="fa-btn" data-action="print-report">Export as PDF (print)</button>
      ${r ? `<button class="fa-btn secondary" data-action="export-csv">Export findings CSV</button>` : ''}
    </div>
  </div>`;

  out += `<div id="report-content" class="fa-card">
    <div style="border-bottom:2px solid var(--ink);padding-bottom:12px;margin-bottom:16px;">
      <div style="font-family:'IBM Plex Mono',monospace;font-size:11px;color:var(--steel);">FIRST-AUDIT FRAMEWORK · SUMMARY REPORT</div>
      <div style="font-family:'Oswald',sans-serif;font-size:22px;text-transform:uppercase;"><span class="fa-org-name">${esc(state.orgName) || '[Organization Name]'}</span></div>
      <div style="font-size:12.5px;color:var(--steel);">Generated ${new Date().toLocaleDateString()} ${r && r.runAt ? '· Audit run ' + r.runAt.toLocaleString() : ''}</div>
    </div>

    <div style="margin-bottom:20px;">
      <div style="font-family:'Oswald',sans-serif;font-size:14px;text-transform:uppercase;margin-bottom:6px;">1. Disparate impact findings</div>
      ${!r ? `<div class="muted">No audit has been run yet.</div>` : r.byGroup.map(g=>`
        <div style="font-size:13px;margin-bottom:8px;">
          <strong>${esc(g.groupCol)}:</strong>
          ${g.anyFlagged
            ? `<span style="color:var(--stamp-red);">flagged — lowest impact ratio ${fmtNum(g.minImpactRatio,2)}, below the 0.80 floor${g.significant ? ', and the gap is unlikely to be chance' : ' (not yet statistically significant at this sample size)'}.</span>`
            : `<span style="color:var(--stamp-green);">within range — lowest impact ratio ${fmtNum(g.minImpactRatio,2)}.</span>`}
        </div>`).join('')}
    </div>

    ${r && r.concentration ? `<div style="margin-bottom:20px;">
      <div style="font-family:'Oswald',sans-serif;font-size:14px;text-transform:uppercase;margin-bottom:6px;">2. Concentration risk</div>
      <div style="font-size:13px;">HHI of ${fmtNum(r.concentration.hhi,0)} across ${esc(r.concentration.entityCol)} — <strong>${esc(r.concentration.level)}</strong>.</div>
    </div>` : ''}

    ${r && r.proxies && r.proxies.length>0 ? `<div style="margin-bottom:20px;">
      <div style="font-family:'Oswald',sans-serif;font-size:14px;text-transform:uppercase;margin-bottom:6px;">3. Potential proxy signals</div>
      ${r.proxies.map(p=>`<div style="font-size:13px;">${esc(p.col)} — ${fmtPct(p.relDiff)} relative gap for the lowest-scoring group in ${esc(p.groupCol)}.</div>`).join('')}
    </div>` : ''}

    <div style="margin-bottom:20px;">
      <div style="font-family:'Oswald',sans-serif;font-size:14px;text-transform:uppercase;margin-bottom:6px;">4. Governance readiness</div>
      <div style="font-size:13px;margin-bottom:8px;">${score.answered} of ${score.total} controls in place (${fmtPct(score.pct)}).</div>
      ${GOVERNANCE_SECTIONS.map(s=>{
        const secAnswered = s.items.filter((_,i)=>state.govAnswers[`${s.key}-${i}`]).length;
        return `<div style="font-size:12.5px;color:var(--steel);">${esc(s.title)}: ${secAnswered}/${s.items.length}</div>`;
      }).join('')}
    </div>

    <div style="margin-bottom:20px;">
      <div style="font-family:'Oswald',sans-serif;font-size:14px;text-transform:uppercase;margin-bottom:6px;">5. Methodology</div>
      <div style="font-size:12.5px;color:var(--steel);line-height:1.6;">
        Disparate impact was screened using the four-fifths (80%) rule, the EEOC-derived threshold most widely used in US algorithmic bias audits, alongside a chi-square test of independence to flag whether the observed gap exceeds what sampling variation would explain. Concentration risk was measured with the Herfindahl-Hirschman Index, the standard antitrust and supply-chain concentration metric, using the conventional bands of under 1,500 (unconcentrated), 1,500–2,500 (moderately concentrated), and over 2,500 (highly concentrated). The governance checklist mirrors the four functions of the NIST AI Risk Management Framework and the management-system logic behind ISO/IEC 42001 certification. This tool performs an initial screening pass; it is not a substitute for a full audit by a qualified professional and does not constitute legal advice.
      </div>
    </div>

    <div style="border-top:1px solid var(--paper-line);padding-top:12px;font-size:11px;color:var(--steel);line-height:1.6;">
      This report is produced by <span class="fa-org-name">${esc(state.orgName) || '[Organization Name]'}</span> for internal risk-assessment purposes. It does not constitute legal, financial, or regulatory advice, and no warranty is made as to completeness or fitness for any specific compliance obligation. Organizations should consult qualified legal counsel and, where required, a certified third-party auditor before relying on these results for a regulatory filing or public disclosure.
    </div>
  </div>`;
  return out;
}

// ---------- Main render ----------
function render(){
  const app = document.getElementById('app');
  let body = '';
  if (state.tab==='data') body = renderDataTab();
  else if (state.tab==='analysis') body = renderAnalysisTab();
  else if (state.tab==='governance') body = renderGovernanceTab();
  else if (state.tab==='report') body = renderReportTab();

  app.innerHTML = renderHeaderNav() + `<div style="padding:24px;max-width:980px;margin:0 auto;">${body}</div>`;
  attachGlobalHandlers();
  if (state.tab==='data' && state.inputMode==='manual') renderManualGrid();
}

function attachGlobalHandlers(){
  document.querySelectorAll('[data-tab]').forEach(el=>{
    el.addEventListener('click', ()=>{ state.tab = el.dataset.tab; render(); });
  });

  // Data tab
  const modeUpload = document.querySelector('[data-action="mode-upload"]');
  if (modeUpload) modeUpload.addEventListener('click', ()=>{ state.inputMode='upload'; render(); });
  const modeManual = document.querySelector('[data-action="mode-manual"]');
  if (modeManual) modeManual.addEventListener('click', ()=>{ state.inputMode='manual'; render(); });

  const uploadClick = document.querySelector('[data-action="upload-click"]');
  if (uploadClick) uploadClick.addEventListener('click', ()=> document.getElementById('file-input').click());
  const fileInput = document.getElementById('file-input');
  if (fileInput) fileInput.addEventListener('change', (e)=>{ if (e.target.files[0]) handleFileInput(e.target.files[0]); });

  const dlTemplate = document.querySelector('[data-action="download-template"]');
  if (dlTemplate) dlTemplate.addEventListener('click', ()=> downloadBlob(TEMPLATE_CSV, 'first-audit-template.csv', 'text/csv'));
  const loadSample = document.querySelector('[data-action="load-sample"]');
  if (loadSample) loadSample.addEventListener('click', loadTemplateAsData);

  document.querySelectorAll('[data-action="toggle-group-col"]').forEach(el=>{
    el.addEventListener('change', ()=>{
      const c = el.dataset.col;
      if (state.selectedGroupCols.includes(c)) state.selectedGroupCols = state.selectedGroupCols.filter(x=>x!==c);
      else state.selectedGroupCols.push(c);
      render();
    });
  });
  const outcomeSel = document.querySelector('[data-field="outcomeCol"]');
  if (outcomeSel) outcomeSel.addEventListener('change', (e)=>{ state.outcomeCol = e.target.value; state.positiveValue=''; render(); });
  const posSel = document.querySelector('[data-field="positiveValue"]');
  if (posSel) posSel.addEventListener('change', (e)=>{ state.positiveValue = e.target.value; render(); });
  const entitySel = document.querySelector('[data-field="entityCol"]');
  if (entitySel) entitySel.addEventListener('change', (e)=>{ state.entityCol = e.target.value; render(); });
  const weightSel = document.querySelector('[data-field="weightCol"]');
  if (weightSel) weightSel.addEventListener('change', (e)=>{ state.weightCol = e.target.value; render(); });
  const runBtn = document.querySelector('[data-action="run-audit"]');
  if (runBtn) runBtn.addEventListener('click', runAudit);

  // Analysis tab
  const exportBtn = document.querySelector('[data-action="export-csv"]');
  if (exportBtn) exportBtn.addEventListener('click', exportResultsCsv);
  const gotoGov = document.querySelector('[data-action="goto-governance"]');
  if (gotoGov) gotoGov.addEventListener('click', ()=>{ state.tab='governance'; render(); });

  // Governance tab
  document.querySelectorAll('[data-action="gov-check"]').forEach(el=>{
    el.addEventListener('change', ()=>{ state.govAnswers[el.dataset.key] = el.checked; render(); });
  });
  const gotoReport = document.querySelector('[data-action="goto-report"]');
  if (gotoReport) gotoReport.addEventListener('click', ()=>{ state.tab='report'; render(); });

  // Report tab
  const orgInput = document.getElementById('org-name-input');
  if (orgInput) orgInput.addEventListener('input', (e)=>{
    state.orgName = e.target.value;
    document.querySelectorAll('.fa-org-name').forEach(n=>{ n.textContent = state.orgName || '[Organization Name]'; });
  });
  const printBtn = document.querySelector('[data-action="print-report"]');
  if (printBtn) printBtn.addEventListener('click', ()=>{
    // sync latest org name typed before print
    if (orgInput) state.orgName = orgInput.value;
    render();
    window.print();
  });
}

render();
</script>
</body>
</html>
