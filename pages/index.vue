<template>
  <div id="canvas" v-if="!gameOver">
    <div id="menu" v-if="menuVisible">
      <template v-if="!gameStarted">
        <button class="menu-button" @click="startGame">Iniciar Partida</button>
        <button class="menu-button" @click="showLeaderboard">Placar</button>
        <button class="menu-button" @click="quitGame">Sair</button>
      </template>
      <template v-else>
        <button class="menu-button" @click="resumeGame">Resumir Partida</button>
        <button class="menu-button" @click="showLeaderboard">Placar</button>
        <button class="menu-button" @click="quitGame">Sair</button>
      </template>
    </div>
    <div id="deep-space" />
    <div id="space-field">
      <SpaceObject id="spaceship" class="spaceship" :data="spaceField.ship" resolution="2" />

      <SpaceObject class="asteroid" :data="asteroid" resolution="2"
                  :key="asteroid.center"
                  v-for="asteroid in spaceField.asteroids" />

      <SpaceObject class="missile" :data="missile" resolution="2"
                  :key="missile.center"
                  v-for="missile in spaceField.missiles" />

      <SpaceObject class="explosion" :data="explosion" resolution="2"
                  :key="explosion.center"
                  v-for="explosion in spaceField.explosions" />
    </div>
  </div>
  <div id="game-over" v-else>
    <div id="deep-space" />
    <h1 class="game-over-text">Game Over</h1>
  </div>
  <div id="leaderboard-modal" v-if="showLeaderboardModal">
      <div class="modal-content">
        <h2 class="modal-heading">Leaderboard</h2>
        <ul>
          <li v-for="(entry, index) in leaderboard" :key="index">
            <p>Date: {{ entry.datetime }}</p>
            <p>Asteroids Destroyed: {{ entry.asteroidsDestroyed }}</p>
            <p>Points: {{ entry.points }}</p>
          </li>
        </ul>
        <button @click="closeLeaderboard">Close</button>
      </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import leaderboardData from './Leaderboard.json';

const menuVisible = ref(true);
const gameStarted = ref(false);
const gamePaused = ref(false);
const gameOver = ref(false);
const leaderboard = ref([]);
const showLeaderboardModal = ref(false);


const startGame = () => {
  menuVisible.value = false;
  gameStarted.value = true;
  gamePaused.value = false;
};

const resumeGame = () => {
  menuVisible.value = false;
  gamePaused.value = false;
};

const showLeaderboard = () => {
  leaderboard.value = leaderboardData;
  showLeaderboardModal.value = true; // Show the leaderboard modal when the button is clicked
};

const closeLeaderboard = () => {
  showLeaderboardModal.value = false; // Hide the leaderboard modal when the "Close" button is clicked
};

const quitGame = () => {
  gameOver.value = true;
};

const {
  data: spaceField,
  refresh: updateSpaceField
} = await $get("/space-field");

onMounted(() => {
  window.addEventListener("keydown", async (event) => {
    if (menuVisible.value || gamePaused.value) return;

    const keyToCommand = {
      "ArrowUp": "MOVE_SHIP_UP",
      "ArrowDown": "MOVE_SHIP_DOWN",
      "ArrowRight": "MOVE_SHIP_RIGHT",
      "ArrowLeft": "MOVE_SHIP_LEFT",
      "Space": "LAUNCH_MISSILE",
      "Escape": "PAUSE_GAME" ,
    };

    const command = keyToCommand[event.code];

    if (command === undefined) return;

    console.log(`Triggering command: ${command}`);
    if (command === "PAUSE_GAME") {
        menuVisible.value = true;
        gamePaused.value = true;
      } else {
      await $post("/ship/commands", { command });
    }
  });

  window.setInterval(() => {
    if (!menuVisible.value && !gamePaused.value) {
      updateSpaceField();
    }
  }, 60);
});
</script>

<style>

@font-face {
  font-family: 'Font';
  src: url('~/pages/fonts/Work_Sans/static/WorkSans-Medium.ttf') format('truetype');
}

#canvas {
  height: calc(100vh - 4rem);
  width: calc(100vw - 4rem);

  padding: 2rem;

  background-color: #36bbf5;
  overflow: hidden;

  position: relative;

  display: flex;
  justify-content: center;
  align-items: center;
}

#game-over {
  height: calc(100vh - 4rem);
  width: calc(100vw - 4rem);

  padding: 2rem;

  background-color: #36bbf5;
  overflow: hidden;

  position: relative;

  display: flex;
  justify-content: center;
  align-items: center;
}

  .game-over-text {
    font-family: 'Font';
    font-size: 6rem;
    color: #fff;
    position: relative;
    z-index: 1;
  }

#menu {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .menu-button {
    margin-bottom: 1rem;
    padding: 1rem 2rem;
    font-family: 'Font';
    font-size: 1.5rem;
    background-color: #36bbf5;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  .menu-button:hover {
    background-color: #2ca4e1;
  }
  
  .menu-button:last-child {
    margin-bottom: 0;
  }
  
  @keyframes slide {
    0% {
      transform: translate(1px);
    }
    50% {
      transform: translate(-1px);
    }
    100% {
      transform: translate(1px);
    }
  }

#leaderboard-modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 999;
}

.modal-content {
  background-color: #36bbf5; /* Match the color with the other menu buttons */
  padding: 20px;
  border-radius: 5px;
  text-align: center; /* Center the content */
  color: #fff; /* Text color */
}

.modal-heading {
  font-size: 24px;
  margin-bottom: 20px;
}

#deep-space {
  height: calc(100% - 4rem);
  width: calc(100% - 4rem);

  background-image: url("~/assets/space.png");
  background-origin: content-box;
  animation: slide 3s linear infinite;

  position: absolute;
  z-index: 0;
}

#space-field {
  height: calc(100% - 4rem);
  width: calc(100% - 4rem);

  position: relative;
}

.spaceship {
  background-image: url("~/assets/spaceship.png");
}

.asteroid {
  background-image: url("~/assets/asteroid.png");
}

.missile {
  background-image: url("~/assets/missile.png");
}

.explosion {
  background-image: url("~/assets/explosion.png");
}
</style>
