<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let isLoading = true;
  let tempData = [];
  let datasetSize = 0; // Variable to store the size of the dataset
  let yearCounts = {}; // Object to store counts for each year
  let currentYear = 2000; // Initialize the current year

  let svg;
  let margin = { top: 55, right: 20, bottom: 30, left: 90 };
  let width = 960 - margin.left - margin.right;
  let height = 500 - margin.top - margin.bottom;

  onMount(async () => {
    const res = await fetch('assets/hurricane_data.csv');
    const csv = await res.text();
    tempData = d3.csvParse(csv, d3.autoType);

    // Calculate the size of the dataset
    datasetSize = tempData.length;

    // Count incidents for each year
    tempData.forEach(row => {
      const year = row['Year'];
      if (yearCounts[year]) {
        yearCounts[year]++;
      } else {
        yearCounts[year] = 1;
      }
    });

    isLoading = false;

    // Call the function to render the bar graph
    renderBarGraph();
  });

function renderBarGraph() {
    // Remove existing SVG
    d3.select("#barGraph").selectAll("svg").remove();

    // Append new SVG
    svg = d3.select("#barGraph").append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    // Data for the bar graph
    let data = Object.keys(yearCounts).map(year => ({ year: parseInt(year), count: yearCounts[year] }));

    // Define a linear color scale based on the count of hurricanes
    let colorScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.count) / 3, 2 * d3.max(data, d => d.count) / 3, d3.max(data, d => d.count)])
      .range(["cyan", 'blue', 'purple', 'red']);

    // X scale
    let x = d3.scaleBand()
      .domain(data.map(d => d.year))
      .range([0, width])
      .padding(0.1);

    // Y scale
    let y = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.count)])
      .nice()
      .range([height, 0]);

    // X axis
    svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x));

    // Y axis
    svg.append("g")
      .call(d3.axisLeft(y));

      // Add y-axis label
    svg.append("text")
        .attr("transform", "rotate(-90)")
        .attr("y", 0 - (margin.left) )
        .attr("x", 0 - (height / 2))
        .attr("dy", "1em")
        .style("text-anchor", "middle")
        .text("Number of Hurricanes");

    // Add text element to display the selected year above the chart
    svg.append("text")
        .attr("x", width / 2)
        .attr("y", -margin.top / 2)
        .attr("text-anchor", "middle")
        .style("font-size", "20px") // Larger font size
        .style("padding-top", "10px") // Padding above the text
        .style("padding-bottom", "15px") // Padding below the text
        .text("Hurricanes from 2000 to 2023");

    // Bars
    svg.selectAll(".bar")
      .data(data)
      .enter().append("rect")
      .attr("class", "bar")
      .attr("x", d => x(d.year))
      .attr("y", height) // Start bars from the bottom
      .attr("width", x.bandwidth())
      .attr("height", 0) // Start bars with zero height
      .attr("fill", d => colorScale(d.count)) // Set fill color based on count using color scale
      .attr("y", d => y(d.count))
      .attr("height", d => height - y(d.count));

   // Add text labels to the bars
  // Calculate bar width and half bar width
  let barWidth = x.bandwidth();
  let halfBarWidth = barWidth / 2;
  let verticalTextOffsetY = 30;
// Add vertical text labels

svg.selectAll('.label')
    .data(data)
    .enter()
    .append('text')
    .attr('x', (d) => x(d.year) + halfBarWidth -250  ) // Set initial x position to the final position
    .attr('y', d => y(d.count) + 10 ) // Set initial y position to the final position
    .text(d => d.count.toLocaleString())
    .attr('transform', d => `rotate(-90 ${x(d.year) + halfBarWidth} ${y(d.count)})`) // Set initial rotation
    .attr('class', 'label')
    .attr('text-anchor', 'right')
    .style('font-family', "Georgia")
    .style('font-weight', '400')
    .style('font-size', '24px')
    .style('fill', 'white')
    .attr('y', d => y(d.count) + 5) // Transition the 'y' attribute to its final position, adjusting it lower
    .attr('x', (d) => x(d.year) + halfBarWidth - 30); // Transition the 'x' attribute to its final position



    // Add hover effect to display value on mouseover
svg.selectAll(".bar")
  .on("mouseover", function(e, d) {
    d3.select(this).attr("fill", "orange");
  })
  .on("mouseout", function(d) {
    d3.select(this)
      .attr("fill", d => colorScale(d.count));
  });

}


</script>

<main>
  {#if isLoading}
    <h2>Loading CSV file...</h2>
  {:else}
    <h2>From 2000 to 2023</h2>
    <p>
      This graph shows the number of hurricanes each year from 2000 to 2023.
      <br> 
      Hover over the bars to highlight them!
    </p>
  {/if}
  <!-- Container for the bar graph -->
  <div id="barGraph"></div>
  <br><br>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
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
    line-height: 2;
    color: black;
    padding: 10px;
  }

  #barGraph {
    width: 120%; 
    max-width: 1200px; 
  }

  svg {
    font: 10px sans-serif;
  }
  .bar:hover {
    fill: orange;
  }
  .axis {
    shape-rendering: crispEdges;
  }
  .axis path,
  .axis line {
    fill: none;
    stroke: #000;
  }
  .hover-text {
    font-size: 12px;
    fill: black;
  }
</style>