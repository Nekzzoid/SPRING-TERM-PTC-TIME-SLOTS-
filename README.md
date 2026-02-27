

<html>
<head>
<meta charset="UTF-8">
<title>Fairview School PTC Enterprise System</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{
margin:0;
font-family:Arial, sans-serif;
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
background:white;
margin:20px auto;
padding:20px;
border-radius:15px;
width:95%;
max-width:1200px;
}
select,input,button{
padding:8px;margin:5px;border-radius:6px;border:1px solid #ccc;
}
button{background:#004aad;color:white;cursor:pointer;}
button:hover{background:#002147;}
.slot{
display:inline-block;margin:5px;padding:10px;border-radius:6px;
cursor:pointer;font-size:12px;min-width:80px;
}
.available{background:#28a745;color:white;}
.selected{background:#ffd700;color:black;}
.booked{background:#dc3545;color:white;}
.disabled{background:gray;}
.hidden{display:none;}
table{width:100%;border-collapse:collapse;margin-top:15px;}
th,td{border:1px solid #ccc;padding:6px;font-size:12px;}
.footer{margin-top:40px;padding:20px;background:#002147;color:white;}
</style>
</head>
<body>

<img src="https://imgtolinkx.com/i/ZgUxsuIJ" width="140">
<h1>FAIRVIEW SCHOOL</h1>
<h3>Parent Teacher Conference Enterprise Booking</h3>
<h4>Summer Term 2025/2026 Academic Session</h4>

<div class="container" id="loginSection">
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
<input type="text" id="studentName" placeholder="Student Name">
<input type="text" id="phone" placeholder="Phone Number">
<select id="teacherSelect"></select>
<select id="classSelect"></select>
<div id="slots"></div>
<button onclick="submitBooking()">Book Appointment</button>
</div>

<div class="container hidden" id="dashboardSection">
<h3 id="dashboardTitle"></h3>

<div id="adminControls" class="hidden">
<button onclick="exportCSV()">Export CSV</button>
<button onclick="downloadPDF()">Download PDF</button>
<button onclick="resetAllBookings()">Reset All</button>
<button onclick="toggleBookingStatus()">Open / Close Booking</button>
<button onclick="togglePasswordPanel()">Change Admin Password</button>
<button onclick="showAnalytics()">Analytics</button>

<select id="filterTeacher" onchange="loadDashboard()">
<option value="">Filter Teacher</option>
</select>

<select id="filterClass" onchange="loadDashboard()">
<option value="">Filter Class</option>
</select>

<div id="passwordPanel" class="hidden">
<input type="password" id="oldPass" placeholder="Current Password">
<input type="password" id="newPass" placeholder="New Password">
<button onclick="changeAdminPassword()">Update</button>
</div>

<div id="analyticsSection" class="hidden">
<canvas id="analyticsChart"></canvas>
</div>

</div>

<table id="bookingTable"></table>
</div>

<div class="footer">
© <span id="year"></span> Fairview School Abuja | 5 Edwin Clark Crescent, Guzape, Abuja. Front Desk: 0911 919 5305. Email: info@fairviewschoolng.com.| Powered by Fairview IT Team
</div>

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
let userRole="",teacherId="",selectedTime="";
let bookingOpen=JSON.parse(localStorage.getItem("bookingStatus"));
if(bookingOpen===null) bookingOpen=true;

function login(){
let role=document.getElementById("role").value;
let id=document.getElementById("teacherId").value;
let pass=document.getElementById("password").value;

let adminPassword=localStorage.getItem("adminPass")||"admin123";

if(role==="teacher"){
if(!teachers[id]||teachers[id]!==pass){alert("Invalid Teacher Login");return;}
teacherId=id;userRole="teacher";
}
else if(role==="admin"){
if(pass!==adminPassword){alert("Invalid Admin Login");return;}
userRole="admin";
}
else{userRole="parent";}

document.getElementById("loginSection").classList.add("hidden");
document.getElementById("logoutSection").classList.remove("hidden");

if(userRole==="parent"){
loadParentOptions();
document.getElementById("bookingSection").classList.remove("hidden");
}

if(userRole==="teacher"||userRole==="admin"){
document.getElementById("dashboardSection").classList.remove("hidden");
loadDashboard();
if(userRole==="admin"){
document.getElementById("adminControls").classList.remove("hidden");
loadFilters();
}
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
opt.value=t;opt.text=t;tSel.add(opt);
});
classes.forEach(c=>{
let opt=document.createElement("option");
opt.value=c;opt.text=c;cSel.add(opt);
});
cSel.onchange=generateSlots;
}

function generateSlots(){
let container=document.getElementById("slots");
container.innerHTML="";
let className=document.getElementById("classSelect").value;

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

let isBooked=bookings.find(b=>b.className===className&&b.time===time);

if(isBooked){
div.className="slot booked";
div.innerHTML=time+" Booked";
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
if(!bookingOpen){alert("Booking closed by Admin");return;}

let parent=document.getElementById("parentName").value;
let phone=document.getElementById("phone").value;
let teacher=document.getElementById("teacherSelect").value;
let className=document.getElementById("classSelect").value;

if(!parent||!phone||!teacher||!selectedTime){alert("Fill all fields");return;}
if(bookings.find(b=>b.parent===parent)){alert("You already booked.");return;}
if(bookings.find(b=>b.className===className&&b.time===selectedTime)){alert("Slot taken");return;}

bookings.push({className,time:selectedTime,parent,phone,teacher});
localStorage.setItem("fairviewBookings",JSON.stringify(bookings));
generateSlots();
alert("Booking Successful");
}

function loadDashboard(){
let table=document.getElementById("bookingTable");
table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th><th>Teacher</th>";
if(userRole==="admin"){table.innerHTML+="<th>Action</th>";}
table.innerHTML+="</tr>";

let data=bookings;

if(userRole==="teacher"){
data=bookings.filter(b=>b.teacher===teacherId);
}

if(userRole==="admin"){
let tFilter=document.getElementById("filterTeacher").value;
let cFilter=document.getElementById("filterClass").value;
if(tFilter) data=data.filter(b=>b.teacher===tFilter);
if(cFilter) data=data.filter(b=>b.className===cFilter);
}

document.getElementById("dashboardTitle").innerText=
userRole==="admin"?"Admin Dashboard":"Teacher Dashboard ("+teacherId+")";

data.forEach((b,i)=>{
table.innerHTML+=`
<tr>
<td>${b.className}</td>
<td>${b.time}</td>
<td>${b.parent}</td>
<td>${b.phone}</td>
<td>${b.teacher}</td>
${userRole==="admin"?`<td><button onclick="deleteBooking(${i})">Reset</button></td>`:""}
</tr>`;
});
}

function deleteBooking(i){
if(!confirm("Reset this slot?"))return;
bookings.splice(i,1);
localStorage.setItem("fairviewBookings",JSON.stringify(bookings));
loadDashboard();
}

function resetAllBookings(){
if(!confirm("Reset ALL bookings?"))return;
bookings=[];
localStorage.removeItem("fairviewBookings");
loadDashboard();
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

function downloadPDF(){
let content="<h2>Fairview School PTC Report</h2><table border='1' cellpadding='5'><tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th><th>Teacher</th></tr>";
bookings.forEach(b=>{
content+=`<tr><td>${b.className}</td><td>${b.time}</td><td>${b.parent}</td><td>${b.phone}</td><td>${b.teacher}</td></tr>`;
});
content+="</table>";
let win=window.open("");
win.document.write(content);
win.print();
}

function toggleBookingStatus(){
bookingOpen=!bookingOpen;
localStorage.setItem("bookingStatus",JSON.stringify(bookingOpen));
alert(bookingOpen?"Booking Opened":"Booking Closed");
}

function togglePasswordPanel(){
document.getElementById("passwordPanel").classList.toggle("hidden");
}

function changeAdminPassword(){
let oldPass=document.getElementById("oldPass").value;
let newPass=document.getElementById("newPass").value;
let current=localStorage.getItem("adminPass")||"admin123";
if(oldPass!==current){alert("Wrong Password");return;}
localStorage.setItem("adminPass",newPass);
alert("Password Updated");
}

function loadFilters(){
let t=document.getElementById("filterTeacher");
let c=document.getElementById("filterClass");
Object.keys(teachers).forEach(x=>{
let o=document.createElement("option");o.value=x;o.text=x;t.add(o);
});
classes.forEach(x=>{
let o=document.createElement("option");o.value=x;o.text=x;c.add(o);
});
}

function showAnalytics(){
document.getElementById("analyticsSection").classList.toggle("hidden");
let counts={};
bookings.forEach(b=>{counts[b.teacher]=(counts[b.teacher]||0)+1;});
new Chart(document.getElementById("analyticsChart"),{
type:'bar',
data:{
labels:Object.keys(counts),
datasets:[{label:'Bookings',data:Object.values(counts),backgroundColor:'#004aad'}]
}
});
}
</script>

</body>
</html>
