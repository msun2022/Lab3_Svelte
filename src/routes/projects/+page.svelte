<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  let years = projects.map(proj => proj.year)
  let range = Math.max(...years) - Math.min(...years);
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>My {projects.length} Projects Over {range} Years</h1>

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


