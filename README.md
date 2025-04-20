<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>경마 게임 - 레인 정렬</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }

    .track {
      position: relative;
      width: 2200px;
      height: 500px;
      border: 3px solid #000;
      margin-bottom: 20px;
      background-color: #f0f0f0;
    }

    .horse {
      position: absolute;
      width: 200px;
      height: 40px;
      color: #fff;
      font-weight: bold;
      text-align: center;
      line-height: 40px;
      border-radius: 5px;
    }

    #horse1 { background-color: #e74c3c; }
    #horse2 { background-color: #27ae60; }
    #horse3 { background-color: #2980b9; }

    input[type="number"] { width: 60px; }
  </style>
</head>
<body>

  <h1>🏇 경마 게임 - 고유 레인 + 승률 + 순위</h1>

  <div>
    <label>1번 말 승률 (%) <input id="rate1" type="number" value="60" min="0" max="100"></label>
    <label>2번 말 승률 (%) <input id="rate2" type="number" value="30" min="0" max="100"></label>
    <label>3번 말 승률 (%) <input id="rate3" type="number" value="10" min="0" max="100"></label>
    <button onclick="startRace()">시작</button>
    <button onclick="resetStats()">승률 초기화</button>
  </div>

  <div class="track" id="track">
    <div class="horse" id="horse1">1번 말</div>
    <div class="horse" id="horse2">2번 말</div>
    <div class="horse" id="horse3">3번 말</div>
  </div>

  <div id="ranking"></div>
  <div id="result"></div>
  <div id="stats"></div>

  <script>
    const horses = [
      { id: 'horse1', name: '1번 말', pos: 0, trackStep: 0, wins: 0, speed: 0, index: 0 },
      { id: 'horse2', name: '2번 말', pos: 0, trackStep: 0, wins: 0, speed: 0, index: 1 },
      { id: 'horse3', name: '3번 말', pos: 0, trackStep: 0, wins: 0, speed: 0, index: 2 }
    ];

    const horseWidth = 200;
    const trackWidth = 2200;
    const fullDistance = 4400;
    const lineTop = [
      [30, 110, 190],   // 첫 줄
      [270, 340, 410]   // 두 번째 줄
    ];
    let raceInterval = null;
    let animationId = null;
    let lastFrame = null;
    let running = false;

    function getRates() {
      const r1 = parseInt(document.getElementById('rate1').value) || 0;
      const r2 = parseInt(document.getElementById('rate2').value) || 0;
      const r3 = parseInt(document.getElementById('rate3').value) || 0;
      const total = r1 + r2 + r3 || 1;
      return [r1 / total, r2 / total, r3 / total];
    }

    function updateSpeeds() {
      const winRates = getRates();
      horses.forEach((horse, i) => {
        const rate = winRates[i];
        if (rate >= 0.6) {
          horse.speed = Math.floor(Math.random() * 301) + 700;
        } else if (rate >= 0.3) {
          horse.speed = Math.floor(Math.random() * 501) + 400;
        } else {
          if (Math.random() < rate + 0.05) {
            horse.speed = Math.floor(Math.random() * 201) + 1000;
          } else {
            horse.speed = Math.floor(Math.random() * 51) + 30;
          }
        }
      });
    }

    function updateRanking() {
      const sorted = [...horses].sort((a, b) =>
        (b.trackStep * trackWidth + b.pos) - (a.trackStep * trackWidth + a.pos)
      );
      const html = sorted.map((h, i) => `<li>${i + 1}위: ${h.name}</li>`).join('');
      document.getElementById('ranking').innerHTML = `<h3>🏁 현재 순위</h3><ul>${html}</ul>`;
    }

    function moveHorses(timestamp) {
      if (!lastFrame) lastFrame = timestamp;
      const delta = (timestamp - lastFrame) / 1000;
      lastFrame = timestamp;

      horses.forEach(horse => {
        horse.pos += horse.speed * delta;

        if (horse.trackStep === 0 && horse.pos + horseWidth >= trackWidth) {
          horse.trackStep = 1;
          horse.pos = 0;
        }

        const left = horse.pos;
        const top = lineTop[horse.trackStep][horse.index];
        const elem = document.getElementById(horse.id);
        elem.style.left = left + 'px';
        elem.style.top = top + 'px';
      });

      updateRanking();
      checkWinner();

      if (running) {
        animationId = requestAnimationFrame(moveHorses);
      }
    }

    function checkWinner() {
      for (let horse of horses) {
        const total = horse.trackStep * trackWidth + horse.pos + horseWidth;
        if (total >= fullDistance) {
          running = false;
          cancelAnimationFrame(animationId);
          clearInterval(raceInterval);
          horse.wins++;
          saveStats();
          showResult(horse.name);
          return;
        }
      }
    }

    function showResult(winnerName) {
      document.getElementById('result').innerHTML = `<h2>🎉 우승: ${winnerName}</h2>`;
      updateStatsUI();
    }

    function resetRace() {
      horses.forEach(horse => {
        horse.pos = 0;
        horse.trackStep = 0;
        horse.speed = 0;
        const elem = document.getElementById(horse.id);
        elem.style.left = '0px';
        elem.style.top = lineTop[0][horse.index] + 'px';
      });
      document.getElementById('result').innerHTML = '';
      document.getElementById('ranking').innerHTML = '';
      lastFrame = null;
    }

    function startRace() {
      if (running) return;
      resetRace();
      loadStats();
      updateStatsUI();
      running = true;
      updateSpeeds();
      raceInterval = setInterval(updateSpeeds, 200);
      animationId = requestAnimationFrame(moveHorses);
    }

    function loadStats() {
      horses.forEach(horse => {
        const stored = localStorage.getItem(horse.id);
        if (stored) horse.wins = parseInt(stored);
      });
    }

    function saveStats() {
      horses.forEach(horse => {
        localStorage.setItem(horse.id, horse.wins);
      });
    }

    function resetStats() {
      horses.forEach(horse => {
        horse.wins = 0;
        localStorage.removeItem(horse.id);
      });
      updateStatsUI();
    }

    function updateStatsUI() {
      let total = horses.reduce((acc, h) => acc + h.wins, 0);
      let html = '<h3>📊 누적 승률</h3><ul>';
      horses.forEach(h => {
        const p = total ? ((h.wins / total) * 100).toFixed(1) : 0;
        html += `<li>${h.name}: ${h.wins}승 (${p}%)</li>`;
      });
      html += '</ul>';
      document.getElementById('stats').innerHTML = html;
    }
  </script>

</body>
</html>
