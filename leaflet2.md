# 🗺️ Leaflet + React - Advanced Topics

## 6. GEOJSON BILAN ISHLASH

### Basic GeoJSON Layer

```jsx
// components/GeoJSONLayer.jsx
import { GeoJSON } from 'react-leaflet';
import { useState, useEffect } from 'react';

function GeoJSONLayer() {
  const [geoData, setGeoData] = useState(null);
  
  useEffect(() => {
    // GeoJSON data yuklash
    fetch('/data/tashkent-districts.geojson')
      .then(res => res.json())
      .then(data => setGeoData(data))
      .catch(err => console.error('Error loading GeoJSON:', err));
  }, []);
  
  // Style function
  const style = (feature) => {
    return {
      fillColor: getColor(feature.properties.population),
      weight: 2,
      opacity: 1,
      color: 'white',
      dashArray: '3',
      fillOpacity: 0.7
    };
  };
  
  // Color based on population
  const getColor = (population) => {
    return population > 500000 ? '#800026' :
           population > 300000 ? '#BD0026' :
           population > 200000 ? '#E31A1C' :
           population > 100000 ? '#FC4E2A' :
           population > 50000  ? '#FD8D3C' :
           population > 20000  ? '#FEB24C' :
           population > 10000  ? '#FED976' :
                                 '#FFEDA0';
  };
  
  // Feature interaction
  const onEachFeature = (feature, layer) => {
    const { name, population, area } = feature.properties;
    
    // Popup
    layer.bindPopup(`
      <div>
        <h3>${name}</h3>
        <p><strong>Population:</strong> ${population.toLocaleString()}</p>
        <p><strong>Area:</strong> ${area} km²</p>
      </div>
    `);
    
    // Hover effects
    layer.on({
      mouseover: (e) => {
        const layer = e.target;
        layer.setStyle({
          weight: 5,
          color: '#666',
          dashArray: '',
          fillOpacity: 0.9
        });
        layer.bringToFront();
      },
      mouseout: (e) => {
        e.target.setStyle(style(feature));
      },
      click: (e) => {
        console.log('Clicked:', feature.properties.name);
      }
    });
  };
  
  if (!geoData) return null;
  
  return (
    <GeoJSON 
      data={geoData} 
      style={style}
      onEachFeature={onEachFeature}
    />
  );
}

export default GeoJSONLayer;
```

### Dynamic GeoJSON (Real-time Updates)

```jsx
// components/DynamicGeoJSON.jsx
import { useState, useEffect } from 'react';
import { GeoJSON } from 'react-leaflet';

function DynamicGeoJSON() {
  const [geoData, setGeoData] = useState(null);
  const [selectedLayer, setSelectedLayer] = useState(null);
  
  useEffect(() => {
    // Initial load
    loadData();
    
    // Periodic update (har 5 sekundda)
    const interval = setInterval(loadData, 5000);
    
    return () => clearInterval(interval);
  }, []);
  
  const loadData = async () => {
    try {
      const response = await fetch('/api/live-vehicles');
      const data = await response.json();
      
      // GeoJSON formatga o'tkazish
      const geoJSON = {
        type: 'FeatureCollection',
        features: data.map(vehicle => ({
          type: 'Feature',
          geometry: {
            type: 'Point',
            coordinates: [vehicle.lng, vehicle.lat]
          },
          properties: {
            id: vehicle.id,
            name: vehicle.name,
            speed: vehicle.speed,
            status: vehicle.status
          }
        }))
      };
      
      setGeoData(geoJSON);
    } catch (error) {
      console.error('Error loading data:', error);
    }
  };
  
  const pointToLayer = (feature, latlng) => {
    const { speed, status } = feature.properties;
    
    // Status bo'yicha rang
    const color = status === 'active' ? '#4CAF50' : 
                  status === 'idle' ? '#FFC107' : '#F44336';
    
    return L.circleMarker(latlng, {
      radius: 8,
      fillColor: color,
      color: '#fff',
      weight: 2,
      opacity: 1,
      fillOpacity: 0.8
    });
  };
  
  const onEachFeature = (feature, layer) => {
    const { name, speed, status } = feature.properties;
    
    layer.bindPopup(`
      <div>
        <h4>${name}</h4>
        <p>Speed: ${speed} km/h</p>
        <p>Status: ${status}</p>
      </div>
    `);
    
    layer.on('click', () => {
      setSelectedLayer(feature.properties.id);
    });
  };
  
  if (!geoData) return <div>Loading...</div>;
  
  return (
    <GeoJSON 
      key={JSON.stringify(geoData)} // Force re-render on data change
      data={geoData}
      pointToLayer={pointToLayer}
      onEachFeature={onEachFeature}
    />
  );
}

export default DynamicGeoJSON;
```

