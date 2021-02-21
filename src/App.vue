<template>
  <div class="constructor">
    <div class="constructor__controler">
      <button class="constructor__controler__btn" @click="addNewForm()">
        Добавить входные данные
      </button>
      <div class="constructor__controler__formlist">
        <d-form
          :formItem.sync="item"
          :index="index"
          :limit="{
            x: width,
            y: height
          }"
          :turn="changedCoordinates.length + (index + 1)"
          v-for="(item, index) in newCoordinates"
          :key="index"
          @deleteForm="deleteForm"
        />
      </div>
      <button
        class="constructor__controler__btn constructor__controler__btn_add"
        v-if="newCoordinates.length"
        @click="addNewCoordinates"
      >
        Добавить точки
      </button>
    </div>
    <!--    <pre class="fixed">{{ changedCoordinates }}{{ tempVar }}</pre>-->
    <div class="constructor__canvas" ref="constructor__canvas">
      <button
        class="constructor__controler__delete constructor__controler__delete_topright"
        @click="clearCanvas(true)"
      >
        Очистить
      </button>
      <canvas v-if="loading" class="canvas" ref="canvas"></canvas>
    </div>
  </div>
</template>

<script>
const points = require("@/coordinates.json");
// import points from "@/coordinates.json";
export default {
  name: "App",
  components: {
    "d-form": () => import("@/components/coordinatesForm")
  },
  data() {
    return {
      coordinates: [],
      changedCoordinates: [],
      newCoordinates: [],
      height: 700,
      width: 700,
      loading: true,
      ctx: null,
      canvas: null,
      tempVar: [],
      firstLoad: true
    };
  },
  methods: {
    validForms() {
      let valid = true;
      this.newCoordinates.forEach(item => {
        Object.keys(item).forEach(key => {
          if (!item[key]) {
            item.invalid[key] = true;
            valid = false;
          } else {
            item.invalid[key] = false;
          }
        });
      });
      return valid;
    },

    addNewCoordinates() {
      if (this.validForms()) {
        this.newCoordinates.forEach(item => {
          delete item.invalid;
          item.x = +item.x;
          item.y = +item.y;
          this.changedCoordinates.push(item);
        });
        this.newCoordinates = [];
      }
    },
    addNewForm() {
      this.newCoordinates.push({
        x: "",
        y: "",
        invalid: {
          x: false,
          y: false
        }
      });
    },
    deleteForm(index) {
      this.newCoordinates.splice(index, 1);
    },
    clearCanvas(clearStorage = false) {
      if (this.ctx) {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      }
      if (clearStorage) {
        localStorage.clear();
      }
    },
    //Drawing figure
    figureDrawing() {
      // this.points = JSON.parse(this.points);
      this.coordinates = JSON.parse(JSON.stringify(this.changedCoordinates));
      // get the canvas element using the DOM
      this.canvas = this.$refs.canvas;
      this.canvas.width = this.$refs.constructor__canvas.clientWidth - 50;
      this.canvas.height = this.$refs.constructor__canvas.clientHeight - 50;
      // Make sure we don't execute when canvas isn't supported
      if (this.canvas.getContext) {
        // use getContext to use the canvas for drawing
        this.ctx = this.canvas.getContext("2d");

        this.ctx.fillStyle = "transparent";

        // calculate max and min x and y
        var minX = this.coordinates[0].x;
        var maxX = this.coordinates[0].x;
        var minY = this.coordinates[0].y;
        var maxY = this.coordinates[0].y;

        for (let i = 1; i < this.coordinates.length; i++) {
          if (this.coordinates[i].x < minX) minX = this.coordinates[i].x;
          if (this.coordinates[i].x > maxX) maxX = this.coordinates[i].x;

          if (this.coordinates[i].y < minY) minY = this.coordinates[i].y;
          if (this.coordinates[i].y > maxY) maxY = this.coordinates[i].y;
        }

        // choose a "central" point
        var center = {
          x: minX + (maxX - minX) / 2,
          y: minY + (maxY - minY) / 2
        };
        //
        // precalculate the angles of each point to avoid multiple calculations on sort
        for (let i = 0; i < this.coordinates.length; i++) {
          this.coordinates[i].angle = Math.acos(
            (this.coordinates[i].x - center.x) /
              this.lineDistance(center, this.coordinates[i])
          );

          if (this.coordinates[i].y > center.y) {
            this.coordinates[i].angle =
              Math.PI + Math.PI - this.coordinates[i].angle;
          }
        }

        // sort by angle
        this.coordinates = this.coordinates.sort(function(a, b) {
          return a.angle - b.angle;
        });

        // Draw shape
        this.ctx.beginPath();

        this.ctx.moveTo(this.coordinates[0].x, this.coordinates[0].y);
        this.ctx.strokeText(
          `${1}(${this.coordinates[0].x};${this.coordinates[0].y})`,
          this.coordinates[0].x,
          this.coordinates[0].y
        );
        for (let i = 1; i < this.coordinates.length; i++) {
          this.ctx.lineTo(this.coordinates[i].x, this.coordinates[i].y);
          this.ctx.strokeText(
            `${i + 1}(${this.coordinates[i].x};${this.coordinates[i].y})`,
            this.coordinates[i].x,
            this.coordinates[i].y
          );
          this.ctx.font = "9pt sans-serif";
        }

        // ctx.lineTo(this.coordinates[0].x, this.coordinates[0].y);

        this.ctx.stroke();
        this.ctx.fill();
      }
    },
    lineDistance(point1, point2) {
      var xs = 0;
      var ys = 0;

      xs = point2.x - point1.x;
      xs = xs * xs;

      ys = point2.y - point1.y;
      ys = ys * ys;

      return Math.sqrt(xs + ys);
    }
  },
  watch: {
    changedCoordinates: {
      handler() {
        this.clearCanvas();
        if (this.firstLoad) {
          let localst = JSON.parse(localStorage.getItem("coordinates"));
          let t = points;
          console.log(localst);
          t.forEach((item, index) => {
            let coor = item.split(";");
            t[index] = {
              x: coor[0],
              y: coor[1]
            };
          });
          this.changedCoordinates = localst?.length ? localst : t;
          this.firstLoad = false;
        }
        if (this.changedCoordinates.length && this.$refs.canvas) {
          this.figureDrawing();
        }
        localStorage.setItem(
          "coordinates",
          JSON.stringify(this.changedCoordinates)
        );
      },
      immediate: true
    }
  },
  mounted() {
    this.width = this.$refs.constructor__canvas.clientWidth - 100;
    this.height = this.$refs.constructor__canvas.clientHeight - 100;
  }
};
</script>
<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.fixed {
  position: fixed;
  bottom: 0;
  right: 0;
  width: 400px;
  height: 400px;
  background: #acb5b0;
  overflow: auto;
}
.constructor {
  display: flex;
  height: 100vh;
  &__controler {
    display: flex;
    flex-direction: column;
    width: 30%;
    padding: 15px 0 15px 25px;
    &__alert {
      color: red;
      font-size: 12px;
      padding-left: 20px;
    }
    ::-webkit-scrollbar {
      width: 5px;
      height: 8px;
      background-color: gray;
      appearance: none;
      -webkit-border-radius: 20px;
      -moz-border-radius: 20px;
      border-radius: 20px;
    }

    ::-webkit-scrollbar-thumb {
      background-color: black;
      border-radius: 20px;
      width: 100%;
    }

    ::-webkit-scrollbar-thumb:hover {
      background-color: #253861;
    }

    /* Стрелки */

    &__formlist {
      height: calc(100% - 100px);
      width: 100%;
      padding-right: 10px;
      overflow: auto;
    }
    &__input {
      display: flex;
      align-items: center;
      margin-top: 10px;
      flex-wrap: wrap;
      span {
        padding-left: 10px;
      }
    }
    &__delete {
      width: 100px;
      align-self: flex-end;
      border: none;
      background: red;
      color: white;
      padding: 5px 15px;
      -webkit-border-radius: 5px;
      -moz-border-radius: 5px;
      border-radius: 5px;
      cursor: pointer;
      &_topright {
        position: fixed;
        top: 10px;
        right: 25px;
      }
    }
    &__add {
      display: flex;
      flex-direction: column;
      margin-top: 20px;
      input {
        height: 25px;
        width: 150px;
        padding: 10px;
      }
    }
    &__btn {
      background: #0f1419;
      width: fit-content;
      padding: 10px 25px;
      border: none;
      -webkit-border-radius: 10px;
      -moz-border-radius: 10px;
      border-radius: 10px;
      cursor: pointer;
      outline: none;
      font-weight: 600;
      color: white;
      overflow: hidden;
      transition: 0.2s;
      &:hover {
        transform: scale(1.05);
      }
      &:active {
        transform: scale(1);
      }
      &_add {
        margin-top: auto;
        margin-left: auto;
        margin-right: 10px;
      }
    }
  }
  &__canvas {
    width: 70%;
    height: 100%;
    padding: 15px;
    border-left: 2px solid black;
    overflow: auto;
  }
}
</style>
