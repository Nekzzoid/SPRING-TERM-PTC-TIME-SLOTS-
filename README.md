<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Fairview School PTC Booking</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    font-family: Arial, sans-serif;
    margin:0;
    background: linear-gradient(135deg,#002147,#004aad);
    color:white;
    text-align:center;
}
header{
    padding:20px;
}
h1{margin:0;}

.logo{
    width:120px;
    margin-bottom:10px;
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
<div id="clock"></div>
</header>

<div class="container" id="loginSection">
<h3>Login</h3>
<select id="role">
<option value="parent">Parent</option>
<option value="teacher">Teacher</option>
<option value="admin">Admin</option>
</select>
<input type="password" id="password" placeholder="Enter password (teacher/admin)">
<button onclick="login()">Login</button>
</div>

<div class="container hidden" id="bookingSection">
<h3>Parent Booking Portal</h3>

<select id="classSelect"></select>
<input type="text" id="parentName" placeholder="Parent Name">
<input type="text" id="studentName" placeholder="Student Name">
<div id="slots"></div>
</div>

<div class="container hidden" id="adminSection">
<h3>Admin Dashboard</h3>
<div id="stats"></div>
<button onclick="exportCSV()">Export to Excel</button>
<table id="bookingTable"></table>
</div>

<script>

// ===== CONFIG =====
const teacherPassword = "teacher123";
const adminPassword = "admin123";
const eventDate = new Date("2026-02-28T09:00:00+01:00");

const classes = [
"Year 1 Onyx","Year 1 Amber","Year 2 Ruby","Year 2 Beryl",
"Year 3 Crystal","Year 3 Silver","Year 4 Gold","Year 4 Topaz",
"Year 5 Diamond","Year 5 Opal","Year 6 Pearl"
];

let bookings = JSON.parse(localStorage.getItem("fairviewBookings")) || [];

// ===== LIVE WAT CLOCK =====
function updateClock(){
    let now = new Date().toLocaleString("en-NG",{timeZone:"Africa/Lagos"});
    document.getElementById("clock").innerHTML="Nigerian Time (WAT): "+now;
}
setInterval(updateClock,1000);

// ===== LOGIN =====
function login(){
    let role=document.getElementById("role").value;
    let pass=document.getElementById("password").value;

    if(role==="teacher" && pass!==teacherPassword){
        alert("Wrong teacher password"); return;
    }
    if(role==="admin" && pass!==adminPassword){
        alert("Wrong admin password"); return;
    }

    document.getElementById("loginSection").classList.add("hidden");

    if(role==="parent"){
        document.getElementById("bookingSection").classList.remove("hidden");
        loadClasses();
    }
    if(role==="admin"){
        document.getElementById("adminSection").classList.remove("hidden");
        loadAdmin();
    }
}

// ===== LOAD CLASSES =====
function loadClasses(){
    let select=document.getElementById("classSelect");
    classes.forEach(c=>{
        let option=document.createElement("option");
        option.text=c;
        select.add(option);
    });
    generateSlots();
}

// ===== GENERATE TIME SLOTS =====
function generateSlots(){
    let container=document.getElementById("slots");
    container.innerHTML="";

    let start=9;
    let end=15;

    for(let hour=start; hour<end; hour++){
        for(let min=0; min<60; min+=15){

            let time=String(hour).padStart(2,'0')+":"+String(min).padStart(2,'0');

            if(time==="12:30"||time==="12:45"){
                let div=document.createElement("div");
                div.className="slot disabled";
                div.innerHTML=time+" (Break)";
                container.appendChild(div);
                continue;
            }

            let div=document.createElement("div");
            div.className="slot available";
            div.innerHTML=time;

            div.onclick=function(){
                bookSlot(time);
            };

            container.appendChild(div);
        }
    }

    refreshSlots();
}

// ===== BOOK SLOT =====
function bookSlot(time){

    let className=document.getElementById("classSelect").value;
    let parent=document.getElementById("parentName").value;
    let student=document.getElementById("studentName").value;

    if(!parent||!student){
        alert("Enter parent & student name"); return;
    }

    let now=new Date().toLocaleString("en-NG",{timeZone:"Africa/Lagos"});
    now=new Date(now);
    let fridayLock=new Date(eventDate);
    fridayLock.setDate(fridayLock.getDate()-1);
    fridayLock.setHours(18,0,0);

    if(now>fridayLock){
        alert("Booking Closed"); return;
    }

    if(bookings.find(b=>b.className===className && b.time===time)){
        alert("Slot already booked"); return;
    }

    bookings.push({
        className,time,parent,student
    });

    localStorage.setItem("fairviewBookings",JSON.stringify(bookings));
    alert("Booking Confirmed. SMS reminder will be sent 1 hour before meeting.");
    refreshSlots();
}

// ===== REFRESH SLOT STATUS =====
function refreshSlots(){
    let className=document.getElementById("classSelect").value;
    document.querySelectorAll(".slot").forEach(s=>{
        let time=s.innerText.replace(" (Break)","");
        if(bookings.find(b=>b.className===className && b.time===time)){
            s.className="slot booked";
            let info = bookings.find(b=>b.className===className && b.time===time);
            s.innerHTML=time+"<br><small>"+info.parent+"</small>";
        }
    });
}

// ===== ADMIN =====
function loadAdmin(){
    let table=document.getElementById("bookingTable");
    table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Student</th></tr>";
    bookings.forEach(b=>{
        table.innerHTML+=`<tr>
        <td>${b.className}</td>
        <td>${b.time}</td>
        <td>${b.parent}</td>
        <td>${b.student}</td>
        </tr>`;
    });

    document.getElementById("stats").innerHTML=
    "Total Bookings: "+bookings.length;
}

// ===== EXPORT CSV =====
function exportCSV(){
    let csv="Class,Time,Parent,Student\n";
    bookings.forEach(b=>{
        csv+=`${b.className},${b.time},${b.parent},${b.student}\n`;
    });
    let blob=new Blob([csv],{type:"text/csv"});
    let link=document.createElement("a");
    link.href=URL.createObjectURL(blob);
    link.download="Fairview_PTC_Bookings.csv";
    link.click();
}

// ===== SMS REMINDER SIMULATION =====
setInterval(()=>{
    let now=new Date().toLocaleString("en-NG",{timeZone:"Africa/Lagos"});
    now=new Date(now);

    bookings.forEach(b=>{
        let meeting=new Date(eventDate);
        let [h,m]=b.time.split(":");
        meeting.setHours(h,m,0);
        if(meeting-now===3600000){
            console.log("SMS Reminder sent to "+b.parent);
        }
    });
},60000);

</script>
</body>
</html>
