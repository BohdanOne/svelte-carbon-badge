<script>
  import { onMount } from 'svelte';

  export let darkMode = false;

  let wcU;
  let results = null;

  $: co2 = results?.c ? `${results.c}g of CO` : 'Measuring CO';
  $: p = results?.p ? `Cleaner than ${results.p}% of pages tested` : '';

  onMount(async () => {
    wcU = encodeURIComponent(window.location.host);
    results = await getResults();
  });

  async function getResults() {
    const cachedResults = localStorage.getItem(`wcb_${wcU}`);
    if (!cachedResults) {
      return fetchResults();
    } else {
      const parsedResults = JSON.parse(cachedResults);
      return areResultsActual(parsedResults) ? parsedResults : fetchResults();
    }
  }

  function areResultsActual(results) {
    const currentTime = new Date().getTime();
    return currentTime - results.time > 86400000;
  }

  async function fetchResults() {
    try {
      const response = await fetch(
        `https://api.websitecarbon.com/b?url=${wcU}`
      );
      const data = await response.json();
      const parsedResults = {
        ...data,
        time: new Date().getTime(),
      };
      localStorage.setItem(`wcb_${wcU}`, JSON.stringify(parsedResults));
      return parsedResults;
    } catch (error) {
      console.log(error);
      return null;
    }
  }
</script>

<div id="wcb" class="carbonbadge" class:wcb-d={darkMode}>
  <div id="wcb_p">
    <span id="wcb_g"
      >{co2}<sub>2 </sub>/view</span
    ><a
      id="wcb_a"
      target="_blank"
      rel="noopener"
      href="https://websitecarbon.com">Website Carbon</a
    >
  </div>
  <span id="wcb_2">{p}</span>
</div>

<style>
  #wcb {
    --b1: #0e11a8;
    --b2: #00ffbc;
    font-size: .9375rem;
    text-align: center;
    color: var(--b1);
    margin: 4em auto;
    display: flex;
    flex-direction: column;
  }
  #wcb sub {
    vertical-align: middle;
    position: relative;
    top: 0.3em;
    font-size: 0.7em;
  }
  #wcb_2,
  #wcb_a,
  #wcb_g {
    font-size: 1em;
    line-height: 1.15;
    font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    text-decoration: none;
    margin: 0.2em 0;
  }
  #wcb_p {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    flex-wrap: wrap;
  }
  #wcb_a,
  #wcb_g {
    padding: 0.3em 0.5em;
    border: 0.13em solid var(--b2);
  }
  #wcb_g {
    border-radius: 0.3em 0 0 0.3em;
    background: #fff;
    border-right: 0;
    min-width: 8.2em;
  }
  #wcb_a {
    border-radius: 0 0.3em 0.3em 0;
    border-left: 0;
    background: var(--b1);
    color: #fff;
    font-weight: 700;
    border-color: var(--b1);
  }
  .wcb-d #wcb_a {
    color: var(--b1);
    background: var(--b2);
    border-color: var(--b2);
  }
  .wcb-d #wcb_2 {
    color: #fff;
  }
</style>
