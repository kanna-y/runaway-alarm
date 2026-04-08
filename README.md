<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>Runaway Alarm Spinner</title>
<style>
*{box-sizing:border-box}
html,body{height:100%}
body{
  margin:0;
  overflow:hidden;
  background:#0f1115;
  color:#fff;
  font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;
  touch-action:none;
}
.info{
  position:fixed;
  top:20px;
  left:50%;
  transform:translateX(-50%);
  z-index:20;
  text-align:center;
  padding:12px 16px;
  border-radius:16px;
  background:rgba(255,255,255,.08);
  border:1px solid rgba(255,255,255,.1);
  backdrop-filter:blur(10px);
}
.info strong{display:block;font-size:18px;margin-bottom:4px}
.info span{font-size:13px;opacity:.8}
.arena{
  position:fixed;
  inset:0;
  overflow:hidden;
}
.bar{
  position:fixed;
  width:300px;
  height:72px;
  border-radius:999px;
  background:rgba(255,255,255,.08);
  border:1px solid rgba(255,255,255,.12);
  box-shadow:0 16px 32px rgba(0,0,0,.28);
  overflow:hidden;
  z-index:10;
  transform-origin:center center;
}
.fill{
  position:absolute;
  left:0;
  top:0;
  bottom:0;
  width:0%;
  background:linear-gradient(90deg, rgba(255,107,139,.35), rgba(122,162,255,.28));
}
.thumb{
  position:absolute;
  left:8px;
  top:8px;
  width:124px;
  height:56px;
  border-radius:999px;
  background:linear-gradient(135deg,#ff6b8b,#ff8fb9);
  color:#fff;
  display:flex;
  align-items:center;
  justify-content:center;
  font-weight:800;
  letter-spacing:.04em;
  user-select:none;
  touch-action:none;
  box-shadow:0 10px 20px rgba(255,107,139,.28);
}
.toast{
  position:fixed;
  left:50%;
  bottom:28px;
  transform:translateX(-50%);
  background:rgba(0,0,0,.72);
  border:1px solid rgba(255,255,255,.12);
  padding:12px 16px;
  border-radius:999px;
  z-index:30;
  font-size:14px;
}
</style>
</head>
<body>
<div class="info">
  <strong>バー本体が画面をくるくる回ります</strong>
  <span>ピンクの「止める」を追いかけて右端まで持っていってください</span>
</div>
<div class="arena"></div>
<div class="bar" id="bar">
  <div class="fill" id="fill"></div>
  <div class="thumb" id="thumb">止める</div>
</div>
<div class="toast">バー本体が逃げます。かなり理不尽です。</div>
<script>
const bar = document.getElementById('bar');
const thumb = document.getElementById('thumb');
const fill = document.getElementById('fill');

const state = {
  dragging: false,
  pointerId: null,
  startX: 0,
  thumbStart: 8,
  thumbLeft: 8,
  progress: 0,
  orbitAngle: 0,
  orbitRadiusX: 110,
  orbitRadiusY: 220,
  centerX: window.innerWidth / 2,
  centerY: window.innerHeight / 2,
  barWidth: 300,
  barHeight: 72,
  rotation: 0,
  targetRotation: 0,
  escapedAt: 0,
};

function clamp(v,min,max){
  return Math.max(min, Math.min(max, v));
}

function resizeOrbit(){
  state.centerX = window.innerWidth / 2;
  state.centerY = window.innerHeight / 2;
  state.orbitRadiusX = Math.max(70, Math.min(window.innerWidth * 0.32, 180));
  state.orbitRadiusY = Math.max(120, Math.min(window.innerHeight * 0.34, 260));
}

function getThumbMaxLeft(){
  return state.barWidth - thumb.offsetWidth - 8;
}

function setThumbLeft(left){
  state.thumbLeft = clamp(left, 8, getThumbMaxLeft());
  thumb.style.left = state.thumbLeft + 'px';
}

function setProgress(p){
  state.progress = clamp(p, 0, 100);
  fill.style.width = state.progress + '%';
}

function updateBarTransform(){
  const x = state.centerX + Math.cos(state.orbitAngle) * state.orbitRadiusX - state.barWidth / 2;
  const y = state.centerY + Math.sin(state.orbitAngle) * state.orbitRadiusY - state.barHeight / 2;
  bar.style.left = x + 'px';
  bar.style.top = y + 'px';
  bar.style.transform = `rotate(${state.rotation}deg)`;
}

function randomEscapeBoost(){
  const now = performance.now();
  if (now - state.escapedAt < 180) return;
  state.escapedAt = now;
  state.orbitAngle += (Math.random() * 0.9 + 0.45) * (Math.random() > 0.5 ? 1 : -1);
  state.targetRotation += (Math.random() * 160 + 80) * (Math.random() > 0.5 ? 1 : -1);
}

function animate(){
  state.orbitAngle += 0.018;
  state.rotation += (state.targetRotation - state.rotation) * 0.12;
  state.targetRotation += Math.sin(performance.now() * 0.002) * 0.9;
  updateBarTransform();
  requestAnimationFrame(animate);
}

function complete(){
  setProgress(100);
  alert('解除成功！かなりえらいです');
  setThumbLeft(8);
  setProgress(0);
  state.orbitAngle = 0;
  state.rotation = 0;
  state.targetRotation = 0;
}

thumb.addEventListener('pointerdown', (e) => {
  state.dragging = true;
  state.pointerId = e.pointerId;
  state.startX = e.clientX;
  state.thumbStart = state.thumbLeft;
  thumb.setPointerCapture(e.pointerId);
});

thumb.addEventListener('pointermove', (e) => {
  if (!state.dragging) return;

  const dx = e.clientX - state.startX;
  let next = state.thumbStart + dx * 0.72;

  const threat = (next - 8) / (getThumbMaxLeft() - 8 || 1);

  if (Math.random() < 0.28 + threat * 0.42) {
    next -= 24 + Math.random() * 42;
  }

  if (Math.random() < 0.24 + threat * 0.34) {
    randomEscapeBoost();
  }

  setThumbLeft(next + (Math.random() - 0.5) * 8);

  const p = ((state.thumbLeft - 8) / (getThumbMaxLeft() - 8 || 1)) * 100;
  setProgress(Math.max(state.progress, p));

  if (state.thumbLeft >= getThumbMaxLeft() - 2) {
    complete();
  }
});

function endDrag(e){
  if (!state.dragging) return;
  try { thumb.releasePointerCapture(e.pointerId); } catch (_) {}
  state.dragging = false;
  state.pointerId = null;
  setThumbLeft(state.thumbLeft - 22);
  setProgress(state.progress - 10);
  randomEscapeBoost();
}

thumb.addEventListener('pointerup', endDrag);
thumb.addEventListener('pointercancel', endDrag);

window.addEventListener('resize', () => {
  resizeOrbit();
  updateBarTransform();
});

resizeOrbit();
setThumbLeft(8);
setProgress(0);
updateBarTransform();
requestAnimationFrame(animate);
</script>
</body>
</html>
