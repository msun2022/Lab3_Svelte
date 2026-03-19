<script>
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let pages = [
        {url: "/", title: "Home"},
        {url: "/projects", title: "Projects"},
        {url: "/resume", title: "CV"},
        {url: "/contact", title: "Contact"},
        {url: "/meta", title: "Meta"},
        {url: "https://github.com/msun2022", title: "Github"},
    ];

    let colorScheme = "light dark";
    let localStorage = globalThis.localStorage ?? {};
    if (localStorage.colorScheme) {
        colorScheme = localStorage.colorScheme;
    }

    let root = globalThis.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);
    $: localStorage.colorScheme = colorScheme
</script>

<label class="color-scheme-switch">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

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

<slot />

<style>
    nav {
        display: flex;
        margin-bottom: 1em;
        margin-top: 2em;
    }

    nav a {
        flex: 1;
        text-decoration: none;
        color: inherit;
        text-align: center;
        padding: 0.5em;

        border-bottom-width: 1px;
        border-bottom-style: solid;
        border-bottom-color: oklch(50% 10% 200 / 40%);
        /*
        Add a similar border above the nav bar, since ours are at page bottom
        */
        border-top-width: 1 px;
        border-top-style: solid;
        border-top-color: oklch(50% 10% 200 / 40%);
    }

    nav a.current {
        border-bottom-width: 0.4em;
        padding-bottom: 0em;
        border-top-width: 0.4em;
        padding-top: 0em;
    }

    nav a:hover {
        border-bottom-color: var(--color-accent);
        border-top-color: var(--color-accent);
        background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
        border-bottom-width: 0.4em;
        padding-bottom: 0em;
        border-top-width: 0.4em;
        padding-top: 0em;
    }

    label.color-scheme-switch {
        position: absolute;
        top: 0.5rem;
        right: 1rem;
        display: inline-flex;
        gap: 8px;
        font-size: 90%;
    }

    /* :root {
        color-scheme: dark
    } */
</style>