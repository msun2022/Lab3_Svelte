<script>
    import { base } from '$app/paths';
    import Scrolly from "svelte-scrolly";
    let scrollyProgress = 0;

    import projects from "$lib/projects.json";
    let sorted_projects = projects.sort((a, b) => a.year - b.year);

    let progressPerProject = 100 / sorted_projects.length;
    $: activeProjectIdx = Math.min(sorted_projects.length-1, Math.floor(scrollyProgress / progressPerProject));
</script>

<Scrolly bind:progress={ scrollyProgress }>
	<!-- Story here -->
    {#each sorted_projects as project}
        <section class="step">
            <div class="step-content">
                <h3>{project.title}</h3>
                <p>{project.story}</p>
            </div>
        </section>
    {/each}
	<svelte:fragment slot="viz">
	    <!-- Visualizations here (these will stay sticky) -->
        <div class="scrolly-wrapper">
            <h3>{sorted_projects[activeProjectIdx].year}</h3>
            <img src="{base}/{sorted_projects[activeProjectIdx].image}" alt="">
        </div>
        <!-- {scrollyProgress}
        {activeProjectIdx} -->
	</svelte:fragment>
</Scrolly>

<style>
    .scrolly-wrapper {
        width: min(1000ch, 60vw);
        position: relative;
        left: 50%;
        transform: translateX(-50%);
    }

    .scrolly-wrapper img {
        width: 60%;
    }

    .step {
        min-height: 80vh;
        padding: 1rem;
        min-width: 20vw;
    }

    .step-content {
        border-left: 2px solid var(--color-accent);
        padding: 0.5rem 1rem;
    }
</style>