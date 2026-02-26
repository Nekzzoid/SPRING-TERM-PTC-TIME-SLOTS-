
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Fairview PTC System</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
/* ANIMATED FAIRVIEW BACKGROUND */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    text-align: center;
    color: #000;

    background: linear-gradient(120deg, #0b5d25, #f2e205, #002147);
    background-size: 300% 300%;
    animation: gradientMove 12s ease infinite;
}

@keyframes gradientMove {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}

/* MOBILE FRIENDLY CONTAINER */
.container {
    background: rgba(255,255,255,0.95);
    margin: 20px auto;
    padding: 20px;
    border-radius: 15px;
    width: 95%;
    max-width: 1200px;
}

@media (max-width: 768px) {
    .container {
        width: 98%;
        padding: 15px;
    }

    table {
        font-size: 12px;
    }

    .slot {
        min-width: 60px;
        font-size: 11px;
    }
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

.available{
    background:#28a745;
}

.selected{
    background:#ffd700;
    color:black;
}

.booked{
    background:#dc3545;
    color:white;
}

.disabled{
    background:gray;
    cursor:not-allowed;
}

.hidden{display:none;}

table{
    width:100%;
    border-collapse:collapse;
}

th,td{
    border:1px solid #ccc;
    padding:6px;
}
</style>
</head>
<body>

<header>
<h1>FAIRVIEW SCHOOL</h1>
<h3>PTC Booking System</h3>
</header>

<!-- LOGIN -->
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

<!-- LOGOUT -->
<div class="container hidden" id="logoutSection">
<button onclick="logout()">Logout</button>
</div>

<!-- PARENT VIEW (READ ONLY BOOKINGS) -->
<div class="container hidden" id="parentSection">
<h3>Parent View (All Bookings)</h3>
<table id="parentTable"></table>
</div>

<!-- BOOKING SECTION -->
<div class="container hidden" id="bookingSection">
<h3>Book Appointment</h3>

<input type="text" id="parentName" placeholder="Your Name">
<input type="text" id="phone" placeholder="Phone Number">

<select id="teacherSelect">
<option value="">Select Class Teacher</option>
<option value="MsVictoria">Ms Victoria</option>
<option value="MsRita">Ms Rita</option>
<option value="MsJudith">Ms Judith</option>
<option value="MsIfeoluwa">Ms Ifeoluwa</option>
<option value="MsZainab">Ms Zainab</option>
<option value="MsMary">Ms Mary</option>
<option value="MsMaryjane">Ms Maryjane</option>
<option value="MrJacob">Mr Jacob</option>
<option value="MsAyoola">Ms Ayoola</option>
<option value="MsImaobong">Ms Imaobong</option>
</select>

<select id="classSelect" onchange="generateSlots()"></select>

<div id="slots"></div>

<button onclick="submitBooking()">Book Appointment</button>
</div>

<!-- TEACHER / ADMIN DASHBOARD -->
<div class="container hidden" id="adminSection">
<h3 id="dashboardTitle"></h3>
<table id="bookingTable"></table>
<button onclick="exportCSV()">Export to Excel</button>
</div>

<script>

/* TEACHER PASSWORDS */
const teachers = {
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

let userRole = "public";
let teacherId = "";
let bookings = JSON.parse(localStorage.getItem("fairviewBookings")) || [];

/* LOGIN */
function login(){
 const role = document.getElementById("role").value;
 const id = document.getElementById("teacherId").value;
 const pass = document.getElementById("password").value;

 if(role === "teacher"){
   if(!id || teachers[id] !== pass){
     alert("Invalid teacher ID or password");
     return;
   }
   teacherId = id;
   userRole = "teacher";
 }

 if(role === "admin"){
   if(pass !== "admin123"){
     alert("Invalid admin password");
     return;
   }
   userRole = "admin";
 }

 if(role === "parent"){
   userRole = "parent";
 }

 document.getElementById("loginSection").classList.add("hidden");
 document.getElementById("logoutSection").classList.remove("hidden");

 if(userRole === "parent"){
   document.getElementById("bookingSection").classList.remove("hidden");
   document.getElementById("parentSection").classList.remove("hidden");
   loadClasses();
   loadParentTable();
 }

 if(userRole === "teacher" || userRole === "admin"){
   document.getElementById("adminSection").classList.remove("hidden");
   loadDashboard();
 }
}

/* LOGOUT */
function logout(){
 location.reload();
}

/* CLASSES */
const classes = [
"Year 1 Onyx","Year 1 Amber","Year 2 Ruby","Year 2 Beryl",
"Year 3 Crystal","Year 3 Silver","Year 4 Gold","Year 4 Topaz",
"Year 5 Diamond","Year 5 Opal","Year 6 Pearl"
];

function loadClasses(){
 let select = document.getElementById("classSelect");
 classes.forEach(c=>{
   let opt=document.createElement("option");
   opt.text=c;
   select.add(opt);
 });
 generateSlots();
}

/* SLOT SELECTION */
let selectedTime = "";

function generateSlots(){
 let container=document.getElementById("slots");
 container.innerHTML="";

 for(let hour=9; hour<15; hour++){
  for(let min=0; min<60; min+=15){

    let time = String(hour).padStart(2,'0')+":"+String(min).padStart(2,'0');

    if(time==="12:30" || time==="12:45"){
      let div=document.createElement("div");
      div.className="slot disabled";
      div.innerHTML=time+" (Break)";
      container.appendChild(div);
      continue;
    }

    let div=document.createElement("div");
    div.className="slot available";
    div.innerHTML=time;
    div.onclick=()=>selectSlot(div, time);
    container.appendChild(div);
  }
 }
}

function selectSlot(div, time){
 document.querySelectorAll(".slot").forEach(s=>s.classList.remove("selected"));
 div.classList.add("selected");
 selectedTime = time;
}

/* BOOK SLOT */
function submitBooking(){
 let parent=document.getElementById("parentName").value;
 let phone=document.getElementById("phone").value;
 let teacher=document.getElementById("teacherSelect").value;
 let className=document.getElementById("classSelect").value;

 if(!parent || !phone || !teacher || !selectedTime){
  alert("Fill all fields and select time");
  return;
 }

 if(bookings.find(b=>b.className===className && b.time===selectedTime)){
  alert("Slot already booked");
  return;
 }

 bookings.push({
  className,
  time:selectedTime,
  parent,
  phone,
  teacher
 });

 localStorage.setItem("fairviewBookings",JSON.stringify(bookings));
 loadParentTable();
 alert("Booking successful");
}

/* PARENT VIEW */
function loadParentTable(){
 let table=document.getElementById("parentTable");
 table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th><th>Teacher</th></tr>";

 bookings.forEach(b=>{
  table.innerHTML+=`
  <tr>
    <td>${b.className}</td>
    <td>${b.time}</td>
    <td>${b.parent}</td>
    <td>${b.phone}</td>
    <td>${b.teacher}</td>
  </tr>`;
 });
}

/* DASHBOARD */
function loadDashboard(){
 let table=document.getElementById("bookingTable");
 table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th></tr>";

 let data = (userRole==="teacher")
   ? bookings.filter(b=>b.teacher===teacherId)
   : bookings;

 document.getElementById("dashboardTitle").innerText =
   (userRole==="admin") ? "Admin Dashboard" : "Teacher Dashboard ("+teacherId+")";

 data.forEach(b=>{
  table.innerHTML+=`
  <tr>
    <td>${b.className}</td>
    <td>${b.time}</td>
    <td>${b.parent}</td>
    <td>${b.phone}</td>
  </tr>`;
 });
}

/* EXPORT TO EXCEL */
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

</script>
</body>
</html>
