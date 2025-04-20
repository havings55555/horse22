<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>2줄 트랙 + 사용자 승률 설정</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }

    .track {
      position: relative;
      width: 2000px;
      height: 260px;
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

  <h1>2줄 트랙 경마 (사용자 승률 설정)</h1>

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

  <div id="result"></div>
  <div id="stats"></div>

  <script>
    const horses = [
      { id: 'horse1', name: '1번 말', pos: 0, trackStep: 0, wins: 0, speed: 0 },
      { id: 'horse2', name: '2번 말', pos: 0, trackStep: 0, wins: 0, speed: 0 },
      { id: 'horse3', name: '3번 말', pos: 0, trackStep: 0, wins: 0, speed: 0 }
    ];

    const horseWidth = 200;
    const trackWidth = 2000;
    const fullDistance = 4000;
    const lineTop = [30, 150];
    let raceInterval = null;
    let animationId = null;
    let lastFrame = null;
    let running = false;
    let winRates = [0.6, 0.3, 0.1];

    function getRates() {
      const r1 = parseInt(document.getElementById('rate1').value) || 0;
      const r2 = parseInt(document.getElementById('rate2').value) || 0;
      const r3 = parseInt(document.getElementById('rate3').value) || 0;
      const total = r1 + r2 + r3 || 1;
      return [r1 / total, r2 / total, r3 / total];
    }

    function updateSpeeds() {
      winRates = getRates();

      horses.forEach((horse, i) => {
        const rate = winRates[i];

        if (rate >= 0.6) {
          horse.speed = Math.floor(Math.random() * 301) + 700;  // 700~1000
        } else if (rate >= 0.3) {
          horse.speed = Math.floor(Math.random() * 501) + 400;  // 400~900
        } else {
          if (Math.random() < rate + 0.05) {
            horse.speed = Math.floor(Math.random() * 201) + 1000; // 빠름
          } else {
            horse.speed = Math.floor(Math.random() * 51) + 30;   // 느림
          }
        }
      });
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
        const top = lineTop[horse.trackStep];
        document.getElementById(horse.id).style.left = left + 'px';
        document.getElementById(horse.id).style.top = top + 'px';
      });

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

    function resetRace() {
      horses.forEach(horse => {
        horse.pos = 0;
        horse.trackStep = 0;
        horse.speed = 0;
        document.getElementById(horse.id).style.left = '0px';
        document.getElementById(horse.id).style.top = lineTop[0] + 'px';
      });
      document.getElementById('result').innerHTML = '';
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
        const storedWins = localStorage.getItem(horse.id);
        if (storedWins) horse.wins = parseInt(storedWins);
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
      let totalWins = horses.reduce((acc, h) => acc + h.wins, 0);
      let statsHTML = '<h3>승률</h3><ul>';
      horses.forEach(h => {
        const percent = totalWins ? ((h.wins / totalWins) * 100).toFixed(1) : 0;
        statsHTML += `<li>${h.name}: ${h.wins}승 (${percent}%)</li>`;
      });
      statsHTML += '</ul>';
      document.getElementById('stats').innerHTML = statsHTML;
    }
  </script>

</body>
</html>
