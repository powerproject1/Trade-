<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Crypto Mobile</title>

<style>
body{
    margin:0;
    font-family:Arial;
    background:#0b1020;
    color:white;
    overflow:hidden;
}

/* 📱 МОБИЛЬНЫЙ КОНТЕЙНЕР */
.container{
    width:100%;
    max-width:420px;
    margin:auto;
    padding:10px;
}

/* карточки компактнее */
.card{
    background:rgba(255,255,255,0.05);
    border:1px solid rgba(255,255,255,0.08);
    border-radius:12px;
    padding:8px;
    margin-top:8px;
    font-size:14px;
}

/* цена меньше */
.price{
    font-size:22px;
    color:#00ffcc;
    font-weight:bold;
}

/* график меньше (ВАЖНО) */
canvas{
    width:100%;
    height:140px;
    border-radius:10px;
}

/* кнопки большие как на телефоне */
button{
    width:100%;
    margin-top:6px;
    padding:12px;
    border:none;
    border-radius:10px;
    font-weight:bold;
    font-size:14px;
    background:linear-gradient(135deg,#00ffcc,#00aaff);
}

/* вкладки */
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
    font-size:13px;
}

.active{
    background:#00ffcc;
    color:black;
}

/* убираем скролл */
html,body{
    height:100%;
}
</style>
</head>

<body>

<div class="container">

<div class="card">
💰 <b id="money">1000</b>$ | 🪙 <b id="coin">0</b>
</div>

<div class="tabs">
<div class="tab active" id="t1" onclick="openTab('market')">📈</div>
<div class="tab" id="t2" onclick="openTab('shop')">🛒</div>
</div>

<!-- 📈 MARKET -->
<div id="marketTab">

<div class="card">
📈 <div class="price" id="price">100</div>
<canvas id="c"></canvas>
</div>

<div class="card">
<button onclick="buy()">BUY</button>
<button onclick="sell()">SELL</button>
</div>

<div class="card">
📊 PnL: <span id="pnl">0</span>
</div>

</div>

<!-- 🛒 SHOP -->
<div id="shopTab" style="display:none;">

<div class="card">
<h3>🛒 SHOP</h3>

<button id="luckBtn" onclick="buyItem('luck')">🍀 x2 — 1200$</button>
<button id="magnetBtn" onclick="buyItem('magnet')">💰 x2 — 2000$</button>
<button id="botBtn" onclick="buyBot()">🤖 BOT — 3500$</button>

</div>

</div>

</div>

<script>

let price=100;
let money=1000;
let coin=0;

let luck=1;
let magnet=1;
let bot=false;

let bought={luck:false,magnet:false,bot:false};

let history=[100];

// 📱 вкладки
function openTab(tab){
    document.getElementById("marketTab").style.display = tab==="market"?"block":"none";
    document.getElementById("shopTab").style.display = tab==="shop"?"block":"none";

    document.getElementById("t1").classList.remove("active");
    document.getElementById("t2").classList.remove("active");

    if(tab==="market") document.getElementById("t1").classList.add("active");
    else document.getElementById("t2").classList.add("active");
}

// 📈 рынок (65/35)
function market(){

    let roll=Math.random();
    let change=0;

    if(roll<0.65) change=Math.random()*10;
    else change=-Math.random()*12;

    change*=luck;

    if(Math.random()<0.03)
        change+=(Math.random()<0.5?-1:1)*40;

    price=Math.max(1,price+change);

    history.push(price);
    if(history.length>45) history.shift();

    draw();
    update();

    if(bot) botTrade();
}

// 📊 график
function draw(){
    let c=document.getElementById("c");
    let ctx=c.getContext("2d");

    c.width=c.offsetWidth;
    c.height=c.offsetHeight;

    ctx.clearRect(0,0,c.width,c.height);

    ctx.beginPath();
    ctx.strokeStyle="#00ffcc";
    ctx.lineWidth=2;

    let step=c.width/history.length;

    for(let i=0;i<history.length;i++){
        let x=i*step;
        let y=c.height-(history[i]/200)*c.height;

        if(i==0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
    }

    ctx.stroke();
}

// 💰 update
function update(){
    document.getElementById("price").innerText=price.toFixed(2);
    document.getElementById("money").innerText=money.toFixed(2);
    document.getElementById("coin").innerText=coin;

    let pnl=(coin*price*magnet+money)-1000;
    document.getElementById("pnl").innerText=pnl.toFixed(2);
}

// 💰 buy/sell
function buy(){
    if(money>=price){
        money-=price;
        coin++;
    }
}

function sell(){
    if(coin>0){
        money+=price*magnet;
        coin--;
    }
}

// 🛒 shop
function buyItem(type){

    if(type==="luck" && !bought.luck && money>=1200){
        money-=1200;
        luck=2;
        bought.luck=true;
        document.getElementById("luckBtn").innerText="✔";
    }

    if(type==="magnet" && !bought.magnet && money>=2000){
        money-=2000;
        magnet=2;
        bought.magnet=true;
        document.getElementById("magnetBtn").innerText="✔";
    }
}

function buyBot(){
    if(!bought.bot && money>=3500){
        money-=3500;
        bot=true;
        bought.bot=true;
        document.getElementById("botBtn").innerText="✔";
    }
}

// 🤖 bot
function botTrade(){

    let last=history[history.length-1];
    let prev=history[history.length-2];

    if(last<prev && money>=price){
        money-=price;
        coin++;
    }

    if(last>prev && coin>0){
        money+=price*magnet;
        coin--;
    }
}

setInterval(market,800);
update();

</script>

</body>
</html>
