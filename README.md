<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Engineering Graphics 3D Projection Solver</title>
<style>
*{box-sizing:border-box} body{margin:0;font-family:Arial,Helvetica,sans-serif;background:#eef3f8;color:#17202a}
header{background:linear-gradient(135deg,#0b4f8a,#1769aa);color:white;padding:18px 20px;box-shadow:0 2px 10px #0002}
h1{margin:0;font-size:25px} header p{margin:6px 0 0;opacity:.9}
.wrap{max-width:1450px;margin:auto;padding:16px;display:grid;grid-template-columns:330px 1fr;gap:16px}
.card{background:white;border-radius:12px;padding:16px;box-shadow:0 3px 14px #0001}
h2{font-size:18px;margin:0 0 12px}.group{margin-bottom:13px}
label{display:block;font-weight:700;font-size:13px;margin-bottom:5px}
input,select,button{width:100%;padding:9px;border:1px solid #c9d3dd;border-radius:7px;font-size:14px}
button{cursor:pointer;border:0;font-weight:700;margin-top:7px}
.primary{background:#0b63ce;color:white}.secondary{background:#e8eef5;color:#123}
.row{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.note{font-size:12px;color:#5c6873;line-height:1.45;margin-top:10px}
.results{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-top:12px}
.metric{background:#f3f7fb;border-radius:8px;padding:10px;text-align:center}
.metric b{display:block;font-size:19px;color:#0b63ce}.metric span{font-size:11px;color:#5c6873}
.stage{background:white;border-radius:12px;padding:12px;box-shadow:0 3px 14px #0001}
.toolbar{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:10px}
.toolbar button{width:auto;padding:8px 12px;margin:0}
.viewgrid{display:grid;grid-template-columns:1.35fr 1fr;gap:12px}
.panel{border:1px solid #d5dde5;border-radius:9px;overflow:hidden;background:#fbfdff}
.panel h3{font-size:14px;margin:0;padding:9px 12px;background:#edf3f8}
canvas{display:block;width:100%;height:auto;background:white}
.steps{padding:12px;font-size:13px;line-height:1.55}
.steps ol{margin:5px 0 0 20px;padding:0}
.legend{display:flex;gap:15px;flex-wrap:wrap;padding:8px 12px;font-size:12px}
.dot{display:inline-block;width:12px;height:3px;vertical-align:middle;margin-right:5px}
.footer{text-align:center;color:#64717d;font-size:12px;padding:12px}
.developer-credit{margin-top:5px;font-size:11px;letter-spacing:.25px;color:#7a8793}
.developer-credit span{font-weight:700;color:#1769aa}
@media(max-width:950px){.wrap{grid-template-columns:1fr}.viewgrid{grid-template-columns:1fr}}
@media(max-width:600px){.results{grid-template-columns:1fr 1fr}h1{font-size:20px}}
</style>
</head>
<body>
<header>
<h1>📐 Engineering Graphics — 3D Projection Solver</h1>
<p>Enter engineering-drawing parameters → calculate inclinations → generate 3D model + 2D projections.</p>
</header>

<div class="wrap">
<section class="card">
<h2>⚙️ Line Projection Inputs</h2>

<div class="group"><label>True Length AB (mm)</label><input id="tl" type="number" value="80" min="1" step="0.1"></div>

<div class="row">
<div class="group"><label>Front View (FV) (mm)</label><input id="fv" type="number" value="50" min="0.1" step="0.1"></div>
<div class="group"><label>Top View (TV) (mm)</label><input id="tv" type="number" value="70" min="0.1" step="0.1"></div>
</div>

<div class="group"><label>Point B condition</label>
<select id="bcond"><option value="vp">B lies in VP</option><option value="hp">B lies in HP</option><option value="both">B lies on XY (HP & VP)</option></select></div>

<div class="group"><label>Point A condition</label>
<select id="acond"><option value="hp">A lies in HP</option><option value="vp">A lies in VP</option><option value="free">A is free</option></select></div>

<div class="group"><label>Drawing scale</label>
<select id="scale"><option value="1">1 : 1</option><option value=".75">3 : 4</option><option value=".5">1 : 2</option><option value=".25">1 : 4</option></select></div>

<button class="primary" onclick="generate()">GENERATE 3D + 2D DIAGRAM</button>
<button class="secondary" onclick="resetAll()">RESET</button>
<button class="secondary" onclick="window.print()">🖨️ Print / Save PDF</button>

<div class="note">
<b>Engineering convention:</b> XY separates VP and HP. The 3D view uses X along XY, Y perpendicular to VP on HP, and Z perpendicular to HP.
<br><br>
For the supplied FV and TV, the program calculates:
θ = cos⁻¹(FV/TL) and φ = cos⁻¹(TV/TL).
</div>

<div class="results">
<div class="metric"><b id="theta">—</b><span>θ with HP</span></div>
<div class="metric"><b id="phi">—</b><span>φ with VP</span></div>
<div class="metric"><b id="dX">—</b><span>ΔX</span></div>
<div class="metric"><b id="dY">—</b><span>ΔY</span></div>
</div>
</section>

<section class="stage">
<div class="toolbar">
<button class="primary" onclick="view3D()">3D View</button>
<button class="secondary" onclick="viewFront()">Front View</button>
<button class="secondary" onclick="viewTop()">Top View</button>
<button class="secondary" onclick="viewBoth()">3D + 2D</button>
<button class="secondary" onclick="resetCamera()">Reset Camera</button>
</div>

<div class="viewgrid" id="viewgrid">
<div class="panel">
<h3>🧊 3D Engineering View — HP + VP + XY + Line AB</h3>
<canvas id="c3d" width="900" height="650"></canvas>
<div class="legend">
<span><i class="dot" style="background:#1769aa"></i>Line AB</span>
<span><i class="dot" style="background:#5b8def"></i>VP</span>
<span><i class="dot" style="background:#e6a23c"></i>HP</span>
<span><i class="dot" style="background:#222"></i>XY</span>
</div>
</div>

<div class="panel">
<h3>📏 2D Projection</h3>
<canvas id="c2d" width="700" height="650"></canvas>
<div class="steps">
<b>Construction steps</b>
<ol id="steps"></ol>
</div>
</div>
</div>
</section>
</div>
<div class="footer">
  <div>Engineering Graphics Solver • Standalone HTML • No internet connection required</div>
  <div class="developer-credit">Developed by <span>Sarthak Bankar</span></div>
</div>

<script>
const C3=document.getElementById('c3d'), C2=document.getElementById('c2d');
const g3=C3.getContext('2d'), g2=C2.getContext('2d');
let cam={yaw:-0.72,pitch:0.42,zoom:1};
let data={};

function val(id){return parseFloat(document.getElementById(id).value)}
function clamp(x,a,b){return Math.max(a,Math.min(b,x))}
function generate(){
  let tl=val('tl'), fv=val('fv'), tv=val('tv'), sc=val('scale');
  if(!(tl>0&&fv>0&&tv>0)){alert('Enter positive True Length, FV and TV.');return}
  if(fv>tl||tv>tl){alert('FV and TV cannot be greater than True Length.');return}
  let th=Math.acos(clamp(fv/tl,0,1)), ph=Math.acos(clamp(tv/tl,0,1));
  // Coordinates: x component common to both projections.
  // FV = sqrt(dx^2 + dz^2), TV = sqrt(dx^2 + dy^2), TL=sqrt(dx^2+dy^2+dz^2)
  let dx2=(fv*fv+tv*tv-tl*tl)/2;
  if(dx2<0){alert('These three lengths are geometrically incompatible. Check TL, FV and TV.');return}
  let dx=Math.sqrt(dx2), dy=Math.sqrt(Math.max(0,tv*tv-dx2)), dz=Math.sqrt(Math.max(0,fv*fv-dx2));
  let acond=document.getElementById('acond').value, bcond=document.getElementById('bcond').value;
  // Position B based on stated condition. A is then placed using the direction components.
  let B={x:0,y:0,z:0};
  if(bcond==='vp') B.y=0;
  if(bcond==='hp') B.z=0;
  if(bcond==='both'){B.y=0;B.z=0}
  let A={x:dx,y:dy,z:dz};
  // If A is specified in HP or VP, align its corresponding coordinate to zero.
  // This preserves the requested educational geometry where possible.
  if(acond==='hp') A.z=0;
  if(acond==='vp') A.y=0;
  // For visualization, use exact projection lengths as primary geometry.
  // If endpoint constraints conflict with FV/TV simultaneously, label the model as normalized.
  if(acond==='hp' && bcond==='vp') { // classic case: A on HP, B on VP
    // choose x from TL/FV/TV-derived relation; then z is FV component, y is TV component.
    A={x:dx,y:dy,z:dz}; B={x:0,y:0,z:0};
  }
  data={tl,fv,tv,th,ph,dx,dy,dz,sc,A,B};
  document.getElementById('theta').textContent=(th*180/Math.PI).toFixed(2)+'°';
  document.getElementById('phi').textContent=(ph*180/Math.PI).toFixed(2)+'°';
  document.getElementById('dX').textContent=dx.toFixed(2)+' mm';
  document.getElementById('dY').textContent=dy.toFixed(2)+' mm';
  document.getElementById('steps').innerHTML=
    `<li>Draw XY as the reference line between VP and HP.</li>
     <li>Set AB = ${tl.toFixed(2)} mm true length.</li>
     <li>Front-view length = ${fv.toFixed(2)} mm, therefore θ = cos⁻¹(${fv.toFixed(2)}/${tl.toFixed(2)}) = <b>${(th*180/Math.PI).toFixed(2)}°</b>.</li>
     <li>Top-view length = ${tv.toFixed(2)} mm, therefore φ = cos⁻¹(${tv.toFixed(2)}/${tl.toFixed(2)}) = <b>${(ph*180/Math.PI).toFixed(2)}°</b>.</li>
     <li>Resolve AB into ΔX = ${dx.toFixed(2)} mm, ΔY = ${dy.toFixed(2)} mm and ΔZ = ${dz.toFixed(2)} mm.</li>
     <li>Project A and B perpendicular to XY to obtain front and top views.</li>`;
  draw3d(); draw2d();
}

function project(p){
  let cy=Math.cos(cam.yaw), sy=Math.sin(cam.yaw), cp=Math.cos(cam.pitch), sp=Math.sin(cam.pitch);
  let x=p.x, y=p.y, z=p.z;
  let X=x*cy-y*sy;
  let Y=(x*sy+y*cy)*sp+z*cp;
  let Z=(x*sy+y*cy)*cp-z*sp;
  return {x:450+X*data.sc*cam.zoom*5, y:335-Y*data.sc*cam.zoom*5, depth:Z};
}
function planePoint(x,y,z){return project({x,y,z})}
function poly(points,fill,stroke){
  g3.beginPath();points.forEach((p,i)=>i?g3.lineTo(p.x,p.y):g3.moveTo(p.x,p.y));g3.closePath();
  g3.fillStyle=fill;g3.fill();g3.strokeStyle=stroke;g3.stroke();
}
function line(a,b,color,w=2,dash=[]){
  g3.save();g3.strokeStyle=color;g3.lineWidth=w;g3.setLineDash(dash);
  g3.beginPath();g3.moveTo(a.x,a.y);g3.lineTo(b.x,b.y);g3.stroke();g3.restore();
}
function txt(t,p,color='#17202a',size=14){
  g3.fillStyle=color;g3.font=`${size}px Arial`;g3.fillText(t,p.x,p.y);
}
function draw3d(){
  g3.clearRect(0,0,C3.width,C3.height);
  g3.fillStyle='#fff';g3.fillRect(0,0,C3.width,C3.height);
  if(!data.tl){return}
  let L=Math.max(data.tl*1.35,80);
  // HP z=0
  poly([planePoint(-L,-L,0),planePoint(L,-L,0),planePoint(L,L,0),planePoint(-L,L,0)],'rgba(235,168,50,.13)','#e2a338');
  // VP y=0
  poly([planePoint(-L,0,-L),planePoint(L,0,-L),planePoint(L,0,L),planePoint(-L,0,L)],'rgba(60,125,220,.11)','#5b8def');
  // axes / XY
  line(planePoint(-L,0,0),planePoint(L,0,0),'#222',3);
  line(planePoint(0,-L,0),planePoint(0,L,0),'#e0a12a',1,[6,5]);
  line(planePoint(0,0,-L),planePoint(0,0,L),'#527db0',1,[6,5]);
  txt('VP',planePoint(L*.7,0,L*.85),'#356fc0',16);
  txt('HP',planePoint(L*.65,L*.7,0),'#b47b15',16);
  txt('XY',planePoint(L*.85,0,0),'#222',15);

  let A=project(data.A),B=project(data.B);
  line(B,A,'#1769aa',5);
  // projectors from endpoints to VP/HP
  let Av=project({x:data.A.x,y:0,z:data.A.z}), Ah=project({x:data.A.x,y:data.A.y,z:0});
  let Bv=project({x:data.B.x,y:0,z:data.B.z}), Bh=project({x:data.B.x,y:data.B.y,z:0});
  line(A,Av,'#777',1,[5,5]); line(A,Ah,'#777',1,[5,5]);
  line(B,Bv,'#777',1,[5,5]); line(B,Bh,'#777',1,[5,5]);
  g3.fillStyle='#1769aa';g3.beginPath();g3.arc(A.x,A.y,7,0,Math.PI*2);g3.fill();
  g3.beginPath();g3.arc(B.x,B.y,7,0,Math.PI*2);g3.fill();
  txt("A", {x:A.x+10,y:A.y-10},'#1769aa',17);
  txt("B", {x:B.x+10,y:B.y+20},'#1769aa',17);
  // direction dimensions
  txt(`AB = ${data.tl.toFixed(1)} mm`,{x:20,y:30},'#17202a',16);
  txt(`θ = ${(data.th*180/Math.PI).toFixed(2)}°`,{x:20,y:54},'#17202a',14);
  txt(`φ = ${(data.ph*180/Math.PI).toFixed(2)}°`,{x:20,y:75},'#17202a',14);
  txt('Drag mouse / finger to rotate • Wheel/pinch to zoom',{x:20,y:625},'#68737d',12);
}
function draw2d(){
  g2.clearRect(0,0,C2.width,C2.height);g2.fillStyle='#fff';g2.fillRect(0,0,C2.width,C2.height);
  if(!data.tl)return;
  let s=Math.min(6*data.sc,500/data.tl), ox=80, xy=320;
  // XY
  g2.strokeStyle='#222';g2.lineWidth=2;g2.beginPath();g2.moveTo(30,xy);g2.lineTo(670,xy);g2.stroke();
  g2.fillStyle='#222';g2.font='bold 14px Arial';g2.fillText('X Y',620,xy-8);
  let fv=data.fv*s, tv=data.tv*s;
  let bx=250, by=xy;
  // Front view above XY
  g2.strokeStyle='#1769aa';g2.lineWidth=5;g2.beginPath();g2.moveTo(bx,by);g2.lineTo(bx+fv,by-fv*.45);g2.stroke();
  // Top view below XY
  g2.strokeStyle='#d34a4a';g2.lineWidth=5;g2.beginPath();g2.moveTo(bx,by);g2.lineTo(bx+tv,by+tv*.35);g2.stroke();
  // projectors
  g2.strokeStyle='#888';g2.lineWidth=1;g2.setLineDash([6,5]);
  g2.beginPath();g2.moveTo(bx,60);g2.lineTo(bx,580);g2.stroke();
  g2.beginPath();g2.moveTo(bx+fv,60);g2.lineTo(bx+tv,580);g2.stroke();g2.setLineDash([]);
  // labels
  g2.fillStyle='#1769aa';g2.font='bold 15px Arial';g2.fillText("a′",bx+fv+8,by-fv*.45);
  g2.fillText("b′",bx-20,by-8);
  g2.fillStyle='#d34a4a';g2.fillText("a",bx+tv+8,by+tv*.35+5);g2.fillText("b",bx-20,by+18);
  g2.fillStyle='#1769aa';g2.font='14px Arial';g2.fillText(`FV = ${data.fv.toFixed(1)} mm`,40,45);
  g2.fillText(`θ = ${(data.th*180/Math.PI).toFixed(2)}°`,40,68);
  g2.fillStyle='#d34a4a';g2.fillText(`TV = ${data.tv.toFixed(1)} mm`,40,500);
  g2.fillText(`φ = ${(data.ph*180/Math.PI).toFixed(2)}°`,40,523);
  g2.fillStyle='#555';g2.font='12px Arial';g2.fillText('Front View (FV)',430,100);g2.fillText('Top View (TV)',430,560);
}
function view3D(){document.getElementById('viewgrid').style.gridTemplateColumns='1fr 0fr';document.querySelectorAll('.viewgrid .panel')[1].style.display='none';document.querySelectorAll('.viewgrid .panel')[0].style.display='block';}
function viewFront(){view2dOnly('front')}
function viewTop(){view2dOnly('top')}
function view2dOnly(type){
  document.getElementById('viewgrid').style.gridTemplateColumns='1fr';
  let ps=document.querySelectorAll('.viewgrid .panel'); ps[0].style.display='none'; ps[1].style.display='block';
  draw2d();
}
function viewBoth(){document.getElementById('viewgrid').style.gridTemplateColumns='1.35fr 1fr';let ps=document.querySelectorAll('.viewgrid .panel');ps.forEach(x=>x.style.display='block')}
function resetCamera(){cam={yaw:-0.72,pitch:0.42,zoom:1};draw3d()}
function resetAll(){document.getElementById('tl').value=80;document.getElementById('fv').value=50;document.getElementById('tv').value=70;document.getElementById('bcond').value='vp';document.getElementById('acond').value='hp';cam={yaw:-.72,pitch:.42,zoom:1};generate()}

let dragging=false,lastX=0,lastY=0;
C3.addEventListener('pointerdown',e=>{dragging=true;lastX=e.clientX;lastY=e.clientY;C3.setPointerCapture(e.pointerId)});
C3.addEventListener('pointermove',e=>{if(!dragging)return;cam.yaw+=(e.clientX-lastX)*.008;cam.pitch+=(e.clientY-lastY)*.008;cam.pitch=clamp(cam.pitch,-1.35,1.35);lastX=e.clientX;lastY=e.clientY;draw3d()});
C3.addEventListener('pointerup',()=>dragging=false);
C3.addEventListener('pointercancel',()=>dragging=false);
C3.addEventListener('wheel',e=>{e.preventDefault();cam.zoom*=e.deltaY<0?1.1:.9;cam.zoom=clamp(cam.zoom,.35,3);draw3d()},{passive:false});

generate();
</script>
</body>
</html>
