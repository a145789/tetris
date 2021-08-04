<template>
  <div class="box">
    <div v-for="line of spaceListComp" class="line">
      <div v-for="lattice of line" :class="`bg ${lattice.bg}`" />
    </div>
  </div>
</template>

<script lang="ts">
import { reactive, defineComponent, onMounted, computed } from 'vue'

const SIZE = 20
const BOX_HIGHT = 700 / SIZE
const BOX_WIDTH = 400 / SIZE
const MAX_Y = BOX_HIGHT - 1
const NORMAL_SPEED = 300

enum Arrow {
  ArrowUp = 'ArrowUp',
  ArrowLeft = 'ArrowLeft',
  ArrowDown = 'ArrowDown',
  ArrowRight = 'ArrowRight'
}
enum Grid {
  Y,
  X
}
enum Computed {
  add,
  cut,
  other
}

interface Attr {
  name: string
  bg: string
  shape: [number, number][]
}

function genSpace(): Attr {
  const poor: Attr[] = [
    {
      name: 'space',
      bg: 'bg-f1707d',
      shape: [
        [BOX_HIGHT - 1, 1],
        [BOX_HIGHT - 1, 2],
        [BOX_HIGHT - 2, 1],
        [BOX_HIGHT - 2, 2]
      ]
    },
    {
      name: 'left-l',
      bg: 'bg-f15536',
      shape: [
        [BOX_HIGHT - 1, 1],
        [BOX_HIGHT - 2, 1],
        [BOX_HIGHT - 2, 2],
        [BOX_HIGHT - 2, 3],
        [BOX_HIGHT - 2, 4]
      ]
    },
    {
      name: 'right-l',
      bg: 'bg-ddff95',
      shape: [
        [BOX_HIGHT - 1, 4],
        [BOX_HIGHT - 2, 1],
        [BOX_HIGHT - 2, 2],
        [BOX_HIGHT - 2, 3],
        [BOX_HIGHT - 2, 4]
      ]
    },
    {
      name: 'tanke',
      bg: 'bg-f1b8f1',
      shape: [
        [BOX_HIGHT - 1, 2],
        [BOX_HIGHT - 2, 1],
        [BOX_HIGHT - 2, 2],
        [BOX_HIGHT - 2, 3]
      ]
    },
    {
      name: 'left-z',
      bg: 'bg-b8f1ed',
      shape: [
        [BOX_HIGHT - 1, 1],
        [BOX_HIGHT - 1, 2],
        [BOX_HIGHT - 2, 2],
        [BOX_HIGHT - 2, 3]
      ]
    },
    {
      name: 'right-z',
      bg: 'bg-f3d751',
      shape: [
        [BOX_HIGHT - 1, 2],
        [BOX_HIGHT - 1, 3],
        [BOX_HIGHT - 2, 1],
        [BOX_HIGHT - 2, 2]
      ]
    },
    {
      name: 'line',
      bg: 'bg-ac5e74',
      shape: [
        [BOX_HIGHT - 1, 1],
        [BOX_HIGHT - 1, 2],
        [BOX_HIGHT - 1, 3],
        [BOX_HIGHT - 1, 4]
      ]
    }
  ]
  return poor[Math.floor(Math.random() * poor.length)]
}
export default defineComponent({
  name: 'HelloWorld',
  setup: () => {
    const spaceList = reactive(
      Array.from<number>({ length: BOX_HIGHT })
        .fill(0)
        .map(() =>
          Array.from<{ bg: string }>({ length: BOX_WIDTH }).map(() => ({
            bg: ''
          }))
        )
    )
    const spaceListComp: typeof spaceList = []
    for (let i = 0; i < spaceList.length; i++) {
      spaceListComp[MAX_Y - i] = spaceList[i]
    }

    let timer: number
    let { shape, bg, name } = genSpace()

    const getBoundary = (
      copyShape: Attr['shape'],
      index: Grid,
      judge: (a: number, b: number) => boolean
    ): boolean => {
      const sectionIdx = [Grid.Y, Grid.X].find(g => g !== index)!
      return copyShape
        .reduce<Attr['shape']>((p, c) => {
          const idx = p.findIndex(g => g[sectionIdx] === c[sectionIdx])
          if (idx !== -1) {
            if (judge(p[idx][index], c[index])) {
              p.splice(idx, 1, c)
            }
          } else {
            p.push(c)
          }
          return p
        }, [])
        .some(([y, x]) => spaceList[y][x].bg !== '')
    }
    function positionHandle<T extends Grid>(
      computed: Extract<Computed, Computed.other>
    ): boolean
    function positionHandle<T extends Grid>(
      computed: Exclude<Computed, Computed.other>,
      index: T,
      judge: (n: number) => boolean,
      boundaryJudge: (a: number, b: number) => boolean
    ): boolean
    function positionHandle(
      computed: any,
      index?: any,
      judge?: any,
      boundaryJudge?: any
    ): boolean {
      const copyShape: Attr['shape'] = JSON.parse(JSON.stringify(shape))

      if (computed !== Computed.other) {
        for (let i = 0, len = shape.length; i < len; i++) {
          if (computed === Computed.cut) {
            copyShape[i][index]--
          } else {
            copyShape[i][index]++
          }
        }

        if (
          copyShape.some(grid => judge(grid[index])) ||
          getBoundary(copyShape, index, boundaryJudge)
        ) {
          return true
        }
      } else {
        const [axisY, axisX] = shape[Math.ceil(copyShape.length / 2)]
        for (let i = 0, len = shape.length; i < len; i++) {
          const [y, x] = shape[i]
          if (y >= axisY) {
            copyShape[i][Grid.X] = axisX - (y - axisY)
          }
          if (y <= axisY) {
            copyShape[i][Grid.X] = axisX + axisY - y
          }
          if (x >= axisX) {
            copyShape[i][Grid.Y] = axisY + x - axisX
          }

          if (x <= axisX) {
            copyShape[i][Grid.Y] = axisY - (axisX - x)
          }
        }
        if (
          copyShape.some(
            ([y, x]) => y < 0 || y === BOX_HIGHT || x < 0 || x === BOX_WIDTH
          ) ||
          copyShape
            .reduce<Attr['shape']>((p, c) => {
              if (!shape.some(([y, x]) => y === c[Grid.Y] && x === c[Grid.X])) {
                p.push(c)
              }
              return p
            }, [])
            .some(([y, x]) => spaceList[y][x].bg !== '')
        ) {
          return false
        }
      }

      intervalHandle(copyShape)
      return false
    }
    const intervalHandle = (copyShape = shape) => {
      for (const [y, x] of shape) {
        spaceList[y][x].bg = ''
      }
      for (const [y, x] of copyShape) {
        spaceList[y][x].bg = bg
      }
      shape = copyShape
    }
    const downHandle = () => {
      if (
        positionHandle(
          Computed.cut,
          Grid.Y,
          n => n < 0,
          (a, b) => a > b
        )
      ) {
        for (let i = 0; i < spaceList.length; i++) {
          const line = spaceList[i]
          if (line.every(lattice => lattice.bg !== '')) {
            for (let l = i; l < spaceList.length; l++) {
              const aboveLine = spaceList[l + 1]
              spaceList[l].forEach((lattice, index) => {
                lattice.bg =
                  l + 1 !== spaceList.length ? aboveLine[index].bg : ''
              })
            }
            i--
          }
        }
        ;({ shape, bg, name } = genSpace())
      }
    }

    onMounted(() => {
      window.addEventListener('keydown', ({ key }) => {
        if (!(key in Arrow)) {
          return
        }
        clearInterval(timer)
        switch (key) {
          case Arrow.ArrowUp:
            if (name === 'space') {
              return
            }
            positionHandle(Computed.other)
            break
          case Arrow.ArrowLeft:
            positionHandle(
              Computed.cut,
              Grid.X,
              n => n < 0,
              (a, b) => a > b
            )
            break
          case Arrow.ArrowDown:
            downHandle()
            break
          case Arrow.ArrowRight:
            positionHandle(
              Computed.add,
              Grid.X,
              n => n === BOX_WIDTH,
              (a, b) => a < b
            )
            break

          default:
            break
        }
      })

      window.addEventListener('keyup', ({ key }) => {
        if (!(key in Arrow)) {
          return
        }
        switch (key) {
          case Arrow.ArrowUp:
            break
          case Arrow.ArrowLeft:
            break
          case Arrow.ArrowDown:
            break
          case Arrow.ArrowRight:
            break

          default:
            break
        }
        timer = setInterval(downHandle, NORMAL_SPEED)
      })

      intervalHandle()
      timer = setInterval(downHandle, NORMAL_SPEED)
    })

    return {
      spaceListComp,
      SIZE,
      BOX_HIGHT,
      BOX_WIDTH
    }
  }
})
</script>

<style>
.box {
  height: 700px;
  width: 400px;
  margin: 20px;
  border: 2px solid #444;
  border-top: none;
  position: relative;
}
.box .line {
  height: v-bind(SIZE + 'px');
  display: flex;
  flex-wrap: wrap;
}
.box .line div {
  width: v-bind(SIZE + 'px');
  height: 100%;
  box-sizing: border-box;
  padding: 2px;
}
.bg {
  background-clip: content-box;
}
.bg-f1707d {
  background-color: #f1707d;
  border: 2px solid #f1707d;
}
.bg-f15536 {
  background-color: #f15536;
  border: 2px solid #f15536;
}
.bg-ddff95 {
  background-color: #ddff95;
  border: 2px solid #ddff95;
}
.bg-f1b8f1 {
  background-color: #f1b8f1;
  border: 2px solid #f1b8f1;
}
.bg-b8f1ed {
  background-color: #b8f1ed;
  border: 2px solid #b8f1ed;
}
.bg-f3d751 {
  background-color: #f3d751;
  border: 2px solid #f3d751;
}
.bg-ac5e74 {
  background-color: #ac5e74;
  border: 2px solid #ac5e74;
}
</style>
