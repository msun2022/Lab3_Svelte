<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>Some of My Projects (Project Count: {projects.length})</h1>
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


<nav>
    <a href="../">Home</a>
    <a href="./" class="current">Projects</a>
    <a href="../contact">Contact</a>
    <a href="../resume">CV</a>
    <a href="https://github.com/msun2022" target="_blank">Github</a>
</nav>


