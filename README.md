<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Runaway Alarm</title>
<style>
body{margin:0;background:#0f1115;color:white;font-family:sans-serif;overflow:hidden}
.track{
position:fixed;
width:300px;
height:70px;
background:#222;
border-radius:40px;
overflow:hidden;
left:20px;
top:200px;
transition:all .15s;
z-index:1000;
}
.thumb{
position:absolute;
left:8px;
top:8px;
width:120px;
height:54px;
background:#ff6b8b;
border-radius:40px;
display:flex;
align-items:center;
justify-content:center;
font-weight:bold;
touch-action:none;
}
.info{
position:fixed;
top:20px;
left:50%;
transform:translateX(-50%);
font-size:18px;
}
</style>
</head>
<body>
<div class="info">捕まえたら勝ちです</div>
<div class="track" id="track">
<div class="thumb" id="thumb">止める</div>
</div>
<script>
const track=document.getElementById("track")
const thumb=document.getElementById("thumb")
let dragging=false
let startX=0
let startLeft=8

function bounds(){
return{
minX:0,
maxX:window.innerWidth-track.offsetWidth,
minY:0,
maxY:window.innerHeight-track.offsetHeight
}
}

function moveTrack(level=1){
const b=bounds()
const strength=[0.2,0.5,0.9][level]
if(Math.random()<strength){
track.style.left=(Math.random()*(b.maxX-b.minX)+b.minX)+"px"
track.style.top=(Math.random()*(b.maxY-b.minY)+b.minY)+"px"
}
}

thumb.addEventListener("pointerdown",e=>{
dragging=true
startX=e.clientX
startLeft=thumb.offsetLeft
thumb.setPointerCapture(e.pointerId)
})

thumb.addEventListener("pointermove",e=>{
if(!dragging)return
let dx=e.clientX-startX
let newLeft=startLeft+dx
let max=track.clientWidth-thumb.offsetWidth-8

moveTrack(2)

if(Math.random()<0.4){
newLeft-=40*Math.random()
}

newLeft=Math.max(8,Math.min(newLeft,max))
thumb.style.left=newLeft+"px"

if(newLeft>=max-2){
alert("起きろ！！！")
location.reload()
}
})

thumb.addEventListener("pointerup",()=>dragging=false)
</script>
</body>
</html>
