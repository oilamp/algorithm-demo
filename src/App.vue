<template>
  <div class="container">
    <h2>冒泡排序</h2>
    <div class="controls">
      <button @click="handleStart">开始</button>
      <button @click="handlePause">{{ isPaused ? '继续' : '暂停' }}</button>
      <button @click="handleReset">重置</button>
    </div>
    <transition-group 
      name="flip-list" 
      tag="div"
      class="list">
      <div 
        v-for="(item) in data" 
        :key="item" 
        class="block" 
        :class="{'is-active': isActive(item), 'is-sorted' : isSorted(item)}">{{ item }}</div>
    </transition-group>
  </div>
</template>

<script>
import { defineComponent, reactive, toRefs } from "vue";

const generateArr = () => [3, 2, 8, 4, 6, 1, 5 , 7]

export default defineComponent({
  name: "app",
  setup() {
    const timer = 1000
    const controls = reactive({
      isStarted: false,
      isPaused: false,
      continueFn: null,
      data: generateArr(),
      active: [],
      sorted: []
    })

    const swap = (i, j) => {
        const t = controls.data[i]
        controls.data[i] = controls.data[j]
        controls.data[j] = t
    }

    const waiting = () => {
      return new Promise(resolve => {
        setTimeout(() => {
          if (!controls.isStarted) return
          if (!controls.isPaused) return resolve()
          controls.continueFn = resolve
        }, timer)
      })
    }

    const setActive = (a, b) => {
      controls.active = [a, b]
    }

    const clearActive = () => {
      controls.active = []
    }

    const addSorted = (val) => {
      controls.sorted.push(val)
    }

    const isActive = (val) => {
      return controls.active.includes(val)
    }

    const isSorted = (val) => {
      return controls.sorted.includes(val)
    }

    const sort = async () => {

      for (let i = 0;i + 1 < controls.data.length;i++) {
        for (let j = 0;j < controls.data.length - i - 1;j++) {
          setActive(controls.data[j], controls.data[j + 1])
          if (controls.data[j] > controls.data[j + 1]) {
            swap(j, j + 1)
          }
          await waiting()
          clearActive()
        }

        addSorted(controls.data[controls.data.length - i - 1])
      }

      await waiting()
      addSorted(controls.data[0])
    }

    const handleStart = async () => {
      if (controls.isStarted) return
      controls.isStarted = true
      await sort()

      clearActive()
      controls.isStarted = false
    }

    const handlePause = () => {
      if (!controls.isStarted) return

      if (!controls.isPaused) {
        controls.isPaused = true
        return
      }

      controls.isPaused = false
      controls.continueFn()
    }

    const handleReset = () => {
      controls.isStarted = false
      controls.isPaused = false
      controls.continueFn = null
      controls.data = reactive(generateArr())
      controls.active = []
      controls.sorted = []
    }

    return {
      ...toRefs(controls),

      isActive,
      isSorted,
      handleStart,
      handlePause,
      handleReset
    }
  }
});
</script>
<style scoped>
.container{
  display: flex;
  flex-flow: column;
  align-items: center;
}
.controls{
  margin-bottom: 30px;
}
button{
  margin: 0 8px;
}

.list{
  display: flex;
  justify-content: center;
}

.block {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 40px;
  height: 40px;
  border-radius: 3px;
  border: 1px solid #ccc;
  margin: 0 8px;
  background-color: #efefef;
  transition: background-color .5s ease;
}

.block.is-active {
  background-color: aquamarine;
}

.block.is-sorted {
  color: #fff;
  background-color: saddlebrown;
}


.flip-list-move {
  transition: transform 1s;
}
</style>