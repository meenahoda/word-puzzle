<template>
  <q-page class="bg-dark text-center">
    <q-btn label="New" color="secondary" @click="newGame" class="q-ma-xl" />
    <div v-if="goal.length > 0" :class="$q.screen.xs ? 'text-light q-headline' : 'text-light q-display-1'">
      <table style="margin: 0 auto;">
        <tr
          v-for="(row, rowIdx) in grid"
          :key="rowIdx"
        >
          <td
            v-for="(col, colIdx) in row"
            :key="colIdx"
            :class="colIdx === goalX && 'text-red'"
          >
            {{col}}
          </td>
        </tr>
      </table>
    </div>
  </q-page>
</template>

<style>
</style>

<script>
import words from 'assets/words.json'
const getRandom = arr => arr[Math.floor(Math.random() * arr.length)]
const wordsLength8 = words.map(w => w.replace(/-/g, '')).filter(w => w.length === 8)
export default {
  name: 'PageIndex',
  data () {
    return {
      goal: '',
      goalX: 0,
      grid: []
    }
  },
  mounted () { this.newGame() },
  methods: {
    reset () {
      this.goal = ''
      this.goalX = 0
      this.grid = []
    },
    newGame () {
      this.reset()
      const chosen = []
      this.goal = getRandom(wordsLength8)
      this.goal.split('').forEach((letter, idx) => {
        const candidates = words.map(w => w.replace(/-/g, '')).filter(w => w !== this.goal && w.includes(letter))
        const word = getRandom(candidates)
        const start = 0 - word.indexOf(letter)
        const end = start + word.length - 1

        chosen.push({ word, start, end, idx })
      })
      const minStart = Math.min(...chosen.map(w => w.start))
      const maxEnd = Math.max(...chosen.map(w => w.end))
      const width = maxEnd - minStart

      this.goalX = 0 - minStart

      this.goal.split('').forEach((letter, idx) => {
        const row = []
        const candidate = chosen[idx]
        const startAt = this.goalX + candidate.start

        let counter = 0
        for (let x = 0; x <= width; x++) {
          if (x === startAt) {
            row.push(candidate.word[0])
            counter++
          } else if (counter > 0) {
            row.push(candidate.word[counter])
            counter++
          } else {
            row.push(null)
          }
        }
        this.grid.push(row)
      })
    }
  }
}
</script>
