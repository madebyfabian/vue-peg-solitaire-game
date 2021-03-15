<template>
  <h1>Solitaire</h1>
  <button @click="handleRefresh">Refresh</button>

  <div class="game grid">
    <div class="row" v-for="(rowData, row) of gameData" :key="row">
      <div 
        v-for="(colVal, col) of rowData" :key="col"
        @click="() => handleItemClick(row, col)"
        class="item"
        :class="{ 
          isField: colVal !== null,
          isActive: colVal === true,
          isMovingSource: movingSource?.row === row && movingSource?.col === col
        }">
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'App',

    data: () => ({
      gameData: [],
      movingSource: null
    }),

    created() {
      this.handleRefresh()
    },

    methods: {  
      handleRefresh() {
        this.gameData = this._generateGameData()
        this.movingSource = null
      },

      handleItemClick( row, col ) {
        if (this.isPosOutsideBoard(row, col))
          return

        // If no moving sourse set AND the item is false/null
        if (!this.movingSource && !this.gameData?.[row]?.[col])
          return

        // If moving source is already set, execute movement
        if (this.movingSource)
          return this.moveTo(row, col)

        this.setMovingSource(row, col)
      },


      moveTo( row, col ) {
        // Check movability again, but this time also with the destination in mind.
        const destPos = { row, col },
              sourcePos = this?.movingSource

        if (!sourcePos)
          return console.error('No moving source!')

        if (!this.isMovableItem(sourcePos, destPos)) {
          this.setMovingSource(row, col)
          return console.error('Item cannot be moved that way!')
        }

        // move old item from field
        let sourceField = this.gameData?.[sourcePos?.row]?.[sourcePos?.col]
        if (sourceField !== null) 
          this.gameData[sourcePos.row][sourcePos.col] = false

        // to new item from field
        let destField = this.gameData?.[destPos?.row]?.[destPos?.col]
        if (destField !== null)
          this.gameData[destPos.row][destPos.col] = true

        // Remove the item between source and dest
        const fieldPosBetween = this.findFieldPosBetweenTwoPos({ ...destPos }, { ...sourcePos })
        if (!fieldPosBetween)
          return console.error('Field between not found')

        let betweenField = this.gameData?.[fieldPosBetween?.row]?.[fieldPosBetween?.col]
        if (betweenField !== null)
          this.gameData[fieldPosBetween.row][fieldPosBetween.col] = false
        
        this.movingSource = null
      },


      /**
       * @param {number} row x position
       * @param {number} col y position
       */
      setMovingSource( row, col ) {
        if (this.isPosOutsideBoard(row, col))
          return console.error('Hejjj, dont play outside the game :P')

        if (!this.isMovableItem({ row, col }))
          return console.warn('The item you selected is not movable :0')
          
        // console.log('set moving source to', { row, col })
        this.movingSource = { row, col }
      },


      /**
       * Checks if an item (sourcePos) is movable.
       * If the @param sourcePos is not set, it will check all directions.
       * 
       * @param {object} sourcePos X/Y value of source position.
       * @param {number} sourcePos.row 
       * @param {number} sourcePos.col
       * 
       * @param {object} destPos X/Y value of destination position.
       * @param {number} destPos.row 
       * @param {number} destPos.col
       */
      isMovableItem( sourcePos, destPos = null ) {
        const moveActions = [ [ 0, 2 ], [ 0, -2 ], [ 2, 0 ], [ -2, 0 ] ]

        // First, check if there is generally any possibility to move.
        let possibleMoveActions = moveActions.map(action => {
          let newRow = sourcePos.row + action[0],
              newCol = sourcePos.col + action[1]

          let newField = this.gameData?.[newRow]?.[newCol]
          if (newField !== false)
            return null

          return { row: newRow, col: newCol }
        }).filter(action => action)

        // If destPos is given, check if it equals to one of the possible actions.
        if (destPos?.row && destPos?.col)
          possibleMoveActions = possibleMoveActions.filter(action => {
            return action.row == destPos.row && action.col == destPos.col
          })

        // Then, check if there is an item between the source and destination (like [->][x][<-])
        possibleMoveActions = possibleMoveActions.filter(action => {
          let betweenFieldPos = this.findFieldPosBetweenTwoPos({ ...action }, { ...sourcePos })
          let fieldValue = this.gameData?.[betweenFieldPos?.row]?.[betweenFieldPos?.col]
          if (fieldValue !== true)
            return false

          return true
        })

        return !!possibleMoveActions.length
      },


      findFieldPosBetweenTwoPos( firstPos, secondPos ) {
        let row = firstPos.row,
            col = firstPos.col

        // If direction is up/down
        if (firstPos.row === secondPos.row)
          col = Math.round((firstPos.col + secondPos.col) / 2) // number between e.g. 2 and 4 (will be 3)
        else
          row = Math.round((firstPos.row + secondPos.row) / 2) // ...
        
        return { row, col }
      },


      /**
    	 * Checks if the given x/y position is outside the playable board.
       */
      isPosOutsideBoard( row, col ) {
        let rowOutsideBoard = row <= 1 || row >= 5,
            colOutsideBoard	= col <= 1 || col >= 5
        return rowOutsideBoard && colOutsideBoard
      },


      /**
       * Used only to generate game data on init.
       */
      _generateGameData() {
        let arr = []
        for (let row = 0; row < 7; row++) {
          let currRow = []
          for (let col = 0; col < 7; col++) {
            let colVal = true
            if (this.isPosOutsideBoard(row, col)) 
              colVal = null

            if (row === 3 && col === 3) 					 
              colVal = false

            currRow.push(colVal)
          }
          arr.push(currRow)
        }

        return arr
      },
    }
  }
