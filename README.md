<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Fairview School PTC System</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
margin:0;
font-family:Arial, sans-serif;
color:#000;
text-align:center;
background: linear-gradient(120deg,#0b5d25,#f2e205,#002147);
background-size:300% 300%;
animation:gradientMove 12s ease infinite;
}

@keyframes gradientMove{
0%{background-position:0% 50%;}
50%{background-position:100% 50%;}
100%{background-position:0% 50%;}
}

.container{
background:rgba(255,255,255,0.95);
margin:20px auto;
padding:20px;
border-radius:15px;
width:95%;
max-width:1200px;
}

select,input,button{
padding:8px;
margin:5px;
border-radius:6px;
border:1px solid #ccc;
}

button{
background:#004aad;
color:white;
cursor:pointer;
}

button:hover{
background:#002147;
}

.slot{
display:inline-block;
margin:5px;
padding:10px;
border-radius:6px;
cursor:pointer;
font-size:12px;
min-width:80px;
}

.available{background:#28a745;color:white;}
.selected{background:#ffd700;color:black;}
.booked{background:#dc3545;color:white;}
.disabled{background:gray;cursor:not-allowed;}

.hidden{display:none;}

table{
width:100%;
border-collapse:collapse;
margin-top:15px;
}

th,td{
border:1px solid #ccc;
padding:6px;
font-size:12px;
}

@media(max-width:768px){
.slot{min-width:60px;font-size:11px;}
table{font-size:11px;}
}

/* Floating Social */
.social-icons{
position:fixed;
right:15px;
bottom:120px;
display:flex;
flex-direction:column;
gap:10px;
z-index:999;
}
.social-icons a{
width:45px;height:45px;
background:#004aad;color:white;
border-radius:50%;
display:flex;align-items:center;justify-content:center;
text-decoration:none;
font-weight:bold;
transition:0.3s;
}
.social-icons a:hover{
background:#ffd700;color:black;
transform:scale(1.1);
}

/* Back To Top */
#backToTop{
position:fixed;
bottom:30px;
right:15px;
background:#ffd700;
border:none;
padding:12px;
border-radius:50%;
cursor:pointer;
display:none;
}

/* Footer */
.footer{
margin-top:60px;
padding:40px 20px;
background:rgba(0,0,0,0.85);
color:white;
position:relative;
}

.footer a{
color:#ffd700;
text-decoration:none;
font-weight:bold;
}

.sun-glow{
width:120px;height:120px;
background:radial-gradient(circle,#ffd700 20%,transparent 70%);
position:absolute;
top:-40px;left:50%;
transform:translateX(-50%);
border-radius:50%;
animation:glowPulse 4s infinite;
opacity:0.4;
}

@keyframes glowPulse{
0%{transform:translateX(-50%) scale(1);}
50%{transform:translateX(-50%) scale(1.2);}
100%{transform:translateX(-50%) scale(1);}
}
</style>
</head>
<body>

<header>
<img src="https://imgtolinkx.com/i/ZgUxsuIJ" width="140">
<h1>FAIRVIEW SCHOOL</h1>
<h3>Parent Teacher Conference Booking</h3>
</header>

<div class="container" id="loginSection">
<h3>Login</h3>
<select id="role">
<option value="parent">Parent</option>
<option value="teacher">Teacher</option>
<option value="admin">Admin</option>
</select>
<input type="text" id="teacherId" placeholder="Teacher ID">
<input type="password" id="password" placeholder="Password">
<button onclick="login()">Login</button>
</div>

<div class="container hidden" id="logoutSection">
<button onclick="logout()">Logout</button>
</div>

<div class="container hidden" id="bookingSection">
<h3>Book Appointment</h3>
<input type="text" id="parentName" placeholder="Parent Name">
<input type="text" id="phone" placeholder="Phone Number">
<select id="teacherSelect"></select>
<select id="classSelect"></select>
<div id="slots"></div>
<button onclick="submitBooking()">Book Appointment</button>
</div>

<div class="container hidden" id="dashboardSection">
<h3 id="dashboardTitle"></h3>
<table id="bookingTable"></table>
<button onclick="exportCSV()">Export to Excel</button>
</div>

<div class="container hidden" id="parentViewSection">
<h3>All Bookings</h3>
<table id="parentTable"></table>
</div>

<div class="social-icons">
<a href="#">F</a>
<a href="#">I</a>
<a href="#">L</a>
</div>

<button id="backToTop" onclick="scrollToTop()">↑</button>

<footer class="footer">
<div class="sun-glow"></div>
<div>
Designed by 
<a href="https://your-link-here.com" target="_blank">
E. Nekzzoid (Mr. E)
</a>
</div>
<div style="margin-top:8px;">
© <span id="year"></span> Fairview School. All Rights Reserved.
</div>
<div style="margin-top:5px;font-size:13px;color:#ccc;">
Powered by Fairview Digital Systems
</div>
</footer>

<script>
document.getElementById("year").textContent=new Date().getFullYear();

const teachers={
"MsVictoria":"victoria123",
"MsRita":"rita123",
"MsJudith":"judith123",
"MsIfeoluwa":"ife123",
"MsZainab":"zainab123",
"MsMary":"mary123",
"MsMaryjane":"maryjane123",
"MrJacob":"jacob123",
"MsAyoola":"ayoola123",
"MsImaobong":"ima123"
};

const classes=[
"Year 1 Onyx","Year 1 Amber","Year 2 Ruby","Year 2 Beryl",
"Year 3 Crystal","Year 3 Silver","Year 4 Gold","Year 4 Topaz",
"Year 5 Diamond","Year 5 Opal","Year 6 Pearl"
];

let bookings=JSON.parse(localStorage.getItem("fairviewBookings"))||[];
let userRole="";
let teacherId="";
let selectedTime="";

function login(){
let role=document.getElementById("role").value;
let id=document.getElementById("teacherId").value;
let pass=document.getElementById("password").value;

if(role==="teacher"){
if(!teachers[id]||teachers[id]!==pass){alert("Invalid Teacher Login");return;}
teacherId=id;userRole="teacher";
}
else if(role==="admin"){
if(pass!=="admin123"){alert("Invalid Admin Password");return;}
userRole="admin";
}
else{userRole="parent";}

document.getElementById("loginSection").classList.add("hidden");
document.getElementById("logoutSection").classList.remove("hidden");

if(userRole==="parent"){
loadParentOptions();
document.getElementById("bookingSection").classList.remove("hidden");
document.getElementById("parentViewSection").classList.remove("hidden");
loadParentTable();
}

if(userRole==="teacher"||userRole==="admin"){
document.getElementById("dashboardSection").classList.remove("hidden");
loadDashboard();
}
}

function logout(){location.reload();}

function loadParentOptions(){
let tSel=document.getElementById("teacherSelect");
let cSel=document.getElementById("classSelect");
tSel.innerHTML="<option>Select Teacher</option>";
cSel.innerHTML="<option>Select Class</option>";
Object.keys(teachers).forEach(t=>{
let opt=document.createElement("option");
opt.text=t;opt.value=t;tSel.add(opt);
});
classes.forEach(c=>{
let opt=document.createElement("option");
opt.text=c;opt.value=c;cSel.add(opt);
});
cSel.onchange=generateSlots;
}

function generateSlots(){
let container=document.getElementById("slots");
container.innerHTML="";
for(let h=9;h<15;h++){
for(let m=0;m<60;m+=15){
let time=String(h).padStart(2,'0')+":"+String(m).padStart(2,'0');
let div=document.createElement("div");

if(time==="12:30"||time==="12:45"){
div.className="slot disabled";
div.innerHTML=time+" Break";
container.appendChild(div);
continue;
}

div.className="slot available";
div.innerHTML=time;
div.onclick=function(){
document.querySelectorAll(".slot").forEach(s=>s.classList.remove("selected"));
div.classList.add("selected");
selectedTime=time;
};
container.appendChild(div);
}
}
}

function submitBooking(){
let parent=document.getElementById("parentName").value;
let phone=document.getElementById("phone").value;
let teacher=document.getElementById("teacherSelect").value;
let className=document.getElementById("classSelect").value;

if(!parent||!phone||!teacher||!selectedTime){alert("Fill all fields");return;}

if(bookings.find(b=>b.className===className&&b.time===selectedTime)){
alert("Slot already booked");return;
}

bookings.push({className,time:selectedTime,parent,phone,teacher});
localStorage.setItem("fairviewBookings",JSON.stringify(bookings));
loadParentTable();
alert("Booking Successful");
}

function loadParentTable(){
let table=document.getElementById("parentTable");
table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Teacher</th></tr>";
bookings.forEach(b=>{
table.innerHTML+=`<tr><td>${b.className}</td><td>${b.time}</td><td>${b.parent}</td><td>${b.teacher}</td></tr>`;
});
}

function loadDashboard(){
let table=document.getElementById("bookingTable");
table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th></tr>";

let data=userRole==="teacher"?bookings.filter(b=>b.teacher===teacherId):bookings;

document.getElementById("dashboardTitle").innerText=
userRole==="admin"?"Admin Dashboard":"Teacher Dashboard ("+teacherId+")";

data.forEach(b=>{
table.innerHTML+=`<tr><td>${b.className}</td><td>${b.time}</td><td>${b.parent}</td><td>${b.phone}</td></tr>`;
});
}

function exportCSV(){
let csv="Class,Time,Parent,Phone,Teacher\n";
bookings.forEach(b=>{
csv+=`${b.className},${b.time},${b.parent},${b.phone},${b.teacher}\n`;
});
let blob=new Blob([csv],{type:"text/csv"});
let link=document.createElement("a");
link.href=URL.createObjectURL(blob);
link.download="Fairview_PTC_Bookings.csv";
link.click();
}

window.onscroll=function(){
if(document.documentElement.scrollTop>300){
document.getElementById("backToTop").style.display="block";
}else{
document.getElementById("backToTop").style.display="none";
}
};

function scrollToTop(){
window.scrollTo({top:0,behavior:"smooth"});
}
</script>

</body>
</html>
