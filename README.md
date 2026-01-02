# C-C_Tokumei_keijiban
クトゥルフ神話の登場人物の名前を乱数で与えられる匿名掲示板です。
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>クトゥルフ神話コードジェネレーター</title>
<style>
:root {
  --bg: #f7f7f7;
  --card: #ffffff;
  --text: #222;
  --shadow: 0 20px 40px rgba(0,0,0,.08);
}

body {
  margin: 0;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--bg);
  font-family: system-ui, -apple-system, sans-serif;
  color: var(--text);
}

.card {
  background: var(--card);
  padding: 50px 60px;
  border-radius: 20px;
  box-shadow: var(--shadow);
  text-align: center;
  min-width: 360px;
}

h1 {
  font-size: 18px;
  letter-spacing: .2em;
  font-weight: 600;
  margin-bottom: 30px;
  color: #555;
}

#result {
  font-size: 42px;
  font-weight: 700;
  letter-spacing: .15em;
  margin: 30px 0 10px;
  transition: color .3s ease, transform .2s ease;
}

#subresult {
  font-size: 12px;
  color: rgba(0,0,0,0.3);
  margin-bottom: 15px;
  transition: color .3s ease;
}

#character {
  font-size: 20px;
  font-weight: 600;
  transition: color .3s ease;
}

