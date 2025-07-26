# Isaac-e-a-Princesa-Sofia-O-Resgate-do-Reino
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Isaac e a Princesa Sofia: A Sombra do Reino</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body, html {
      margin: 0; padding: 0; background: #000; overflow: hidden;
      font-family: 'Arial', sans-serif;
      color: white;
      user-select: none;
    }
    #gameCanvas {
      display: block;
      margin: auto;
      background: #444;
    }
    #dialogueBox {
      position: fixed;
      bottom: 10px;
      left: 10px;
      right: 10px;
      background: rgba(0,0,0,0.8);
      padding: 15px;
      border-radius: 10px;
      font-size: 18px;
      display: none;
      user-select: text;
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="800" height="600"></canvas>
  <div id="dialogueBox"></div>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // Game variables
    let keys = {};
    let player = {
      x: 400,
      y: 300,
      size: 32,
      speed: 3,
      color: '#6b4423', // castanho (cabelo e olhos)
      hp: 5,
      attacking: false,
      attackTimer: 0,
    };

    let enemy = {
      x: 600,
      y: 300,
      size: 32,
      speed: 1.5,
      color: 'green',
      hp: 3,
      direction: 1,
      moveRange: 100,
      startX: 600,
    };

    let gameState = 'playing'; // 'playing', 'dialogue', 'end'

    // Dialogue data
    const dialogues = [
      "Isaac… você enfrentou tudo isso… só pra me salvar?",
      "Claro, Sofia. Eu jamais deixaria você nas mãos daquele monstro.",
      "Você é meu herói… e quer saber? Eu… eu te amo, Isaac.",
      "💖 FIM 💖",
    ];
    let dialogueIndex = 0;
    const dialogueBox = document.getElementById('dialogueBox');

    // Listen to keys
    window.addEventListener('keydown', e => {
      keys[e.key.toLowerCase()] = true;
      if (gameState === 'dialogue' && e.key === 'Enter') {
        nextDialogue();
      }
    });
    window.addEventListener('keyup', e => {
      keys[e.key.toLowerCase()] = false;
    });

    function rectsColliding(r1, r2) {
      return !(r2.x > r1.x + r1.size ||
               r2.x + r2.size < r1.x ||
               r2.y > r1.y + r1.size ||
               r2.y + r2.size < r1.y);
    }

    function drawPlayer() {
      ctx.fillStyle = player.color;
      ctx.fillRect(player.x, player.y, player.size, player.size);
      if (player.attacking) {
        ctx.fillStyle = '#c0c0c0';
        ctx.fillRect(player.x + player.size, player.y + player.size / 4, 10, player.size / 2);
      }
    }

    function drawEnemy() {
      ctx.fillStyle = enemy.color;
      ctx.fillRect(enemy.x, enemy.y, enemy.size, enemy.size);
    }

    function update() {
      if (gameState === 'playing') {
        // Move player
        if (keys['arrowleft'] || keys['a']) player.x -= player.speed;
        if (keys['arrowright'] || keys['d']) player.x += player.speed;
        if (keys['arrowup'] || keys['w']) player.y -= player.speed;
        if (keys['arrowdown'] || keys['s']) player.y += player.speed;

        // Keep player inside canvas
        player.x = Math.max(0, Math.min(canvas.width - player.size, player.x));
        player.y = Math.max(0, Math.min(canvas.height - player.size, player.y));

        // Attack logic
        if (keys[' '] && !player.attacking) {
          player.attacking = true;
          player.attackTimer = 20;
        }
        if (player.attacking) {
          player.attackTimer--;
          if (player.attackTimer <= 0) {
            player.attacking = false;
          }
        }

        // Enemy moves back and forth
        enemy.x += enemy.speed * enemy.direction;
        if (enemy.x > enemy.startX + enemy.moveRange || enemy.x < enemy.startX - enemy.moveRange) {
          enemy.direction *= -1;
        }

        // Enemy hits player
        if (rectsColliding(player, enemy)) {
          player.hp -= 0.02; // perde vida lentamente se tocar
          if (player.hp <= 0) {
            gameState = 'end';
            showDialogue("Você foi derrotado... Tente novamente!");
          }
        }

        // Player attack hits enemy
        if (player.attacking) {
          const attackHitbox = {
            x: player.x + player.size,
            y: player.y + player.size / 4,
            size: 10,
          };
          if (rectsColliding(attackHitbox, enemy)) {
            enemy.hp -= 0.1;
            if (enemy.hp <= 0) {
              gameState = 'dialogue';
              showDialogue(dialogues[dialogueIndex]);
            }
          }
        }
      }
      draw();
      if (gameState !== 'end') {
        requestAnimationFrame(update);
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      // background
      ctx.fillStyle = '#003300';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // draw player and enemy
      drawPlayer();
      if (enemy.hp > 0) {
        drawEnemy();
      }

      // draw HP bar
      ctx.fillStyle = 'red';
      ctx.fillRect(20, 20, player.hp * 40, 10);
      ctx.strokeStyle = 'white';
      ctx.strokeRect(20, 20, 200, 10);
    }

    function showDialogue(text) {
      dialogueBox.style.display = 'block';
      dialogueBox.textContent = text;
    }

    function nextDialogue() {
      dialogueIndex++;
      if (dialogueIndex < dialogues.length) {
        showDialogue(dialogues[dialogueIndex]);
      } else {
        dialogueBox.textContent = "Obrigado por jogar, Sofia! - Isaac 💗";
      }
    }

    // Start game loop
    update();
  </script>
</body>
</html>
