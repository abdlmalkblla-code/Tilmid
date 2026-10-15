# Tilmid
Txt

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#111827">
<title>ClassSpin | عجلة القسم</title>
<style>
:root{color-scheme:dark;--bg:#0b1020;--card:#151d31;--line:#29344c;--text:#f8fafc;--muted:#a5b4cf;--accent:#8b5cf6}
*{box-sizing:border-box}
body{margin:0;background:radial-gradient(circle at top,#20234b,var(--bg) 55%);color:var(--text);font-family:Arial,sans-serif;min-height:100vh}
header{text-align:center;padding:30px 16px 15px}
header h1{margin:0 0 10px;font-size:32px}
header p{color:var(--muted);margin:0}
main{width:min(100% - 24px,950px);margin:20px auto 50px;display:grid;gap:18px}
.card{background:#151d31ed;border:1px solid var(--line);border-radius:20px;padding:20px;min-width:0}
h2{font-size:20px;margin:0 0 16px}
label{display:block;color:var(--muted);margin:12px 0 7px}
input,textarea,select{width:100%;padding:13px;border-radius:11px;border:1px solid #394663;background:#0d1425;color:white;font:inherit}
textarea{min-height:135px;resize:vertical}
button{border:0;border-radius:11px;padding:12px 16px;background:var(--accent);color:white;font:inherit;font-weight:bold;cursor:pointer}
button:active{transform:scale(.98)}
button.secondary{background:#29344c}
button.danger{background:#9f2945}
button:disabled{opacity:.45;cursor:not-allowed}
.actions{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px}
.hint,.muted{color:var(--muted);font-size:13px;line-height:1.8}
.section-row{display:flex;align-items:center;justify-content:space-between;gap:10px;padding:12px 0;border-bottom:1px solid var(--line)}
.section-row:last-child{border-bottom:0}
.section-row strong{overflow-wrap:anywhere}
#stage{position:relative;width:min(100%,410px);aspect-ratio:1;margin:20px auto}
canvas{width:100%;height:100%;display:block}
.pointer{position:absolute;top:-5px;left:50%;transform:translateX(-50%);z-index:2;width:0;height:0;border-left:15px solid transparent;border-right:15px solid transparent;border-top:29px solid #fff;filter:drop-shadow(0 2px 3px #0008)}
.center{text-align:center}
#winner{font-size:25px;font-weight:bold;color:#c4b5fd;min-height:36px;margin:12px 0}
footer{text-align:center;color:var(--muted);font-size:12px;padding:20px}
</style>
</head>
<body>
<header>
  <h1>🎓 ClassSpin</h1>
  <p>اختار القسم، دوّر العجلة، وخلي الحظ يختار التلميذ!</p>
</header>

<main>
<section class="card">
<h2>📚 الأقسام ديالي</h2>
<label for="sectionName">اسم القسم</label>
<input id="sectionName" placeholder="مثال: جذع مشترك علوم إنسانية">
<label for="studentNames">أسماء التلاميذ (كل اسم فسطّر)</label>
<textarea id="studentNames" placeholder="محمد&#10;سارة&#10;يوسف&#10;مريم"></textarea>
<div class="actions">
<button id="saveSection">💾 حفظ القسم</button>
<button id="newSection" class="secondary">＋ قسم جديد</button>
</div>
<label for="sectionSelect">الأقسام المحفوظة</label>
<select id="sectionSelect"></select>
<div id="sectionList"></div>
<p class="hint">الأقسام محفوظة تلقائياً فهاد المتصفح والجهاز. ما تدخلش معلومات شخصية حساسة.</p>
</section>

<section class="card">
<h2>🎯 عجلة الأسماء</h2>
<label for="wheelSection">اختار القسم اللي بغيتي</label>
<select id="wheelSection"></select>
<div id="stage">
<div class="pointer"></div>
<canvas id="wheel" width="700" height="700"></canvas>
</div>
<div class="center">
<div id="winner" aria-live="polite">واجدين؟ 🚀</div>
<p id="status" class="muted">اختار قسماً فيه جوج تلاميذ على الأقل.</p>
<div class="actions" style="justify-content:center">
<button id="spin">🎡 دوّر العجلة</button>
<button id="next" class="secondary">تلميذ آخر</button>
<button id="reset" class="secondary">إعادة الاختيارات</button>
</div>
</div>
</section>
</main>
<footer>ClassSpin · أداة تعليمية لاختيار التلاميذ عشوائياً</footer>

<script>
const $ = id => document.getElementById(id);
const KEY = "classspin_sections_v1";
let sections = load();
let rotation = 0;
let spinning = false;
let drawn = [];
let activeId = null;
let lastWinner = "";

function load(){
  try{
    const x = JSON.parse(localStorage.getItem(KEY) || "[]");
    return Array.isArray(x) ? x : [];
  }catch{return []}
}
function persist(){
  localStorage.setItem(KEY, JSON.stringify(sections));
}
function uid(){
  return Date.now().toString(36)+Math.random().toString(36).slice(2,8);
}
function current(){
  return sections.find(s => s.id === activeId) || sections[0] || null;
}
function escapeHtml(s){
  return String(s).replace(/[&<>"']/g,c=>({
    "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
  }[c]));
}
function fillSelect(select, selected){
  select.innerHTML = "";
  sections.forEach(s=>{
    const o = document.createElement("option");
    o.value=s.id;
    o.textContent=s.name;
    select.appendChild(o);
  });
  if(selected && sections.some(s=>s.id===selected)) select.value=selected;
}
function renderSections(){
  fillSelect($("sectionSelect"),activeId);
  fillSelect($("wheelSection"),activeId);
  $("sectionList").innerHTML = sections.map(s=>`
    <div class="section-row">
      <div><strong>${escapeHtml(s.name)}</strong><div class="muted">${s.names.length} تلميذ</div></div>
      <div class="actions">
        <button class="secondary" data-edit="${s.id}">فتح</button>
        <button class="danger" data-delete="${s.id}">حذف</button>
      </div>
    </div>`).join("");
  if(!sections.length){
    $("sectionList").innerHTML='<p class="muted">مازال ما زدتي حتى قسم. زيد الأول من الفوق.</p>';
  }
  const s=current();
  if(s){
    $("sectionName").value=s.name;
    $("studentNames").value=s.names.join("\n");
  }
  drawWheel();
}
function saveSection(){
  const name=$("sectionName").value.trim();
  const names=[...new Set($("studentNames").value.split(/\n|,/).map(x=>x.trim()).filter(Boolean))];
  if(!name){alert("كتب اسم القسم أولاً.");return}
  if(!names.length){alert("دخل اسم تلميذ واحد على الأقل.");return}
  let s=sections.find(x=>x.id===activeId);
  if(s){s.name=name;s.names=names}
  else{
    s={id:uid(),name,names};
    sections.push(s);
    activeId=s.id;
  }
  drawn=drawn.filter(n=>s.names.includes(n));
  lastWinner="";
  persist();
  renderSections();
  $("status").textContent="تحفظ القسم بنجاح ✅";
}
function drawWheel(){
  const canvas=$("wheel");
  const ctx=canvas.getContext("2d");
  const s=current();
  const names=s?s.names:[];
  const w=canvas.width,c=w/2,r=w/2-8;
  ctx.clearRect(0,0,w,w);
  if(!names.length){
    ctx.beginPath();ctx.arc(c,c,r,0,Math.PI*2);
    ctx.fillStyle="#29344c";ctx.fill();
    ctx.fillStyle="#fff";ctx.textAlign="center";
    ctx.font="bold 27px Arial";ctx.fillText("زيد قسمك الأول",c,c);
    updateButtons();
    return;
  }
  const colors=["#8b5cf6","#0ea5e9","#10b981","#f59e0b","#ec4899","#ef4444","#6366f1","#14b8a6","#f97316","#a855f7"];
  const step=2*Math.PI/names.length;
  names.forEach((name,i)=>{
    const start=rotation+i*step-Math.PI/2;
    const end=start+step;
    ctx.beginPath();ctx.moveTo(c,c);ctx.arc(c,c,r,start,end);ctx.closePath();
    ctx.fillStyle=colors[i%colors.length];ctx.fill();
    ctx.strokeStyle="#ffffff";ctx.lineWidth=3;ctx.stroke();
    ctx.save();ctx.translate(c,c);ctx.rotate(start+step/2);
    ctx.textAlign="right";ctx.fillStyle="#fff";
    let font=names.length>24?15:names.length>14?19:25;
    ctx.font="bold "+font+"px Arial";
    let label=name.length>17?name.slice(0,16)+"…":name;
    ctx.shadowColor="#0006";ctx.shadowBlur=3;
    ctx.fillText(label,r-22,8);
    ctx.restore();
  });
  ctx.beginPath();ctx.arc(c,c,45,0,Math.PI*2);
  ctx.fillStyle="#111827";ctx.fill();
  ctx.strokeStyle="#fff";ctx.lineWidth=4;ctx.stroke();
  ctx.fillStyle="#fff";ctx.textAlign="center";ctx.font="bold 22px Arial";
  ctx.fillText("🎓",c,c+8);
  updateButtons();
}
function updateButtons(){
  const s=current();
  const remaining=s?s.names.filter(n=>!drawn.includes(n)).length:0;
  $("spin").disabled=spinning||!s||s.names.length<2||remaining===0;
  $("next").disabled=spinning||!s||s.names.length<2||remaining===0;
  $("reset").disabled=spinning||!drawn.length;
  if(s && s.names.length<2) $("status").textContent="خاصك تزيد جوج تلاميذ على الأقل.";
  else if(s && remaining===0) $("status").textContent="دوزو التلاميذ كاملين! تقدر تعاود الاختيارات.";
  else if(s) $("status").textContent="باقي "+remaining+" تلميذ ما تختاروش فهاد الدورة.";
  else $("status").textContent="زيد قسم واختار الأسماء ديالو.";
}
function pick(){
  const s=current();
  if(!s||spinning)return;
  const remaining=s.names.filter(n=>!drawn.includes(n));
  if(remaining.length===0){updateButtons();return}
  const winner=remaining[Math.floor(Math.random()*remaining.length)];
  const index=s.names.indexOf(winner);
  const step=2*Math.PI/s.names.length;
  const targetModulo=((-(index+0.5)*step)%(2*Math.PI)+2*Math.PI)%(2*Math.PI);
  const currentModulo=((rotation%(2*Math.PI))+2*Math.PI)%(2*Math.PI);
  const delta=(targetModulo-currentModulo+2*Math.PI)%(2*Math.PI);
  const extra=5+Math.floor(Math.random()*3);
  const start=rotation;
  const end=rotation+extra*2*Math.PI+delta;
  const duration=4200;
  const began=performance.now();
  spinning=true;
  $("winner").textContent="🎡 العجلة كتدور...";
  updateButtons();
  function frame(now){
    const t=Math.min(1,(now-began)/duration);
    const eased=1-Math.pow(1-t,4);
    rotation=start+(end-start)*eased;
    drawWheel();
    if(t<1){requestAnimationFrame(frame)}
    else{
      rotation=end%(2*Math.PI);
      drawn.push(winner);
      lastWinner=winner;
      spinning=false;
      $("winner").textContent="🎉 "+winner;
      $("status").textContent="هو المختار! جا الدور ديالو يجاوب.";
      drawWheel();
    }
  }
  requestAnimationFrame(frame);
}
$("saveSection").addEventListener("click",saveSection);
$("newSection").addEventListener("click",()=>{
  activeId=null;
  $("sectionName").value="";
  $("studentNames").value="";
  $("sectionSelect").value="";
  $("winner").textContent="واجدين؟ 🚀";
  drawWheel();
});
$("sectionSelect").addEventListener("change",e=>{
  activeId=e.target.value||null;drawn=[];lastWinner="";renderSections();
});
$("wheelSection").addEventListener("change",e=>{
  activeId=e.target.value||null;drawn=[];lastWinner="";
  const s=current();
  if(s){$("sectionSelect").value=s.id;$("sectionName").value=s.name;$("studentNames").value=s.names.join("\n")}
  $("winner").textContent="واجدين؟ 🚀";renderSections();
});
$("sectionList").addEventListener("click",e=>{
  const edit=e.target.closest("[data-edit]");
  const del=e.target.closest("[data-delete]");
  if(edit){
    activeId=edit.dataset.edit;drawn=[];lastWinner="";
    $("winner").textContent="واجدين؟ 🚀";renderSections();
  }
  if(del){
    const s=sections.find(x=>x.id===del.dataset.delete);
    if(s&&confirm("واش متأكد بغيتي تحذف القسم "+s.name+"؟")){
      sections=sections.filter(x=>x.id!==s.id);
      if(activeId===s.id)activeId=sections[0]?.id||null;
      drawn=[];lastWinner="";persist();renderSections();
    }
  }
});
$("spin").addEventListener("click",pick);
$("next").addEventListener("click",pick);
$("reset").addEventListener("click",()=>{
  drawn=[];lastWinner="";
  $("winner").textContent="رجعنا من الأول 🔄";
  drawWheel();
});
renderSections();
</script>
</body>
</html>
