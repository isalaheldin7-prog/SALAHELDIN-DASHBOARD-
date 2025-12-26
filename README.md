# SALAHELDIN-DASHBOARD-
"لوحة تحكم لتسجيل وتحليل الصفقات يدويًا مع إحصائيات ورسم بياني، مجاني ومناسب للجوال."
!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SALAHELDIN DASHBOARD</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
:root{--bg:#090b0f;--card:#12151c;--blue:#3067ff;--green:#26a69a;--red:#ef5350;--text:#e0e0e0}
body{margin:0;font-family:sans-serif;direction:rtl;background:var(--bg);color:var(--text)}
.header{display:flex;align-items:center;justify-content:space-between;padding:20px;background:var(--card);border-bottom:1px solid var(--blue)}
.header b{font-size:18px;letter-spacing:1px;color:var(--blue)}
.dashboard{display:grid;grid-template-columns:repeat(2, 1fr);gap:10px;padding:15px}
.stat-card{background:var(--card);padding:15px;border-radius:12px;text-align:center;border:1px solid #1e222d}
.stat-card h4{margin:0;font-size:10px;color:#888;text-transform:uppercase}
.stat-card b{font-size:20px;display:block;margin-top:5px}
.main-chart{background:var(--card);margin:15px;padding:15px;border-radius:12px;border:1px solid #1e222d}
.form-section{background:var(--card);margin:15px;padding:20px;border-radius:12px}
input, select, textarea, button{width:100%;padding:12px;margin:8px 0;border-radius:8px;border:1px solid #2a2e39;background:#1e222d;color:#fff;box-sizing:border-box}
textarea{height:70px}
button{background:var(--blue);border:none;font-weight:bold;cursor:pointer;transition:0.3s}
button:hover{opacity:0.8}
.trades-list{margin:15px;overflow-x:auto}
table{width:100%;border-collapse:collapse;background:var(--card);font-size:13px}
th, td{padding:12px;text-align:center;border-bottom:1px solid #1e222d}
.win{color:var(--green)} .loss{color:var(--red)}
.note-text{font-size:11px;color:#aaa;max-width:120px;display:inline-block;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
</style>
</head>
<body>

<div class="header">
  <b>SALAHELDIN DASHBOARD 🚀</b>
  <small style="color:#888">إدارة الاستثمارات</small>
</div>

<div class="dashboard">
  <div class="stat-card"><h4>عدد الصفقات</h4><b id="tCount">0</b></div>
  <div class="stat-card"><h4>نسبة النجاح</h4><b id="winRate" class="win">0%</b></div>
  <div class="stat-card"><h4>صافي الأرباح</h4><b id="net">$0</b></div>
  <div class="stat-card"><h4>عامل الربح</h4><b id="pf">0.0</b></div>
</div>

<div class="main-chart">
  <canvas id="equityChart" height="180"></canvas>
</div>

<div class="form-section">
  <h3>➕ إضافة عملية تداول</h3>
  <input id="asset" placeholder="الأصل (مثلاً: الذهب، BTC، تسلا)">
  <select id="type">
    <option value="buy">شراء (Long)</option>
    <option value="sell">بيع (Short)</option>
  </select>
  <div style="display:flex; gap:10px;">
    <input id="entry" type="number" placeholder="سعر الدخول">
    <input id="exit" type="number" placeholder="سعر الخروج">
  </div>
  <input id="risk" type="number" placeholder="المخاطرة الفعيلة بالدولار ($)">
  <textarea id="note" placeholder="ملاحظات الصفقة (لماذا اخترت هذا الدخول؟)"></textarea>
  <button onclick="addTrade()">حفظ في لوحة تحكم صلاح الدين</button>
</div>

<div class="trades-list">
  <table>
    <thead><tr><th>الأصل</th><th>R:R</th><th>النتيجة</th><th>الملاحظة</th></tr></thead>
    <tbody id="rows"></tbody>
  </table>
  <button style="background:#ef5350; margin-top:20px; font-size:12px;" onclick="clearAll()">🗑️ مسح السجل بالكامل</button>
</div>

<script>
let trades = JSON.parse(localStorage.getItem("salaheldin_db")) || [];
let chart;

function addTrade(){
  const a=asset.value, t=type.value, e=+entry.value, x=+exit.value, r=+risk.value, n=note.value;
  if(!a||!e||!x||!r) return alert("يرجى إدخال البيانات كاملة");
  
  // حساب الربح والخسارة بناءً على النسبة المئوية للمخاطرة
  const pnlPercent = t==="buy" ? (x-e)/e : (e-x)/e;
  const realPnl = (pnlPercent * (e/r)) * r; 
  
  trades.push({
    a, 
    pnl: realPnl, 
    rr: (realPnl/r).toFixed(2), 
    note: n || "بدون ملاحظات"
  });
  
  localStorage.setItem("salaheldin_db", JSON.stringify(trades));
  asset.value=""; entry.value=""; exit.value=""; risk.value=""; note.value="";
  render();
}

function render(){
  rows.innerHTML="";
  let wins=0, net=0, gLoss=0, gWin=0, equity=[0];
  
  trades.slice().reverse().forEach(t=>{
    net += t.pnl;
    equity.push(net);
    if(t.pnl > 0) { wins++; gWin += t.pnl; } else { gLoss += Math.abs(t.pnl); }
    
    rows.innerHTML += `
      <tr>
        <td><b>${t.a}</b></td>
        <td>${t.rr}</td>
        <td class="${t.pnl>=0?'win':'loss'}">${t.pnl.toFixed(2)}$</td>
        <td><span class="note-text" title="${t.note}">${t.note}</span></td>
      </tr>`;
  });

  tCount.textContent = trades.length;
  winRate.textContent = trades.length ? ((wins/trades.length)*100).toFixed(1) + "%" : "0%";
  document.getElementById("net").textContent = "$" + net.toFixed(2);
  pf.textContent = gLoss > 0 ? (gWin/gLoss).toFixed(2) : gWin.toFixed(1);
  
  drawChart(equity);
}

function drawChart(data){
  const ctx = document.getElementById("equityChart").getContext("2d");
  if(chart) chart.destroy();
  chart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: data.map((_,i)=> i),
      datasets: [{
        data: data,
        borderColor: '#3067ff',
        backgroundColor: 'rgba(48, 103, 255, 0.1)',
        fill: true,
        tension: 0.4,
        pointRadius: 2
      }]
    },
    options: { 
      plugins: { legend: { display: false } }, 
      scales: { x: { display: false }, y: { grid: { color: '#1e222d' } } } 
    }
  });
}

function clearAll(){
  if(confirm("يا صلاح الدين، هل أنت متأكد من حذف السجل بالكامل؟")) {
    localStorage.removeItem("salaheldin_db");
    trades=[];
    render();
  }
}
render();
</script>
</body>
</html>
