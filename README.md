# probashi-khabar-hisab
Probashi Food Expense Calculator Android App
index.html
<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>প্রবাসী খাবার হিসাব</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;background:#f1f5f2;color:#18352b;
 font-family:Arial,"Noto Sans Bengali",sans-serif;
}
header{
 background:#087b52;color:white;padding:25px 18px;
 border-radius:0 0 24px 24px;
}
header h2{margin:0 0 8px}
header p{margin:0;color:#d8f5e7}
main{padding:16px;max-width:600px;margin:auto}
.card{
 background:white;border-radius:16px;padding:17px;
 margin-bottom:15px;box-shadow:0 3px 12px #0000000c;
}
.stats{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.stat{background:#e6f5ed;padding:16px;border-radius:13px}
.stat small{color:#547263}
.stat strong{display:block;font-size:22px;margin-top:8px}
h3{margin:0 0 14px}
label{display:block;margin:12px 0 6px;font-size:14px}
input,select{
 width:100%;padding:13px;border:1px solid #d6e2db;
 border-radius:10px;font-size:16px;background:white;
}
button{
 border:0;border-radius:10px;padding:13px 15px;
 background:#087b52;color:white;font-size:15px;
 font-weight:bold;cursor:pointer;
}
.primary{width:100%;margin-top:16px}
.secondary{background:#e6f5ed;color:#087b52}
.danger{background:#ffe9e9;color:#b52c2c;padding:8px}
.entry{
 border-bottom:1px solid #edf0ed;padding:12px 0;
 display:flex;justify-content:space-between;gap:8px;
 align-items:center;
}
.entry:last-child{border:0}
.entry small{color:#73857b}
footer{text-align:center;color:#82938a;padding:20px}
</style>
</head>
<body>

<header>
<h2>🍽️ প্রবাসী খাবার হিসাব</h2>
<p>আপনার প্রতিদিনের খাবারের খরচ রাখুন</p>
</header>

<main>
<div class="stats">
 <div class="stat">
  <small>আজকের খরচ</small>
  <strong id="today">৳০</strong>
 </div>
 <div class="stat">
  <small>এই মাসের খরচ</small>
  <strong id="month">৳০</strong>
 </div>
</div>

<div class="card" style="margin-top:15px">
<h3>💰 মাসিক বাজেট</h3>
<label>আপনার মাসিক বাজেট (টাকা)</label>
<input id="budget" type="number" min="0" placeholder="যেমন ১৫০০০">
<button class="primary" onclick="saveBudget()">বাজেট সংরক্ষণ</button>
<p id="remaining">বাকি বাজেট: ৳০</p>
</div>

<div class="card">
<h3>➕ নতুন খাবারের খরচ</h3>
<label>তারিখ</label>
<input type="date" id="date">
<label>খাবারের নাম</label>
<select id="food">
<option>সকালের নাস্তা</option>
<option>দুপুরের খাবার</option>
<option>রাতের খাবার</option>
<option>চা / নাস্তা</option>
<option>অন্যান্য</option>
</select>
<label>খরচ (বাংলাদেশি টাকা)</label>
<input type="number" id="amount" min="1" placeholder="টাকার পরিমাণ">
<label>বিস্তারিত (ঐচ্ছিক)</label>
<input id="note" placeholder="যেমন: ভাত, মাছ">
<button class="primary" onclick="addExpense()">খরচ যোগ করুন</button>
</div>

<div class="card">
<h3>📋 খরচের তালিকা</h3>
<div id="entries">এখনো কোনো খরচ যোগ করা হয়নি।</div>
</div>

<div class="card">
<h3>📊 হিসাব ডাউনলোড</h3>
<button class="primary secondary" onclick="downloadCSV()">CSV হিসাব ডাউনলোড</button>
<button class="primary danger" onclick="clearAll()">সব হিসাব মুছুন</button>
</div>
</main>

<footer>প্রবাসী খাবার হিসাব ❤️</footer>

<script>
const $=id=>document.getElementById(id);
let expenses=JSON.parse(localStorage.getItem("foodExpenses")||"[]");
let budget=Number(localStorage.getItem("foodBudget")||0);

function todayDate(){
 const d=new Date();
 return d.getFullYear()+"-"+String(d.getMonth()+1).padStart(2,"0")+"-"+String(d.getDate()).padStart(2,"0");
}
$("date").value=todayDate();
$("budget").value=budget;

function money(n){
 return "৳"+Number(n).toLocaleString("bn-BD");
}
function save(){
 localStorage.setItem("foodExpenses",JSON.stringify(expenses));
}
function saveBudget(){
 budget=Math.max(0,Number($("budget").value)||0);
 localStorage.setItem("foodBudget",budget);
 render();
 alert("বাজেট সংরক্ষণ হয়েছে!");
}
function addExpense(){
 const date=$("date").value;
 const food=$("food").value;
 const amount=Number($("amount").value);
 const note=$("note").value.trim();
 if(!date||!Number.isFinite(amount)||amount<=0){
  alert("সঠিক তারিখ ও খরচ লিখুন।");return;
 }
 expenses.unshift({
  id:Date.now(),date,food,amount,note
 });
 save();
 $("amount").value="";
 $("note").value="";
 render();
}
function removeExpense(id){
 if(!confirm("এই খরচটি মুছে ফেলবেন?"))return;
 expenses=expenses.filter(x=>x.id!==id);
 save();render();
}
function render(){
 const now=todayDate();
 const month=now.slice(0,7);
 const todayTotal=expenses.filter(x=>x.date===now)
 .reduce((s,x)=>s+x.amount,0);
 const monthTotal=expenses.filter(x=>x.date.startsWith(month))
 .reduce((s,x)=>s+x.amount,0);
 $("today").textContent=money(todayTotal);
 $("month").textContent=money(monthTotal);
 $("remaining").textContent="বাকি বাজেট: "+money(budget-monthTotal);

 const box=$("entries");
 if(!expenses.length){
  box.textContent="এখনো কোনো খরচ যোগ করা হয়নি।";
  return;
 }
 box.innerHTML="";
 expenses.forEach(x=>{
  const row=document.createElement("div");
  row.className="entry";
  const info=document.createElement("div");
  const title=document.createElement("strong");
  title.textContent=x.food;
  const detail=document.createElement("small");
  detail.textContent=x.date+" · "+x.note;
  info.append(title,document.createElement("br"),detail);
  const right=document.createElement("div");
  right.style.textAlign="right";
  const price=document.createElement("strong");
  price.textContent=money(x.amount);
  const del=document.createElement("button");
  del.className="danger";
  del.textContent="মুছুন";
  del.style.display="block";
  del.style.marginTop="6px";
  del.onclick=()=>removeExpense(x.id);
  right.append(price,del);
  row.append(info,right);
  box.append(row);
 });
}
function downloadCSV(){
 if(!expenses.length){alert("ডাউনলোড করার মতো হিসাব নেই।");return}
 const rows=[["Date","Food","Amount BDT","Note"],
 ...expenses.map(x=>[x.date,x.food,x.amount,x.note])];
 const csv="\uFEFF"+rows.map(r=>r.map(v=>'"'+String(v).replace(/"/g,'""')+'"').join(",")).join("\n");
 const blob=new Blob([csv],{type:"text/csv;charset=utf-8"});
 const url=URL.createObjectURL(blob);
 const a=document.createElement("a");
 a.href=url;a.download="probashi-food-expenses.csv";
 a.click();URL.revokeObjectURL(url);
}
function clearAll(){
 if(!confirm("সব হিসাব স্থায়ীভাবে মুছবেন?"))return;
 expenses=[];save();render();
}
render();
</script>
</body>
</html>
