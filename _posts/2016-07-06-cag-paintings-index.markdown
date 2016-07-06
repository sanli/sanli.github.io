---
layout: post
title:  "中华珍宝馆所有馆藏列表"
date:   2016-07-06 11:30
categories: message
---
<style>
path {
  stroke: #fff;
  /* stroke-width: 1.5; */
  cursor: pointer;
}

text {
  font: 11px sans-serif;
  cursor: pointer;
}

</style>
<script src="//d3js.org/d3.v3.min.js"></script>
<div id="cagd3" class="col-md-12"></div>
<div id="info" class="col-md-12"></div>
<script>

var width = 760,
    height = 700,
    radius = (Math.min(width, height) / 2) - 10;

var formatNumber = d3.format(",d");

var x = d3.scale.linear()
    .range([0, 2 * Math.PI]);

// var y = d3.scale.sqrt()
//     .range([0, radius]);
var y = d3.scale.pow().exponent(1.3).domain([0, 1]).range([0, radius])

var color = d3.scale.category20c();

var partition = d3.layout.partition()
    .value(function(d) { return d.size; });

var a = function(t) {
    return .299 * t.r + .587 * t.g + .114 * t.b
}

var padding = 5;
var showlabelx = 0.01;

var arc = d3.svg.arc()
    .startAngle(function(d) { return Math.max(0, Math.min(2 * Math.PI, x(d.x))); })
    .endAngle(function(d) { return Math.max(0, Math.min(2 * Math.PI, x(d.x + d.dx))); })
    .innerRadius(function(d) { return Math.max(0, y(d.y)); })
    .outerRadius(function(d) { return Math.max(0, y(d.y + d.dy)); });

var svg = d3.select("#cagd3").append("svg")
    .attr("width", width)
    .attr("height", height)
  	.append("g")
    .attr("transform", "translate(" + width / 2 + "," + (height / 2) + ")");
var info = d3.select("#info");
var pathy, texty ;


d3.json("http://ltfc.net/cagstore/outline_d3.json", function(error, root) {
  if (error) throw error;

  svg.selectAll("path")
      .data(partition.nodes(root))
      .enter().append("path")
      .attr("d", arc)
      .style("fill", function(d) { return color((d.children ? d : d.parent).name); })
      .on("click", click)
      .on("mouseover", mouseover)
      .append("title")
      .text(function(d) { return d.name + "\n" + formatNumber(d.value); });

   var texty = svg.selectAll("text")
   	  .data(partition.nodes(root))
      .enter().append("text")
      .style("fill-opacity", 1).style("fill", function(d) {
      	  return colorvalue( d3.rgb( color((d.children ? d : d.parent).name)) ) < 125 ? "#eee" : "#000" ;
      })
      .style("visibility", function(e) {
      	return e.dx < showlabelx ? 'hidden' : null;
	  })
      .attr("text-anchor", function(t) {
          return x(t.x + t.dx / 2) > Math.PI ? "end" : "start"
	  })
	  .attr("dy", ".2em").attr("transform", function(t) {
	        var n = (t.name || "").split(" ").length > 1
	          , e = 180 * x(t.x + t.dx / 2) / Math.PI - 90
	          , r = e + (n ? -.5 : 0);
	        return "rotate(" + r + ")translate(" + (y(t.y) + padding) + ")rotate(" + (e > 90 ? -180 : 0) + ")"
	    })
	  .on("click", click)
	  .on("mouseover", mouseover);
	texty.append("tspan").attr("x", 0).text(function(t) {
        return t.depth ? t.name.split(" ")[0] : ""
    }),
    texty.append("tspan").attr("x", 0).attr("dy", "1em").text(function(t) {
        return t.depth ? t.name.split(" ")[1] || "" : ""
    })
});

function colorvalue(color) {
    return .299 * color.r + .587 * color.g + .114 * color.b
}

// 递归遍历子节点
function isSubnode(n, e) {
    return n === e ?
    	!0 : n.children ?
    		n.children.some(function(n) {
		        return isSubnode(n, e)
		    }) : !1 ;
}

function mouseover(d){
	var label = d.name ;
  var age = d.parent ? d.parent.name : '';
	var painting = d.paintings
    ? d.paintings.map( paint =>
      `<a href='http://ltfc.net/outline/${age}/${label}/${paint}'>${paint}</a>`).join('</br>')
    : '';
	info.html(`<h1>${label}</h1>${painting}`);
}

function click(d) {
  svg.transition()
      .duration(750)
      .tween("scale", function() {
        var xd = d3.interpolate(x.domain(), [d.x, d.x + d.dx]),
            yd = d3.interpolate(y.domain(), [d.y, 1]),
            yr = d3.interpolate(y.range(), [d.y ? 20 : 0, radius]);
        return function(t) { x.domain(xd(t)); y.domain(yd(t)).range(yr(t)); };
      })
    .selectAll("path")
    .attrTween("d", function(d) {
    	return function() { return arc(d); };
    });

   svg.selectAll("text")
    .style("visibility", function(e) {
    	var wheelx = e.dx / d.dx;
       	return isSubnode(d, e)
       		? wheelx < showlabelx ? 'hidden' : null
       		: d3.select(this).style("visibility");
    })
    .transition()
    .duration(750)
    .attrTween("text-anchor", function(t) {
        return function() {
            return x(t.x + t.dx / 2) > Math.PI ? "end" : "start"
        }
    }).attrTween("transform", function(t) {
        var n = (t.name || "").split(" ").length > 1;
        return function() {
            var e = 180 * x(t.x + t.dx / 2) / Math.PI - 90
              , r = e + (n ? -.5 : 0);
            return "rotate(" + r + ")translate(" + (y(t.y) + padding) + ")rotate(" + (e > 90 ? -180 : 0) + ")"
        }
    }).style("fill-opacity", function(e) {
        return isSubnode(d, e) ? 1 : 1e-6
    }).each("end", function(e) {
        d3.select(this).style("visibility", function(e) {
        	var wheelx = e.dx / d.dx;
	        return isSubnode(d, e)
	        	? wheelx < showlabelx ? 'hidden' : null
	        	: d3.select(this).style("visibility");
	    });
    });

}
d3.select(self.frameElement).style("height", height + "px");

</script>
