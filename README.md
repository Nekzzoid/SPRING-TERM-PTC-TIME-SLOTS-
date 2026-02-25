<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fairview School | PTC Booking 2026</title>

<style>
:root{
 --green:#138a36;
 --yellow:#f2e205;
 --red:#e10600;
 --white:#ffffff;
}

body{
 margin:0;
 font-family:Segoe UI, sans-serif;
 background:linear-gradient(135deg,var(--green),#0b5d25);
 color:white;
 text-align:center;
}

/* Header */
header{ padding:30px 20px; }
.logo{ width:110px; }

/* Animated Sun */
.sun{
 width:100px;
 height:100px;
 background:var(--yellow);
 border-radius:50%;
 margin:20px auto;
 animation:spin 25s linear infinite;
 box-shadow:0 0 30px var(--yellow);
}
@keyframes spin{
 from{transform:rotate(0deg);}
 to{transform:rotate(360deg);}
}

/* Countdown */
.countdown{
 display:flex;
 justify-content:center;
 flex-wrap:wrap;
 gap:15px;
}
.box{
 background:rgba(255,255,255,0.15);
 padding:15px;
 border-radius:12px;
 min-width:80px;
}
.number{ font-size:30px; font-weight:bold; }
.label{ font-size:12px; }

/* Booking Section */
.booking{
 background:white;
 color:black;
 margin:30px auto;
 padding:25px;
 border-radius:15px;
 max-width:500px;
}

.booking h3{ color:var(--green); }

select,input{
 width:100%;
 padding:10px;
 margin:8px 0;
 border-radius:8px;
 border:1px solid #ccc;
}

button{
 background:var(--yellow);
 border:none;
 padding:12px;
 width:100%;
 border-radius:25px;
 font-weight:bold;
 cursor:pointer;
 margin-top:10px;
}

button:hover{
 background:var(--green);
 color:white;
}

#classLinkBtn{
 margin-top:10px;
 display:none;
}

footer{
 margin-top:40px;
 padding:15px;
 color:var(--yellow);
}
</style>
</head>

<body>

<header>
<img src="fairview-logo.png" class="logo">
<h1>Parent-Teacher Conference 2026</h1>
<p>Saturday, 28 February 2026 | 9:00 AM (WAT)</p>
</header>

<div class="sun"></div>

<h2>Countdown to PTC</h2>
<div class="countdown" id="countdown">
<div class="box"><div class="number" id="days">0</div><div class="label">DAYS</div></div>
<div class="box"><div class="number" id="hours">0</div><div class="label">HOURS</div></div>
<div class="box"><div class="number" id="minutes">0</div><div class="label">MINUTES</div></div>
<div class="box"><div class="number" id="seconds">0</div><div class="label">SECONDS</div></div>
</div>

<!-- BOOKING FORM -->
<div class="booking">
<h3>📅 Book Your Appointment</h3>

<input type="text" id="parentName" placeholder="Parent Full Name" required>
<input type="tel" id="phone" placeholder="Phone Number" required>
<input type="email" id="email" placeholder="Email Address" required>

<select id="classSelect" onchange="updateTeachers()" required>
<option value="">Select Class</option>
<option value="Year 1 Onyx">Year 1 Onyx</option>
<option value="Year 1 Amber">Year 1 Amber</option>
<option value="Year 2 Ruby">Year 2 Ruby</option>
<option value="Year 2 Beryl">Year 2 Beryl</option>
<option value="Year 3 Crystal">Year 3 Crystal</option>
<option value="Year 3 Silver">Year 3 Silver</option>
<option value="Year 4 Gold">Year 4 Gold</option>
<option value="Year 4 Topaz">Year 4 Topaz</option>
<option value="Year 5 Diamond">Year 5 Diamond</option>
<option value="Year 5 Opal">Year 5 Opal</option>
<option value="Year 6 Pearl">Year 6 Pearl</option>
</select>

<select id="teacherSelect" required>
<option value="">Select Teacher</option>
</select>

<input type="datetime-local" id="appointmentDate" required>

<button onclick="submitBooking()">Confirm Booking</button>

<button id="classLinkBtn" onclick="openClassLink()">
🔗 Open Class Link
</button>

</div>

<footer>
Soaring High 🚀 | We are the heartbeat of Fairview - Primary department
 Mr. E
</footer>

<script>
// Countdown
const eventDate = new Date("2026-02-28T09:00:00+01:00").getTime();

