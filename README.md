## 📍 1. MAP TEXNOLOGIYALARI: Leaflet / Mapbox / MapLibre

### Asosiy Savollar

**1. Leaflet va Mapbox/MapLibre orasidagi asosiy farqlar?**

**Leaflet:**
- Oddiy, yengil (39KB)
- Raster tile'lar bilan ishlaydi
- Plugin ekosistemasi katta
- 2D xaritalar uchun ideal
- Canvas/SVG rendering

**Mapbox/MapLibre:**
- WebGL-based rendering
- Vector tile'lar bilan ishlaydi
- 3D, terrain, extrusion qo'llab-quvvatlaydi
- Style specification (JSON format)
- Performance yuqori (GPU acceleration)
- MapLibre - Mapbox'ning open-source versiyasi

**Qachon qaysi birini tanlash:**
- Oddiy 2D xarita kerak → Leaflet
- 3D, ko'p data, murakkab styling → Mapbox/MapLibre

---

**2. Tile layer nima? Raster va vector tile'lar farqi?**

**Tile Layer:**
- Xarita katta rasmni kichik kvadrat bo'laklarga (tile) bo'lib yuklash
- Har bir tile 256x256 yoki 512x512 piksel
- Zoom level bo'yicha: 0 (dunyo), 18+ (ko'cha darajasi)

**Raster Tiles:**
```
- PNG/JPG formatda
- Oldindan render qilingan rasmlar
- Har bir zoom uchun alohida rasmlar
- Katta hajm (MB)
- Style o'zgartirish mumkin emas
- Misol: OpenStreetMap tiles
```

**Vector Tiles:**
```
- Geometrik data (points, lines, polygons)
- PBF (Protocol Buffer) formatda
- Client-side rendering
- Kichik hajm (KB)
- Runtime'da style o'zgartirish mumkin
- Rotatsiya, tilt qo'llab-quvvatlaydi
```

---

**3. Leafletda juda ko'p markerlar qo'shilganda performance qanday pasayadi? Optimallashtiriladi?**

**Muammo:**
- Har bir marker - alohida DOM element
- 1000+ markerlar → DOM bloated → lag
- Har bir marker event listenerlar tutadi

**Yechimlar:**

```javascript
// ❌ Yomon: Har bir marker alohida
markers.forEach(point => {
  L.marker([point.lat, point.lng]).addTo(map);
});

// ✅ Yaxshi 1: Marker Clustering
const markers = L.markerClusterGroup();
points.forEach(point => {
  markers.addLayer(L.marker([point.lat, point.lng]));
});
map.addLayer(markers);

// ✅ Yaxshi 2: Canvas Renderer
const canvas = L.canvas();
points.forEach(point => {
  L.circleMarker([point.lat, point.lng], {
    renderer: canvas,
    radius: 5
  }).addTo(map);
});

// ✅ Yaxshi 3: Virtualization (faqat visible markerlar)
map.on('moveend', () => {
  const bounds = map.getBounds();
  const visiblePoints = points.filter(p => 
    bounds.contains([p.lat, p.lng])
  );
  updateMarkers(visiblePoints);
});

// ✅ Yaxshi 4: GeoJSON layer
L.geoJSON(geojsonData, {
  pointToLayer: (feature, latlng) => {
    return L.circleMarker(latlng);
  }
}).addTo(map);
```

---

**4. Marker cluster qanday ishlaydi va qachon kerak bo'ladi?**

**Ishlash prinsipi:**
```
1. Markerlar koordinatalari bo'yicha guruhlangan
2. Zoom out → yaqin markerlar birlashadi
3. Zoom in → clusterlar ajraladi
4. Har bir clusterda markerlar soni ko'rsatiladi
```

**Implementatsiya:**
```javascript
const markers = L.markerClusterGroup({
  // Cluster radius (px)
  maxClusterRadius: 80,
  
  // Animatsiya
  spiderfyOnMaxZoom: true,
  showCoverageOnHover: false,
  zoomToBoundsOnClick: true,
  
  // Custom icon
  iconCreateFunction: (cluster) => {
    const count = cluster.getChildCount();
    let size = 'small';
    if (count > 100) size = 'large';
    else if (count > 10) size = 'medium';
    
    return L.divIcon({
      html: `<div><span>${count}</span></div>`,
      className: `marker-cluster marker-cluster-${size}`,
      iconSize: L.point(40, 40)
    });
  }
});

// Markerlarni qo'shish
locations.forEach(loc => {
  const marker = L.marker([loc.lat, loc.lng]);
  marker.bindPopup(loc.name);
  markers.addLayer(marker);
});

map.addLayer(markers);
```

**Qachon kerak:**
- 100+ markerlar
- Markerlar yaqin joylashgan
- Mobile devices (performance)
- Data density yuqori

---

**5. Custom icon va popup'larni qanday yaratish va optimallashtirish?**

