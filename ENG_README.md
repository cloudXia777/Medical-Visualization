# README (ENG_VERSION)
Porject of Hagen University
***
**OUTLINE**
* **Requirements**
* **Scheme**
* **Details**
***

## 1. Requirements
**Interaction with Medical Recommender Systems, shows the results of recommendation by graph**

#### 1.1 Level layout   
Three levels, i,e. disese-level, symptom-level and othe-level.
    
#### 1.2 Weight visualzation 
Weight visualization reflects the tightness of relationships between different nodes.

#### 1.3 Outstanding concentrated area (Fisheye)    
Fisheye outstands the area where user is concentrating on, and enables user to see the details clearly。
    
#### 1.4 Infromation visualization with table   
Showing more information with form of table by right clicking nodes.

#### 1.5 Node information expansion        
By clicking on the node, add the nodes and edges associated with it, and place them in the center area, and the other nodes slide to both sides.
    
#### 1.6 Zoom
Scale the interface with the mouse wheel.

## 2. Scheme

#### **2.1 Overall framework (Django+Neo4j+D3)**  
![image](https://github.com/cloudXia777/Medical-Visualization/blob/master/image/%E6%80%BB%E4%BD%93%E6%A1%86%E6%9E%B6%E5%9B%BE.png)  

#### **2.2 Implementation design**
1. [Django](https://www.djangoproject.com/)as web application development framework.
2. Apply [D3.force](https://d3js.org/) to achieve knowledge graph visualization.
3. Implement data interaction between JS and backend using AJAX.
4. Apply [Neomodel](https://neomodel.readthedocs.io/en/latest/) to achieve data interaction between Django and [Neo4j](https://neo4j.com/) graph database.  

## 3、Implementation details
#### **3.1 Development environment**    
* Django    
--Django 1.11   
--python 3.6    
--neomodel 3.3.2
* D3    
--D3.V3
* neo4j  
--3.2.14 Community Version  
* jQuery    
--3.2.1 

#### **3.2 Development modules**
**3.2.1 Front-end Design**  
1.Head（LOGO、search）  
2.Main（Area of nodes and edges）   
3.Footer（tagging）

**3.2.2 Level layout**  
Forcibly define the value of each node in the y axis; according to the category of the node, specify the position in the y axis thereof
```
nodes.forEach(function (o, i) {
/* height: the helight of Main
   100: the distance between two level, set it by adjusting requirements*/
    if (o.group === 1) {
        o.y = height / 2;
    } else if (o.group === 2) {
        o.y = height / 2 + 100;
    } else {
        o.y = height / 2 - 100;
    }
});
```
*Note: Using a forced level will cause the connection between nodes to be disobeyed from the initial length setting.*     

**3.2.3 Weight Visualization**    
    
```
/* Setting the stroke-width attribute*/
.att("stroke-width", function(d){
    return Math.sqrt(d.value);
})
```

**3.2.3 Drag behaviour**  

```
/*Setting drag-start, drag-on, drag-end*/
var drag = force.drag()
    .on("dragstart", function(d){
        d3.event.sourceEvent.stopPropagation();
        d.fixed = false; //拖曳对象固定
    })
    .on("dragend", function (d,i) {
        d3.select(this).style("fill", color[d.group-1]); //变为原来的颜色
    })
    .on("drag", function (d) {
        d3.select(this).style("fill", "yellow"); //拖曳过程中为黄色
        d3.select(this).attr("cx", d.x = d3.event.x)
                        .attr("cy", d.y = d3.event.y);
    });
```

**3.2.4 Fisheye distortion**  

```
/*cite [1] sample*/
var fisheye = d3.fisheye.circular()
    .radius(300)
    .distortion(2);
svg.on("mousemove", function () {
    fisheye.focus(d3.mouse(this));
    circles.each(function (d) {d.fisheye = fisheye(d);})
        .attr("cx", function (d) {return d.fisheye.x;})
        .attr("cy", function (d) {return d.fisheye.y;})
        .attr("r", function (d) {return d.fisheye.z * 14;});
    lines.attr("x1", function (d) {return d.source.fisheye.x;})
        .attr("y1", function (d) {return d.source.fisheye.y;})
        .attr("x2", function (d) {return d.target.fisheye.x;})
        .attr("y2", function (d) {return d.target.fisheye.y;});
    texts.each(function (d) {d.fisheye = fisheye(d);})
        .attr("x", function (d) { return d.fisheye.x; })
        .attr("y", function (d) { return d.fisheye.y; })
        .style("font-size", function (d) {
            return  '15px'
        })
});
<script language="javascript" type = "text/javascript" src="{% static 'js/fisheye.js' %}"></script>
```

**3.2.5 Setting nodes center**

```
/*1. Compute distance from click-node to center of screen
  2. Find the connected graph where the click node is located
  3. All nodes in the connected graph move the same distance*/

var pos;
nodes.forEach(function (d,i) {
    if (d.id === linkDis){
        pos = d.x;
        dist = width / 2 - d.x;
    }
});

function setCenter(){
    circles.transition().ease('linear').duration(animationStep)
        .attr('cx', function(d) {
            if (n2N(ClickName, d.id)) {
                return d.x + dist;
            }
            else
                return d.x;
        })
        .attr('cy', function(d) { return d.y; });

    lines.transition().ease('linear').duration(animationStep)
        .attr('x1', function(d) {
            if (n2N(ClickName, d.source.id)) {
                return d.source.x + dist;
            }
            else {
                return d.source.x;
            }
        })
        .attr('y1', function(d) { return d.source.y; })
        .attr('x2', function(d) {
            if (n2N(ClickName, d.target.id))
                return d.target.x + dist;
            else
                return d.target.x;
        })
        .attr('y2', function(d) { return d.target.y ; });

    texts.transition().ease('linear').duration(animationStep)
        .attr('x', function(d) {
             if (n2N(ClickName, d.id))
                return d.x + dist;
             else
                return d.x;
        })
        .attr('y', function(d) { return d.y; });
}
```

**3.2.6 Pop up**  
```
/*1. Judge the location of the remaining nodes
  2. Move*/
// in the right or left of click-node
nodes.forEach(function (d, i) {
    if (! n2N(ClickName, d.id)){
        if (d.x < pos)
            direct[i] = 'left';
        else
            direct[i] = 'right';
    }
});
var k = 10 * e.alpha; 
nodes.forEach(function (d, i) {
    if (! n2N(ClickName, d.id)){
        if (direct[i] === 'left')
            d.x -= 4 * k;
        if (direct[i] === 'right')
            d.x += 4 * k;
    }
    if (d.group === 1) {
        d.y = height / 2;
    } else if (d.group === 2) {
        d.y = height / 2 + 100;
    } else {
        d.y = height / 2 - 100;
    }
});
```
**3.2.7 Collision detion**  
```
function tick(e){
    var q = d3.geom.quadtree(nodes),
    i = 0,
    n = nodes.length;
    while (++i < n) q.visit(d3.collide(nodes[i]));
}
<script src="{% static 'js/collide.js' %}"></script>
```
**3.2.8 Table**  
```
/*Right click: crete table*/
function rclick(d,e) {
    d3.event.preventDefault(e);

    var childTable = document.getElementById("tableShow");
    //console.log(childTable);
    if (childTable){
        var parentDiv = document.getElementById("tabId");
        //console.log(parentDiv);
        parentDiv.removeChild(childTable);
    }
    var table = d3.select(".tabTopLeft").append("table").attr("id", "tableShow").attr("draggable", true);
    var header = table.append("thead").append("tr");
    header.selectAll("th")
        .data(["Disease", "Flu"])
        .enter()
        .append("th")
        .text(function(d) {return d;});
    var tablebody = table.append("tbody");
    // 
    var clickResult = [["Full_name", "Influenza"], ["Chi_name", "流感"], ["Season", "winter"]];
    rows = tablebody.selectAll("tr")
        .data(clickResult)
        .enter()
        .append("tr");
    cells = rows.selectAll("td")
        .data(function(d) {return d;})
        .enter()
        .append("td")
        .text(function(d){return d;});
}
```
#### **3.3 References**
[1] https://github.com/d3/d3-plugins/tree/master/fisheye    
[2] https://observablehq.com/@d3/force-directed-graph?collection=@d3/d3-force   
[3] https://bl.ocks.org/mbostock    
[4] https://gist.github.com/pkerpedjiev/0389e39fad95e1cf29ce    
[5] https://bl.ocks.org/mbostock/1093130    
[6] http://bl.ocks.org/sgruhier/1d692762f8328a2c9957    
[7] https://www.npmjs.com/package/d3-context-menu   
[8] http://visualdataweb.de/webvowl/#   
## 4、Future work
* Front-end page adaption
* After selecting individual diseases, the remaining unselected nodes disappear