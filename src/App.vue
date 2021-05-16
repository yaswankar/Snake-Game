<template>
  <div id="app">
    <button v-if="started" @click="startGame">↻</button>
    <button v-else @click="startGame">Start</button>
    <div id="score">{{score}}</div>
    <canvas id="gameCanvas" width="300" height="300"></canvas>
  </div>
</template>

<script>
const GAME_SPEED = 100;
const CANVAS_BORDER_COLOUR = 'black';
const CANVAS_BACKGROUND_COLOUR = "black";
const SNAKE_COLOUR = 'lightgreen';
const HEAD_COLOUR = 'red';
const SNAKE_BORDER_COLOUR = 'white';
const FOOD_COLOUR = 'yellow';
const FOOD_BORDER_COLOUR = 'darkred';

export default {
  name: 'App',
  data() {
    return {
     snake : [
        {x: 150, y: 150},
        {x: 140, y: 150},
        {x: 130, y: 150},
        {x: 120, y: 150},
        {x: 110, y: 150}
      ],
      score:0, // User score
      changingDirection:false, // When set to true the snake will change direction
      foodX: 0, // Food x-coordinate
      foodY: 0, // Food y-coordinate
      dx:10, // Horizontal velocity
      dy:0, // Vertical velocity
      ctx: null, // canvas Context
      gameCanvas: null, // canvas dom
      started: false // game already started
    }
  },
  mounted() {
    // Get the canvas element
    this.gameCanvas = document.getElementById("gameCanvas");
    // Return a two dimensional drawing context
    this.ctx = this.gameCanvas.getContext("2d");
    this.clearCanvas();
  },
  methods: {
    // Start game
    startGame() {
      this.started = true;
      this.clearCanvas();
      this.reset();
      this.render();
      // Create the first food location
      this.createFood();
      // Call changeDirection whenever a key is pressed
      document.addEventListener("keydown", this.changeDirection);
    },
    reset() {
      this.snake = [
        {x: 150, y: 150},
        {x: 140, y: 150},
        {x: 130, y: 150},
        {x: 120, y: 150},
        {x: 110, y: 150}
      ];
  
      // The user's score
      this.score = 0;
      // When set to true the snake is changing direction
      this.changingDirection=false;
      // Food x-coordinate
      this.foodX= 0;
      // Food y-coordinate
      this.foodY= 0;
      // Horizontal velocity
      this.dx=10;
      // Vertical velocity
      this.dy=0;
    },
    /**
     * render function of the game
     * called repeatedly to advance the game
     */
    render() {
      // If the game ended return early to stop game
      if (this.didGameEnd()) return;
      let self = this;
      setTimeout(function onTick() {
        self.changingDirection = false;
        self.clearCanvas();
        self.drawFood();
        self.advanceSnake();
        self.drawSnake();

        // Call game again
        self.render();
      }, GAME_SPEED)
    },

    /**
     * Change the background colour of the canvas to CANVAS_BACKGROUND_COLOUR and
     * draw a border around it
     */
    clearCanvas() {
      //  Select the colour to fill the drawing
      this.ctx.fillStyle = CANVAS_BACKGROUND_COLOUR;
      //  Select the colour for the border of the canvas
      this.ctx.strokestyle = CANVAS_BORDER_COLOUR;

      // Draw a "filled" rectangle to cover the entire canvas
      this.ctx.fillRect(0, 0, this.gameCanvas.width, this.gameCanvas.height);
      // Draw a "border" around the entire canvas
      this.ctx.strokeRect(0, 0, this.gameCanvas.width, this.gameCanvas.height);
    },

    /**
     * Draw the food on the canvas
     */
    drawFood() {
      this.ctx.fillStyle = FOOD_COLOUR;
      this.ctx.strokestyle = FOOD_BORDER_COLOUR;
      this.ctx.fillRect(this.foodX, this.foodY, 10, 10);
      this.ctx.strokeRect(this.foodX, this.foodY, 10, 10);
    },

    /**
     * Advances the snake by changing the x-coordinates of its parts
     * according to the horizontal velocity and the y-coordinates of its parts
     * according to the vertical veolocity
     */
    advanceSnake() {
      // Create the new Snake's head
      const head = {x: this.snake[0].x + this.dx, y: this.snake[0].y + this.dy};
      // Add the new head to the beginning of this.snake body
      this.snake.unshift(head);

      const didEatFood = this.snake[0].x === this.foodX && this.snake[0].y === this.foodY;
      if (didEatFood) {
        // Increase score
        this.score += 10;
        // Generate new food location
        this.createFood();
      } else {
        // Remove the last part of snake body
        this.snake.pop();
      }
    },

    /**
     * Returns true if the head of the snake touched another part of the game
     * or any of the walls
     */
    didGameEnd() {
      for (let i = 4; i < this.snake.length; i++) {
        if (this.snake[i].x === this.snake[0].x && this.snake[i].y === this.snake[0].y) return true
      }

      if(this.snake[0].x < 0) this.snake[0].x = this.gameCanvas.width - 10;
      if(this.snake[0].x > this.gameCanvas.width - 10) this.snake[0].x = 0;
      if(this.snake[0].y < 0) this.snake[0].y = this.gameCanvas.height - 10;
      if(this.snake[0].y > this.gameCanvas.height - 10) this.snake[0].y = 0;
    },

    /**
     * Generates a random number that is a multiple of 10 given a minumum
     * and a maximum number
     * @param { number } min - The minimum number the random number can be
     * @param { number } max - The maximum number the random number can be
     */
    randomTen(min, max) {
      return Math.round((Math.random() * (max-min) + min) / 10) * 10;
    },

    /**
     * Creates random set of coordinates for the snake food.
     */
    createFood() {
      // Generate a random number the food x-coordinate
      this.foodX = this.randomTen(0, this.gameCanvas.width - 10);
      // Generate a random number for the food y-coordinate
      this.foodY = this.randomTen(0, this.gameCanvas.height - 10);

      // if the new food location is where the snake currently is, generate a new food location
      this.snake.forEach((part) => {
        const foodIsoNsnake = part.x == this.foodX && part.y == this.foodY;
        if (foodIsoNsnake) this.createFood();
      });
    },

    /**
     * Draws the snake on the canvas
     */
    drawSnake() {
      // loop through the snake parts drawing each part on the canvas
      this.snake.forEach(this.drawSnakePart)
    },

    /**
     * Draws a part of the snake on the canvas
     * @param { object } snakePart - The coordinates where the part should be drawn
     * @param { number } index - to denote the body part index
     */
    drawSnakePart(snakePart, index) {
      // Set the colour of the snake part
      if(index === 0) this.ctx.fillStyle = HEAD_COLOUR;
      else this.ctx.fillStyle = SNAKE_COLOUR;

      // Set the border colour for the snake part
      this.ctx.strokestyle = SNAKE_BORDER_COLOUR;
      // Draw a "filled" rectangle to represent the snake part at the coordinates
      // the part is located
      this.ctx.fillRect(snakePart.x, snakePart.y, 10, 10);
    },

    /**
     * Changes the vertical and horizontal velocity of the snake according to the
     * key that was pressed.
     * @param { object } event - The keydown event
     */
    changeDirection(event) {
      const LEFT_KEY = 37;
      const RIGHT_KEY = 39;
      const UP_KEY = 38;
      const DOWN_KEY = 40;
      /**
       * Preventing the snake from going to the reverse direction
       * Example scenario:
       * Snake is moving to the right. User presses down and immediately left
       * and the snake immediately changes direction without taking a step down first
       */
      if (this.changingDirection) return;
      this.changingDirection = true;

      const keyPressed = event.keyCode;

      const goingUp = this.dy === -10;
      const goingDown = this.dy === 10;
      const goingRight = this.dx === 10;
      const goingLeft = this.dx === -10;

      if (keyPressed === LEFT_KEY && !goingRight) {
        this.dx = -10;
        this.dy = 0;
      }
      if (keyPressed === UP_KEY && !goingDown) {
        this.dx = 0;
        this.dy = -10;
      }
      if (keyPressed === RIGHT_KEY && !goingLeft) {
        this.dx = 10;
        this.dy = 0;
      }
      if (keyPressed === DOWN_KEY && !goingUp) {
        this.dx = 0;
        this.dy = 10;
      }
    }
  }
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
  #score {
    text-align: center;
    font-size: 16px;
    margin: 20px 10px;  
    font-weight: bold;
    font-family: 'Antic Slab', serif;
  }
  #gameCanvas {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border: 10px solid #4d4d4d;
  }
</style>
