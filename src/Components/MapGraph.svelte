<script>
    import { onMount } from 'svelte';
    import mapboxgl from 'mapbox-gl'; // Import Mapbox GL library
    import * as d3 from 'd3'; // Import D3 library
    
    //import geojson from '../../Simple.geojson';

    let map; // Variable to hold the map object
    let Rainfall = true; // Initial visibility for Rain
    let WindSpeed = true; // Initial visibility for WindSpeed
    let DamageUSD = true; // Initial visibility for DamageUSD
    let Fatalities = true; // Initial visibility for Fatalities
    let data = [];
    let fifthrow = '';
    
    let circleSize = 3; // Initial circle size
    let circleOpacity = 0.6; // Initial circle opacity

    onMount(async () => {
    // Load the CSV data using D3
    const res = await fetch('assets/hurricane_data.csv');
    const csv = await res.text();
    data = d3.csvParse(csv, d3.autoType);    

    fifthrow = data[4]['Affected Areas']; 
    console.log('Affected Areas in the 5th row:', fifthrow);

    let statesToColor = fifthrow.split(',\xa0').map(state => state.trim());
    console.log(statesToColor);

    let newEngland = ['Maine', 'Vermont', 'New Hampshire', 'Massachusetts', 'Connecticut'];
    statesToColor = [...statesToColor, ...newEngland];

    const color = 'red';
    const stateColorArray = statesToColor.map(state => [state, color]);
    console.log(stateColorArray);

    mapboxgl.accessToken = "pk.eyJ1Ijoibmdob3NoMjQiLCJhIjoiY2x3NXcwMW9wMW9rODJxbnZ3YnQ0M2YxbSJ9.4pqWQb2ek3y_3Ahya28EHA"; 

    map = new mapboxgl.Map({
      container: 'map', // HTML element ID where the map will be rendered
      style: 'mapbox://styles/mapbox/light-v10', // Map style
      center: [-98.5795, 39.8283], // Center coordinates for continental US
      zoom: 3 // Initial zoom level
    });

     // Add map controls (optional)
    //map.addControl(new mapboxgl.NavigationControl());

    map.on('load', async () => {

      const response = await fetch('assets/Simple.geojson');
      const geodata = await response.json();

      const can_res = await fetch('assets/canada_provinces.geojson');
      const geo_canada = await can_res.json();

      map.addSource('states', {
        type: 'geojson',
        data: geodata
      });

      map.addLayer({
        id: 'states-layer',
        type: 'fill',
        source: 'states',
        paint: {
            'fill-color': {
                property: 'NAME',
                type: "categorical",
                stops: stateColorArray
            },
            'fill-opacity': 0.7
        }
      });

      map.addSource('canada', {
        type: 'geojson',
        data: geo_canada
      });

      map.addLayer({
        id: 'canada-layer',
        type: 'fill',
        source: 'canada',
        paint: {
            'fill-color': {
                property: 'prov_name_en',
                type: "categorical",
                stops: [
                    ['Nova Scotia', 'red'],
                    ['New Brunswick', 'red'],
                    ['Prince Edward Island', 'red'],
                    ['Newfoundland and Labrador', 'red']
                ]
            },
            'fill-opacity': 0.7
        }
      });



      // Define the colors for each type
      const colors = {
        'Rainfall': 'blue',
        'WindSpeed': 'purple',
        'DamageUSD': 'green',
        'Fatalities': 'red',
      };
  });

  map.on('mousemove', 'states-layer', function (e) {
    // Change the cursor style to a pointer when hovering over the layer
    map.getCanvas().style.cursor = 'pointer';

    // Get the properties of the feature under the mouse pointer
    //var featureProperties = e.features[0].properties;

    // Create a tooltip HTML content based on the feature properties
    var tooltipContent = '<h3>' + 'Hurricane Allison' + '</h3>' +
                         '<p>Category: ' + 'Tropical Storm' + '</p>';

    // Display the tooltip at the mouse position
    new mapboxgl.Popup()
        .setLngLat(e.lngLat)
        .setHTML(tooltipContent)
        .addTo(map);
});

// Reset the cursor style and close the tooltip when the mouse leaves the layer
map.on('mouseleave', 'states-layer', function () {
    map.getCanvas().style.cursor = '';
    map.getCanvas().title = '';
});


});

  // Watch for changes in the checkbox values
  $: {
    if (map && map.loaded()) {
      map.setLayoutProperty('circles-Rainfall', 'visibility', Rainfall ? 'visible' : 'none');
      map.setLayoutProperty('circles-WindSpeed', 'visibility', WindSpeed ? 'visible' : 'none');
      map.setLayoutProperty('circles-DamageUSD', 'visibility', DamageUSD ? 'visible' : 'none');
      map.setLayoutProperty('circles-Fatalities', 'visibility', Fatalities ? 'visible' : 'none');
    }
  }

</script>

<main>
    <h2>Effects of the hurricanes</h2>

    <p>Below is a map showing the affected areas of different hurricanes,
        categorized by the different damage effects such as rainfall, windspeed, 
        damage in USD, and fatalities.
    </p>

    <div id="checkbox-container" class="checkbox-grid">
        <!-- Add checkboxes for each effect -->
        <label ><input type="checkbox" bind:checked={Rainfall}> Rainfall</label>
        <label ><input type="checkbox" bind:checked={WindSpeed}> WindSpeed</label>
        <label ><input type="checkbox" bind:checked={DamageUSD}> DamageUSD</label>
        <label ><input type="checkbox" bind:checked={Fatalities}> Fatalities</label>
      </div>
      <!-- Container for the map -->
      <div id="map"></div>
</main>

<style>
    #map {
      height: 550px; /* Adjust height as needed */
      width: 100%; /* Make map fill container width */
      margin-bottom: 20px; /* Add some space below the map */
    }
    h2 {
      text-align: left;
      font-family: "Kode Mono", monospace;
      font-weight: 400;
      font-size: 42px;
      line-height: 1;
      color: black;
      padding: 10px;
    }
    p {
          text-align: left;
      font-family: "Kode Mono", monospace;
      font-weight: 400;
      font-size: 32 px;
      line-height: 2;
      color: black;
      }
    .checkbox-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); /* Adjust based on your preference */
      grid-gap: 10px; /* Adjust the gap between checkboxes */
      padding: 10px;
      border: 3px solid #ccc; /* Add border around the grid */
      border-radius: 5px; /* Add some border radius for a rounded appearance */
    }
  </style>