### GeoJSON with Filters

```jsx
// components/FilterableGeoJSON.jsx
import { useState } from 'react';
import { GeoJSON } from 'react-leaflet';

function FilterableGeoJSON({ data }) {
  const [filter, setFilter] = useState('all');
  
  // Filter function
  const filterFeature = (feature) => {
    if (filter === 'all') return true;
    return feature.properties.type === filter;
  };
  
  const filteredData = {
    ...data,
    features: data.features.filter(filterFeature)
  };
  
  return (
    <>
      {/* Filter controls */}
      <div style={{
        position: 'absolute',
        top: 10,
        left: 10,
        zIndex: 1000,
        background: 'white',
        padding: '10px',
        borderRadius: '5px',
        boxShadow: '0 2px 5px rgba(0,0,0,0.2)'
      }}>
        <h4>Filter by type:</h4>
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('restaurant')}>Restaurants</button>
        <button onClick={() => setFilter('hotel')}>Hotels</button>
        <button onClick={() => setFilter('park')}>Parks</button>
      </div>
      
      <GeoJSON 
        key={filter} // Re-render on filter change
        data={filteredData}
        pointToLayer={(feature, latlng) => {
          return L.circleMarker(latlng, {
            radius: 6,
            fillColor: getColorByType(feature.properties.type),
            color: '#fff',
            weight: 2,
            fillOpacity: 0.8
          });
        }}
      />
    </>
  );
}

function getColorByType(type) {
  switch(type) {
    case 'restaurant': return '#FF5722';
    case 'hotel': return '#2196F3';
    case 'park': return '#4CAF50';
    default: return '#9E9E9E';
  }
}

export default FilterableGeoJSON;
```

---

## 7. MARKER CLUSTERING (react-leaflet-cluster)

### Installation

```bash
npm install react-leaflet-cluster
```

### Basic Clustering

```jsx
// components/MarkerClusterMap.jsx
import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet';
import MarkerClusterGroup from 'react-leaflet-cluster';
import 'leaflet/dist/leaflet.css';

function MarkerClusterMap() {
  // 1000+ markers
  const markers = generateMarkers(1000);
  
  return (
    <MapContainer center={[41.2995, 69.2401]} zoom={10}>
      <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
      
      <MarkerClusterGroup
        chunkedLoading
        maxClusterRadius={60}
        spiderfyOnMaxZoom={true}
        showCoverageOnHover={false}
        zoomToBoundsOnClick={true}
      >
        {markers.map(marker => (
          <Marker key={marker.id} position={marker.position}>
            <Popup>
              <div>
                <h4>{marker.name}</h4>
                <p>{marker.description}</p>
              </div>
            </Popup>
          </Marker>
        ))}
      </MarkerClusterGroup>
    </MapContainer>
  );
}

function generateMarkers(count) {
  const markers = [];
  
  for (let i = 0; i < count; i++) {
    markers.push({
      id: i,
      position: [
        41.2995 + (Math.random() - 0.5) * 0.5,
        69.2401 + (Math.random() - 0.5) * 0.5
      ],
      name: `Location ${i + 1}`,
      description: `Description for location ${i + 1}`
    });
  }
  
  return markers;
}

export default MarkerClusterMap;
```

### Custom Cluster Icons

