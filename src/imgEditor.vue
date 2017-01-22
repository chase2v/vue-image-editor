<template lang="html">
  <div class="c-imgEditor">
    <div class="c-imgEditor-preview" @mousemove.stop="onMousemovePreview" @mouseup.stop="onMouseupCutDot" @mouseleave.stop="onMouseleavePreview">
      <img :src="imgSrc" ref="img" hidden/>
      <div class="img-map" ref="imgMap">
        <canvas ref="cvs"></canvas>
        <!-- <div class="c-imgEditor-resize">

        </div> -->
        <div
          class="c-imgEditor-cut"
          v-show="isCutting"
          ref="cut"
          @mousedown.stop="onMousedownCutDot">
          <span class="c-imgEditor-cut-dot" data-id="0"></span>
          <span class="c-imgEditor-cut-dot" data-id="1"></span>
          <span class="c-imgEditor-cut-dot" data-id="2"></span>
          <span class="c-imgEditor-cut-dot" data-id="3"></span>
          <span class="c-imgEditor-cut-dot" data-id="4"></span>
          <span class="c-imgEditor-cut-dot" data-id="5"></span>
          <span class="c-imgEditor-cut-dot" data-id="6"></span>
          <span class="c-imgEditor-cut-dot" data-id="7"></span>
          <span class="c-imgEditor-cut-title">裁剪</span>
        </div>
      </div>
    </div>
    <div class="c-imgEditor-main">
      <input type="file" @change="onChangeInput" />
    </div>
  </div>
</template>

<script>
import Panel from '../panel/panel'

export default {
  props: {
    config: {
      type: Object
    }
  },
  data () {
    return {
      imgSrc: null,
      isCutting: false,
      isMousedown: false, // 标识：是否在裁剪
      curDot: null, // 当前在调整的节点
      imgWidth: undefined,
      imgHeight: undefined,
      cvsTop: undefined,
      cvsLeft: undefined
    }
  },
  methods: {
    resize () {

    },
    onMousedownCutDot (e) {
      this.isMousedown = true
      this.curDot = e.target
    },
    onMousemovePreview (e) {
      if (e.target.tagName !== 'SPAN') {
        if (this.isMousedown && this.curDot) {
          switch (this.curDot.dataset.id) {
            case '0':
              this.$refs.cut.style.top = (e.clientY - this.cvsTop) + 'px'
              this.$refs.cut.style.left = (e.clientX - this.cvsLeft) + 'px'
              break
            case '1':
              this.$refs.cut.style.top = (e.clientY - this.cvsTop) + 'px'
              break
            case '2':
              this.$refs.cut.style.right = this.imgWidth - (e.clientX - this.cvsLeft) + 'px'
              this.$refs.cut.style.top = (e.clientY - this.cvsTop) + 'px'
              break
            case '3':
              this.$refs.cut.style.left = (e.clientX - this.cvsLeft) + 'px'
              break
            case '4':
              this.$refs.cut.style.right = this.imgWidth - (e.clientX - this.cvsLeft) + 'px'
              break
            case '5':
              this.$refs.cut.style.left = (e.clientX - this.cvsLeft) + 'px'
              this.$refs.cut.style.bottom = this.imgHeight - (e.clientY - this.cvsTop) + 'px'
              break
            case '6':
              this.$refs.cut.style.bottom = this.imgHeight - (e.clientY - this.cvsTop) + 'px'
              break
            case '7':
              this.$refs.cut.style.right = this.imgWidth - (e.clientX - this.cvsLeft) + 'px'
              this.$refs.cut.style.bottom = this.imgHeight - (e.clientY - this.cvsTop) + 'px'
              break
            default:
          }
        }
        // else {}
      }
      // else {}
    },
    onMouseupCutDot (e) {
      this.isMousedown = 0
    },
    onMouseleavePreview (e) {
      this.isMouseDown = 0
    },
    onChangeInput (e) {
      let imgSrc = window.URL.createObjectURL(e.currentTarget.files[0])
      let img = new window.Image()
      img.onload = (e) => {
        this.isCutting = true

        let ratioImage = img.width / img.height
        let ratioImg = 4 / 3
        ratioImage = ratioImage.toFixed(3)
        ratioImg = ratioImg.toFixed(3)
        if (ratioImage >= ratioImg) {
          this.$refs.img.style.width = '400px'
          this.$refs.img.style.height = 400 / ratioImage + 'px'
        } else {
          this.$refs.img.style.height = '300px'
          this.$refs.img.style.width = ratioImage * 300 + 'px'
        }

        this.imgWidth = parseInt(window.getComputedStyle(this.$refs.img).width)
        this.imgHeight = parseInt(window.getComputedStyle(this.$refs.img).height)

        this.$refs.imgMap.style.width = this.imgWidth + 'px'
        this.$refs.imgMap.style.height = this.imgHeight + 'px'

        // // 在 cavas 上绘制图像
        this.$refs.cvs.width = this.imgWidth
        this.$refs.cvs.height = this.imgHeight
        let ctx = this.$refs.cvs.getContext('2d')
        ctx.drawImage(this.$refs.img, 0, 0, this.imgWidth, this.imgHeight)

        this.cvsLeft = this.$refs.cvs.getBoundingClientRect().left
        this.cvsTop = this.$refs.cvs.getBoundingClientRect().top
      }
      img.src = imgSrc
      this.imgSrc = imgSrc
    }
  },
  components: {
    'c-panel': Panel
  },
  mounted () {

  }
}
</script>

<style lang="scss">
.c-imgEditor {
  .c-imgEditor-preview {
    position: absolute;
    top: 200px;
    left: 50%;
    transform: translateX(-50%);

    width: 400px;
    height: 300px;

    background-color: #000;
  }
  .img-map{
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
  canvas {
    width: 100%;
    height: 100%;
  }
  .c-imgEditor-cut {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;

    box-sizing: border-box;

    border: 1px solid rgb(0, 163, 255);
    background-color: rgba(255, 255, 255, .45);

    .c-imgEditor-cut-title {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);

      color: rgb(0, 163, 255);
      font-size: 20px;
      font-weight: bold;
      font-family: Microsoft Yahei;

      cursor: pointer;
    }

    .c-imgEditor-cut-dot {
      position: absolute;

      width: 8px;
      height: 8px;

      border-radius: 50%;
      background-color: rgb(0, 163, 255);

      &:nth-child(1) {
        left: -5px;
        top: -5px;
        cursor: nwse-resize;
      }
      &:nth-child(2) {
        left: 50%;
        top: -5px;
        transform: translateX(-50%);
        cursor: ns-resize;
      }
      &:nth-child(3) {
        right: -5px;
        top: -5px;
        cursor: nesw-resize;
      }
      &:nth-child(4) {
        left: -5px;
        top: 50%;
        transform: translateY(-50%);
        cursor: ew-resize;
      }
      &:nth-child(5) {
        right: -5px;
        top: 50%;
        transform: translateY(-50%);
        cursor: ew-resize;
      }

      &:nth-child(6) {
        left: -5px;
        bottom: -5px;
        cursor: nesw-resize;
      }
      &:nth-child(7) {
        left: 50%;
        bottom: -5px;
        transform: translateX(-50%);
        cursor: ns-resize;
      }
      &:nth-child(8) {
        right: -5px;
        bottom: -5px;
        cursor: nwse-resize;
      }
    }
  }
}
</style>
