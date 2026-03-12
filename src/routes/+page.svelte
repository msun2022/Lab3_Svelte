<script>
    import projects from "$lib/projects.json";
    import Project from "$lib/Project.svelte";
    import readings from "$lib/reading.json";
    import ReadingItem from "$lib/ReadingItem.svelte";
    import GithubStat from "$lib/GithubStat.svelte";

    import { onMount } from "svelte";

    let githubData = null; // This will eventually hold our Github stats
    let loading = true; // This will be true *until* the fetch's promise resolves to a value
    let error = null; // If the API call resulted in an error, it will go into this variable

    // async function retrieveGithubData(){
    //     try {
    //         console.log("Page has been mounted!")
    //         let response = await fetch("https://api.github.com/users/msun2022");
    //         console.log(response);
    //         githubData = await response.json();
    //         console.log(githubData);
    //     } catch (err) { // if the "try" block runs into an error, cancel excecution and run this code instead
    //         error = err;
    //     }
    //     loading = false;
    // }
    // onMount(retrieveGithubData);

    onMount(async () => {
        try {
            console.log("Page has been mounted!")
            let response = await fetch("https://api.github.com/users/msun2022");
            console.log(response);
            githubData = await response.json();
            console.log(githubData);
        } catch (err) { // if the "try" block runs into an error, cancel excecution and run this code instead
            error = err;
        }
        loading = false;
    });
</script>

<h1>Maxwell Sun</h1>

<p>
    I am currently a fourth year undergraduate at MIT studying math and 
    computer science with a focus on machine learning (double major
    in course 18 and 6-4, in MIT lingo). After graduation, I will be 
    working in a proprietary trading firm in Miami. In my free time, 
    I like to go to the gym, eat tasty food, and hang out with my 
    buddies. I am a proud brother of Theta Chi's beta chapter.
</p>
<img src="images/formal_pic.png" alt="Headshot of me in suit" >
<p>This is a picture of me during the 2025 spring semester.</p>

<h2>My Reading List (Hopeful)</h2>
<div class="reading">
    {#each readings as r}
        <ReadingItem data={r} />
    {/each}
</div>

{#if loading}
    <p>Loading...</p>
{:else if error}
    <p>Something went wrong: {error.message}</p>
{:else}
    <!-- <section>
        <h2>My GitHub Stats (Username: {githubData.login})</h2>
        <dl>
            <dt>Followers</dt>
            <dd>{githubData.followers}</dd>
            <dt>Following</dt>
            <dd>{githubData.following}</dd>
            <dt>Public Repositories</dt>
            <dd>{githubData.public_repos}</dd>
        </dl>
    </section> -->
    <section>
        <h2>My GitHub Stats</h2>
        <dl class="github-stat-list">
            <GithubStat label="Username" value={githubData.login} />
            <GithubStat label="Followers" value={githubData.followers} />
            <GithubStat label="Following" value={githubData.following} />
            <GithubStat label="Public Repos" value={githubData.public_repos} />
            <GithubStat label="Created" value={githubData.created_at} />
        </dl>
    </section>
{/if}

<h2>Latest Projects</h2>
<div class="projects">
    {#each projects.slice(-3) as p}
        <Project data={p} />
    {/each}
</div>

<style>
  .github-stat-list {
    display: flex;
    flex-wrap: wrap;
    gap: 30px;
    margin: 0;
    padding: 0.7rem;
    border: 2px solid #7e7e7e;
    border-radius: 8px;
  }
</style>

<!-- <nav>
    <a href="./" class="current">Home</a>
    <a href="./projects">Projects</a>
    <a href="./contact">Contact</a>
    <a href="./resume">CV</a>
    <a href="https://github.com/msun2022" target="_blank">Github</a>
</nav> -->

