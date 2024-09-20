<script lang="ts">
    interface Player {
        name: string;
        timer: number; // Cumulative time in seconds
        startTimestamp?: number; // Stores the timestamp when the timer started
    }

    let players: Player[] = [];
    let playerName: string = "";
    let selectedPlayers: string[] = [];

    // State to track if the game has started
    let gameStarted = false;

    // Function to add a player to the list
    function addPlayer(): void {
        if (
            playerName.trim() !== "" &&
            !players.some((player) => player.name === playerName)
        ) {
            const newPlayer: Player = {
                name: playerName,
                timer: 0,
            };
            players = [...players, newPlayer];
            playerName = "";
        }
    }

    // Function to toggle selection of players
    function toggleSelection(playerName: string): void {
        if (!gameStarted) {
            if (selectedPlayers.includes(playerName)) {
                selectedPlayers = selectedPlayers.filter(
                    (p) => p !== playerName,
                );
            } else if (selectedPlayers.length < 5) {
                selectedPlayers = [...selectedPlayers, playerName];
            }
        }
    }

    // Function to start timers for selected players
    function startTimers(): void {
        const now = Date.now();
        players = players.map((player) => {
            if (selectedPlayers.includes(player.name)) {
                player.startTimestamp = now;
            }
            return player;
        });
    }

    // Function to stop timers for all players
    function stopTimers(): void {
        const now = Date.now();
        players = players.map((player) => {
            if (
                selectedPlayers.includes(player.name) &&
                player.startTimestamp
            ) {
                player.timer += (now - player.startTimestamp) / 1000;
                player.startTimestamp = undefined;
            }
            return player;
        });
    }

    // Function to start the game
    function startGame(): void {
        if (selectedPlayers.length === 5) {
            gameStarted = true;
            startTimers();
        }
    }

    // Function to stop the game
    function stopGame(): void {
        gameStarted = false;
        stopTimers();
    }

    // Compute the player status
    $: playerStatus = players.map((player) => ({
        ...player,
        selected: selectedPlayers.includes(player.name),
    }));

    function exportToCSV(): void {
        const headers = ["Player Name", "Timer"];
        const csvRows = players.map(
            (player) =>
                `${player.name},${Math.floor(player.timer / 60)}:${(player.timer % 60).toFixed(1).padStart(4, "0")}`,
        );
        const csvContent = [headers.join(","), ...csvRows].join("\n");

        // Create a Blob and link to download the file
        const blob = new Blob([csvContent], {
            type: "text/csv;charset=utf-8;",
        });
        const link = document.createElement("a");
        const url = URL.createObjectURL(blob);
        link.href = url;
        link.setAttribute("download", "player_timers.csv");
        link.click();
        URL.revokeObjectURL(url);
    }
</script>

<div class="container">
    <!-- Display the list of players -->
    <h5>Player List: On the court ({selectedPlayers.length}/5)</h5>
    {#if !gameStarted}
        <div>
            <input
                type="text"
                bind:value={playerName}
                placeholder="Enter player name"
                on:keydown={(e) => e.key === "Enter" && addPlayer()}
            />
            <button on:click={addPlayer}>Add</button>
        </div>
    {/if}

    <div class="player-list">
        {#each playerStatus as { name, selected, startTimestamp, timer }}
            <div class="player-item">
                <span class:selected>{name}</span>
                <span class="timer"
                    >{Math.floor(timer / 60)}:{(timer % 60)
                        .toFixed(1)
                        .padStart(4, "0")}
                </span>
                <button
                    class="button"
                    on:click={() => toggleSelection(name)}
                    disabled={(!selected && selectedPlayers.length >= 5) ||
                        gameStarted}
                >
                    {selected ? "Deselect" : "Select"}
                </button>
            </div>
        {/each}
    </div>

    <!-- Game controls: Start Game and Stop Game -->
    <div class="game-controls">
        {#if !gameStarted}
            <button
                class="button"
                on:click={startGame}
                disabled={selectedPlayers.length !== 5}
            >
                Start Game
            </button>
        {:else}
            <button class="button" on:click={stopGame}> Stop Game </button>
        {/if}
    </div>

    <div class="csv">
        {#if !gameStarted}
            <button on:click={exportToCSV}>Export to CSV</button>
        {/if}
    </div>
</div>

<style>
    .container {
        margin: 20px;
        max-width: 400px;
    }

    .player-list {
        margin-top: 10px;
    }

    .player-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 5px;
        border-bottom: 1px solid #ddd;
    }

    .button {
        padding: 5px 10px;
        cursor: pointer;
    }

    .button:disabled {
        cursor: not-allowed;
        opacity: 0.5;
    }

    .selected {
        font-weight: bold;
        color: green;
    }

    .timer {
        font-size: 0.9em;
        color: #555;
    }

    .game-controls {
        margin-top: 20px;
    }

    .csv {
        margin-top: 40px;
    }
</style>
