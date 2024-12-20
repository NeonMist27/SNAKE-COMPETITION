---
layout: base
title: Snake
permalink: /snake/
---

<style>
    body {
        margin: 0;
        padding: 0;
    }

    .wrap {
        margin-left: auto;
        margin-right: auto;
    }

    canvas {
        display: none;
        border-style: solid;
        border-width: 10px;
        background-image: url('{{ site.baseurl }}/images/github.png');
        background-size: cover;
    }

    canvas:focus {
        outline: none;
    }

    #gameover p, #setting p, #menu p {
        font-size: 20px;
    }

    #gameover a, #setting a, #menu a {
        font-size: 30px;
        display: block;
    }

    #gameover a:hover, #setting a:hover, #menu a:hover {
        cursor: pointer;
    }

    #menu {
        display: block;
    }

    #gameover {
        display: none;
    }

    #setting {
        display: none;
    }

    #setting input {
        display: none;
    }

    #setting label {
        cursor: pointer;
    }

    #setting input:checked + label {
        background-color: #FFF;
        color: #000;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <p class="fs-4">Score: <span id="score_value">0</span></p>
    </header>
    <div class="container bg-secondary" style="text-align:center;">
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
            <p>Press <b>F</b> to toggle Fullscreen</p>
        </div>
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="150" checked/>
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="100"/>
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="50"/>
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked/>
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0"/>
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
    (function () {
        const canvas = document.getElementById("snake");
        const ctx = canvas.getContext("2d");
        const BLOCK = 10;

        let snake, snake_dir, snake_next_dir, snake_speed, food, score, wall;
        let snakeColorIndex = 0, foodColorIndex = 0, borderColorIndex = 0;

        const COLORS = ["red", "blue", "green", "yellow", "purple", "orange", "pink", "cyan", "lime"];
        const BORDER_COLORS = ["#FF5733", "#33FFBD", "#FF33F6", "#339FFF", "#FFD700"];

        window.onload = function () {
            document.getElementById("new_game").onclick = newGame;
            document.getElementById("new_game1").onclick = newGame;
            document.getElementById("new_game2").onclick = newGame;

            window.addEventListener("keydown", function (evt) {
                if (evt.code === "Space") newGame();
                if (evt.code === "KeyF") toggleFullScreen();

                // Arrow key controls for snake movement
                if (evt.code === "ArrowUp" && snake_dir !== 2) { snake_next_dir = 0; snake_dir = 0; }    // Up
                if (evt.code === "ArrowRight" && snake_dir !== 3) { snake_next_dir = 1; snake_dir = 1; } // Right
                if (evt.code === "ArrowDown" && snake_dir !== 0) { snake_next_dir = 2; snake_dir = 2; }  // Down
                if (evt.code === "ArrowLeft" && snake_dir !== 1) { snake_next_dir = 3; snake_dir = 3; }  // Left
            });

            setSnakeSpeed(200); // Slower snake speed
            setWall(1);
            setBorderColor();
        };

        function newGame() {
            snake = [{ x: 15, y: 15 }];
            snake_dir = 1;
            snake_next_dir = 1;
            food = {};
            score = 0;

            snakeColorIndex = 0;
            foodColorIndex = 1;
            setScore(score);
            placeFood();
            canvas.style.display = "block";
            mainLoop();
        }

        function mainLoop() {
            const head = { ...snake[0] };
            switch (snake_next_dir) {
                case 0: head.y--; break;
                case 1: head.x++; break;
                case 2: head.y++; break;
                case 3: head.x--; break;
            }

            snake.unshift(head);
            if (head.x === food.x && head.y === food.y) {
                score++;
                snakeColorIndex = (snakeColorIndex + 1) % COLORS.length;
                foodColorIndex = (foodColorIndex + 1) % COLORS.length;
                setScore(score);
                placeFood();
                setBorderColor();
            } else {
                snake.pop();
            }

            checkCollision(head);
            renderGame();
            setTimeout(mainLoop, snake_speed);
        }

        function renderGame() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            snake.forEach(segment => drawBlock(segment.x, segment.y, COLORS[snakeColorIndex]));
            drawBlock(food.x, food.y, COLORS[foodColorIndex]);
        }

        function drawBlock(x, y, color) {
            ctx.fillStyle = color;
            ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
        }

        function placeFood() {
            food.x = Math.floor(Math.random() * canvas.width / BLOCK);
            food.y = Math.floor(Math.random() * canvas.height / BLOCK);
        }

        function checkCollision(head) {
            if (wall && (head.x < 0 || head.y < 0 || head.x >= canvas.width / BLOCK || head.y >= canvas.height / BLOCK)) {
                alert("Game Over!");
                newGame();
            }
            if (snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y)) {
                alert("Game Over!");
                newGame();
            }
        }

        function setScore(newScore) {
            document.getElementById("score_value").innerText = newScore;
        }

        function setSnakeSpeed(value) {
            snake_speed = parseInt(value);
        }

        function setWall(value) {
            wall = parseInt(value);
        }

        function setBorderColor() {
            canvas.style.borderColor = BORDER_COLORS[borderColorIndex % BORDER_COLORS.length];
            borderColorIndex++;
        }

        function toggleFullScreen() {
            if (!document.fullscreenElement) {
                canvas.requestFullscreen();
            } else {
                document.exitFullscreen();
            }
        }
    })();
</script>
