<template>
  <div class="main" :style="{
    '--w': args.width,
    '--h': args.height,
    '--color': args.color,
    '--bg': args.bg,
    '--mine-color': args.mineColor
  }">
    <div class="player">
      <div class="line" v-for="i in args.height" :key="i">
        <k-item @click="onClick(i - 1, j - 1)" v-for="j in args.width"
          @contextmenu="($event.preventDefault(), onRightClick(i - 1, j - 1))" :x="i - 1" :y="j - 1" :map="map" />
      </div>
    </div>
  </div>
</template>
<script setup>
import Color from "color";
import KItem from "./KItem.vue"
import { reactive, ref, watchEffect } from "vue"
const nowUrl = new URL(window.location.href)
const searchParams = (name, parser, decider) =>
  decider(parser(nowUrl.searchParams.get(name), 10))

const args = reactive({
  width: searchParams("w", parseInt, (value) => value > 1 ? value : 16),
  height: searchParams("h", parseInt, (value) => value > 1 ? value : 16),
  percent: searchParams("p", parseFloat, (value) => (value < 1) && (value > 0) ? value : 0.15),
  color: searchParams("c", a => a, (value) => {
    try {
      return Color(value || "1")
    } catch {
      return Color("#a8bfff")
    }
  }),

})
watchEffect(() => {
  args.color
  args.bg = (() =>
    searchParams("b", a => a, (value) => {
      try {
        return Color(value || "1")
      } catch {
        return args.color.isLight() ? Color("#000") : Color("#fff")
      }
    })
  )()
  args.mineColor = (() =>
    searchParams("m", a => a, (value) => {
      try {
        return Color(value || "1")
      } catch {
        return Color("#f55")
      }
    })
  )()
})

window.color = Color
const map = ref()
const inited = ref(false)
watchEffect(() => {
  map.value = Array.from({ length: args.height }, (_, i) => Array.from({ length: args.width }, (_, j) => ({
    value: undefined,
    tag: undefined,
  })))
  inited.value = false
})
function init() {
  for (const i of map.value) {
    for (const j of i) {
      if (j.value === undefined) {
        j.value = Math.random() < args.percent
      }
    }
  }
  function getValue(x, y) {
    return map.value[x]&&map.value[x][y]&&map.value[x][y].value ? 1 : 0
  }
  for (let i=0;i < map.value.length;i++) {
    for (let j=0;j < map.value[i].length;j++) {
      map.value[i][j].showValue=(
        getValue(i - 1, j - 1) +
        getValue(i - 1, j) +
        getValue(i - 1, j + 1) +
        getValue(i, j - 1) +
        getValue(i, j + 1) +
        getValue(i + 1, j - 1) +
        getValue(i + 1, j) +
        getValue(i + 1, j + 1)
      )
    }
  }
  inited.value = true
}
function onClick(x, y) {
  console.log("click", x, y)
  if (!inited.value) {
    function setValue(x, y,value) {
      try{
        map.value[x][y].value=value
      }catch{}
    }
    setValue(x, y, false)
    setValue(x - 1, y, false)
    setValue(x + 1, y, false)
    setValue(x, y - 1, false)
    setValue(x, y + 1, false)
    setValue(x - 1, y - 1, false)
    setValue(x - 1, y + 1, false)
    setValue(x + 1, y - 1, false)
    setValue(x + 1, y + 1, false)
    init()
  }
  map.value[x][y].tag = 1
  if(map.value[x][y].showValue===0&&!map.value[x][y].value){
    const searched=new WeakSet()
    function dfs(x, y) {
      if (x < 0 || x >= map.value.length || y < 0 || y >= map.value[x].length) {
        return
      }
      if (searched.has(map.value[x][y])) {
        return
      }
      searched.add(map.value[x][y])
      if (map.value[x][y].tag === 2) {
        return
      }
      if (map.value[x][y].value) {
        return
      }
      map.value[x][y].tag = 1
      if(map.value[x][y].showValue===0){
        dfs(x - 1, y)
        dfs(x + 1, y)
        dfs(x, y - 1)
        dfs(x, y + 1)
        dfs(x - 1, y - 1)
        dfs(x - 1, y + 1)
        dfs(x + 1, y - 1)
        dfs(x + 1, y + 1)
      }
    }
    dfs(x, y)
  }
  checkStatus()
}
function onRightClick(x, y) {
  console.log("right click", x, y)
  if (!inited.value) {
    return
  }
  map.value[x][y].tag = 2
  checkStatus()
}
function onLoss(){
  alert("FAIL")
  location.reload()
}
function onWin(){
  alert("WIN")
  location.reload()
}
function checkStatus() {
  let flag=true;
  for (const i of map.value) {
    for (const j of i) {
      if (j.tag === 1&&j.value) {
        setTimeout(onLoss,17)
      }
      flag =flag && ((j.value && j.tag === 2) || (!j.value && j.tag === 1))
    }
  }
  if (flag) {
    setTimeout(onWin,17)
  }
}
</script>
<style scoped lang="scss">
.main {
  position: fixed;
  inset: 0;
  background-color: var(--bg);
  display: flex;
  justify-content: center;
  align-items: center;
  --size: calc(min(70vh / var(--h), 70vw / var(--w)));
  --gap: calc(min(10vh / (var(--h) - 1), 10vw / (var(--w) - 1)));
}

.player {
  display: flex;
  flex-direction: column;
  flex-grow: 0;
  flex-shrink: 0;

  row-gap: var(--gap);

  &>.line {
    display: flex;
    flex-grow: 0;
    flex-shrink: 0;
    column-gap: var(--gap);
  }
}
</style>