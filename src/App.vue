<script setup lang="ts">
interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  // 角度
  theta: number
}

const WIDTH = 400
const HEIGHT = 400

const el = $ref<HTMLCanvasElement>()
// ! 是 ts 语法表示肯定存在
const ctx = $computed(() => el!.getContext('2d')!)

const init = () => {
  ctx.strokeStyle = '#000'
}

// 每条分支的结束点 应该是实时算出来的
const getEndPoint = (b: Branch) => {
  return {
    x: b.start.x + b.length * Math.cos(b.theta),
    y: b.start.y + b.length * Math.sin(b.theta),
  }
}

const lineTo = (start: Point, end: Point) => {
  ctx.beginPath()
  ctx.moveTo(start.x, start.y)
  ctx.lineTo(end.x, end.y)
  ctx.stroke()
}

// 画枝条
const drawBranch = (b: Branch) => {
  lineTo(b.start, getEndPoint(b))
}

// 递归 每一步的操作
const step = (b: Branch) => {
  drawBranch(b)
  const endPoint = getEndPoint(b)
  // 50% 的概率 画左支
  if (Math.random() < 0.5) {
    step({
      start: endPoint,
      length: b.length,
      theta: b.theta - 0.2,
    })
  }
  // 50% 的概率 画右支
  if (Math.random() < 0.5) {
    step({
      start: endPoint,
      length: b.length,
      theta: b.theta + 0.2,
    })
  }
}

onMounted(() => {
  init()
  const initBranch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 40,
    theta: -Math.PI / 2,
  }
  step(initBranch)
})
</script>

<template>
  <div>
    <canvas ref="el" width="400" height="400" border />
  </div>
</template>
