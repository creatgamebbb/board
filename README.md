# board
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>WingSpan - 得点計算</title>
  <style>
    body {
      background-color: #f7f9fc;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      color: #333;
      padding: 20px;
      margin: 0;
    }
    h2 {
      color: #0077b6;
      text-align: center;
      margin-bottom: 20px;
    }
    #controls {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 15px;
      margin-bottom: 20px;
    }
    #player-forms {
      display: flex;
      gap: 20px;
      flex-wrap: wrap;
      justify-content: center;
    }
    .container {
      border: 1px solid #d1e3f8;
      box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
      padding: 15px;
      width: 300px;
      box-sizing: border-box;
      border-radius: 10px;
      background-color: #ffffff;
      transition: transform 0.3s, box-shadow 0.3s;
    }
    .container:hover {
      transform: translateY(-5px);
      box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
    }
    .player-name {
      font-weight: bold;
      cursor: pointer;
      margin-bottom: 10px;
      color: #023e8a;
      text-align: center;
    }
    .player-name-input {
      display: none;
      width: 100%;
      padding: 8px;
      margin-bottom: 10px;
      border: 1px solid #d1e3f8;
      border-radius: 5px;
      font-size: 14px;
    }
    .item-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 8px;
    }
    .item-row label {
      flex: 1;
      color: #555;
    }
    .item-row input {
      flex: 1;
      padding: 5px;
      margin-left: 10px;
      border: 1px solid #d1e3f8;
      border-radius: 5px;
      outline: none;
      transition: border-color 0.3s;
    }
    .item-row input:focus {
      border-color: #0077b6;
    }
    .result {
      font-weight: bold;
      margin-top: 15px;
      text-align: right;
      color: #0077b6;
    }
    #winner {
      margin-top: 20px;
      font-size: 18px;
      font-weight: bold;
      text-align: center;
      color: #d9534f;
    }
    #reset-button {
      padding: 10px 20px;
      background-color: #0077b6;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    #reset-button:hover {
      background-color: #005f99;
    }
    #history {
      margin-top: 20px;
      font-size: 14px;
      padding: 10px;
      border-top: 1px solid #d1e3f8;
      background-color: #ffffff;
      border-radius: 5px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    #history h3 {
      margin-top: 0;
      color: #555;
      text-align: center;
    }
  </style>
