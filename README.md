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
            --skyblue: #a2e4f6; /* 하늘색 변수 */
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

        /* 중앙 전광판: 하늘색 배경 적용 */
        #display-box {
            height: 180px; margin: 20px 0; border: 8px solid var(--dark);
            border-radius: 25px; display: flex; align-items: center; justify-content: center;
            background: var(--skyblue);
            position: relative; overflow: hidden;
        }
        #display-area { font-size: 3.5rem; font-weight: 900; color: var(--dark); transition: all 0.1s; text-align: center; line-height: 1.2; }
       
        .shake { animation: shake 0.1s infinite; }
        @keyframes shake {
            0% { transform: translate(2px, 2px); }
            50% { transform: translate(-2px, -2px); }
            100% { transform: translate(2px, -2px); }
        }
       
        /* 당첨자 문구 팝업 효과 */
        .winner-pop {
            animation: pop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            color: var(--primary) !important;
            font-size: 2.5rem !important;
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

