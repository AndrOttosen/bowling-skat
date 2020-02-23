<template>
  <div>
    <ul v-for="(points, i) in games.points" :key="points.id">
      <li>{{points}} {{calcPoints[i]}}</li>
    </ul>

    <ul>
      <li>{{result}}</li>
      <li>{{games.token}}</li>
    </ul>

    <button @click="validateResult()">Validér resultat</button>
    <p v-if="success !== null">Success Check {{success}}</p>
  </div>
</template>


<script>
import axios from 'axios'

export default {
  data() {
    return {
      games: {
        points: []
      },
      calcPoints: [],
      result: null,
      success: null
    }
  },
  mounted() {
    axios.get('http://13.74.31.101/api/points').then(response => {
      this.games = response.data
      let points = []
      this.games.points.forEach((element, i) => {
        let sum = element.reduce((acc, curr) => acc + curr, 0)
        sum = this.bonus(sum, i)
        if (points.length === 0 && sum !== 10) {
          points.push(sum)
        } else if (points[i - 1]) {
          sum += points[i - 1]
          points.push(sum)
        } else {
          sum += points[i - 1]
          points.push(sum)
        }
      })
      console.log('Hello')
      if (points.length > 10) {
        points.pop()
      }
      console.log(points)
      this.calcPoints = points
      this.result = points[points.length - 1]
    })
  },
  methods: {
    bonus(sum, i) {
      // comment

      if (i + 1 < this.games.points.length) {
        if (this.games.points[i][0] === 10) {
          if (this.games.points[i + 1][0] === 10) {
            sum += this.games.points[i + 1].reduce((acc, curr) => acc + curr, 0)
            if (this.games.points[i + 2]) {
              if (this.games.points[i + 2][0] === 10) {
                sum += this.games.points[i + 2][0]
              } else {
                sum += this.games.points[i + 2][0]
              }
            }
          } else {
            sum += this.games.points[i + 1].reduce((acc, curr) => acc + curr, 0)
          }
        } else if (this.games.points[i + 1] && sum === 10) {
          sum += this.games.points[i + 1][0]
        }
      }
      return sum
    },
    validateResult() {
      axios
        .post('http://13.74.31.101/api/points', {
          token: this.games.token,
          points: this.calcPoints
        })
        .then(response => {
          this.success = response.data.success
          console.log(response.data.success)
        })
    }
  }
}
</script>
