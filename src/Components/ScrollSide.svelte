<script>
  import Scrolly from "./Scrolly.svelte";
  import katexify from "../katexify";
  import { select, selectAll } from "d3-selection";

  // scroll iterator
  let value;

  // Paragraph text for scrolly
  $: steps = [
    `<h1 class='background-info'>What causes hurricanes?</h1>
      <p>
        Hurricanes are formed over warm ocean waters, and the interaction with warm air creates low pressure zones that further fuel the storm. This creates a rotating system of clouds and thunderstorms, which can then move over land and cause severe damage.
      </p>
    <br><br>
  `,
    `<h1 class='background-info'>Categories</h1>
      <p>
        Hurricanes are classified into 6 categories: Tropical Storm, and then Category 1 through 5. A Category storm is the most severe hurricane. These classifications are based on the hurricane wind speed. Category 5 hurricanes have a sustained wind speed of greater than 157 miles per hours.
        </p>
    <br><br>
  `,
  ];

  const target2event = {
    0: () => {
      select("#chart1").style("background-image", "none");
    },
    1: () => {
      select("#chart1").style("background-image", `url(${images[4]})`)
      .style("background-size", "contain")
      .style("background-repeat", "no-repeat");
    },

    2: () => {
      select("#chart1").style("background-image", `url(${images[3]})`)
      .style("background-size", "contain")
      .style("background-repeat", "no-repeat");
    },
  };

  const images = [
    "assets/hurricane1.jpeg",
    "assets/hurricane_katrina.jpeg",
    "assets/hurricane_katrina.jpeg",
    "assets/hurricane_ian.jpeg",
    "assets/categories.jpeg"
  ];

  $: if (typeof value !== "undefined") target2event[value]();
</script>

<h2 class="body-header">Background Info</h2>
<p class="body-text">
  Here is some additional information to understand how hurricanes are formed and classified.
</p>
<section style="background-image: url({images[value]})">
  <!-- scroll container -->
  <div class="section-container">
    <div class="steps-container">
      <Scrolly bind:value>
        {#each steps as text, i}
          <div class="step" class:active={value === i}>    
            <div class="step-content">{@html text}</div>
          </div>
        {/each}
        <div class="spacer" />
      </Scrolly>
    </div>
    <div class="charts-container">
      <div class="chart-one">
        <svg id="chart1" />
      </div>
    </div>
  </div>
  <br /><br />
</section>
<p class="body-text">Now we can dive into further exploration of the hurricanes in North America over the past 2 decades.</p>


<style>
  #chart1 {
    width: 100%;
    height: 100%;
  }
  .chart-one {
    width: 100%;
    height: 100%;
    background-position: center;
  }

  /* space after scroll is finished */
  .spacer {
    height: 5vh;
  }

  .charts-container {
    position: sticky;
    top: 10%;
    display: grid;
    width: 50%;
    grid-template-columns: 100%;
    grid-row-gap: 0rem;
    grid-column-gap: 0rem;
    grid-template-rows: 1fr;
    height: 70vh;
  }

  .section-container {
    margin-top: 40px;
    text-align: center;
    transition: background 100ms;
    display: flex;
  }

  .step {
    height: 110vh;
    display: flex;
    place-items: center;
    justify-content: center;
  }

  .step-content {
    font-size: 20px;
    background: var(--bg);
    color: #ccc;
    border-radius: 1px;
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    text-align: left;
    width: 75%;
    margin: auto;
    max-width: 500px;
    font-family: 'Georgia';
    line-height: 1.3;
    border: 5px solid var(--default);
  }

  .step.active .step-content {
    background: #f1f3f3ee;
    color: var(--squid-ink);
  }

  .steps-container {
    height: 100%;
  }

  .steps-container {
    flex: 1 1 40%;
    z-index: 10;
  }

  .body-header {
    font-size: 30px; 
    font-weight: bold; 
    margin-bottom: 10px; 
    font-family: 'Georgia';
    max-width: 800px;
    text-align: center;
  }

  .body-text {
    font-size: 22px; 
    line-height: 1.5; 
    margin-bottom: 20px; 
    margin-top: 20px;
    text-align: center;
    font-family: 'Georgia';
    max-width: 1000px;

  }

  /* Comment out the following line to always make it 'text-on-top' */
  @media screen and (max-width: 950px) {
    .section-container {
      flex-direction: column-reverse;
    }

    .steps-container {
      pointer-events: none;
    }

    .charts-container {
      top: 7.5%;
      width: 95%;
      margin: auto;
    }

    .step {
      height: 130vh;
    }

    .step-content {
      width: 95%;
      max-width: 768px;
      font-size: 17px;
      line-height: 1.6;
    }

    .spacer {
      height: 100vh;
    }
  }
</style>
