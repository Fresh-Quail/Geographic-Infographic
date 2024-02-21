<template>

    <div id="country"></div>

</template>
    
    <script>
    import * as d3 from 'd3';
  
    export default {
    name: 'CountryCard',
  
    mounted() {
        this.countryData()
    },
    data() {
      return {
        countries: ["United States", "China"],
        category: 'GDP',
      }
    },
    methods: {
      updateChart(selected){
        if(selected == 0){
          document.getElementById('country').innerHTML = 
          `<h1>
            Top Ten Countries by 
            <span>
              <select onclick="${console.log('test')}" name="cars" id="category">
                <option value="Population">Population</option>
                <option value="Population/Km2">Population/KM2</option>
                <option value="Birth Rate">Birth Rate</option>
                <option value="Life Expectancy">Life Expectancy</option>
                <option value="GDP">GDP</option>
                <option value="Co2-Emissions">CO2 Emissions</option>
                <option value="Armed Forces size">Armed Forces Size</option>
              </select>
            </span>
          </h1>`;
          var e = document.getElementById("category");
          var value = e.options[e.selectedIndex].value;
          console.log(value)
          this.topTen(value);
        }
        else if(selected == 1){
          d3.select('#chart').selectAll('g').remove()
          document.getElementById('country').innerHTML = `<h1>${this.countries[0]} versus ${this.countries[1]}</h1>`;
          this.showCountry = true;
          if(this.countries[0] != '')
            this.countryData(this.countries[0]);
          else
            this.countryData(this.countries[1]);
        }
        else{
          document.getElementById('country').innerHTML = `<h1>${this.countries[0]} versus ${this.countries[1]}</h1>`;
          this.showCountry = true;
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
                    <p><strong>Country:</strong> ${countryInfo.Country}</p>
                    <p><strong>Largest City:</strong> ${countryInfo["Largest city"]}</p>
                    <p><strong>Population:</strong> ${countryInfo.Population}</p>
                    <p><strong>Life Expectancy:</strong> ${countryInfo["Life expectancy"] || 'N/A'}</p>
                    <p><strong>GDP:</strong> ${countryInfo.GDP || 'N/A'}</p>
                    <p><strong>Armed Forces Size:</strong> ${countryInfo["Armed Forces size"] || 'N/A'}</p>
                    <p><strong>CO2 Emissions:</strong> ${countryInfo["Co2-Emissions"] || 'N/A'}</p>`;
                    console.log(countryInfo);
                  }
                })
            }
        }).catch(error => console.error('Error fetching country data:', error));
      },
  }
  };
  </script>
    
  <style>
  </style>