<template>
    <div class="card" :id="myChartID">
        <svg></svg>
    </div>
</template>
<script>
import * as d3 from "d3";
//import { setEnvironmentData } from "worker_threads";
//import {drawsunburst} from "./drawsunburst"
export default{
    name : 'SunBurst',
    data(){
        return{
            year : 2020
        }
    },
    props:{
        mysundata : Object,
        myChartID : String
    },
    created(){
        //console.log("data passed:")
        //console.log(this.mysundata)
    },
    mounted(){
        //console.log("Chart ID:", this.myChartID)
        let sundata = this.dataProcess(this.mysundata)
        //console.log("Data Passed down as a Prop  ", sundata)
        this.drawChart(sundata, this.myChartID)
    },
    watch:{
        mysundata(newval, oldval){
            //console.log("New sundata", newval)
            let sundata = this.dataProcess(newval)
            //console.log("Data Passed down as a Prop  ", sundata)
            this.drawsunburst( sundata, this.myChartID)
        }
    },
    methods:{
        givefather(data){
            this.$emit("givefather", data);
            //console.log("Send data to father:", data)
        },
        drawChart(data, id){
            //console.log("Drawing Chart!")
            this.drawsunburst(data, id);
        },
        dataProcess(data){
            let formattedData = {
            name : "alltitles",
            children : [],
            }            
            Object.keys(data).forEach(item =>{
                let thistype = {
                    name: item,
                    children : []
                }
                Object.keys(data[item]).forEach(item2 =>{
                    //console.log(item2)
                    let thistype2={
                        name: item2,
                        children : []
                    }
                    Object.keys(data[item][item2]).forEach(item3 =>{
                        //console.log(item3)
                        //data[item][item2][item3].value = 1;
                        //thistype2.children.push(data[item][item2][item3])
                        let thistype3={
                            name : item3,
                            children : []
                        }
                        Object.keys(data[item][item2][item3]).forEach(item4 =>{
                            data[item][item2][item3][item4].value = 1;
                            thistype3.children.push(data[item][item2][item3][item4]);
                        })
                        thistype2.children.push(thistype3);
                    })
                    thistype.children.push(thistype2);
                })
                formattedData.children.push(thistype);
            })
            //alldataprocessed.children.push(formattedData);
            //console.log("Sunburst Data Processed", formattedData)
            return formattedData;
        },
        drawsunburst(data, id) {
            //console.log("Data have children:", data.children)
            let vueThis = this;
            id = '#'+id            
            let color = d3.scaleOrdinal(d3.quantize(d3.interpolateRainbow, data.children.length + 1));
            let format = d3.format(",d");
            let width = 600;
            let radius = width / 6;

            const arc = d3.arc()
            .startAngle(d => d.x0)
            .endAngle(d => d.x1)
            .padAngle(d => Math.min((d.x1 - d.x0) / 2, 0.005))
            .padRadius(radius * 1.5)
            .innerRadius(d => d.y0 * radius)
            .outerRadius(d => Math.max(d.y0 * radius, d.y1 * radius - 1));

            const partition = data => {
                                        const root = d3.hierarchy(data)
                                            .sum(d => d.value)
                                            .sort((a, b) => b.value - a.value);
                                        return d3.partition()
                                            .size([2 * Math.PI, root.height + 1])
                                            (root);
                                        }
            const root = partition(data);
            //console.log("Root is", root);
            root.each(d => d.current = d);

            d3.select(id).select("svg").remove();
            
            const svg = d3.select(id).append("svg")
                .attr("viewBox", [0, 0, width, width])
                .style("font", "9px sans-serif");

            const g = svg.append("g")
                .attr("transform", `translate(${width / 2},${width / 2})`);
        
            const path = g.append("g")
            .selectAll("path")
            .data(root.descendants().slice(1))
            .join("path")
                .attr("fill", d => { while (d.depth > 1) d = d.parent; return color(d.data.name); })
                .attr("fill-opacity", d => arcVisible(d.current) ? (d.children ? 0.7 : 0.5) : 0)
                .attr("pointer-events", d => arcVisible(d.current) ? "auto" : "none")
        
                .attr("d", d => arc(d.current));
        
            path//.filter(d => d.children)
                .style("cursor", "pointer")
                .on("click", clicked);
        
            path.append("title")
                .text(d => `${d.ancestors().map(d => d.data.name).reverse().join("/")}\n${format(d.value)}`);
        
            const label = g.append("g")
                .attr("pointer-events", "none")
                .attr("text-anchor", "middle")
                .style("user-select", "none")
            .selectAll("text")
            .data(root.descendants().slice(1))
            .join("text")
                .attr("dy", "0.35em")
                .attr("fill-opacity", d => +labelVisible(d.current))
                .attr("transform", d => labelTransform(d.current))
                .text(d => d.data.name + ':'+d.value);
            
            
            label.selectAll("text")
                .on("mouseover", function(d) {
                        d3.select(this)
                        .transition()
                        .duration(100)
                        .attr("font-size", "14px")
                    })
                .on("mouseout", function(d) {
                        d3.select(this)
                        .transition()
                        .duration(100)
                        .attr("font-size", "9px")
                    })
                .on("click", clicked);
        
            const parent = g.append("circle")
                .datum(root)
                .attr("r", radius)
                .attr("fill", "none")
                .attr("pointer-events", "all")
                .on("click", clicked);
        
            function clicked(event, p) {
                //console.log("clicked on:", p);
                if (p.children == null) {
                    vueThis.givefather(p.data.id)
                    return
                }
                parent.datum(p.parent || root);
            
                root.each(d => d.target = {
                    x0: Math.max(0, Math.min(1, (d.x0 - p.x0) / (p.x1 - p.x0))) * 2 * Math.PI,
                    x1: Math.max(0, Math.min(1, (d.x1 - p.x0) / (p.x1 - p.x0))) * 2 * Math.PI,
                    y0: Math.max(0, d.y0 - p.depth),
                    y1: Math.max(0, d.y1 - p.depth)
                });
            
                const t = g.transition().duration(750);
            
                // Transition the data on all arcs, even the ones that aren’t visible,
                // so that if this transition is interrupted, entering arcs will start
                // the next transition from the desired position.
                path.transition(t)
                    .tween("data", d => {
                        const i = d3.interpolate(d.current, d.target);
                        return t => d.current = i(t);
                    })
                    .filter(function(d) {
                    return +this.getAttribute("fill-opacity") || arcVisible(d.target);
                    })
                    .attr("fill-opacity", d => arcVisible(d.target) ? (d.children ? 0.7 : 0.5) : 0)
                    .attr("pointer-events", d => arcVisible(d.target) ? "auto" : "none") 
            
                    .attrTween("d", d => () => arc(d.current));
            
                label.filter(function(d) {
                    return +this.getAttribute("fill-opacity") || labelVisible(d.target);
                    }).transition(t)
                    .attr("fill-opacity", d => +labelVisible(d.target))
                    .attrTween("transform", d => () => labelTransform(d.current));
            }
            
            function arcVisible(d) {
            return d.y1 <= 3 && d.y0 >= 1 && d.x1 > d.x0;
            }
        
            function labelVisible(d) {
            return d.y1 <= 3 && d.y0 >= 1 && (d.y1 - d.y0) * (d.x1 - d.x0) > 0.03;
            }
        
            function labelTransform(d) {
            const x = (d.x0 + d.x1) / 2 * 180 / Math.PI;
            const y = (d.y0 + d.y1) / 2 * radius;
            return `rotate(${x - 90}) translate(${y},0) rotate(${x < 180 ? 0 : 180})`;
            }
        
            return svg.node();
        }
        
    }
}

</script>
<style scoped>
.card{
    width: 90%;
    height: 90%;
}
/*.pieChartContainer {
    background-color: white;
    width: 100%;
    height: 100%;
}*/
</style>