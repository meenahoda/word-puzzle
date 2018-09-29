<template>
  <q-page class="bg-dark text-center">
    <q-btn label="New" color="secondary" @click="newGame" class="q-ma-xl" />
    <div v-if="goal.length > 0" :class="$q.screen.xs ? 'text-light q-headline' : 'text-light q-display-1'">
      <table style="margin: 0 auto;">
        <tr
          v-for="(row, rowIdx) in revealed"
          :key="rowIdx"
        >
          <td
            v-for="(col, colIdx) in row"
            :key="colIdx"
            :class="colIdx === goalX ? 'text-red q-px-sm' : 'q-px-sm'"
          >
            <span
              v-if="col"
              @click="open(rowIdx, colIdx, col !== '_')"
              :class="col !== '_' || 'cursor-pointer'"
            >
              {{col}}
            </span>
          </td>
        </tr>
      </table>
    </div>

    <q-dialog
      v-model="choosingLetter"
      :ok="false"
    >
      <div slot="title" class="text-center">Choose a letter</div>
      <div slot="body" class="text-center">
        <div class="q-mb-md">
          <span
            v-for="(letter, idx) in matchingWord"
            :key="idx"
          >
            {{letter}}
          </span>
        </div>
        <div class="q-mb-md">// TODO: Show hint</div>
        <div :style="$q.screen.gt.md ? 'width: 40%; margin: 0 auto; text-align: center;' : 'width: 80%; margin: 0 auto; text-align: center;'">
          <q-btn
            v-for="letter in alphabet.split('')"
            :key="letter"
            :label="letter"
            @click="chosenLetter(letter)"
            color="secondary"
            class="q-mr-md q-mb-md"
          />
        </div>
      </div>
      <template slot="buttons" slot-scope="props">
        <q-btn flat color="secondary" label="Close" @click="props.cancel" />
      </template>
    </q-dialog>
  </q-page>
</template>

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
      grid: [],
      revealed: [],

      choosingLetter: false,
      matchingWord: '',
      matchingLetter: '',
      matchingRow: 0,
      matchingCol: 0,

      alphabet: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    }
  },
  mounted () { this.newGame() },
  methods: {
    open (row, col, matched) {
      if (!matched) {
        this.matchingLetter = this.grid[row][col]
        this.matchingRow = row
        this.matchingCol = col
        this.matchingWord = this.revealed[row].filter(e => e).join('')
        this.choosingLetter = true
      }
    },
    chosenLetter (letter) {
      if (letter === this.matchingLetter.toUpperCase()) {
        this.revealed[this.matchingRow][this.matchingCol] = letter
        this.choosingLetter = false
        this.matchingWord = ''
        this.matchingLetter = ''
        this.matchingRow = 0
        this.matchingCol = 0
      } else {
        this.$q.notify({
          type: 'negative',
          message: 'Nope'
        })
      }
    },
    reset () {
      this.goal = ''
      this.goalX = 0
      this.grid = []
      this.revealed = []
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
        this.revealed.push(row.map(e => e && '_'))
      })
    }
  }
}
</script>
