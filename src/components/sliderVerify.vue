<template>
  <div class="hello">
    <canvas width="310"
            height="155"
            ref="canvas"></canvas>
    <div class="refreshIcon" @click="reset"></div>
    <canvas width="310"
            height="155"
            ref="block"
            style="position:absolute;left:0;top:0"></canvas>
    <div class="sliderContainer"
         ref="sliderContainer">
      <div class="sliderMask"
           ref="sliderMask">
        <div class="slider"
             ref="slider"
             @mousedown="handleDrag($event)">
          <span class="sliderIcon"></span>
        </div>
      </div>
      <span class="sliderText">向右滑动滑块填充拼图</span>
    </div>
  </div>
</template>

<script>
var x = 150,
  y = 40,
  w = 42,
  r = 10,
  PI = Math.PI;
function draw (ctx, operation) {
  ctx.beginPath();
  ctx.moveTo(x, y);
  ctx.lineTo(x + w / 2, y);
  ctx.arc(x + w / 2, y - r + 2, r, 0, 2 * PI);
  ctx.lineTo(x + w / 2, y);
  ctx.lineTo(x + w, y);
  ctx.lineTo(x + w, y + w / 2);
  ctx.arc(x + w + r - 2, y + w / 2, r, 0, 2 * PI);
  ctx.lineTo(x + w, y + w / 2);
  ctx.lineTo(x + w, y + w);
  ctx.lineTo(x, y + w);
  ctx.lineTo(x, y);
  ctx.fillStyle = "#fff";
  ctx[operation]();
  ctx.beginPath();
  ctx.arc(x, y + w / 2, r, 1.5 * PI, 0.5 * PI); // 只需要画正方形内的半圆就行，方便背景图片的裁剪
  ctx.globalCompositeOperation = "xor";
  ctx.fill();
}
export default {
  name: "sliderVerify",
  data () {
    return {
      canvas: null,
      block: null,
      canvas_ctx: null,
      block_ctx: null,
      originX: null,
      originY: null,
      trail: [],
      imgsrc: this.getRandomImg(),
      isMouseDown: false
    };
  },
  methods: {
    handleDrag (e) {
      this.originX = e.x
      this.originY = e.y
      this.isMouseDown = true
      const convasw = 310
      document.addEventListener('mousemove', (e) => {
        if (!this.isMouseDown) return false
        const moveX = e.x - this.originX
        const moveY = e.y - this.originY
        if (moveX < 0 || moveX + 38 >= convasw) return false
        this.$refs.slider.style.left = moveX + 'px'
        var blockLeft = (convasw - 40 - 20) / (convasw - 40) * moveX
        this.$refs.block.style.left = blockLeft + 'px'

        this.$refs.sliderContainer.classList.add('sliderContainer_active')
        this.$refs.sliderMask.style.width = moveX + 'px'
        this.trail.push(moveY)
      })
      document.addEventListener('mouseup', (e) => {
        if (!this.isMouseDown) return false
        this.isMouseDown = false
        if (e.x == this.originX) return false
        this.$refs.sliderContainer.classList.remove('sliderContainer_active')
        const { spliced, TuringTest } = this.verify()
        if (spliced) {
          if (TuringTest) {
            this.$refs.sliderContainer.classList.add('sliderContainer_success')
            this.success && this.success()
          } else {
            this.$refs.sliderContainer.classList.add('sliderContainer_fail')
            this.reset()
          }
        } else {
          this.$refs.sliderContainer.classList.add('sliderContainer_fail')
          this.fail && this.fail()
          setTimeout(() => {
            this.reset()
          }, 1000)
        }
      })
    },
    verify () {
      const arr = this.trail // 拖动时y轴的移动距离
      const average = arr.reduce((x, y) => x + y) / arr.length // 平均值
      const deviations = arr.map(x => x - average) // 偏差数组
      const stddev = Math.sqrt(deviations.map(x => x * x).reduce((x, y) => x + y) / arr.length) // 标准差
      const left = parseInt(this.$refs.block.style.left)
      return {
        spliced: Math.abs(left - x) < 10,
        TuringTest: average !== stddev, // 只是简单的验证拖动轨迹，相等时一般为0，表示可能非人为操作
      }
    },
    success () {
      console.log('success')
      setTimeout(() => {
        this.reset()
      }, 1000)
    },
    fail () {
      console.log('fail')
      setTimeout(() => {
        this.reset()
      }, 1000)
    },
    reset () {
      this.$refs.sliderContainer.className = 'sliderContainer'
      this.$refs.slider.style.left = 0
      this.$refs.block.style.left = 0
      this.$refs.sliderMask.style.width = 0
      this.$refs.canvas.getContext("2d").clearRect(0, 0, 310, 155)
      this.$refs.block.getContext("2d").clearRect(0, 0, 310, 155)
      this.$refs.block.width = 310
      this.imgsrc = this.getRandomImg()
      this.initImage()
    },
    getRandomNumberByRange (start, end) {
      return Math.round(Math.random() * (end - start) + start)
    },
    getRandomImg () {
      return 'https://picsum.photos/300/150/?image=' + this.getRandomNumberByRange(0, 1084)
    },
    initImage () {
      var img = document.createElement("img");
      img.crossOrigin = "anonymous";
      img.onload = () => {
        this.canvas_ctx.drawImage(img, 0, 0, 310, 155);
        this.block_ctx.drawImage(img, 0, 0, 310, 155);
        var blockWidth = w + r * 2;
        var _y = y - r * 2 + 2; // 滑块实际的y坐标
        var ImageData = this.block_ctx.getImageData(x, _y, blockWidth, blockWidth);
        this.block.width = blockWidth;
        this.block_ctx.putImageData(ImageData, 0, _y);
      };
      img.src = this.imgsrc

    draw(this.canvas_ctx, "fill");
    draw(this.block_ctx, "clip");
    }
  },
  mounted () {
      this.canvas = this.$refs.canvas;
      this.block = this.$refs.block;
      this.canvas_ctx = this.canvas.getContext("2d");
      this.block_ctx = this.block.getContext("2d");
      this.initImage()
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  position: relative;
  width: 310px;
}
.sliderContainer {
  position: relative;
  text-align: center;
  width: 310px;
  height: 40px;
  line-height: 40px;
  margin-top: 15px;
  background: #f7f9fa;
  color: #45494c;
  border: 1px solid #e4e7eb;
}

.sliderContainer_active .slider {
  height: 38px;
  top: -1px;
  border: 1px solid #1991fa;
}

.sliderContainer_active .sliderMask {
  height: 38px;
  border-width: 1px;
}

.sliderContainer_success .slider {
  height: 38px;
  top: -1px;
  border: 1px solid #52ccba;
  background-color: #52ccba !important;
}

.sliderContainer_success .sliderMask {
  height: 38px;
  border: 1px solid #52ccba;
  background-color: #d2f4ef;
}

.sliderContainer_success .sliderIcon {
  background-position: 0 0 !important;
}

.sliderContainer_fail .slider {
  height: 38px;
  top: -1px;
  border: 1px solid #f57a7a;
  background-color: #f57a7a !important;
}

.sliderContainer_fail .sliderMask {
  height: 38px;
  border: 1px solid #f57a7a;
  background-color: #fce1e1;
}

.sliderContainer_fail .sliderIcon {
  background-position: 0 -83px !important;
}
.sliderContainer_active .sliderText,
.sliderContainer_success .sliderText,
.sliderContainer_fail .sliderText {
  display: none;
}

.sliderMask {
  position: absolute;
  left: 0;
  top: 0;
  height: 40px;
  border: 0 solid #1991fa;
  background: #d1e9fe;
}

.slider {
  position: absolute;
  top: 0;
  left: 0;
  width: 40px;
  height: 40px;
  background: #fff;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  transition: background 0.2s linear;
}

.slider:hover {
  background: #1991fa;
}

.slider:hover .sliderIcon {
  background-position: 0 -13px;
}

.sliderIcon {
  position: absolute;
  top: 15px;
  left: 13px;
  width: 14px;
  height: 10px;
  background: url(http://cstaticdun.126.net//2.6.3/images/icon_light.f13cff3.png)
    0 -26px;
  background-size: 34px 471px;
}

.refreshIcon {
  position: absolute;
  right: 0;
  top: 0;
  width: 34px;
  height: 34px;
  cursor: pointer;
  background: url(http://cstaticdun.126.net//2.6.3/images/icon_light.f13cff3.png)
    0 -437px;
  background-size: 34px 471px;
}
</style>
