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
// 为了 防止 长的过于单调 加入 树深度 depth 控制  保证其至少可以达到某个深度 枝繁叶茂
const step = (b: Branch, depth = 0) => {
  drawBranch(b)
  const endPoint = getEndPoint(b)
  // 50% 的概率 画左支
  if (depth < 3 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: endPoint,
      // (Math.random() * 10 - 5) 生成 -5 到 5 的随机数
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta - 0.3 * Math.random(),
    }, depth + 1))
  }
  // 50% 的概率 画右支
  if (depth < 3 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: endPoint,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta + 0.3 * Math.random(),
    }, depth + 1))
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
// 显示还是太快 设置计数器 控制刷新显示速度
let frameCount = 0
const refreshFrame = () => {
  requestAnimationFrame(() => {
    frameCount++
    // 设置 每3帧 画一次
    if (frameCount % 5 === 0)
      frame()

    refreshFrame()
  })
}

onMounted(() => {
  init()
  const initBranch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 30,
    theta: -Math.PI / 2,
  }
  step(initBranch)
  // 刷新页面显示
  refreshFrame()
})
</script>

<template>
  <div flex="~" h-full justify-center items-center>
    <canvas ref="el" width="400" height="400" border />
  </div>
</template>
