# BiSaiChouQian
乒乓球分组抽签工具规则：① 同公司选手必须分到不同组；② 同公司人数尽量平均分到上半区(A–D)与下半区(E–H)；③ 已确定的种子选手固定在指定组，只补该组其余名额；④ 每次随机可重抽。所有计算都在本机浏览器完成，不上传任何数据。
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>八组随机抽签工具</title>
<style>
  :root{
    --navy:#16263f; --navy2:#1f3a5f; --gold:#c9a24b; --gold-l:#e8d49a;
    --up:#eaf2fb; --up-line:#3a6ea5; --down:#eef6ee; --down-line:#4a8a55;
    --bg:#f2f4f7; --card:#fff; --text:#222; --muted:#6b7686; --danger:#c0392b;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  body{font-family:"PingFang SC","Microsoft YaHei",system-ui,sans-serif;background:var(--bg);color:var(--text);padding:20px;line-height:1.5}
  .wrap{max-width:1180px;margin:0 auto}
  header{background:linear-gradient(135deg,var(--navy),var(--navy2));color:#fff;border-radius:14px;padding:22px 26px;margin-bottom:18px;box-shadow:0 4px 14px rgba(20,40,70,.18)}
  header h1{font-size:22px;letter-spacing:2px;display:flex;align-items:center;gap:10px}
  header h1 .dot{width:10px;height:10px;background:var(--gold);border-radius:50%}
  header p{color:#cdd9e8;font-size:13px;margin-top:6px}
  .panel{background:var(--card);border-radius:12px;padding:18px 20px;margin-bottom:18px;box-shadow:0 2px 8px rgba(0,0,0,.06)}
  .panel h2{font-size:16px;color:var(--navy);margin-bottom:12px;border-left:4px solid var(--gold);padding-left:10px}
  textarea{width:100%;height:220px;font-family:Consolas,monospace;font-size:13px;padding:12px;border:1px solid #cdd4dd;border-radius:8px;resize:vertical;line-height:1.8}
  textarea:focus{outline:none;border-color:var(--up-line)}
  .fmt{font-size:12.5px;color:var(--muted);margin-top:8px;background:#f7f9fb;border-radius:8px;padding:10px 12px}
  .fmt code{background:#e9eef4;padding:1px 6px;border-radius:4px;color:var(--navy2)}
  .btn-row{display:flex;flex-wrap:wrap;gap:10px;margin-top:14px}
  button{border:none;border-radius:8px;padding:10px 22px;font-size:15px;cursor:pointer;font-weight:600;transition:.15s}
  .btn-primary{background:var(--navy2);color:#fff}
  .btn-primary:hover{background:var(--navy)}
  .btn-gold{background:var(--gold);color:#fff}
  .btn-gold:hover{background:#b38f3a}
  .btn-ghost{background:#eef1f5;color:var(--navy2)}
  .btn-ghost:hover{background:#e2e8ee}
  .warn{color:var(--danger);font-size:13.5px;margin-top:10px;display:none}
  /* 结果区 */
  .half-title{font-size:15px;font-weight:700;margin:14px 0 10px;padding:6px 12px;border-radius:8px}
  .half-up{background:var(--up);border-left:4px solid var(--up-line);color:var(--up-line)}
  .half-down{background:var(--down);border-left:4px solid var(--down-line);color:var(--down-line)}
  .grid{display:grid;grid-template-columns:repeat(4,1fr);gap:14px}
  @media(max-width:900px){.grid{grid-template-columns:repeat(2,1fr)}}
  .group-card{background:#fff;border:1px solid #dde3ea;border-radius:10px;overflow:hidden}
  .group-head{padding:10px 14px;font-weight:700;font-size:16px;color:#fff;display:flex;justify-content:space-between;align-items:center}
  .group-card.up .group-head{background:var(--up-line)}
  .group-card.down .group-head{background:var(--down-line)}
  .group-head .cnt{font-size:13px;font-weight:500;background:rgba(255,255,255,.22);padding:2px 10px;border-radius:20px}
  .group-body{padding:10px 12px;min-height:110px}
  .player{display:flex;justify-content:space-between;align-items:center;padding:6px 8px;border-radius:6px;margin-bottom:4px;font-size:14px;background:#f7f9fb}
  .player .company{font-size:12px;color:var(--muted)}
  .seed-tag{background:var(--gold);color:#fff;font-size:11px;padding:1px 8px;border-radius:10px;margin-left:6px}
  .empty{color:#aab3bd;font-size:13px;text-align:center;padding:24px 0}
  /* 校验表 */
  table{width:100%;border-collapse:collapse;font-size:13.5px;margin-top:10px}
  th,td{border:1px solid #dde3ea;padding:8px 10px;text-align:center}
  th{background:var(--navy);color:#fff}
  td.good{color:var(--down-line);font-weight:600}
  td.bad{color:var(--danger);font-weight:600}
  .stat-line{font-size:13.5px;color:var(--muted);margin-top:10px}
  .stat-line b{color:var(--navy)}
  #result{display:none}
  /* 循环赛程 */
  .sched-group{border:1px solid #dde3ea;border-radius:10px;margin-top:14px;overflow:hidden}
  .sched-head{background:var(--navy2);color:#fff;padding:10px 16px;font-weight:700;display:flex;justify-content:space-between;align-items:center}
  .sched-head .meta{font-size:12.5px;font-weight:400;color:#cdd9e8}
  .sched-body{padding:8px 16px 14px}
  .round-row{display:flex;align-items:baseline;gap:12px;padding:8px 0;border-bottom:1px dashed #e6ebf0;flex-wrap:wrap}
  .round-row:last-child{border-bottom:none}
  .round-label{flex:0 0 64px;font-weight:700;color:var(--navy2);font-size:14px}
  .match{font-size:14px;background:#f7f9fb;border-radius:6px;padding:5px 12px;margin:3px 8px 3px 0}
  .match .vs{color:var(--gold);font-weight:700;margin:0 6px}
  .match .co{font-size:12px;color:var(--muted)}
  .bye{font-size:13px;color:var(--muted);font-style:italic}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1><span class="dot"></span>八组随机抽签工具</h1>
    <p>规则：8 组（A–H）人数尽量均衡；同公司选手不得同组；同公司尽量平均分到上半区(A–D)/下半区(E–H)；种子选手固定进指定组，仅补该组其余名额；可反复重抽。</p>
  </header>

  <!-- ===== 录入面板 ===== -->
  <div class="panel">
    <h2>选手录入（每行一人，逗号分隔）</h2>
    <textarea id="input" placeholder="普通选手：姓名,公司&#10;种子选手：姓名,公司,种子,A&#10;示例：&#10;王强,中原公司&#10;李娜,中原公司,种子,B&#10;张伟,华北公司"></textarea>
    <div class="fmt">
      格式：<code>姓名,公司</code> 为普通选手；<code>姓名,公司,种子,组号</code> 为固定种子（组号填 A–H，大小写均可）。支持中英文逗号、空格。空行自动忽略。
    </div>
    <div class="btn-row">
      <button class="btn-primary" onclick="runDraw()">开始抽签</button>
      <button class="btn-gold" onclick="runDraw(true)">重新抽取</button>
      <button class="btn-ghost" onclick="loadDemo()">填入示例</button>
      <button class="btn-ghost" onclick="clearAll()">清空</button>
    </div>
    <div class="warn" id="warn"></div>
  </div>

  <!-- ===== 结果面板 ===== -->
  <div class="panel" id="result">
    <h2>抽签结果 <span style="font-size:13px;color:var(--muted);font-weight:400" id="meta"></span></h2>
    <div class="half-title half-up">上半区 A – D</div>
    <div class="grid" id="gridUp"></div>
    <div class="half-title half-down">下半区 E – H</div>
    <div class="grid" id="gridDown"></div>

    <h2 style="margin-top:22px">合规校验</h2>
    <div class="stat-line" id="statLine"></div>
    <table id="checkTable">
      <thead><tr><th>公司</th><th>总人数</th><th>上半区(A–D)</th><th>下半区(E–H)</th><th>同组冲突</th><th>分布评价</th></tr></thead>
      <tbody></tbody>
    </table>

    <h2 style="margin-top:22px">小组循环赛次序（组内每人互赛一场，固定位轮转法）</h2>
    <div class="stat-line">每轮只进行 1 场，逐轮打完组内所有对阵（共 n(n−1)/2 轮）；排程优先安排刚出场过的选手多休息，避免背靠背连续作战。</div>
    <div id="scheduleWrap"></div>
  </div>
</div>

<script>
const GROUPS = ['A','B','C','D','E','F','G','H'];
const UP = new Set(['A','B','C','D']);

/* ---------- 解析输入 ---------- */
function parseInput(){
  const raw = document.getElementById('input').value;
  const players = [];
  const errors = [];
  raw.split(/\r?\n/).forEach((line,idx)=>{
    line = line.trim();
    if(!line) return;
    const parts = line.split(/[,，]/).map(s=>s.trim()).filter(Boolean);
    if(parts.length<2){ errors.push(`第${idx+1}行格式不完整：${line}`); return; }
    const name = parts[0], company = parts[1];
    let seed=false, fixedGroup=null;
    if(parts[2] && /种子|seed/i.test(parts[2])){
      seed = true;
      if(parts[3]){
        fixedGroup = parts[3].toUpperCase();
        if(!GROUPS.includes(fixedGroup)) errors.push(`第${idx+1}行组号非法：${parts[3]}，应为 A–H`);
      } else {
        errors.push(`第${idx+1}行种子选手缺少指定组：${line}`);
      }
    }
    players.push({name, company, seed, fixedGroup});
  });
  return {players, errors};
}

/* ---------- 工具 ---------- */
function shuffle(arr){
  for(let i=arr.length-1;i>0;i--){
    const j = Math.floor(Math.random()*(i+1));
    [arr[i],arr[j]]=[arr[j],arr[i]];
  }
  return arr;
}

/* ---------- 小组单循环：每轮1场，逐轮进行；优先安排休息最久的选手 ---------- */
function linearSchedule(list){
  const n = list.length;
  if(n < 2) return [];
  const remaining = [];
  for(let i=0;i<n;i++) for(let j=i+1;j<n;j++) remaining.push([list[i],list[j]]);
  const lastPlay = {};
  list.forEach(p => lastPlay[p.name] = -99);
  const schedule = [];
  const total = remaining.length;
  for(let round=0; round<total; round++){
    let best = null, bestScore = -Infinity;
    for(const pr of remaining){
      const [a,b] = pr;
      const gap = (round-lastPlay[a.name]) + (round-lastPlay[b.name]);
      const score = gap + Math.random()*0.5; // 同分时随机打破
      if(score > bestScore){ bestScore = score; best = pr; }
    }
    schedule.push(best);
    lastPlay[best[0].name] = round;
    lastPlay[best[1].name] = round;
    remaining.splice(remaining.indexOf(best),1);
  }
  return schedule;
}

/* ---------- 单次分配（贪心+随机），返回 null 表示该次不可行 ---------- */
function attempt(players, cap){
  const members = Object.fromEntries(GROUPS.map(g=>[g,[]]));
  const inGroup = {};   // company -> Set(groups)
  const half = {};      // company -> {up,down}

  // 1) 先放种子
  for(const p of players){
    if(!p.seed) continue;
    const g = p.fixedGroup;
    if((inGroup[p.company]||new Set()).has(g)) return null;      // 同公司种子撞组
    if(members[g].length >= cap[g]) return null;                // 种子超员
    members[g].push(p);
    (inGroup[p.company] = inGroup[p.company]||new Set()).add(g);
    half[p.company] = half[p.company]||{up:0,down:0};
    if(UP.has(g)) half[p.company].up++; else half[p.company].down++;
  }

  // 2) 打乱非种子，贪心入组
  const rest = shuffle(players.filter(p=>!p.seed));
  for(const p of rest){
    const cands = GROUPS.filter(g=>
      members[g].length < cap[g] && !(inGroup[p.company]||new Set()).has(g)
    );
    if(!cands.length) return null;
    // 软目标：优先补足该公司人数较少的半区；同半区内优先人数少的组（保均衡）
    const h = half[p.company]||{up:0,down:0};
    const preferUp = h.up <= h.down;
    cands.sort((a,b)=>{
      const da = (UP.has(a)===preferUp?0:1) + members[a].length/100;
      const db = (UP.has(b)===preferUp?0:1) + members[b].length/100;
      return da-db;
    });
    const chosen = cands[0];
    members[chosen].push(p);
    (inGroup[p.company] = inGroup[p.company]||new Set()).add(chosen);
    half[p.company] = half[p.company]||{up:0,down:0};
    if(UP.has(chosen)) half[p.company].up++; else half[p.company].down++;
  }
  return {members, half};
}

/* ---------- 评分：人数极差*10 + 各公司上下半区偏差之和 ---------- */
function score(members, half){
  const sizes = GROUPS.map(g=>members[g].length);
  const spread = Math.max(...sizes)-Math.min(...sizes);
  let split = 0;
  for(const c in half) split += Math.abs(half[c].up-half[c].down);
  return spread*10 + split;
}

/* ---------- 主流程 ---------- */
function runDraw(isRedraw){
  const {players, errors} = parseInput();
  const warn = document.getElementById('warn');
  warn.style.display='none';

  if(players.length===0){ showWarn('请先录入选手。'); return; }
  if(errors.length){ showWarn(errors.join('；')); return; }

  // 同公司人数 > 8 必然无法满足“不同组”
  const cnt = {};
  players.forEach(p=>cnt[p.company]=(cnt[p.company]||0)+1);
  for(const c in cnt) if(cnt[c]>8){ showWarn(`公司「${c}」有 ${cnt[c]} 人，超过 8 组，不可能全部分到不同组，请核减。`); return; }

  // 每组目标人数：总人数均分，余数分给前几组
  const total = players.length;
  const base = Math.floor(total/8), extra = total%8;
  const cap = {};
  GROUPS.forEach((g,i)=>cap[g]=base+(i<extra?1:0));

  // 检查种子占用是否超出该组容量
  const seedCount = {};
  players.forEach(p=>{ if(p.seed) seedCount[p.fixedGroup]=(seedCount[p.fixedGroup]||0)+1; });
  for(const g in seedCount){
    if(seedCount[g]>cap[g]){ showWarn(`组 ${g} 容量仅 ${cap[g]} 人，但已固定 ${seedCount[g]} 名种子，请调整。`); return; }
  }

  // 多次随机取最优
  let best=null, bestScore=Infinity;
  const TRIALS = 3000;
  for(let t=0;t<TRIALS;t++){
    const res = attempt(players, cap);
    if(!res) continue;
    const s = score(res.members, res.half);
    if(s<bestScore){ bestScore=s; best=res; }
  }

  if(!best){ showWarn('在当前约束下未能排出可行方案，请检查种子固定位置或同公司人数。'); return; }
  render(best, total);
}

function showWarn(msg){
  const w = document.getElementById('warn');
  w.textContent = '⚠ '+msg;
  w.style.display='block';
}

/* ---------- 渲染 ---------- */
function render(best, total){
  document.getElementById('result').style.display='block';
  document.getElementById('meta').textContent = `（共 ${total} 人，每组目标 ${Math.floor(total/8)}–${Math.ceil(total/8)} 人；点「重新抽取」可换一组随机结果）`;

  const gridUp = document.getElementById('gridUp');
  const gridDown = document.getElementById('gridDown');
  gridUp.innerHTML=''; gridDown.innerHTML='';

  GROUPS.forEach(g=>{
    const card = document.createElement('div');
    card.className = 'group-card '+(UP.has(g)?'up':'down');
    const list = best.members[g];
    let rows = list.length
      ? list.map(p=>`<div class="player"><span>${escapeHtml(p.name)}${p.seed?'<span class="seed-tag">种子</span>':''}</span><span class="company">${escapeHtml(p.company)}</span></div>`).join('')
      : '<div class="empty">（空）</div>';
    card.innerHTML = `<div class="group-head"><span>第 ${g} 组</span><span class="cnt">${list.length} 人</span></div><div class="group-body">${rows}</div>`;
    (UP.has(g)?gridUp:gridDown).appendChild(card);
  });

  // 校验表
  const tbody = document.querySelector('#checkTable tbody');
  tbody.innerHTML='';
  let conflict = 0, badSplit = 0;
  const companies = Object.keys(best.half).sort();
  companies.forEach(c=>{
    const {up,down} = best.half[c];
    // 同组冲突：检查每组内是否出现两次
    let clash = false;
    GROUPS.forEach(g=>{
      const n = best.members[g].filter(p=>p.company===c).length;
      if(n>1) clash=true;
    });
    if(clash) conflict++;
    const diff = Math.abs(up-down);
    if(diff>1) badSplit++;
    const evalTxt = diff===0?'完美均分':(diff===1?'基本均衡':'偏差较大');
    tbody.innerHTML += `<tr><td>${escapeHtml(c)}</td><td>${up+down}</td><td>${up}</td><td>${down}</td>
      <td class="${clash?'bad':'good'}">${clash?'有冲突':'无'}</td>
      <td class="${diff>1?'bad':'good'}">${evalTxt}</td></tr>`;
  });

  const sizes = GROUPS.map(g=>best.members[g].length);
  document.getElementById('statLine').innerHTML =
    `各组人数：<b>${sizes.join(' / ')}</b>（极差 ${Math.max(...sizes)-Math.min(...sizes)}）；`+
    `同组冲突 <b class="${conflict?'bad':'good'}">${conflict} 处</b>；上下半区偏差>1 的公司 <b>${badSplit} 家</b>。`;

  // ===== 小组循环赛次序（每轮1场，逐轮进行） =====
  const wrap = document.getElementById('scheduleWrap');
  wrap.innerHTML = '';
  GROUPS.forEach(g=>{
    const list = best.members[g];
    if(list.length < 2) return;
    const schedule = linearSchedule(list);
    const box = document.createElement('div');
    box.className = 'sched-group';
    let html = `<div class="sched-head"><span>第 ${g} 组循环赛</span><span class="meta">${list.length} 人 · 共 ${schedule.length} 轮（每轮 1 场）</span></div><div class="sched-body">`;
    schedule.forEach(([a,b], idx)=>{
      html += `<div class="round-row"><span class="round-label">第${idx+1}轮</span>`
        + `<span class="match">${escapeHtml(a.name)}<span class="co">（${escapeHtml(a.company)}）</span><span class="vs">VS</span>${escapeHtml(b.name)}<span class="co">（${escapeHtml(b.company)}）</span></span>`
        + `</div>`;
    });
    html += `</div>`;
    box.innerHTML = html;
    wrap.appendChild(box);
  });
}

function escapeHtml(s){return String(s).replace(/[&<>"']/g,m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]));}

/* ---------- 示例 / 清空 ---------- */
function loadDemo(){
  document.getElementById('input').value =
`王强,中原公司,种子,A
李娜,中原公司,种子,E
张伟,华北公司
刘芳,华北公司
陈杰,华北公司
赵敏,华北公司
孙磊,华东公司
周婷,华东公司
吴涛,华东公司
郑爽,华南公司
冯雪,华南公司
褚亮,西南公司
卫红,西南公司
蒋勇,西南公司
韩梅,西北公司
杨光,西北公司
朱琳,东北公司
秦宇,东北公司
许晨,华中公司
何静,华中公司
罗刚,中原公司
高燕,华北公司
林浩,华东公司
梁雪,华南公司
`;
  document.getElementById('warn').style.display='none';
}
function clearAll(){
  document.getElementById('input').value='';
  document.getElementById('result').style.display='none';
  document.getElementById('warn').style.display='none';
}
</script>
</body>
</html>
