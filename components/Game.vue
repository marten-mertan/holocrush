<template>
  <div class="game">
    <div class="game-field">
      <div v-for="(cell, index) in field.cells" 
           :key="index" 
           :class="['game-cell', {'in-streak': cell.inStreak}]"
           :style="{'top': `${cell.y*80}px`, 'left': `${cell.x*80}px`}"
      >
        <svg v-if="cell.icon" 
             :class="cell.icon">
          <use :xlink:href="`icons/all.svg#${cell.icon}`" />
        </svg>
      </div>
    </div>
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
          cells: []
        },
      }
    },

    methods: {
      generateCells() {
        for (let i=0; i<this.field.width; i++) {
          for (let j=0; j<this.field.height; j++) {
            let cell = {
              id: i*this.field.height + j,
              x: j,
              y: i,
              icon: '',
              value: '',
              inStreak: false, // тестовое поле, показывает, входит ли тайл в триплет
            };

            // TODO: сделать проверку randomTile, чтобы поле генерировалось изначально без триплетов
            const randomTile = this.field.availableTiles[Math.floor(Math.random()*this.field.availableTiles.length)];
            cell.icon = randomTile.icon;
            cell.value = randomTile.value;
            cell.name = randomTile.name;

            this.field.cells.push(cell);
          }
        }

        // тестовая проверка поля на наличие триплетов
        for (let i=0; i<this.field.cells.length; i++) {
          let cell = this.field.cells[i];
          if (this.isCellInStreak(cell.id)) {
            this.field.cells[i].inStreak = true;
          }
        }
      },

      isCellInStreak(cellID) {
        return this.isCellInVerticalStreak(cellID) || this.isCellInHorizontalStreak(cellID);
      },

      isCellInVerticalStreak(cellID) {
        const cell = this.field.cells[cellID];
        let topStreak = 0;
        let bottomStreak = 0;

        let tmp = cell.y - 1;
        while(tmp >= 0 && this.field.cells[cellID - (topStreak + 1) * this.field.width].value === cell.value){
          topStreak++;
          tmp--;
        }

        tmp = cell.y + 1;
        while(tmp < this.field.height && this.field.cells[cellID + (bottomStreak + 1) * this.field.width].value === cell.value){
          bottomStreak++;
          tmp++;
        }

        return (topStreak + bottomStreak) > 1;
      },

      isCellInHorizontalStreak(cellID) {
        const cell = this.field.cells[cellID];
        let leftStreak = 0;
        let rightStreak = 0;

        let tmp = cell.x - 1;
        while(tmp >= 0 && this.field.cells[cellID - leftStreak - 1].value === cell.value){
          leftStreak++;
          tmp--;
        }

        tmp = cell.x + 1;
        while(tmp < this.field.width && this.field.cells[cellID + rightStreak + 1].value === cell.value){
          rightStreak++;
          tmp++;
        }

        return (leftStreak + rightStreak) > 1;
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
      width: 640px;
      height: 640px;
      margin: auto;
    }

    &-cell {
      position: absolute;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 80px;
      height: 80px;

      svg {
        position: relative;
        width: 60px;
        height: 60px;
        touch-action: none;
        z-index: 2;
      }

      &.in-streak {
        background-color: aqua;
      }
    }
  }
</style>
