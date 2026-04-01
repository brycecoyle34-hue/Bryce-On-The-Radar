# Bryce-On-The-Radar
Storm Chasing 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Bryce on the Radar</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0b0f1a;
  color: white;
}

header {
  background: linear-gradient(90deg, #111a2b, #1f2b45);
  padding: 15px;
  text-align: center;
  font-size: 26px;
  font-weight: bold;
}

nav {
  background: #0d1422;
  padding: 10px;
  text-align: center;
}

nav a {
  color: white;
  margin: 0 12px;
  text-decoration: none;
  font-weight: bold;
}

.section {
  padding: 15px;
}

#map {
  height: 500px;
  border-radius: 12px;
}

.alert-box {
  background: #1a2336;
  padding: 12px;
  margin-top: 10px;
  border-left: 5px solid red;
  border-radius: 8px;
}

iframe {
  width: 100%;
  height: 350px;
  border-radius: 10px;
  border: none;
}

button {
  background: red;
  border: none;
  padding: 10px 15px;
  color: white;
  border-radius: 6px;
  cursor: pointer;
}
</style>
</head>

<body>

<header>🌪️ Bryce on the Radar</header>

<nav>
  <a href="#live">Live</a>
  <a href="#radar">Radar</a>
  <a href="#alerts">Alerts</a>
</nav>

<div class="section" id="live">
  <h2>🔴 Live Stream</h2>
  <iframe src="https://www.youtube.com/embed/live_stream?channel=YOUR_CHANNEL_ID"></iframe>
</div>

<div class="section" id="radar">
  <h2>📡 Live Radar & Warnings</h2>
  <div id="map"></div>
</div>

<div class="section" id="alerts">
  <h2>⚠️ Live Alerts</h2>
  <button onclick="toggleSound()">Toggle Alert Sound</button>
  <div id="alerts-container">Loading alerts...</div>
</div>

<audio id="alertSound" src="https://actions.google.com/sounds/v1/alarms/siren.ogg"></audio>

<script>
// Initialize Map
var map = L.map('map').setView([41.8781, -87.6298], 6);

// Base map
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// Radar layer
L.tileLayer(
  "https://tilecache.rainviewer.com/v2/radar/latest/256/{z}/{x}/{y}/2/1_1.png",
  { opacity: 0.6 }
).addTo(map);

// Layers
let warningLayer = L.layerGroup().addTo(map);
let reportLayer = L.layerGroup().addTo(map);

// Alert sound toggle
let soundEnabled = true;
function toggleSound() {
  soundEnabled = !soundEnabled;
}

// Fetch Alerts + Plot
async function loadAlerts() {
  const res = await fetch("https://api.weather.gov/alerts/active");
  const data = await res.json();

  let container = document.getElementById("alerts-container");
  container.innerHTML = "";
  warningLayer.clearLayers();

  data.features.slice(0, 15).forEach(alert => {
    let p = alert.properties;

    // Alert box
    let div = document.createElement("div");
    div.className = "alert-box";
    div.innerHTML = `<strong>${p.event}</strong><br>${p.areaDesc}`;
    container.appendChild(div);

    // Play sound for tornado warnings
    if (p.event.includes("Tornado") && soundEnabled) {
      document.getElementById("alertSound").play();
    }

    // Draw polygon if exists
    if (alert.geometry) {
      let coords = alert.geometry.coordinates;
      let latlngs = coords[0].map(c => [c[1], c[0]]);

      L.polygon(latlngs, {
        color: p.event.includes("Tornado") ? "red" : "yellow"
      }).addTo(warningLayer);
    }
  });
}

// Simulated storm reports (example markers)
function loadReports() {
  reportLayer.clearLayers();

  let reports = [
    { lat: 41.5, lon: -88.0, type: "Tornado" },
    { lat: 40.8, lon: -87.5, type: "Hail" },
    { lat: 42.0, lon: -89.0, type: "Wind" }
  ];

  reports.forEach(r => {
    let color = r.type === "Tornado" ? "red" :
                r.type === "Hail" ? "blue" : "green";

    L.circleMarker([r.lat, r.lon], {
      radius: 8,
      color: color
    })
    .bindPopup(r.type + " Report")
    .addTo(reportLayer);
  });
}

// Initial load
loadAlerts();
loadReports();

// Auto refresh
setInterval(loadAlerts, 60000);
setInterval(loadReports, 120000);
</script>

</body>
</html>