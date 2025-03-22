# endles-
<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Eindeloze Runner</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { text-align: center; font-family: Arial, sans-serif; background: #70c5ce; }
        canvas { background: #fff; display: block; margin: 20px auto; border: 2px solid black; }
        #gameOverScreen {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px;
            text-align: center;
            border-radius: 10px;
        }
        #restartBtn {
            background: red;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            margin-top: 10px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>Eindeloze Runner</h1>
    <p>Druk op SPATIE om te springen</p>
    <canvas id="gameCanvas" width="800" height="400"></canvas>
    
    <div id="gameOverScreen">
        <h2>Game Over!</h2>
        <p>Je score: <span id="finalScore"></span></p>
        <button id="restartBtn">Opnieuw spelen</button>
    </div>

    <!-- YouTube Video die onzichtbaar is, autoplay wordt ingeschakeld -->
    <iframe width="1" height="1" src="https://www.youtube.com/embed/a0lsTHE6BBU?autoplay=1&loop=1&playlist=a0lsTHE6BBU" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        // Speler (mannetje)
        const player = {
            x: 50,
            y: 300,
            width: 30,
            height: 50,
            velocityY: 0,
            jumping: false,
            jumpHeight: 20 // Houdt de springhoogte bij
        };

        let obstacles = [];
        let obstacleSpeed = 4;
        let frame = 0;
        let score = 0;
        let gameOver = false;
        const gravity = 0.5;
        const jumpPower = -10;

        // Functie om de achtergrond (straat) te tekenen
        function drawStreet() {
            // Lucht (blauw)
            ctx.fillStyle = "#87CEEB";
            ctx.fillRect(0, 0, canvas.width, 200); // Bovenste deel voor de lucht

            // Grond (grijs voor de straat)
            ctx.fillStyle = "#A9A9A9";
            ctx.fillRect(0, 200, canvas.width, 150); // Onderste deel voor de weg

            // Witte strepen (straatmarkering)
            ctx.strokeStyle = "white";
            ctx.lineWidth = 5;
            ctx.setLineDash([20, 20]); // Streepeffect
            ctx.beginPath();
            ctx.moveTo(0, 275);
            ctx.lineTo(canvas.width, 275);
            ctx.stroke();
            ctx.setLineDash([]); // Verwijder het streepeffect
        }

        // Functie om het mannetje te tekenen (met schaduw)
        function drawPlayer(x, y) {
            // Schaduw
            ctx.fillStyle = "rgba(0, 0, 0, 0.2)";
            ctx.beginPath();
            ctx.arc(x + 15, y + 20, 15, 0, Math.PI * 2); // Schaduw onder het mannetje
            ctx.fill();

            // Hoofd
            ctx.fillStyle = "peachpuff";
            ctx.beginPath();
            ctx.arc(x + 15, y + 10, 10, 0, Math.PI * 2);
            ctx.fill();

            // Lichaam
            ctx.fillStyle = "blue";
            ctx.fillRect(x + 5, y + 20, 20, 25);

            // Benen
            ctx.fillStyle = "black";
            ctx.fillRect(x + 5, y + 45, 8, 15);
            ctx.fillRect(x + 17, y + 45, 8, 15);

            // Armen
            ctx.fillStyle = "peachpuff";
            ctx.fillRect(x - 5, y + 20, 10, 10);
            ctx.fillRect(x + 25, y + 20, 10, 10);

            // Wolkje tekenen als mannetje springt
            if (player.jumping) {
                ctx.fillStyle = "white";
                ctx.beginPath();
                ctx.arc(x + 15, y + 50, 20, Math.PI, 0, true); // Wolk onder het mannetje
                ctx.fill();
            }
        }

        // Functie om obstakels te genereren (hond, computer, vuilnisbak)
        function generateObstacle() {
            const obstacleType = Math.random() < 0.33 ? 'vuilnisbak' : (Math.random() < 0.5 ? 'computer' : 'hond');
            let obstacleY = 320;
            if (obstacleType === 'hond') {
                obstacleY = 310; // Hond op de grond
            }
            obstacles.push({ x: canvas.width, y: obstacleY, width: 40, height: 30, type: obstacleType });
        }

        // Functie om obstakels te tekenen
        function drawObstacle(obstacle) {
            if (obstacle.type === 'vuilnisbak') {
                // Teken Vuilnisbak
                ctx.fillStyle = "green";
                ctx.fillRect(obstacle.x, obstacle.y, 40, 60);
                ctx.fillStyle = "gray";
                ctx.fillRect(obstacle.x + 10, obstacle.y + 20, 20, 40); // Lichter groen voor de deksel
            } else if (obstacle.type === 'computer') {
                // Teken Computer
                ctx.fillStyle = "lightgray";
                ctx.fillRect(obstacle.x, obstacle.y, 50, 50); // Hoofd van de computer
                ctx.fillStyle = "black";
                ctx.fillRect(obstacle.x + 5, obstacle.y + 5, 40, 30); // Scherm
                ctx.fillStyle = "gray";
                ctx.fillRect(obstacle.x + 10, obstacle.y + 40, 30, 10); // Basis
            } else if (obstacle.type === 'hond') {
                // Teken Hond op de grond
                ctx.fillStyle = "brown";
                ctx.fillRect(obstacle.x, obstacle.y, 40, 20); // Lichaam van hond
                ctx.fillStyle = "black";
                ctx.fillRect(obstacle.x + 10, obstacle.y - 10, 10, 10); // Hoofd van hond
                ctx.fillRect(obstacle.x + 10, obstacle.y + 15, 8, 10); // Pootje 1
                ctx.fillRect(obstacle.x + 25, obstacle.y + 15, 8, 10); // Pootje 2
            }
        }

        function updateGame() {
            if (gameOver) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Teken de straat achtergrond
            drawStreet();

            // Speler update
            player.velocityY += gravity;
            player.y += player.velocityY;

            // Zorg ervoor dat het mannetje niet door de grond zakt
            if (player.y >= 300) {
                player.y = 300;
                player.jumping = false;
            }

            // Obstakels genereren
            if (frame % 100 === 0) {
                generateObstacle();
            }

            // Obstakels updaten en tekenen
            for (let i = 0; i < obstacles.length; i++) {
                obstacles[i].x -= obstacleSpeed;

                // Speler raakt obstakel (collision detection)
                if (
                    player.x < obstacles[i].x + obstacles[i].width &&
                    player.x + player.width > obstacles[i].x &&
                    player.y < obstacles[i].y + obstacles[i].height &&
                    player.y + player.height > obstacles[i].y
                ) {
                    endGame();
                    return;
                }

                // Teken obstakel (vuilnisbak, computer, hond)
                drawObstacle(obstacles[i]);
            }

            // Obstakels opruimen
            obstacles = obstacles.filter(obstacle => obstacle.x > -obstacle.width);

            // Score verhogen
            score++;

            // Grond tekenen (onderkant van de weg)
            ctx.fillStyle = "green";
            ctx.fillRect(0, 350, canvas.width, 50);

            // Teken speler (mannetje)
            drawPlayer(player.x, player.y);

            // Score tonen
            ctx.fillStyle = "black";
            ctx.font = "20px Arial";
            ctx.fillText("Score: " + score, 10, 30);

            frame++;
            requestAnimationFrame(updateGame);
        }

        // Game Over functie
        function endGame() {
            gameOver = true;
            document.getElementById("finalScore").innerText = score;
            document.getElementById("gameOverScreen").style.display = "block";
        }

        // Springfunctie
        function jump() {
            if (!player.jumping) {
                player.velocityY = jumpPower;
                player.jumping = true;
            }
        }

        // Herstart de game
        function restartGame() {
            obstacles = [];
            score = 0;
            frame = 0;
            gameOver = false;
            player.y = 300;
            player.velocityY = 0;
            document.getElementById("gameOverScreen").style.display = "none";
            updateGame();
        }

        // Event Listeners
        document.addEventListener("keydown", function(event) {
            if (event.code === "Space" && !gameOver) {
                jump();
            }
        });

        document.getElementById("restartBtn").addEventListener("click", restartGame);

        updateGame();
    </script>
</body>
</html>
