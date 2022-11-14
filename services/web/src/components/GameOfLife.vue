<script setup lang="ts">
import { ComponentPublicInstance } from 'vue'
import init, { Universe, Cell } from 'wasm-game-of-life'

const CELL_SIZE = 12 // px
const GRID_COLOR = '#CCCCCC'
const DEAD_COLOR = '#FFFFFF'
const ALIVE_COLOR = '#000000'

const canvasRef = async (el: Element | null | ComponentPublicInstance) => {
  const { memory } = await init()

  if (!el) {
    return
  }

  const canvas = el as HTMLCanvasElement

  const universe = Universe.new()

  const width = universe.width()
  const height = universe.height()

  canvas.height = (CELL_SIZE + 1) * height + 1
  canvas.width = (CELL_SIZE + 1) * width + 1

  const ctx = canvas.getContext('2d')

  if (!ctx) {
    return
  }

  const drawGrid = () => {
    ctx.beginPath()
    ctx.strokeStyle = GRID_COLOR

    // Vertical lines
    for (let i = 0; i <= width; i++) {
      ctx.moveTo(i * (CELL_SIZE + 1) + 1, 0)
      ctx.lineTo(i * (CELL_SIZE + 1) + 1, (CELL_SIZE + 1) * height + 1)
    }

    // Horizontal lines
    for (let j = 0; j <= height; j++) {
      ctx.moveTo(0, j * (CELL_SIZE + 1) + 1)
      ctx.lineTo((CELL_SIZE + 1) * width + 1, j * (CELL_SIZE + 1) + 1)
    }

    ctx.stroke()
  }

  const getIndex = (row: number, column: number) => row * width + column

  const drawCells = () => {
    const cellsPtr = universe.cells()
    const cells = new Uint8Array(memory.buffer, cellsPtr, width * height)

    ctx.beginPath()

    for (let row = 0; row < height; row++) {
      for (let col = 0; col < width; col++) {
        const idx = getIndex(row, col)

        ctx.fillStyle = cells[idx] === Cell.Dead ? DEAD_COLOR : ALIVE_COLOR

        ctx.fillRect(
          col * (CELL_SIZE + 1) + 1,
          row * (CELL_SIZE + 1) + 1,
          CELL_SIZE,
          CELL_SIZE
        )
      }
    }

    ctx.stroke()
  }

  const renderLoop = () => {
    universe.tick()

    drawGrid()
    drawCells()

    requestAnimationFrame(renderLoop)
  }

  drawGrid()
  drawCells()
  requestAnimationFrame(renderLoop)
}
</script>

<template>
  <canvas :ref="canvasRef"></canvas>
</template>

<style scoped></style>
