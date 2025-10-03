<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Liga dos Super Heróis Metalúrgicos</title>
<style>
  body {
    font-family: 'Press Start 2P', monospace;
    background-color: #e6d2aa;
    color: #2d2d2d;
    display: flex;
    justify-content: center;
    padding: 20px;
  }
  .game-container {
    background-color: #dcd0b1;
    border: 4px solid #2d2d2d;
    padding: 20px;
    width: 320px;
  }
  .indicator, .machine, .player-actions {
    border: 2px solid #2d2d2d;
    margin: 10px 0;
    padding: 5px;
  }
  .indicator h3, .machine h3 {
    margin: 5px 0;
    font-size: 12px;
  }
  .status-bar {
    display: flex;
    height: 10px;
    background: #555;
    margin-top: 5px;
  }
  .status-fill {
    height: 100%;
  }
  .smiley {
    font-size: 18px;
    margin-right: 5px;
  }
  .buttons button {
    margin: 2px;
    padding: 5px;
    font-size: 10px;
  }
  .machine-button {
    cursor: pointer;
    margin: 2px;
    padding: 5px;
    font-size: 10px;
    border: 2px solid #2d2d2d;
    background-color: #a3b18a;
  }
  .machine-button.broken { background-color: #d95d39; }
</style>
</head>
<body>
<div class="game-container">

  <!-- Indicadores do Metalúrgico -->
  <div class="indicator">
    <h3>Saúde do Metalúrgico</h3>
    <div>😃 Feliz <span id="health-text">bem alimentado</span></div>
    <div class="status-bar"><div id="health-bar" class="status-fill" style="width: 100%; background: green;"></div></div>
  </div>

  <!-- Estado das Máquinas -->
  <div class="machine">
    <h3>Estado das Máquinas</h3>
    <div class="buttons">
      <button class="machine-button" onclick="interactMachine('funciona')">🟢 Funciona</button>
      <button class="machine-button" onclick="interactMachine('lubrificacao')">🟡 Lubrificação</button>
      <button class="machine-button" onclick="interactMachine('quebrada')">🔴 Quebrada</button>
    </div>
    <div id="machine-status">Funcionando bem</div>
  </div>

  <!-- Energia / Combustível -->
  <div class="indicator">
    <h3>Energia / Combustível</h3>
    <div class="status-bar"><div id="energy-bar" class="status-fill" style="width: 100%; background: orange;"></div></div>
  </div>

  <!-- Humor do Bicho -->
  <div class="indicator">
    <h3>Humor do Bicho</h3>
    <div id="pet-humor">😃 Feliz</div>
  </div>

  <!-- Ações do Jogador -->
  <div class="player-actions">
    <h3>Ações do Jogador</h3>
    <button onclick="feed()">Dar comida 🍔</button>
    <button onclick="maintenance()">Manutenção 🛠️</button>
    <button onclick="security()">Segurança 🛡️</button>
    <button onclick="produce()">Produção 🏭</button>
    <button onclick="refuel()">Recarregar 🔥</button>
  </div>

</div>

<script>
  let health = 100;
  let energy = 100;
  let petHappy = true;
  let machineStatus = 'funciona';

  function updateUI() {
    // Metalúrgico
    document.getElementById('health-bar').style.width = health + '%';
    document.getElementById('health-bar').style.background = health > 50 ? 'green' : health > 20 ? 'yellow' : 'red';
    document.getElementById('health-text').innerText = health > 50 ? 'bem alimentado' : health > 20 ? 'cansado' : 'muito cansado';

    // Energia
    document.getElementById('energy-bar').style.width = energy + '%';
    document.getElementById('energy-bar').style.background = energy > 50 ? 'orange' : energy > 20 ? 'yellow' : 'red';

    // Humor do Bicho
    document.getElementById('pet-humor').innerText = petHappy ? '😃 Feliz' : '😢 Triste';

    // Máquina
    let machineText = '';
    if(machineStatus === 'funciona') machineText = 'Funcionando bem';
    else if(machineStatus === 'lubrificacao') machineText = 'Precisa de lubrificação/limpeza';
    else machineText = 'Quebrada, precisa conserto';
    document.getElementById('machine-status').innerText = machineText;
  }

  function feed() { health = Math.min(health + 20, 100); updatePetMood(); updateUI(); }
  function maintenance() { if(machineStatus !== 'funciona') machineStatus='funciona'; updatePetMood(); updateUI(); }
  function security() { health = Math.min(health + 10, 100); updatePetMood(); updateUI(); }
  function produce() { energy = Math.max(energy - 15, 0); updatePetMood(); updateUI(); }
  function refuel() { energy = 100; updatePetMood(); updateUI(); }

  function interactMachine(status) {
    machineStatus = status;
    updatePetMood();
    updateUI();
  }

  function updatePetMood() {
    petHappy = health > 50 && machineStatus === 'funciona';
  }

  // Atualiza tela inicial
  updateUI();
</script>

</body>
</html>

