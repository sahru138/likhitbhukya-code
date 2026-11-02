<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The Six Realms: Portal Quest</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  background: #101426;
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
}

h1 { color: #ffd166; }

#hud {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 20px;
  padding: 12px;
  font-size: 16px;
}

canvas {
  background: #203c32;
  border: 4px solid #ffffff;
  max-width: 95vw;
  width: 720px;
  height: auto;
  image-rendering: pixelated;
}

button {
  background: #ffd166;
  color: #182033;
  border: none;
  padding: 12px 24px;
  border-radius: 9px;
  font-weight: bold;
  cursor: pointer;
  margin: 8px;
}

button:hover { background: #ffb703; }

#screen {
  position: fixed;
  inset: 0;
  background: rgba(10, 15, 30, 0.96);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  z-index: 5;
}

#screen h2 {
  color: #ffd166;
  font-size: 36px;
}

#screen p {
  font-size: 18px;
  max-width: 500px;
  line-height: 1.6;
}

#screen.hidden { display: none; }

#help { color: #cbd5e1; }
</style>
</head>

<body>

<h1>🌍 THE SIX REALMS</h1>
<h3>Portal Quest</h3>

<div id="hud">
  <span id="world">World: 1</span>
  <span id="level">Level: 1 / 60</span>
  <span id="items">Items: 0 / 3</span>
  <span id="coins">Coins: 0</span>
  <span id="xp">XP: 0</span>
</div>

<canvas id="game" width="720" height="480"></canvas>

<p id="help">
  W = Up | A = Left | S = Down | D = Right
</p>

<div id="screen">
  <h2>🌟 THE SIX REALMS</h2>
  <p>
    Six magical worlds. Sixty challenging levels.
    Collect treasures, overcome obstacles and repair the
    portal machine!
  </p>
  <button id="startButton">START ADVENTURE</button>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const TILE = 48;
const COLS = 15;
const ROWS = 10;

const worlds = [
  {
    name: "Enchanted Forest",
    welcome: "Welcome to the Enchanted Forest!",
    description: "Find the lost crystals, avoid obstacles and unlock the forest portal.",
    color: "#245c3c",
    item: "💎"
  },
  {
    name: "Lava Kingdom",
    welcome: "Welcome to the Lava Kingdom!",
    description: "Cross dangerous paths, avoid lava and find the fire key.",
    color: "#8d3024",
    item: "🔥"
  },
  {
    name: "Ocean Realm",
    welcome: "Welcome to the Ocean Realm!",
    description: "Explore underwater areas, find hidden treasure and solve water puzzles.",
    color: "#075985",
    item: "🐚"
  },
  {
    name: "Cyber City",
    welcome: "Welcome to Cyber City!",
    description: "Restore electricity, activate machines and unlock the next portal.",
    color: "#3730a3",
    item: "⚡"
  },
  {
    name: "Frozen Kingdom",
    welcome: "Welcome to the Frozen Kingdom!",
    description: "Find hidden keys, solve ice puzzles and reach the ice portal.",
    color: "#397a9c",
    item: "🔑"
  },
  {
    name: "Final Dimension",
    welcome: "Welcome to the Final Dimension!",
    description: "Find the missing machine parts and repair the portal machine.",
    color: "#512c70",
    item: "⚙️"
  }
];

let currentLevel = 1;
let coins = 0;
let xp = 0;
let collected = 0;
let player = { x: 1, y: 1 };
let items = [];
let walls = [];
let gameRunning = false;

const screen = document.getElementById("screen");

function saveProgress() {
  localStorage.setItem("portalQuestSave", JSON.stringify({
    currentLevel, coins, xp
  }));
}

function loadProgress() {
  const saved = localStorage.getItem("portalQuestSave");
  if (saved) {
    const data = JSON.parse(saved);
    currentLevel = data.currentLevel || 1;
    coins = data.coins || 0;
    xp = data.xp || 0;
  }
}

function getWorld() {
  return worlds[Math.floor((currentLevel - 1) / 10)];
}

function getLevelInWorld() {
  return ((currentLevel - 1) % 10) + 1;
}

function updateHUD() {
  document.getElementById("world").textContent =
    "World: " + getWorld().name;

  document.getElementById("level").textContent =
    "Level: " + currentLevel + " / 60";

  document.getElementById("items").textContent =
    "Items: " + collected + " / 3";

  document.getElementById("coins").textContent =
    "Coins: " + coins;

  document.getElementById("xp").textContent =
    "XP: " + xp;
}

function showScreen(title, message, buttonText, action) {
  screen.classList.remove("hidden");
  screen.innerHTML = `
    <h2>${title}</h2>
    <p>${message}</p>
    <button id="screenButton">${buttonText}</button>
  `;
  document.getElementById("screenButton").onclick = action;
}

function welcomeWorld() {
  const world = getWorld();

  showScreen(
    "🌍 " + world.name,
    "<strong>" + world.welcome + "</strong><br><br>" +
    world.description +
    "<br><br>Level " + getLevelInWorld() + " of 10",
    "ENTER WORLD",
    startLevel
  );
}

