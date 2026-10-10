# earth-to-universe

<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>从地球到宇宙 · 3D地理探索</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#020611;color:#eaf4ff;
font-family:Arial,"Microsoft YaHei",sans-serif}
#scene{position:fixed;inset:0}
.panel{position:fixed;z-index:2;background:#071426d9;
border:1px solid #294461;border-radius:14px;padding:16px;
backdrop-filter:blur(12px)}
#info{left:18px;top:18px;width:270px}
#controls{right:18px;top:18px;width:235px}
#bottom{left:50%;bottom:20px;transform:translateX(-50%);
width:min(580px,92vw);text-align:center}
h2{font-size:19px;margin:0 0 10px;color:#91d7ff}
p{font-size:13px;line-height:1.8;color:#c5d8ec}
button{background:#12395b;color:#eaf7ff;border:1px solid #3676a6;
border-radius:8px;padding:9px 12px;margin:4px;cursor:pointer}
button:hover{background:#1e5a85}
input{width:100%;accent-color:#58c9ff}
#hint{font-size:12px;color:#8ba9c6}
#layerButtons{display:flex;flex-wrap:wrap}
@media(max-width:650px){
#info{left:8px;top:8px;width:190px;padding:10px}
#controls{right:8px;top:auto;bottom:105px;width:165px;padding:10px}
.panel p{font-size:11px}
h2{font-size:15px}
#bottom{bottom:8px}
}
</style>
</head>
<body>
<div id="scene"></div>

<section class="panel" id="info">
<h2 id="title">地球 · 我们的家园</h2>
<p id="description">这是一次从地球出发的宇宙尺度探索。拖动旋转视角，滚动鼠标滚轮改变观察尺度。</p>
<div id="layerButtons">
<button data-layer="crust">地壳</button>
<button data-layer="mantle">地幔</button>
<button data-layer="outer">外核</button>
<button data-layer="inner">内核</button>
</div>
<p id="layerInfo">点击某一结构，了解地球内部。</p>
</section>

<section class="panel" id="controls">
<h2>探索控制</h2>
<button id="cut">切开地球</button>
<button id="reset">返回地球</button>
<p>连续缩放尺度</p>
<input id="zoom" type="range" min="0" max="100" value="0">
<p id="scaleLabel">地球尺度</p>
<p id="hint">拖动旋转 · 滚轮缩放 · 点击结构查看知识</p>
</section>

<section class="panel" id="bottom">
<strong>从地球，到太阳系，再到银河系</strong>
<p style="margin:5px 0">观察尺度不断扩大时，天体之间的关系如何变化。</p>
</section>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x020611);
scene.fog=new THREE.FogExp2(0x020611,0.0015);

const camera=new THREE.PerspectiveCamera(
55,innerWidth/innerHeight,0.1,3000);
camera.position.set(0,0,7);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.outputColorSpace=THREE.SRGBColorSpace;
document.getElementById('scene').appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0x8aaacc,1.8));
const sunLight=new THREE.DirectionalLight(0xffffff,3.2);
sunLight.position.set(-5,3,8);
scene.add(sunLight);

const starsGeo=new THREE.BufferGeometry();
const starPositions=[];
for(let i=0;i<3500;i++){
 starPositions.push((Math.random()-.5)*1800,
 (Math.random()-.5)*1800,(Math.random()-.5)*1800);
}
starsGeo.setAttribute('position',
new THREE.Float32BufferAttribute(starPositions,3));
scene.add(new THREE.Points(starsGeo,
new THREE.PointsMaterial({color:0xffffff,size:0.8})));

const world=new THREE.Group();
scene.add(world);

function sphere(radius,color,segments=48){
 return new THREE.Mesh(
 new THREE.SphereGeometry(radius,segments,segments),
 new THREE.MeshStandardMaterial({
 color,roughness:.8,metalness:.05
 }));
}

const earthGroup=new THREE.Group();
world.add(earthGroup);

const earth=sphere(2,0x2879bd,96);
earth.material.roughness=.95;
earthGroup.add(earth);

// 程序生成的示意性大陆纹理，不代表精确地理边界
const landGroup=new THREE.Group();
earthGroup.add(landGroup);
const landMaterial=new THREE.MeshStandardMaterial({
 color:0x56a76d,roughness:1
});
function addLandPatch(lat,lon,sizeX,sizeY){
 const geometry=new THREE.SphereGeometry(2.012,24,16,
 lon*Math.PI/180, sizeX,
 Math.PI/2-lat*Math.PI/180-sizeY/2,sizeY);
 const mesh=new THREE.Mesh(geometry,landMaterial);
 landGroup.add(mesh);
}
addLandPatch(25,-100,.65,.45);
addLandPatch(5,-60,.4,.8);
addLandPatch(45,40,.95,.5);
addLandPatch(-20,20,.45,.55);
addLandPatch(30,115,.8,.42);
addLandPatch(-25,135,.32,.28);

const atmosphere=new THREE.Mesh(
 new THREE.SphereGeometry(2.08,64,64),
 new THREE.MeshBasicMaterial({
 color:0x43baff,transparent:true,opacity:.13,
 side:THREE.BackSide
 }));
earthGroup.add(atmosphere);

// 地球内部结构：平时隐藏，点击“切开地球”后显示
const interior=new THREE.Group();
earthGroup.add(interior);
const layerData=[
 {id:'crust',name:'地壳',radius:1.98,color:0x9a7b52,
 text:'地壳是地球最外层的固体圈层。大陆地壳通常较厚，洋壳通常较薄。'},
 {id:'mantle',name:'地幔',radius:1.55,color:0xe66b3d,
 text:'地幔位于地壳之下，主要由固态岩石组成。地幔内部的缓慢运动与板块运动密切相关。'},
 {id:'outer',name:'外核',radius:1.02,color:0xf4b43e,
 text:'外核主要由液态铁、镍等物质组成。其运动与地球磁场的形成有关。'},
 {id:'inner',name:'内核',radius:.52,color:0xffe59b,
 text:'内核是地球最内部的固体部分，主要由铁、镍等物质组成。虽然温度极高，但巨大压力使其保持固态。'}
];
const layerMeshes=[];
layerData.forEach((d,i)=>{
 const m=sphere(d.radius,d.color,64);
 m.userData.layer=d;
 m.visible=false;
 interior.add(m);
 layerMeshes.push(m);
});

// 月球与地月轨道
const moonOrbit=new THREE.Group();
world.add(moonOrbit);
const orbitLine=new THREE.Mesh(
 new THREE.TorusGeometry(3.2,.009,6,120),
 new THREE.MeshBasicMaterial({color:0x45627c}));
moonOrbit.add(orbitLine);
const moon=sphere(.28,0xb8c0ca,32);
moon.position.set(3.2,0,0);
moonOrbit.add(moon);

// 太阳系示意
const solarGroup=new THREE.Group();
world.add(solarGroup);
const sun=sphere(1.15,0xffb32c,48);
solarGroup.add(sun);
const planets=[
 ['水星',.12,2.0,0xa9a9a9],
 ['金星',.2,2.8,0xe4bc83],
 ['地球',.22,3.8,0x318bd0],
 ['火星',.16,4.8,0xd85b45],
 ['木星',.52,6.4,0xd8b38c],
 ['土星',.43,8.0,0xd8c48b],
 ['天王星',.31,9.5,0x86d8df],
 ['海王星',.30,11.0,0x4779e7]
];
const planetObjects=[];
planets.forEach((p,i)=>{
 const orbit=new THREE.Mesh(
 new THREE.TorusGeometry(p[2],.008,4,128),
 new THREE.MeshBasicMaterial({color:0x34475d}));
 orbit.rotation.x=Math.PI/2;
 solarGroup.add(orbit);
 const group=new THREE.Group();
 group.userData.name=p[0];
 const body=sphere(p[1],p[3],32);
 body.position.x=p[2];
 group.add(body);
 if(p[0]==='土星'){
  const ring=new THREE.Mesh(
   new THREE.TorusGeometry(.63,.055,8,48),
   new THREE.MeshStandardMaterial({color:0xd9cba5}));
  ring.position.x=p[2];
  ring.rotation.x=Math.PI/2.5;
  group.add(ring);
 }
 solarGroup.add(group);
 planetObjects.push(group);
});

// 银河系示意：星点构成旋臂，太阳系位置偏离中心
const galaxyGroup=new THREE.Group();
world.add(galaxyGroup);
const galaxyPositions=[];
for(let i=0;i<5000;i++){
 const arm=i%4;
 const r=Math.random()*38+1;
 const a=arm*Math.PI/2+r*.15+(Math.random()-.5)*.6;
 galaxyPositions.push(Math.cos(a)*r,
 (Math.random()-.5)*.7,
 Math.sin(a)*r);
}
const galaxyGeo=new THREE.BufferGeometry();
galaxyGeo.setAttribute('position',
 new THREE.Float32BufferAttribute(galaxyPositions,3));
const galaxyStars=new THREE.Points(galaxyGeo,
 new THREE.PointsMaterial({color:0xa8c8ff,size:.07}));
galaxyGroup.add(galaxyStars);
const galaxyCore=sphere(2.2,0xc6b4ff,32);
galaxyGroup.add(galaxyCore);
const sunLocation=sphere(.22,0xffdc70,16);
sunLocation.position.set(13,.1,0);
galaxyGroup.add(sunLocation);

// 交互与连续尺度
let zoomValue=0;
let cutMode=false;
let dragging=false;
let lastX=0,lastY=0;
let rotX=0,rotY=0;
const raycaster=new THREE.Raycaster();
const pointer=new THREE.Vector2();

function updateScale(value){
 zoomValue=THREE.MathUtils.clamp(value,0,100);
 // 通过连续距离变化呈现尺度过渡
 const distance=THREE.MathUtils.mapLinear(
 zoomValue,0,100,7,250);
 camera.position.setLength(distance);
 const solarT=THREE.MathUtils.smoothstep(zoomValue,22,48);
 const galaxyT=THREE.MathUtils.smoothstep(zoomValue,58,88);

 earthGroup.visible=zoomValue<43;
 moonOrbit.visible=zoomValue<58;
 solarGroup.visible=zoomValue>24&&zoomValue<78;
 galaxyGroup.visible=zoomValue>58;

 earthGroup.scale.setScalar(1-solarT*.88);
 moonOrbit.scale.setScalar(.8+solarT*1.5);
 solarGroup.scale.setScalar(.2+solarT*1.4);
 galaxyGroup.scale.setScalar(.15+galaxyT*1.8);

 let label='地球尺度';
 if(zoomValue>=15)label='地月系尺度';
 if(zoomValue>=32)label='太阳系尺度';
 if(zoomValue>=62)label='银河系尺度';
 if(zoomValue>=88)label='更广阔的宇宙';
 document.getElementById('scaleLabel').textContent=
 label+' · '+Math.round(zoomValue)+'%';
}
document.getElementById('zoom').addEventListener('input',e=>{
 updateScale(Number(e.target.value));
});
renderer.domElement.addEventListener('wheel',e=>{
 e.preventDefault();
 const slider=document.getElementById('zoom');
 slider.value=THREE.MathUtils.clamp(
 Number(slider.value)+e.deltaY*.025,0,100);
 updateScale(Number(slider.value));
},{passive:false});

renderer.domElement.addEventListener('pointerdown',e=>{
 dragging=true;lastX=e.clientX;lastY=e.clientY;
});
window.addEventListener('pointerup',()=>dragging=false);
window.addEventListener('pointermove',e=>{
 if(!dragging)return;
 rotY+=(e.clientX-lastX)*.006;
 rotX+=(e.clientY-lastY)*.006;
 lastX=e.clientX;lastY=e.clientY;
});
document.getElementById('cut').onclick=()=>{
 cutMode=!cutMode;
 interior.visible=cutMode;
 earth.visible=!cutMode;
 landGroup.visible=!cutMode;
 atmosphere.visible=!cutMode;
 if(cutMode){
  layerMeshes.forEach((m,i)=>{
   m.visible=true;
   const offset=(3-i)*.07;
   m.position.set(offset,0,0);
  });
  document.getElementById('layerInfo').textContent=
  '点击地球结构模型，查看各层知识。';
 }else{
  layerMeshes.forEach(m=>m.visible=false);
 }
};
document.getElementById('reset').onclick=()=>{
 document.getElementById('zoom').value=0;
 updateScale(0);
 cutMode=false;interior.visible=false;
 earth.visible=true;landGroup.visible=true;
 atmosphere.visible=true;
 layerMeshes.forEach(m=>m.visible=false);
 document.getElementById('title').textContent='地球 · 我们的家园';
 document.getElementById('description').textContent=
 '从地球出发，连续探索地月系、太阳系与银河系。';
};
document.querySelectorAll('[data-layer]').forEach(btn=>{
 btn.onclick=()=>{
  const d=layerData.find(x=>x.id===btn.dataset.layer);
  document.getElementById('title').textContent=d.name;
  document.getElementById('layerInfo').textContent=d.text;
  document.getElementById('description').textContent=
  '地球内部结构 · 科学示意模型';
  cutMode=true;interior.visible=true;
  earth.visible=false;landGroup.visible=false;
  atmosphere.visible=false;
  layerMeshes.forEach(m=>m.visible=true);
 };
});

// 点击模型时识别地球结构或行星
renderer.domElement.addEventListener('click',e=>{
 pointer.x=(e.clientX/innerWidth)*2-1;
 pointer.y=-(e.clientY/innerHeight)*2+1;
 raycaster.setFromCamera(pointer,camera);
 const hits=raycaster.intersectObjects(
 [...layerMeshes,...planetObjects.flatMap(g=>g.children)],true);
 if(!hits.length)return;
 let obj=hits[0].object;
 while(obj&&!obj.userData.layer&&!obj.userData.name&&obj.parent){
  obj=obj.parent;
 }
 if(obj.userData.layer){
  const d=obj.userData.layer;
  document.getElementById('title').textContent=d.name;
  document.getElementById('layerInfo').textContent=d.text;
 }else if(obj.userData.name){
  document.getElementById('title').textContent=obj.userData.name;
  document.getElementById('description').textContent=
  '这是太阳系中的一颗行星。行星的大小、颜色和轨道距离在本模型中采用示意比例。';
 }
});

function animate(){
 requestAnimationFrame(animate);
 earthGroup.rotation.y+=.0008;
 moonOrbit.rotation.y+=.002;
 planetObjects.forEach((p,i)=>{p.rotation.y+=.0005/(i+1)});
 galaxyGroup.rotation.y+=.00012;
 world.rotation.x=rotX*.15;
 world.rotation.y=rotY*.15;
 renderer.render(scene,camera);
}
updateScale(0);
animate();

window.addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
