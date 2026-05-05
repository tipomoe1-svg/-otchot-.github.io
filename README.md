<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>РћС‚С‡С‘С‚РЅРёРє!</title>
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
        input { background-color: #38373b; border: #f0f0f0 1px solid; padding: 3px; color: #f0f0f0; }
    </style>
</head>
<body>
    <div id="app">
        <div class="container">
            <!-- Р‘Р»РѕРє РёРіСЂРѕРєРѕРІ -->
            <div class="section">
                <h2>РРіСЂРѕРєРё</h2>
                <div v-for="(player, index) in players" :key="player.id" class="player-row">
                    <input v-model="player.name" placeholder="РРјСЏ РёРіСЂРѕРєР°">
                    <input v-model="player.playerId" placeholder="ID РёРіСЂРѕРєР°">
                    
                    <label>
                        <input type="radio" v-model="player.type" value="kitten"> РљРѕС‚С‘РЅРѕРє
                    </label>
                    <label>
                        <input type="radio" v-model="player.type" value="bs"> Р‘РЎ
                    </label>
                    <label>
                        <input type="radio" v-model="player.type" value="is"> РРЎ
                    </label>
                    <label>
                        <input type="radio" v-model="player.type" value="adult"> Р’Р·СЂРѕСЃР»С‹Р№
                    </label>
                    
                    <button @click="removePlayer(index)" class="delete-btn" :disabled="players.length === 1">РЈРґР°Р»РёС‚СЊ</button>
                </div>
                <button @click="addPlayer" class="add-btn">+ Р”РѕР±Р°РІРёС‚СЊ РёРіСЂРѕРєР°</button>
            </div>

            <!-- Р‘Р»РѕРє РёРіСЂ -->
            <div class="section">
                <h2>РРіСЂС‹</h2>
                <div v-for="(gameSession, index) in gameSessions" :key="gameSession.id" class="game-section">
                    <div class="game-row">
                        <div class="search-container">
                            <input 
                                v-model="gameSession.searchQuery" 
                                @input="filterGames(index)"
                                @focus="gameSession.showSuggestions = true"
                                placeholder="РџРѕРёСЃРє..."
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
                        <button @click="removeGameSession(index)" class="delete-btn" :disabled="gameSessions.length === 1">РЈРґР°Р»РёС‚СЊ РёРіСЂСѓ</button>
                    </div>

                    <!-- Р Р°СѓРЅРґС‹ -->
                    <div v-if="gameSession.selectedGame" class="rounds-list">
                        <div v-for="(round, roundIndex) in gameSession.rounds" :key="roundIndex" class="round-item">
                            <h4>Р Р°СѓРЅРґ {{ roundIndex + 1 }}</h4>
                            <div class="player-points">
                                <div v-for="player in filteredPlayers(gameSession)" :key="player.id" class="player-point-row">
                                    <span>{{ player.name }}</span>
                                    <select v-if="gameSession.selectedGame.points.length" v-model="round.points[player.id]">
                                        <option v-for="point in [...gameSession.selectedGame.points, 0]" :key="point" :value="point">
                                            {{ point }}
                                        </option>
                                    </select>
                                    <input v-else type="number" v-model.number="round.points[player.id]" placeholder="Р‘Р°Р»Р»С‹">
                                </div>
                            </div>
                        </div>
                        <div id = "round-buttons">
                            <button @click="addRound(index)">+ Р”РѕР±Р°РІРёС‚СЊ СЂР°СѓРЅРґ</button>
                            <button @click="addCustomRounds(index)">+ Р”РѕР±Р°РІРёС‚СЊ РЅРµСЃРєРѕР»СЊРєРѕ СЂР°СѓРЅРґРѕРІ</button>
                            <button @click="removeRound(index)" :disabled="gameSession.rounds.length === 1">- РЈРґР°Р»РёС‚СЊ СЂР°СѓРЅРґ</button>
                        </div>
                    </div>
                </div>
                <button @click="addGameSession" class="add-btn">+ Р”РѕР±Р°РІРёС‚СЊ РёРіСЂСѓ</button>
            </div>
            <div class="section">
                <h2>РћС‚С‡С‘С‚ РѕР± РёРіСЂР°С…</h2>
                <textarea v-model="gameReport" rows="10" style="width: 100%;" placeholder="РћС‚С‡С‘С‚ Р±СѓРґРµС‚ СЃРіРµРЅРµСЂРёСЂРѕРІР°РЅ Р°РІС‚РѕРјР°С‚РёС‡РµСЃРєРё"></textarea>
            </div>
            <div id = "link-to-main"><a href = "../">РќРђ Р“Р›РђР’РќРЈР®</a></div>
        </div>
    </div>

    <script>
        const { createApp, ref, computed, watch } = Vue;

        const games = [
            {
                name: "РџСѓС‚Р°РЅРёС†Р°",
                solo: false,
                adults: true,
                points: [],
                rounds: 5
            },
            {
                name: "Р”РѕРіРѕРЅСЏР»РєРё",
                solo: false,
                adults: false,
                points: [7, 3],
                rounds: 3
            },
            {
                name: "Р’РёРєС‚РѕСЂРёРЅР°",
                solo: false,
                adults: true,
                points: [4, 2],
                rounds: 5
            },
            {
                name: "РљРѕС‚СЏС‡РёР№ РїР°С‚СЂСѓР»СЊ",
                solo: true,
                adults: true,
                points: [10, 5],
                rounds: 1
            },
            {
                name: "РџРѕР»Рµ С‡СѓРґРµСЃ",
                solo: false,
                adults: true,
                points: [10, 5],
                rounds: 3
            },
            {
                name: "Р›РёРїСѓС‡РєРё",
                solo: false,
                adults: false,
                points: [4, 2],
                rounds: 3
            },
            {
                name: "РЎРѕСЃС‚Р°РІСЊ СЃР»РѕРІРѕ",
                solo: false,
                adults: false,
                points: [4, 2],
                rounds: 5
            },
            {
                name: "Р Р°СЃС€РёС„СЂСѓР№",
                solo: true,
                adults: true,
                points: [],
                rounds: 5
            },
            {
                name: "Р—Р°Р№РјРё РєР»РµС‚РєСѓ",
                solo: false,
                adults: true,
                points: [5, 3],
                rounds: 5
            },
            {
                name: "РџРѕРіРѕРЅСЏ",
                solo: false,
                adults: false,
                points: [5, 3],
                rounds: 3
            },
            {
                name: "РћР±Р»РѕРј",
                solo: false,
                adults: false,
                points: [4, 1],
                rounds: 5
            },
            {
                name: "Р›Р°Р±РёСЂРёРЅС‚",
                solo: true,
                adults: true,
                points: [5, 3, 1],
                rounds: 3
            },
            {
                name: "РћС…РѕС‚Р° РЅР° Р±Р°Р±РѕС‡РµРє",
                solo: true,
                adults: false,
                points: [],
                rounds: 1
            },
            {
                name: "РўСЂРё Р±СѓРєРІС‹",
                solo: false,
                adults: true,
                points: [3, 1],
                rounds: 5
            },
            {
                name: "РљР»Р°РЅРѕРІРѕРµ РёРјСЏ",
                solo: true,
                adults: true,
                points: [5, 3],
                rounds: 5
            },
            {
                name: "Р¦РµР»СЊ",
                solo: true,
                adults: false,
                points: [5, 3, 1],
                rounds: 3
            },
            {
                name: "РЎР»РѕРІР°СЂРёРє",
                solo: true,
                adults: false,
                points: [4],
                rounds: 5
            },
            {
                name: "РџРµСЃРѕС‡РЅРёС†Р°",
                solo: true,
                adults: true,
                points: [],
                rounds: 1
            },
            {
                name: "РўСЂРё С„Р°РєС‚Р°",
                solo: false,
                adults: true,
                points: [5, 3],
                rounds: 3
            },
            {
                name: "Р”Р°Р№ РїСЏС‚СЊ",
                solo: false,
                adults: true,
                points: [],
                rounds: 3
            }
        ];

        createApp({
            setup() {
                const players = ref([{ id: generateId(), name: '', isAdult: false }]);
                const gameSessions = ref([createGameSession()]);
                const gameReport = computed(() => {
                    const lines = [];
                    
                    lines.push(`РЎР»РµРґРёР»: ID СЃР»РµРґСЏС‰РµРіРѕ | [catID]`);
                    lines.push(`РРіСЂРѕРІРёРє: ID РРіСЂРѕРІРёРєР° | [catID]`);
                    lines.push(`Р’СЂРµРјСЏ: РІСЂРµРјСЏ РЅР°С‡Р°Р»Р° РёРіСЂ - РІСЂРµРјСЏ РєРѕРЅС†Р° РёРіСЂ; ${new Date().getDate().toString().padStart(2, '0')}.${(new Date().getMonth() + 1).toString().padStart(2, '0')}.${new Date().getFullYear().toString().slice(-2)}`);
                    
                    // РРіСЂС‹
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
                    lines.push(`РРіСЂР°: ${gameLines.join(', ')}`);
                    
                    // РљРѕС‚СЏС‚Р°
                    lines.push(`РљРѕС‚СЏС‚Р°:`);
                    
                    const adultPoints = {};
                    const kittenPoints = {};
                    
                    // РЎРѕР±РёСЂР°РµРј Р±Р°Р»Р»С‹ РІР·СЂРѕСЃР»С‹С… РґР»СЏ СЂР°СЃРїСЂРµРґРµР»РµРЅРёСЏ
                    gameSessions.value.forEach(session => {
                        if (session.selectedGame) {
                            session.rounds.forEach(round => {
                                const adultPlayersInRound = players.value.filter(p => p.type === 'adult' && round.points[p.id] > 0);
                                const kittenPlayersInRound = players.value.filter(p => (p.type === 'kitten' || p.type === 'bs' || p.type === 'is') && round.points[p.id] > 0);
                                
                                if (adultPlayersInRound.length > 0 && kittenPlayersInRound.length > 0) {
                                    adultPlayersInRound.forEach(adult => {
                                        const pointsToDistribute = round.points[adult.id] || 0;
                                        const pointsPerKitten = Math.floor(pointsToDistribute / kittenPlayersInRound.length);
                                        
                                        kittenPlayersInRound.forEach(kitten => {
                                            kittenPoints[kitten.id] = (kittenPoints[kitten.id] || 0) + pointsPerKitten;
                                        });
                                    });
                                }
                                
                                // РћР±С‹С‡РЅС‹Рµ Р±Р°Р»Р»С‹
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
                    
                    // Р¤РѕСЂРјРёСЂСѓРµРј СЃС‚СЂРѕРєРё РґР»СЏ РёРіСЂРѕРєРѕРІ
                    players.value.forEach(player => {
                        if (player.type !== 'adult') {
                            const totalPoints = kittenPoints[player.id] || 0;
                            const catID = player.playerId ? `cat${player.playerId}` : 'catID';
                            
                            if (player.type === 'bs') {
                                lines.push(`${player.playerId || 'ID'} | [${catID}] (${totalPoints}) [Р‘РЎ]`);
                            } else if (player.type === 'is') {
                                let visitedGames = 0;
                                gameSessions.value.forEach(session => {
                                    if (session.selectedGame) {
                                        const playerRounds = session.rounds.filter(round => round.points[player.id] > 0).length;
                                        const gameCount = Math.floor(playerRounds / session.selectedGame.rounds);
                                        visitedGames += gameCount;
                                    }
                                });
                                lines.push(`${player.playerId || 'ID'} | [${catID}] (${totalPoints}; ${visitedGames}) [РРЎ]`);
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
                        name: '', 
                        playerId: '',
                        type: 'kitten' // 'kitten', 'bs', 'is', 'adult'
                    });
                }

                function removePlayer(index) {
                    if (players.value.length > 1) {
                        const removedPlayer = players.value.splice(index, 1)[0];
                        
                        // РЈРґР°Р»СЏРµРј Р±Р°Р»Р»С‹ СѓРґР°Р»РµРЅРЅРѕРіРѕ РёРіСЂРѕРєР° РёР· РІСЃРµС… РёРіСЂ
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
                        id: generateId(),
                        searchQuery: '',
                        selectedGame: null,
                        filteredGames: [],
                        showSuggestions: false,
                        rounds: []
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

                function filterGames(sessionIndex) {
                    const session = gameSessions.value[sessionIndex];
                    const query = session.searchQuery.toLowerCase();
                    
                    session.filteredGames = getAvailableGames().filter(game => 
                        game.name.toLowerCase().includes(query)
                    );
                }

                // function getAvailableGames() {
                //     const soloPlayers = players.value.length === 1;
                //     const soloAdultPlayer = soloPlayers && players.value[0].isAdult;
                //     const allAdultPlayers = players.value.length > 0 && players.value.every(p => p.isAdult);
                //     const hasNonAdultPlayers = players.value.some(p => !p.isAdult);

                //     return games.filter(game => {
                //         // Solo РёРіСЂС‹ РґРѕСЃС‚СѓРїРЅС‹ РўРћР›Р¬РљРћ РєРѕРіРґР° РёРіСЂРѕРє РѕРґРёРЅ
                //         // РќРµ-solo РёРіСЂС‹ РґРѕСЃС‚СѓРїРЅС‹ РўРћР›Р¬РљРћ РєРѕРіРґР° РёРіСЂРѕРєРѕРІ Р±РѕР»СЊС€Рµ РѕРґРЅРѕРіРѕ
                //         if ((game.solo && !soloPlayers) || (!game.solo && soloPlayers)) {
                //             return false;
                //         }

                //         // РРіСЂС‹ РўРћР›Р¬РљРћ РґР»СЏ РЅРµ-РІР·СЂРѕСЃР»С‹С… (adults: false) - СЃРєСЂС‹РІР°РµРј РµСЃР»Рё:
                //         // - РµСЃС‚СЊ РІР·СЂРѕСЃР»С‹Рµ РёРіСЂРѕРєРё
                //         if (!game.adults && !hasNonAdultPlayers) {
                //             return false;
                //         }

                //         return true;
                //     });
                // }

                function getAvailableGames() {
                    return games;
                }

                function selectGame(sessionIndex, game) {
                    const session = gameSessions.value[sessionIndex];
                    session.selectedGame = game;
                    session.searchQuery = game.name;
                    session.showSuggestions = false;
                    
                    // РЎРѕР·РґР°РµРј СЂР°СѓРЅРґС‹
                    session.rounds = [];
                    for (let i = 0; i < game.rounds; i++) {
                        session.rounds.push({
                            points: {}
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
                            points: {}
                        });
                    }
                }

                function addCustomRounds(sessionIndex) {
                    const session = gameSessions.value[sessionIndex];
                    if (session.selectedGame) {
                        const count = parseInt(prompt('РЎРєРѕР»СЊРєРѕ СЂР°СѓРЅРґРѕРІ РґРѕР±Р°РІРёС‚СЊ?', '1'));
                        if (count > 0) {
                            for (let i = 0; i < count; i++) {
                                session.rounds.push({
                                    points: {}
                                });
                            }
                        }
                    }
                }

                function filteredPlayers(gameSession) {
                    if (!gameSession.selectedGame) return players.value;
                    
                    if (!gameSession.selectedGame.adults) {
                        return players.value.filter(player => !player.isAdult);
                    }
                    
                    return players.value;
                }

                // Р—Р°РєСЂС‹РІР°РµРј РїРѕРґСЃРєР°Р·РєРё РїСЂРё РєР»РёРєРµ РІРЅРµ РїРѕР»СЏ РїРѕРёСЃРєР°
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