function startLevel() {
  screen.classList.add("hidden");

  player = { x: 1, y: 1 };
  collected = 0;

  // Three collectibles in each level.
  items = [
    { x: 3, y: 2 },
    { x: 7, y: 5 },
    { x: 11, y: 7 }
  ];

  // Obstacles.
  walls = [
    { x: 4, y: 1 },
    { x: 4, y: 2 },
    { x: 4, y: 3 },
    { x: 8, y: 4 },
    { x: 8, y: 5 },
    { x: 8, y: 6 },
    { x: 2, y: 7 },
    { x: 3, y: 7 },
    { x: 10, y: 2 },
    { x: 11, y: 2 }
  ];

  gameRunning = true;
  updateHUD();
  drawGame();
}

function drawGame() {
  const world = getWorld();

  ctx.fillStyle = world.color;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Draw grid.
  for (let x = 0; x < COLS; x++) {
    for (let y = 0; y < ROWS; y++) {
      ctx.strokeStyle = "rgba(255,255,255,0.08)";
      ctx.strokeRect(x*TILE, y*TILE, TILE, TILE);
    }
  }

  // Draw obstacles.
  walls.forEach(w => {
    ctx.fillStyle = "#292936";
    ctx.fillRect(w.x*TILE+3, w.y*TILE+3, TILE-6, TILE-6);
    ctx.fillStyle = "#888";
    ctx.fillText("🪨", w.x*TILE+12, w.y*TILE+31);
  });

  // Draw collectibles.
  items.forEach(item => {
    ctx.font = "27px Arial";
    ctx.fillText(world.item,
      item.x*TILE+9, item.y*TILE+34);
  });

  // Draw portal.
  ctx.font = "34px Arial";
  ctx.fillText(
    "🌀", 13*TILE+5, 8*TILE+36
  );

  // Draw player.
  ctx.font = "34px Arial";
  ctx.fillText(
    "🧙",
    player.x*TILE+5,
    player.y*TILE+36
  );
}

function movePlayer(dx, dy) {
  if (!gameRunning) return;

  const newX = player.x + dx;
  const newY = player.y + dy;

  if (
    newX < 0 || newX >= COLS ||
    newY < 0 || newY >= ROWS
  ) return;

  const blocked = walls.some(w =>
    w.x === newX && w.y === newY
  );

  if (blocked) return;

  player.x = newX;
  player.y = newY;

  // Collect items.
  const index = items.findIndex(item =>
    item.x === player.x && item.y === player.y
  );

  if (index !== -1) {
    items.splice(index, 1);
    collected++;
    coins += 10;
    xp += 20;
    updateHUD();
  }

  // Enter portal after collecting all three.
  if (player.x === 13 && player.y === 8) {
    if (collected === 3) {
      completeLevel();
      return;
    } else {
      showMessage("🔒 Portal Locked",
        "Collect all 3 items to unlock the portal!");
    }
  }

  drawGame();
}

function showMessage(title, message) {
  const old = document.getElementById("message");
  if (old) old.remove();

  const div = document.createElement("div");
  div.id = "message";
  div.textContent = title + " – " + message;
  div.style.padding = "12px";
  div.style.color = "#ffd166";
  document.body.appendChild(div);

  setTimeout(() => div.remove(), 1800);
}

function completeLevel() {
  gameRunning = false;
  xp += 50;
  coins += 25;

  if (currentLevel === 60) {
    saveProgress();
    showScreen(
      "🏆 YOU SAVED THE REALMS!",
      "You completed all six worlds, found the machine parts, repaired the portal machine and activated the final portal! You are the ultimate explorer!",
      "PLAY AGAIN",
      () => {
        currentLevel = 1;
        coins = 0;
        xp = 0;
        saveProgress();
        welcomeWorld();
      }
    );
    return;
  }

  const oldWorld = Math.floor((currentLevel - 1) / 10);
  currentLevel++;

  saveProgress();
  updateHUD();

  const newWorld = Math.floor((currentLevel - 1) / 10);

  if (newWorld !== oldWorld) {
    welcomeWorld();
  } else {
    showScreen(
      "⭐ LEVEL COMPLETED!",
      "Amazing! You earned 50 XP and 25 coins. Get ready for level " +
      currentLevel + "!",
      "NEXT LEVEL",
      startLevel
    );
  }
}

document.addEventListener("keydown", event => {
  const key = event.key.toLowerCase();

  if (["w", "a", "s", "d", "arrowup",
       "arrowdown", "arrowleft", "arrowright"].includes(key)) {
    event.preventDefault();
  }

  if (key === "w" || key === "arrowup")
    movePlayer(0, -1);

  if (key === "s" || key === "arrowdown")
    movePlayer(0, 1);

  if (key === "a" || key === "arrowleft")
    movePlayer(-1, 0);

  if (key === "d" || key === "arrowright")
    movePlayer(1, 0);
});

document.getElementById("startButton").onclick = () => {
  loadProgress();
  welcomeWorld();
};

loadProgress();
updateHUD();
</script>

</body>
</html>

AT last 
PRINT "You have completed the game's every single world, to say i am kinda impressed, you made it to the last world, finally, you can just roam anywhere now, but remember, if an update comes, you gotta be available, worlds will increase, camping pressure will be more, harder to move through worlds, you might enable PVP, for any fights, 1v1, 2v2, 3v3, and etc.
