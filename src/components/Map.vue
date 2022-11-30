<script>
export default {
  // reactive state
  data() {
    return {
      cities: [],
      routes: [],

      canvas: null,

      zoom: 1.0,
      offsetY: 0.0,
      offsetX: 0.0,
      offsetStep: 15,
      requestAnimID: 0,

      search: "",
      citySearched: "",

      cityStart: "",
      cityEnd: "",
    };
  },

  // functions that mutate state and trigger updates
  methods: {
    loadCities() {
      fetch("city.json")
        .then((response) => response.json())
        .then((data) => {
          this.cities = data;
          console.log("loaded " + this.cities.length + " cities");
          this.loadRoutes();
        });
    },
    loadRoutes() {
      fetch("route.json")
        .then((res) => res.json())
        .then((data) => {
          this.routes = data;
          this.drawPoints();
        });
    },
    //-------------------------------------------------------------------------
    // TOOLS METHODS

    debugDrawCenterDot() {
      let ctx = this.canvas;
      const ctxXCenter = ctx.canvas.width / 2;
      const ctxYCenter = ctx.canvas.height / 2;
      ctx.fillStyle = "#FF0000";
      ctx.fillRect(ctxXCenter, ctxYCenter, 5, 5);
    },

    updateX(coordX) {
      return coordX * this.zoom + this.offsetX;
    },
    updateY(coordY) {
      return coordY * this.zoom + this.offsetY;
    },

    // before using, make sure the element exist to avoid error
    // set offset value to set choosed element draw to the canvas center
    setElemToCenter(elementX, elementY) {
      let ctx = this.canvas;
      //get canvas center, set it at each search in case of canvas size change
      const ctxXCenter = ctx.canvas.width / 2;
      const ctxYCenter = ctx.canvas.height / 2;

      // substract center value to city coords value to determine the offset to apply to
      // all cities to set the selected city render at canvas center
      let xDiff = ctxXCenter - elementX * this.zoom;
      let yDiff = ctxYCenter - elementY * this.zoom;

      // apply to the offset
      this.offsetX = xDiff;
      this.offsetY = yDiff;
    },

    /*
      define if add or remove offsetStep on X or Y depending of bool
      boolVertOrHoriz true = vertical
    */
    setUpDownLeftRight(boolVOrH, boolAorM) {
      // select Y or X
      if (boolVOrH) {
        // select + or -
        if (boolAorM) {
          this.offsetY += this.offsetStep;
        } else {
          this.offsetY -= this.offsetStep;
        }
      } else {
        if (boolAorM) {
          this.offsetX += this.offsetStep;
        } else {
          this.offsetX -= this.offsetStep;
        }
      }
    },

    // Select offset change depending of button
    // draw & restart requestAnim
    requestAnimCallBack(boolVOrH, boolAorM) {
      this.setUpDownLeftRight(boolVOrH, boolAorM);
      this.drawPoints();
      this.requestAnimID = window.requestAnimationFrame(() =>
        this.requestAnimCallBack(boolVOrH, boolAorM)
      );
    },

    // start animFrame and clear it on mousedown or up
    keepPress(e, boolVOrH, boolAorM) {
      if (e.type === "mousedown") {
        this.requestAnimID = window.requestAnimationFrame(() =>
          this.requestAnimCallBack(boolVOrH, boolAorM)
        );
      }
      if (e.type === "mouseup") {
        // stop Animation
        window.cancelAnimationFrame(this.requestAnimID);
      }
    },

    //-------------------------------------------------------------------------
    // DRAW LINES

    drawLine(x1, y1, x2, y2) {
      let ctx = this.canvas;
      ctx.beginPath();
      ctx.strokeStyle = "black";
      ctx.lineWidth = 1;
      ctx.moveTo(x1, y1);
      ctx.lineTo(x2, y2);
      ctx.stroke();
      ctx.closePath();
    },

    drawLines() {
      for (let i = 0; i < this.routes.length; i++) {
        let route = this.routes[i];

        let cityStart = this.cities.filter(
          (city) => city.Code === route.Begin
        )[0];
        let cityEnd = this.cities.filter((city) => city.Code === route.End)[0];

        this.drawLine(
          this.updateX(cityStart.x),
          this.updateY(cityStart.y),
          this.updateX(cityEnd.x),
          this.updateY(cityEnd.y)
        );
      }
    },
    //-------------------------------------------------------------------------
    // DRAWPOINTS

    drawPoints() {
      console.log("drawPoints");

      let ctx = this.canvas;
      // clear canvas first
      ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

      // FOR LOOP -------------------------------------------|
      // draw cities points on canvas
      for (let i = 0; i < this.cities.length; i++) {
        let city = this.cities[i];

        // apply default style upfront, or it would replace
        // city searched "style change" if set after
        ctx.font = "11px Arial";
        ctx.fillStyle = "black";

        // if search is empty, skip
        if (this.citySearched) {
          // during loop when cityName equal to the one searched
          if (this.cities[i].Name === this.citySearched.Name) {
            // apply new "style"
            ctx.fillStyle = "blue";
            ctx.font = "17px Arial bold";
          }
        }

        let cityVarX = this.updateX(city.x);
        let cityVarY = this.updateY(city.y);

        ctx.fillRect(cityVarX, cityVarY, 2, 2);
        ctx.fillText(city.Name, cityVarX, cityVarY);
      } // FOR LOOP END --------------------------------------- |

      this.drawLines();
    },
    //-------------------------------------------------------------------------
    // ZOOM & MOVEMENT

    zoomIn() {
      this.zoom += 0.1;
      this.drawPoints();
    },
    zoomOut() {
      this.zoom -= 0.1;
      this.drawPoints();
    },
    goUp(e) {
      this.keepPress(e, true, true);
    },
    goDown(e) {
      this.keepPress(e, true, false);
    },
    goLeft(e) {
      this.keepPress(e, false, true);
    },
    goRight(e) {
      this.keepPress(e, false, false);
    },

    //-------------------------------------------------------------------------
    // SEARCH CITY & PATH

    searchCity() {
      this.citySearched = this.cities.filter(
        (city) => city.Name.toLowerCase() === this.search.toLowerCase()
      )[0];
      // if search isnt empty string (if search incorrect : undefined. It will be converted to false too)
      if (this.citySearched) {
        this.setElemToCenter(this.citySearched.x, this.citySearched.y);
        this.drawPoints();
      }
      // reset search field to empty string
      this.search = "";
    },

    tripDraw() {},
  },
  //---------------------------------------------
  //---------------------------------------------
  // lifecycle hooks
  mounted() {
    var c = document.getElementById("myCanvas");
    this.canvas = c.getContext("2d");
  },
};
</script>

