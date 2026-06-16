<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Crypto Mobile</title>

<style>
body{
    margin:0;
    font-family:Arial;
    background:#0b1020;
    color:white;
    overflow:hidden;
}

.container{
    width:100%;
    max-width:420px;
    margin:auto;
    padding:10px;
}

.card{
    background:rgba(255,255,255,0.05);
    border:1px solid rgba(255,255,255,0.08);
    border-radius:12px;
    padding:8px;
    margin-top:8px;
}

.price{
    font-size:22px;
    color:#00ffcc;
    font-weight:bold;
}

canvas{
    width:100%;
    height:140px;
    border-radius:10px;
}

button{
    width:100%;
    margin-top:6px;
    padding:12px;
    border:none;
    border-radius:10px;
    font-weight:bold;
    background:linear-gradient(135deg,#00ffcc,#00aaff);
}

.tabs{
    display:flex;
    gap:6px;
    margin-top:8px;
}

.tab{
    flex:1;
    padding:8px;
    text-align:center;
    border-radius:10px;
    background:#151c2c;
}

.active{
    background:#00ffcc;
    color:black;
}

#msg{
    margin-top:6px;
    color:#ff4d4d;
    font-weight:bold;
    min-height:18px;
}
</style>
</head>

<body>

<div class="container">

<div class="card">
💰 <b id="money">0</b>$ | 🪙 <b id="coin">0</b>
</div>

<div class="tabs">
<div class="tab active" id="t1" onclick="openTab('market')">📈</div>
<div class="tab" id="t2" onclick="openTab('shop')">🛒</div>
</div>

<div id="marketTab">

<div class="card">
📈 <div class="price" id="price">0</div>
<canvas id="c"></canvas>
</div>

<div class="card">
<button onclick="buy()">BUY</button>
<button onclick="sell()">SELL</button>
</div>

</div>

<div id="shopTab" style="display:none;">

<div class="card">
<h3>🛒 SHOP</h3>

<button onclick="buyBot()">🤖 BOT — 3000$</button>

<div id="msg"></div>

</div>

</div>

</div>

<script>

let price = Number(localStorage.getItem("price")) || 100;
let money = Number(localStorage.getItem("money")) || 1000;
let coin = Number(localStorage.getItem("coin")) || 0;

let bot = localStorage.getItem("bot") === "true";
let boughtBot = localStorage.getItem("boughtBot") === "true";

let history = JSON.parse(localStorage.getItem("history")) || [price];

const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");

function resize(){
    canvas.width = canvas.offsetWidth;
    canvas.height = 140;
}
window.addEventListener("resize", resize);
resize();

// msg
function showMsg(t){
    const el = document.getElementById("msg");
    el.innerText = t;
    setTimeout(()=>el.innerText="",1500);
}

// save
function save(){
    localStorage.setItem("money",money);
    localStorage.setItem("coin",coin);
    localStorage.setItem("price",price);
    localStorage.setItem("bot",bot);
    localStorage.setItem("boughtBot",boughtBot);
    localStorage.setItem("history",JSON.stringify(history));
}

// tabs
function openTab(tab){
    document.getElementById("marketTab").style.display = tab==="market"?"block":"none";
    document.getElementById("shopTab").style.display = tab==="shop"?"block":"none";

    document.getElementById("t1").classList.toggle("active",tab==="market");
    document.getElementById("t2").classList.toggle("active",tab==="shop");
}

// market
function market(){

    let change = (Math.random()-0.5)*12;
    price = Math.max(1, price + change);

    history.push(price);
    if(history.length > 40) history.shift();

    draw();
    update();
    save();

    if(bot) botTrade();
}

// graph
function draw(){

    ctx.clearRect(0,0,canvas.width,canvas.height);

    ctx.beginPath();
    ctx.strokeStyle="#00ffcc";
    ctx.lineWidth=2;

    let step = canvas.width/(history.length-1);

    for(let i=0;i<history.length;i++){
        let x = i*step;
        let y = canvas.height - (history[i]/200)*canvas.height;

        if(i===0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
    }

    ctx.stroke();
}

// UI
function update(){
    document.getElementById("price").textContent = price.toFixed(2);
    document.getElementById("money").textContent = money.toFixed(0);
    document.getElementById("coin").textContent = coin;
}

// buy/sell
function buy(){
    if(money >= price){
        money -= price;
        coin++;
        save();
    } else {
        showMsg("Недостаточно средств");
    }
}

function sell(){
    if(coin > 0){
        money += price;
        coin--;
        save();
    }
}

// bot
function buyBot(){
    if(!boughtBot && money >= 3000){
        money -= 3000;
        bot = true;
        boughtBot = true;
        save();
        showMsg("БОТ КУПЛЕН");
    } else {
        showMsg("Недостаточно средств");
    }
}

function botTrade(){

    let last = history[history.length-1];
    let prev = history[history.length-2];

    if(last < prev && money >= price){
        money -= price;
        coin++;
    }

    if(last > prev && coin > 0){
        money += price;
        coin--;
    }
}

// loop
function loop(){
    market();
    requestAnimationFrame(()=>setTimeout(loop,200));
}

loop();

update();

</script>

</body>
</html>
