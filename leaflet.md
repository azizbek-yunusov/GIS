# 🗺️ Leaflet + React - To'liq Qo'llanma

## 1. SETUP VA ASOSIY KONFIGURATSIYA

### Installation

```bash
npm install leaflet react-leaflet
npm install -D @types/leaflet  # TypeScript uchun
```

### Basic Setup

```jsx
// App.jsx
import React from 'react';
import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';
import './App.css';

// Leaflet icon fix (React bilan ishlashda default iconlar ko'rinmaydi)
import L from 'leaflet';
import icon from 'leaflet/dist/images/marker-icon.png';
import iconShadow from 'leaflet/dist/images/marker-shadow.png';

let DefaultIcon = L.icon({
  iconUrl: icon,
  shadowUrl: iconShadow,
  iconSize: [25, 41],
  iconAnchor: [12, 41]
});

L.Marker.prototype.options.icon = DefaultIcon;

function App() {
  const position = [41.2995, 69.2401]; // Tashkent

  return (
    <MapContainer
      center={position}
      zoom={13}
      style={{ height: '100vh', width: '100%' }}
    >
      <TileLayer
        attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      />
      
      <Marker position={position}>
        <Popup>
          Tashkent, Uzbekistan
        </Popup>
      </Marker>
    </MapContainer>
  );
}

export default App;
```

```css
/* App.css */
.leaflet-container {
  height: 100%;
  width: 100%;
}
```

---

## 2. KOMPONENTLAR VA HOOKS

### Map Access Hook

```jsx
// hooks/useMap.js
import { useMap } from 'react-leaflet';
import { useEffect } from 'react';

// Map instance'ga kirish
function MapController() {
  const map = useMap();
  
  useEffect(() => {
    console.log('Map center:', map.getCenter());
    console.log('Map zoom:', map.getZoom());
    
    // Map events
    map.on('moveend', () => {
      console.log('Map moved to:', map.getCenter());
    });
    
    return () => {
      map.off('moveend');
    };
  }, [map]);
  
  return null;
}

// Usage
<MapContainer center={position} zoom={13}>
  <TileLayer url="..." />
  <MapController />
</MapContainer>
```

### Change Map Center Dynamically

```jsx
// components/MapCenterController.jsx
import { useEffect } from 'react';
import { useMap } from 'react-leaflet';

function MapCenterController({ center, zoom }) {
  const map = useMap();
  
  useEffect(() => {
    if (center) {
      map.setView(center, zoom || map.getZoom(), {
        animate: true,
        duration: 1
      });
    }
  }, [center, zoom, map]);
  
  return null;
}

export default MapCenterController;
```

```jsx
// Usage
function App() {
  const [center, setCenter] = useState([41.2995, 69.2401]);
  const [zoom, setZoom] = useState(13);
  
  return (
    <>
      <button onClick={() => setCenter([41.3111, 69.2797])}>
        Go to Chorsu
      </button>
      
      <MapContainer center={center} zoom={zoom}>
        <TileLayer url="..." />
        <MapCenterController center={center} zoom={zoom} />
      </MapContainer>
    </>
  );
}
```

---

## 3. MARKERLAR VA POPUPS

### Dynamic Markers

```jsx
// components/MapWithMarkers.jsx
import { useState } from 'react';
import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet';
import L from 'leaflet';

function MapWithMarkers() {
  const [markers, setMarkers] = useState([
    { id: 1, position: [41.2995, 69.2401], title: 'Amir Temur Square' },
    { id: 2, position: [41.3111, 69.2797], title: 'Chorsu Bazaar' },
    { id: 3, position: [41.3264, 69.2877], title: 'Tashkent Tower' }
  ]);
  
  return (
    <MapContainer 
      center={[41.2995, 69.2401]} 
      zoom={12}
      style={{ height: '100vh' }}
    >
      <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
      
      {markers.map(marker => (
        <Marker 
          key={marker.id} 
          position={marker.position}
        >
          <Popup>
            <div>
              <h3>{marker.title}</h3>
              <button onClick={() => handleMarkerClick(marker.id)}>
                Details
              </button>
            </div>
          </Popup>
        </Marker>
      ))}
    </MapContainer>
  );
  
  function handleMarkerClick(id) {
    console.log('Marker clicked:', id);
  }
}

export default MapWithMarkers;
```

### Custom Icons

```jsx
// utils/mapIcons.js
import L from 'leaflet';

export const restaurantIcon = L.icon({
  iconUrl: '/icons/restaurant.png',
  iconSize: [32, 32],
  iconAnchor: [16, 32],
  popupAnchor: [0, -32]
});

export const hotelIcon = L.icon({
  iconUrl: '/icons/hotel.png',
  iconSize: [32, 32],
  iconAnchor: [16, 32],
  popupAnchor: [0, -32]
});

export const parkIcon = L.icon({
  iconUrl: '/icons/park.png',
  iconSize: [32, 32],
  iconAnchor: [16, 32],
  popupAnchor: [0, -32]
});
```

