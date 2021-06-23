<template>
  <Layout title="绘图">
    <div
      id="draw-graphic-mark-k-line"
      class="k-line-chart"/>
    <div
      class="k-line-chart-menu-container">
      <button
        v-for="({ key, text }) in graphicMarks"
        :key="key"
        v-on:click="setGraphicMarkType(key)">
        {{ text }}
      </button>
      <button
        v-on:click="removeAllGraphicMark()">
        清除
      </button>
    </div>
  </Layout>
</template>

<script>
import {dispose, init} from "klinecharts"
import {checkPointOnSegment} from "klinecharts/lib/mark/graphicHelper"
import generatedKLineDataList from "../generatedKLineDataList"
import Layout from "@/Layout"

const rect = {
  name: 'rect',
  totalStep: 3,
  checkMousePointOn: (key, type, points, mousePoint) => {
    return checkPointOnSegment(points[0], points[1], mousePoint)
  },
  createGraphicDataSource: (step, tpPoint, xyPoints) => {
    if (xyPoints.length === 2) {
      return [
        {
          type: 'line',
          isDraw: false,
          isCheck: true,
          dataSource: [
            [{...xyPoints[0]}, {x: xyPoints[1].x, y: xyPoints[0].y}],
            [{x: xyPoints[1].x, y: xyPoints[0].y}, {...xyPoints[1]}],
            [{...xyPoints[1]}, {x: xyPoints[0].x, y: xyPoints[1].y}],
            [{x: xyPoints[0].x, y: xyPoints[1].y}, {...xyPoints[0]}]
          ]
        },
        {
          type: 'polygon',
          isDraw: true,
          isCheck: false,
          style: 'fill',
          dataSource: [[
            {...xyPoints[0]},
            {x: xyPoints[1].x, y: xyPoints[0].y},
            {...xyPoints[1]},
            {x: xyPoints[0].x, y: xyPoints[1].y}
          ]]
        },
        {
          type: 'polygon',
          isDraw: true,
          isCheck: false,
          dataSource: [[
            {...xyPoints[0]},
            {x: xyPoints[1].x, y: xyPoints[0].y},
            {...xyPoints[1]},
            {x: xyPoints[0].x, y: xyPoints[1].y}
          ]]
        }
      ]
    }
    return []
  }
}

const martin = {
  name: 'martin',
  totalStep: 3,
  checkMousePointOn: (key, type, points, mousePoint) => {
    return checkPointOnSegment(points[0], points[1], mousePoint)
  },
  createGraphicDataSource: (step, tpPoint, xyPoints) => {
    if (xyPoints.length === 2) {
      return [
        // {
        //   type: 'line',
        //   isDraw: false,
        //   isCheck: true,
        //   dataSource: [
        //     [{ x: xyPoints[0].x, y: xyPoints[1].y }, { ...xyPoints[0] }],//0x1y b
        //     [{ x: xyPoints[1].x, y: xyPoints[0].y }, { ...xyPoints[1] }],// a
        //     [{ ...xyPoints[0] }, { x: xyPoints[1].x, y: xyPoints[0].y }],
        //     [{ ...xyPoints[1] }, { x: xyPoints[0].x, y: xyPoints[1].y }]
        //   ]
        // },
        {
          type: 'polygon',
          isDraw: true,
          isCheck: false,
          style: 'fill',
          dataSource: [[
            {...xyPoints[0]},
            // {x: xyPoints[1].x, y: xyPoints[0].y},//做多
            {...xyPoints[1]},  //做空
            {x: xyPoints[0].x, y: xyPoints[1].y}
          ]]
        },
        {
          type: 'polygon',
          isDraw: true,
          isCheck: false,
          dataSource: [[
            {...xyPoints[0]},
            // {x: xyPoints[1].x, y: xyPoints[0].y},
            {...xyPoints[1]},
            {x: xyPoints[0].x, y: xyPoints[1].y} //0x1y
          ]]
        }
      ]
    }
    return []
  },
  // drawExtend: (ctx, graphicDataSources, markOptions, viewport, precision, xAxis, yAxis) => {
  //   console.log("+++++++++++++++++++++")
  //   console.log(markOptions)
  //   console.log(graphicDataSources)
  //   console.log("========================")
  //   // console.log(ctx)
  //
  //   let y0 = graphicDataSources[0].dataSource[0][0].y
  //   let y1 = graphicDataSources[0].dataSource[0][1].y
  //   if (y1 > y0) {
  //     markOptions.polygon.stroke.color = '#F55353'//red
  //     markOptions.polygon.fill.color = 'rgba(246,42,66, 0.1)'//red
  //   } else {
  //     markOptions.polygon.stroke.color = '#4BF95F'//green
  //     markOptions.polygon.fill.color = 'rgba(72,231,109, 0.2)'//green
  //   }
  // }
}


const circle = {
  name: 'circle',
  totalStep: 3,
  checkMousePointOn: (key, type, points, mousePoint) => {
    const xDis = Math.abs(points.x - mousePoint.x)
    const yDis = Math.abs(points.y - mousePoint.y)
    const r = Math.sqrt(xDis * xDis + yDis * yDis)
    return Math.abs(r - points.radius) < 3;
  },
  createGraphicDataSource: (step, tpPoint, xyPoints) => {
    if (xyPoints.length === 2) {
      const xDis = Math.abs(xyPoints[0].x - xyPoints[1].x)
      const yDis = Math.abs(xyPoints[0].y - xyPoints[1].y)
      const radius = Math.sqrt(xDis * xDis + yDis * yDis)
      return [
        {
          type: 'arc',
          isDraw: true,
          isCheck: false,
          style: 'fill',
          dataSource: [
            {...xyPoints[0], radius, startAngle: 0, endAngle: Math.PI * 2}
          ]
        },
        {
          type: 'arc',
          isDraw: true,
          isCheck: true,
          dataSource: [
            {...xyPoints[0], radius, startAngle: 0, endAngle: Math.PI * 2}
          ]
        }
      ];
    }
    return []
  }
}

export default {
  name: 'DrawGraphicMarkKLineChart',
  components: {Layout},
  data: function () {
    return {
      graphicMarks: [
        {key: 'priceLine', text: '价格线'},
        {key: 'priceChannelLine', text: '价格通道线'},
        {key: 'parallelStraightLine', text: '平行直线'},
        {key: 'fibonacciLine', text: '斐波那契回调'},
        {key: 'rect', text: '自定义矩形'},
        {key: 'martin', text: '自定义马丁'},
        {key: 'circle', text: '自定义圆'}
      ]
    }
  },
  mounted: function () {
    this.kLineChart = init('draw-graphic-mark-k-line')
    this.kLineChart.addCustomGraphicMark(rect)
    this.kLineChart.addCustomGraphicMark(circle)
    this.kLineChart.addCustomGraphicMark(martin)
    this.kLineChart.applyNewData(generatedKLineDataList())
  },
  methods: {
    setGraphicMarkType: function (graphicMarkType) {
      // this.kLineChart.createGraphicMark(graphicMarkType)
      this.createMartinMark();
    },
    createMartinMark(){
      console.log("create martin mark")
      this.kLineChart.createGraphicMark('martin',{
        points: [
          { timestamp: 1614171282000, price: 18987 },
          { timestamp: 1614171202000, price: 16098 },
        ]
      })
    },
    removeAllGraphicMark() {
      this.kLineChart.removeGraphicMark()
    }
  },
  destroyed: function () {
    dispose('draw-graphic-mark-k-line')
  }
}
</script>