</head>
<body>
  <h2>WingSpan - 複数の得点計算</h2>

  <div id="controls">
    <button id="reset-button" onclick="resetScores()">リセット</button>
    <label for="player-count">プレイヤー数:</label>
    <select id="player-count" onchange="generatePlayerForms()">
      <option value="1">1</option>
      <option value="2">2</option>
      <option value="3">3</option>
      <option value="4">4</option>
      <option value="5">5</option>
    </select>
  </div>

  <div id="player-forms"></div>

  <div id="winner">勝者: まだ決まっていません</div>

  <!-- 履歴を表示する場所 -->
  <div id="history"></div>

  <script>
    let historyList = [];

    function generatePlayerForms() {
      const playerCount = parseInt(document.getElementById('player-count').value);
      const playerFormsContainer = document.getElementById('player-forms');

      // フォームをクリア
      playerFormsContainer.innerHTML = '';

      // 各項目の名前を配列で定義
      const itemNames = [
        "鳥カード",
        "ボーナスカード",
        "ラウンド終了時目的",
        "卵",
        "鳥カード上に蓄えた餌",
        "差し込んだ鳥カード"
      ];

      // プレイヤー数に応じてフォームを生成
      for (let i = 1; i <= playerCount; i++) {
        const container = document.createElement('div');
        container.className = 'container';
        container.id = `player-${i}`;

        // プレイヤー名の表示と編集フィールドを追加
        const playerName = document.createElement('div');
        playerName.className = 'player-name';
        playerName.textContent = `プレイヤー ${i}`;
        playerName.onclick = () => toggleNameInput(i);
        container.appendChild(playerName);

        const nameInput = document.createElement('input');
        nameInput.type = 'text';
        nameInput.className = 'player-name-input';
        nameInput.placeholder = `プレイヤー ${i} の名前`;
        nameInput.onblur = () => updatePlayerName(i);
        container.appendChild(nameInput);

        // 各項目の入力フィールドを生成
        itemNames.forEach((itemName) => {
          const itemRow = document.createElement('div');
          itemRow.className = 'item-row';

          const label = document.createElement('label');
          label.textContent = itemName;
          itemRow.appendChild(label);

          const input = document.createElement('input');
          input.type = 'number';
          input.className = 'score';
          input.placeholder = '数値を入力';
          input.oninput = () => calculateTotal(i);
          itemRow.appendChild(input);

          container.appendChild(itemRow);
        });

        const result = document.createElement('div');
        result.className = 'result';
        result.innerHTML = `合計: <span id="total-${i}">0</span>`;
        container.appendChild(result);

        playerFormsContainer.appendChild(container);
      }
    }

    function toggleNameInput(playerNumber) {
      const playerContainer = document.getElementById(`player-${playerNumber}`);
      const playerName = playerContainer.querySelector('.player-name');
      const nameInput = playerContainer.querySelector('.player-name-input');

      playerName.style.display = 'none';
      nameInput.style.display = 'block';
      nameInput.value = playerName.textContent;
      nameInput.focus();
    }

    function updatePlayerName(playerNumber) {
      const playerContainer = document.getElementById(`player-${playerNumber}`);
      const playerName = playerContainer.querySelector('.player-name');
      const nameInput = playerContainer.querySelector('.player-name-input');

      playerName.textContent = nameInput.value.trim() || `プレイヤー ${playerNumber}`;
      nameInput.style.display = 'none';
      playerName.style.display = 'block';
    }

    function calculateTotal(playerNumber) {
      const playerContainer = document.getElementById(`player-${playerNumber}`);
      const scoreInputs = playerContainer.getElementsByClassName('score');

      let total = 0;
      for (let input of scoreInputs) {
        total += parseFloat(input.value) || 0;
      }

      document.getElementById(`total-${playerNumber}`).textContent = total;
      updateWinner();
    }

    function updateWinner() {
      const playerCount = parseInt(document.getElementById('player-count').value);
      let highestScore = 0;
      let winnerName = "まだ決まっていません";

      for (let i = 1; i <= playerCount; i++) {
        const playerContainer = document.getElementById(`player-${i}`);
        const playerName = playerContainer.querySelector('.player-name').textContent;
        const totalScore = parseFloat(document.getElementById(`total-${i}`).textContent) || 0;

        if (totalScore > highestScore) {
          highestScore = totalScore;
          winnerName = playerName;
        }
      }

      document.getElementById('winner').textContent = `勝者: ${winnerName} (合計: ${highestScore}点)`;
    }

    function resetScores() {
      const playerCount = parseInt(document.getElementById('player-count').value);
      let roundSummary = "ラウンド結果:\n";

      // 現在のプレイヤー名と得点を保存
      for (let i = 1; i <= playerCount; i++) {
        const playerContainer = document.getElementById(`player-${i}`);
        const playerName = playerContainer.querySelector('.player-name').textContent;
        const totalScore = document.getElementById(`total-${i}`).textContent;

        roundSummary += `${playerName}: ${totalScore}点\n`;
      }

      historyList.push(roundSummary);
      displayHistory();

      // 各プレイヤーの得点をリセット
      for (let i = 1; i <= playerCount; i++) {
        const playerContainer = document.getElementById(`player-${i}`);
        const scoreInputs = playerContainer.getElementsByClassName('score');

        for (let input of scoreInputs) {
          input.value = '';
        }

        document.getElementById(`total-${i}`).textContent = '0';
      }

      document.getElementById('winner').textContent = '勝者: まだ決まっていません';
    }

    function displayHistory() {
      const historyContainer = document.getElementById('history');
      historyContainer.innerHTML = "<h3>履歴</h3>" + historyList.map(entry => `<div>${entry}</div>`).join('');
    }

    window.onload = generatePlayerForms;
  </script>
</body>
</html>
