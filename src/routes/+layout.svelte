<script>
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let pages = [
        {url: "/", title: "Home"},
        {url: "/projects", title: "Projects"},
        {url: "/resume", title: "CV"},
        {url: "/contact", title: "Contact"},
        {url: "https://github.com/msun2022", title: "Github"},
    ];
</script>

<nav>
    {#each pages as p}
        <a 
            href={p.url.startsWith("http") ? p.url : base + p.url}
            class:current={p.url === "/" // is this link the home page?
            ? $page.url.pathname === (base + "/") 
            // if yes - set current = true if the path name matches. 
            // Else, set current = true if the path name starts correctly
            : $page.url.pathname.startsWith(base + p.url)}
            target={p.url.startsWith("http") ? "_blank" : null}
        >
            {p.title}
        </a>
    {/each}
</nav>

<!-- <nav>
    <a href="." class="current">Home</a>
    <a href="projects">Projects</a>
    <a href="contact">Contact</a>
    <a href="resume">CV</a>
    <a href="your github link here" target="_blank">Github</a>
</nav> -->

<!--
class:current={$page.url.pathname.replace(/\/$/, '') === (base + p.url).replace(/\/$/, '')}
-->

<slot />