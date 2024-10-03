<script setup>
import { onMounted, ref } from 'vue';


// const matrice = ref([
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
//   [0,0,0,0,0,0,0,0,0,0],
// ])
const gridSize = ref(10);
let matrice = ref()

const end = ref(true)
const newBestScore = ref(false)
let snake
let direction
let score = ref(null)
let bestScore = ref(0)
let snakeHead
let speed = ref(200)
let currentSpeed

onMounted(()=> {
  document.querySelectorAll('select').forEach(selectElement => {
  selectElement.addEventListener('change', (event) => {
    event.target.blur();  // Retire le focus du champ <select>
  });
});

})



const startGame = () => {
  const size = parseInt(gridSize.value)
  matrice.value = Array.from({ length: size }, () => Array(size).fill(0))
  snake = [{ x: (Math.floor(size / 2)) - (Math.floor(size / 4)), y: (Math.floor(size / 2 )) }];
  matrice.value[snake[0].x][snake[0].y] = 1;
  end.value = false
  score.value = 0
  currentSpeed = speed.value
  newBestScore.value = false
  let storedBestScore = localStorage.getItem(`bestScore-${currentSpeed}-${size}`);
  if (storedBestScore) {
    bestScore.value = parseInt(storedBestScore);
  } else {
    bestScore.value = 0
  }

  snakeHead = snake;
  direction = { x: 1, y: 0 };

  generateFood()
  gameLoop()
}

function gameLoop() {
  const size = parseInt(gridSize.value)

  const alive = moveSnake();
  if (alive) {
    drawBoard()
    setTimeout(gameLoop, currentSpeed); // Refaire la boucle après 200ms
  } else {
    if (score.value > bestScore.value) {
      newBestScore.value = true
      bestScore.value = score.value
      localStorage.setItem(`bestScore-${currentSpeed}-${size}`, score.value);
    }
    end.value = true
  }
}

const drawBoard = () => {
  const gameBoard = document.getElementById('gameBoard')

  gameBoard.innerHTML = ""
  
  matrice.value.forEach((element, indexX) => {
    const newLine = document.createElement('div')
    newLine.classList.add('line')
    element.forEach((ind, indexY) => {
      const newDiv = document.createElement('div')
      newDiv.classList.add('tile')
      if (ind === 1) {
        newDiv.classList.add('snake')
      }
      if (ind === 2 ) {
        newDiv.classList.add('apple')
      }
      
      if (indexX === snakeHead.x && indexY === snakeHead.y) {
        newDiv.classList.add('head')
      }
      newLine.appendChild(newDiv)
    })
    gameBoard.appendChild(newLine)

  
  });
}

const generateFood = () => {
  let foodPosition;
  const size = parseInt(gridSize.value)
  do {
    foodPosition = {
      x: Math.floor(Math.random() * size),
      y: Math.floor(Math.random() * size)
    };
  } while (matrice.value[foodPosition.x][foodPosition.y] !== 0); // S'assurer que la case est vide
  matrice.value[foodPosition.x][foodPosition.y] = 2; // Placer la nourriture
  return foodPosition;
}

const moveSnake = () => {
  const head = snake[0]
  const newHead = {x: head.x + direction.x, y: head.y + direction.y}
  snakeHead = newHead
  if(newHead.x < 0 || newHead.y < 0 || newHead.x > matrice.value.length -1 || newHead.y > matrice.value.length -1 || matrice.value[newHead.x][newHead.y] === 1 ) {
    return false
  }
  snake.unshift(newHead)

  
  if (matrice.value[newHead.x][newHead.y] === 2) {
    score.value ++
    generateFood();
    
  } else {
    const tail = snake.pop()
    matrice.value[tail.x][tail.y] = 0
  }

  

  matrice.value[newHead.x][newHead.y] = 1;
  

  return true
}

function changeDirection(dir) {
  if (dir === 'right' && (direction.x !== -1 && direction.y !== 0)) {
    direction = {x:1, y:0}
  }
  if (dir === 'left' && (direction.x !== 1 && direction.y !== 0)) {
    direction = {x:-1, y:0}
  }
  if (dir === 'up' && (direction.x !== 0 && direction.y !== 1)) {
    direction = {x:0, y:-1}
  }
  if (dir === 'down' && (direction.x !== 0 && direction.y !== -1)) {
    direction = {x:0, y:1}
  }

}