<template>
  <div id="map" class="map">
    <canvas id="myCanvas" width="560" height="360" />
  </div>
  <div class="searchForm">
    <form @submit.prevent="searchCity">
      <label for="searchID">City search :</label>
      <input v-model="search" type="text" id="searchID" />
      <button>search</button>
    </form>
    <form @submit.prevent="tripDraw">
      <p>Shortest Path :</p>
      <label for="startID">City start :</label>
      <input v-model="cityStart" type="text" id="startID" />
      <label for="endID">City end :</label>
      <input v-model="cityEnd" type="text" id="endID" />
      <button>search</button>
    </form>
  </div>

  <div class="button-bar">
    <div class="button" @click="zoomIn">Zoom in</div>

    <div class="button" @click="zoomOut">Zoom out</div>
  </div>
  <div class="button-bar">
    <div class="button" @mousedown="goUp" @mouseup="goUp">Go up</div>
  </div>
  <div class="button-bar">
    <div class="button" @mousedown="goLeft" @mouseup="goLeft">Go left</div>
    <div class="button" @click="loadCities">Load cities</div>
    <div class="button" @mousedown="goRight" @mouseup="goRight">Go right</div>
  </div>
  <div class="button-bar">
    <div class="button" @mousedown="goDown" @mouseup="goDown">Go down</div>
  </div>
</template>

<style lang="scss">
.searchForm {
  width: 560px;
  margin-inline: auto;
  padding-inline-start: 0.5rem;
  margin-block: 1rem;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0.5rem;
}

.searchForm form {
  width: 50%;
  display: flex;
  flex-direction: column;
}

.button {
  background-color: hsla(160, 100%, 37%, 1);
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: center;
  color: white;
  border-radius: 1em;
  width: 64px;
  height: 64px;
  cursor: pointer;
}
.button-bar {
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
  align-items: center;
  margin-top: 1rem;
}
.map {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 560px;
  height: 360px;
  background-color: #eee;
}
#myCanvas {
  border: 1px solid grey;
}
</style>
