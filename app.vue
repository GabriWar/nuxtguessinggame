<template>
    <title>Jogo da adivinhação</title>
  <head>
    <link rel='icon' type='image/svg+xml' href='/favicon.png'>
  </head>
    <div class="bg">
      <h1>Advinhe o numero</h1>
      <div class="game">
        <p v-if="!gameStarted">Clique no botão para iniciar o jogo</p>
        <button v-if="!gameStarted" @click="startGame">Iniciar Jogo</button>
        <p v-if="gameStarted && !gameOver">adivinhe um numero entre 0 e 100:</p>
        <input v-model="guess" v-if="gameStarted && !gameOver" type="number" min="1" max="100" @keyup.enter="checkGuess" />
        <button v-if="gameStarted && !gameOver" @click="checkGuess">enter</button>
        <p v-if="gameOver">voce acertou o numero em {{ attempts }} tentativas.</p>
        <p v-if="!gameOver && higher">o número é menor</p>
        <p v-if="!gameOver && lower">o número é maior</p>
        <p v-if="gameOver">Pontuacao: {{ milliseconds/attempts }} </p>
        <p>tentativas: {{ attempts }}</p>
        <p v-if="gameOver">Tempo: {{ milliseconds }} milissegundos</p>
      </div>
      <input v-model="playerName" v-if="gameOver" type="text" placeholder="digite seu nome" class="input" />
      <button v-if="gameOver" @click="sendScore">enviar</button>
      <p class="aviso" v-if="gameOver">sem caracteres especiais, por favor :)</p>
      <table class="table" v-if="highScores.length > 0">
        <thead>
          <tr>
            <th>Rank</th>
            <th>Nome</th>
            <th>Pontuacao</th>
            <th>Tempo</th>
            <th>Tentativas</th>
          </tr>
        </thead>
        <tbody style="text-align: center;">
          <tr v-for="(score, index) in highScores" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ score.names }}</td>
            <td>{{ score.scores }}</td>
            <td>{{ score.time }}</td>
            <td>{{ score.tentativas }}</td>

          </tr>
        </tbody>
      </table>
      <p v-else>sem pontuacoes</p>
    </div>
    <div class="creditos"><a href="https://github.com/GabriWar"> Criado por: Gabriel Guerra</a></div>
  </template>

  <script setup>
  import { ref, onMounted } from 'vue';
  const client = useSupabaseClient();
  const targetNumber = Math.floor(Math.random() * 100) + 1;
  const guess = ref('');
  const playerName = ref('');
  const attempts = ref(0);
  const gameOver = ref(false);
  const higher = ref(false);
  const lower = ref(false);
  const highScores = ref([]);
  const gameStarted = ref(false);
  const timer = ref(null);
  const milliseconds = ref(0);
  

  async function fetchHighScores() {
    const { data, error } = await client
      .from('scores')
      .select('*')
      .order('scores', { ascending: true })
      .limit(10);
    if (error) {
      console.error(error);
    } else {
      highScores.value = data;
    }
  }

  onMounted(fetchHighScores);

  function startGame() {
    gameStarted.value = true;
    startTimer();
  }


  function startTimer() {
    timer.value = setInterval(() => {
      milliseconds.value++;
    }, 1);
  }

  function checkGuess() {
    attempts.value++;

    if (parseInt(guess.value) === targetNumber) {
      gameOver.value = true;
      console.log(timer.value);
      clearInterval(timer.value);
    } else if (parseInt(guess.value) > targetNumber) {
      higher.value = true;
      lower.value = false;
    } else {
      higher.value = false;
      lower.value = true;
    }
  }

  async function sendScore() {
    const playerNameRegex = /^[a-zA-Z0-9\s]+$/;

    if (!playerNameRegex.test(playerName.value)) {
      console.error('nome invalido');
      return;
    }

    const { error } = await client
      .from('scores')
      .insert([{ names: playerName.value, scores: milliseconds.value/attempts.value, time: milliseconds.value, tentativas: attempts.value }]);

    if (error) {
      console.error(error);
    } else {
      console.log('score enviada');
      fetchHighScores();
      gameOver.value = false;
    }
  }

  </script>

<style>
html,
body,
input,
button,
thead,
tbody,
tr,
th,
td {
  background-color: #000;
  color: #fff;
  border-color: white;
  font-size: clamp(24px, 5vw, 48px);
}
h1 {
  text-align: center;
}

th {
  text-align: center;
  padding-left: 5vw;
  padding-right: 5vw;
}

td {
  text-align: center;
  padding-left: 5vw;
  padding-right: 5vw;
}

a {
  color: #fff;
}

.creditos {
  font-size: 20px;
  bottom: 0;
  right: 0;
  position: absolute;
}
.aviso{
  color: grey;
  margin-top:10px;
}
</style>
