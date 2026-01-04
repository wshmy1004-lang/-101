<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>제20회 전국 교사대회 경품추첨기</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root {
            --primary: #ff4757;
            --gold: #ffa502;
            --dark: #2f3542;
            --skyblue: #a2e4f6; /* 하늘색 변수 추가 */
        }
        body { font-family: 'Pretendard', sans-serif; background: #2f3542; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .container {
            background: white; padding: 40px; border-radius: 30px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.3); width: 100%; max-width: 550px;
            text-align: center; position: relative;
        }
        .sync-section { margin-bottom: 20px; display: flex; gap: 10px; }
        #sheet-id { flex: 1; padding: 12px; border-radius: 12px; border: 2px solid #ddd; font-size: 14px; outline: none; background: #f8f9fa; }
        #sync-btn { background: var(--dark); color: white; border: none; padding: 10px 20px; border-radius: 12px; cursor: pointer; font-weight: bold; transition: 0.2s; }

        /* 1. 가운데 전광판 배경색 수정 (background 부분) */
        #display-box {
            height: 180px; margin: 20px 0; border: 8px solid var(--dark);
            border-radius: 25px; display: flex; align-items: center; justify-content: center;
            background: var(--skyblue); /* 이 부분이 하늘색으로 변경되었습니다 */
            position: relative; overflow: hidden;
        }

        #display-area { font-size: 3.5rem; font-weight: 900; color: var(--dark); transition: all 0.1s; }
       
        .shake { animation: shake 0.1s infinite; }
        @keyframes shake {
            0% { transform: translate(2px, 2px); }
            50% { transform: translate(-2px, -2px); }
            100% { transform: translate(2px, -2px); }
        }
       
        /* 2. 당첨자 팝업 글씨 크기 조절 */
        .winner-pop {
            animation: pop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            color: var(--primary) !important;
            font-size: 2.5rem !important; /* 문구가 길어지므로 폰트 크기를 약간 줄임 */
        }
        @keyframes pop { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }

        .controls { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
        select, button { padding: 15px; border-radius: 12px; border: 1px solid #ddd; font-size: 16px; outline: none; }
        #draw-btn {
            background: var(--primary); color: white; border: none; font-weight: bold;
            cursor: pointer; grid-column: span 2; font-size: 1.5rem;
            box-shadow: 0 5px 0 #b33939; transition: 0.1s;
        }
        #draw-btn:active { transform: translateY(3px); box-shadow: 0 2px 0 #b33939; }
        #draw-btn:disabled { background: #ced4da; box-shadow: none; cursor: not-allowed; }

        .winner-log { margin-top: 30px; text-align: left; max-height: 200px; overflow-y: auto; }
        .winner-item {
            background: #f8f9fa; margin-bottom: 8px; padding: 12px 20px;
            border-radius: 10px; display: flex; justify-content: space-between;
            border-left: 5px solid var(--gold); animation: slideIn 0.4s ease-out;
        }
        @keyframes slideIn { from { opacity: 0; transform: translateX(-20px); } to { opacity: 1; transform: translateX(0); } }
    </style>
</head>
<body>

<div class="container">
    <h2 style="margin-top:0; color: var(--dark);">🏆 제20회 전국 교사대회 추첨</h2>

    <div class="sync-section">
        <input type="text" id="sheet-id" placeholder="ID가 내장되어 있습니다.">
        <button id="sync-btn" onclick="fetchSheetData()">데이터 연동</button>
    </div>

    <div id="display-box">
        <div id="display-area">READY</div>
    </div>

    <div class="controls">
        <select id="region-select">
            <option value="all">🌏 전체 연회</option>
        </select>
        <div style="display:flex; align-items:center; justify-content:center; background:#f1f2f6; border-radius:12px; font-weight:bold; font-size:14px;">
            잔여: <span id="count" style="margin-left:8px; color:var(--primary);">0</span>명
        </div>
        <button id="draw-btn" onclick="startDraw()" disabled>START DRAW</button>
    </div>

    <div class="winner-log" id="winners"></div>
</div>

<script>
    let participants = [];
    const displayArea = document.getElementById('display-area');
    const drawBtn = document.getElementById('draw-btn');

    async function fetchSheetData() {
        const fixedSheetId = "1vQU883J4q58v4Foy0G5rQktJPW5EKxNs5P8KxA5Xa2YpaiDJorZ3RkBofIPoln0Vx1DIjGL5tnpt7";
        const rawInput = document.getElementById('sheet-id').value.trim();
        let targetId = rawInput || fixedSheetId;
        let sheetId = targetId;
        const match = targetId.match(/\/d\/([^/]+)/);
        if (match) sheetId = match[1];

        const url = `https://docs.google.com/spreadsheets/d/${sheetId}/export?format=csv`;

        try {
            const response = await fetch(url);
            if (!response.ok) throw new Error();
            const data = await response.text();
            const rows = data.split('\n').map(row => row.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/).map(cell => cell.replace(/"/g, '').trim()));
            const headers = rows[0];
            const nameIdx = headers.findIndex(h => h.includes('성함') || h.includes('이름') || h.toLowerCase().includes('name'));
            const regionIdx = headers.findIndex(h => h.includes('지역') || h.includes('연회') || h.toLowerCase().includes('region'));

            participants = rows.slice(1).filter(row => row[nameIdx]).map(row => ({
                name: row[nameIdx],
                region: regionIdx !== -1 ? (row[regionIdx] || '기타') : '전체'
            }));

            if(participants.length > 0) {
                initRegions();
                updateDisplay();
                drawBtn.disabled = false;
                displayArea.innerText = "GO!";
                alert(`성공! 명단을 불러왔습니다.`);
            }
        } catch (error) {
            alert("연동 실패! 시트 게시 상태를 확인하세요.");
        }
    }

    function initRegions() {
        const regions = [...new Set(participants.map(p => p.region))];
        const select = document.getElementById('region-select');
        select.innerHTML = '<option value="all">🌏 전체 연회</option>';
        regions.forEach(r => {
            const opt = document.createElement('option');
            opt.value = r; opt.innerText = r;
            select.appendChild(opt);
        });
    }

    function updateDisplay() {
        const region = document.getElementById('region-select').value;
        const pool = participants.filter(p => region === 'all' || p.region === region);
        document.getElementById('count').innerText = pool.length;
    }

    document.getElementById('region-select').onchange = updateDisplay;

    function celebration() {
        confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 }, zIndex: 1000 });
        setTimeout(() => {
            confetti({ particleCount: 100, spread: 100, origin: { x: 0.3, y: 0.7 } });
            confetti({ particleCount: 100, spread: 100, origin: { x: 0.7, y: 0.7 } });
        }, 300);
    }

    function startDraw() {
        const region = document.getElementById('region-select').value;
        const pool = participants.filter(p => region === 'all' || p.region === region);
        if (pool.length === 0) return alert("대상자가 없습니다.");

        drawBtn.disabled = true;
        displayArea.classList.remove('winner-pop');
        displayArea.classList.add('shake');

        let duration = 2000;
        let startTime = Date.now();

        const rolling = setInterval(() => {
            const elapsed = Date.now() - startTime;
            const randomPerson = pool[Math.floor(Math.random() * pool.length)];
            displayArea.innerText = randomPerson.name;

            if (elapsed >= duration) {
                clearInterval(rolling);
                displayArea.classList.remove('shake');
                finalize(pool);
            }
        }, 60);
    }

    function finalize(pool) {
        const winner = pool[Math.floor(Math.random() * pool.length)];
        participants = participants.filter(p => p !== winner);
       
        // 3. 당첨자 문구 수정 부분 (축하합니다! 문구 추가)
        displayArea.innerText = `축하합니다!\n${winner.name}님!`;
        displayArea.style.whiteSpace = "pre-line"; // 줄바꿈 적용
        displayArea.classList.add('winner-pop');
       
        celebration();

        const log = document.createElement('div');
        log.className = 'winner-item';
        log.innerHTML = `<span><strong>${winner.name}</strong></span> <span>${winner.region}</span>`;
        document.getElementById('winners').prepend(log);

        updateDisplay();
        drawBtn.disabled = false;
    }
</script>
</body>
</html>
