<template>
    <h1>
        Top Ten Countries by 
        <span>
        <select v-model="this.category" name="cars" id="category">
            <option value="Population">Population</option>
            <option value="Population/Km2">Population/KM2</option>
            <option value="Birth Rate">Birth Rate</option>
            <option value="Life Expectancy">Life Expectancy</option>
            <option value="GDP">GDP</option>
            <option value="Co2-Emissions">CO2 Emissions</option>
            <option value="Armed Forces size">Armed Forces Size</option>
        </select>
        </span>
    </h1>
    <svg id="chart"></svg>
</template>

<script>
import * as d3 from 'd3';


export default {
name: 'TopTen',
mounted() {
    this.topTen(this.category);
},
data() {
    return {
    category: 'GDP',
    }
},
methods: {
    topTen(category){
        const url = 'world-data-2023.csv';
        d3.select('#chart').selectAll('g').remove()
        const chart = d3.select('#chart')
        .attr('width', '100%')
        .attr('height', '80%')
        // chart.attr('viewBox', `0 0 ${width} ${height}`);
        const width = chart.node().getBoundingClientRect().width;
        const height = chart.node().getBoundingClientRect().height;
        const margin = {top: 25, left: 75, right: 60, bottom: 30}
        const innerWidth = width - margin.left;
        const innerHeight = height - margin.top;

        const render = data => {
        const xScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => Number(d[category]))])
            .range([0, width - margin.right - 40]);

            console.log(xScale.domain())

            const yScale = d3.scaleBand()
            .domain(data.map(d => d.Country))
            .range([margin.top, height - margin.bottom])
            .padding(0.2);

        //Moves the bars in the chart, contained in g
        const g = chart.append('g')
            .attr('transform', `translate(${margin.left},0)`);

        //Establishes the axis and moves it
        g.append('g').call(d3.axisLeft(yScale));
        g.append('g').call(d3.axisBottom(xScale))
            .attr('transform', `translate(0, ${height - margin.top})`);

        //Adds all of the bars to the chart
        g.selectAll('rect').data(data)
            .enter().append('rect')
            .attr("fill", (d) => d3.color('steelblue'))
            .attr('y', d => yScale(d.Country))
            .attr('width', d => xScale(Number(d[category])))
            .attr('height', yScale.bandwidth());
        }

        d3.csv(url).then(data => {
            data.sort((a, b) => b[category] - a[category]);
            data = data.slice(0, 10);
            // data.forEach(d => {{console.log(d[category]); console.log(parseInt(d[category]))}});
            render(data);
        })
    },
}
};
</script>

<style>
</style>