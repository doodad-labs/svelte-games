<script lang="ts">
    import { BROWSER } from "esm-env";
    import { onMount, onDestroy } from "svelte";

    // Upscale the game for better visibility
    const canvasSize = 300 * 3;
    const scale = 3;
    const baseSize = 300;

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;

    let loaded: boolean = $state(false);
    let gameStarted: boolean = $state(false);
    let isMobile: boolean = $state(false);
    let isFocused: boolean = $state(false);
    let score: number = $state(0);
    let gameOver: boolean = $state(false);

    // Game objects
    type Tank = {
        x: number;
        y: number;
        width: number;
        height: number;
        color: string;
        direction: number; // in radians
        speed: number;
        health: number;
        cooldown: number;
        maxCooldown: number;
    };

    type Projectile = {
        x: number;
        y: number;
        width: number;
        height: number;
        direction: number;
        speed: number;
        owner: "player" | "enemy";
    };

    type Obstacle = {
        x: number;
        y: number;
        width: number;
        height: number;
    };

    let player: Tank = {
        x: 0,
        y: 0,
        width: 20,
        height: 15,
        color: "#4CAF50",
        direction: 0,
        speed: 2,
        health: 3,
        cooldown: 0,
        maxCooldown: 30,
    };

    let enemies: Tank[] = [];
    let projectiles: Projectile[] = [];
    let obstacles: Obstacle[] = [];
    let keys: { [key: string]: boolean } = {};

    // Game settings
    const numEnemies = 3;
    const numObstacles = 15;
    const enemySpeed = 1;
    const enemyFireRate = 100;

    // Initialize game
    function initGame() {
        // Reset game state
        gameOver = false;
        score = 0;
        projectiles = [];
        ctx.resetTransform(); // Reset canvas transform

        // Create player tank in center
        player = {
            x: baseSize / 2 - 10,
            y: baseSize / 2 - 7.5,
            width: 20,
            height: 15,
            color: "#4CAF50",
            direction: 0,
            speed: 2,
            health: 3,
            cooldown: 0,
            maxCooldown: 30,
        };

        // Create enemies
        enemies = [];
        for (let i = 0; i < numEnemies; i++) {
            enemies.push(createEnemy());
        }

        // Create obstacles
        obstacles = [];
        for (let i = 0; i < numObstacles; i++) {
            obstacles.push(createObstacle());
        }

        gameStarted = true;
        requestAnimationFrame(gameLoop);
    }

    function createEnemy(): Tank {
        // Place enemies at edges of the map
        let x, y;
        if (Math.random() < 0.5) {
            x = Math.random() < 0.5 ? 0 : baseSize - 20;
            y = Math.random() * baseSize;
        } else {
            x = Math.random() * baseSize;
            y = Math.random() < 0.5 ? 0 : baseSize - 15;
        }

        return {
            x,
            y,
            width: 20,
            height: 15,
            color: "#F44336",
            direction: Math.random() * Math.PI * 2,
            speed: enemySpeed,
            health: 1,
            cooldown: Math.floor(Math.random() * enemyFireRate),
            maxCooldown: enemyFireRate,
        };
    }

    function createObstacle(): Obstacle {
        const size = 20 + Math.random() * 30;
        return {
            x: Math.random() * (baseSize - size),
            y: Math.random() * (baseSize - size),
            width: size,
            height: size,
        };
    }

    function shoot(tank: Tank, owner: "player" | "enemy") {
        if (tank.cooldown <= 0) {
            const projectileSize = 5;
            const projectileX =
                tank.x +
                tank.width / 2 -
                projectileSize / 2 +
                (Math.cos(tank.direction) * tank.width) / 2;
            const projectileY =
                tank.y +
                tank.height / 2 -
                projectileSize / 2 +
                (Math.sin(tank.direction) * tank.height) / 2;

            projectiles.push({
                x: projectileX,
                y: projectileY,
                width: projectileSize,
                height: projectileSize,
                direction: tank.direction,
                speed: 5,
                owner,
            });

            tank.cooldown = tank.maxCooldown;
        }
    }

    function checkCollision(
        x1: number,
        y1: number,
        w1: number,
        h1: number,
        x2: number,
        y2: number,
        w2: number,
        h2: number,
    ): boolean {
        return x1 < x2 + w2 && x1 + w1 > x2 && y1 < y2 + h2 && y1 + h1 > y2;
    }

    function gameLoop() {
        if (gameOver) return;

        // Clear canvas
        ctx.clearRect(0, 0, canvasSize, canvasSize);
        ctx.save();
        ctx.scale(scale, scale);

        // Draw grid background
        ctx.strokeStyle = "#f0f0f0";
        ctx.lineWidth = 0.5;
        for (let x = 0; x <= baseSize; x += 20) {
            ctx.beginPath();
            ctx.moveTo(x, 0);
            ctx.lineTo(x, baseSize);
            ctx.stroke();
        }
        for (let y = 0; y <= baseSize; y += 20) {
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(baseSize, y);
            ctx.stroke();
        }

        // Update and draw obstacles
        ctx.fillStyle = "#795548";
        obstacles.forEach((obstacle) => {
            ctx.fillRect(
                obstacle.x,
                obstacle.y,
                obstacle.width,
                obstacle.height,
            );
        });

        // Update player
        if (player.health > 0) {
            let moved = false;
            let newX = player.x;
            let newY = player.y;

            if (keys["ArrowUp"] || keys["w"]) {
                newY -= player.speed;
                player.direction = -Math.PI / 2;
                moved = true;
            }
            if (keys["ArrowDown"] || keys["s"]) {
                newY += player.speed;
                player.direction = Math.PI / 2;
                moved = true;
            }
            if (keys["ArrowLeft"] || keys["a"]) {
                newX -= player.speed;
                player.direction = Math.PI;
                moved = true;
            }
            if (keys["ArrowRight"] || keys["d"]) {
                newX += player.speed;
                player.direction = 0;
                moved = true;
            }

            // Check collision with walls
            if (
                newX >= 0 &&
                newX + player.width <= baseSize &&
                newY >= 0 &&
                newY + player.height <= baseSize
            ) {
                // Check collision with obstacles
                let canMove = true;
                for (const obstacle of obstacles) {
                    if (
                        checkCollision(
                            newX,
                            newY,
                            player.width,
                            player.height,
                            obstacle.x,
                            obstacle.y,
                            obstacle.width,
                            obstacle.height,
                        )
                    ) {
                        canMove = false;
                        break;
                    }
                }

                if (canMove) {
                    player.x = newX;
                    player.y = newY;
                }
            }

            // Shoot with space
            if (keys[" "]) {
                shoot(player, "player");
            }

            // Decrease cooldown
            if (player.cooldown > 0) {
                player.cooldown--;
            }

            // Draw player
            ctx.save();
            ctx.translate(
                player.x + player.width / 2,
                player.y + player.height / 2,
            );
            ctx.rotate(player.direction);
            ctx.fillStyle = player.color;
            ctx.fillRect(
                -player.width / 2,
                -player.height / 2,
                player.width,
                player.height,
            );

            // Draw tank barrel
            ctx.fillStyle = "#333";
            ctx.fillRect(player.width / 2 - 3, -2, 10, 4);
            ctx.restore();

            // Draw health
            for (let i = 0; i < player.health; i++) {
                ctx.fillStyle = "#4CAF50";
                ctx.fillRect(5 + i * 10, 5, 8, 8);
            }
        }

        // Update and draw enemies
        enemies.forEach((enemy, index) => {
            if (enemy.health <= 0) return;

            // Simple AI: move toward player
            if (player.health > 0) {
                const dx =
                    player.x + player.width / 2 - (enemy.x + enemy.width / 2);
                const dy =
                    player.y + player.height / 2 - (enemy.y + enemy.height / 2);
                const distance = Math.sqrt(dx * dx + dy * dy);

                enemy.direction = Math.atan2(dy, dx);

                // Move if not too close
                if (distance > 50) {
                    enemy.x += Math.cos(enemy.direction) * enemy.speed;
                    enemy.y += Math.sin(enemy.direction) * enemy.speed;
                }

                // Shoot at player
                if (distance < 200 && Math.random() < 0.02) {
                    shoot(enemy, "enemy");
                }
            }

            // Keep enemy within bounds
            enemy.x = Math.max(0, Math.min(baseSize - enemy.width, enemy.x));
            enemy.y = Math.max(0, Math.min(baseSize - enemy.height, enemy.y));

            // Check collision with obstacles
            for (const obstacle of obstacles) {
                if (
                    checkCollision(
                        enemy.x,
                        enemy.y,
                        enemy.width,
                        enemy.height,
                        obstacle.x,
                        obstacle.y,
                        obstacle.width,
                        obstacle.height,
                    )
                ) {
                    // Move away from obstacle
                    enemy.x -= Math.cos(enemy.direction) * enemy.speed;
                    enemy.y -= Math.sin(enemy.direction) * enemy.speed;
                    enemy.direction = Math.random() * Math.PI * 2;
                    break;
                }
            }

            // Decrease cooldown
            if (enemy.cooldown > 0) {
                enemy.cooldown--;
            }

            // Draw enemy
            ctx.save();
            ctx.translate(
                enemy.x + enemy.width / 2,
                enemy.y + enemy.height / 2,
            );
            ctx.rotate(enemy.direction);
            ctx.fillStyle = enemy.color;
            ctx.fillRect(
                -enemy.width / 2,
                -enemy.height / 2,
                enemy.width,
                enemy.height,
            );

            // Draw tank barrel
            ctx.fillStyle = "#333";
            ctx.fillRect(enemy.width / 2 - 3, -2, 10, 4);
            ctx.restore();
        });

        // Update and draw projectiles
        for (let i = projectiles.length - 1; i >= 0; i--) {
            const projectile = projectiles[i];

            // Move projectile
            projectile.x += Math.cos(projectile.direction) * projectile.speed;
            projectile.y += Math.sin(projectile.direction) * projectile.speed;

            // Check if out of bounds
            if (
                projectile.x < 0 ||
                projectile.x > baseSize ||
                projectile.y < 0 ||
                projectile.y > baseSize
            ) {
                projectiles.splice(i, 1);
                continue;
            }

            // Check collision with obstacles
            let hitObstacle = false;
            for (const obstacle of obstacles) {
                if (
                    checkCollision(
                        projectile.x,
                        projectile.y,
                        projectile.width,
                        projectile.height,
                        obstacle.x,
                        obstacle.y,
                        obstacle.width,
                        obstacle.height,
                    )
                ) {
                    hitObstacle = true;
                    break;
                }
            }

            if (hitObstacle) {
                projectiles.splice(i, 1);
                continue;
            }

            // Check collision with player
            if (
                projectile.owner === "enemy" &&
                player.health > 0 &&
                checkCollision(
                    projectile.x,
                    projectile.y,
                    projectile.width,
                    projectile.height,
                    player.x,
                    player.y,
                    player.width,
                    player.height,
                )
            ) {
                player.health--;
                projectiles.splice(i, 1);
                if (player.health <= 0) {
                    gameOver = true;
                }
                continue;
            }

            // Check collision with enemies
            if (projectile.owner === "player") {
                for (let j = enemies.length - 1; j >= 0; j--) {
                    const enemy = enemies[j];
                    if (
                        enemy.health > 0 &&
                        checkCollision(
                            projectile.x,
                            projectile.y,
                            projectile.width,
                            projectile.height,
                            enemy.x,
                            enemy.y,
                            enemy.width,
                            enemy.height,
                        )
                    ) {
                        enemy.health--;
                        projectiles.splice(i, 1);
                        if (enemy.health <= 0) {
                            score += 100;
                            enemies.splice(j, 1);
                            // Spawn new enemy
                            if (
                                enemies.length < numEnemies &&
                                Math.random() < 0.5
                            ) {
                                enemies.push(createEnemy());
                            }
                        }
                        break;
                    }
                }
            }

            // Draw projectile if still exists
            if (projectiles[i]) {
                ctx.fillStyle =
                    projectile.owner === "player" ? "#FFC107" : "#F44336";
                ctx.fillRect(
                    projectile.x,
                    projectile.y,
                    projectile.width,
                    projectile.height,
                );
            }
        }

        // Draw score
        ctx.fillStyle = "#333";
        ctx.font = "12px Arial";
        ctx.textAlign = "right";
        ctx.fillText(`Score: ${score}`, baseSize - 10, 15);

        // Check game over conditions
        if (player.health <= 0) {
            ctx.fillStyle = "rgba(0, 0, 0, 0.7)";
            ctx.fillRect(0, 0, baseSize, baseSize);
            ctx.fillStyle = "#fff";
            ctx.font = "20px Arial";
            ctx.textAlign = "center";
            ctx.fillText("Game Over", baseSize / 2, baseSize / 2 - 20);
            ctx.font = "16px Arial";
            ctx.fillText(
                `Final Score: ${score}`,
                baseSize / 2,
                baseSize / 2 + 10,
            );
            ctx.font = "14px Arial";
            ctx.fillText("Click to restart", baseSize / 2, baseSize / 2 + 40);
            return;
        }

        ctx.restore();
        requestAnimationFrame(gameLoop);
    }

    function handleClick() {
        if (!gameStarted || gameOver) {
            initGame();
        }
    }

    // Handle keyboard input
    function handleKeyDown(e: KeyboardEvent) {
        if (!isFocused) return;
        keys[e.key.toLowerCase()] = true;

        // Prevent spacebar from scrolling
        if (e.key === " ") {
            e.preventDefault();
        }

        // Start game on first key press
        if (!gameStarted) {
            initGame();
        }
    }

    function handleKeyUp(e: KeyboardEvent) {
        keys[e.key.toLowerCase()] = false;
    }

    onMount(() => {
        if (!BROWSER) return;

        // Initialize canvas
        ctx = canvas.getContext("2d")!;
        canvas.width = canvasSize;
        canvas.height = canvasSize;
        loaded = true;

        // Check if mobile
        isMobile =
            /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
                navigator.userAgent,
            );

        // Add event listeners
        window.addEventListener("keyup", handleKeyUp);
        canvas.addEventListener("touchstart", handleClick);
    });

    onDestroy(() => {
        if (!BROWSER) return;
        if (canvas) {
            canvas.removeEventListener("touchstart", handleClick);
        }
        window.removeEventListener("keyup", handleKeyUp);
    });
