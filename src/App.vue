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

// 为了实现延时效果 长出一支后 另一支才开始长
// 递归转堆栈 放入的是 一个个的 闭包函数
const pendingTasks: Function[] = []

// 递归 每一步的操作
const step = (b: Branch) => {
  drawBranch(b)
  const endPoint = getEndPoint(b)
  // 50% 的概率 画左支
  if (Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: endPoint,
      length: b.length,
      theta: b.theta - 0.2,
    }))
  }
  // 50% 的概率 画右支
  if (Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: endPoint,
      length: b.length,
      theta: b.theta + 0.2,
    }))
  }
}

// 遍历执行 pendingTasks 中的任务
const frame = () => {
  // 避免函数执行过程中 影响 step 递归中 pendingTasks 的使用 处理前先把数据浅拷贝 进行处理 紧接着把 pendingTasks 清空
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

// window.requestAnimationFrame() 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行
// 跟 setInterval 类似 只是会自适应刷新

// requestAnimationFrame(() => {
//   frame()
// })

// 递归调用 requestAnimationFrame 刷新
const refreshFrame = () => {
  requestAnimationFrame(() => {
    frame()
    refreshFrame()
  })
}

onMounted(() => {
  init()
  const initBranch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 40,
    theta: -Math.PI / 2,
  }
  step(initBranch)
  // 刷新页面显示
  refreshFrame()
})
</script>

<template>
  <div>
    <canvas ref="el" width="400" height="400" border />
  </div>
</template>
