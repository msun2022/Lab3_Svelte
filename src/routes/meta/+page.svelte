<script>
    import { base } from '$app/paths';
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';

    let locData = [];
    let commits = [];

    onMount(async () => {
        locData = await d3.csv(`${base}/loc.csv`, row => ({
            ...row,
            line: Number(row.line),
            length: Number(row.length),
            depth: Number(row.depth),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));
        commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/msun2022/Lab3_Svelte/commit/" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length,
                lines: lines
            };

            return ret;
        });
        commits = d3.sort(commits, d => -d.totalLines); 
        // sort in descending order for scatter plot painting
    });

    // Bar chart data, use selected commits if some selected
    $: selectData = (
        selectedCommits.length > 0 
        ? selectedCommits.flatMap(d => d.lines)
        : locData
    );
    $: selectCounts = d3.rollup(selectData, v => v.length, d => d.type);
    $: allLangs = Array.from(new Set(locData.map(d => d.type)));
    // $: barData = d3.rollups(locData, v => v.length, d => d.type)
    //     .map(([type, count]) => ({ label: String(type), value: count }));
    $: barData = allLangs.map(
        lang => ({ label: String(lang), value: selectCounts.get(lang) || 0 })
    );
    import BarHorizontal from '$lib/BarHorizontal.svelte';

    // Conditional title for horizontal bar chart
    let title = "";
    $: title = (
        selectedCommits.length > 0 
        ? `${selectedCommits.length} Selected Commits: Lines of Code per Language`
        : "Website Breakdown: Lines of Code per Language"
    );

    // stuff for scatter plot, first some positioning stuff
    let width = 1000, height = 450;
    let margin = { top: 20, right: 20, bottom: 30, left: 60 };
    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    // Reactively create scatterplot scales
    $: [minDate, maxDate] = d3.extent(commits.map(c => c.date));
    $: dayAfterMaxDate = new Date(maxDate);
    $: dayAfterMaxDate.setDate(dayAfterMaxDate.getDate() + 1);
    
    // Axes and mark size scales
    $: xScale = d3.scaleTime()
                .domain([minDate, dayAfterMaxDate])
                .range([usableArea.left, usableArea.right])
                .nice();
    $: yScale = d3.scaleLinear()
                .domain([24, 0])
                .range([usableArea.bottom, usableArea.top]);
    $: rScale = d3.scaleSqrt()
                .domain(d3.extent(commits.map(c => c.totalLines)))
                .range([3,40])

    // Create axes
    let xAxis, yAxis, yAxisGridlines;
    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(
            d3.axisLeft(yScale)
            .tickFormat(d => String(d % 24).padStart(2, "0") + ":00")
        );

        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
            .tickFormat("")
            .tickSize(-usableArea.width)
        );
    }

    // Hovering tooltip
    let hoveredIndex = -1;
    let cursor = {x: 0, y: 0};
    let commitTooltip;
    let tooltipPosition = {x: 0, y: 0};
    let clickedCommits = [];
    $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

    async function dotInteraction (index, evt) {
        let hoveredDot = evt.target;
        if (evt.type === "mouseenter") {
            hoveredIndex = index;
            cursor = {x: evt.x, y: evt.y};
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed", // because we use position: fixed
                middleware: [
                    offset(5), // spacing from tooltip to dot
                    autoPlacement() // see https://floating-ui.com/docs/autoplacement
                ],
            }); 
        }
        else if (evt.type === "mouseleave") {
            hoveredIndex = -1
        }
        else if (evt.type === "click") {
            let commit = commits[index]
            if (!clickedCommits.includes(commit)) {
                // Add the commit to the clickedCommits array
                clickedCommits = [...clickedCommits, commit];
            }
            else {
                // Remove the commit from the array
                clickedCommits = clickedCommits.filter(c => c !== commit);
            }
        }
    }

    // Brush stuff
    let svg;
    $: {
        d3.select(svg).call(d3.brush()
            .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
            .on("start brush end", brushed)); 

        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    $: brushSelection = null;
    function brushed (evt) {
        brushSelection = evt.selection;
    }
    function isCommitBrushed (commit) {
        if (!brushSelection) {
            return false;
        }
        // TODO return true if commit is within brushSelection
        // and false if not
        let min_x = brushSelection[0][0], max_x = brushSelection[1][0];
        let min_y = brushSelection[0][1], max_y = brushSelection[1][1];
        let x = xScale(commit.datetime), y = yScale(commit.hourFrac);
        return (x >= min_x) && (x <= max_x) && (y >= min_y) && (y <= max_y);
    }
    $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];
    $: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));

    // Line chart stuff
    let linesByDate = [];
    $: {
        const editsPerDate = d3.rollups(
            locData, 
            v => v.length,
            d => d3.timeDay.floor(d.datetime)
        ).map(([date, count]) => ({ date, count }));

        const [minDate, maxDate] = d3.extent(editsPerDate, d => d.date);
        const allDates = d3.timeDays(minDate, d3.timeDay.offset(maxDate, 1));

        linesByDate = allDates.map(date => ({
            date,
            count: editsPerDate.find(d => d.date.getTime() === date.getTime())?.count ?? 0
        }));
    }
    import LineChart from '../../lib/LineChart.svelte';
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>

