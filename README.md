import os, zipfile, textwrap, json, csv, math, random
from pathlib import Path

base = Path("/mnt/data/SWYNEX-Final-IoT-Mini-Project")
base.mkdir(parents=True, exist_ok=True)

files = {
"index.html": """<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SWYNEX Remote IoT Monitoring Dashboard</title>
  <link rel="stylesheet" href="style.css">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
  <header>
    <div>
      <h1>Remote IoT Monitoring</h1>
      <p>SWYNEX Final IoT Mini-Project</p>
    </div>
    <span class="status"><i></i> System Online</span>
  </header>

  <main>
    <section class="cards">
      <div class="card"><h3>Temperature</h3><strong id="temp">-- °C</strong><p>Simulated sensor</p></div>
      <div class="card"><h3>Humidity</h3><strong id="humidity">-- %</strong><p>Simulated sensor</p></div>
      <div class="card"><h3>Device Status</h3><strong id="device">ONLINE</strong><p id="time">--</p></div>
      <div class="card"><h3>Readings</h3><strong id="count">0</strong><p>Live updates</p></div>
    </section>

    <section class="panel">
      <h2>Sensor Trends</h2>
      <canvas id="sensorChart"></canvas>
    </section>

    <section class="panel">
      <h2>IoT Architecture</h2>
      <div class="architecture">
        <div>🌡️<b>Sensors</b><small>Temperature<br>Humidity</small></div>
        <span>→</span>
        <div>📡<b>IoT Gateway</b><small>Data transfer<br>Simulation</small></div>
        <span>→</span>
        <div>☁️<b>Cloud Layer</b><small>Processing<br>Storage</small></div>
        <span>→</span>
        <div>📊<b>Dashboard</b><small>Monitoring<br>Visualization</small></div>
      </div>
    </section>

    <section class="panel">
      <h2>Project Information</h2>
      <p>This dashboard demonstrates a safe, simulated remote IoT monitoring workflow. Sensor values are generated in software and displayed through a browser dashboard. No physical hardware experiment is required.</p>
      <div class="limits">
        <b>Limitations:</b> Data is simulated, there is no real sensor/network connection, and readings are intended only for demonstration and learning.
      </div>
    </section>
  </main>

  <footer>SWYNEX Final IoT Mini-Project • Educational Demonstration</footer>
  <script src="script.js"></script>
</body>
</html>
""",

"style.css": """*{box-sizing:border-box}body{margin:0;font-family:Arial,Helvetica,sans-serif;background:#f4f7fb;color:#182033}header{background:#101936;color:white;padding:28px 7%;display:flex;justify-content:space-between;align-items:center}header h1{margin:0 0 6px;font-size:30px}header p{margin:0;opacity:.8}.status{background:#1d2b52;padding:10px 16px;border-radius:22px}.status i{display:inline-block;width:9px;height:9px;background:#31d07c;border-radius:50%;margin-right:7px}main{max-width:1150px;margin:30px auto;padding:0 20px}.cards{display:grid;grid-template-columns:repeat(4,1fr);gap:18px}.card,.panel{background:white;border-radius:16px;padding:22px;box-shadow:0 5px 20px #17213a12}.card h3{margin:0 0 12px;color:#657089}.card strong{font-size:30px}.card p{color:#7b8497;margin-bottom:0}.panel{margin-top:22px}.panel h2{margin-top:0}.panel canvas{max-height:360px}.architecture{display:flex;align-items:center;justify-content:space-between;gap:12px;flex-wrap:wrap}.architecture div{flex:1;min-width:150px;text-align:center;padding:22px 12px;border:2px solid #dbe2ef;border-radius:14px;background:#f8faff;font-size:28px}.architecture b,.architecture small{display:block}.architecture b{font-size:16px;margin-top:8px}.architecture small{font-size:12px;color:#68748c;line-height:1.5}.architecture span{font-size:28px}.limits{margin-top:15px;padding:15px;background:#f5f7fb;border-left:4px solid #5267a8}footer{text-align:center;padding:28px;color:#778198}@media(max-width:800px){.cards{grid-template-columns:repeat(2,1fr)}.architecture span{display:none}}@media(max-width:500px){.cards{grid-template-columns:1fr}header{display:block}.status{display:inline-block;margin-top:15px}}
""",

"script.js": """const labels=[], temps=[], hums=[];
const ctx=document.getElementById('sensorChart').getContext('2d');
const chart=new Chart(ctx,{type:'line',data:{labels,datasets:[
 {label:'Temperature (°C)',data:temps,tension:.35,borderWidth:2},
 {label:'Humidity (%)',data:hums,tension:.35,borderWidth:2}
]},options:{responsive:true,interaction:{mode:'index',intersect:false},scales:{y:{beginAtZero:false}}}});

let count=0;
function updateSensor(){
  const now=new Date();
  const time=now.toLocaleTimeString();
  const temp=(24+Math.random()*7).toFixed(1);
  const humidity=(48+Math.random()*22).toFixed(1);
  labels.push(time); temps.push(temp); hums.push(humidity);
  if(labels.length>12){labels.shift();temps.shift();hums.shift();}
  chart.update();
  document.getElementById('temp').textContent=temp+' °C';
  document.getElementById('humidity').textContent=humidity+' %';
  document.getElementById('time').textContent='Updated '+time;
  document.getElementById('count').textContent=++count;
}
updateSensor();
setInterval(updateSensor,2000);
""",

"simulate_iot.py": """import csv, random
from datetime import datetime, timedelta

OUTPUT = "sensor_data.csv"
rows = []
start = datetime.now() - timedelta(minutes=29)

for i in range(30):
    timestamp = start + timedelta(minutes=i)
    temperature = round(random.uniform(24, 31), 2)
    humidity = round(random.uniform(48, 70), 2)
    status = "ONLINE"
    rows.append([timestamp.strftime("%Y-%m-%d %H:%M:%S"), temperature, humidity, status])

with open(OUTPUT, "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["timestamp", "temperature_c", "humidity_percent", "device_status"])
    writer.writerows(rows)

print(f"Generated {len(rows)} simulated IoT readings in {OUTPUT}")
""",

"analyze_data.py": """import csv
from statistics import mean

with open("sensor_data.csv", newline="", encoding="utf-8") as f:
    rows = list(csv.DictReader(f))

temps = [float(r["temperature_c"]) for r in rows]
humidity = [float(r["humidity_percent"]) for r in rows]

print("Remote IoT Monitoring - Data Summary")
print("-------------------------------------")
print("Readings:", len(rows))
print(f"Average temperature: {mean(temps):.2f} °C")
print(f"Minimum temperature: {min(temps):.2f} °C")
print(f"Maximum temperature: {max(temps):.2f} °C")
print(f"Average humidity: {mean(humidity):.2f} %")
print(f"Minimum humidity: {min(humidity):.2f} %")
print(f"Maximum humidity: {max(humidity):.2f} %")
""",

"sensor_data.csv": """timestamp,temperature_c,humidity_percent,device_status
2026-10-06 18:30:00,26.4,55.2,ONLINE
2026-10-06 18:31:00,27.1,57.8,ONLINE
2026-10-06 18:32:00,25.8,53.6,ONLINE
2026-10-06 18:33:00,28.2,61.4,ONLINE
2026-10-06 18:34:00,29.0,64.1,ONLINE
2026-10-06 18:35:00,27.6,59.3,ONLINE
2026-10-06 18:36:00,26.9,56.7,ONLINE
2026-10-06 18:37:00,30.1,67.2,ONLINE
2026-10-06 18:38:00,28.5,62.5,ONLINE
2026-10-06 18:39:00,25.9,52.8,ONLINE
""",

"architecture.svg": """<svg xmlns="http://www.w3.org/2000/svg" width="1200" height="360" viewBox="0 0 1200 360">
<rect width="1200" height="360" fill="#f4f7fb"/>
<text x="600" y="48" text-anchor="middle" font-family="Arial" font-size="28" font-weight="bold" fill="#182033">Remote IoT Monitoring Architecture</text>
<g font-family="Arial" text-anchor="middle">
<rect x="55" y="115" width="210" height="130" rx="18" fill="white" stroke="#5a6fb0" stroke-width="3"/>
<text x="160" y="155" font-size="25">SENSORS</text><text x="160" y="188" font-size="16">Temperature</text><text x="160" y="213" font-size="16">Humidity</text>
<rect x="335" y="115" width="210" height="130" rx="18" fill="white" stroke="#5a6fb0" stroke-width="3"/>
<text x="440" y="155" font-size="25">GATEWAY</text><text x="440" y="188" font-size="16">Data Transfer</text><text x="440" y="213" font-size="16">Simulation</text>
<rect x="615" y="115" width="210" height="130" rx="18" fill="white" stroke="#5a6fb0" stroke-width="3"/>
<text x="720" y="155" font-size="25">CLOUD</text><text x="720" y="188" font-size="16">Processing</text><text x="720" y="213" font-size="16">Storage</text>
<rect x="895" y="115" width="250" height="130" rx="18" fill="white" stroke="#5a6fb0" stroke-width="3"/>
<text x="1020" y="155" font-size="25">DASHBOARD</text><text x="1020" y="188" font-size="16">Visualization</text><text x="1020" y="213" font-size="16">Remote Monitoring</text>
</g>
<g stroke="#5a6fb0" stroke-width="4" fill="none"><path d="M265 180h65"/><path d="M545 180h65"/><path d="M825 180h65"/></g>
<g fill="#5a6fb0"><path d="M325 170l15 10-15 10z"/><path d="M605 170l15 10-15 10z"/><path d="M885 170l15 10-15 10z"/></g>
<text x="600" y="310" text-anchor="middle" font-family="Arial" font-size="15" fill="#68748c">Educational simulation — no physical hardware or unsafe experiment required</text>
</svg>
""",

"README.md": """# SWYNEX Final IoT Mini-Project

## Remote IoT Monitoring System

A safe, simulated IoT monitoring project that demonstrates sensor-data generation, processing, visualization, and remote dashboard monitoring.

### Features
- Simulated temperature and humidity readings
- Browser-based live dashboard
- Interactive sensor trend chart
- Python sensor-data generator
- CSV dataset
- Data analysis script
- IoT architecture diagram
- Limitations and safety notes

### Project Structure
```text
SWYNEX-Final-IoT-Mini-Project/
├── index.html
├── style.css
├── script.js
├── simulate_iot.py
├── analyze_data.py
├── sensor_data.csv
├── architecture.svg
└── README.md# SWYNEX-Final-IoT-Mini-Project
This project is a simulated **Remote IoT Monitoring System** that collects sensor data such as temperature and humidity, processes it, and displays the results through visualizations and a simple dashboard. It demonstrates the basic workflow of an IoT system using Python and simulated data.
