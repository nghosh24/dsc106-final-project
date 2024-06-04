<script>
    import { onMount } from 'svelte';
    import mapboxgl from 'mapbox-gl'; // Import Mapbox GL library
    import * as d3 from 'd3'; // Import D3 library
    
    //import geojson from '../../Simple.geojson';

    let map; // Variable to hold the map object
    let Allison = true; // Initial visibility for Rain
    let Katrina = false; // Initial visibility for WindSpeed
    let Sandy = false; // Initial visibility for DamageUSD
    let Hurr4 = true; // Initial visibility for Fatalities
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
      zoom: 2.8 // Initial zoom level
    });

     // Add map controls (optional)
    //map.addControl(new mapboxgl.NavigationControl());

    map.on('load', async () => {

      const response = await fetch('assets/Simple.geojson');
      const geodata = await response.json();

      const can_res = await fetch('assets/canada_provinces.geojson');
      const geo_canada = await can_res.json();

      const carib_res = await fetch('assets/caribbean-islands.geojson');
      const geo_carib = await carib_res.json();

      map.addSource('states', {
        type: 'geojson',
        data: geodata
      });

      //other way to do the layer, but makes gray parts
      /*map.addLayer({
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
      */

      map.addLayer({
        id: 'allison-states',
        type: 'fill',
        source: 'states',
        paint: {
            'fill-color': 'green',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'NAME'], ['literal', statesToColor]]
      });

      map.addSource('canada', {
        type: 'geojson',
        data: geo_canada
      });

      let eastern_canada = ['Nova Scotia', 'New Brunswick', 'Prince Edward Island', 'Newfoundland and Labrador'];

      map.addLayer({
        id: 'allison-canada',
        type: 'fill',
        source: 'canada',
        paint: {
            'fill-color': 'green',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'prov_name_en'], ['literal', eastern_canada]]
      });


      map.addSource('caribbean', {
        type: 'geojson',
        data: geo_carib
      });

      map.addLayer({
        id: 'katrina-caribbean',
        type: 'fill',
        source: 'caribbean',
        paint: {
            'fill-color': 'blue',
            'fill-opacity': 0.5
        },
        filter: ['==', ['get', 'name'], 'Bahamas']
      });

      map.setLayoutProperty('katrina-caribbean', 'visibility', 'none');

      let katrinaStates = statesToColor.filter(state => state !== 'Texas');

      map.addLayer({
        id: 'katrina-states',
        type: 'fill',
        source: 'states',
        paint: {
            'fill-color': 'blue',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'NAME'], ['literal', katrinaStates]]
      });
      map.setLayoutProperty('katrina-states', 'visibility', 'none');


      map.addLayer({
        id: 'katrina-canada',
        type: 'fill',
        source: 'canada',
        paint: {
            'fill-color': 'blue',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'prov_name_en'], ['literal', eastern_canada]]
      });
      map.setLayoutProperty('katrina-canada', 'visibility', 'none');


      let sandyStates = ['Ohio', 'Michicgan', 'Kentucky', 'Tennessee', 'West Virginia'];
      sandyStates = sandyStates.concat(katrinaStates);

      map.addLayer({
        id: 'sandy-states',
        type: 'fill',
        source: 'states', 
        paint: {
            'fill-color': 'red',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'NAME'], ['literal', sandyStates]]
      });
      map.setLayoutProperty('sandy-states', 'visibility', 'none');


      map.addLayer({
        id: 'sandy-canada',
        type: 'fill',
        source: 'canada',
        paint: {
            'fill-color': 'red',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'prov_name_en'], ['literal', eastern_canada]]
      });
      map.setLayoutProperty('sandy-canada', 'visibility', 'none');


      let sandyCaribbean = ['Cuba', 'Haiti', 'Dominican Republic', 'Puerto Rico',
    'Jamaica', 'Cayman Islands'];

      map.addLayer({
        id: 'sandy-caribbean',
        type: 'fill',
        source: 'caribbean',
        paint: {
            'fill-color': 'red',
            'fill-opacity': 0.5
        },
        filter: ['in', ['get', 'name'], ['literal', sandyCaribbean]]
      });
      map.setLayoutProperty('sandy-caribbean', 'visibility', 'none');


  });

  


  var tooltip = new mapboxgl.Popup({
    closeButton: false,
    closeOnClick: false
    });

//hurrican allison
map.on('mousemove', 'allison-states', function (e) {
// Change the cursor style to a pointer when hovering over the layer
    map.getCanvas().style.cursor = 'pointer';

    // Create a tooltip HTML content based on the feature properties
    var tooltipContent = '<h2>Hurricane Allison</h2>' +
                        '<p>Category: Tropical Storm</p>' +
                        '<p>Year: 2001</p>' + 
                        '<p>Rain (Inches): 40</p>' + 
                        '<p>Highest Wind Speed: 60 mph</p>' + 
                        '<p>Damages: 9000000000 USD</p>' + 
                        '<p>Fatalities: 55</p>';

    // Update the tooltip content
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    // Display the tooltip at the mouse position
    tooltip.setLngLat(e.lngLat)
        .setOffset([0, tooltipOffsetY])
        .addTo(map);
});

map.on('mouseleave', 'allison-states', function () {
    // Reset the cursor style and close the tooltip
    map.getCanvas().style.cursor = '';
    tooltip.remove(); // Remove the tooltip
});