</script>

<svelte:window onkeydown={handleKeyDown} onkeyup={handleKeyUp} />

<div class="tanks">
    <div class="container">
        {#if !gameStarted && !gameOver}
            <div class="start-screen">
                <h2>Tanks Game</h2>
                <p>Use WASD or arrow keys to move</p>
                <p>Press SPACE to shoot</p>
                <button onclick={initGame}>Start Game</button>
            </div>
        {/if}

        <canvas
            class="{isFocused ? 'focused' : ''} {loaded ? '' : 'hidden'}"
            onmouseleave={() => (isFocused = false)}
            onmouseenter={() => (isFocused = true)}
            onclick={handleClick}
            bind:this={canvas}
            width={canvasSize}
            height={canvasSize}
        ></canvas>

        <svg
            class={loaded ? "hidden" : ""}
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
        >
            <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
            ></circle>
            <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            ></path>
        </svg>
    </div>
</div>

<style>
    .tanks {
        min-width: 300px;
    }

    .tanks .container {
        position: relative;
        min-width: 300px;
        min-height: 300px;
        width: 100%;
        aspect-ratio: 1;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .start-screen {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        background: rgba(255, 255, 255, 0.9);
        z-index: 10;
        text-align: center;
        padding: 20px;
        box-sizing: border-box;
    }

    .start-screen h2 {
        margin-bottom: 10px;
        color: #333;
    }

    .start-screen p {
        margin: 5px 0;
        color: #666;
    }

    .start-screen button {
        margin-top: 20px;
        padding: 10px 20px;
        background: #4caf50;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 16px;
    }

    .start-screen button:hover {
        background: #45a049;
    }

    .tanks svg {
        width: 1.25rem;
        height: 1.25rem;
        color: black;
        animation: spin 1s linear infinite;
    }

    @keyframes spin {
        to {
            transform: rotate(360deg);
        }
    }

    .tanks canvas {
        min-width: 300px;
        min-height: 300px;
        width: 100%;
        aspect-ratio: 1;
        background: white;
        border-radius: 0.25rem;
        border: 1px solid rgba(0, 0, 0, 0.05);
        touch-action: none;
    }

    .tanks canvas.focused {
        border-color: #2196f3;
    }

    .tanks .hidden {
        display: none;
    }

    canvas {
        border: 1px solid rgba(0, 0, 0, 0.05);
        touch-action: none;
    }
</style>
