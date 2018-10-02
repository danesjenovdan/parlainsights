<template>
  <div id="viz-container">
    <div id="viz">
      <svg
        :width="width + margin.left + margin.right"
        :height="height + margin.top + margin.bottom"
        id="viz-svg"
      >
        <g class="chart" v-bind:transform="`translate(${margin.left}px, ${margin.top}px)`"></g>
        <rect
          :width="width"
          :height="height"
          v-bind:transform="`translate(${-margin.left}px, ${-margin.top}px)`"
          fill="#1e3343"
        ></rect>
      </svg>
    </div>
    <div class="buttons">
      <button @click="start()">start</button>
      <button @click="stalni()">stalni</button>
      <button @click="spoli()">spoli</button>
      <button @click="starost()">starost</button>
      <button @click="izobrazba()">izobrazba</button>
      <button @click="stranke()">stranke</button>
    </div>
  </div>
</template>

<script>
/* eslint no-param-reassign: 0 */

import * as d3 from 'd3v4';

export default {
  name: 'Visualisation',
  props: {
    selectedMPs: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      width: 0,
      height: 0,
      margin: { top: 0, right: 0, bottom: 0, left: 0 },
      radius: 8,
      defaultColor: '#ffffff',
      highlightColor: '#009cdd',
      updateDuration: 1500,
      activeForces: [],
      counter: 0,
      barYactive: false,
      barYaxisDOM: null,

      svg: null,

      randomX: null,
      randomY: null,
      simulation: null,
      circled: null,

      data: null,
      triggerState: null,

      circles: null,
    };
  },

  watch: {
    selectedMPs(newMPs) {
      console.log(newMPs);
      this.updateNodes();
    },
  },

  mounted() {
    this.width = document.getElementById('viz').clientWidth;
    this.height = document.getElementById('viz').clientHeight;
    this.margin = {
      top: this.height * 0.1,
      right: this.width * 0.08,
      bottom: this.height * 0.06,
      left: this.width * 0.08,
    };
    this.randomX = d3.scaleLinear().domain([0, 1]).range([0, this.width - this.radius]);
    this.randomY = d3.scaleLinear().domain([0, 1]).range([0, this.height - this.radius]);
    this.simulation = d3.forceSimulation();

    this.tooltipdiv = d3.select('#viz').append('div')
      .attr('class', 'tooltip');

    d3.csv('https://docs.google.com/spreadsheets/d/e/2PACX-1vTi7tYdUb3pKE8f5csL4sKw4Nk0o_-apnsjultSyQc_reQ7a87QFxOxpkOq4oEvjyB3bBNzyVoXm_We/pub?gid=578366954&single=true&output=csv', (error, data) => {
      console.log('d3 csv returned');
      if (error) throw error;

      this.data = data;

      this.data.forEach((d) => {
        d.fade = 1;
        d.color = this.defaultColor;
        d.starost = this.getAge(d.birthday);
        d.izobrazba = this.encodeEducation(d.izobrazba);
        d.kratica = this.encodeParty(d.kratica_stranke);
      });

      this.simulation.nodes(this.data).on('tick', this.ticked);
      this.initSim();

      this.$nextTick(() => {
        const tempsvg = d3.select('#viz-svg');

        this.circles = tempsvg.append('g')
          .attr('class', 'circles')
          .selectAll('circle')
          .data(this.data)
          .enter()
          .append('circle')
          .attr('r', this.radius)
          .attr('opacity', 0.2)
          .attr('cx', d => d.x)
          .attr('cy', d => d.y)
          .attr('fill', (d) => {
            d.color = this.defaultColor;
            return d.color;
          })
          .on('mouseover', (d) => { // setup tooltip
            this.tooltipdiv.transition()
              .duration(200)
              .style('opacity', 0.9);
            // console.log($(this).parents('#kompas-scatter')));
            this.tooltipdiv.html(`<p>${d.name}</p><p>${d.kratica_stranke}</p>`)
              .style('left', `${((d3.event.pageX - (this.tooltipdiv.node().getBoundingClientRect().width / 2)) + 10)}px`)
              .style('top', `${(d3.event.pageY - 30)}px`);
          })
          .on('mouseout', () => {
            this.tooltipdiv.transition()
              .duration(200)
              .style('opacity', 0);
          });

        this.start();
      });
    });
  },

  methods: {
    getAge(dateString) {
      const today = new Date();
      const birthDate = new Date(dateString);
      let age = today.getFullYear() - birthDate.getFullYear();
      const m = today.getMonth() - birthDate.getMonth();
      if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
        age -= 1;
      }
      return age;
    },
    ticked() {
      this.circles
        .attr('cx', d => d.x)
        .attr('cy', d => d.y);
    },

    initSim() {
      this.simulation
        .force('x', d3.forceX(() => this.randomX(Math.random())))
        .force('y', d3.forceY(() => this.randomY(Math.random())))
        .alphaMin(0.5)
        .stop();
      // loop through simulation ticks to land in a stable state with randomly distributed nodes
      for (let i = 0; i < 100; i += 1) this.simulation.tick();

      this.activeForces.push('x', 'y');
    },

    isolate(force, filter) {
      const initialize = force.initialize;
      force.initialize = () => {
        initialize.call(force, this.data.filter(filter));
      };
      return force;
    },

    encodeEducation(education) {
      switch (education) {
        case '4':
          return 1;
        case '5':
          return 2;
        case '6/1':
          return 3;
        case '6/2':
          return 4;
        case '7':
          return 5;
        case '8/1':
          return 6;
        case '8/2':
          return 7;
        default:
          return -1;
      }
    },

    encodeParty(party) {
      switch (party) {
        case 'SDS':
          return 1;
        case 'DeSUS':
          return 2;
        case 'IMNS':
          return 3;
        case 'Levica':
          return 4;
        case 'NeP - AČ':
          return 5;
        case 'NeP - BK':
          return 5;
        case 'NeP - JV':
          return 5;
        case 'NeP - MH':
          return 5;
        case 'NeP - ZL':
          return 5;
        case 'NSI':
          return 6;
        case 'PS NP':
          return 7;
        case 'SD':
          return 8;
        case 'SMC':
          return 9;
        default:
          return -1;
      }
    },

    breakdown(breakdownObjects) {
      /* breaks down nodes into groups centers of gravity defined by x and y functions */
      // first remove forces
      this.removeForces();

      for (let i = 0; i < breakdownObjects.length; i += 1) {
        switch (breakdownObjects[i].type) {
          case 'xy': {
            const customX = d3
              .forceX(breakdownObjects[i].breakX)
              .strength(breakdownObjects[i].strengthX);
            const customY = d3
              .forceY(breakdownObjects[i].breakY)
              .strength(breakdownObjects[i].strengthY);

            this.simulation
              .force(`x${i.toString()}`, customX)
              .force(`y${i.toString()}`, customY);
            this.activeForces.push(`x${i.toString()}`);
            this.activeForces.push(`y${i.toString()}`);

            break;
          }

          case 'charge': {
            const customCharge = this.isolate(d3.forceManyBody(), breakdownObjects[i].filter)
              .strength(breakdownObjects[i].strength);
            this.simulation
              .force(`charge${i.toString()}`, customCharge);
            this.activeForces.push(`charge${i.toString()}`);

            break;
          }

          default:
            break;
        }
      }

      this.simulation
        .force('collide', d3.forceCollide(this.radius))
        .alpha(0.5)
        .alphaMin(0.01)
        .restart();

      this.activeForces.push('collide');
    },

    barchartY(filterFunction, accessor) {
      /* creates a bar chart on the left hand side from values (not a force function!) */
      this.removeForces();

      const barY = d3.scaleLinear()
        .domain(d3.extent(this.data.filter(filterFunction), d => +accessor(d)))
        .range([this.height, 0]);

      const barYaxis = d3.axisLeft(barY);

      const xValObj = {};

      this.simulation.stop();

      let radiusMulti = 0;
      if (this.width > this.height) {
        radiusMulti = 10;
      } else {
        radiusMulti = 5;
      }
      this.circles
        .transition()
        .duration(this.duration)
        .attr('opacity', d => d.fade)
        .attr('fill', d => d.color)
        .attr('cy', (d) => {
          const y = barY(+accessor(d));
          // manually update d.x to let simulation know x has changed
          d.y = y;
          return d.y;
        })
        .attr('cx', (d) => {
          if (xValObj[accessor(d)]) {
            const x = ((xValObj[accessor(d)] += 1) * (this.radius * radiusMulti));
            // manually update d.x to let simulation know x has changed
            d.x = x;
            return x;
          }
          const x = ((xValObj[accessor(d)] = 1) * (this.radius * radiusMulti));
          d.x = x;
          return x;
        });

      const tempsvg = d3.select('#viz-svg');
      this.barYaxisDOM = tempsvg.append('g')
        .attr('class', 'y-axis')
        .call(barYaxis);

      this.barYactive = true;
    },

    splitGenders() {
      const xFunction = (d) => {
        const boyBool = d.spol === 'm';
        return boyBool ? this.width / 3 : (this.width / 3) * 2;
      };
      const yFunction = () => this.height / 2;

      const breakdownObjs = [
        {
          type: 'xy',
          breakX: xFunction,
          strengthX: 0.15,
          breakY: yFunction,
          strengthY: 0.15,
        }, {
          type: 'charge',
          filter: d => d,
          strength: -1.5,
        },
      ];

      this.breakdown(breakdownObjs);
    },
    splitAge() {
      const filterFunction = d => d.starost !== null;
      const accessor = d => d.starost;
      this.barchartY(filterFunction, accessor);
    },
    splitEducation() {
      const xFunction = d => (d.izobrazba !== null ? (this.width / 2) : d.x);
      const yFunction = (d) => {
        const yScale = d3.scaleLinear()
          .domain(d3.extent(this.data, p => p.izobrazba))
          .range([this.height - 75, 0 + 75]);
        return d.izobrazba != null ? yScale(d.izobrazba) : d.y;
      };

      const xyBreakdown = {
        type: 'xy',
        breakX: xFunction,
        filterX: d => d,
        strengthX: 0.24,
        breakY: yFunction,
        filterY: d => d,
        strengthY: 0.18,
      };

      const chargeBreakdown = {
        type: 'charge',
        filter: d => d.izobrazba !== null,
        strength: -1.5,
      };

      const chargeBreakdownNull = {
        type: 'charge',
        filter: d => d.izobrazba === null,
        strength: -1.5,
      };

      return this.breakdown([xyBreakdown, chargeBreakdown, chargeBreakdownNull]);
    },
    splitDecade() {
      const xFunction = d => (d.decade !== null ? (this.width / 2) : d.x);
      const yFunction = (d) => {
        const yScale = d3.scaleLinear()
          .domain(d3.extent(this.data, (p => (p.decade / 10) - 3)))
          .range([this.height - 75, 0 + 75]);
        return d.izobrazba != null ? yScale((d.decade / 10) - 3) : d.y;
      };

      const xyBreakdown = {
        type: 'xy',
        breakX: xFunction,
        filterX: d => d,
        strengthX: 0.24,
        breakY: yFunction,
        filterY: d => d,
        strengthY: 0.18,
      };

      const chargeBreakdown = {
        type: 'charge',
        filter: d => d.izobrazba !== null,
        strength: -1.5,
      };

      const chargeBreakdownNull = {
        type: 'charge',
        filter: d => d.izobrazba === null,
        strength: -1.5,
      };

      return this.breakdown([xyBreakdown, chargeBreakdown, chargeBreakdownNull]);
    },
    splitParty() {
      const xFunction = d => (d.kratica !== null ? (this.width / 2) : d.x);
      const yFunction = (d) => {
        const yScale = d3.scaleLinear()
          .domain(d3.extent(this.data, p => p.kratica))
          .range([this.height - 75, 0 + 75]);
        return d.kratica != null ? yScale(d.kratica) : d.y;
      };

      const xyBreakdown = {
        type: 'xy',
        breakX: xFunction,
        filterX: d => d,
        strengthX: 0.24,
        breakY: yFunction,
        filterY: d => d,
        strengthY: 0.18,
      };

      const chargeBreakdown = {
        type: 'charge',
        filter: d => d.kratica !== null,
        strength: -1.5,
      };

      const chargeBreakdownNull = {
        type: 'charge',
        filter: d => d.kratica === null,
        strength: -1.5,
      };

      return this.breakdown([xyBreakdown, chargeBreakdown, chargeBreakdownNull]);
    },

    removeForces() {
      if (this.barYactive) {
        this.barYaxisDOM.remove();
        this.barYactive = false;
      }

      for (let i = 0; i < this.activeForces.length; i += 1) {
        this.simulation
          .force(this.activeForces[i], null);
      }

      this.activeForces = [];
      this.simulation
        .on('end', null);
    },

    baseSim(alphaMin) {
      /* creates basic petri dish of nodes */
      this.removeForces();

      this.simulation
        .stop()
        .force('charge', d3.forceManyBody().strength(-2))
        .force('x', d3.forceX(this.width / 2).strength(0.12))
        .force('y', d3.forceY(this.height / 2).strength(0.12))
        .force('collide', d3.forceCollide(this.radius))
        .alphaMin(alphaMin)
        .alpha(1)
        .restart();
      this.activeForces.push('charge', 'x', 'y', 'collide');
    },

    highlight(filterFunction) {
      this.data.forEach((d) => {
        d.color = filterFunction(d) ? this.highlightColor : this.defaultColor;
      });
    },

    fade(filterFunction) {
      this.data.forEach((d) => {
        d.fade = filterFunction(d) ? 0 : 1;
      });
    },

    updateNodes() {
      console.log(this.circles);
      this.circles
        .transition()
        .duration(this.updateDuration)
        .attr('opacity', d => d.fade)
        .attr('fill', (d) => {
          if (this.selectedMPs.indexOf(parseInt(d.id, 10)) === -1) {
            console.log(d);
            return d.color;
          } else  {
            return '#ff5e41';
          }
        });
    },

    start() {
      this.highlight(() => false);
      this.fade(() => false);

      if (this.triggerState !== 'start') {
        this.baseSim(0.001);
        this.updateNodes();
        this.triggerState = 'start';
      }
    },
    stalni() {
      this.highlight(d => d.constant === 'TRUE');
      this.fade(() => false);

      if (this.triggerState !== 'constant') {
        this.triggerState = 'constant';
      }
      this.updateNodes();
    },
    spoli() {
      this.highlight(d => d.spol === 'f');
      this.fade(() => false);

      if (this.triggerState !== 'genders') {
        this.splitGenders();
        this.triggerState = 'genders';
      }
      this.updateNodes();
    },
    starost() {
      this.highlight(() => false);
      this.fade(() => false);

      // if (this.triggerState !== 'starost') {
      //   // this.splitDecade();
      //   this.splitAge();
      //   this.triggerState = 'starost';
      // }
      // this.updateNodes();

      if (this.triggerState !== 'starost') {
        this.updateNodes();
        this.baseSim(0.1); // .on('end', function() {
        this.splitAge();
        // });
      } else {
        this.splitAge();
      }
      this.triggerState = 'starost';
    },
    izobrazba() {
      this.highlight(() => false);
      this.fade(() => false);

      if (this.triggerState !== 'izobrazba') {
        this.splitEducation();
        this.triggerState = 'izobrazba';
      }
      this.updateNodes();
    },
    stranke() {
      this.highlight(d => d.kratica_stranke.indexOf('NeP') !== -1);
      this.fade(() => false);

      if (this.triggerState !== 'stranke') {
        this.splitParty();
        this.triggerState = 'stranke';
      }
      this.updateNodes();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
#viz-container {
  width: 100%;
  // height: 100%;
  // overflow: hidden;
  // position: relative;
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  display: block;
}
#viz {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  display: block;
  z-index: 1;
}
.y-axis line {
  stroke: white;
}
.y-axis path {
  stroke: white;
}
.y-axis text {
  fill: white;
}
.buttons {
  position: fixed;
  bottom: 40px;
  width: 100%;
  z-index: 200;
}
</style>
<style lang="scss">
.tooltip {
  position: absolute;
  text-align: center;
  border: 0px;
  pointer-events: none;
  background-color: #525252;
  border-radius: 3px;
  padding: 2px 10px;
  opacity: 0;

  color: #ffffff;
}
</style>
