<script>
    import { onMount } from 'svelte';
    import mapboxgl from 'mapbox-gl'; // Import Mapbox GL library
    import * as d3 from 'd3'; // Import D3 library
    import geojson from 'Simple.geojson';

    let map; // Variable to hold the map object
    let Rainfall = true; // Initial visibility for Rain
    let WindSpeed = true; // Initial visibility for WindSpeed
    let DamageUSD = true; // Initial visibility for DamageUSD
    let Fatalities = true; // Initial visibility for Fatalities
    
    let circleSize = 3; // Initial circle size
    let circleOpacity = 0.6; // Initial circle opacity

    onMount(async () => {
    // Load the CSV data using D3
    const data = await d3.csv('https://raw.githubusercontent.com/nghosh24/dsc106-final-project/main/hurricane_data.csv');
    console.log("First 5 tuples:", data.slice(0, 5));

    mapboxgl.accessToken = "pk.eyJ1Ijoibmdob3NoMjQiLCJhIjoiY2x3NXcwMW9wMW9rODJxbnZ3YnQ0M2YxbSJ9.4pqWQb2ek3y_3Ahya28EHA"; 

    map = new mapboxgl.Map({
      container: 'map', // HTML element ID where the map will be rendered
      style: 'mapbox://styles/mapbox/light-v11', // Map style
      center: [-98.5795, 39.8283], // Center coordinates for continental US
      zoom: 4 // Initial zoom level
    });

     // Add map controls (optional)
     map.addControl(new mapboxgl.NavigationControl());

     // Add circles for each data point
    map.on('load', () => {
      // Define the colors for each ethnicity
      const colors = {
        'Rainfall': 'blue',
        'WindSpeed': 'purple',
        'DamageUSD': 'green',
        'Fatalities': 'red',
      };

      // Group data by ethnicity
      const dataByEthnicity = d3.group(data, d => d.Ethnicity);

      // Add a layer for each ethnicity
      dataByEthnicity.forEach((data, ethnicity) => {
        const layerId = `circles-${ethnicity}`;

        map.addSource(layerId, {
          type: 'geojson',
          data: {
            type: 'FeatureCollection',
            features: data.map(d => ({
              type: 'Feature',
              geometry: {
                type: 'Point',
                coordinates: [parseFloat(d.Longitude), parseFloat(d.Latitude)]
              }
            }))
          }
        });

        map.addLayer({
          id: layerId,
          source: layerId,
          type: 'circle',
          paint: {
            'circle-radius': circleSize,
            'circle-color': colors[ethnicity],
            'circle-opacity': circleOpacity
          },
          layout: {
            'visibility': 'visible' // Initially set visibility to visible
          }
        });
      });
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