```jsx
// components/CustomMarkers.jsx
import { Marker, Popup } from 'react-leaflet';
import { restaurantIcon, hotelIcon, parkIcon } from '../utils/mapIcons';

const places = [
  { id: 1, type: 'restaurant', position: [41.2995, 69.2401], name: 'Caravan' },
  { id: 2, type: 'hotel', position: [41.3111, 69.2797], name: 'Hyatt Regency' },
  { id: 3, type: 'park', position: [41.3264, 69.2877], name: 'Alisher Navoi Park' }
];

function CustomMarkers() {
  const getIcon = (type) => {
    switch(type) {
      case 'restaurant': return restaurantIcon;
      case 'hotel': return hotelIcon;
      case 'park': return parkIcon;
      default: return restaurantIcon;
    }
  };
  
  return (
    <>
      {places.map(place => (
        <Marker 
          key={place.id}
          position={place.position}
          icon={getIcon(place.type)}
        >
          <Popup>
            <strong>{place.name}</strong>
          </Popup>
        </Marker>
      ))}
    </>
  );
}

export default CustomMarkers;
```

### DivIcon (HTML-based Custom Icons)

```jsx
// components/CustomDivIcon.jsx
import { Marker, Popup } from 'react-leaflet';
import L from 'leaflet';
import { renderToStaticMarkup } from 'react-dom/server';

function CustomDivIconMarker({ position, count, status }) {
  const iconMarkup = renderToStaticMarkup(
    <div style={{
      background: status === 'active' ? '#4CAF50' : '#FF5252',
      borderRadius: '50%',
      width: '40px',
      height: '40px',
      display: 'flex',
      alignItems: 'center',
      justifyContent: 'center',
      color: 'white',
      fontWeight: 'bold',
      border: '3px solid white',
      boxShadow: '0 2px 5px rgba(0,0,0,0.3)'
    }}>
      {count}
    </div>
  );
  
  const customIcon = L.divIcon({
    html: iconMarkup,
    className: 'custom-div-icon',
    iconSize: [40, 40],
    iconAnchor: [20, 20]
  });
  
  return (
    <Marker position={position} icon={customIcon}>
      <Popup>
        Status: {status}<br/>
        Count: {count}
      </Popup>
    </Marker>
  );
}

export default CustomDivIconMarker;
```

---

## 4. CLICK EVENTS VA INTERAKTIVLIK

### Map Click Event

```jsx
// components/MapClickHandler.jsx
import { useMapEvents } from 'react-leaflet';
import { useState } from 'react';

function MapClickHandler({ onMapClick }) {
  const [clickedPosition, setClickedPosition] = useState(null);
  
  useMapEvents({
    click: (e) => {
      const { lat, lng } = e.latlng;
      console.log('Map clicked:', lat, lng);
      
      setClickedPosition([lat, lng]);
      onMapClick && onMapClick({ lat, lng });
    },
    
    moveend: (e) => {
      const map = e.target;
      console.log('Map bounds:', map.getBounds());
    },
    
    zoomend: (e) => {
      const map = e.target;
      console.log('Zoom level:', map.getZoom());
    }
  });
  
  return clickedPosition ? (
    <Marker position={clickedPosition}>
      <Popup>You clicked here!</Popup>
    </Marker>
  ) : null;
}

export default MapClickHandler;
```

```jsx
// Usage
function App() {
  const handleMapClick = (coords) => {
    console.log('Coordinates:', coords);
  };
  
  return (
    <MapContainer center={[41.2995, 69.2401]} zoom={13}>
      <TileLayer url="..." />
      <MapClickHandler onMapClick={handleMapClick} />
    </MapContainer>
  );
}
```

### Add Marker on Click

```jsx
// components/AddMarkerOnClick.jsx
import { useState } from 'react';
import { MapContainer, TileLayer, Marker, Popup, useMapEvents } from 'react-leaflet';

function AddMarkerOnClick() {
  const [markers, setMarkers] = useState([]);
  
  function LocationMarker() {
    useMapEvents({
      click(e) {
        const newMarker = {
          id: Date.now(),
          position: [e.latlng.lat, e.latlng.lng],
          name: `Marker ${markers.length + 1}`
        };
        
        setMarkers([...markers, newMarker]);
      }
    });
    
    return null;
  }
  
  return (
    <MapContainer center={[41.2995, 69.2401]} zoom={13}>
      <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
      <LocationMarker />
      
      {markers.map(marker => (
        <Marker key={marker.id} position={marker.position}>
          <Popup>
            <div>
              <h4>{marker.name}</h4>
              <p>
                Lat: {marker.position[0].toFixed(4)}<br/>
                Lng: {marker.position[1].toFixed(4)}
              </p>
              <button onClick={() => deleteMarker(marker.id)}>
                Delete
              </button>
            </div>
          </Popup>
        </Marker>
      ))}
    </MapContainer>
  );
  
  function deleteMarker(id) {
    setMarkers(markers.filter(m => m.id !== id));
  }
}

export default AddMarkerOnClick;
```