document.addEventListener('keydown', (event) => {
  if ((event.key === 'ArrowUp' || event.key === 'z') && direction.y !== 1) direction = { x: 0, y: -1 };
  if ((event.key === 'ArrowDown' || event.key === 's') && direction.y !== -1) direction = { x: 0, y: 1 };
  if ((event.key === 'ArrowLeft' || event.key === 'q') && direction.x !== 1) direction = { x: -1, y: 0 };
  if ((event.key === 'ArrowRight' || event.key === 'd') && direction.x !== -1) direction = { x: 1, y: 0 };
  if (event.key === 'r' || event.key === ' ') startGame();
});


</script>

<template>
  <main class="flex justify-center mt-12 gap-10">
    <div class="flex flex-col items-center">
      <div class="text-[#C8ACD6] flex justify-between w-full mb-2">
        <span class="p-1 bg-[#433D8B] border-2 border-[#C8ACD6]">Score : {{ score }}</span>
        <span class="p-1 bg-[#433D8B] border-2 border-[#C8ACD6]"> Best score : {{ bestScore }}</span>
      </div>
      <div id="gameBoard"></div>
      <div v-if="end" class="absolute top-0 w-screen h-screen bg-black/50">
        <div class="bg-[#17153B] restart-panel p-20 left-1/2">
          <div class="flex flex-col gap-4">
            <div v-if="score" class="flex justify-between items-center">
              <p>Score : {{ score }}</p>
              <p>Best score : {{ bestScore }}</p>

            </div>
            <h1 v-if="newBestScore" class="bestScore text-3xl">New Best Score !!!</h1>
            <h1 class="text-2xl font-semibold">Choose your gameMode</h1>
            <label for="speed" class="flex flex-col">
              Speed
              <select name="speed" id="speed"  v-model="speed">
                <option value="400">Slow</option>
                <option value="300">Medium</option>
                <option value="200">Fast</option>
                <option value="100">Realy fast</option>
                <option value="50">Extremely fast</option>
              </select>
            </label>
            <label for="size" class="flex flex-col">
              Size
              <select name="size" id="size" v-model="gridSize">
                <option value="5">5 x 5</option>
                <option value="10">10 x 10</option>
                <option value="15">15 x 15</option>
                <option value="20">20 x 20</option>
              </select>
            </label>
        
            <button @click="startGame()" class="text-[#C8ACD6] text-nowrap w-full mt-2">Restart ('R' or 'Space')</button>
        
        </div>

      </div>
        </div>
        
      <div class="flex flex-col justify-center items-center gap-2 w-40 action-button mt-4">
        <button @click="changeDirection('up')">▲ (Z)</button>
        <div class="flex gap-2 justify-between">
          <button @click="changeDirection('left')">◄ (Q)</button>
          <button @click="changeDirection('down')">▼ (S)</button>
          <button @click="changeDirection('right')">►	(D)</button>
        </div>
        

      </div>
      
    </div>
    
    
  </main>
  
</template>

<style>
body {
  background: #17153B;
  color: #C8ACD6;
}
.tile {
  width: 30px;
  aspect-ratio: 1;
  background: #433D8B;
  /* border: 1px solid black; */
}

#gameBoard {
  display: flex;
  border: 2px solid #2E236C;
  background: #433D8B;


}

.restart-panel {
  position: absolute;
  left: 50%;
  top: 40%;
  transform: translate(-50%, -50%);
}

.line {
  display: flex;
  flex-direction: column;

}

.snake {
  background: #c185df;
  border: 2px solid #433D8B;
  border-radius: 20%;

}

.head {
  background: #AD49E1;
  border: none;
  border-radius: 30%;

}

.apple {
  background-color: #e733d8;
  border-radius: 40%;
}

button, select {
  text-wrap: nowrap;
  background-color: #433D8B;
  border: 2px solid 2E236C;
  min-width: 4rem;
  box-shadow: 3px 3px 0px 0px #C8ACD6;
  color: #C8ACD6;
  padding: 0.5em;

  transition: 100ms ease;
}

button:hover, select:hover {
  box-shadow: 5px 5px 0px 0px #C8ACD6;

}

button:active {
  box-shadow: 1px 1px 0px 0px #C8ACD6;

}

input, select {
  background-color: #433D8B;
}

/* Make an annimation of the text color of the New Best Score */
.bestScore {
  text-shadow: -5px 0px #433D8B;
  animation: colorChange 5s infinite;
}

@keyframes colorChange {
  0% {
    color: #C8ACD6;
  }
  20% {
    color: yellow;
  }
  40% {
    color: rgb(255, 0, 140);
  }
  60% {
    color: rgb(0, 162, 255);
  }
  80% {
    color: #4ee649;
  }
}

</style>
