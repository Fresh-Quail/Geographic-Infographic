<template>
  <div id="whole" class="row">
    <svg id="world" class="col-md-6"></svg>
    
    <div id="country-card" class="col-md-6">
      <div style="height: 100%;">
        <h2 class="text-default col-md-12" style="color: steelblue"><i class="fa fa-globe"></i><strong> Country Information</strong></h2>
        <h4>Click a few countries to begin! You can drag too!</h4>
        <!-- <h2 class="col-md-6">Country Information</h2> -->
        <div class="row">
        <div class="col-md-6">
          <!-- <label for="country1">Enter Country #1 Name:&nbsp&nbsp</label>
          <input type="text" id="Country1" placeholder="e.g. United States" v-bind="country1"> -->
          <!-- <button @click="fetchCountryData()">Get Information</button> -->
        </div>
        <div class="col-md-6">
          <!-- <label for="country2">Enter Country#2 Name:&nbsp&nbsp</label>
          <input type="text" id="Country2" placeholder="e.g. United Kingdom" v-bind="country2"> -->
        </div>
        <!-- <button class="btn btn-info" @click="fetchCountriesData()">Get Info!</button> -->
      </div>
          <div id="country"></div>
          <!-- <label for="Top Ten"><strong>Country vs Country Percentages</strong></label> -->
          <svg id="chart"></svg>
          <!-- <label for="Top Ten"><strong>Top Ten Country Populations</strong></label>
            <svg id="chart2" class="content"></svg> -->

      </div>
    
    </div>
  </div>
  </template>
  
  <script>
  import * as d3 from 'd3';
  import versor from "versor"

  export default {
  name: 'Globe',
  components: {

  },

  mounted() {
    this.drawGlobe();
    // this.topTen(this.category);
    // this.divergingBarChart("United States", "China");
    this.validify();
    this.updateChart(0);
  },
  data() {
    return {
      invalidCountries: [],
      countries: ["United States", "China"],
      category: 'Population',
    }
  },
  methods: {
    drawGlobe(){
      const svg = d3.select('#world');
      const width = +svg.node().getBoundingClientRect().width;
      const height = +svg.node().getBoundingClientRect().height;
      
      const g = svg.append('g').attr('width', width).attr('height', height);
      
      const projection = d3.geoOrthographic().scale(300).translate([width/2, height/2]);
      const path = d3.geoPath(projection);
      

      g.append('path').attr('class', 'sphere').attr('d', path({type: 'Sphere'}));

      svg.call(drag(path.projection(), g, path));

      let numSelected = 0;
      let selected = {}
      let ref = this;
      let countryColor = {}
      // Stores the second country selected in order to switch colors
      let d3Element = 0;
      d3.json('https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json').then(function(data) {
        const countries = topojson.feature(data, data.objects.countries); 
        const paths = g.selectAll('path').data(countries.features);
        paths.enter()
          .append('path').attr('class', 'country')
          .attr('d', path)
          .append('title').text(obj => obj.properties.name);

        // Adds a fill on mouseover, keeps fill if the element is clicked, removes fill if it is not clicked
        // Selects what country/countries to display
        g.selectAll('.country').on('click', function(d, i) {
          //Checks if there is an entry in the dataset for the country
          if(ref.invalidCountries.includes(i.properties.name))
              alert("Sorry, " + i.properties.name + " is not supported by this graphic as a country.")
          else{
            // Selects or deselects a country
            if(numSelected < 2){
              selected[i.properties.name] = !selected[i.properties.name];
              if(selected[i.properties.name]){
                // Stores the name of the country selected
                ref.countries[numSelected] = i.properties.name;
                // Changes the color of the hover based on country
                if(numSelected == 0)
                  d3.select(this).style('fill', d3.schemeRdBu[3][0])
                else if(numSelected == 1){
                  d3Element = d3.select(this)
                  d3Element.style('fill', d3.schemeRdBu[3][2])
                }
                numSelected++;
              }
              else{
                // Deselects the country and removes the name
                numSelected--;
                ref.countries[0] == ""
              }
              // Changes the chart based on the number of selected countries
              ref.updateChart(numSelected);
            }
            // Prevents user from selecting more than two countries
            // And deselects the selected country
            else if(selected[i.properties.name]){
              //Swaps the second country to the first
              if(ref.countries[0] == i.properties.name){
                ref.countries[0] = ref.countries[1];
                ref.countries[1] == "";
                d3Element.style('fill', d3.schemeRdBu[3][0])
              }
              else{
                ref.countries[1] = "";
              }
              selected[i.properties.name] = !selected[i.properties.name];
              numSelected--;
              ref.updateChart(numSelected);
            }
          }
        })
        // Adds hover effect
        .on('mouseover', function(d, i) {
          if(d3.select(this).style('fill') != 'rgb(103, 169, 207)' &&
            d3.select(this).style('fill') != 'rgb(239, 138, 98)'){
            countryColor[i.properties.name] = d3.select(this).style('fill')
            if(numSelected == 0)
              d3.select(this).style('fill', d3.schemeRdBu[3][0])
            else if(numSelected == 1)
              d3.select(this).style('fill', d3.schemeRdBu[3][2])
          }
        })
        // Refills with the correct color
        .on('mouseout', function(d, i){
          if(!selected[i.properties.name])
            d3.select(this).style('fill', countryColor[i.properties.name])
        })
      });

      // Adds rotating 
      let rotate = true
      const rotated = () => {
        let rotation = projection.rotate();
        rotation[0] -= 0.1;
        
        projection.rotate(rotation);

        g.selectAll("path").attr("d", path);
        g.select("path").attr('class', 'sphere').attr('d', path({type: 'Sphere'}));
        if(rotate)
          window.requestAnimationFrame(rotated);
      };
      window.requestAnimationFrame(rotated);
      // Adds dragging, math courtesy of Observable HQ
      function drag(projection, g, path) {
        let v0, q0, r0, a0, l;
        function dragstarted({x, y}) {
          v0 = versor.cartesian(projection.invert([x, y]));
          q0 = versor(r0 = projection.rotate());
          //Stops the globe from rotating
          rotate = false
      }

      function dragged(event) {
        const v1 = versor.cartesian(projection.rotate(r0).invert([event.x, event.y]));
        const delta = versor.delta(v0, v1);
        let q1 = versor.multiply(q0, delta);
        projection.rotate(versor.rotation(q1));
        g.selectAll("path").attr("d", path);
        g.select("path").attr('class', 'sphere').attr('d', path({type: 'Sphere'}));
        // Once it comes close to instability, restart.
        if (delta[0] < 0.7) dragstarted.apply(this, [event, this]);
      }

      function restartRotate(){
        rotate = true;
        window.requestAnimationFrame(rotated);
      }
      return d3.drag()
          .on("start", dragstarted)
          .on("drag", dragged)
          .on("end", restartRotate);
      }
      
    },

    updateChart(selected){
      if(selected == 0){
        document.getElementById('country').innerHTML = 
        `<h1>
          Top Ten Countries by 
          <span>
            <select selected = ${this.category} name="cars" id="category">
              <option value="Population">Population</option>
              <option value="Density(P/Km2)">Population/KM2</option>
              <option value="Birth Rate">Birth Rate</option>
              <option value="Life expectancy">Life Expectancy</option>
              <option value="GDP">GDP</option>
              <option value="Co2-Emissions">CO2 Emissions</option>
              <option value="Armed Forces size">Armed Forces Size</option>
            </select>
          </span>
        </h1>`;
        const ref = this;
        const select = document.getElementById("category")
        select.addEventListener('change', function(a) {
          var value = select.options[select.selectedIndex].value;
          ref.category = value
          ref.topTen(ref.category);
        })
        select.value = ref.category
        this.topTen(ref.category);
      }
      else if(selected == 1){
        d3.select('#chart').selectAll('g').remove()
        if(this.countries[0] != '')
          this.countryData(this.countries[0]);
        else
          this.countryData(this.countries[1]);
      }
      else{
        document.getElementById('country').innerHTML = `<h1>${this.countries[0]} versus ${this.countries[1]}</h1>`;
        this.divergingBarChart(this.countries[0], this.countries[1]);
      }
    },

    countryData(inp) {
      const countryInput = inp;
      const url = 'world-data-2023.csv';
      d3.csv(url)
        .then(data => {
          if (data.status === 404) {
            document.getElementById('country').innerHTML = '<p>Country not found</p>';
          } else {
            data.forEach(countryInfo => {
              if(countryInfo.Country == countryInput){
                document.getElementById('country').innerHTML = `
                  <p><strong>Country:</strong> ${countryInfo.Country || 'N/A'}</p>
                  <p><strong>Largest City:</strong> ${countryInfo["Largest city"] || 'N/A'}</p>
                  <p><strong>Population:</strong> ${countryInfo.Population || 'N/A'}</p>
                  <p><strong>Life Expectancy:</strong> ${countryInfo["Life expectancy"] || 'N/A'}</p>
                  <p><strong>GDP:</strong> ${countryInfo.GDP || 'N/A'}</p>
                  <p><strong>Armed Forces Size:</strong> ${countryInfo["Armed Forces size"] || 'N/A'}</p>
                  <p><strong>CO2 Emissions:</strong> ${countryInfo["Co2-Emissions"] || 'N/A'}</p>`;
                }
              })
          }
      }).catch(error => console.log('Error fetching country data:'));
    },

    // fetchCountriesData() {
    //   const country1 = document.getElementById('Country1').value;
    //   const country2 = document.getElementById('Country2').value;
    //   document.getElementById('Country1').innerText = "";

    //   const url = 'world-data-2023.csv';

    //   d3.csv(url)
    //     .then(data => {
    //       if (data.status === 404) {
    //         document.getElementById('countryData').innerHTML = '<p>Country not found</p>';
    //       } else {
    //         let maps = data.map(d => d.Country);
    //         if (maps.includes(country1) && maps.includes(country2))
    //           this.divergingBarChart(country1, country2);
    //       }
    //   }).catch(error => console.error('Error fetching country data:', error));
    // },

  topTen(category){ 
    const url = 'world-data-2023.csv';
    d3.select('#chart').selectAll('g').remove()
    d3.select('#world').selectAll('text').remove()
    const chart = d3.select('#chart')
    .attr('width', '100%')
    .attr('height', '80%')
    // chart.attr('viewBox', `0 0 ${width} ${height}`);
    const width = chart.node().getBoundingClientRect().width;
    const height = chart.node().getBoundingClientRect().height;
    const margin = {top: 25, left: 125, right: 60, bottom: 30}
    const innerWidth = width - margin.left;
    const innerHeight = height - margin.top;

    const renderChart = data => {
      const xScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => Number(d[category]))])
        .range([0, width - margin.right - 100]);

        const yScale = d3.scaleBand()
        .domain(data.map(d => d.Country))
        .range([margin.top, height - margin.bottom])
        .padding(0.2);

      //Moves the bars in the chart, contained in g
      const g = chart.append('g')
        .attr('transform', `translate(${margin.left},0)`);

      //Establishes the axis and moves it
      let xAxis = d3.axisBottom(xScale);
      if(category == "Population")
        xAxis.ticks(8)
      else if(category == "GDP")
        xAxis.ticks(4)
      else if(category == "Armed Forces size")
        xAxis.ticks(9)
              
      g.append('g').call(d3.axisLeft(yScale));
      g.append('g').call(xAxis)
        .attr('transform', `translate(0, ${height - margin.top})`);

      //Adds all of the bars to the chart
      g.selectAll('rect').data(data)
        .enter().append('rect')
          .attr("fill", (d) => d3.color('steelblue'))
          .attr('y', d => yScale(d.Country))
          .attr('width', d => xScale(Number(d[category])))
          .attr('height', yScale.bandwidth());
    }

    const renderChoro = (data)=> {
      const globe = d3.select('#world').selectAll('g').selectAll('.country');
      const domain = d3.extent(data.map(d => Number(d[category])).filter(d => d != 0))
      const colorScale = d3.scaleSequential(d3.interpolateGreens).domain(domain)
      // console.log(colorScale.domain())
      // console.log(colorScale(Number(data[0][category])))
      globe.style('fill', (d) => {
        let country = data.find(dd => dd.Country == d.properties.name)
        if(country != undefined){
          return (colorScale(country[category]))
        }
      })
      // Adds legend to the chart
      const legend = d3.scaleSequential(d3.interpolateGreens).domain([0, 255])
      for(let i = 0; i < 256; i++){
        const rect = d3.select('#world').append('rect')
          .attr('x', 620 + i)
          .attr('y', 20)
          .attr('width', 1)
          .attr('height', 10)
          .attr('fill', legend(i))
        }
        d3.select('#world').append('text')
          .attr('x', 620)
          .attr('y', 15)
          .attr("fill", d3.color('black'))
          .attr("text-anchor", "start")
          .attr("font-weight", "bold")
          .text(category + ": " + domain[0] + " - " + domain[1])

    }
    d3.csv(url).then(data => {
        data.sort((a, b) => b[category] - a[category]);
        // data.forEach(d => {{console.log(d[category]); console.log(parseInt(d[category]))}});
        renderChart(data.slice(0, 10));
        renderChoro(data);
    }).catch(error => console.log('Nothing to see here\n'));
  },

  divergingBarChart(country1, country2){
    const url = 'world-data-2023.csv';
    d3.select('#chart').selectAll('g').remove()

    
    const svg = d3.select('#chart')
    .attr('width', '100%')
    .attr('height', '80%')
    // .attr('viewBox', `0 0 ${width} ${height}`);
    const width = svg.node().getBoundingClientRect().width
    const height = svg.node().getBoundingClientRect().height
    const margin = {top: 50, left: 120, right: 120, bottom: 30}

    // Formatting for scale and bars on chart
    const format = d3.format("+.1%");
    const tickFormat = d3.format("+.0%");

    const render = data => {
      
      const xScale = d3.scaleLinear()
        .domain(d3.extent(data, d => d.value))
        .range([margin.left, width - margin.right]);

      const yScale = d3.scaleBand()
        .domain(data.map(d => d.name))
        .rangeRound([margin.top, height - margin.bottom])
        .padding(0.1);

      const g = svg.append('g')
        
      g.selectAll('rect').data(data)
        .enter().append("rect")
        .attr("fill", (d) => d3.schemeRdBu[3][d.value > 0 ? 2 : 0])
        .attr("x", (d) => (xScale(Math.min(d.value, 0))))
        .attr("y", (d) => yScale(d.name))
        .attr("width", (d) => {
          // console.log(d.value)
          return (Math.abs(xScale(d.value) - xScale(0)))
        })
        .attr("height", yScale.bandwidth());

    // Add a text label for each category.
    g.append("g")
        .attr("font-family", "sans-serif")
        .attr("font-size", 15)
      .selectAll().data(data)
        .enter().append('text')
        .attr("text-anchor", d => d.value < 0 ? "end" : "start")
        .attr("x", (d) => xScale(d.value) + Math.sign(d.value) * 4)
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

    d3.csv(url).then(function(data) {
        let cntry1;
        let cntry2;

        data.forEach(d => {
          if(country1 == d.Country)
            cntry1 = d;
          if(country2 == d.Country)
            cntry2 = d;
        })
          // console.log(cntry1)
          // console.log(cntry2)
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

        newData = newData.filter(d => {if(d.value !=- Infinity && d.value != 1) return (d.value)})
        render(newData);
    }).catch(error => console.log('Country does not exist in dataset. Unsupported.'));
  },

  validify(){
    const url1 = 'world-data-2023.csv';
    const url2 = 'https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json';

    Promise.all([d3.csv(url1), d3.json(url2)]).then(([csvData, jsonData]) => {
      const jsonNames = topojson.feature(jsonData, jsonData.objects.countries).features.map(data => data.properties.name)
      const csvNames = csvData.map(d => d.Country)
      jsonNames.forEach(jsonName => {
        if(!csvNames.includes(jsonName)){
          this.invalidCountries.push(jsonName);
        }
      });
      // console.log(this.invalidCountries)
    })
  }
}
};
</script>
  
<style>
.country {
  border: 1px solid lightblue;
  background-color: #f0f0f0;
}

.sphere{
  fill: #3377FF;
}

#whole {
  height: 100vh;
  margin: 0;
}

#world{
  flex: 1;
  box-sizing: border-box;
  padding: 20px;
}

#country-card {
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  height: 100%;
}
</style>