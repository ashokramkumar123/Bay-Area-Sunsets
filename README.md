# Bay-Area-Sunsets
mapboxgl.accessToken = 'pk.eyJ1IjoiYXNob2tyYW1rdW1hcjEyMyIsImEiOiJjbGJyOGZjbTQwMDVnM3Bxem0yZTlrYXhiIn0.KVQ_MC5sbFAc4o3CN4485g'
//Mapbox token
var map = new mapboxgl.Map({
  container: 'map', // container id
  style: 'mapbox://styles/mapbox/streets-v11', 
  center: [-122.411, 37.785], // starting position [lng, lat]
  zoom: 10, // starting zoom
  transformRequest: transformRequest
});

$(document).ready(function () {
      $.ajax({
        type: "GET",
        //YOUR TURN: Replace with csv export link
        url: 'https://docs.google.com/spreadsheets/d/1BlZUy06utGnMwSbWhOxavutrsPNOIvAuyFASbZjYG88/gviz/tq?tqx=out:csv&sheet=Sheet_Mapper',
        dataType: "text",
        success: function (csvData) { makeGeoJSON(csvData); }
      });
      
      //Add the the layer to the map
map.addLayer({
  id: 'csvData',
  type: 'circle',
  source: {
    type: 'geojson',
    data: data
  },
  paint: {
    'circle-radius': 5,
    'circle-color': 'purple'
  }
});

// When a click event occurs on a feature in the csvData layer, open a popup at the
// location of the feature, with description HTML from its properties.
map.on('click', 'csvData', function(e) {
  var coordinates = e.features[0].geometry.coordinates.slice();

  //set popup text
  //You can adjust the values of the popup to match the headers of your CSV.
  // For example: e.features[0].properties.Name is retrieving information from the field Name in the original CSV.
  var description =
    `<h3>` +
    e.features[0].properties.Name +
    `</h3>` +
    `<h4>` +
    `<b>` +
    `Address: ` +
    `</b>` +
    e.features[0].properties.Address +
    `</h4>` +
    `<h4>` +
    `<b>` +
    `Phone: ` +
    `</b>` +
    e.features[0].properties.Phone +
    `</h4>`;

  // Ensure that if the map is zoomed out such that multiple
  // copies of the feature are visible, the popup appears
  // over the copy being pointed to.
  while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
    coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
  }

  //add Popup to map

  new mapboxgl.Popup()
    .setLngLat(coordinates)
    .setHTML(description)
    .addTo(map);
});

// Change the cursor to a pointer when the mouse is over the places layer.
map.on('mouseenter', 'csvData', function() {
  map.getCanvas().style.cursor = 'pointer';
});

// Change it back to a pointer when it leaves.
map.on('mouseleave', 'places', function() {
  map.getCanvas().style.cursor = '';
});

var bbox = turf.bbox(data);
map.fitBounds(bbox, { padding: 50 });
