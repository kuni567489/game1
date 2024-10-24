# game1
game
<html><head><base href="https://asteroidspc.vercel.app/"><meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>Asteroids PC Version</title>
<style>
  body, html {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background-color: #000;
    cursor: none;
  }
  #gameCanvas {
    display: block;
    width: 100%;
    height: 100%;
  }
  #score {
    position: absolute;
    top: 20px;
    left: 20px;
    color: white;
    font-family: Arial, sans-serif;
    font-size: 24px;
  }
</style>
</head>
<body>
<canvas id="gameCanvas"></canvas>
<div id="score">Score: 0</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreElement = document.getElementById('score');

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const ship = {
  x: canvas.width / 2,
  y: canvas.height / 2,
  radius: 20,
  angle: 0,
  thrust: {
    x: 0,
    y: 0
  },
  color: '#00FFFF'
};

const bullets = [];
const asteroids = [];
const particles = [];
const stars = [];
let score = 0;
let mouseX = 0;
let mouseY = 0;
let isFiring = false;

function createAsteroid() {
  const asteroid = {
    x: Math.random() < 0.5 ? 0 : canvas.width,
    y: Math.random() * canvas.height,
    radius: Math.random() * 30 + 20,
    speed: Math.random() * 2 + 1,
    angle: Math.random() * Math.PI * 2,
    vertices: Math.floor(Math.random() * 5) + 5,
    color: `hsl(${Math.random() * 360}, 50%, 50%)`
  };
  asteroids.push(asteroid);
}

for (let i = 0; i < 5; i++) {
  createAsteroid();
}

function createStars() {
  for (let i = 0; i < 100; i++) {
    stars.push({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      radius: Math.random() * 1.5 + 0.5,
      brightness: Math.random(),
      twinkleSpeed: Math.random() * 0.05 + 0.01
    });
  }
}

createStars();

function drawShip() {
  ctx.save();
  ctx.translate(ship.x, ship.y);
  ctx.rotate(ship.angle);
  ctx.strokeStyle = ship.color;
  ctx.fillStyle = 'rgba(0, 255, 255, 0.2)';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(ship.radius, 0);
  ctx.lineTo(-ship.radius, -ship.radius / 2);
  ctx.lineTo(-ship.radius, ship.radius / 2);
  ctx.closePath();
  ctx.stroke();
  ctx.fill();

  ctx.beginPath();
  ctx.moveTo(-ship.radius, 0);
  ctx.lineTo(-ship.radius - 10, -5);
  ctx.lineTo(-ship.radius - 10, 5);
  ctx.closePath();
  ctx.fillStyle = 'orange';
  ctx.fill();

  ctx.restore();
}

function drawBullets() {
  bullets.forEach(bullet => {
    ctx.beginPath();
    ctx.arc(bullet.x, bullet.y, 3, 0, Math.PI * 2);
    ctx.fillStyle = 'yellow';
    ctx.fill();

    ctx.beginPath();
    ctx.arc(bullet.x, bullet.y, 5, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(255, 255, 0, 0.3)';
    ctx.fill();
  });
}

function drawAsteroids() {
  asteroids.forEach(asteroid => {
    ctx.beginPath();
    ctx.moveTo(
      asteroid.x + asteroid.radius * Math.cos(0),
      asteroid.y + asteroid.radius * Math.sin(0)
    );

    for (let i = 1; i < asteroid.vertices; i++) {
      ctx.lineTo(
        asteroid.x + asteroid.radius * Math.cos(i * Math.PI * 2 / asteroid.vertices),
        asteroid.y + asteroid.radius * Math.sin(i * Math.PI * 2 / asteroid.vertices)
      );
    }

    ctx.closePath();
    ctx.strokeStyle = asteroid.color;
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.fillStyle = `${asteroid.color}33`;
    ctx.fill();
  });
}

function drawParticles() {
  particles.forEach((particle, index) => {
    particle.life--;
    if (particle.life <= 0) {
      particles.splice(index, 1);
      return;
    }

    ctx.beginPath();
    ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2);
    ctx.fillStyle = `rgba(${particle.color}, ${particle.life / particle.initialLife})`;
    ctx.fill();

    particle.x += particle.speedX;
    particle.y += particle.speedY;
    particle.size *= 0.95;
  });
}

function createExplosion(x, y, color) {
  for (let i = 0; i < 20; i++) {
    particles.push({
      x: x,
      y: y,
      speedX: (Math.random() - 0.5) * 5,
      speedY: (Math.random() - 0.5) * 5,
      size: Math.random() * 3 + 1,
      color: color,
      life: Math.random() * 30 + 10,
      initialLife: 40
    });
  }
}

function drawStars() {
  stars.forEach(star => {
    star.brightness += star.twinkleSpeed;
    if (star.brightness > 1 || star.brightness < 0) {
      star.twinkleSpeed = -star.twinkleSpeed;
    }
    ctx.beginPath();
    ctx.arc(star.x, star.y, star.radius, 0, Math.PI * 2);
    ctx.fillStyle = `rgba(255, 255, 255, ${0.5 + star.brightness * 0.5})`;
    ctx.fill();
  });
}