</script>

<style lang="scss">
  * {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
  }

  html {
    position: fixed;
    height: 100%;
    overflow: hidden;
  }

  :root {
    color-scheme: dark;
  }

  body {
    background: #000;
    color: #ccc;
    margin: 0;
    width: 100vw;
    height: 100vh;
    overflow-y: scroll;
    overflow-x: hidden;
    -webkit-overflow-scrolling: touch;
  }

  h1 {
    text-align: center;
    margin: 3rem 0 2rem;
  }

  button {
    margin: 2rem auto 5rem;
    display: block;
    width: 75vw;
    max-width: 200px;
    padding: 1rem 2rem;
    border-radius: .75rem;
    border: none;
    background: linear-gradient(to right bottom, #facc15, #f97316);
    color: #fff;
    filter: saturate(.75) brightness(.75);
  }

  .grid {
    display: grid;
    margin: 0 auto;
    grid-template-rows: repeat(7, 40px);
    gap: .75rem;

    .row {
      display: grid;
      grid-template-columns: repeat(7, 40px);
      justify-content: center;
      align-items: center;
      gap: .75rem;
      height: 40px;

      .item {
        width: 40px;
        height: 40px;
        border-radius: 100%;
        transition: transform 150ms ease;
        overflow: hidden;
        position: relative;
        filter: saturate(.75) brightness(.75);

        &::before, &::after {
          content: '';
          position: absolute;
          top: 0;
          left: 0;
          height: 100%;
          width: 100%;
          opacity: 0;
          transition: opacity 150ms ease;
        }

        &::before {
          background: linear-gradient(to right bottom, rgb(168, 85, 247), rgb(99, 102, 241));
          z-index: 2;
        }

        &::after {
          background: linear-gradient(to right bottom, rgb(236, 72, 153), rgb(244, 63, 94));
          z-index: 3;
        }

        &.isField {
          background: radial-gradient(#2a2a2d 33%, #000000);
          cursor: pointer;
        }

        &.isActive::before {
          opacity: 1
        }
        
        &.isMovingSource {
          transform: scale(1.2);

          &::after {
            opacity: 1;
          }
        }
      }
    }
  }
</style>