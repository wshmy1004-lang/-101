<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>제20회 전국 교사대회 추첨기</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root { --primary: #ff4757; --dark: #2f3542; --skyblue: #a2e4f6; }
        body { font-family: sans-serif; background: var(--dark); display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .container { background: white; padding: 30px; border-radius: 25px; width: 90%; max-width: 500px; text-align: center; }
       
        /* 하늘색 전광판 */
        #display-box {
            height: 160px; margin: 20px 0; border: 6px solid var(--dark);
            border-radius: 20px; display: flex; align-items: center; justify-content: center;
            background: var(--skyblue); overflow: hidden;
        }
        #display-area { font-size: 3rem; font-weight: bold; color: var(--dark); white-space: pre-line; }
       
        .sync-section { margin-bottom: 20px; display: flex; gap: 5px; }
        #sheet-id { flex: 1; padding: 10px; border-radius: 10px; border: 1px solid #ccc; font-size: 12px; }
        #sync-btn { background: #57606f; color: white; border: none; padding: 10px 15px; border-radius: 10px; cursor: pointer; font-weight: bold; }
       
        .controls { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        #draw-btn { background: var(--primary); color: white; border: none; padding: 15px; border-radius: 12px; font-size: 1.2rem; font-weight: bold; grid-column: span 2; cursor: pointer; }
        #draw-btn:disabled { background: #ccc; }
       
        .winner-pop { animation: pop 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275); color: var(--primary) !important; font-size: 2.2rem !important; }
        @keyframes pop { 0% { transform: scale(0.6); } 100% { transform: scale(1); } }
        .shake { animation: shake 0.1s infinite; }
        @keyframes shake { 0% { transform: translate(1px, 1px); } 50% { transform: translate(-1px, -1px); } 100% { transform: translate(1px, -1px); } }
    </style>
</head>
<body>

<div class="container">
    <h3 style="margin-top:0;">🏆 제20회 전국 교사대회 추첨</h3>
   
    <div class="sync-section">
        <input type="text" id="sheet-id" placeholder="시트 주소를 입력하세요">
        <button id="sync-btn" onclick="fetchSheetData()">데이터 연동</button>
    </div>

    <div id="display-box">
        <div id="display-area">READY</div>
    </div>

    <div class="controls">
        <select id="region-select" onchange="updateDisplay()">
            <option value="all">🌏 전체 연회</option>
        </select>
        <div style="background:#f1f2f6; border-radius:10px; display:flex; align-items:center; justify-content:center; font-weight:bold;">
            잔여: <span id="count" style="color:var(--primary); margin-left:5px;">0</span>명
        </div>
        <button id="draw-btn" onclick="startDraw()" disabled>START DRAW</button>
    </div>
</div>

<script>
    let participants = [];
    const displayArea = document.getElementById('display-area');
    const drawBtn = document.getElementById('draw-btn');

    async function fetchSheetData() {
        // [사진에서 확인된 최신 시트 ID 고정]
        const fixedId = "16ytZT6dp033IWPDXSz05rvlyH5BXQnEoD1mTeBihHSg";
        const inputId = document.getElementById('sheet-id').value.trim();
        let sheetId = inputId || fixedId;

        // ID 추출 로직 강화
        if (sheetId.includes('/d/')) {
            sheetId = sheetId.split('/d/')[1].split('/')[0];
        }

        // CSV URL 생성 (gid=0은 첫 번째 탭을 의미함)
        const url = `https://docs.google.com/spreadsheets/d/${sheetId}/export?format=csv&gid=0`;

        try {
            const response = await fetch(url);
            if (!response.ok) throw new Error();
           
            const csvText = await response.text();
            // 줄바꿈 및 쉼표 파싱
            const rows = csvText.split('\n').map(r => r.split(',').map(c => c.trim().replace(/"/g, '')));
           
            const headers = rows[0];
            // '성함', '이름', '성명' 중 검색
            const nIdx = headers.findIndex(h => h.includes('성함') || h.includes('이름') || h.includes('성명'));
            // '연회', '지역' 중 검색
            const rIdx = headers.findIndex(h => h.includes('연회') || h.includes('지역'));

            if (nIdx === -1) {
                alert("시트 첫 줄에 '성함' 또는 '이름' 제목이 없습니다!");
                return;
            }

            participants = rows.slice(1)
                .filter(row => row[nIdx] && row[nIdx].length > 0)
                .map(row => ({
                    name: row[nIdx],
                    region: rIdx !== -1 ? (row[rIdx] || '기타') : '전체'
                }));

            if (participants.length > 0) {
                initRegions();
                updateDisplay();
                drawBtn.disabled = false;
                displayArea.innerText = "LOADED!";
                alert(participants.length + "명의 명단을 가져왔습니다!");
            } else {
                alert("불러올 이름 데이터가 없습니다.");
            }
        } catch (e) {
            alert("연동 실패! \n1. 구글 시트의 [웹에 게시] 확인 \n2. 시트 주소 확인");
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
        const reg = document.getElementById('region-select').value;
        const pool = participants.filter(p => reg === 'all' || p.region === reg);
        document.getElementById('count').innerText = pool.length;
    }

    function startDraw() {
        const reg = document.getElementById('region-select').value;
        const pool = participants.filter(p => reg === 'all' || p.region === reg);
        if (pool.length === 0) return;

        drawBtn.disabled = true;
        displayArea.classList.remove('winner-pop');
        displayArea.classList.add('shake');

        let count = 0;
        const timer = setInterval(() => {
            const rand = pool[Math.floor(Math.random() * pool.length)];
            displayArea.innerText = rand.name;
            if (count++ > 20) {
                clearInterval(timer);
                displayArea.classList.remove('shake');
                const winner = pool[Math.floor(Math.random() * pool.length)];
                // 당첨 문구 수정
                displayArea.innerText = `축하합니다!\n${winner.name}님!`;
                displayArea.classList.add('winner-pop');
                confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 } });
                participants = participants.filter(p => p !== winner);
                updateDisplay();
                drawBtn.disabled = pool.length <= 1;
            }
        }, 80);
    }
</script>

</body>
</html>
