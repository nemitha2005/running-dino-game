<template>
  <div id="game-container" @touchstart="handleTouch">
    <canvas ref="gameCanvas"></canvas>
  </div>
</template>

<script>
export default {
  name: 'DinoGame',
  data() {
    return {
      dino: {
        x: 50,
        y: 150,
        width: 44,
        height: 47,
        image: new Image(),
      },
      gravity: 0.6,
      isJumping: false,
      obstacles: [],
      obstacleImages: [],
      obstacleFrequency: 120,
      frameCount: 0,
      background: {
        image: new Image(),
        x: 0,
      },
      score: 0,
      timer: 0,
    };
  },
  mounted() {
    this.dino.image.src = require('@/assets/character.png');
    this.background.image.src = require('@/assets/forest.jpg');
    this.dino.image.onload = () => {
      this.initGame();
    };
    for (let i = 1; i <= 3; i++) {
      let img = new Image();
      img.src = require(`@/assets/obstacle${i}.png`);
      this.obstacleImages.push(img);
    }
    window.addEventListener('resize', this.resizeCanvas);
  },
  beforeUnmount() {
    document.removeEventListener('keydown', this.handleJump);
    window.removeEventListener('resize', this.resizeCanvas);
  },
  methods: {
    initGame() {
      this.resizeCanvas();
      const context = this.$refs.gameCanvas.getContext('2d');
      this.gameLoop(context);
      document.addEventListener('keydown', this.handleJump);
    },
    resizeCanvas() {
      const canvas = this.$refs.gameCanvas;
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight / 2; // Adjust height as needed
    },
    handleJump(event) {
      if (event.code === 'Space' && !this.isJumping) {
        this.isJumping = true;
        this.dino.velocityY = -10;
      }
    },
    handleTouch() {
      if (!this.isJumping) {
        this.isJumping = true;
        this.dino.velocityY = -10;
      }
    },
    createObstacle() {
      const obstacle = {
        x: this.$refs.gameCanvas.width,
        y: this.$refs.gameCanvas.height - 50,
        width: 20,
        height: 20,
        image: this.obstacleImages[Math.floor(Math.random() * this.obstacleImages.length)],
      };
      this.obstacles.push(obstacle);
    },
    update() {
      if (this.isJumping) {
        this.dino.velocityY += this.gravity;
        this.dino.y += this.dino.velocityY;
        if (this.dino.y >= this.$refs.gameCanvas.height - 50) {
          this.dino.y = this.$refs.gameCanvas.height - 50;
          this.dino.velocityY = 0;
          this.isJumping = false;
        }
      }

      this.obstacles.forEach(obstacle => {
        obstacle.x -= 5;
      });

      this.obstacles = this.obstacles.filter(obstacle => obstacle.x + obstacle.width > 0);

      this.frameCount++;
      if (this.frameCount % this.obstacleFrequency === 0) {
        this.createObstacle();
      }

      this.background.x -= 2;
      if (this.background.x <= -this.$refs.gameCanvas.width) {
        this.background.x = 0;
      }

      this.score++;
      this.timer = Math.floor(this.frameCount / 60);

      this.checkCollisions();
    },
    draw(context) {
      context.clearRect(0, 0, context.canvas.width, context.canvas.height);
      context.drawImage(this.background.image, this.background.x, 0, context.canvas.width, context.canvas.height);
      context.drawImage(this.background.image, this.background.x + context.canvas.width, 0, context.canvas.width, context.canvas.height);

      context.drawImage(this.dino.image, this.dino.x, this.dino.y, this.dino.width, this.dino.height);

      this.obstacles.forEach(obstacle => {
        context.drawImage(obstacle.image, obstacle.x, obstacle.y, obstacle.width, obstacle.height);
      });

      context.fillStyle = 'black';
      context.font = '20px Arial';
      context.fillText(`Score: ${this.score}`, 10, 20);
      context.fillText(`Time: ${this.timer}s`, context.canvas.width - 100, 20);
    },
    gameLoop(context) {
      this.update();
      this.draw(context);
      requestAnimationFrame(() => this.gameLoop(context));
    },
    checkCollisions() {
      this.obstacles.forEach(obstacle => {
        if (
          this.dino.x < obstacle.x + obstacle.width &&
          this.dino.x + this.dino.width > obstacle.x &&
          this.dino.y < obstacle.y + obstacle.height &&
          this.dino.y + this.dino.height > obstacle.y
        ) {
          alert('Game Over');
          this.obstacles = [];
          this.frameCount = 0;
          this.dino.y = this.$refs.gameCanvas.height - 50;
          this.dino.velocityY = 0;
          this.isJumping = false;
          this.score = 0;
          this.timer = 0;
        }
      });
    },
  },
};
</script>

<style scoped>
#game-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: #87ceeb; /* Sky blue background */
}

canvas {
  border: 2px solid #000;
  width: 100%; /* Make canvas responsive */
  height: auto; /* Maintain aspect ratio */
}
</style>
