<template>
  <div class="div_center" id="div_center">
    <h4> CONSTELLATION BASED ON ITERATION </h4>
    <div class="demo1">
      <tree_menu></tree_menu>
    </div>
    <div class="demo2" id="treeview"></div>
    <sca_location v-if="!show1"></sca_location>
  </div>

</template>

<script>
 import * as d3 from 'd3';
 import * as d4 from '../../public/d3';
 import $ from 'jquery';
 import tree_menu from "./tree_menu";
  import Readcsv from "../ToolJs/Readcsv";
  import sca_location from "./sca_location";
  export default {
     name: 'div_center',
    data() {
      return {
        control_set: {},
        show1: true,

      }
    },
    components: {
      tree_menu,
      sca_location,
    },
   methods: {
      node_view(type_name, style, bubbledata) {
        //console.log(style["secondary_node"])
        d3.select('#div_center').selectAll('svg').remove();

        var width = $('#div_center').width(),
          height = $('#div_center').height() * 0.96,
          root;

        //缩放功能
        var zoom = d3.behavior.zoom().scaleExtent([1, 8]).on("zoom", zoomed);

        function zoomed() {
          svg.attr("transform",
            "translate(" + zoom.translate() + ")" +
            "scale(" + zoom.scale() + ")"
          );
        }

        var svg = d3.select("#div_center").append("svg")
          .attr("width", width)
          .attr("height", height)
          .append("g")
          .attr("id", "all_g")
          .call(zoom)


        const len = Object.keys(type_name).length

        function draw_charts(name, type_name, x_transform, y_transform, radius) {

          var data_inner = [
            [radius * 0.9, radius * 0.16],
            [-radius * 0.9, radius * 0.16],
            [radius * 0.9, -radius * 0.16],
            [-radius * 0.9, -radius * 0.16],
            [radius / 1.9, radius / 3.15],
            [-radius / 1.9, radius / 3.15],
            [-radius / 1.9, -radius / 3.15],
            [radius / 1.9, -radius / 3.15],
            [0, radius * 0.375],
            [0, -radius * 0.375]
          ]

          var data_outer = [
            [-radius * 1.2, 0],
            [radius * 1.2, 0],
            [radius * 0.9, radius * 0.3],
            [-radius * 0.9, radius * 0.3],
            [-radius * 0.9, -radius * 0.3],
            [radius * 0.9, -radius * 0.3],
            [radius * 0.35, -radius * 0.43],
            [radius * 0.35, radius * 0.43],
            [-radius * 0.35, radius * 0.43],
            [-radius * 0.35, -radius * 0.43],
          ]

          for (var obj in type_name) {

            for (var i = 0; i < data_inner.length; i++) {
              data_inner[i].push(Object.values(type_name[obj][0])[i], Object.keys(type_name[obj][0])[i])
              data_outer[i].push(Object.values(type_name[obj][1])[i], Object.keys(type_name[obj][1])[i])
            }
          }

          //绘制极坐标轴
          function polar_plot() {
            //比例尺
            var r_inner = d3.scale.linear()
              .domain([0, radius * 0.9])
              .range([0, radius]);

            var r_outer = d3.scale.linear()
              .domain([0, radius * 0.9])
              .range([0, radius]);

            var line = d3.svg.line.radial()
              .radius(function (d) {
                return r(d[1]);
              })
              .angle(function (d) {
                return -d[0] + Math.PI / 2;
              });

            //内半径坐标轴
            var gr_inner = svg.append("g")
              .attr("class", "r axis inner")
              .attr("transform", "translate(" + x_transform / 2 + "," + y_transform / 2 + ")");

            gr_inner.append("circle")
              .attr("r", radius)
              .attr("fill", "none")
              .attr("stroke", "steelblue")
              .attr("stroke-dasharray", "4")
              .style("opacity", style["secondary_node"][1])
              .attr("transform", "translate(" + x_transform / 2 + "," + y_transform / 2 + ")");

            //外半径坐标轴
            var gr_outer = svg.append("g")
              .attr("class", "r axis outer")
              .attr("transform", "translate(" + x_transform / 2 + "," + y_transform / 2 + ")");

            gr_outer.append("circle")
              .attr("r", radius * 1.2)
              .attr("fill", "none")
              .attr("stroke", "steelblue")
              .attr("stroke-dasharray", "4")
              .style("opacity", style["secondary_node"][1])
              .attr("transform", "translate(" + x_transform / 2 + "," + y_transform / 2 + ")");

            var temp1 = x_transform - radius / 2
            var temp2 = y_transform + radius * 1.4

            svg.append("text")
              .text(name)
              .style("font-size", 18)
              .style("fill", '#000000')
              .attr("text-align", "center")
              .attr("transform", "translate(" + temp1 + "," + temp2 + ")");

            //角度坐标轴
            // var ga = svg.append("g")
            //     .attr("class", "a axis")
            //     .selectAll("g")
            //     .data(d3.range(0, 360, 36))  //0-360度中间隔360度画一条线
            //     .enter()
            //     .append("g")
            //     .attr("transform", "translate(" + width / 4 + "," + height / 4 + ")");
            // .attr("transform", function(d) {
            //    // console.log("rotate(" + -d + ")")
            //     return "rotate(" + -d + ")";
            // });
            //
            // ga.append("line")
            //     .attr("x2", radius);

            var color = ['#cbdcdf', '#a5bbbf', '#cce8e2', '#ce848f', '#EFBAC0',
              '#F9F3E1', '#EDE9BF', '#F7E7E0', '#F7DDC0',
              '#F9BCB2', '#ea8991', '#cbdcdf', '#a5bbbf',
              '#cce8e2', '#ce848f', '#EFBAC0', '#F9F3E1',
              '#EDE9BF', '#F7E7E0', '#F7DDC0'];

            svg.selectAll("point")
              .data(data_inner)
              .enter()
              .append("circle")
              .attr("class", "point")

              .attr("id", function (d) {
                return d[3]
              })
              .attr("transform", function (d) {
                var an = d[0],
                  ra = r_inner(d[1]),
                  x = ra + x_transform + d[1] * 1.55,
                  y = an + y_transform;
                return "translate(" + [x, y] + ")";
              })
              .attr("r", function (d) {
                // console.log(d)
                return d[2] * (style["secondary_node"][0] / 2)
              })
              .attr("fill", function (d, i) {
                return color[i];
              })
              .style("opacity", style["secondary_node"][1])

            svg.selectAll("point")
              .data(data_outer)
              .enter()
              .append("circle")
              .attr("class", "point")
              .attr("id", function (d) {
                return d[3]
              })
              .attr("transform", function (d) {
                //console.log(d[1])
                // console.log( r(d[1]))
                var an = d[0],
                  ra = r_outer(d[1]),
                  x = ra + x_transform + d[1] * 1.55,
                  y = an + y_transform;
                return "translate(" + [x, y] + ")";
              })
              .attr("r", function (d) {
                return d[2] * (style["secondary_node"][0] / 2)
              })
              .attr("fill", function (d, i) {
                return color[i];
              })
              .style("opacity", style["secondary_node"][1])
          }

          //绘制中心径向树图
          function center_tree(bubbledata) {


            var color = ['#cbdcdf', '#a5bbbf', '#cce8e2', '#ce848f', '#EFBAC0',
              '#F9F3E1', '#EDE9BF', '#F7E7E0', '#F7DDC0',
              '#F9BCB2', '#ea8991', '#cbdcdf', '#a5bbbf',
              '#cce8e2', '#ce848f', '#EFBAC0', '#F9F3E1',
              '#EDE9BF', '#F7E7E0', '#F7DDC0'];

            //外圈圆形
            var forces = svg.append("circle")
              .attr("id", "circle")
              .attr("cx", x_transform)
              .attr("cy", y_transform)
              .attr("r", radius * 0.8)
              .style("fill", 'rgba(203, 220, 223,0.2)')
              .style("stroke", "rgba(123,172,186,0.7)")
              .style("stroke-width", "10px")
            //.attr("transform", "translate(" -10  + "," -10  + ")");

            var tooltip = d3.select("#all_g").append("div")
              .attr("class", "tooltip") //用于css设置类样式
              .attr("opacity", 0.0);

            //绘制气泡饼图
            function bubblepie(data) {

              var packwidth = radius * 0.8 * 2
              var packheight = radius * 0.8 * 2

              var pack = d4.pack()
                .size([packwidth, packheight])
                .padding(1.5);

              var num, pid;
              var root = d4.hierarchy({children: data})
                .sum(function (d) {
                  return d.count;
                })
                .each(function (d) {
                  if (d.parent == null) {
                    num = d.value
                  }
                  if (id = d.data.name) {
                    var id
                    d.id = id;
                    d.class = id;
                    pid = num / (d.value);
                    d.colorPick = pid > 100 ? 5 : (pid > 50 ? 4 : (pid > 10 ? 3 : (pid > 2 ? 2 : 1)));
                  }
                });

              var node = svg.append("g").attr("id", "pie_g")
                .selectAll(".node")
                .data(pack(root).leaves())
                .enter().append("g")
                .attr("class", "node")
                .attr("id", function (d) {
                  return name + '__' + d.id
                })
                .attr("transform", function (d) {
                  DrawPie(d)
                  var x = d.x + x_transform - radius * 0.8
                  var y = d.y + y_transform - radius * 0.8
                  return "translate(" + x + "," + y + ")";
                })

              function DrawPie(data) {

                var radius = data.r

                var g = svg.select("#" + name + '__' + data.id)
                  .append("g")

                var pie = d4.pie()
                  .sort(null)
                  .value(function (d) {
                    return d.value;
                  });

                var path = d4.arc()
                  .outerRadius(radius)
                  .innerRadius(radius / 2);

                var path0 = d4.arc()
                  .outerRadius(radius + 10)
                  .innerRadius((radius + 10) / 2);

                var label = d4.arc()
                  .outerRadius(radius - 20)
                  .innerRadius((radius - 20) / 2);

                var label0 = d4.arc()
                  .outerRadius(radius + 20)
                  .innerRadius((radius + 20) / 2);

                var arc = g.selectAll(".arc")
                  .data(pie(data.data.piedata))
                  .enter().append("g")
                  .attr("class", "arc")


                arc.append("path")
                  .attr("d", path)
                  .attr("class", function (d) {
                    return d.data.area
                  })
                  .attr("fill", function (d, i) {
                    return color[i];
                  })
                // .attr("transform", function(d) {
                //     // console.log(this.id)
                //     return "translate(" + d.x + "," + d.y + ")";
                // })

                arc.on("mouseover", function (d, i) {
                  // d3.select(this).select("path")
                  //     .attr("d",path0);
                  d4.select(this).select("path").attr("fill", "#ad4646")

                  var x = event.pageX;
                  var y = event.pageY;

                  svg.append("text")
                    .attr("id", "tooltip")
                    .attr("x", x)
                    .attr("y", y)
                    .attr("text-anchor", "middle")
                    .attr("font-family", "sans-setif")
                    .attr("font-size", "15px")
                    .attr("font-weight", "bold")
                    .attr("fill", "black")
                    .text(d.data.value + "%");
                });

                arc.on("mouseout", function (d, i) {
                  d3.select("#tooltip").remove();
                  d4.select(this).select(".正影响").attr("fill", '#cbdcdf')
                  d4.select(this).select(".负影响").attr("fill", '#a5bbbf')

                });

                arc.append("text")
                  .attr("transform", d => {
                    return "translate(" + path.centroid(d) + ")";
                  })
                  .text(function (d) {
                    return d.data.value;
                  })
                  .style("font-size", radius / 8)

              }
            }

            bubblepie(bubbledata[name])

          }

          center_tree(bubbledata)
          polar_plot()

        }

        let name = []

        for (var obj in type_name) {
          name.push(obj)
        }
        // console.log(name)
        if (len == '1') {
          draw_charts(name, type_name, width * 0.4, height * 0.5, width * 0.18, bubbledata)
        } else if (len == '2') {
          draw_charts(name[0], type_name, width * 0.18, height * 0.6, width * 0.13, bubbledata)
          draw_charts(name[1], type_name, width * 0.5, height * 0.35, width * 0.13, bubbledata)
        } else if (len == '3') {
          draw_charts(name[0], type_name, width * 0.17, height * 0.28, width * 0.105, bubbledata)
          draw_charts(name[1], type_name, width * 0.6, height * 0.28, width * 0.105, bubbledata)
          draw_charts(name[2], type_name, width * 0.4, height * 0.7, width * 0.105, bubbledata)
        } else if (len == '4') {
          draw_charts(name[0], type_name, width * 0.18, height * 0.25, width * 0.09, bubbledata)
          draw_charts(name[1], type_name, width * 0.5, height * 0.25, width * 0.09, bubbledata)
          draw_charts(name[2], type_name, width * 0.18, height * 0.75, width * 0.09, bubbledata)
          draw_charts(name[3], type_name, width * 0.5, height * 0.75, width * 0.09, bubbledata)
        } else if (len == '5') {
          draw_charts(name[1], type_name, width * 0.18, height * 0.25, width * 0.09, bubbledata)
          draw_charts(name[2], type_name, width * 0.45, height * 0.25, width * 0.09, bubbledata)
          draw_charts(name[3], type_name, width * 0.18, height * 0.75, width * 0.09, bubbledata)
          draw_charts(name[4], type_name, width * 0.45, height * 0.75, width * 0.09, bubbledata)
          draw_charts(name[5], type_name, width * 0.7, height * 0.25, width * 0.09, bubbledata)
        }
      },

    },

       mounted() {
      // this.node_view(type_name,style)
    },

      computed: {
      control2() {
        return this.$store.getters.get_control2;
        // return this.$store.getters.get_hotData;
      },
      center_data() {
        return this.$store.getters.get_centerData;
      },
      centerDataTree() {
        return this.$store.getters.get_centerDataTree;
      },
    },
    watch: {
      // tree_menu:function () {
      //    //this.=this.$store.getters.get_treeMenu
      // },
      control2: function () {
        let bubbledata = {
          "SMS_malware": [
            {name: "tree1", count: 345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree2", count: 3345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree3", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree4", count: 245, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree5", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree6", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree7", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree8", count: 2345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree9", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree10", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree11", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree12", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree13", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree14", count: 5045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree15", count: 1235, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree16", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree17", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree18", count: 345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree19", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree20", count: 345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree21", count: 1345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree22", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree23", count: 145, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree24", count: 245, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree25", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree26", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree27", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree28", count: 125, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree29", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree30", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree31", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree32", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree33", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree34", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree35", count: 1245, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree36", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree37", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree38", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree39", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree40", count: 234, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree41", count: 125, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree42", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree43", count: 145, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree44", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree45", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree46", count: 25, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree47", count: 105, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree48", count: 234, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree49", count: 334, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree50", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]}
          ],
          "Bank_malware": [
            {name: "tree1", count: 345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree2", count: 3345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree3", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree4", count: 245, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree5", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree6", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree7", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree8", count: 2345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree9", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree10", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree11", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree12", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree13", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree14", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree15", count: 1235, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree16", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree17", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree18", count: 345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree19", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree20", count: 345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree21", count: 1345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree22", count: 13035, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree23", count: 145, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree24", count: 245, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree25", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree26", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree27", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree28", count: 125, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree29", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree30", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree31", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree32", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree33", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree34", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree35", count: 1245, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree36", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree37", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree38", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree39", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree40", count: 234, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree41", count: 125, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree42", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree43", count: 145, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree44", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree45", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree46", count: 25, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree47", count: 105, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree48", count: 234, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree49", count: 334, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree50", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]}
          ],
          "Benign_application": [
            {name: "tree1", count: 345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree2", count: 3345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree3", count: 10345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree4", count: 245, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree5", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree6", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree7", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree8", count: 2345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree9", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree10", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree11", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree12", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree13", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree14", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree15", count: 1235, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree16", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree17", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree18", count: 345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree19", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree20", count: 345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree21", count: 1345, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree22", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree23", count: 145, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree24", count: 245, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree25", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree26", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree27", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree28", count: 125, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree29", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree30", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree31", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree32", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree33", count: 2345, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree34", count: 1045, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree35", count: 1245, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree36", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree37", count: 1345, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree38", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree39", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree40", count: 234, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree41", count: 125, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree42", count: 345, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree43", count: 145, piedata: [{"area": "正影响 ", "value": 20}, {"area": "负影响 ", "value": 80}]},
            {name: "tree44", count: 2345, piedata: [{"area": "正影响 ", "value": 50}, {"area": "负影响 ", "value": 50}]},
            {name: "tree45", count: 545, piedata: [{"area": "正影响 ", "value": 70}, {"area": "负影响 ", "value": 30}]},
            {name: "tree46", count: 25, piedata: [{"area": "正影响 ", "value": 65}, {"area": "负影响 ", "value": 35}]},
            {name: "tree47", count: 105, piedata: [{"area": "正影响 ", "value": 45}, {"area": "负影响 ", "value": 55}]},
            {name: "tree48", count: 234, piedata: [{"area": "正影响 ", "value": 60}, {"area": "负影响 ", "value": 40}]},
            {name: "tree49", count: 334, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]},
            {name: "tree50", count: 335, piedata: [{"area": "正影响 ", "value": 30}, {"area": "负影响 ", "value": 70}]}
          ]
        }
        this.control_set = this.$store.getters.get_control2;
        // console.log(this.control_set,'sssss')
        this.node_view(
          {
            "SMS_malware": [
              {
                "attr_a": 10,
                "attr_b": 8,
                "attr_c": 7,
                "attr_d": 13,
                "attr_e": 12,
                "attr_f": 10,
                "attr_g": 8,
                "attr_h": 7,
                "attr_i": 13,
                "attr_j": 12
              },   //属性
              {
                "p_a": 10,
                "p_b": 8,
                "p_c": 7,
                "p_d": 13,
                "p_e": 12,
                "p_f": 10,
                "p_g": 8,
                "p_h": 7,
                "p_i": 13,
                "p_j": 12
              },   //性能
            ],
            "Bank_malware": [
              {
                "attr_a": 10,
                "attr_b": 8,
                "attr_c": 7,
                "attr_d": 13,
                "attr_e": 12,
                "attr_f": 10,
                "attr_g": 8,
                "attr_h": 7,
                "attr_i": 13,
                "attr_j": 12
              },   //属性
              {
                "p_a": 10,
                "p_b": 8,
                "p_c": 7,
                "p_d": 13,
                "p_e": 12,
                "p_f": 10,
                "p_g": 8,
                "p_h": 7,
                "p_i": 13,
                "p_j": 12
              },   //性能
            ],
            "Benign_application": [
              {
                "attr_a": 10,
                "attr_b": 8,
                "attr_c": 7,
                "attr_d": 13,
                "attr_e": 12,
                "attr_f": 10,
                "attr_g": 8,
                "attr_h": 7,
                "attr_i": 13,
                "attr_j": 12
              },   //属性
              {
                "p_a": 10,
                "p_b": 8,
                "p_c": 7,
                "p_d": 13,
                "p_e": 12,
                "p_f": 10,
                "p_g": 8,
                "p_h": 7,
                "p_i": 13,
                "p_j": 12
              },   //性能
            ]
          }, this.control_set, bubbledata)

      },
      center_data: function () {
        let draw_data = [];
        let main_data = this.center_data;
        for (let i = 0; i < main_data.length; i++) {
          let key = main_data[i].label;
          draw_data.push(key.split(':')[0]);
        }
        Readcsv.Read_centerData(draw_data);
        console.log(draw_data, 'hhhhhh')
        // console.log(this.$store.getters.get_centerDataTree,'wwww')
        // this.node_view(draw_data);
      },

      centerDataTree: function () {
        let this_data = this.centerDataTree;
        this.node_view(this_data, this.control_set)
        //  this.node_view(
        //    {
        //      "SMS_malware": [
        //        {
        //          "att_a": 10,
        //          "att_b": 8,
        //          "att_c": 7,
        //          "att_d": 13,
        //          "att_e": 12,
        //          "att_f": 10,
        //          "att_g": 8,
        //          "att_h": 7,
        //          "att_i": 13,
        //          "att_j": 12
        //        },   //属性
        //        {
        //          "p_a": 10,
        //          "p_b": 8,
        //          "p_c": 7,
        //          "p_d": 13,
        //          "p_e": 12,
        //          "p_f": 10,
        //          "p_g": 8,
        //          "p_h": 7,
        //          "p_i": 13,
        //          "p_j": 12
        //        },   //性能
        //      ]
        //    }, this.control_set)
      }


    }
  }
</script>

<style scoped>
  .div_center {
    width: 60vw;
    position: relative;
    /*margin: 0 0.2%;*/
    border-top: 3px solid rgb(255, 255, 255);
    border-bottom: 3px solid rgb(255, 255, 255);

  }

  .demo2 {
    height: 20vh;
    width: 43vw;
    position: absolute;
    right: 10px;
    bottom: -20px;
    z-index: 999;
    border: 3px solid rgb(242, 242, 242);
    /*border-left:  3px solid rgb(242,242,242);*/
  }

  #div_center {
    width: 100%;
    height: 100%;
  }

  .demo1 {
    height: 20vh;
    width: 11vw;
    position: absolute;
    left: 0;
    bottom: -20px;
    z-index: 999;
    border: 3px solid rgb(242, 242, 242);
  }

  .axis text {
    font: 10px sans-serif;
  }

  .axis line,
  .axis circle {
    fill: none;
    stroke: steelblue;
    stroke-dasharray: 4;
  }

  .axis:last-of-type circle {
    stroke: steelblue;
    stroke-dasharray: none;
  }

  .line {
    fill: none;
    stroke: orange;
    stroke-width: 3px;
  }

  .tooltip {
    font-family: simsun;
    font-size: 16px;
    width: 120px;
    height: auto;
    position: absolute;
    text-align: center;
    border-style: solid;
    border-width: 1px;
    background-color: white;
    border-radius: 5px;
  }

  h4 {
    font-family: 微软雅黑;
    background-color: rgb(242, 242, 242);
    padding: 3px 3px;
    border-bottom: 2px solid rgb(200, 200, 200);
    text-align: center;

  }


</style>
