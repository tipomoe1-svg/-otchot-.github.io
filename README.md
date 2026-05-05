<!DOCTYPE html>

<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Отчётник!</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <style>
        body { font-family: Arial, Helvetica, sans-serif; background-color: #242326; color: #f0f0f0 }
        .container { max-width: 800px; margin: 0 auto; padding: 20px; }
        .section { margin-bottom: 30px; border: 1px solid #ccc; padding: 20px 15px; border-radius: 5px; background-color: #38373b; }
        .player-row, .game-row { display: flex; align-items: center; margin-bottom: 10px; gap: 10px; }
        .add-btn { background: #4CAF50; color: white; border: none; padding: 5px 10px; border-radius: 3px; cursor: pointer; }
        .delete-btn { background: #f44336; color: white; border: none; padding: 5px 10px; border-radius: 3px; cursor: pointer; }
        .rounds-list { margin: 10px 0; padding-left: 20px; }
        .round-item { margin-bottom: 10px; }
        .player-points { display: flex; flex-direction: column; gap: 5px; margin-top: 5px; }
        .player-point-row { display: flex; align-items: center; gap: 10px; }
        .search-container { position: relative; }
        .suggestions { position: absolute; background: white; border: 1px solid #ccc; max-height: 150px; overflow-y: auto; z-index: 1000; width: 100%; }
        .suggestion-item { padding: 5px 10px; cursor: pointer; font-size: 10pt; color: #f0f0f0; background: #38373b; }
        .suggestion-item:hover { background: #4d4b52; }
        #round-buttons > button { display: inline-block; margin-top: 10px; margin-left: 15px; }
        .section > button { margin-top: 10px; }
        a { text-decoration: none; font-weight: bold; color: #f0f0f0 }
        h4 { margin-bottom: 10px; }
        h2 { margin-top: 10px; }
        input, select { background-color: #38373b !important; border-color:#f0f0f0 !important;
            padding: 3px !important;
            color:#f0f0f0 !important;
            border-radius:3px !important;
            box-shadow:none !important;
            outline:none !important;
            width:max-content;
            min-width:max-content;
            max-width:max-content;
            flex-grow :1;
         }
    </style>
</head>
<body>
    <div id="app">
        <div class="container">
            <!-- Блок игроков -->
            <div class="section">
                <h2>Игроки</h2>
                <div v-for="(player, index) in players" :key="player.id" class="player-row">
                    <input v-model="player.name" placeholder="Имя игрока">
                    <input v-model="player.playerId" placeholder="ID игрока">
                    <label>
                        <input type="radio" v-model="player.type" value="kitten"> Котёнок
                    </label>
                    <label>
                        <input type="radio" v-model="player.type" value="bs"> БС
                    </label>
                    <label>
                        <input type="radio" v-model="player.type" value="is"> ИС
                    </label>
                    <label>
                        <input type="radio" v-model="player.type" value="adult"> Взрослый
                    </label>
                    <button @click="removePlayer(index)" class="delete-btn" :disabled="players.length === 1">Удалить</button>
                </div>
                <button @click="addPlayer" class="add-btn">+ Добавить игрока</button>
            </div>
                    <!-- Блок игр -->
        <div class="section">
            <h2>Игры</h2>
            <div v-for="(gameSession, index) in gameSessions" :key="gameSession.id" class="game-section">
                <div class="game-row">
                    <div class="search-container">
                        <input 
                            v-model="gameSession.searchQuery" 
                            @input="filterGames(index)"
                            @focus="gameSession.showSuggestions = true"
                            placeholder="Поиск..."
                        >
                        <div v-if="gameSession.showSuggestions && gameSession.filteredGames.length" class="suggestions">
                            <div 
                                v-for="game in gameSession.filteredGames" 
                                :key="game.name"
                                @click="selectGame(index, game)"
                                class="suggestion-item"
                            >
                                {{ game.name }}
                            </div>
                        </div>
                    </div>
                    <button @click="removeGameSession(index)" class="delete-btn" :disabled="gameSessions.length === 1">Удалить игру</button>
                </div>

                <!-- Раунды -->
                <div v-if="gameSession.selectedGame" class="rounds-list">
                    <div v-for="(round, roundIndex) in gameSession.rounds" :key="roundIndex" class="round-item">
                        <h4>Раунд {{ roundIndex + 1 }}</h4>
                        <div class="player-points">
                            <div v-for="player in filteredPlayers(gameSession)" :key="player.id" class="player-point-row">
                                <span>{{ player.name }}</span>
                                <select v-if="gameSession.selectedGame.points.length" v-model.number="round.points[player.id] || 0">
                                    <option v-for="point in [...gameSession.selectedGame.points, 0]" :key="point" :value="point">
                                        {{ point }}
                                    </option>
                                </select>
                                <input v-else type="number" v-model.number="round.points[player.id] || 0" placeholder="Баллы">
                            </div>
                        </div>
                    </div>
                    <div id = "round-buttons">
                        <button @click="addRound(index)">+ Добавить раунд</button>
                        <button @click="addCustomRounds(index)">+ Добавить несколько раундов</button>
                        <button @click="removeRound(index)" :disabled="gameSession.rounds.length === 1">- Удалить раунд</button>
                    </div>
                </div>
            </div>
            <button @click="addGameSession" class="add-btn">+ Добавить игру</button>
        </div>

        <!-- Отчёт -->
        <div class="section">
            <h2>Отчёт об играх</h2>
            <textarea v-model.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal.trimmed.literal= "gameReport" rows="15" style="width: 100%; font-family: monospace;" readonly placeholder="Отчёт будет сгенерирован автоматически"></textarea>
        </div>

        <!-- Ссылка (для GitHub Pages) -->
        <a href="#" id = "link-to-main">НА ГЛАВНУЮ</a> 

    </div>
</div>

<script>
    const { createApp, ref, computed } = Vue;

    const games = [
        // ... (весь список игр из твоего кода)
        // Я оставил их без изменений, чтобы не раздувать ответ.
        // Просто скопируй их отсюда или из своего файла.
        {
            name:"Путаница",
            solo:false,
            adults:true,
            points:[7,3],
            rounds:5
        },
        {
            name:"Догонялки",
            solo:false,
            adults:false,
            points:[7,3],
            rounds:3
        },
         {
            name:"Викторина",
            solo:false,
            adults:true,
            points:[4,2],
            rounds:5
         },
         {
             name:"Котячий патруль",
             solo:true,
             adults:true,
             points:[10,5],
             rounds:1
         },
         {
             name:"Поле чудес",
             solo:false,
             adults:true,
             points:[10,5],
             rounds:3
         },
         {
             name:"Липучки",
             solo:false,
             adults:false,
             points:[4,2],
             rounds:3
         },
         {
             name:"Составь слово",
             solo:false,
             adults:false,
             points:[4,2],
             rounds:5
         },
         {
             name:"Расшифруй",
             solo:true,
             adults:true,
             points:[4,2],
             rounds:5
         },
         {
             name:"Займи клетку",
             solo:false,
             adults:true,
             points:[5,3],
             rounds:5
         },
         {
             name:"Погоня",
             solo:false,
             adults:false,
             points:[5,3],
             rounds:3
         },
         {
             name:"Облом",
             solo:false,
             adults:false,
             points:[4,1],
             rounds:5
         },
         {
             name:"Лабиринт",
             solo:true,
             adults:true,
             points:[5,3,1],
             rounds:3
         },
         {
              name:"Охота на бабочек",
              solo:true,
              adults:false,
              points:[5,3],
              rounds:1
          },
          {
              name:"Три буквы",
              solo:false,
              adults:true,
              points:[3,1],
              rounds:5
          },
          {
              name:"Клановое имя",
              solo:true,
              adults:true,
              points:[5,3],
              rounds:5
          },
          {
              name:"Цель",
              solo:true,
              adults:false,
              points:[5,3,1],
              rounds:3
          },
          {
              name:"Словарик",
              solo:true,
              adults:false,
              points:[4],
              rounds:5
          },
          {
              name:"Песочница",
              solo:true,
              adults:true,
              points:[4],
              rounds:1
          },
          {
               name:"Три факта",
               solo:false,
               adults:true,
               points:[5,3],
               rounds:3
           },
           {