**Custom Icon:**
```javascript
// Simple icon
const customIcon = L.icon({
  iconUrl: '/markers/custom.png',
  iconSize: [32, 32],
  iconAnchor: [16, 32],
  popupAnchor: [0, -32]
});

// DivIcon (HTML-based, yaxshiroq)
const divIcon = L.divIcon({
  html: `
    <div class="custom-marker">
      <img src="${user.avatar}" />
      <span class="status ${user.status}"></span>
    </div>
  `,
  className: 'custom-marker-wrapper',
  iconSize: [40, 40]
});

// SVG icon (masshtablanadigan)
const svgIcon = L.divIcon({
  html: `
    <svg width="24" height="24">
      <circle cx="12" cy="12" r="10" fill="#FF5733"/>
      <text x="12" y="16" text-anchor="middle" fill="white" 
            font-size="12">${count}</text>
    </svg>
  `,
  className: 'svg-marker'
});
```

**Custom Popup Optimallashtiruvi:**
```javascript
// ❌ Yomon: Har bir marker uchun popup yaratish
markers.forEach(m => {
  m.bindPopup(`<div class="popup">${m.data}</div>`);
});

// ✅ Yaxshi: Lazy popup creation
const popup = L.popup();

markers.forEach(m => {
  m.on('click', (e) => {
    popup
      .setLatLng(e.latlng)
      .setContent(generatePopupContent(m.data))
      .openOn(map);
  });
});

// ✅ Yaxshi: Popup pooling
const popupPool = new Map();

function getPopup(id) {
  if (!popupPool.has(id)) {
    popupPool.set(id, createPopup(id));
  }
  return popupPool.get(id);
}

// ✅ Yaxshi: Virtual scrolling in popup
function createLargePopup(items) {
  return `
    <div class="popup-content" style="max-height: 300px; overflow-y: auto">
      ${items.slice(0, 50).map(item => `
        <div class="item">${item.name}</div>
      `).join('')}
    </div>
  `;
}
```

---

**6. Mapbox style specification qanday ishlaydi?**

Mapbox style - bu JSON format xarita qanday ko'rinishini belgilaydi:

```javascript
{
  "version": 8,
  "name": "Custom Style",
  
  // 1. SOURCES - data manbalari
  "sources": {
    "restaurants": {
      "type": "geojson",
      "data": "/api/restaurants.geojson"
    },
    "mapbox-streets": {
      "type": "vector",
      "url": "mapbox://mapbox.mapbox-streets-v8"
    }
  },
  
  // 2. LAYERS - qanday chizish
  "layers": [
    {
      "id": "restaurant-points",
      "type": "circle",
      "source": "restaurants",
      
      // Filter
      "filter": ["==", ["get", "type"], "fast-food"],
      
      // Paint properties
      "paint": {
        "circle-radius": [
          "interpolate", ["linear"], ["zoom"],
          10, 3,
          15, 8
        ],
        "circle-color": [
          "match", ["get", "rating"],
          5, "#00FF00",
          4, "#FFFF00",
          "#FF0000"
        ],
        "circle-opacity": 0.8
      }
    },
    {
      "id": "restaurant-labels",
      "type": "symbol",
      "source": "restaurants",
      "layout": {
        "text-field": ["get", "name"],
        "text-size": 12,
        "text-offset": [0, 1.5]
      }
    }
  ]
}
```

**Dynamic style update:**
```javascript
// Layerni qo'shish
map.addLayer({
  id: 'traffic',
  type: 'line',
  source: 'traffic-source',
  paint: {
    'line-color': ['get', 'congestion_color'],
    'line-width': 3
  }
});

// Paint propertyni o'zgartirish
map.setPaintProperty('traffic', 'line-opacity', 0.5);

// Filterni o'zgartirish
map.setFilter('restaurants', ['>=', ['get', 'rating'], 4]);
```

---

**7. Layer va source o'rtasida qanday farq bor?**

**Source (Data):**
- Data manbai
- NIMA ko'rsatish
- Types: vector, raster, geojson, image, video

**Layer (Visualization):**
- Data qanday chizilishi
- QANDAY ko'rsatish  
- Types: fill, line, circle, symbol, heatmap, fill-extrusion

```javascript
// 1. Source yaratish
map.addSource('earthquakes', {
  type: 'geojson',
  data: '/earthquakes.geojson'
});

// 2. Bir source - ko'p layerlar
// Layer 1: Kichik earthquakes
map.addLayer({
  id: 'small-earthquakes',
  type: 'circle',
  source: 'earthquakes',
  filter: ['<', ['get', 'magnitude'], 5],
  paint: {
    'circle-radius': 4,
    'circle-color': '#FFA500'
  }
});

// Layer 2: Katta earthquakes
map.addLayer({
  id: 'large-earthquakes',
  type: 'circle',
  source: 'earthquakes',
  filter: ['>=', ['get', 'magnitude'], 5],
  paint: {
    'circle-radius': 10,
    'circle-color': '#FF0000'
  }
});

// Layer 3: Labels
map.addLayer({
  id: 'earthquake-labels',
  type: 'symbol',
  source: 'earthquakes',
  layout: {
    'text-field': ['get', 'magnitude']
  }
});
```

---

Davom ettiraman keyingi qismda? GeoJSON strukturalari va keyingi savollarga o'tamiz.
