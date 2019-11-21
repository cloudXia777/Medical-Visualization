# 说明文档（中文版）
Project of Hagen University
***

## 1、项目需求
**基于医学推荐系统的的交互可视化。以图的形式展示推荐结果，并支持交互操作。**

<<<<<<< HEAD
#### 1.1 分类层级显示    
    疾病、症状、其他三个层级显示结果。
    
#### 1.2 关系权重可视化  
    反映疾病和症状（或其他类）之间联系的紧密程度（即共现图权重）。  

#### 1.3 突出视图区域（鱼眼视图）    
    突出用户正在查看的视图区域，使得能够用户清晰地查看细节。
    
#### 1.4 表格信息展示    
    通过点击节点，以表格形式显示该节点更多的信息。

#### 1.5 节点信息展开        
    通过点击节点，增加与之相关的节点和边，且将其放置中心区域，其他节点往两边滑动。
    
#### 1.6 图的缩放
    通过鼠标齿轮对界面进行缩放。

## 2、项目方案

#### **2.1 总体框架图(Django+Neo4j+D3)**  
图片    
![image](https://github.com/cloudXia777/Medical-Visualization/blob/master/image/%E6%80%BB%E4%BD%93%E6%A1%86%E6%9E%B6%E5%9B%BE.png)  

#### **2.2 方案设计**
1. 选择[Django](https://www.djangoproject.com/)作为web应用开发框架
2. 使用[D3.force](https://d3js.org/)布局实现实现图谱可视化
3. 使用AJAX实现JS与后端的数据交互
4. 使用[Neomodel](https://neomodel.readthedocs.io/en/latest/)实现Django与[Neo4j](https://neo4j.com/)图数据库的数据交互  

## 3、实现细节
#### **3.1 开发环境**    
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

#### **3.2 开发模块**
**3.2.1 前端设计**  
分为三个部分：1.头区域（LOGO、检索区）2.主体区域（节点活动区域）。3.尾部区域（节点指示）。

**3.2.2 层级显示**  
强制限定每个节点y方向的值；根据节点的类别信息，指定其所处y方向的位置；
```
nodes.forEach(function (o, i) {
/* height 表示主体区域的高度
   100为层级之间的距离，可以根据显示区域自行设置*/
    if (o.group === 1) {
        o.y = height / 2;
    } else if (o.group === 2) {
        o.y = height / 2 + 100;
    } else {
        o.y = height / 2 - 100;
    }
});
```
*注意使用强制层级会造成节点之间的连线不服从最初的长度设置*     

**3.2.3 权重可视化**    
    
```
/*设置连线宽度属性，这里为权重开根号*/
.att("stroke-width", function(d){
    return Math.sqrt(d.value);
})
```

**3.2.3 拖拽动作**  

```
/*设置拖拽时的动作，包含了开始、运动中、结束*/
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

**3.2.4 鱼眼失真**  

```
/*引用【1】的示例*/
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
            /*根据节点半径的大小而变化*/
            return  '15px'
        })
});
<script language="javascript" type = "text/javascript" src="{% static 'js/fisheye.js' %}"></script>
```

**3.2.5 点击节点居中**

```
/*1. 计算点击节点与中心的距离
  2. 查找点击节点所在的连通图
  3. 连通图中所有节点移动相同距离*/

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

**3.2.6 节点自动两边移动**  
```
/*1. 判断其余节点所在位置
  2. 然后再移动*/
// 判断其余节点在点击左边、右边
nodes.forEach(function (d, i) {
    if (! n2N(ClickName, d.id)){
        if (d.x < pos)
            direct[i] = 'left';
        else
            direct[i] = 'right';
    }
});
var k = 10 * e.alpha; //得到近似1的值
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
**3.2.7 碰撞检测**  
```
function tick(e){
    var q = d3.geom.quadtree(nodes),
    i = 0,
    n = nodes.length;
    while (++i < n) q.visit(d3.collide(nodes[i]));
}
<script src="{% static 'js/collide.js' %}"></script>
```
**3.2.8 表格显示**  
```
/*右击新建表格*/
function rclick(d,e) {
    d3.event.preventDefault(e);
    // 先删除，再新建
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
    // 这里可以替换服务器响应
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
#### **3.3 参考文献**
[1] https://github.com/d3/d3-plugins/tree/master/fisheye    
[2] https://observablehq.com/@d3/force-directed-graph?collection=@d3/d3-force   
[3] https://bl.ocks.org/mbostock    
[4] https://gist.github.com/pkerpedjiev/0389e39fad95e1cf29ce    
[5] https://bl.ocks.org/mbostock/1093130    
[6] http://bl.ocks.org/sgruhier/1d692762f8328a2c9957    
[7] https://www.npmjs.com/package/d3-context-menu   
[8] http://visualdataweb.de/webvowl/#   
## 4、未来工作
* 前端视图自适应
* 选中个别疾病后，其余未选中的节点消失
* 目前所设定的是以疾病为起点，其余为目标节点
