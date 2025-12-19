<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>モンハン装備ガチャ</title>

<style>
body {
  font-family: "Segoe UI","Hiragino Kaku Gothic ProN",sans-serif;
  min-height: 100vh;
  margin: 0;
  background-image: url("background.jpg");
  background-repeat: no-repeat;
  background-position: center center;
  background-attachment: fixed;
  background-size: cover;
  color:#f5f5f5;
  text-align:center;
  padding:30px;
}

h1 { color:#f0c75e; text-shadow:0 0 6px #664c00; }

button {
  padding:10px 16px; margin:6px;
  border-radius:8px; border:2px solid #f0c75e;
  background:#2b2b2bcc; color:#f0c75e;
  cursor:pointer;
}

.card {
  background: rgba(30,30,30,0.75);
  padding:20px;
  border-radius:12px;
  margin:20px auto;
  width:90%; max-width:520px;
  text-align:left;
  border:1px solid #f0c75e55;
}

ul { list-style:none; padding:0; }
li { background:rgba(44,44,44,0.8); margin:10px 0; padding:12px; border-radius:6px; }

.saved-card { border-bottom:1px solid #555; margin-bottom:10px; padding-bottom:10px; }
.delete-btn { background:#7a1f1f; border-color:#ff6666; color:#fff; }

input[type="text"] {
  width:90%; padding:8px;
  border-radius:6px; border:1px solid #f0c75e;
  background:#2b2b2bcc; color:#fff;
}

/* 文字サイズ調整 */
.armor-name {
  font-size: 1.35em;
  font-weight: bold;
}

.armor-monster {
  font-size: 1.15em;
}

.armor-material {
  font-size: 1.0em;
}
</style>
</head>

<body>

<h1>🎯 モンハン装備ガチャ</h1>

<div class="card" style="text-align:center">
  <b>ランク選択</b><br>
  <label><input type="radio" name="rank" value="low" checked> 下位</label>
  <label><input type="radio" name="rank" value="high"> 上位</label>
  <label><input type="radio" name="rank" value="mr"> MR</label>
</div>

<button onclick="generateLoadout()">🎲 装備を決定</button>

<div id="result" class="card">ここに結果が表示されます</div>

<div class="card">
  <h3>💾 保存</h3>
  <input id="setName" type="text" placeholder="装備名">
  <br>
  <button onclick="saveLoadout()">保存</button>
</div>

<div class="card">
  <h3>📚 保存した装備</h3>
  <div id="savedList"></div>
</div>

<script>
const weapons = [
  "大剣","太刀","片手剣","双剣","ハンマー","狩猟笛",
  "ランス","ガンランス","スラッシュアックス",
  "チャージアックス","操虫棍","ライトボウガン",
  "ヘビィボウガン","弓"
];

const monstersByRank = {
  low: ["アオアシラ","オサイズチ","クルルヤック","ラングロトラ","ドスフロギィ","ドスバギィ","ウルクスス"],
  highOnly: ["リオレイア","リオレウス","ジンオウガ","ナルガクルガ","タマミツネ","マガイマガド"],
  mrOnly: ["メル・ゼナ","原初を刻むメル・ゼナ","ルナガロン","ガランゴルム"],
  nonLarge: [
    { name:"チェーン", material:"素材：鉱石" },
    { name:"ハンター", material:"素材：小型モンスター" },
    { name:"ボーン", material:"素材：小型モンスター" }
  ]
};

let current = null;
const rand = arr => arr[Math.floor(Math.random()*arr.length)];

function getMonsterRank(m) {
  if (monstersByRank.low.includes(m)) return "下位";
  if (monstersByRank.highOnly.includes(m)) return "上位";
  if (monstersByRank.mrOnly.includes(m)) return "MR";
  return "";
}

function getArmorName(src, part) {
  const suf = {頭:"ヘルム",胴:"メイル",腕:"アーム",腰:"コイル",脚:"グリーヴ"};
  return typeof src === "string" ? src + suf[part] : src.name + suf[part];
}

function getMaterial(src) {
  return typeof src === "string" ? `素材：${src}` : src.material;
}

function generateLoadout() {
  const rank = document.querySelector('input[name="rank"]:checked').value;
  let pool = [];

  if (rank === "low") pool = monstersByRank.low;
  if (rank === "high") pool = monstersByRank.low.concat(monstersByRank.highOnly);
  if (rank === "mr") pool = monstersByRank.low.concat(monstersByRank.highOnly, monstersByRank.mrOnly);

  pool = pool.concat(monstersByRank.nonLarge);

  current = {
    weapon: rand(weapons),
    armor: {
      頭: rand(pool), 胴: rand(pool),
      腕: rand(pool), 腰: rand(pool), 脚: rand(pool)
    }
  };

  let html = `<h2>🗡 武器：${current.weapon}</h2><ul>`;
  for (const part in current.armor) {
    const src = current.armor[part];
    const monster = typeof src === "string" ? `${src}（${getMonsterRank(src)}）` : src.name;
    html += `
      <li>
        <div class="armor-name">【${part}】${getArmorName(src,part)}</div>
        <div class="armor-monster">┗ モンスター：${monster}</div>
        <div class="armor-material">┗ ${getMaterial(src)}</div>
      </li>`;
  }
  html += "</ul>";
  document.getElementById("result").innerHTML = html;
}

function saveLoadout() {
  if (!current) return;
  const name = document.getElementById("setName").value || "無名装備";
  const data = JSON.parse(localStorage.getItem("mh_sets") || "[]");
  data.push({ name, ...current });
  localStorage.setItem("mh_sets", JSON.stringify(data));
  renderSaved();
}

function deleteLoadout(i) {
  const data = JSON.parse(localStorage.getItem("mh_sets") || "[]");
  data.splice(i,1);
  localStorage.setItem("mh_sets", JSON.stringify(data));
  renderSaved();
}

function renderSaved() {
  const data = JSON.parse(localStorage.getItem("mh_sets") || "[]");
  const list = document.getElementById("savedList");
  if (data.length === 0) {
    list.innerHTML = "<p>保存なし</p>";
    return;
  }
  list.innerHTML = data.map((s,i)=>`
    <div class="saved-card">
      <b>${s.name}</b><br>
      武器：${s.weapon}<br>
      <button class="delete-btn" onclick="deleteLoadout(${i})">削除</button>
    </div>
  `).join("");
}

renderSaved();
</script>
</body>
</html>
