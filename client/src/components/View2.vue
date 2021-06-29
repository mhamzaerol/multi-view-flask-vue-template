<template>
  <div>
    <div style="background: grey">View2</div>
    <div id="view2svg"></div>
  </div>
</template>

<script>
import DataService from "../services/data-service";
import PipeService from "../services/pipe-service";
import NetService from "../services/net-service";
import * as d3 from "d3";
import cloud from "d3-cloud";
// console.log(cloud);
export default {
  name: "View2",
  data() {
    return {
      selection: null,
    };
  },
  mounted() {
    PipeService.$on(PipeService.UPDATE_VIEW2, () => {
      this.selection = DataService.selection;
      this.draw(this.selection);
      // console.log(DataService.selection)
    });
  },
  methods: {
    draw(selection) {
      d3.select("#view2svg").select("*").remove();
      if (selection == "lollipop plot") {
        // set the dimensions and margins of the graph
        var margin = { top: 10, right: 30, bottom: 40, left: 100 },
          width = 460 - margin.left - margin.right,
          height = 500 - margin.top - margin.bottom;

        // append the svg object to the body of the page
        var svg = d3
          .select("#view2svg")
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr(
            "transform",
            "translate(" + margin.left + "," + margin.top + ")"
          );

        // Parse the Data
        // d3.csv(
        //   "https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/7_OneCatOneNum_header.csv",
        //   function (data) {
        // sort data
        NetService.loadData((resp) => {
        //   console.log(resp);
          let data = resp.data;
        //   console.log(data);
          data.sort(function (b, a) {
            return a.Value - b.Value;
          });

          // Add X axis
          var x = d3.scaleLinear().domain([0, 13000]).range([0, width]);
          svg
            .append("g")
            .attr("transform", "translate(0," + height + ")")
            .call(d3.axisBottom(x))
            .selectAll("text")
            .attr("transform", "translate(-10,0)rotate(-45)")
            .style("text-anchor", "end");

          // Y axis
          var y = d3
            .scaleBand()
            .range([0, height])
            .domain(
              data.map(function (d) {
                return d.Country;
              })
            )
            .padding(1);
          svg.append("g").call(d3.axisLeft(y));

          // Lines
          svg
            .selectAll("myline")
            .data(data)
            .enter()
            .append("line")
            .attr("x1", function (d) {
              return x(d.Value);
            })
            .attr("x2", x(0))
            .attr("y1", function (d) {
              return y(d.Country);
            })
            .attr("y2", function (d) {
              return y(d.Country);
            })
            .attr("stroke", "grey");

          // Circles
          svg
            .selectAll("mycircle")
            .data(data)
            .enter()
            .append("circle")
            .attr("cx", function (d) {
              return x(d.Value);
            })
            .attr("cy", function (d) {
              return y(d.Country);
            })
            .attr("r", "7")
            .style("fill", "#69b3a2")
            .attr("stroke", "black");
        });
      } else if (selection == "word cloud") {
        // List of words
        var myWords = [
          "Hello",
          "Everybody",
          "How",
          "Are",
          "You",
          "Today",
          "It",
          "Is",
          "A",
          "Lovely",
          "Day",
          "I",
          "Love",
          "Coding",
          "In",
          "My",
          "Van",
          "Mate",
        ];

        // set the dimensions and margins of the graph
        var margin = { top: 10, right: 10, bottom: 10, left: 10 },
          width = 450 - margin.left - margin.right,
          height = 450 - margin.top - margin.bottom;

        // append the svg object to the body of the page
        var svg = d3
          .select("#view2svg")
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr(
            "transform",
            "translate(" + margin.left + "," + margin.top + ")"
          );

        // Constructs a new cloud layout instance. It run an algorithm to find the position of words that suits your requirements
        var layout = cloud()
          .size([width, height])
          .words(
            myWords.map(function (d) {
              return { text: d };
            })
          )
          .padding(10)
          .fontSize(60)
          .on("end", draw);
        layout.start();

        // This function takes the output of 'layout' above and draw the words
        // Better not to touch it. To change parameters, play with the 'layout' variable above
        function draw(words) {
          svg
            .append("g")
            .attr(
              "transform",
              "translate(" +
                layout.size()[0] / 2 +
                "," +
                layout.size()[1] / 2 +
                ")"
            )
            .selectAll("text")
            .data(words)
            .enter()
            .append("text")
            .style("font-size", function (d) {
              return d.size + "px";
            })
            .attr("text-anchor", "middle")
            .attr("transform", function (d) {
              return "translate(" + [d.x, d.y] + ")rotate(" + d.rotate + ")";
            })
            .text(function (d) {
              return d.text;
            });
        }
      } else if(selection == "Fluency Chart") {

        const MIN_SCORE = 0;
        const MAX_SCORE = 100;

        // This function generates a random number in the range [l, r)
        function getRandomNumber(l, r) {
          return l + Math.random() * (r - l);
        }

        // This function generates a random int in the range [l, r]
        function getRandomInt(l, r) {
          return Math.floor(getRandomNumber(l, r + 1));
        }

        // This function randomly generates a fluency scores list
        function generateFluencyScoresList() {
          const MAX_N = 10;
          let n = getRandomInt(1, MAX_N);
          let list = [];
          for(let i = 0; i < n; i++) {
            list.push(getRandomNumber(MIN_SCORE, MAX_SCORE));
          }
          return list;
        }

        
        // keeps the fluency scores list
        let fluencyScores = generateFluencyScoresList();

        // set the dimensions and margins of the graph
        var margin = { top: 10, right: 10, bottom: 30, left: 30 },
          width = 450 - margin.left - margin.right,
          height = 450 - margin.top - margin.bottom;
        var padding = {top: 30, right: 20, bottom: 30, left: 20};

        // append the svg object to the body of the page
        var svg = d3
          .select("#view2svg")
          .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
          .append("g")
            .attr(
              "transform",
              "translate(" + margin.left + "," + margin.top + ")"
            );
          
          // add the x Axis
          var x = d3.scaleLinear()
                    .domain([1, fluencyScores.length])
                    .range([padding.left, width - padding.right])
          svg.append("g")
              .attr("transform", "translate(0," + height + ")")
              .call(d3.axisBottom(x).ticks(fluencyScores.length - 1));
          
          // add the y Axis
          var y = d3.scaleLinear()
                    .domain([MIN_SCORE, MAX_SCORE])
                    .range([height - padding.top, padding.bottom]);
          svg.append("g")
              .call(d3.axisLeft(y));
        
          const radius = 6;
          const gapBetweenCircles = 5;

          // Plot the area
          svg.append("path")
              .datum(fluencyScores.map(function(score, index) { return [index + 1, score]; }))
              .attr("fill", "none")
              .attr("opacity", ".8")
              .attr("stroke", "#000")
              .attr("stroke-width", 1)
              .attr("stroke-linejoin", "round")
              .attr("d",  d3.line()
                .curve(d3.curveCatmullRom)
                  .x(function(d) { return x(d[0]); })
                  .y(function(d) { return y(d[1]); })
              );
          
          let fluencyScoresForLines = [
            ...fluencyScores.map(function(score, index) { return [[index + 1, score], [index + 1, score - gapBetweenCircles]] }), 
            ...fluencyScores.map(function(score, index) { return [[index + 1, score], [index + 1, score + gapBetweenCircles]] })
          ];

          svg.append("g")
            .selectAll("line")
            .data(fluencyScoresForLines)
            .enter()
            .append("line")
              .style("stroke", "black")
              .style("stroke-width", 1)
                .attr("x1", function(d) { return x(d[0][0]); })
                .attr("y1", function(d) { return y(d[0][1]); })
                .attr("x2", function(d) { return x(d[1][0]); })
                .attr("y2", function(d) { return y(d[1][1]); });
          
          svg.append("g")
              .selectAll(".in-circle")
              .data(fluencyScores.map(function(score, index) { return [index + 1, score]; }))
              .enter()
              .append("circle")
                .attr("fill", "#EEEEEE")
                .attr("stroke", "black")
                .attr("r", radius)
                .attr("cx", function(d) { return x(d[0]); })
                .attr("cy", function(d) { return y(d[1]); })
                .on("mouseover", function(d,i) {
                  d3.select(this)
                      .attr("r", radius * 2)
                  })
                .on("mouseout", function(d,i) {
                    d3.select(this)
                      .attr("r", radius)
                  })

          svg.append("g")
              .selectAll(".ext-circle")
              .data(fluencyScores.map(function(score, index) { return [index + 1, score - gapBetweenCircles]; }))
              .enter()
              .append("circle")
                .attr("class", "ext-circle")
                .attr("fill", "#C9D9F8")
                .attr("stroke", "black")
                .attr("r", radius)
                .attr("cx", function(d) { return x(d[0]); })
                .attr("cy", function(d) { return y(d[1]); })
                .on("mouseover", function(d,i) {
                  d3.select(this)
                      .attr("r", radius * 2)
                  })
                .on("mouseout", function(d,i) {
                    d3.select(this)
                      .attr("r", radius)
                  });

          svg.append("g")
              .selectAll(".ext-circle")
              .data(fluencyScores.map(function(score, index) { return [index + 1, score + gapBetweenCircles]; }))
              .enter()
              .append("circle")
                .attr("class", "ext-circle")
                .attr("fill", "#ffb6c1")
                .attr("stroke", "black")
                .attr("r", radius)
                .attr("cx", function(d) { return x(d[0]); })
                .attr("cy", function(d) { return y(d[1]); })
                .on("mouseover", function(d,i) {
                  d3.select(this)
                      .attr("r", radius * 2)
                  })
                .on("mouseout", function(d,i) {
                    d3.select(this)
                      .attr("r", radius)
                  })
              
      }
    },
  },
};
</script>