function drawCursor() {
  ctx.beginPath();
  ctx.arc(mouseX, mouseY, 5, 0, Math.PI * 2);
  ctx.fillStyle = 'white';
  ctx.fill();

  ctx.beginPath();
  ctx.moveTo(mouseX - 10, mouseY);
  ctx.lineTo(mouseX + 10, mouseY);
  ctx.moveTo(mouseX, mouseY - 10);
  ctx.lineTo(mouseX, mouseY + 10);
  ctx.strokeStyle = 'white';
  ctx.lineWidth = 2;
  ctx.stroke();
}

function update() {
  ship.x += ship.thrust.x;
  ship.y += ship.thrust.y;

  // Calculate angle between ship and mouse cursor
  ship.angle = Math.atan2(mouseY - ship.y, mouseX - ship.x);

  // Apply thrust towards the mouse cursor
  const thrustPower = 0.1;
  ship.thrust.x += Math.cos(ship.angle) * thrustPower;
  ship.thrust.y += Math.sin(ship.angle) * thrustPower;

  // Apply friction
  ship.thrust.x *= 0.99;
  ship.thrust.y *= 0.99;

  createExplosion(
    ship.x - Math.cos(ship.angle) * ship.radius,
    ship.y - Math.sin(ship.angle) * ship.radius,
    '255, 100, 0'
  );

  wrapCoordinates(ship);

  bullets.forEach((bullet, index) => {
    bullet.x += bullet.speed * Math.cos(bullet.angle);
    bullet.y += bullet.speed * Math.sin(bullet.angle);
    
    if (bullet.x < 0 || bullet.x > canvas.width || bullet.y < 0 || bullet.y > canvas.height) {
      bullets.splice(index, 1);
    }
  });

  asteroids.forEach((asteroid, asteroidIndex) => {
    asteroid.x += Math.cos(asteroid.angle) * asteroid.speed;
    asteroid.y += Math.sin(asteroid.angle) * asteroid.speed;

    wrapCoordinates(asteroid);

    bullets.forEach((bullet, bulletIndex) => {
      if (distanceBetween(asteroid, bullet) < asteroid.radius) {
        createExplosion(asteroid.x, asteroid.y, asteroid.color);
        asteroids.splice(asteroidIndex, 1);
        bullets.splice(bulletIndex, 1);
        score += 10;
        updateScore();
        if (asteroid.radius > 20) {
          for (let i = 0; i < 2; i++) {
            asteroids.push({
              x: asteroid.x,
              y: asteroid.y,
              radius: asteroid.radius / 2,
              speed: asteroid.speed * 1.5,
              angle: Math.random() * Math.PI * 2,
              vertices: Math.floor(Math.random() * 5) + 5,
              color: asteroid.color
            });
          }
        }
      }
    });

    if (distanceBetween(asteroid, ship) < asteroid.radius + ship.radius) {
      createExplosion(ship.x, ship.y, ship.color);
      gameOver();
    }
  });

  if (asteroids.length < 5) {
    createAsteroid();
  }

  if (isFiring) {
    fireBullet();
  }
}

function wrapCoordinates(obj) {
  if (obj.x < 0) obj.x = canvas.width;
  if (obj.x > canvas.width) obj.x = 0;
  if (obj.y < 0) obj.y = canvas.height;
  if (obj.y > canvas.height) obj.y = 0;
}

function distanceBetween(obj1, obj2) {
  return Math.sqrt(Math.pow(obj1.x - obj2.x, 2) + Math.pow(obj1.y - obj2.y, 2));
}

function gameLoop() {
  ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  drawStars();
  update();
  drawShip();
  drawBullets();
  drawAsteroids();
  drawParticles();
  drawCursor();

  requestAnimationFrame(gameLoop);
}

function fireBullet() {
  if (bullets.length < 5) {  // Limit the number of bullets
    bullets.push({
      x: ship.x + Math.cos(ship.angle) * ship.radius,
      y: ship.y + Math.sin(ship.angle) * ship.radius,
      angle: ship.angle,
      speed: 7
    });
    createExplosion(
      ship.x + Math.cos(ship.angle) * ship.radius,
      ship.y + Math.sin(ship.angle) * ship.radius,
      '255, 255, 0'
    );
  }
}

function gameOver() {
  ship.x = canvas.width / 2;
  ship.y = canvas.height / 2;
  ship.thrust = { x: 0, y: 0 };
  asteroids.length = 0;
  bullets.length = 0;
  score = 0;
  updateScore();
  for (let i = 0; i < 5; i++) {
    createAsteroid();
  }
}

function updateScore() {
  scoreElement.textContent = `Score: ${score}`;
}

window.addEventListener('resize', () => {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  createStars();
});

canvas.addEventListener('mousemove', (e) => {
  mouseX = e.clientX;
  mouseY = e.clientY;
});

canvas.addEventListener('mousedown', () => {
  isFiring = true;
});

canvas.addEventListener('mouseup', () => {
  isFiring = false;
});

gameLoop();
</script>
</body>
</html>