map.on('mousemove', 'allison-canada', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Allison</h2>' +
                        '<p>Category: Tropical Storm</p>' +
                        '<p>Year: 2001</p>' + 
                        '<p>Rain (Inches): 40</p>' + 
                        '<p>Highest Wind Speed: 60 mph</p>' + 
                        '<p>Damages: 9000000000 USD</p>' + 
                        '<p>Fatalities: 55</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'allison-canada', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});

//katrina
map.on('mousemove', 'katrina-states', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Katrina</h2>' +
                        '<p>Category: 5</p>' +
                        '<p>Year: 2005</p>' + 
                        '<p>Rain (Inches): 16.43</p>' + 
                        '<p>Highest Wind Speed: 97 mph</p>' + 
                        '<p>Damages: 623000000 USD</p>' + 
                        '<p>Fatalities: 14</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'katrina-states', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});

map.on('mousemove', 'katrina-canada', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Katrina</h2>' +
                        '<p>Category: 5</p>' +
                        '<p>Year: 2005</p>' + 
                        '<p>Rain (Inches): 16.43</p>' + 
                        '<p>Highest Wind Speed: 97 mph</p>' + 
                        '<p>Damages: 623000000 USD</p>' + 
                        '<p>Fatalities: 14</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'katrina-canada', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});

map.on('mousemove', 'katrina-caribbean', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Katrina</h2>' +
                        '<p>Category: 5</p>' +
                        '<p>Year: 2005</p>' + 
                        '<p>Rain (Inches): 16.43</p>' + 
                        '<p>Highest Wind Speed: 97 mph</p>' + 
                        '<p>Damages: 623000000 USD</p>' + 
                        '<p>Fatalities: 14</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'katrina-caribbean', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});

//hurrican sandy
map.on('mousemove', 'sandy-states', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Sandy</h2>' +
                        '<p>Category: 3</p>' +
                        '<p>Year: 2012</p>' + 
                        '<p>Rain (Inches): 3</p>' + 
                        '<p>Highest Wind Speed: 115 mph</p>' + 
                        '<p>Damages: 68700000000 USD</p>' + 
                        '<p>Fatalities: 233</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'sandy-states', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});

map.on('mousemove', 'sandy-canada', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Sandy</h2>' +
                        '<p>Category: 3</p>' +
                        '<p>Year: 2012</p>' + 
                        '<p>Rain (Inches): 3</p>' + 
                        '<p>Highest Wind Speed: 115 mph</p>' + 
                        '<p>Damages: 68700000000 USD</p>' + 
                        '<p>Fatalities: 233</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'sandy-canada', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});

map.on('mousemove', 'sandy-caribbean', function (e) {
    map.getCanvas().style.cursor = 'pointer';

    var tooltipContent = '<h2>Hurricane Sandy</h2>' +
                        '<p>Category: 3</p>' +
                        '<p>Year: 2012</p>' + 
                        '<p>Rain (Inches): 3</p>' + 
                        '<p>Highest Wind Speed: 115 mph</p>' + 
                        '<p>Damages: 68700000000 USD</p>' + 
                        '<p>Fatalities: 233</p>';
    tooltip.setHTML(tooltipContent);

    var tooltipOffsetY = -550;

    tooltip.setLngLat(e.lngLat)
           .setOffset([0, tooltipOffsetY])
           .addTo(map);
});

map.on('mouseleave', 'sandy-caribbean', function () {
    map.getCanvas().style.cursor = '';
    tooltip.remove(); 
});




});

    // Watch for changes in the checkbox values
$: {
    if (map && map.loaded()) {
    map.setLayoutProperty('allison-states', 'visibility', Allison ? 'visible' : 'none');
    map.setLayoutProperty('allison-canada', 'visibility', Allison ? 'visible' : 'none');
    map.setLayoutProperty('katrina-caribbean', 'visibility', Katrina ? 'visible' : 'none');
    map.setLayoutProperty('katrina-states', 'visibility', Katrina ? 'visible' : 'none');
    map.setLayoutProperty('katrina-canada', 'visibility', Katrina ? 'visible' : 'none');
    map.setLayoutProperty('sandy-caribbean', 'visibility', Sandy ? 'visible' : 'none');
    map.setLayoutProperty('sandy-states', 'visibility', Sandy ? 'visible' : 'none');
    map.setLayoutProperty('sandy-canada', 'visibility', Sandy ? 'visible' : 'none');
    }
};

</script>

<main>
    <h2>Effects of the hurricanes</h2>

    <p>Below is a map showing a few of the major hurricanes from the last 20 years. 
        The user can hover over the affected areas to learn more about individual 
        hurricans statistics.
    </p>

    <div id="checkbox-container" class="checkbox-grid">
        <!-- Add checkboxes for each effect -->
        <label ><input type="checkbox" bind:checked={Allison}> Hurrican Allison</label>
        <label ><input type="checkbox" bind:checked={Katrina}> Hurricane Katrina</label>
        <label ><input type="checkbox" bind:checked={Sandy}> Hurricane Sandy</label>
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
      text-align: center;
      font-family: "Georgia";
      font-weight: 400;
      font-size: 42px;
      line-height: 1;
      color: black;
      padding: 10px;
    }
    p {
      text-align: center;
      font-family: "Georgia";
      font-weight: 400;
      font-size: 20px;
      max-width: 800px;
      line-height: 1.5;
      color: black;
      margin: 0 auto;
      margin-bottom: 20px;
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