<template>
  <div class="hello">
    <svg id="bubble-chart" width="954" height="450" />
  </div>
</template>

<script>
  import * as d3 from "d3";
  import render from "d3-render";
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  components: {},
  data() {
    return {
      covidData: [
        {
          "name": "U.S.A.",
          "id": "USA",
          "value": "245540.0",
          "fill": "#D8352A",
          "colour": "white"
        },
        {
          "name": "Spain",
          "id": "ESP",
          "value": "126535.0",
          "fill": "#48509E",
          "colour": "#383838"
        },
        {
          "name": "Italy",
          "id": "ITA",
          "value": "115242.0",
          "fill": "#48509E",
          "colour": "#383838"
        },
        {
          "name": "China",
          "id": "CHN",
          "value": "82465.0",
          "fill": "#F5A623",
          "colour": "#383838"
        },
        {
          "name": "Germany",
          "id": "DEU",
          "value": "79696.0",
          "fill": "#48509E",
          "colour": "#383838"
        },
        {
          "name": "France",
          "id": "FRA",
          "value": "59105.0",
          "fill": "#48509E",
          "colour": "#383838"
        },

      ],
      countries: null,
      colours: {
        pink: "#D8352A",
        red: "#D8352A",
        blue: "#48509E",
        green: "#02A371",
        yellow: "#F5A623",
        hyperGreen: "#19C992",
        purple: "#B1B4DA",
        orange: "#F6E7AD",
        charcoal: "#383838",
      },
    }
  },
  methods: {
    getdata() {
      this.drawType();
    },

    drawType() {
      //设置svg大小
      const width = 954;
      const height = 450;

      //定义圆圈组件
      const circleComponent = ({ r, cx, cy, fill, duration }) => {
        return {
          append: "circle",
          r,
          cx,
          cy,
          fill,
          duration,
        };
      };

      //定义文字组件
      const textComponent = ({
                               key,
                               text,
                               x = 0,
                               y = 0,
                               fontWeight = "bold",
                               fontSize = "12px",
                               textAnchor = "middle",
                               fillOpacity = 1,
                               colour,
                               r,
                               duration = 1000,
                             }) => {
        return {
          append: "text",
          key,
          text,
          x,
          y,
          textAnchor,
          fontFamily: "sans-serif",
          fontWeight,
          fontSize,
          fillOpacity: { enter: fillOpacity, exit: 0 },
          fill: colour,
          duration,
          style: {
            pointerEvents: "none",
          },
        };
      };

      //对数值进行转换,比如42288变为42k
      const format = (value) => {
        const newValue = d3.format("0.2s")(value);
        if (newValue.indexOf("m") > -1) {
          return parseInt(newValue.replace("m", "")) / 1000;
        }
        return newValue;
      };

      //将各地区名称长度和数值与圆圈大小相比较，实现信息动态变化
      const labelComponent = ({ isoCode, countryName, value, r, colour }) => {
        // Don't show any text for radius under 12px
        if (r < 12) {
          return [];
        }
        //console.log(r);
        const circleWidth = r * 2;
        const nameWidth = countryName.length * 10;
        const shouldShowIso = nameWidth > circleWidth;
        const newCountryName = shouldShowIso ? isoCode : countryName;
        const shouldShowValue = r > 18;

        let nameFontSize;

        if (shouldShowValue) {
          nameFontSize = shouldShowIso ? "10px" : "12px";
        } else {
          nameFontSize = "8px";
        }

        return [
          textComponent({
            key: isoCode,
            text: newCountryName,
            fontSize: nameFontSize,
            y: shouldShowValue ? "-0.2em" : "0.3em",
            fillOpacity: 1,
            colour,
          }),
          ...(shouldShowValue
                  ? [
                    textComponent({
                      key: isoCode,
                      text: format(value),
                      fontSize: "10px",
                      y: shouldShowIso ? "0.9em" : "1.0em",
                      fillOpacity: 0.7,
                      colour,
                    }),
                  ]
                  : []),
        ];
      };

      //设置气泡组件
      const bubbleComponent = ({
                                 name,
                                 id,
                                 value,
                                 r,
                                 x,
                                 y,
                                 fill,
                                 colour,
                                 duration = 1000,
                               }) => {
        return {
          append: "g",
          key: id,
          transform: {
            enter: `translate(${x + 1},${y + 1})`,
            exit: `translate(${width / 2},${height / 2})`,
          },
          duration,
          delay: Math.random() * 300,
          children: [
            circleComponent({ key: id, r, fill, duration }),
            ...labelComponent({
              key: id,
              countryName: name,
              isoCode: id,
              value,
              r,
              colour,
              duration,
            }),
          ],
        };
      };

      const pack = (data) =>
              d3
                      .pack()
                      .size([width - 2, height - 2])
                      .padding(2)(d3.hierarchy({ children: data }).sum((d) => d.value));

      //生成气泡图表
      const renderBubbleChart = (selection, data) => {
        const root = pack(data);
        const renderData = root.leaves().map((d) => {
          return bubbleComponent({
            id: d.data.id,
            name: d.data.name,
            value: d.data.value,
            r: d.r,
            x: d.x,
            y: d.y,
            fill: d.data.fill,
            colour: d.data.colour,
          });
        });
        return render(selection, renderData);
      };

      const renderBubbleChartContainer = (data) => {
        return renderBubbleChart("#bubble-chart", data);
      };
      renderBubbleChartContainer(this.covidData);
    },
  },
  mounted() {
    this.getdata();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
