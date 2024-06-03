<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import mapboxgl from 'mapbox-gl';

  mapboxgl.accessToken = 'pk.eyJ1IjoiZXNoYWFucm95IiwiYSI6ImNsd2Z5bDh2YzIxZnAyam52Z3MxbDFqcnEifQ.c0DEmcGJGcj3Dh7voJ6N0w'; // Replace with your Mapbox access token

  let map;

  onMount(async () => {
    map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/light-v10', // Use a basic light style
      center: [-98.5795, 39.8283], // Center the map on the USA
      zoom: 4
    });

    // Load the JSON data using D3
    const data = await d3.json('public/Hurricane_data.json');

    // Convert the hurricane data to GeoJSON format
    const hurricaneGeoJSON = {
      type: 'FeatureCollection',
      features: data.map(hurricane => ({
        type: 'Feature',
        properties: {
          name: hurricane.Name,
          year: hurricane.Year,
          category: hurricane.Category,
          rainInch: hurricane["Rain Inch."],
          windSpeed: hurricane["Highest Wind Speed"],
          damage: hurricane["Damage(USD)"],
          fatalities: hurricane.Fatalities,
          affectedAreas: hurricane["Affected Areas"]
        },
        geometry: {
          type: 'Point',
          coordinates: getCoordinates(hurricane["Affected Areas"]) // Adjust this function based on your data
        }
      }))
    };

    map.on('load', () => {
      // Add the hurricane data as a source
      map.addSource('hurricanes', {
        type: 'geojson',
        data: hurricaneGeoJSON
      });

      // Add a layer to visualize the hurricane data
      map.addLayer({
        id: 'hurricanes',
        type: 'circle',
        source: 'hurricanes',
        paint: {
          'circle-radius': 6,
          'circle-color': '#FF5733',
          'circle-stroke-width': 1,
          'circle-stroke-color': '#FFFFFF'
        }
      });

      // Add a popup on click
      map.on('click', 'hurricanes', (e) => {
        const coordinates = e.features[0].geometry.coordinates.slice();
        const description = `
          <strong>${e.features[0].properties.name}</strong><br>
          Year: ${e.features[0].properties.year}<br>
          Category: ${e.features[0].properties.category}<br>
          Rain: ${e.features[0].properties.rainInch} inches<br>
          Wind Speed: ${e.features[0].properties.windSpeed} mph<br>
          Damage: $${e.features[0].properties.damage}<br>
          Fatalities: ${e.features[0].properties.fatalities}<br>
          Affected Areas: ${e.features[0].properties.affectedAreas}
        `;

        new mapboxgl.Popup()
          .setLngLat(coordinates)
          .setHTML(description)
          .addTo(map);
      });

      // Change the cursor to a pointer when the mouse is over the hurricanes layer
      map.on('mouseenter', 'hurricanes', () => {
        map.getCanvas().style.cursor = 'pointer';
      });

      // Change the cursor back to the default when it leaves the hurricanes layer
      map.on('mouseleave', 'hurricanes', () => {
        map.getCanvas().style.cursor = '';
      });
    });
  });

  // Example function to get coordinates from affected areas (this should be adjusted to your data)
  function getCoordinates(affectedAreas) {
    // Return some default coordinates for now
    return [-98.5795, 39.8283];
  }
</script>

<style>
  #map {
    height: 600px;
    width: 100%;
  }
</style>

<main>
  <div id="map"></div>
</main>