button {
  background: linear-gradient(135deg, #e6c75b, #c9a227);
  border: none;
  border-radius: 999px;
  padding: 14px 40px;
  font-size: 16px;
  font-weight: 600;
  letter-spacing: .1em;
  color: #fff;
  cursor: pointer;
  box-shadow: 0 10px 20px rgba(0,0,0,.15);
  transition: transform .15s ease, box-shadow .15s ease;
}

button:hover {
  transform: translateY(-2px);
  box-shadow: 0 14px 28px rgba(0,0,0,.2);
}

button:active {
  transform: translateY(0);
  box-shadow: 0 8px 16px rgba(0,0,0,.15);
}
</style>
</head>
<body>
<div class="card">
  <h1>クトゥルフコード</h1>
  <div id="subresult">----</div>
  <div id="result">----</div>
  <div id="character">----</div>
  <button onclick="drawCode()">生成</button>
</div>

<script>
// --- 文字・数字 ---
const LETTERS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
const DIGITS = "0123456789";
const usedCodes = new Set();

// --- クトゥルフ神話キャラクター（日本語＋色） ---
const CTHULHU_ENTITIES_FULL = [
  { name: "アザトース", color: "red" }, { name: "ヨグ＝ソトース", color: "red" },
  { name: "ニャルラトホテプ", color: "red" }, { name: "シャブ＝ニグラス", color: "red" },
  { name: "クトゥルフ", color: "red" }, { name: "ハスター", color: "red" },
  { name: "ダゴン", color: "red" }, { name: "ツァトゥグァ", color: "red" },
  { name: "ガタノトーア", color: "red" }, { name: "ボクラグ", color: "red" },

  { name: "イタカ", color: "orange" }, { name: "ノーデンス", color: "orange" },
  { name: "ヒプノス", color: "orange" }, { name: "Zhar", color: "orange" },
  { name: "Lloigor", color: "orange" }, { name: "アトラク＝ナチャ", color: "orange" },
  { name: "イーゴーン", color: "orange" }, { name: "ミ＝ゴ", color: "orange" },
  { name: "ナイトガント", color: "orange" }, { name: "深きものども", color: "orange" },
  { name: "古き者たち", color: "orange" }, { name: "シャゴス", color: "orange" },

  { name: "ブラウン・ジェンキン", color: "yellow" }, { name: "飛行ポリプ", color: "yellow" },
  { name: "バイキーズ", color: "yellow" }, { name: "星の使徒", color: "yellow" },
  { name: "ムーンビースト", color: "yellow" }, { name: "隠された者たち", color: "yellow" },
  { name: "スティアニーの子", color: "yellow" }, { name: "蛇人", color: "yellow" },
  { name: "夢の国の存在", color: "yellow" }, { name: "ユゴスの菌類", color: "yellow" },
  { name: "チョ＝チョ", color: "yellow" }, { name: "エルダー・サインの守護者", color: "yellow" },

  { name: "アブホロス", color: "blue" }, { name: "アララ", color: "blue" },
  { name: "アムムツェバ", color: "blue" }, { name: "アモン＝ゴルロス", color: "blue" },
  { name: "アニムス", color: "blue" }, { name: "アプーム＝ザハ", color: "blue" },
  { name: "アーラッサ", color: "blue" }, { name: "アイイグ", color: "blue" },
  { name: "アイリス", color: "blue" }, { name: "バオート・ズッカ＝モッグ", color: "blue" },
  { name: "バサタン", color: "blue" }, { name: "セレスティアル・トイメーカー", color: "blue" },
  { name: "チャウナール・ファウグン", color: "blue" }, { name: "クトハート", color: "blue" },
  { name: "クトエギャ", color: "blue" }, { name: "クトゥガ", color: "blue" },
  { name: "クティラ", color: "blue" }, { name: "クトガ", color: "blue" },
  { name: "シアエギャ", color: "blue" }, { name: "クィノフ＝ケー", color: "blue" },
  { name: "ダイグラ", color: "blue" }, { name: "ディサラ", color: "blue" },
  { name: "ゼウァ", color: "blue" }, { name: "アイホルト", color: "blue" },
  { name: "アイドロン", color: "blue" }, { name: "エイロル", color: "blue" },
  { name: "エテプセッド・エグニス", color: "blue" }, { name: "フェンリック", color: "blue" },
  { name: "フタッグア", color: "blue" }, { name: "ガダモン", color: "blue" },
  { name: "ギスグス", color: "blue" }, { name: "ギホベグ", color: "blue" },
  { name: "グラーキ", color: "blue" }, { name: "グリース", color: "blue" },
  { name: "グルーン", color: "blue" }, { name: "ゴボゲグ", color: "blue" },
  { name: "ゴルゴロス", color: "blue" }, { name: "ゴロテス", color: "blue" },
  { name: "グラウサ", color: "blue" }, { name: "グリーン・ゴッド", color: "blue" },
  { name: "グロス＝ゴルカ", color: "blue" }, { name: "グトゥハナイ", color: "blue" },
  { name: "グラサナカ", color: "blue" }, { name: "グルラ＝ヤ", color: "blue" },
  { name: "グワルロス", color: "blue" }, { name: "グズティオス", color: "blue" },
  { name: "ハン", color: "blue" }, { name: "H'チェレゴス", color: "blue" },
  { name: "ハイオグ＝ヤイ", color: "blue" }, { name: "ハスタリュク", color: "blue" },
  { name: "クロスミエビクス", color: "blue" }, { name: "クルパンガ", color: "blue" },
  { name: "リサリア", color: "blue" }, { name: "M'ナガラ", color: "blue" },
  { name: "ムノムカ", color: "blue" }, { name: "モルディギアン", color: "blue" },
  { name: "モルモ", color: "blue" }, { name: "モルトルグ", color: "blue" },
  { name: "ミノグラ", color: "blue" }, { name: "エンギル＝コラス", color: "blue" },
  { name: "ネームレス・ミスト", color: "blue" }, { name: "母なる膿", color: "blue" },
  { name: "ルピン種", color: "blue" }, { name: "Fungi From Yuggoth", color: "blue" },
  { name: "Tcho-Tcho", color: "blue" }
];

// --- コード生成 ---
function generateUniqueCode(){
  let code;
  do {
    let letters="", digits="";
    for(let i=0;i<4;i++) letters+=LETTERS[Math.floor(Math.random()*LETTERS.length)];
    for(let i=0;i<4;i++) digits+=DIGITS[Math.floor(Math.random()*DIGITS.length)];
    code = letters+digits;
  } while(usedCodes.has(code));
  usedCodes.add(code);
  return code;
}

function isNumberZorome(code){ return code.slice(4).split("").every(v=>v===code[4]); }
function isLetterZorome(code){ return code.slice(0,4).split("").every(v=>v===code[0]); }

function assignEntity(code){
  let sum=0; for(let c of code) sum+=c.charCodeAt(0);
  return CTHULHU_ENTITIES_FULL[sum % CTHULHU_ENTITIES_FULL.length];
}

function getCodeColor(code){
  const n=isNumberZorome(code), l=isLetterZorome(code);
  if(n && l) return "#c9a227";
  if(n) return "#c0392b";
  if(l) return "#2980b9";
  return "#222";
}

function drawCode(){
  const code=generateUniqueCode();
  const r=document.getElementById("result");
  const s=document.getElementById("subresult");
  const c=document.getElementById("character");
  const entity=assignEntity(code);

  r.textContent=code; r.style.color=getCodeColor(code); r.style.transform="scale(1.05)";
  s.textContent=code; s.style.color="rgba(0,0,0,0.3)";
  c.textContent=entity.name; c.style.color=entity.color;

  setTimeout(()=>r.style.transform="scale(1)",150);
}
</script>
</body>
</html>
