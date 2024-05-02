<template>
  <div class="container">
    <div class="content">
      <canvas class="field"></canvas>
      <div class="start_button" @click="startButtonClick()">Новый путь</div>
    </div>

    <div class="status_bar">
      <p class="heading">Информация</p>

      <div class="status_items">
        <transition-group name="status_items">
          <status-item v-for="path in paths_status" :key="path.id" :path="path" />
        </transition-group>
      </div>
      <div class="empty-text" v-show="!paths_status.length">Начните новый путь!</div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import StatusItem from './components/StatusItem.vue';
import { ICeil, IPathInfo } from './types';


export default defineComponent({
  name: 'App',
  components: {
    StatusItem
  },

  data() {

    return {
      field: [] as ICeil[],
      paths: 1 as number,
      start_point: { x: 0, y: 0, wall: false } as ICeil,
      end_point: { x: 19, y: 19, wall: false } as ICeil,
      current_point: { x: 0, y: 0, wall: false } as ICeil,
      paths_status: [] as IPathInfo[],
      start_button_clicked: false as boolean,
      size_cell: 20 as number,
      result: [] as ICeil[],
      savetime: 0 as number,
      // eslint-disable-next-line
      ctx: null as any,
      // eslint-disable-next-line
      intervalTimer: null as any
    }

  },

  methods: {

    getNeighbours(x: number, y: number): ICeil[] {
      let arr: ICeil[] = []
      if (this.field.length) {
        if (x - 1 >= 0) arr.push(this.field[(x - 1) * 20 + y]) // left
        if (x + 1 < 20) arr.push(this.field[(x + 1) * 20 + y]) // right
        if (y - 1 >= 0) arr.push(this.field[x * 20 + y - 1]) // top
        if (y + 1 < 20) arr.push(this.field[x * 20 + y + 1]) // bottom
      }
      return arr
    },
    async drawField(): Promise<void> {
      for (let i = 0; i < this.size_cell; i++) {
        for (let j = 0; j < this.size_cell; j++) {
          this.field.push({
            x: i,
            y: j,
            wall: false,
          })
          this.ctx.strokeRect(i * this.size_cell, j * this.size_cell, this.size_cell, this.size_cell)
        }
      }
    },
    async generateWalls(): Promise<void> {
      for (let i = 0; i < this.size_cell; i++) {
        for (let j = 0; j < this.size_cell; j++) {
          this.field[i * this.size_cell + j].wall = false
          if (Math.random() > 0.95) this.field[i * this.size_cell + j].wall = true
          if (i * this.size_cell + j === 0 || i * this.size_cell + j === 399) this.field[i * this.size_cell + j].wall = false
        }
      }
    },
    async updateField(): Promise<void> {
      for (let i = 0; i < this.size_cell; i++) {
        for (let j = 0; j < this.size_cell; j++) {
          this.ctx.strokeRect(i * this.size_cell, j * this.size_cell, this.size_cell, this.size_cell)
          if (i === this.start_point.x && j === this.start_point.y) this.ctx.fillStyle = "#59dd58"
          else if (i === this.end_point.x && j === this.end_point.y) this.ctx.fillStyle = "#dd5558"
          else if (this.field[i * this.size_cell + j].wall) this.ctx.fillStyle = "#292528"
          else this.ctx.fillStyle = "#dcdfd6"

          this.ctx.fillRect(i * this.size_cell, j * this.size_cell, this.size_cell, this.size_cell)
        }
      }
    },
    async findPath(): Promise<void> {
      let neighbours: ICeil[] = this.getNeighbours(this.current_point.x, this.current_point.y)
      let arr = neighbours.filter((item) => !item.wall && item.x >= this.current_point.x && item.y >= this.current_point.y)
      let arr_help = neighbours.filter((item) => !item.wall && item.x <= this.current_point.x && item.y <= this.current_point.y)

      const rand = Math.floor(Math.random() * arr.length)
      const next = arr.splice(rand, 1)[0]
      if (next) {
        if (next.x === this.end_point.x && next.y === this.end_point.y) return
        this.result.push(next)
        this.current_point = next
        this.findPath()
      } else {
        if ((this.current_point.x === this.end_point.x - 1 && this.current_point.y === this.end_point.y) || (this.current_point.x === this.end_point.x && this.current_point.y === this.end_point.y - 1)) return
        const rand_help = Math.floor(Math.random() * arr_help.length)
        const next_help = arr_help.splice(rand_help, 1)[0]
        this.result.push(next_help)
        this.current_point = next_help
        this.findPath()
      }


    },
    async drawPath(): Promise<void> {
      const r = Math.floor(Math.random() * 255).toString(16)
      const g = Math.floor(Math.random() * 255).toString(16)
      const b = Math.floor(Math.random() * 255).toString(16)
      this.ctx.fillStyle = "#"
        + (r.length === 1 ? "0" + r : r)
        + (g.length === 1 ? "0" + g : g)
        + (b.length === 1 ? "0" + b : b)
        + "aa"
      await this.findPath()
      for (let i = 0; i < this.result.length; i++) {
        let timer = setTimeout(() => {
          this.ctx.fillRect(this.result[i].x * this.size_cell, this.result[i].y * this.size_cell, this.size_cell, this.size_cell)
          this.paths_status[0].steps++
          clearTimeout(timer)
          if (i === this.result.length - 1) {
            clearInterval(this.intervalTimer)
            this.savetime = 0
            this.start_button_clicked = false
            this.paths += 1
          }
        }, i * 50)
      }
    },
    async startButtonClick(): Promise<void> {
      if (this.start_button_clicked) return
      this.paths_status.unshift({
        id: this.paths_status.length + 1,
        steps: 0,
        time: 0
      })

      this.savetime = new Date().getTime()

      this.intervalTimer = setInterval(() => {
        this.paths_status[0].time = (new Date().getTime()) - this.savetime
      }, 0)

      this.result = []
      this.start_point = { x: 0, y: 0, wall: false }
      this.end_point = { x: 19, y: 19, wall: false }
      this.current_point = { x: 0, y: 0, wall: false }
      this.start_button_clicked = true
      await this.drawPath()

    }
  },
  async mounted() {
    
    const canvas: HTMLCanvasElement = document.querySelector(".field") as HTMLCanvasElement
    canvas.width = 400
    canvas.height = 400
    this.ctx = canvas.getContext("2d") as CanvasRenderingContext2D

    await this.drawField()
    await this.generateWalls()
    await this.updateField()
  },
});
</script>

<style lang="scss">@import url('./style.scss');</style>./types