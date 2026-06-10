<!doctype html>
<html lang="uk">
<head>
  <meta charset="utf-8" />
  <title>MGRS Map Viewer</title>

  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

  <style>
    body { margin: 0; font-family: Arial, sans-serif; }
    #top { padding: 10px; background: #f0f0f0; }
    #mgrsText {
      width: 100%;
      height: 180px;
      box-sizing: border-box;
      font-family: monospace;
      font-size: 14px;
    }
    #info { margin-top: 6px; }
    #map { height: calc(100vh - 245px); }
  </style>
</head>

<body>
  <div id="top">
    <h3>MGRS координати</h3>

    <textarea id="mgrsText" placeholder="Вставте текст з MGRS координатами, наприклад: 36T VS 123000 123000"></textarea>

    <div id="info">
      Знайдено валідних координат: <span id="count">0</span>
    </div>
  </div>

  <div id="map"></div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mgrs@1.0.0/dist/mgrs.min.js"></script>

  <script>
    const map = L.map("map").setView([49, 32], 6);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      maxZoom: 19,
      attribution: "&copy; OpenStreetMap contributors"
    }).addTo(map);

    const markersLayer = L.layerGroup().addTo(map);

    const textArea = document.getElementById("mgrsText");
    const countSpan = document.getElementById("count");

    /*
      Підтримує формати:
      36TVS123000123000
      36T VS 123000 123000
      36T VS123000123000
      36T  VS   123000   123000
      38S MB 12345 67890
    */
    const mgrsRegex =
      /\b\d{1,2}\s*[C-HJ-NP-X]\s*[A-HJ-NP-Z]\s*[A-HJ-NP-Z](?:\s*\d{1,10}){2}\b/gi;

    function updateMap() {
      markersLayer.clearLayers();

      const text = textArea.value;
      const matches = text.match(mgrsRegex) || [];

      let validCount = 0;
      const bounds = [];

      matches.forEach(rawCoord => {
        const normalizedCoord = rawCoord.replace(/\s+/g, "").toUpperCase();

        try {
          const [lon, lat] = mgrs.toPoint(normalizedCoord);

          L.marker([lat, lon])
            .bindPopup(`
              <strong>MGRS:</strong> ${rawCoord}<br>
              <strong>Normalized:</strong> ${normalizedCoord}<br>
              <strong>Lat/Lon:</strong> ${lat.toFixed(6)}, ${lon.toFixed(6)}
            `)
            .addTo(markersLayer);

          bounds.push([lat, lon]);
          validCount++;
        } catch (e) {
          console.warn("Некоректна MGRS координата:", rawCoord);
        }
      });

      countSpan.textContent = validCount;

      if (bounds.length > 0) {
        map.fitBounds(bounds, { padding: [30, 30] });
      }
    }

    textArea.addEventListener("input", updateMap);
  </script>
</body>
</html>