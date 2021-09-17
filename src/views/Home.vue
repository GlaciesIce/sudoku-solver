<template>
  <div class="home">
    <div class="container">
      <h1 class="text-center text-white">Sudoku Solver</h1>
      <h1 class="text-center">Random Useless Project</h1>

      <table class="table" v-if="sudokuData != null">
        <tr v-for="(srow, i) in sudokuData" :key="i" :class="{'border-row' : (i+1)%3==0}">
          <!-- {{srow}} -->
          <td v-for="(scol, j) in srow" :key="j" :class="{'bg-green' : valid[i][j], 'bg-red' : !valid[i][j], 'border-column' : (j+1)%3==0}">
            <input v-model="sudokuData[i][j]" style="width: 50px; text-align: center;">
          </td>
        </tr>
      </table>

      <button class="btn btn-primary" @click="checkValid">Check Invalid</button>
      <button class="btn btn-success ml-3" @click="solveSudoku">Solve</button>

    </div>
  </div>
</template>

<script>
  export default {
    name: 'Home',
    data () {
      return {
        sudokuData : null,
        valid : null,
        oValid : null,

        sets : [
          [0, 1, 2],
          [3, 4, 5],
          [6, 7, 8]
        ]
      }
    },
    mounted () {
      this.generateValid();
      this.initializeBoard();
    },
    methods : {
      generateValid () {
        let valid = [];
        for (var i = 1; i <= 9; i++) {
          let validRow = [];
          for (var j = 1; j<= 9; j++) {
            validRow.push(true);
          }
          valid.push(validRow);
        }
        this.oValid = valid;
      },
      initializeBoard (isSample) {
        // this.sudokuData = new Array(9).fill(new Array(9).fill(null));
        let data = [];
        let iteration = -1;
        for (var i = 1; i <= 9; i++) {

          //Every 3 row, add 1 to iteration
          if ((i-1) % 3 == 0) iteration++;

          let row = [];
          for (var j = 1; j <= 9; j++) {
            // console.log(iteration);
            if (isSample)
              row.push((((i-1)*3 + (j-1) + iteration)%9 + 1).toString());
            else
              row.push('');
          }

          data.push(row);
        }

        this.sudokuData = data;
        this.valid = JSON.parse(JSON.stringify(this.oValid));
      },
      checkValid () {
        //Reset valid colors
        this.valid = JSON.parse(JSON.stringify(this.oValid));

        let data = JSON.parse(JSON.stringify(this.sudokuData));
        console.log(JSON.stringify(data));
        
        //tdata == transposed data
        let tdata = data[0].map((_, colIndex) => data.map(row => row[colIndex]));

        let overallValid = true;

        //Check Row Data Similarities
        for (var i = 0; i < 9; i++) {
          for (var j = 0; j < 9; j++) {
            //If space is empty, zero, or null, skip
            if (data[i][j] == '0' || data[i][j] == '' || data[i][j] == null) continue;

            //If 2 numbers are identical in a row, true, else, false
            let identical = data[i].filter(x=>x==data[i][j]).length >= 2 ? true : false
            
            if (identical) {
              this.$set(this.valid[i], j, false);
              // this.valid[i][j] = false;
              overallValid = false;
              // console.log(`Position x:${i}, y:${j}`);
            }
          }
        }

        //Check Column Data Similarities
        for (i = 0; i < 9; i++) {
          for (j = 0; j < 9; j++) {
            //If space is empty, zero, or null, skip
            if (data[i][j] == '0' || data[i][j] == '' || data[i][j] == null) continue;
            
            //If 2 numbers are identical in a column, true, else, false
            let identical = tdata[i].filter(x=>x==tdata[i][j]).length >= 2 ? true : false

            //If found identical or decided false from validation above, continue false
            if (identical) {
              this.$set(this.valid[j], i, false);
              // console.log(`Transposed Position x:${j}, y:${i}`);
              overallValid = false;
            }
          }
        }

        let sets = [
          [0, 1, 2],
          [0, 1, 2],
          [0, 1, 2],
          [3, 4, 5],
          [3, 4, 5],
          [3, 4, 5],
          [6, 7, 8],
          [6, 7, 8],
          [6, 7, 8]
        ]

        //Check 3x3 Grid Similarities
        for (i = 0; i < 9; i++) {
          for (j = 0; j < 9; j++) {

            //If space is empty, zero, or null, skip
            if (data[i][j] == '0' || data[i][j] == '' || data[i][j] == null) continue;

            //x and y coordinate sets
            let xSet = sets[i];
            let ySet = sets[j];

            let currNumber = data[i][j];
            
            //Check similarities within the 3x3 grid
            for (var k = 0; k < xSet.length; k++) {
              for (var l = 0; l < ySet.length; l++) {

                //If in its own position, skip
                if (i == xSet[k] && j == ySet[l]) continue;

                //If number is identical
                if (currNumber == data[xSet[k]][ySet[l]]) {

                  // console.log(`${currNumber} == ${data[xSet[k]][ySet[l]]}`);

                  //Set both locations to invalid;
                  this.$set(this.valid[i], j, false);
                  this.$set(this.valid[xSet[k]], ySet[l], false);

                  overallValid = false;
                }
              }
            }

          }
        }

        return overallValid;
      },

      isSafe(board, row, col, num) {
        //Row unique check
        for (let d = 0; d < board.length; d++) {
          //Check proposed number in the row
          if (board[row][d] == num) return false;
        }

        //Column unique check
        for (let r = 0; r < board.length; r++) {
          //Check proposed number in column
          if (board[r][col] == num) return false;
        }

        //3x3 grid unique check
        let sqrt = Math.floor(Math.sqrt(board.length));
        let boxRowStart = row - row % sqrt;
        let boxColStart = col - col % sqrt;

        for (let r = boxRowStart; r < boxRowStart + sqrt; r++) {
          for (let d = boxColStart; d < boxColStart + sqrt; d++) {
            if (board[r][d] == num) return false;
          }
        }

        //If everything is unique
        return true;
      },

      recursiveSolve (board, n) {
        let row = -1;
        let col = -1;
        let isEmpty = true;

        this.sudokuData = board;
        this.checkValid();
        setTimeout(100);

        for (let i = 0; i < n; i++) {
          for (let j = 0; j < n; j++) {
            if (board[i][j] == 0) {
              row = i;
              col = j;

              //Missing values in sudoku
              isEmpty = false;
              break;
            }
          }

          if (!isEmpty) break;
        }

        //No empty space left
        if (isEmpty) return true;

        for (let num = 1; num <= n; num++) {
          if (this.isSafe(board, row, col, num)) {
            board[row][col] = num;
            if (this.recursiveSolve(board, n)) return true;
            else board[row][col] = 0;
          }
        }

        return false;
      },

      pacifyInput () {
        let data = JSON.parse(JSON.stringify(this.sudokuData));
        for (var i = 0; i < 9; i++) {
          for (var j = 0; j < 9; j++) {
            if (data[i][j] == '0' || data[i][j] == '' || data[i][j] == null) data[i][j] = '0';
          }
        }
        this.sudokuData = data;
      },

      solveSudoku () {
        this.pacifyInput();

        if (this.recursiveSolve(JSON.parse(JSON.stringify(this.sudokuData)), this.sudokuData.length)){
          alert('Success');
        } else {
          alert("No Solution");
        }
      },
    }
  }
</script>

<style scoped>
  table {
    border-top: 3px solid black;
    border-left: 3px solid black;
  }

  table tr td {
    text-align: center;
    border: 1px solid black;
  }

  .bg-green {
    background-color: lightgreen;
  }

  .bg-red {
    background-color: rgb(255, 190, 185);
  }

  .border-row {
    /* border-top: 3px solid black; */
    border-bottom: 3px solid black;
  }

  .border-column {
    /* border-left: 3px solid black; */
    border-right: 3px solid black;
  }
</style>