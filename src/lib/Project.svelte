<script>
    import { base } from '$app/paths';
    export let data = {};
</script>

<article>
    <h3>{data.title}</h3>
    <img src="{base}/{data.image}" alt="">
    <p>{data.description}</p>
</article>

<style>
    img {
        max-width: 100%;
    }

    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    article h3 {
        margin: 0; 
    }
</style>