<script>
    import { base } from '$app/paths';
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let locData = [];

    onMount(async () => {
        locData = await d3.csv(`${base}/loc.csv`, row => ({
            ...row,
            line: Number(row.line),
            length: Number(row.length),
            depth: Number(row.depth)
        }));
    });

    $: barData = d3.rollups(locData, v => v.length, d => d.type)
        .map(([type, count]) => ({ label: String(type), value: count }));
    import BarHorizontal from '$lib/BarHorizontal.svelte';
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>
    <h1>Meta</h1>

    <BarHorizontal data={barData} />