```jsx
// components/CustomClusterMap.jsx
import MarkerClusterGroup from 'react-leaflet-cluster';
import L from 'leaflet';

function CustomClusterMap() {
  const markers = generateMarkers(500);
  
  // Custom icon creator
  const createClusterCustomIcon = (cluster) => {
    const count = cluster.getChildCount();
    
    let size = 'small';
    if (count > 100) size = 'large';
    else if (count > 10) size = 'medium';
    
    return L.divIcon({
      html: `<div class="cluster-icon cluster-${size}">
        <span>${count}</span>
      </div>`,
      className: 'custom-marker-cluster',
      iconSize: L.point(40, 40, true)
    });
  };
  
  return (
    <MapContainer center={[41.2995, 69.2401]} zoom={10}>
      <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
      
      <MarkerClusterGroup
        iconCreateFunction={createClusterCustomIcon}
        maxClusterRadius={80}
      >
        {markers.map(marker => (
          <Marker key={marker.id} position={marker.position}>
            <Popup>{marker.name}</Popup>
          </Marker>
        ))}
      </MarkerClusterGroup>
    </MapContainer>
  );
}

export default CustomClusterMap;
```

```css
/* styles/cluster.css */
.custom-marker-cluster {
  background: transparent;
}

.cluster-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  color: white;
  font-weight: bold;
  box-shadow: 0 2px 5px rgba(0,0,0,0.3);
}

.cluster-small {
  background: #4CAF50;
  width: 30px;
  height: 30px;
  font-size: 12px;
}

.cluster-medium {
  background: #FF9800;
  width: 40px;
  height: 40px;
  font-size: 14px;
}

.cluster-large {
  background: #F44336;
  width: 50px;
  height: 50px;
  font-size: 16px;
}
```

### Cluster Events

```jsx
// components/ClusterWithEvents.jsx
import { useRef, useEffect } from 'react';
import MarkerClusterGroup from 'react-leaflet-cluster';

function ClusterWithEvents() {
  const clusterRef = useRef();
  
  useEffect(() => {
    if (clusterRef.current) {
      const cluster = clusterRef.current;
      
      // Cluster click event
      cluster.on('clusterclick', (e) => {
        console.log('Cluster clicked:', e.layer.getAllChildMarkers().length);
      });
      
      // Animation end
      cluster.on('animationend', () => {
        console.log('Cluster animation ended');
      });
    }
  }, []);
  
  return (
    <MarkerClusterGroup ref={clusterRef}>
      {/* markers */}
    </MarkerClusterGroup>
  );
}

export default ClusterWithEvents;
```

---

## 8. LEAFLET PLUGINS BILAN ISHLASH

### Leaflet.heat (Heatmap)

```bash
npm install leaflet.heat
```

```jsx
// components/HeatmapLayer.jsx
import { useEffect } from 'react';
import { useMap } from 'react-leaflet';
import L from 'leaflet';
import 'leaflet.heat';

function HeatmapLayer({ points }) {
  const map = useMap();
  
  useEffect(() => {
    if (!points || points.length === 0) return;
    
    // Convert to heatmap format: [[lat, lng, intensity], ...]
    const heatData = points.map(point => [
      point.lat,
      point.lng,
      point.intensity || 1
    ]);
    
    // Create heatmap layer
    const heatLayer = L.heatLayer(heatData, {
      radius: 25,
      blur: 15,
      maxZoom: 17,
      max: 1.0,
      gradient: {
        0.0: 'blue',
        0.5: 'lime',
        1.0: 'red'
      }
    }).addTo(map);
    
    return () => {
      map.removeLayer(heatLayer);
    };
  }, [map, points]);
  
  return null;
}

export default HeatmapLayer;
```

```jsx
// Usage
function App() {
  const heatmapData = [
    { lat: 41.2995, lng: 69.2401, intensity: 0.8 },
    { lat: 41.3111, lng: 69.2797, intensity: 1.0 },
    { lat: 41.3264, lng: 69.2877, intensity: 0.5 },
    // ... more points
  ];
  
  return (
    <MapContainer center={[41.2995, 69.2401]} zoom={12}>
      <TileLayer url="..." />
      <HeatmapLayer points={heatmapData} />
    </MapContainer>
  );
}
```

### Leaflet.draw (Drawing Tools)

```bash
npm install leaflet-draw
npm install @types/leaflet-draw
```

