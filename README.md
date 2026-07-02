# aikengames
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bullet Fusion</title>
<style>
  body {
    margin: 0;
    font-family: Arial, "Microsoft JhengHei", sans-serif;
    background: #111827;
    color: white;
    text-align: center;
  }

  h1 {
    margin-top: 20px;
  }

  #score {
    font-size: 24px;
    margin: 10px;
  }

  #board {
    width: 360px;
    height: 360px;
    margin: 20px auto;
    display: grid;
    grid-template-columns: repeat(8, 1fr);
    grid-template-rows: repeat(8, 1fr);
    gap: 4px;
    background: #374151;
    padding: 6px;
    border-radius: 14px;
  }

  .cell {
    background: #1f2937;
    border-radius: 10px;
    font-size: 28px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    user-select: none;
    transition: 0.15s;
  }

  .cell.selected {
    outline: 3px solid #facc15;
    transform: scale(1.08);
  }

  .cell.pop {
    animation: pop 0.25s ease;
  }

  @keyframes pop {
    0% { transform: scale(1); }
    50% { transform: scale(1.35); }
    100% { transform: scale(1); }
  }

  button {
    padding: 12px 24px;
    font-size: 18px;
    border: none;
    border-radius: 10px;
    background: #ef4444;
    color: white;
    cursor: pointer;
  }

  button:hover {
    background: #dc2626;
  }

  .info {
    font-size: 14px;
    color: #d1d5db;
    margin: 10px 20px;
  }
</style>
</head>
<body>

<h1>💥 Bullet Fusion 💥</h1>
<div id="score">分數：0</div>

<div id="board"></div>

<button onclick="restartGame()">重新開始</button>

<div class="info">
  小子彈 → 大子彈 → 手榴彈 → 閃光彈 → 原子彈<br>
  點選相鄰兩格交換，三個以上相同武器會升階！
</div>

<script>
const size = 8;
const boardElement = document.getElementById("board");
const scoreElement = document.getElementById("score");

const weapons = [
  { name: "小子彈", icon: "🔸" },
  { name: "大子彈", icon: "🔶" },
  { name: "手榴彈", icon: "💣" },
  { name: "閃光彈", icon: "✨" },
  { name: "原子彈", icon: "☢️" }
];

let board = [];
let selected = null;
let score = 0;

function createBoard() {
  board = [];

  for (let r = 0; r < size; r++) {
    board[r] = [];
    for (let c = 0; c < size; c++) {
      board[r][c] = randomWeapon();
    }
  }

  renderBoard();
}

function randomWeapon() {
  return Math.floor(Math.random() * 4); 
}

function renderBoard() {
  boardElement.innerHTML = "";

  for (let r = 0; r < size; r++) {
    for (let c = 0; c < size; c++) {
      const cell = document.createElement("div");
      cell.className = "cell";
      cell.textContent = weapons[board[r][c]].icon;
      cell.dataset.row = r;
      cell.dataset.col = c;

      if (selected && selected.r === r && selected.c === c) {
        cell.classList.add("selected");
      }

      cell.addEventListener("click", () => handleClick(r, c));
      boardElement.appendChild(cell);
    }
  }

  scoreElement.textContent = "分數：" + score;
}

function handleClick(r, c) {
  if (!selected) {
    selected = { r, c };
    renderBoard();
    return;
  }

  if (selected.r === r && selected.c === c) {
    selected = null;
    renderBoard();
    return;
  }

  if (isAdjacent(selected.r, selected.c, r, c)) {
    swap(selected.r, selected.c, r, c);

    const matches = findMatches();

    if (matches.length > 0) {
      processMatches(matches);
    } else {
      swap(selected.r, selected.c, r, c);
    }

    selected = null;
    renderBoard();
  } else {
    selected = { r, c };
    renderBoard();
  }
}

function isAdjacent(r1, c1, r2, c2) {
  return Math.abs(r1 - r2) + Math.abs(c1 - c2) === 1;
}

function swap(r1, c1, r2, c2) {
  const temp = board[r1][c1];
  board[r1][c1] = board[r2][c2];
  board[r2][c2] = temp;
}

function findMatches() {
  let matches = [];

  for (let r = 0; r < size; r++) {
    let count = 1;

    for (let c = 1; c < size; c++) {
      if (board[r][c] === board[r][c - 1]) {
        count++;
      } else {
        if (count >= 3) {
          for (let i = 0; i < count; i++) {
            matches.push({ r, c: c - 1 - i });
          }
        }
        count = 1;
      }
    }

    if (count >= 3) {
      for (let i = 0; i < count; i++) {
        matches.push({ r, c: size - 1 - i });
      }
    }
  }

  for (let c = 0; c < size; c++) {
    let count = 1;

    for (let r = 1; r < size; r++) {
      if (board[r][c] === board[r - 1][c]) {
        count++;
      } else {
        if (count >= 3) {
          for (let i = 0; i < count; i++) {
            matches.push({ r: r - 1 - i, c });
          }
        }
        count = 1;
      }
    }

    if (count >= 3) {
      for (let i = 0; i < count; i++) {
        matches.push({ r: size - 1 - i, c });
      }
    }
  }

  return removeDuplicates(matches);
}

function removeDuplicates(arr) {
  const map = {};
  return arr.filter(item => {
    const key = item.r + "," + item.c;
    if (map[key]) return false;
    map[key] = true;
    return true;
  });
}

function processMatches(matches) {
  const first = matches[0];
  const currentLevel = board[first.r][first.c];

  score += matches.length * 100;

  if (currentLevel === 4) {
    atomicBomb();
    return;
  }

  matches.forEach(pos => {
    board[pos.r][pos.c] = randomWeapon();
  });

  board[first.r][first.c] = Math.min(currentLevel + 1, 4);

  setTimeout(() => {
    const newMatches = findMatches();
    if (newMatches.length > 0) {
      processMatches(newMatches);
      renderBoard();
    }
  }, 200);
}

function atomicBomb() {
  score += 5000;

  for (let r = 0; r < size; r++) {
    for (let c = 0; c < size; c++) {
      board[r][c] = randomWeapon();
    }
  }

  alert("☢️ 原子彈爆炸！全場清空！");
}

function restartGame() {
  score = 0;
  selected = null;
  createBoard();
}

createBoard();
</script>

</body>
</html>
