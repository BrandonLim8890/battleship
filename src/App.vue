<template>
  <div id="app">
    Computer Board, you attack on this
    <div class="row-wrapper">
      <div v-for="rows in grid" :key="rows.id" class="column-wrapper">
        <div
          v-for="pos in rows"
          :key="pos.id"
          v-on:click="game.playTurn({ x: pos.x, y: pos.y })"
          class="grid player-grid"
          v-bind:class="{
            ship: computer1.board.ships.some((ship) =>
              ship.positions.some(
                (position) => position.x == pos.x && position.y == pos.y
              )
            ),

            miss: computer1.board.missedHits.some(
              (position) => position.x == pos.x && position.y == pos.y
            ),

            hit: computer1.board.ships.some((ship) =>
              ship.hitPositions.some(
                (position) => position.x == pos.x && position.y == pos.y
              )
            ),
          }"
        >
          {{ pos.x }}, {{ pos.y }}
        </div>
      </div>
    </div>

    Your board, computer attacks on this
    <div class="row-wrapper">
      <div v-for="rows in grid" :key="rows.id" class="column-wrapper">
        <div
          v-for="pos in rows"
          :key="pos.id"
          class="grid computer-grid"
          v-bind:class="{
            ship: player1.board.ships.some((ship) =>
              ship.positions.some(
                (position) => position.x == pos.x && position.y == pos.y
              )
            ),

            miss: player1.board.missedHits.some(
              (position) => position.x == pos.x && position.y == pos.y
            ),

            hit: player1.board.ships.some((ship) =>
              ship.hitPositions.some(
                (position) => position.x == pos.x && position.y == pos.y
              )
            ),
          }"
        >
          {{ pos.x }}, {{ pos.y }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      grid: [],
      player1: this.player(this.gameBoardFactory()),
      computer1: this.computer(this.gameBoardFactory()),
      game: this.gameFactory(),
      delay: 500,
    }
  },
  methods: {
    shipFactory(length, positions) {
      let hitPositions = []

      let hit = (position) => {
        hitPositions.push(position)
      }

      let isSunk = () => {
        return hitPositions.length == length
      }

      return { length, positions, hitPositions, hit, isSunk }
    },

    gameBoardFactory() {
      let ships = []
      let missedHits = []
      let takenHits = []

      let receiveAttack = (position) => {
        // Checks if position has already been selecged
        if (
          takenHits.some(
            (curPos) => curPos.x == position.x && curPos.y == position.y
          ) ||
          missedHits.some(
            (curPos) => curPos.x == position.x && curPos.y == position.y
          )
        )
          return "can't select that spot"
        // Checks if position has a ship
        for (let ship in ships) {
          let currentShip = ships[ship]
          if (
            currentShip.positions.some(
              (curPos) => curPos.x == position.x && curPos.y == position.y
            )
          ) {
            currentShip.hit(position)
            takenHits.push(position)
            return currentShip.isSunk() ? 'Ship Sunk!' : 'Not sunk but hit!'
          }
        }
        // If nothing passes then they have hit nothing
        missedHits.push(position)
        return 'missed'
      }

      let placeShip = (length, positions) => {
        let ship = this.shipFactory(length, positions)
        ships.push(ship)
      }

      let allSunk = () => {
        let result = true
        for (let ship in ships) {
          let currentShip = ships[ship]
          if (!currentShip.isSunk()) return false
        }
        return result
      }

      return { ships, missedHits, takenHits, receiveAttack, placeShip, allSunk }
    },

    computer(board) {
      let hitStack = []
      let hit = (oppBoard) => {
        let randomPos = {
          x: Math.floor(Math.random() * 10),
          y: Math.floor(Math.random() * 10),
        }

        let hitResult = (hitStack == undefined || hitStack.length == 0) ? oppBoard.receiveAttack(randomPos) : oppBoard.receiveAttack(hitStack[0])

        if (hitResult == "can't select that spot") {
          console.log(hitResult + 'hitting again')
          hit(oppBoard)
        } else if (hitResult == 'Ship Sunk!') {
          hitStack = []
          setTimeout(() => hit(oppBoard, this.delay))
        } else if (hitResult == 'Not sunk but hit!') {
          let nextHit = {}
          if (randomPos.x  == 9)
            nextHit.x = 
          setTimeout(() => hit(oppBoard), this.delay)
        }
      }

      let placeShips = () => {
        board.placeShip(3, [
          { x: 2, y: 1 },
          { x: 1, y: 1 },
          { x: 0, y: 1 },
        ])
        board.placeShip(4, [
          { x: 3, y: 0 },
          { x: 3, y: 1 },
          { x: 3, y: 2 },
          { x: 3, y: 3 },
        ])
        board.placeShip(5, [
          { x: 9, y: 8 },
          { x: 8, y: 8 },
          { x: 7, y: 8 },
          { x: 6, y: 8 },
          { x: 5, y: 8 },
        ])
      }

      return { hit, board, placeShips }
    },

    player(board) {
      let hit = (oppBoard, position) => {
        return oppBoard.receiveAttack(position)
      }

      let placeShips = () => {
        board.placeShip(3, [
          { x: 2, y: 1 },
          { x: 1, y: 1 },
          { x: 0, y: 1 },
        ])
        board.placeShip(4, [
          { x: 3, y: 0 },
          { x: 3, y: 1 },
          { x: 3, y: 2 },
          { x: 3, y: 3 },
        ])
        board.placeShip(5, [
          { x: 9, y: 8 },
          { x: 8, y: 8 },
          { x: 7, y: 8 },
          { x: 6, y: 8 },
          { x: 5, y: 8 },
        ])
      }

      return { board, hit, placeShips }
    },

    gameFactory() {
      let initializeGame = () => {
        for (let row = 0; row < 10; row++) {
          let currentRow = []
          for (let col = 0; col < 10; col++) {
            currentRow.push({
              x: col,
              y: row,
            })
          }
          this.grid.push(currentRow)
        }
        this.player1.placeShips()
        this.computer1.placeShips()
      }

      let playTurn = (position) => {
        let player = this.player1
        let computer = this.computer1
        let hitResult = player.hit(computer.board, position)

        if (hitResult != 'missed') {
          if (player.board.allSunk()) console.log('You Won!')
        } else {
          setTimeout(() => computer.hit(player.board), this.delay)
        }
      }

      return { initializeGame, playTurn }
    },
  },
  mounted() {
    this.game.initializeGame()
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.row-wrapper {
  margin: 0 auto 50px auto;
  display: grid;
  grid-template-rows: repeat(10, 50px);
  text-align: center;
  width: 500px;
}

.column-wrapper {
  display: grid;
  grid-template-columns: repeat(10, 50px);
}

.grid {
  outline: black 1px solid;
}

.ship {
  background-color: red;
}

.miss {
  background-color: blue;
}

.hit {
  background-color: purple;
}
</style>
