<template>
  <div :class="['game', game.status]">
    <transition-group class="game-field" name="game-cell" tag="div">
        <div v-for="cell in field.cells" 
            :key="cell.key" 
            :class="['game-cell', {'picked': field.pickedCell && field.pickedCell.id === cell.id}]"
            @mousedown="mouseDown(cell)"
            @mouseup="mouseUp(cell)"
        >
          <svg v-if="cell.icon" 
              :class="cell.icon">
            <use :xlink:href="`icons/all.svg#${cell.icon}`" />
          </svg>
        </div>
      </transition-group>
  </div>
</template>

<script>
  export default {
    name: 'Game',

    data() {
      return {
        field: {
          width: 8,
          height: 8,
          startGrid: "................................................................", // заготовка для разных типов полей, пока не используется
          availableTiles: [
            {
                value: 1,
                name: 'aqua',
                icon: 'icon-aqua'
            },
            {
                value: 2,
                name: 'marine',
                icon: 'icon-marine'
            },
            {
                value: 3,
                name: 'fubuki',
                icon: 'icon-fubuki'
            },
            {
                value: 4,
                name: 'noel',
                icon: 'icon-noel'
            },
            {
                value: 5,
                name: 'korone',
                icon: 'icon-korone'
            },
          ],
          cells: [],
          pickedCell: null,
          cellsInStreak: [],
        },
        game: {
          status: 'loading',
          score: 0,
        },
      };
    },

    methods: {
      mouseDown(cell) {
        this.field.pickedCell = cell;
        this.game.status = 'picked';
      },

      mouseUp(cell) {
        if (Math.abs(cell.x - this.field.pickedCell.x) + Math.abs(cell.y - this.field.pickedCell.y) === 1) {
          this.swapTwoCells(this.field.pickedCell, cell);

          //
          // проигрываем анимацию
          //

          setTimeout(() => { 
            if (this.isFieldHasStreaks()) {
              this.game.status = 'falling';
              this.removeStreaks();
            } else {
              this.swapTwoCells(this.field.pickedCell, cell);
            }

            this.field.pickedCell = null;
            this.game.status = 'picking';
          }, 300);
        } else {
          console.log('no');
          this.field.pickedCell = null;
          this.game.status = 'picking';
        }
      },

      generateCells() {
        for (let i=0; i<this.field.width; i++) {
          for (let j=0; j<this.field.height; j++) {
            let cell = {
              id: i*this.field.height + j, // индекс в массиве ячеек
              key: i*this.field.height + j, // неизменяемый ключ
              x: j,
              y: i,
              icon: '',
              value: '',
            };

            do {
              const randomTile = this.field.availableTiles[Math.floor(Math.random()*this.field.availableTiles.length)];
              cell.icon = randomTile.icon;
              cell.value = randomTile.value;
              cell.name = randomTile.name;
            } while(this.isCellInStreak(cell))

            this.field.cells.push(cell);
          }
        }

        this.game.status = 'picking';
      },

      setRandomCell(id) {
        const randomTile = this.field.availableTiles[Math.floor(Math.random()*this.field.availableTiles.length)];
        this.field.cells[id].icon = randomTile.icon;
        this.field.cells[id].value = randomTile.value;
        this.field.cells[id].name = randomTile.name;
      },

      isCellInVerticalStreak(cell) {
        let topStreak = 0;
        let bottomStreak = 0;

        let tmp = cell.y - 1;
        while(tmp >= 0 && this.field.cells[cell.id - (topStreak + 1) * this.field.width] && this.field.cells[cell.id - (topStreak + 1) * this.field.width].value === cell.value){
          topStreak++;
          tmp--;
        }

        tmp = cell.y + 1;
        while(tmp < this.field.height && this.field.cells[cell.id + (bottomStreak + 1) * this.field.width] && this.field.cells[cell.id + (bottomStreak + 1) * this.field.width].value === cell.value){
          bottomStreak++;
          tmp++;
        }

        return (topStreak + bottomStreak) > 1;
      },

      isCellInHorizontalStreak(cell) {
        let leftStreak = 0;
        let rightStreak = 0;

        let tmp = cell.x - 1;
        while(tmp >= 0 && this.field.cells[cell.id - leftStreak - 1] && this.field.cells[cell.id - leftStreak - 1].value === cell.value){
          leftStreak++;
          tmp--;
        }

        tmp = cell.x + 1;
        while(tmp < this.field.width && this.field.cells[cell.id + rightStreak + 1] && this.field.cells[cell.id + rightStreak + 1].value === cell.value){
          rightStreak++;
          tmp++;
        }

        return (leftStreak + rightStreak) > 1;
      },

      isCellInStreak(cell) {
        return this.isCellInVerticalStreak(cell) || this.isCellInHorizontalStreak(cell);
      },

      isFieldHasStreaks() {
        this.field.cellsInStreak = [];
        for (let i=0; i<this.field.cells.length; i++) {
          if (this.isCellInStreak(this.field.cells[i])) {
            this.field.cellsInStreak.push(this.field.cells[i]);
          }
        }

        return this.field.cellsInStreak.length ? true : false;
      },

      removeStreaks() {
        if (this.field.cellsInStreak.length) {
          const len = this.field.cellsInStreak.length;
          for (let i=0; i<len; i++) {
            this.field.cells[this.field.cellsInStreak[i].id].value = -1;
            this.field.cells[this.field.cellsInStreak[i].id].name = 'empty';
            this.field.cells[this.field.cellsInStreak[i].id].icon = null;
          }
          this.cellsFalling();
        }
      },

      swapTwoCells(firstCell, secondCell) {
        const firstTempCell = {...firstCell};
        const lastTempCell = {...secondCell};

        // присваиваем новые значения поменяным ячейкам с сохранением индексов ячеек
        this.field.cells[firstTempCell.id].x = lastTempCell.x;
        this.field.cells[firstTempCell.id].y = lastTempCell.y;
        this.field.cells[firstTempCell.id].id = lastTempCell.id;
        this.field.cells[lastTempCell.id].x = firstTempCell.x;
        this.field.cells[lastTempCell.id].y = firstTempCell.y;
        this.field.cells[lastTempCell.id].id = firstTempCell.id;

        // меняем местами ячейки в массиве
        [this.field.cells[firstTempCell.id], this.field.cells[lastTempCell.id]] = [this.field.cells[lastTempCell.id], this.field.cells[firstTempCell.id]]; 
      },

      isFieldHasEmpties() {
        const len = this.field.cells.length;
        for (let i=0; i<len; i++) {
          if (this.field.cells[i].value === -1) {
            return true;
          }
        }
        return false;
      },

      cellsFalling() {
        const len = this.field.cells.length;
        const y = this.field.height;
        const x = this.field.width;
        for (let i=0; i<x; i++) {
          for (let j=y-1; j>=0; j--) {
            if (this.field.cells[j*x+i].value === -1) {
              let isCellExist = false;
              // ищем первый тайл с нужным значением выше
              for(let m = j-1; m>=0; m--) {
                if (this.field.cells[m*x+i].value !== -1) {
                  isCellExist = true;
                  this.swapTwoCells(this.field.cells[j*x+i], this.field.cells[m*x+i]);
                  break;
                }
              }

              if (!isCellExist) {
                break;
              }
            }
          }
        }

        //
        // проигрываем анимацию
        //

        setTimeout(() => {
          for (let i=0; i<this.field.cells.length; i++) {
            if (this.field.cells[i].value === -1) {
              this.setRandomCell(i);
            }
          }

          setTimeout(() => {
            if (this.isFieldHasStreaks()) {
              this.removeStreaks();
            }
          }, 300);
        }, 300);
      },
    },

    mounted() {
      this.generateCells();
    },
  }
</script>

<style lang="scss">
  .game {

    &-field {
      position: relative;
      display: flex;
      flex-wrap: wrap;
      width: 640px;
      height: 640px;
      margin: auto;
    }

    &-cell {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 80px;
      height: 80px;
      transform: translate3d(0, 0, 0);

      &-move {
        transition: transform .3s ease;
      }

      svg {
        position: relative;
        width: 60px;
        height: 60px;
        touch-action: none;
        z-index: 2;
      }
    }
  }
</style>
