<script>
    import * as d3 from 'd3';

    let width = 400;
    let height = 300;

    // let data = [
    //     { label: "A", value: 10 },
    //     { label: "B", value: 20 },
    //     { label: "C", value: 15 },
    //     { label: "D", value: 8 },
    //     { label: "E", value: 25 }
    // ];
    export let data = [];

    // let margin = { top: 20, right: 20, bottom: 30, left: 60 };
    let margin = { top: 40, right: 100, bottom: 80, left: 60 };
    let innerWidth  = width  - margin.left - margin.right;
    let innerHeight = height - margin.top  - margin.bottom;

    $: xScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerWidth])
        .padding(0.2);

    $: yScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.value) || 1])
        .range([innerHeight, 0]);

    $: colorScale = d3.scaleOrdinal()
        .domain(data.map(d => d.label))
        .range(d3.quantize(d3.interpolateBlues, data.length));
    // $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    //     .domain(data.map(d => d.label));

    let xAxis, yAxis;

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(
            d3.axisLeft(yScale)
                .tickFormat(d => Number.isInteger(d) ? d : "")
                .tickValues(d3.range(0, d3.max(data, d => d.value) + 1))
        );
    }

    $: maxBar = d3.greatest(data, d => d.value);

    let selectedIndex = -1;
    let liveText = "";
    function toggleBar(index, event) {
        if (!event.key || event.key === "Enter") {
            selectedIndex = index;
            const d = data[index];
            liveText = `${d.label}: ${d.value} projects selected.`;
        }
    }
    $: description = `A bar chart showing project counts by year. ${data.map(d => `${d.label}: ${d.value} projects`).join(', ')}.`;

    let showChart = true;

    function toggleView() {
        showChart = !showChart;
        liveText = showChart ? "Bar chart view shown." : "Table view shown.";
    }
</script>

<button
    on:click={toggleView}
    aria-pressed={!showChart}
    aria-label="Toggle between bar chart and table view"
    class="toggle-button">
        {showChart ? 'Show Table' : 'Show Chart'}
</button>

{#if showChart}
    <div class="container">
        <svg 
            viewBox="0 0 {width} {height}"
            role="img"
            aria-labelledby="bar-title bar-desc">
            <title id="bar-title">Projects by Year</title>
            <desc id="bar-desc">{description}</desc>
        
            <g transform="translate({margin.left}, {margin.top + innerHeight})"
            bind:this={xAxis} />
            <g transform="translate({margin.left}, {margin.top})"
            bind:this={yAxis} />
            <g transform="translate({margin.left}, {margin.top})">
                {#each data as d, index}
                    <rect
                        x={xScale(d.label)}
                        y={yScale(d.value)}
                        width={xScale.bandwidth()}
                        height={innerHeight - yScale(d.value)}
                        fill={colorScale(d.label)}
                        opacity={selectedIndex === -1 || selectedIndex === index ? 1 : 0.45}
                        on:click={e => toggleBar(index, e)}
                        on:keyup={e => toggleBar(index, e)}
                        tabindex="0"
                        role="button"
                        aria-label={`${d.label}: ${d.value} projects`}
                    />
                {/each}
                {#if maxBar}
                    <!-- highlight outline around the tallest bar -->
                    <rect
                        x={xScale(maxBar.label)}
                        y={yScale(maxBar.value)}
                        width={xScale.bandwidth()}
                        height={innerHeight - yScale(maxBar.value)}
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                    />
                    <!-- leader line -->
                    <line
                        x1={xScale(maxBar.label) + xScale.bandwidth()}
                        y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
                        x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
                        y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
                        stroke="currentColor"
                        stroke-width="1"
                    />
                    <!-- annotation text at end of leader line -->
                    <text
                        x={xScale(maxBar.label) + xScale.bandwidth() + 35}
                        y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
                        dominant-baseline="middle"
                        class="annotation">
                        Year with most projects
                    </text>
                {/if}
            </g>

            <!-- Chart Title -->
            <text
                x={margin.left + innerWidth / 2}
                y={margin.top / 2}
                text-anchor="middle"
                class="chart-title">
                Projects per Year
            </text>

            <!-- x-axis label -->
            <text
                x={innerWidth / 2 + 50}
                y={innerHeight + margin.bottom }
                text-anchor="middle"
                class="axis-label">
                Year
            </text>

            <!-- y-axis label -->
            <text
                x={-(innerHeight / 2) - 30}
                y={-margin.left + 90}
                text-anchor="middle"
                transform="rotate(-90)"
                class="axis-label">
                Number of Projects
            </text>
        </svg>
        <p aria-live="polite" class="sr-only">{liveText}</p>
        <ul class="legend">
            {#each data as d}
                <li style="--color: {colorScale(d.label)}">
                    <span class="swatch"></span>
                    {d.label} <em>({d.value})</em>
                </li>
            {/each}
        </ul>
    </div>
{:else}
    <table aria-label="Table showing project counts by year" class="data-table">
        <caption>Projects by Year</caption>
        <thead>
            <tr>
            <th id="year-header" scope="col">Year</th>
            <th id="projects-header" scope="col">Projects</th>
            </tr>
        </thead>
        <tbody>
            {#each data as d, i}
            <tr>
                <th id="row-{i}" scope="row">{d.label}</th>
                <td aria-labelledby="row-{i} projects-header">{d.value}</td>
            </tr>
            {/each}
        </tbody>
    </table>
{/if}


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
        font-size: 1em;
        font-weight: bold;
        fill: currentColor;
    }

    /* Chart Axes Styling */
    .axis-label {
        font-size: 0.8em;
        fill: currentColor;
    }

    /* Annotation */
    .annotation {
        font-size: 0.7em;
        fill: gray;
        font-style: italic;
    }

    rect {
        transition: 300ms;
        outline: none;
        stroke: currentColor;
        stroke-width: 1;
    }

    svg:hover rect:not(:hover), .container:focus-within rect:not(:focus-visible) { opacity: 50%; }
    rect:focus-visible {
    stroke: white;
    stroke-width: 2px;
    stroke-dasharray: 4;
    }

    .sr-only {
        position: absolute;
        left: -9999px;
        width: 1px;
        height: 1px;
        overflow: hidden;
    }

    .data-table {
        margin-top: 1rem;
        margin-bottom: 1rem;
        border-collapse: collapse;
        width: 100%;
        max-width: 30em;
        --table-border: color-mix(in srgb, currentColor, transparent 80%);
        --table-header-bg: color-mix(in srgb, currentColor, transparent 92%);
    }

    .data-table caption {
        font-weight: bold;
        margin-bottom: 0.5em;
        text-align: left;
    }

    .data-table th, .data-table td {
        border: 1px solid var(--table-border);
        padding: 0.5em;
        text-align: left;
    }

    .data-table th {
        background-color: var(--table-header-bg);
    }
</style>