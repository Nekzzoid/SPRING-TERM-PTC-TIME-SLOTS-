<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Fairview PTC System</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    font-family: Arial, sans-serif;
    margin:0;
    background: linear-gradient(135deg,#002147,#004aad);
    color:white;
    text-align:center;
}

header{ padding:20px; }

.logo{
    width:120px;
}

.container{
    background:white;
    color:black;
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
    padding:8px;
    border-radius:6px;
    cursor:pointer;
    font-size:12px;
    min-width:80px;
}

.available{
    background:#28a745;
    animation:pulse 2s infinite;
}

.booked{
    background:#dc3545;
    color:white;
}

.disabled{
    background:gray;
    cursor:not-allowed;
}

@keyframes pulse{
    0%{opacity:1;}
    50%{opacity:0.6;}
    100%{opacity:1;}
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
<img src="https://cdn.phototourl.com/uploads/2026-02-26-8f429b47-8d09-4435-8a91-9306a9fcf9f2.png" class="logo">
<h1>FAIRVIEW SCHOOL</h1>
<h3>PTC Booking System</h3>
</header>

<!-- LOGIN WITH TEACHER ID -->
<div class="container" id="loginSection">
<h3>Login</h3>

<select id="role">
<option value="parent">Parent</option>
<option value="teacher">Teacher</option>
<option value="admin">Admin</option>
</select>

<input type="text" id="userId" placeholder="Teacher ID (if teacher)">
<input type="password" id="password" placeholder="Password">

<button onclick="login()">Login</button>
</div>

<!-- PARENT DASHBOARD (CAN SEE ALL BOOKINGS) -->
<div class="container hidden" id="bookingSection">
<h3>Parent Dashboard (View All Bookings)</h3>

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

<!-- DASHBOARD (TEACHER OR ADMIN OR PARENT VIEW) -->
<div class="container hidden" id="adminSection">
<h3 id="dashboardTitle"></h3>

<table id="bookingTable"></table>
</div>

<script>

// ===== DATABASE (TEACHER IDS) =====
const users = {
 teachers: {
  "MsVictoria":"pass123",
  "MsRita":"pass123",
  "MsJudith":"pass123",
  "MsIfeoluwa":"pass123",
  "MsZainab":"pass123",
  "MsMary":"pass123",
  "MsMaryjane":"pass123",
  "MrJacob":"pass123",
  "MsAyoola":"pass123",
  "MsImaobong":"pass123"
 },
 admin: {
  "admin":"admin123"
 }
};

let userRole = "public";
let teacherId = "";
let bookings = JSON.parse(localStorage.getItem("fairviewBookings")) || [];

// ===== LOGIN WITH TEACHER ID =====
function login(){
 const role = document.getElementById("role").value;
 const id = document.getElementById("userId").value;
 const pass = document.getElementById("password").value;

 if(role === "teacher"){
   if(!id || users.teachers[id] !== pass){
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

 if(userRole === "parent"){
   document.getElementById("bookingSection").classList.remove("hidden");
   loadClasses();
 }

 if(userRole === "teacher" || userRole === "admin"){
   document.getElementById("adminSection").classList.remove("hidden");
   loadDashboard();
 }
}

// ===== CLASSES =====
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

// ===== SLOT GENERATION (9–3, 15 MIN, BREAK 12:30–1) =====
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
    div.onclick=()=>selectSlot(time);
    container.appendChild(div);
  }
 }
 refreshSlots();
}

function selectSlot(time){
 selectedTime = time;
 alert("Selected: "+time);
}

// ===== BOOK SLOT (PARENT CAN SEE ALL BOOKINGS) =====
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
 alert("Booking confirmed");

 refreshSlots();
 loadDashboard();
}

// ===== REFRESH SLOT STATUS (VISIBLE TO ALL) =====
function refreshSlots(){
 let className=document.getElementById("classSelect").value;

 document.querySelectorAll(".slot").forEach(s=>{
  let time=s.innerText.replace(" (Break)","");
  let booked = bookings.find(b=>b.className===className && b.time===time);

  if(booked){
    s.className="slot booked";
    s.innerHTML=time+"<br><small>"+booked.parent+"</small>";
  }
 });
}

// ===== DASHBOARD (SHOW ALL BOOKINGS) =====
function loadDashboard(){
 let table=document.getElementById("bookingTable");
 table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th><th>Teacher</th></tr>";

 document.getElementById("dashboardTitle").innerText =
   (userRole==="admin") ? "Admin Dashboard" : "Teacher Dashboard ("+teacherId+")";

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

</script>
</body>
</html>
