<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  let years = projects.map(proj => proj.year)
  let range = Math.max(...years) - Math.min(...years);

  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  // let rawData = [];
  // let wrangled = [];
  // let wrangled_perc = [];
  // let lines_total = 0;

  // onMount(async () => {
  //   rawData = await d3.json('/lab6_example.json');
  //   lines_total = d3.sum(rawData, d => d.lines);
  //   wrangled = d3.rollups(
  //     rawData,
  //     v => d3.sum(v, d => d.lines),
  //     d => d.language
  //   );
  //   wrangled_perc = d3.rollups(
  //     rawData,
  //     v => d3.sum(v, d => d.lines)/lines_total,
  //     d => d.language
  //   )
  // });

  import Bar from '$lib/Bar.svelte';
  $: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>My {projects.length} Projects Over {range} Years</h1>

<Bar data={barData}/>

<p>Scroll down to see my a timeline of my projects and how they relate to my professional and personal life</p>

<ProjectNarrative />

<p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>

<div class="projects">
  {#each projects as p}
    <!-- <article>
        <h2>{p.title}</h2>
        <img src={p.image} alt="">
        <p>{p.description}</p>
    </article> -->
    <Project data={p} />
  {/each}
</div>

<!-- <section>
  <h2>Data wrangling result</h2>
  <pre>{JSON.stringify(wrangled, null, 2)}</pre>
  <pre>{JSON.stringify(wrangled_perc, null, 2)}</pre>
</section> -->

<style>
  .outro {
    margin-bottom: 3rem;
  }
</style>


<!-- <nav>
    <a href="../">Home</a>
    <a href="./" class="current">Projects</a>
    <a href="../contact">Contact</a>
    <a href="../resume">CV</a>
    <a href="https://github.com/msun2022" target="_blank">Github</a>
</nav> -->


