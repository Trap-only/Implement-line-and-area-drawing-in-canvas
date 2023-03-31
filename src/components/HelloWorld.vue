<template>
  <div class="hello">
    <div>当前的类型：画箭头线</div>
    <p>在下面的黑色区域，点击鼠标左键开始吧</p>
    <!-- 新的电子围栏绘画 -->
    <canvas ref="tagcanvas" id="cvs" onselectstart="return false;" style="z-index:3" width='600' height='400' @mousedown="mouseDown($event,'electricFence')"></canvas>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      fenceFlag: false, // 鼠标移动标识
      ctx: '', // 一个 CanvasRenderingContext2D 对象，使用它可以绘制到 Canvas 元素中。
      cvs: null, // canvas元素
      points: [], // canvas上画的坐标点的数组
    }
  },
  methods: {
    // 鼠标按下时计算坐标
    mouseDown (event) {
      if (event.button === 0 && !this.fenceFlag) {
        this.points.push({
          x:
            event.pageX -
            (this.cvs.getBoundingClientRect().left +
              document.documentElement.scrollLeft),
          y:
            event.pageY -
            (this.cvs.getBoundingClientRect().top +
              document.documentElement.scrollTop)
        })
          if (this.points.length >= 1 && this.points.length < 2) {
            this.cvs.addEventListener('mousemove', this.mouseMove, false)
            this.clearCanvas(this.cvs, this.ctx)
            this.drawOldSettingborder(this.points, this.ctx)
          }
          if (this.points.length >= 2) {
            this.cvs.removeEventListener('mousemove', this.mouseMove)
          }
          this.getArrowPoint(this.ctx, this.points)
      } else if (event.button === 2) {
        this.fenceFlag = true
        this.cvs.removeEventListener('mousemove', this.mouseMove)
      }
    },
    // 鼠标移动时计算坐标
    mouseMove (event) {
        let array = this.points.concat({
          x:
            event.pageX -
            (this.cvs.getBoundingClientRect().left +
              document.documentElement.scrollLeft),
          y:
            event.pageY -
            (this.cvs.getBoundingClientRect().top +
              document.documentElement.scrollTop)
        })
        this.clearCanvas(this.cvs, this.ctx)
          if (array.length <= 2) {
            // 当类型为划线，并且划线的点小于2个的时候，才可以再画，否则就不能再画点
            this.drawOldSettingborder(array, this.ctx)
            // 当有两个点，画箭头方向
            if (array.length === 2 ) {
              this.getArrowPoint(this.ctx, array)
            }
          } else {
            this.cvs.removeEventListener('mousemove', this.mouseMove)
          }
    },
    // 清除画布
    clearCanvas (canvas, ctx) {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
    },
    // 绘制电子围栏
    drawOldSettingborder (recs, ctx, text = '') {
      if (recs.length === 0) {
        return 0
      }
      ctx.lineWidth = 2
      ctx.strokeStyle = '#f00'
      ctx.beginPath()
      ctx.moveTo(recs[0].x, recs[0].y)
      for (var i = 0; i < recs.length; i++) {
        ctx.lineTo(recs[i].x, recs[i].y)
      }
      ctx.closePath()
      ctx.stroke()
      if (text !== '') {
        ctx.font = '14px Arial bolder'
        ctx.fillStyle = 'red'
        ctx.textAlign = 'left'
        let x = recs[0].x + 10
        let y = recs[0].y + 20
        ctx.fillText(text, x, y)
      }
      // 如果类型是划线，则需要画箭头
      if (recs.length === 2 && this.type === 'crossLine') {
        this.getArrowPoint(ctx, recs)
      }
    },
    // 画方向箭头书写的坐标
    getArrowPoint (ctx, array) {
      if (array.length === 2) {
        let x1 = array[0].x // 第一个点的X
        let x2 = array[1].x // 第2个点的X
        let y1 = array[0].y // 第一个点的y
        let y2 = array[1].y // 第2个点的y
        // 中点c的位置为
        let c = { x: (x1 + x2) / 2, y: (y1 + y2) / 2 }
        // 根据x、y的值 计算是x1-x2 还是x2-x1 以及y2-y1 还是 y1-y2
        let borderX = null
        let borderY = null
        if (x1 > x2 && y2 > y1) {
          borderX = x2 - x1
          borderY = y2 - y1
        } else if (x1 > x2 && y1 > y2) {
          borderX = x1 - x2
          borderY = y2 - y1
        } else if (x2 > x1 && y1 > y2) {
          borderX = x2 - x1
          borderY = y1 - y2
        } else {
          borderX = x1 - x2
          borderY = y1 - y2
        }
        // 斜边l1的长度为
        let l1 = Math.sqrt(Math.pow(borderX, 2) + Math.pow(borderY, 2))

        // 定义l2的默认长度为50
        let l2 = 15
        let h2 = (l2 * borderX) / l1
        let s2 = Math.sqrt(Math.pow(l2, 2) - Math.pow(h2, 2))
        let endX = array[1].x >= array[0].x ? s2 + c.x : c.x - s2
        let endY = c.y + h2
        let end2X = 2 * c.x - endX
        let end2Y = 2 * c.y - endY
        this.drawLineArrow(ctx, c.x, c.y, endX, endY, '#f00', 'A')
        this.drawLineArrow(ctx, c.x, c.y, end2X, end2Y, '#f00', 'B')
      }
    },
    // 画箭头
    drawLineArrow (ctx, fromX, fromY, toX, toY, color, text) {
      var headlen = 10 // 自定义箭头线的长度
      var theta = 45 // 自定义箭头线与直线的夹角，个人觉得45°刚刚好
      var arrowX, arrowY // 箭头线终点坐标
      // 计算各角度和对应的箭头终点坐标
      var angle = (Math.atan2(fromY - toY, fromX - toX) * 180) / Math.PI
      var angle1 = ((angle + theta) * Math.PI) / 180
      var angle2 = ((angle - theta) * Math.PI) / 180
      var topX = headlen * Math.cos(angle1)
      var topY = headlen * Math.sin(angle1)
      var botX = headlen * Math.cos(angle2)
      var botY = headlen * Math.sin(angle2)
      ctx.beginPath()
      // 画直线
      ctx.moveTo(fromX, fromY)
      ctx.lineTo(toX, toY)

      arrowX = toX + topX
      arrowY = toY + topY
      // 画上边箭头线
      ctx.moveTo(arrowX, arrowY)
      ctx.lineTo(toX, toY)

      arrowX = toX + botX
      arrowY = toY + botY
      // 画下边箭头线
      ctx.lineTo(arrowX, arrowY)
      ctx.strokeStyle = color
      ctx.stroke()
      // 是否要显示A、B文字
      ctx.font = '14px Arial bolder'
      ctx.fillStyle = '#fff'
      ctx.textAlign = 'left'
      ctx.fillText(text, toX, toY)
    }
  },
  mounted () {
    this.$nextTick(() => {
      this.ctx = this.$refs.tagcanvas.getContext('2d')
      this.cvs = document.getElementById('cvs')
    })
  },
}
</script>

<style scoped lang="scss">
#cvs {
  border: 1px solid pink;
  background: #000;
}
</style>