<!-- Scatterplot -->
<h3 style="text-align: center;">Commits by Date and Time of Day</h3>
<svg viewBox="0 0 {width} {height}" bind:this={svg}>
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" 
        bind:this={yAxisGridlines} />
    <g class="dots">
        {#each commits as commit, index }
            <circle
                on:mouseenter={evt => dotInteraction(index, evt)}
	            on:mouseleave={evt => dotInteraction(index, evt)}
                on:click={ evt => dotInteraction(index, evt) }
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r={ rScale(commit.totalLines) }
                fill="steelblue"
                class:selected={ selectedCommits.includes(commit) }
            />
        {/each}
    </g>
</svg>

<dl 
    class="info tooltip" 
    hidden={hoveredIndex === -1} 
    style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"
    bind:this={commitTooltip}
>
    <dt>Commit</dt>
    <dd>
        <a href="{ hoveredCommit.url }" target="_blank">
            { hoveredCommit.id }
        </a>
    </dd>

    <dt>Date</dt>
    <dd>
        { hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }
    </dd>

    <dt>Time</dt>
    <dd>
        { hoveredCommit.datetime?.toLocaleString("en", {timeStyle: "long"}) }
    </dd>

    <dt>Author</dt>
    <dd>
        { hoveredCommit.author }
    </dd>

    <dt>Lines</dt>
    <dd>
        { hoveredCommit.totalLines }
    </dd>
</dl>

<BarHorizontal data={barData} title={title} />

<LineChart data={linesByDate} />

<style>
	svg {
		overflow: visible;
        margin-bottom: -30px;
	}

    .gridlines {
        stroke-opacity: .2;
    }

    .dots {
        fill-opacity: 60%;
    }

    circle {
        transition: 200ms;
        &:hover {
            fill:darkgreen;
        }
    }

    .selected {
        fill: var(--color-accent);
    }

    /* Toolbar styling */
    dl.info {
        display: grid;
        grid-template-columns: max-content 1fr;
        gap: 0.5rem 1rem;
        margin: 0;

        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }

    dl.info dt {
        grid-column: 1;
        margin: 0;
        color: color-mix(in srgb, currentColor, transparent 25%);
        font-weight: normal;
    }

    dl.info dd {
        grid-column: 2;
        margin: 0;
        font-weight: 500;
        color: currentColor;
    }

    .tooltip {
        position: fixed;
        top: 1em;
        left: 1em;
        padding: 0.5rem;
        /* max-width: 12rem; */
        background-color: color-mix(in srgb, currentColor, transparent 95%);
        border: 1px solid Canvas;
        border-radius: 5px;
        box-shadow: 0 8px 32px color-mix(in srgb, currentColor, transparent 85%);
        backdrop-filter: blur(10px) brightness(90%); 
    }
</style>
