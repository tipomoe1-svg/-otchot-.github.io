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
        input, select { background-color: #38373b; border: #f0f0f0 1px solid; padding: 3px; color: #f0f0f0; }
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
            <div class="section">
                <h2>Отчёт об играх</h2>
                <textarea v-model="gameReport" rows="10" style="width: 100%;" placeholder="Отчёт будет сгенерирован автоматически"></textarea>
            </div>
            <div id = "link-to-main"><a href = "../">НА ГЛАВНУЮ</a></div>
        </div>
    </div>

    <script>
        const { createApp, ref, computed } = Vue;

        const games = [
            // ... (список игр без изменений)
            {
                name: "Путаница",
                solo: false,
                adults: true,
                points: [],
                rounds: 5
            },
            // ... (остальные игры)
            {
                name: "Словарик",
                solo: true,
                adults: false,
                points: [4],
                rounds: 5
            },
            {
                name: "Песочница",
                solo: true,
                adults: true,
                points: [],
                rounds: 1
            },
            {
                name: "Три факта",
                solo: false,
                adults: true,
                points: [5, 3],
                rounds: 3
            },
            {
                name: "Дай пять",
                solo: false,
                adults: true,
                points: [],
                rounds: 3
            }
        ];

        createApp({
            setup() {
                const players = ref([{ id: generateId(), name: '', playerId:'', type:'kitten' }]);
                const gameSessions = ref([createGameSession()]);

                const gameReport = computed(() => {
                    const lines = [];
                    
                    lines.push(`Следи`: ID следующего | [catID]`);
                    lines.push(`Игровик`: ID Игровика | [catID]`);
                    lines.push(`Время`: время начала игры - время конца игры; ${new Date().getDate().toString().padStart(2, '0')}.${(new Date().getMonth() + 1).toString().padStart(2, '0')}.${new Date().getFullYear().toString().slice(-2)}`);
                    
                    // Игры
                    const gameLines = [];
                    gameSessions.value.forEach(session => {
                        if (session.selectedGame) {
                            const minRounds = session.selectedGame.rounds;
                            const playedRounds = session.rounds.length;
                            const gameCount = Math.floor(playedRounds / minRounds);
                            if (gameCount > 0) {
                                gameLines.push(`${session.selectedGame.name} (${gameCount})`);
                            }
                        }
                    });
                    lines.push(`Игра`: ${gameLines.join(', ')}`);
                    
                    // Котята
                    lines.push(`Котята`:);
                    
                    const adultPoints = {};
                    const kittenPoints = {};
                    
                    // Собираем баллы взрослых для распределения
                    gameSessions.value.forEach(session => {
                        if (session.selectedGame) {
                            session.rounds.forEach(round => {
                                const adultPlayersInRound = players.value.filter(p => p.type === 'adult' && (round.points[p.id] || 0) > 0);
                                const kittenPlayersInRound = players.value.filter(p => (p.type === 'kitten' || p.type === 'bs' || p.type === 'is') && (round.points[p.id] || 0) > 0);
                                
                                if (adultPlayersInRound.length > 0 && kittenPlayersInRound.length > 0) {
                                    adultPlayersInRound.forEach(adult => {
                                        const pointsToDistribute = round.points[adult.id] || 0;
                                        const pointsPerKitten = Math.floor(pointsToDistribute / kittenPlayersInRound.length);
                                        
                                        kittenPlayersInRound.forEach(kitten => {
                                            kittenPoints[kitten.id] = (kittenPoints[kitten.id] || 0) + pointsPerKitten;
                                        });
                                    });
                                }
                                
                                // Обычные баллы
                                players.value.forEach(player => {
                                    if (player.type !== 'adult' || kittenPlayersInRound.length === 0) {
                                        const points = round.points[player.id] || 0;
                                        if (player.type === 'adult') {
                                            adultPoints[player.id] = (adultPoints[player.id] || 0) + points;
                                        } else {
                                            kittenPoints[player.id] = (kittenPoints[player.id] || 0) + points;
                                        }
                                    }
                                });
                            });
                        }
                    });
                    
                    // Формируем строки для игроков
                    players.value.forEach(player => {
                        if (player.type !== 'adult') {
                            const totalPoints = kittenPoints[player.id] || 0;
                            const catID = player.playerId ? `cat${player.playerId}` : 'catID';
                            
                            if (player.type === 'bs') {
                                lines.push(`${player.playerId || 'ID'} | [${catID}] (${totalPoints}) [БС]`);
                            } else if (player.type === 'is') {
                                let visitedGames = 0;
                                gameSessions.value.forEach(session => {
                                    if (session.selectedGame) {
                                        const playerRounds = session.rounds.filter(round => (round.points[player.id] || 0) > 0).length;
                                        const gameCount = Math.floor(playerRounds / session.selectedGame.rounds);
                                        visitedGames += gameCount;
                                    }
                                });
                                lines.push(`${player.playerId || 'ID'} | [${catID}] (${totalPoints}; ${visitedGames}) [ИС]`);
                            } else {
                                lines.push(`${player.playerId || 'ID'} | [${catID}] (${totalPoints})`);
                            }
                        }
                    });
                    
                    return lines.join('\n');
                });

                function generateId() {
                    return Date.now().toString(36) + Math.random().toString(36).substr(2);
                }

                function addPlayer() {
                    players.value.push({ 
                        id: generateId(), 
                        name:'', 
                        playerId:'', 
                        type:'kitten' 
                    });
                }

                function removePlayer(index) {
                    if (players.value.length > 1) {
                        const removedPlayer = players.value.splice(index, 1)[0];
                        
                        // Удаляем баллы удалённого игрока из всех игр
                        gameSessions.value.forEach(gameSession => {
                            gameSession.rounds.forEach(round => {
                                if (round.points[removedPlayer.id] !== undefined) {
                                    delete round.points[removedPlayer.id];
                                }
                            });
                        });
                    }
                }

                function createGameSession() {
                    return {
                        id : generateId(),
                        searchQuery : '',
                        selectedGame : null,
                        filteredGames : [],
                        showSuggestions : false,
                        rounds : []
                    };
                }

                function addGameSession() {
                    gameSessions.value.push(createGameSession());
                }

                function removeGameSession(index) {
                    if (gameSessions.value.length > 1) {
                        gameSessions.value.splice(index, 1);
                    }
                }

                 function getAvailableGames() {
                     const soloPlayers = players.value.length === 1;
                     return games.filter(game => 
                         (!game.solo && soloPlayers) ? false :
                         (game.solo && !soloPlayers) ? false :
                         true
                     );
                 }

                 function filterGames(sessionIndex) {
                     const session = gameSessions.value[sessionIndex];
                     const query = session.searchQuery.toLowerCase();
                     
                     session.filteredGames = getAvailableGames().filter(game => 
                         game.name.toLowerCase().includes(query)
                     );
                 }

                 function selectGame(sessionIndex, game) {
                     const session = gameSessions.value[sessionIndex];
                     session.selectedGame = game;
                     session.searchQuery = game.name;
                     session.showSuggestions = false;
                     
                     // Создаём раунды
                     session.rounds = [];
                     for (let i = 0; i < game.rounds; i++) {
                         session.rounds.push({
                             points : {}
                         });
                     }
                 }

                 function removeRound(sessionIndex) {
                     const session = gameSessions.value[sessionIndex];
                     if (session.rounds.length > 1) {
                         session.rounds.pop();
                     }
                 }

                 function addRound(sessionIndex) {
                     const session = gameSessions.value[sessionIndex];
                     if (session.selectedGame) {
                         session.rounds.push({
                             points : {}
                         });
                     }
                 }

                 function addCustomRounds(sessionIndex) {
                     const session = gameSessions.value[sessionIndex];
                     if (session.selectedGame) {
                         const count = parseInt(prompt('Сколько раундов добавить?', '1'));
                         if (count > 0) {
                             for (let i = 0; i < count; i++) {
                                 session.rounds.push({
                                     points : {}
                                 });
                             }
                         }
                     }
                 }

                 function filteredPlayers(gameSession) {
                     if (!gameSession.selectedGame) return players.value;
                     
                     if (!gameSession.selectedGame.adults) {
                         return players.value.filter(player => player.type !== 'adult');
                     }
                     
                     return players.value;
                 }
                 
                 document.addEventListener('click', (e) => {
                     if (!e.target.matches('input')) {
                         gameSessions.value.forEach(session => {
                             session.showSuggestions = false;
                         });
                     }
                 });
                 
                 return {
                     players,
                     gameSessions,
                     gameReport,
                     addPlayer,
                     removePlayer,
                     addGameSession,
                     removeGameSession,
                     filterGames,
                     selectGame,
                     removeRound,
                     addRound,
                     addCustomRounds,
                     filteredPlayers
                 };
             }
         }).mount('#app');
     </script>
 </body>
</html>
