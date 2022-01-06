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
        const firstCell = {...this.field.pickedCell};
        const lastCell = {...cell};
        if (Math.abs(lastCell.x - firstCell.x) + Math.abs(lastCell.y - firstCell.y) === 1) {

          // присваиваем новые значения поменяным ячейкам с сохранением индексов ячеек
          this.field.cells[firstCell.id].x = lastCell.x;
          this.field.cells[firstCell.id].y = lastCell.y;
          this.field.cells[firstCell.id].id = lastCell.id;
          this.field.cells[lastCell.id].x = firstCell.x;
          this.field.cells[lastCell.id].y = firstCell.y;
          this.field.cells[lastCell.id].id = firstCell.id;

          // меняем местами ячейки в массиве
          [this.field.cells[firstCell.id], this.field.cells[lastCell.id]] = [this.field.cells[lastCell.id], this.field.cells[firstCell.id]]; 

          //
          // проигрываем анимацию
          //

          setTimeout(() => { 
            if (this.isFieldHasStreaks()) {
              console.log('nice')
            } else {
              // присваиваем новые значения поменяным ячейкам с сохранением индексов ячеек
              this.field.cells[firstCell.id].x = lastCell.x;
              this.field.cells[firstCell.id].y = lastCell.y;
              this.field.cells[firstCell.id].id = lastCell.id;
              this.field.cells[lastCell.id].x = firstCell.x;
              this.field.cells[lastCell.id].y = firstCell.y;
              this.field.cells[lastCell.id].id = firstCell.id;

              // меняем местами ячейки в массиве
              [this.field.cells[firstCell.id], this.field.cells[lastCell.id]] = [this.field.cells[lastCell.id], this.field.cells[firstCell.id]]; 
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
        for (let i=0; i<this.field.cells.length; i++) {
          if (this.isCellInStreak(this.field.cells[i])) {
            return true;
          }
        }
        return false;
      }
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

      &-move {
        transition: transform .3s ease-in-out;
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
