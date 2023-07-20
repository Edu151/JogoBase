<template>
  <div>
    <canvas ref="canvas" width="400" height="300"></canvas>
    <button @click="startGame" v-if="!gameStarted">Iniciar Jogo</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      gameStarted: false,
      paddleWidth: 80,
      paddleHeight: 10,
      paddleSpeed: 6,
      ballRadius: 6,
      ballSpeed: 4,
      brickRowCount: 5,
      brickColumnCount: 8,
      brickWidth: 45,
      brickHeight: 15,
      brickPadding: 10,
      brickOffsetTop: 30,
      brickOffsetLeft: 30,
      score: 0,
      lives: 3,
      paddleX: 0,
      ballX: 0,
      ballY: 0,
      ballDx: 0,
      ballDy: 0,
      bricks: [],
    };
  },
  mounted() {
    this.initGame();
    this.draw();
  },
  methods: {
    initGame() {
      const canvas = this.$refs.canvas;
      this.paddleX = (canvas.width - this.paddleWidth) / 2;
      this.ballX = canvas.width / 2;
      this.ballY = canvas.height - 30;
      this.ballDx = this.ballSpeed;
      this.ballDy = -this.ballSpeed;

      this.bricks = [];
      for (let c = 0; c < this.brickColumnCount; c++) {
        this.bricks[c] = [];
        for (let r = 0; r < this.brickRowCount; r++) {
          this.bricks[c][r] = { x: 0, y: 0, status: 1 };
        }
      }
    },
    startGame() {
      this.gameStarted = true;
      this.initGame();
    },
    drawPaddle(context) {
      context.beginPath();
      context.rect(this.paddleX, this.$refs.canvas.height - this.paddleHeight, this.paddleWidth, this.paddleHeight);
      context.fillStyle = '#0095DD';
      context.fill();
      context.closePath();
    },
    drawBall(context) {
      context.beginPath();
      context.arc(this.ballX, this.ballY, this.ballRadius, 0, Math.PI * 2);
      context.fillStyle = '#0095DD';
      context.fill();
      context.closePath();
    },
    drawBricks(context) {
      for (let c = 0; c < this.brickColumnCount; c++) {
        for (let r = 0; r < this.brickRowCount; r++) {
          if (this.bricks[c][r].status === 1) {
            const brickX = c * (this.brickWidth + this.brickPadding) + this.brickOffsetLeft;
            const brickY = r * (this.brickHeight + this.brickPadding) + this.brickOffsetTop;
            this.bricks[c][r].x = brickX;
            this.bricks[c][r].y = brickY;
            context.beginPath();
            context.rect(brickX, brickY, this.brickWidth, this.brickHeight);
            context.fillStyle = '#0095DD';
            context.fill();
            context.closePath();
          }
        }
      }
    },
    collisionDetection() {
      for (let c = 0; c < this.brickColumnCount; c++) {
        for (let r = 0; r < this.brickRowCount; r++) {
          const brick = this.bricks[c][r];
          if (brick.status === 1) {
            if (
              this.ballX > brick.x &&
              this.ballX < brick.x + this.brickWidth &&
              this.ballY > brick.y &&
              this.ballY < brick.y + this.brickHeight
            ) {
              this.ballDy = -this.ballDy;
              brick.status = 0;
              this.score++;
              if (this.score === this.brickRowCount * this.brickColumnCount) {
                alert('Você venceu! Parabéns!');
                this.gameStarted = false;
                this.initGame();
              }
            }
          }
        }
      }
    },
    drawScore(context) {
      context.font = '16px Arial';
      context.fillStyle = '#0095DD';
      context.fillText('Pontuação: ' + this.score, 8, 20);
    },
    drawLives(context) {
      context.font = '16px Arial';
      context.fillStyle = '#0095DD';
      context.fillText('Vidas: ' + this.lives, this.$refs.canvas.width - 65, 20);
    },
    draw() {
      const canvas = this.$refs.canvas;
      const context = canvas.getContext('2d');
      context.clearRect(0, 0, canvas.width, canvas.height);

      if (this.gameStarted) {
        this.drawBricks(context);
        this.drawPaddle(context);
        this.drawBall(context);
        this.drawScore(context);
        this.drawLives(context);
        this.collisionDetection();

        // Movimentação da bola
        this.ballX += this.ballDx;
        this.ballY += this.ballDy;

        // Colisão com as paredes laterais
        if (this.ballX + this.ballRadius > canvas.width || this.ballX - this.ballRadius < 0) {
          this.ballDx = -this.ballDx;
        }

        // Colisão com a parede superior
        if (this.ballY - this.ballRadius < 0) {
          this.ballDy = -this.ballDy;
        }

        // Colisão com a pá
        if (
          this.ballY + this.ballRadius > canvas.height - this.paddleHeight &&
          this.ballX > this.paddleX &&
          this.ballX < this.paddleX + this.paddleWidth
        ) {
          this.ballDy = -this.ballDy;
        }

        // Fim de jogo (bola caiu)
        if (this.ballY + this.ballRadius > canvas.height) {
          this.lives--;
          if (!this.lives) {
            alert('Fim de jogo! Tente novamente!');
            this.gameStarted = false;
            this.initGame();
          } else {
            this.ballX = canvas.width / 2;
            this.ballY = canvas.height - 30;
            this.ballDx = this.ballSpeed;
            this.ballDy = -this.ballSpeed;
            this.paddleX = (canvas.width - this.paddleWidth) / 2;
          }
        }

        // Movimentação da pá
        if (this.$q.platform.is.desktop) {
          const movePaddle = (event) => {
            const rect = canvas.getBoundingClientRect();
            const mouseX = event.clientX - rect.left;
            if (mouseX > 0 && mouseX < canvas.width) {
              this.paddleX = mouseX - this.paddleWidth / 2;
            }
          };

          canvas.addEventListener('mousemove', movePaddle, false);
        } else {
          const touchMove = (event) => {
            event.preventDefault();
            const touch = event.targetTouches[0];
            const relativeX = touch.pageX - canvas.offsetLeft;
            if (relativeX > 0 && relativeX < canvas.width) {
              this.paddleX = relativeX - this.paddleWidth / 2;
            }
          };

          canvas.addEventListener('touchmove', touchMove, false);
        }
      }

      requestAnimationFrame(this.draw);
    },
  },
};
</script>

<style scoped>
canvas {
  border: 1px solid #000;
  background-color: #f0f0f0;
}

button {
  margin-top: 10px;
}
</style>
q
