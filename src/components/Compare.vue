<template>
    <h1 id="compare"></h1>
    <svg id="chart"></svg>
</template>

<script>
import * as d3 from 'd3';

export default {
    name: 'Compare',
    mounted() {
        this.divergingBarChart("United States", "China");
    },
    data() {
        return {
        countries: ["United States", "China"],
        }
    },
    methods: {
    divergingBarChart(country1, country2){
        const url = 'world-data-2023.csv';
        document.getElementById('compare').innerHTML = `<h1>${this.countries[0]} versus ${this.countries[1]}</h1>`; 
        d3.select('#chart').selectAll('g').remove()

        
        const svg = d3.select('#chart')
        .attr('width', '100%')
        .attr('height', '80%')
        // .attr('viewBox', `0 0 ${width} ${height}`);
        const width = svg.node().getBoundingClientRect().width
        const height = svg.node().getBoundingClientRect().height
        console.log(width)
        console.log(height)
        const margin = {top: 50, left: 120, right: 60, bottom: 30}

        // Formatting for scale and bars on chart
        const format = d3.format("+.1%");
        const tickFormat = d3.format("+.0%");

        const render = data => {
        
        const xScale = d3.scaleLinear()
            .domain(d3.extent(data, d => d.value))
            .rangeRound([margin.left, width - margin.right]);

        const yScale = d3.scaleBand()
            .domain(data.map(d => d.name))
            .rangeRound([margin.top, height - margin.bottom])
            .padding(0.1);

        const g = svg.append('g')
            // .attr('transform', `translate(${margin.left},${margin.top})`);
            
        /* ----- */
        g.selectAll('rect').data(data)
            .enter().append("rect")
            .attr("fill", (d) => d3.schemeRdBu[3][d.value > 0 ? 2 : 0])
            .attr("x", (d) => (xScale(Math.min(d.value, 0))))
            .attr("y", (d) => yScale(d.name))
            .attr("width", (d) => (Math.abs(xScale(d.value) - xScale(0))))
            .attr("height", yScale.bandwidth());

        // Add a text label for each category.
        g.append("g")
            .attr("font-family", "sans-serif")
            .attr("font-size", 15)
        .selectAll().data(data)
            .enter().append('text')
            .attr("text-anchor", d => d.value < 0 ? "end" : "start")
            .attr("x", (d) => xScale(d.value) + Math.sign(d.value - 0) * 4)
            .attr("y", (d) => yScale(d.name) + yScale.bandwidth() / 2)
            .attr("dy", "0.35em")
            .text(d => format(d.value));

        // Add the axes and grid lines.
        g.append("g")
        .attr("transform", `translate(0,${margin.top})`)
        .call(d3.axisTop(xScale).ticks(width / 100).tickFormat(tickFormat))
        .call(g => g.selectAll(".tick line").clone()
                .attr("y2", height - margin.top - margin.bottom)
                .attr("stroke-opacity", 0.1))
                .attr("font-size", 15);

        g.append("g")
        .attr("transform", `translate(${xScale(0)},0)`)
        .call(d3.axisLeft(yScale).tickSize(0).tickPadding(6))
        .call(g => g.selectAll(".tick text").filter((d, i) => data[i].value <= 0)
            .attr("text-anchor", "start")
            .attr("font-size", 13)
            .attr("x", 10))
        .call(g => g.selectAll(".tick text").filter((d, i) => data[i].value > 0)
            .attr("font-size", 13));
        }

        d3.csv(url).then(data => {
            let cntry1 = 0;
            let cntry2 = 0;

            data.forEach(d => {
            if(country1 == d.Country)
                cntry1 = d;
            if(country2 == d.Country)
                cntry2 = d;
            })

            let newData = [];
            newData[0] = {name: "Population", value: (cntry2["Population"] - cntry1["Population"]) / cntry2["Population"]}
            newData[1] = {name: "Population/Km2)", value: (cntry2["Density(P/Km2)"] - cntry1["Density(P/Km2)"]) / cntry2["Density(P/Km2)"]}
            newData[2] = {name: "GDP", value: (cntry2["GDP"] - cntry1["GDP"]) / cntry2["GDP"]};
            newData[3] = {name: "Armed Forces size", value: (cntry2["Armed Forces size"] - cntry1["Armed Forces size"]) / cntry2["Armed Forces size"]};
            newData[4] = {name: "Co2-Emissions", value: (cntry2["Co2-Emissions"] - cntry1["Co2-Emissions"]) / cntry2["Co2-Emissions"]};
            newData[5] = {name: "Life expectancy", value: (cntry2["Life expectancy"] - cntry1["Life expectancy"]) / cntry2["Life expectancy"]};
            newData[6] = {name: "Birth Rate", value: (cntry2["Birth Rate"] - cntry1["Birth Rate"]) / cntry2["Birth Rate"]};
            newData.sort((a, b) => a.value - b.value)
            newData.map((d) => {return {name: d.name, value: Number(d.value)}})
            render(newData);
        }).catch(error => console.error('Error fetching country data:', error));
    },
    }
};
</script>

<style>
</style>