---

## 5. POLYLINES VA POLYGONS

### Polyline (Yo'l chizish)

```jsx
// components/RouteMap.jsx
import { Polyline, Marker } from 'react-leaflet';

function RouteMap() {
  const route = [
    [41.2995, 69.2401], // Start: Amir Temur Square
    [41.3045, 69.2505],
    [41.3111, 69.2797], // Chorsu Bazaar
    [41.3200, 69.2850],
    [41.3264, 69.2877]  // End: Tashkent Tower
  ];
  
  const polylineOptions = {
    color: '#2196F3',
    weight: 5,
    opacity: 0.7,
    smoothFactor: 1
  };
  
  return (
    <>
      {/* Route line */}
      <Polyline positions={route} pathOptions={polylineOptions} />
      
      {/* Start marker */}
      <Marker position={route[0]}>
        <Popup>Start: Amir Temur Square</Popup>
      </Marker>
      
      {/* End marker */}
      <Marker position={route[route.length - 1]}>
        <Popup>End: Tashkent Tower</Popup>
      </Marker>
    </>
  );
}

export default RouteMap;
```

### Animated Polyline

```jsx
// components/AnimatedRoute.jsx
import { useEffect, useState } from 'react';
import { Polyline, Marker } from 'react-leaflet';
import L from 'leaflet';

function AnimatedRoute({ fullRoute }) {
  const [displayedRoute, setDisplayedRoute] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  
  useEffect(() => {
    if (currentIndex < fullRoute.length) {
      const timer = setTimeout(() => {
        setDisplayedRoute([...displayedRoute, fullRoute[currentIndex]]);
        setCurrentIndex(currentIndex + 1);
      }, 100); // 100ms interval
      
      return () => clearTimeout(timer);
    }
  }, [currentIndex, fullRoute, displayedRoute]);
  
  const carIcon = L.icon({
    iconUrl: '/icons/car.png',
    iconSize: [32, 32],
    iconAnchor: [16, 16]
  });
  
  return (
    <>
      <Polyline 
        positions={displayedRoute} 
        pathOptions={{ color: '#4CAF50', weight: 4 }}
      />
      
      {displayedRoute.length > 0 && (
        <Marker 
          position={displayedRoute[displayedRoute.length - 1]}
          icon={carIcon}
        />
      )}
    </>
  );
}

export default AnimatedRoute;
```

### Polygon (Hudud chizish)

```jsx
// components/DistrictPolygon.jsx
import { Polygon, Popup } from 'react-leaflet';

function DistrictPolygon() {
  // Yunusabad tumani (misol)
  const yunusabadBoundary = [
    [41.3500, 69.2500],
    [41.3500, 69.3000],
    [41.3200, 69.3000],
    [41.3200, 69.2500],
    [41.3500, 69.2500]
  ];
  
  const polygonOptions = {
    fillColor: '#FF6B6B',
    fillOpacity: 0.3,
    color: '#C92A2A',
    weight: 2
  };
  
  return (
    <Polygon positions={yunusabadBoundary} pathOptions={polygonOptions}>
      <Popup>
        <div>
          <h3>Yunusabad District</h3>
          <p>Population: ~300,000</p>
          <p>Area: 25 km²</p>
        </div>
      </Popup>
    </Polygon>
  );
}

export default DistrictPolygon;
```

### Interactive Polygon (Edit mode)

```jsx
// components/EditablePolygon.jsx
import { useState } from 'react';
import { Polygon, useMapEvents } from 'react-leaflet';

function EditablePolygon() {
  const [points, setPoints] = useState([]);
  const [isDrawing, setIsDrawing] = useState(false);
  
  useMapEvents({
    click(e) {
      if (isDrawing) {
        setPoints([...points, [e.latlng.lat, e.latlng.lng]]);
      }
    }
  });
  
  const clearPolygon = () => {
    setPoints([]);
    setIsDrawing(false);
  };
  
  const finishDrawing = () => {
    setIsDrawing(false);
  };
  
  return (
    <>
      <div style={{ position: 'absolute', top: 10, right: 10, zIndex: 1000 }}>
        <button onClick={() => setIsDrawing(!isDrawing)}>
          {isDrawing ? 'Stop Drawing' : 'Start Drawing'}
        </button>
        <button onClick={clearPolygon}>Clear</button>
        {points.length > 2 && (
          <button onClick={finishDrawing}>Finish</button>
        )}
      </div>
      
      {points.length > 2 && (
        <Polygon 
          positions={points}
          pathOptions={{ color: '#9C27B0', fillOpacity: 0.4 }}
        />
      )}
    </>
  );
}

export default EditablePolygon;
```

Davom ettiraman? GeoJSON, Marker Clustering, va boshqa murakkab mavzularga o'tamiz.
