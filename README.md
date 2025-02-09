<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mines Gambling Game</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; }
        .grid { display: grid; grid-template-columns: repeat(5, 60px); gap: 5px; justify-content: center; margin-top: 20px; }
        .cell { width: 60px; height: 60px; background: #ccc; display: flex; align-items: center; justify-content: center; font-size: 24px; cursor: pointer; border: 2px solid #444; }
        .cell.revealed { background: lightgreen; cursor: default; }
        .cell.mine { background: red; }
        .end-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.8); color: white; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 999; }
        .end-screen.hidden { display: none; }
        #resetButton { margin-top: 20px; padding: 10px 20px; font-size: 16px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Mines Gambling Game</h1>
    <p>Click tiles to reveal safe spots. Hit a mine and lose!</p>
    <div class="grid" id="grid"></div>
    <p id="status">Balance: $100</p>
    <div id="endScreen" class="end-screen hidden">
        <h2 id="endMessage"></h2>
        <button id="resetButton" class="hidden">Reset Game</button>
    </div>
    <script>
        const gridSize = 5;
        let balance = 100;
        let mineIndex;
        let revealedCells = 0;
        let gameEnded = false;

        // Generate a mine in the grid
        function generateMine() {
            mineIndex = Math.floor(Math.random() * (gridSize * gridSize));
        }

        // Create the grid of cells
        function createGrid() {
            const grid = document.getElementById("grid");
            grid.innerHTML = ""; // Clear any previous grid
            for (let i = 0; i < gridSize * gridSize; i++) {
                const cell = document.createElement("div");
                cell.classList.add("cell");
                cell.dataset.index = i;
                cell.addEventListener("click", () => revealCell(i));
                grid.appendChild(cell);
            }
        }

        // Reveal a cell when clicked
        function revealCell(index) {
            if (gameEnded) return; // Prevent further actions if the game has ended
            
            const cell = document.querySelector(`.cell[data-index='${index}']`);
            if (!cell || cell.classList.contains("revealed")) return;

            if (index === mineIndex) {
                // Player hits a mine
                cell.classList.add("mine");
                cell.innerHTML = "💣";
                balance = Math.floor(balance / 2); // Half the balance if hit a mine
                document.getElementById("status").innerText = `You hit the mine! Balance: $${balance}`;
                freezeBoard();
                checkEndGame();
            } else {
                // Safe spot
                cell.classList.add("revealed");
                cell.innerHTML = "💎";
                revealedCells++;
                balance += 5; // Increase balance for safe picks
                document.getElementById("status").innerText = `Balance: $${balance}`;
                checkVictory();
            }
        }

        // Freeze the board (disable further clicks)
        function freezeBoard() {
            const cells = document.querySelectorAll(".cell");
            cells.forEach(cell => cell.style.pointerEvents = "none");
            document.getElementById("resetButton").classList.remove("hidden");
        }

        // Check if the player won by revealing all safe cells
        function checkVictory() {
            if (revealedCells === gridSize * gridSize - 1) {
                document.getElementById("endMessage").innerText = "Lucky Pick!";
                document.getElementById("endScreen").classList.remove("hidden");
                gameEnded = true; // End the game
            }
        }

        // Check if the game should end due to win/loss conditions
        function checkEndGame() {
            if (balance <= 0) {
                document.getElementById("endMessage").innerText = "Game Over! You're out of money!";
                document.getElementById("endScreen").classList.remove("hidden");
                gameEnded = true; // End the game
            } else if (balance >= 1000) {
                document.getElementById("endMessage").innerText = "99% of gamblers quit before winning!";
                document.getElementById("endScreen").classList.remove("hidden");
                gameEnded = true; // End the game
            }
        }

        // Reset the game
        function resetGame() {
            balance = 100;
            revealedCells = 0;
            gameEnded = false;
            generateMine(); // Generate the mine at the start
            createGrid(); // Create the grid of cells
            document.getElementById("status").innerText = `Balance: $${balance}`;
            document.getElementById("endScreen").classList.add("hidden");
            document.getElementById("resetButton").classList.add("hidden");

            // Re-enable the grid cells for clicks
            const cells = document.querySelectorAll(".cell");
            cells.forEach(cell => cell.style.pointerEvents = "auto");
        }

        // Add reset game button functionality
        document.getElementById("resetButton").addEventListener("click", resetGame);

        // Initialize the game
        generateMine(); // Generate the mine at the start
        createGrid(); // Create the grid of cells
    </script>
</body>
</html>
