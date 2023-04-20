<script setup>
import { onMounted, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
const onClickLeft = () => history.back()
const route = useRoute()
const router = useRouter()

let canvas = null
let context = null
const touchPressed = ref(false)
const lastX = ref()
const lastY = ref()

// 初始化获取canvas
onMounted(() => {
  canvas = document.getElementById('myCanvas')
  canvas.width = document.body.clientWidth * 0.8
  canvas.height = document.body.clientHeight * 0.75
  context = canvas.getContext('2d')
  GetLandName()
})

// 根据权利人姓名进行画图
const GetLandName = () => {
  const canvas = document.getElementById('my').getContext('2d')
  const winW = document.body.clientWidth * 0.8
  const winH = document.body.clientHeight * 0.75
  document.getElementById('my').width = winH
  document.getElementById('my').height = winW
  canvas.font = '140px Arial'
  canvas.strokeText('测试', winH / 9.6, winW / 1.8)
  rotateBase64Img(canvas.canvas.toDataURL(), 90)
}

// 旋转base64图像
const rotateBase64Img = (src, edg) => {
  const canvas = document.createElement('canvas') // 创建canvas
  const ctx = canvas.getContext('2d')
  if (edg % 90 != 0) {
    console.error('旋转角度必须是90的倍数!')
    throw '旋转角度必须是90的倍数!'
  }
  edg < 0 && (edg = (edg % 360) + 360)
  const quadrant = (edg / 90) % 4 //旋转象限
  const cutCoor = { sx: 0, sy: 0, ex: 0, ey: 0 } //裁剪坐标
  const image = new Image()
  image.crossOrigin = 'anonymous'
  image.src = src
  image.onload = function () {
    const imgW = image.width //图片宽度
    const imgH = image.height //图片高度
    const size = imgW > imgH ? imgW : imgH //canvas初始大小
    canvas.width = size * 2
    canvas.height = size * 2
    switch (quadrant) {
      case 0:
        cutCoor.sx = size
        cutCoor.sy = size
        cutCoor.ex = size + imgW
        cutCoor.ey = size + imgH
        break
      case 1:
        cutCoor.sx = size - imgH
        cutCoor.sy = size
        cutCoor.ex = size
        cutCoor.ey = size + imgW
        break
      case 2:
        cutCoor.sx = size - imgW
        cutCoor.sy = size - imgH
        cutCoor.ex = size
        cutCoor.ey = size
        break
      case 3:
        cutCoor.sx = size
        cutCoor.sy = size - imgW
        cutCoor.ex = size + imgH
        cutCoor.ey = size + imgW
        break
    }
    ctx.translate(size, size) // 重新映射画布上的 (0,0) 位置
    ctx.rotate((edg * Math.PI) / 180) // 旋转画布
    ctx.drawImage(image, 0, 0) // 绘制图像
    const imgData = ctx.getImageData(cutCoor.sx, cutCoor.sy, cutCoor.ex, cutCoor.ey)
    if (quadrant % 2 == 0) {
      canvas.width = imgW
      canvas.height = imgH
    } else {
      canvas.width = imgH
      canvas.height = imgW
    }
    ctx.putImageData(imgData, 0, 0) // 将图像数据（从指定的 ImageData 对象）放回画布上
    divs.style.backgroundImage = 'url(' + canvas.toDataURL() + ')'
  }
}

// 清空画布
const clearArea = () => {
  context.setTransform(1, 0, 0, 1, 0, 0) // setTransform() 允许您缩放、旋转、移动并倾斜当前的环境
  context.clearRect(0, 0, context.canvas.width, context.canvas.height)
  GetLandName()
}

// 提交图片
const saveImageInfo = () => {
  if (isCanvasBlank(canvas)) return alert('请绘制')
  rotateBase64(canvas.toDataURL())
}

const rotateBase64 = data => {
  const imgView = new Image()
  imgView.src = data
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  const cutCoor = { sx: 0, sy: 0, ex: 0, ey: 0 } // 裁剪坐标
  imgView.onload = () => {
    const imgW = imgView.width
    const imgH = imgView.height
    const size = imgH
    canvas.width = size * 2
    canvas.height = size * 2
    cutCoor.sx = size
    cutCoor.sy = size - imgW
    cutCoor.ex = size + imgH
    cutCoor.ey = size + imgW
    context.translate(size, size)
    context.rotate((Math.PI / 2) * 3)
    context.drawImage(imgView, 0, 0)
    const imgData = context.getImageData(cutCoor.sx, cutCoor.sy, cutCoor.ex, cutCoor.ey)
    canvas.width = imgH
    canvas.height = imgW
    context.putImageData(imgData, 0, 0)
    let a = canvas.toDataURL('image/png')
    console.log('File对象', a)
    router.push({ path: 'signConfirm', query: { contractor_code: route.query.contractor_code, base64: a } })
    /* var arr = a.split(','),
      mime = arr[0].match(/:(.*?);/)[1],
      bstr = atob(arr[1]),
      n = bstr.length,
      u8arr = new Uint8Array(n)
    while (n--) {
      u8arr[n] = bstr.charCodeAt(n)
    }
    let b = new File([u8arr], '签名照', {
      type: mime
    }) */
  }
}
//判断当前画板是否为空
const isCanvasBlank = canvas => {
  var blank = document.createElement('canvas')
  blank.width = canvas.width
  blank.height = canvas.height
  return canvas.toDataURL() == blank.toDataURL()
}

// 手指点击
const beginDraw = event => {
  event.preventDefault()
  const touch = event.targetTouches[0]
  touchPressed.value = true
  draw(touch.pageX - canvas.offsetLeft, touch.pageY - canvas.offsetTop - 48, false)
}
// 手指移动
const drawing = event => {
  event.preventDefault()
  divs.style.backgroundImage = ''
  const touch = event.targetTouches[0]
  if (touchPressed.value) {
    draw(touch.pageX - canvas.offsetLeft, touch.pageY - canvas.offsetTop - 48, true)
  }
}
// 手指松开
const endDraw = event => {
  event.preventDefault()
  touchPressed.value = false
}
// 签名
const draw = (x, y, isDown) => {
  if (isDown) {
    context.beginPath()
    context.strokeStyle = '#000000'
    context.lineWidth = 4
    context.lineJoin = 'round'
    context.moveTo(lastX.value, lastY.value)
    context.lineTo(x, y)
    context.closePath()
    context.stroke()
  }
  lastX.value = x
  lastY.value = y
}
</script>

<template>
  <div>
    <van-nav-bar title="手写签名" @click-left="onClickLeft">
      <template #left>
        <van-icon name="arrow-left" color="black" />
      </template>
    </van-nav-bar>

    <div class="canvas">
      <div class="canvas_canvas" id="divs">
        <canvas id="myCanvas" @touchstart="beginDraw($event)" @touchmove="drawing($event)" @touchend="endDraw($event)"></canvas>
      </div>

      <div class="bottom">
        <div class="canvas_info">权利人: <span class="name">测试</span></div>
        <div class="canvas_footer">
          <div @click="clearArea" class="canvas_footer_re">重写</div>
          <div @click="onClickLeft" class="canvas_footer_sa">返回</div>
          <div @click="saveImageInfo" class="canvas_footer_su">提交</div>
        </div>
      </div>

      <canvas id="my" v-show="false"></canvas>
    </div>
  </div>
</template>

<style scoped lang="scss">
.canvas {
  width: 100%;
  height: calc(100vh - 48px);
  text-align: center;
  .canvas_canvas {
    width: 350px;
    margin: 0 auto;
    position: relative;

    #myCanvas {
      border: 2px solid rgb(119, 119, 119);
      margin-top: 15px;
      border-radius: 4px;
    }
  }
}
.bottom {
  position: relative;

  .canvas_footer,
  .canvas_info {
    transform: rotate(90deg);
    -ms-transform: rotate(90deg);
    -moz-transform: rotate(90deg);
    -webkit-transform: rotate(90deg);
    -o-transform: rotate(90deg);
  }
  .canvas_info {
    position: absolute;
    top: 50px;
    right: 0;
    font-weight: bold;
    margin: 0 20px;
    font-size: 16px;
    .name {
      color: rgb(243, 67, 67);
      font-size: 17px;
    }
  }
  .canvas_footer {
    position: absolute;
    left: 30%;
    display: flex;
    flex-flow: column;
    line-height: 30px;
    text-align: center;
    color: white;
    font-weight: bold;
    font-size: 16px;
    .canvas_footer_re {
      width: 100px;
      background: #f11919;
      border-radius: 10px;
      margin-bottom: 15px;
    }
    .canvas_footer_sa {
      width: 100px;
      background: #1989f1;
      border-radius: 10px;
      margin-bottom: 15px;
    }
    .canvas_footer_su {
      width: 100px;
      background: #2dcb7a;
      border-radius: 10px;
      margin-bottom: 15px;
    }
  }
}
</style>
