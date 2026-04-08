<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Runaway Alarm</title>
<style>
body{
  margin:0;
  overflow:hidden;
  background:#0f1115;
  color:white;
  font-family:sans-serif;
}

.bar{
  position:fixed;
  width:300px;
  height:70px;
  border-radius:40px;
  background:#222;
  left:50%;
  top:50%;
  transform-origin:center;
}

.fill{
  position:absolute;
  left:0;
  top:0;
  height:100%;
  width:0%;
  background:rgba(255,100,120,0.3);
}

.thumb{
  position:absolute;
  left:8px;
  top:8px;
  width:120px;
  height:54px;
  border-radius:40px;
  background:#ff6b8b;
  display:flex;
  align-items:center;
  justify-content:center;
  font-weight:bold;
  touch-action:none;
}
</style>
</head>
<body>

<div class="bar" id="bar">
  <div class="fill" id="fill"></div>
  <div class="thumb" id="thumb">止める</div>
</div>

<script>
const bar = document.getElementById("bar")
const thumb = document.getElementById("thumb")
const fill = document.getElementById("fill")

let dragging=false
let startX=0
let startLeft=8

let angle=0
let radiusX=120
let radiusY=200

function animate(){
  angle+=0.02

  let x = window.innerWidth/2 + Math.cos(angle)*radiusX - 150
  let y = window.innerHeight/2 + Math.sin(angle)*radiusY - 35

  bar.style.left = x + "px"
  bar.style.top = y + "px"
  bar.style.transform = "rotate("+(angle*40)+"deg)"

  requestAnimationFrame(animate)
}
animate()

thumb.addEventListener("pointerdown",e=>{
  dragging=true
  startX=e.clientX
  startLeft=thumb.offsetLeft
  thumb.setPointerCapture(e.pointerId)
})

thumb.addEventListener("pointermove",e=>{
  if(!dragging)return

  let dx=e.clientX-startX
  let next=startLeft+dx*0.7

  let max=bar.clientWidth-thumb.offsetWidth-8
  let threat=(next-8)/(max-8)

  // ⭐ 強化された逃げ
  if(Math.random() < 0.35 + threat*0.55){
    next -= 35 + Math.random()*55
  }

  // ⭐ 細かい嫌がらせ
  if(Math.random() < 0.2 + threat*0.3){
    next -= 20
  }

  next=Math.max(8,Math.min(next,max))
  thumb.style.left=next+"px"

  let progress=(next-8)/(max-8)*100
  fill.style.width=progress+"%"

  if(next>=max-2){
    alert("起きろ！！！")
    location.reload()
  }
})

thumb.addEventListener("pointerup",()=>dragging=false)
</script>

</body>
</html>
