<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mines Gambling Game</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; position: relative; min-height: 100vh; }
        .grid { display: grid; grid-template-columns: repeat(5, 60px); gap: 5px; justify-content: center; margin-top: 20px; }
        .cell { 
            width: 60px; height: 60px; background: #ccc; 
            display: flex; align-items: center; justify-content: center; 
            font-size: 24px; cursor: pointer; border: 2px solid #444;
            transition: transform 0.3s ease, background 0.3s ease; 
        }
        .cell:hover { background: #bbb; }
        .cell.revealed { background: lightgreen; transform: rotateY(180deg); cursor: default; }
        .cell.mine { background: red; transform: rotateY(180deg); }
        
        .end-screen { 
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.8); color: white; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
            z-index: 1000; transition: opacity 0.5s ease-in-out; 
        }
        .end-screen.hidden { display: none; opacity: 0; }
        .end-screen h2 { font-size: 32px; font-weight: bold; text-transform: uppercase; animation: fadeIn 1s ease; }

        #resetContainer {
            display: none;
            margin-top: 20px;
        }

        #resetButton { 
            padding: 12px 24px; font-size: 18px; 
            cursor: pointer; background: #ffcc00; border: none; 
            border-radius: 5px; font-weight: bold; transition: 0.3s ease;
        }
        #resetButton:hover { background: #ffaa00; }

        @keyframes fadeIn {
            0% { opacity: 0; transform: translateY(30px); }
            100% { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>
    <h1>Mustard Penguin</h1>
    <p>Click tiles to reveal safe spots. Hit a mine and lose!</p>
    <div class="grid" id="grid"></div>
    <p id="status">Balance: $100</p>

    <div id="resetContainer">
        <button id="resetButton">Reset Game</button>
    </div>

    <div id="endScreen" class="end-screen hidden">
        <h2 id="endMessage"></h2>
    </div>

    <script>
        const gridSize = 5;
        let balance = 100;
        let mineIndex;
        let revealedCells = 0;
        let gameEnded = false;

        function generateMine() {
            mineIndex = Math.floor(Math.random() * (gridSize * gridSize));
        }

        function createGrid() {
            const grid = document.getElementById("grid");
            grid.innerHTML = "";
            for (let i = 0; i < gridSize * gridSize; i++) {
                const cell = document.createElement("div");
                cell.classList.add("cell");
                cell.dataset.index = i;
                cell.addEventListener("click", () => revealCell(i));
                grid.appendChild(cell);
            }
        }

        function revealCell(index) {
            if (gameEnded) return;
            
            const cell = document.querySelector(`.cell[data-index='${index}']`);
            if (!cell || cell.classList.contains("revealed")) return;

            cell.style.transform = "rotateY(180deg)"; // Flip animation

            if (index === mineIndex) {
                cell.classList.add("mine");
                cell.innerHTML = "💣";
                balance = Math.floor(balance / 2);
                document.getElementById("status").innerText = `You hit the mine! Balance: $${balance}`;
                freezeBoard();
                checkEndGame();
            } else {
                cell.classList.add("revealed");
                cell.innerHTML = "💎";
                revealedCells++;
                balance += 10; // Increased diamond value from $5 to $10
                document.getElementById("status").innerText = `Balance: $${balance}`;
                checkVictory();
            }
        }

        function freezeBoard() {
            document.querySelectorAll(".cell").forEach(cell => cell.style.pointerEvents = "none");
        }

        function checkVictory() {
            if (revealedCells === gridSize * gridSize - 1) {
                document.getElementById("endMessage").innerText = "Lucky Pick! You won!";
                document.getElementById("endScreen").classList.remove("hidden");
                document.getElementById("resetContainer").style.display = "block"; // Show reset button below board
                gameEnded = true;
                freezeBoard();
            }
        }

        function checkEndGame() {
            if (balance <= 0) {
                document.getElementById("endMessage").innerText = "Game Over! You're out of money!";
                document.getElementById("endScreen").classList.remove("hidden");
                document.getElementById("resetContainer").style.display = "block"; // Show reset button below board
                gameEnded = true;
                freezeBoard();
            } else if (balance >= 500) { // Win condition lowered from 1000 to 500
                document.getElementById("endMessage").innerText = "99% of gamblers quit before winning!";
                document.getElementById("endScreen").classList.remove("hidden");
                document.getElementById("resetContainer").style.display = "block"; // Show reset button below board
                gameEnded = true;
                freezeBoard();
            }
        }

        function resetGame() {
            revealedCells = 0;
            gameEnded = false;
            generateMine();
            createGrid();
            document.getElementById("status").innerText = `Balance: $${balance}`;
            document.getElementById("endScreen").classList.add("hidden");
            document.getElementById("resetContainer").style.display = "none"; // Hide reset button
            document.querySelectorAll(".cell").forEach(cell => cell.style.pointerEvents = "auto");
        }

        document.getElementById("resetButton").addEventListener("click", resetGame);

        generateMine();
        createGrid();
    </script>
</body>
</html>
