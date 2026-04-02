<script>
    import * as d3 from 'd3';

    let width = 400;
    let height = 170;

    export let data = [];
    export let title = "";

    let margin = { top: 40, right: 70, bottom: 70, left: 60 };
    let innerWidth  = width  - margin.left - margin.right;
    let innerHeight = height - margin.top  - margin.bottom;

    $: catScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerHeight])
        .padding(0.2);

    $: valScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.value) || 1])
        .range([0, innerWidth]);

    $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
        .domain(data.map(d => d.label));

    let xAxis, yAxis;

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(
            d3.axisBottom(valScale)
            .ticks(Math.min(d3.max(data, d => d.value), 10))
        );
        d3.select(yAxis).call(d3.axisLeft(catScale));
    }

    $: maxBar = d3.greatest(data, d => d.value);
</script>

<div class="container">
    <svg viewBox="0 0 {width} {height}">
        <g transform="translate({margin.left}, {margin.top + innerHeight})"
        bind:this={xAxis} />
        <g transform="translate({margin.left}, {margin.top})"
        bind:this={yAxis} />
        <g transform="translate({margin.left}, {margin.top})">
            {#each data as d}
                <rect
                    x=0
                    y={catScale(d.label)}
                    width={valScale(d.value)}
                    height={catScale.bandwidth()}
                    fill={colorScale(d.label)}
                />
            {/each}
            {#if maxBar}
                <!-- highlight outline around the tallest bar -->
                <rect
                    x=0
                    y={catScale(maxBar.label)}
                    width={valScale(maxBar.value)}
                    height={catScale.bandwidth()}
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                />
                <!-- leader line -->
                <line
                    x1={valScale(maxBar.value)}
                    y1={catScale(maxBar.label) + catScale.bandwidth() / 2}
                    x2={valScale(maxBar.value) + 15}
                    y2={catScale(maxBar.label) + catScale.bandwidth() / 2}
                    stroke="currentColor"
                    stroke-width="1"
                />
                <!-- annotation text at end of leader line -->
                <text
                    x={valScale(maxBar.value) + 20}
                    y={catScale(maxBar.label) + catScale.bandwidth() / 2}
                    text-anchor="start"
                    dominant-baseline="middles"
                    class="annotation">
                    Most lines
                </text>
            {/if}
        </g>

        <!-- Chart Title -->
        <text
            x={margin.left + innerWidth / 2}
            y={margin.top / 2}
            text-anchor="middle"
            class="chart-title">
            {title}
        </text>

        <!-- x-axis label -->
        <text
            x={innerWidth / 2 + 50}
            y={innerHeight + margin.bottom }
            text-anchor="middle"
            class="axis-label">
            Number of Lines of Code
        </text>

        <!-- y-axis label -->
        <text
            x={-(innerHeight / 2) - 30}
            y={-margin.left + 70}
            text-anchor="middle"
            transform="rotate(-90)"
            class="axis-label">
            Coding Language
        </text>
    </svg>
    <ul class="legend">
        {#each data as d}
            <li style="--color: {colorScale(d.label)}">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>


<style>
    svg {
        max-width: 100%;
        height: auto;
        overflow: visible;
    }

    /* Side-by-side layout */
    .container {
        display: flex;
        align-items: flex-start; /* Keeps legend at the top if chart is tall */
        justify-content: flex-start;
        gap: 2rem; /* Space between the chart and the legend */
        margin: 2rem 0;
        margin-bottom: -1rem;
        max-width: 85%;
    }

    /* The Legend Grid */
    .legend {
        flex: 1; /* Fills available width */
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
        gap: 1.5rem; /* Larger gap between different legend items */
        padding: 0;
        list-style: none;
    }

    /* Individual List Items */
    li {
        display: flex;
        align-items: center;
        /* Proximity: Smaller gap here than in the .legend grid gap */
        gap: 0.5rem; 
        font-family: sans-serif;
        font-size: 0.9rem;
    }

    /* The Color Swatch */
    .swatch {
        width: 1rem;
        height: 1rem;
        background-color: var(--color);
        border-radius: 3px; /* Slight rounding for a modern look */
        flex-shrink: 0; /* Prevents square from squishing if text is long */
    }

    li em {
        font-style: italic;
        margin-left: 0.2rem;
    }

    /* Chart Title */
    .chart-title {
        font-size: 0.7em;
        font-weight: bold;
        fill: currentColor;
    }

    /* Chart Axes Styling */
    .axis-label {
        font-size: 0.5em;
        fill: currentColor;
    }

    /* Annotation */
    .annotation {
        font-size: 0.4em;
        fill: color-mix(in srgb, currentColor, transparent 30%);
        font-style: italic;
    }
</style>