```jsx
// components/DrawingTools.jsx
import { useEffect, useState } from 'react';
import { FeatureGroup } from 'react-leaflet';
import { EditControl } from 'react-leaflet-draw';
import 'leaflet-draw/dist/leaflet.draw.css';

function DrawingTools() {
  const [shapes, setShapes] = useState([]);
  
  const onCreated = (e) => {
    const { layerType, layer } = e;
    
    const newShape = {
      id: Date.now(),
      type: layerType,
      data: layer.toGeoJSON(),
      latlngs: layer.getLatLngs ? layer.getLatLngs() : layer.getLatLng()
    };
    
    setShapes([...shapes, newShape]);
    console.log('Shape created:', newShape);
  };
  
  const onEdited = (e) => {
    const { layers } = e;
    
    layers.eachLayer((layer) => {
      console.log('Shape edited:', layer.toGeoJSON());
    });
  };
  
  const onDeleted = (e) => {
    const { layers } = e;
    
    layers.eachLayer((layer) => {
      console.log('Shape deleted');
    });
  };
  
  return (
    <FeatureGroup>
      <EditControl
        position="topright"
        onCreated={onCreated}
        onEdited={onEdited}
        onDeleted={onDeleted}
        draw={{
          rectangle: true,
          circle: true,
          circlemarker: false,
          marker: true,
          polyline: true,
          polygon: {
            allowIntersection: false,
            showArea: true,
            drawError: {
              color: '#e1e100',
              message: '<strong>Error:</strong> shape edges cannot cross!'
            },
            shapeOptions: {
              color: '#97009c'
            }
          }
        }}
      />
    </FeatureGroup>
  );
}

export default DrawingTools;
```

### Leaflet Routing Machine

```bash
npm install leaflet-routing-machine
```

```jsx
// components/RoutingControl.jsx
import { useEffect } from 'react';
import { useMap } from 'react-leaflet';
import L from 'leaflet';
import 'leaflet-routing-machine';
import 'leaflet-routing-machine/dist/leaflet-routing-machine.css';

function RoutingControl({ waypoints }) {
  const map = useMap();
  
  useEffect(() => {
    if (!waypoints || waypoints.length < 2) return;
    
    const routingControl = L.Routing.control({
      waypoints: waypoints.map(wp => L.latLng(wp.lat, wp.lng)),
      routeWhileDragging: true,
      showAlternatives: true,
      altLineOptions: {
        styles: [
          { color: 'black', opacity: 0.15, weight: 9 },
          { color: 'white', opacity: 0.8, weight: 6 },
          { color: 'blue', opacity: 0.5, weight: 2 }
        ]
      },
      createMarker: function(i, waypoint, n) {
        const marker = L.marker(waypoint.latLng, {
          draggable: true,
          icon: L.icon({
            iconUrl: i === 0 ? '/icons/start.png' : 
                     i === n - 1 ? '/icons/end.png' : 
                     '/icons/waypoint.png',
            iconSize: [32, 32]
          })
        });
        
        return marker;
      }
    }).addTo(map);
    
    routingControl.on('routesfound', function(e) {
      const routes = e.routes;
      const summary = routes[0].summary;
      
      console.log('Route found:');
      console.log('Distance:', (summary.totalDistance / 1000).toFixed(2), 'km');
      console.log('Time:', Math.round(summary.totalTime / 60), 'minutes');
    });
    
    return () => {
      map.removeControl(routingControl);
    };
  }, [map, waypoints]);
  
  return null;
}

export default RoutingControl;
```

```jsx
// Usage
function App() {
  const [waypoints, setWaypoints] = useState([
    { lat: 41.2995, lng: 69.2401 }, // Start
    { lat: 41.3111, lng: 69.2797 }, // Waypoint
    { lat: 41.3264, lng: 69.2877 }  // End
  ]);
  
  return (
    <MapContainer center={[41.2995, 69.2401]} zoom={13}>
      <TileLayer url="..." />
      <RoutingControl waypoints={waypoints} />
    </MapContainer>
  );
}
```

Davom ettiraman? Search, Geocoding, Layer Control va Performance Optimization'ga o'tamiz.
