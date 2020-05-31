<template>
  <div class="container">
    <h2>Game "Find Matches"</h2>
    <h4>Find two matching squares</h4>
    <div class="row">
      <div class="sidebar">
        <div class="sidebar__item">
          <button @click="generateGame" class="button">New game</button>
        </div>
        <div class="sidebar__item">
          <div>Level:</div>
          <input
            type="number"
            v-model="level"
            class="input"
            min="1"
            max="5"
            @change="changeLevel($event)"
          />
        </div>
        <div class="sidebar__item">
          Record time:
          <span v-if="recordTime > 0">
            {{ Moment(recordTime).format("mm:ss:SS") }}
          </span>
        </div>
        <div class="sidebar__item">Current time {{ currentTime }}</div>
      </div>
      <div class="tile" :style="styleTile">
        <div
          v-for="(item, index) in items"
          :key="index"
          class="tile__item"
          :style="styleTileItem(item)"
          @click="openTile(index)"
        >
          <span v-if="item.active">
            {{ item.id }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Moment from "moment";

export default {
  name: "Home",
  data() {
    return {
      items: [],
      rows: 4,
      activeItems: [],
      openItems: [],
      disabled: false,
      level: 2,
      sizeTile: 750,
      startTime: null,
      recordTime: null,
      currentTime: null,
      interval: null,
      Moment
    };
  },
  computed: {
    styleTile() {
      return {
        width: `${this.sizeTile}px`,
        height: `${this.sizeTile}px`
      };
    }
  },
  methods: {
    styleTileItem(item) {
      const size = `${this.sizeTile / this.rows - 10}px`;
      return {
        background: item.active ? item.color : "gray",
        width: size,
        height: size,
        "font-size": `calc(${size}/2)`
      };
    },
    getRandomColor() {
      const letters = "0123456789ABCDEF";
      let color = "#";
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    },
    generateGame() {
      this.rows = Number(this.level) * 2;
      this.items = [];
      this.activeItems = [];
      this.openItems = [];
      this.currentTime = null;
      this.startTime = null;
      clearInterval(this.interval);
      const countItems = (this.rows * this.rows) / 2;
      for (let i = 0; i < countItems; i++) {
        let color = this.getRandomColor();
        const item = {
          id: i,
          active: false,
          color
        };
        this.items.splice(i, 0, item, Object.assign({}, item));
      }
      this.items = this.items.sort(function() {
        return Math.random() - 0.5;
      });
    },
    openTile(index) {
      if (!this.startTime) {
        this.startGame();
      }
      if (this.disabled || this.items[index].active) {
        return;
      }
      this.items[index].active = true;
      this.activeItems.push(this.items[index].id);
      if (this.activeItems.length === 2) {
        if (this.activeItems[0] !== this.activeItems[1]) {
          this.roundGame();
        } else {
          this.openItems.push(...this.activeItems);
          this.activeItems = [];
        }
      }
      if (this.openItems.length === this.items.length) {
        this.stopGame();
      }
    },
    startGame() {
      this.startTime = Moment();
      this.timer();
    },
    stopGame() {
      const endTime = Moment();
      this.timer(false);
      const diffTime = endTime.diff(this.startTime);
      this.currentTime = Moment(diffTime).format("mm:ss:SS");
      if (this.recordTime === 0 || this.recordTime > diffTime) {
        this.recordTime = diffTime;
        localStorage.setItem(`level-${this.level}`, diffTime);
      }
      this.startTime = null;
    },
    timer(active = true) {
      if (active) {
        this.interval = setInterval(() => {
          this.currentTime = Moment(Moment().diff(this.startTime)).format(
            "mm:ss:SS"
          );
        }, 100);
      }
      if (active === false) {
        clearInterval(this.interval);
      }
    },
    roundGame() {
      this.disabled = true;
      setTimeout(() => {
        this.items.forEach(el => {
          if (el.id === this.activeItems[0] || el.id === this.activeItems[1]) {
            el.active = false;
          }
        });
        this.disabled = false;
        this.activeItems = [];
      }, 500);
    },
    changeLevel(event) {
      const value = event.target.value;
      if (value > 5) {
        this.level = 5;
      } else if (value < 1) {
        this.level = 1;
      } else {
        this.level = value;
      }
      this.recordTime = Number(localStorage.getItem(`level-${this.level}`));
    }
  },
  mounted() {
    this.generateGame();
    this.recordTime = Number(localStorage.getItem(`level-${this.level}`));
  }
};
</script>
<style lang="scss">
.row {
  display: flex;
  flex-direction: row;
  justify-content: center;
  flex-wrap: wrap;
}
.sidebar {
  display: flex;
  flex-direction: column;
  padding: 10px;
  box-sizing: border-box;
  text-align: left;
  width: 200px;
  &__item {
    margin-bottom: 10px;
  }
}
.tile {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  &__item {
    margin: 4px;
    font-size: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: bold;
    cursor: pointer;
  }
}
</style>
