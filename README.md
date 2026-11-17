# Chess-Calculator-Premium
Premium Chess and Calculator Android 

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Chess + Calculator</title>
<style>
:root {
  --bg:#f1f5f4;
  --ink:#1b2a29;
  --card:#fff;
  --light:#d3e6df;
  --dark:#3f7d78;
  --accent:#f2b134;
}
* { box-sizing:border-box; }
body {
  margin:0;
  background:var(--bg);
  color:var(--ink);
  font-family:Arial,sans-serif;
}
main {
  max-width:460px;
  margin:auto;
  padding:16px;
}
nav {
  display:flex;
  gap:8px;
  margin-bottom:15px;
}
nav button {
  flex:1;
  padding:13px;
  border:0;
  border-radius:10px;
  background:#dce8e4;
  font-size:17px;
  font-weight:bold;
}
nav button.active {
  background:var(--dark);
  color:white;
}
.panel[hidden] { display:none; }
#board {
  display:grid;
  grid-template-columns:repeat(8,1fr);
  width:100%;
  aspect-ratio:1;
}
.sq {
  border:0;
  padding:0;
  font-size:clamp(24px,9vw,40px);
  display:flex;
  align-items:center;
  justify-content:center;
}
.sq.light { background:var(--light); }
.sq.dark { background:var(--dark); }
.sq.selected { background:var(--accent); }
.row {
  display:flex;
  gap:10px;
  margin-top:12px;
}
.row button {
  flex:1;
  padding:12px;
  border:0;
  border-radius:10px;
  font-weight:bold;
}
.display {
  background:white;
  padding:20px;
  border-radius:14px;
  text-align:right;
  margin-bottom:12px;
  min-height:100px;
}
#result {
  font-size:40px;
  font-weight:bold;
}
.keys {
  display:grid;
  grid-template-columns:repeat(4,1fr);
  gap:8px;
}
.keys button {
  height:65px;
  border:0;
  border-radius:12px;
  font-size:23px;
  background:#dce8e4;
}
.keys button.op {
  background:var(--dark);
  color:white;
}
.keys button.equal {
  background:var(--accent);
}
</style>
</head>
<body>
<main>

<nav>
<button class="active" onclick="tab('chess',this)">♟ Chess</button>
<button onclick="tab('calc',this)">🧮 Calculator</button>
</nav>

<section id="chess" class="panel">
<p id="status">Your move (White)</p>
<div id="board"></div>
<div class="row">
<button onclick="undo()">Undo</button>
<button onclick="reset()">New Game</button>
</div>
</section>

<section id="calc" class="panel" hidden>
<div class="display">
<div id="expression"></div>
<div id="result">0</div>
</div>
<div class="keys" id="keys"></div>
</section>
</main>

<script>
function tab(id,btn) {
 document.querySelectorAll('.panel').forEach(p=>p.hidden=true);
 document.getElementById(id).hidden=false;
 document.querySelectorAll('nav button').forEach(b=>b.classList.remove('active'));
 btn.classList.add('active');
}

/* Chess game */
const pieces={
 r:'♜',n:'♞',b:'♝',q:'♛',k:'♚',p:'♟',
 R:'♖',N:'♘',B:'♗',Q:'♕',K:'♔',P:'♙'
};
const start=[
'r','n','b','q','k','b','n','r',
'p','p','p','p','p','p','p','p',
'','','','','','','','',
'','','','','','','','',
'','','','','','','','',
'','','','','','','','',
'P','P','P','P','P','P','P','P',
'R','N','B','Q','K','B','N','R'
];
let board=[...start],selected=-1,white=true;
let history=[];

function draw() {
 const el=document.getElementById('board');
 el.innerHTML='';
 board.forEach((p,i)=>{
  const b=document.createElement('button');
  b.className='sq '+((Math.floor(i/8)+i%8)%2?'dark':'light');
  if(i===selected)b.classList.add('selected');
  b.textContent=pieces[p]||'';
  b.onclick=()=>move(i);
  el.appendChild(b);
 });
}
function move(i) {
 if(selected<0) {
  if(board[i] && (white ? board[i]===board[i].toUpperCase() : board[i]===board[i].toLowerCase())) selected=i;
 } else {
  history.push({board:[...board],white});
  board[i]=board[selected];
  board[selected]='';
  selected=-1;
  white=!white;
  document.getElementById('status').textContent=white?'Your move (White)':'Black to move';
 }
 draw();
}
function undo() {
 if(!history.length)return;
 const h=history.pop();
 board=h.board;
 white=h.white;
 selected=-1;
 draw();
}
function reset() {
 board=[...start];
 selected=-1;
 white=true;
 history=[];
 document.getElementById('status').textContent='Your move (White)';
 draw();
}
draw();

/* Calculator */
let expression='';
const keys=[
'C','⌫','%','÷',
'7','8','9','×',
'4','5','6','−',
'1','2','3','+',
'0','.','(',')',
'='
];
const keyBox=document.getElementById('keys');
keys.forEach(k=>{
 const b=document.createElement('button');
 b.textContent=k;
 if(['÷','×','−','+'].includes(k))b.className='op';
 if(k==='=')b.className='equal';
 b.onclick=()=>press(k);
 keyBox.appendChild(b);
});
function press(k) {
 if(k==='C')expression='';
 else if(k==='⌫')expression=expression.slice(0,-1);
 else if(k==='=') {
  try {
   const safe=expression.replace(/÷/g,'/').replace(/×/g,'*').replace(/−/g,'-');
   if(!/^[0-9+\-*/().% ]+$/.test(safe))throw Error();
   const result=Function('"use strict";return ('+safe+')')();
   if(!Number.isFinite(result))throw Error();
   expression=String(result);
  } catch(e) {
   document.getElementById('result').textContent='Error';
   return;
  }
 } else expression+=k;
 show();
}
function show() {
 document.getElementById('expression').textContent=expression;
 document.getElementById('result').textContent=expression||'0';
}
</script>
</body>
</html>