setInterval(()=>{
 const now=new Date().getTime();
 const distance=eventDate-now;

 if(distance<=0){
  document.getElementById("countdown").innerHTML="<h2>🎉 PTC HAS STARTED!</h2>";
  return;
 }

 document.getElementById("days").innerHTML=Math.floor(distance/(1000*60*60*24));
 document.getElementById("hours").innerHTML=Math.floor((distance%(1000*60*60*24))/(1000*60*60));
 document.getElementById("minutes").innerHTML=Math.floor((distance%(1000*60*60))/(1000*60));
 document.getElementById("seconds").innerHTML=Math.floor((distance%(1000*60))/1000);

},1000);

// Teachers per class
const teachers = {
 "Year 1 Onyx": ["Ms. Victoria"],
 "Year 1 Amber": ["Ms. Rita, Judith"],
 "Year 2 Ruby": ["Ms. Irene, Victoria"],
 "Year 2 Beryl": ["Ms. Ifeoluwa"],
 "Year 3 Crystal": ["Ms. Zainab"],
 "Year 3 Silver": ["Ms. Mary"],
 "Year 4 Gold": ["Ms. Maryjane"],
 "Year 4 Topaz": ["Mr. Jacob"],
 "Year 5 Diamond": ["Mr. Michael"],
 "Year 5 Opal": ["Ms. Ayoola"],
 "Year 6 Pearl": ["Ms. Imaobong"]
};

// External class links
const classLinks = {
 "Year 1 Onyx": "https://calendar.google.com/calendar/u/0/appointments/schedules/AcZssZ0rYRtfsS0YSZnutOo0_TqW8oPJgK-F9g_qKDUeRw0uPHg7uYmdKwIzhwDfRXBmetIxoEUEVpKe",
 "Year 1 Amber": "https://calendar.app.google/JkBUKhqXFwFCFuLs7",
 "Year 2 Ruby": "https://meet.google.com/ruby",
 "Year 2 Beryl": "https://calendar.app.google/dgdvBJFCeTH6AchZ8",
 "Year 3 Crystal": "https://calendar.app.google/Y83p4YJ5PEdu6p8g9",
 "Year 3 Silver": "https://calendar.app.google/6MLhowC1ff6dKVDk6",
 "Year 4 Gold": "https://fairview.edu/gold",
 "Year 4 Topaz": "https://calendar.app.google/uMppKwDXSQtbu8CQ8",
 "Year 5 Diamond": "https://calendar.app.google/HRv1sGNca44dymbN9",
 "Year 5 Opal": "https://calendar.app.google/9nMPUL3aaayTJPcG6",
 "Year 6 Pearl": "https://calendar.app.google/JkBUKhqXFwFCFuLs7"
};

function updateTeachers(){
 const classValue = document.getElementById("classSelect").value;
 const teacherSelect = document.getElementById("teacherSelect");
 const linkBtn = document.getElementById("classLinkBtn");

 teacherSelect.innerHTML = "<option value=''>Select Teacher</option>";

 if(teachers[classValue]){
  teachers[classValue].forEach(teacher=>{
   let option = document.createElement("option");
   option.value = teacher;
   option.text = teacher;
   teacherSelect.appendChild(option);
  });
 }

 // Show link button if available
 if(classLinks[classValue]){
   linkBtn.style.display = "block";
 } else {
   linkBtn.style.display = "none";
 }
}

document.getElementById("appointmentDate").min = new Date().toISOString().slice(0,16);

// Booking submit to Google Apps Script backend
function submitBooking(){

 const data={
  name:document.getElementById("parentName").value,
  phone:document.getElementById("phone").value,
  email:document.getElementById("email").value,
  class:document.getElementById("classSelect").value,
  teacher:document.getElementById("teacherSelect").value,
  date:document.getElementById("appointmentDate").value.split("T")[0],
  time:document.getElementById("appointmentDate").value.split("T")[1]
 };

 fetch("PASTE_YOUR_GOOGLE_APPS_SCRIPT_URL_HERE",{
  method:"POST",
  body:JSON.stringify(data)
 })
 .then(res=>res.text())
 .then(response=>{
   if(response=="Slot already booked"){
     alert("❌ This time slot is already booked. Please choose another.");
   }else{
     alert("✅ Booking Confirmed! Email & SMS Sent.");
   }
 });

}

function openClassLink(){
 const classValue = document.getElementById("classSelect").value;
 if(classLinks[classValue]){
  window.open(classLinks[classValue], "_blank");
 }
}
</script>

</body>
</html>
