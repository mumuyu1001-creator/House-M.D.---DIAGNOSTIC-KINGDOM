[house.html](https://github.com/user-attachments/files/24726103/house.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HOUSE M.D. - CLINIC DATABASE</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@500;900&display=swap" rel="stylesheet">
    <style>
        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background-image: url('https://i.redd.it/qvqqhsd1fci71.png');
            background-size: cover; background-position: center; background-attachment: fixed;
            margin: 0; overflow-x: hidden;
        }
        body::before {
            content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(255, 255, 255, 0.2); backdrop-filter: blur(10px); z-index: -1;
        }
        .module-card { 
            transition: all 0.2s ease; cursor: pointer; 
            background: rgba(255, 255, 255, 0.9); border-radius: 12px; 
            border: 2px solid #1a1a1a; box-shadow: 4px 4px 0px 0px #1a1a1a;
        }
        .module-card:hover { transform: translate(-2px, -2px); box-shadow: 6px 6px 0px 0px #1a1a1a; }
        
        /* 色塊頭像 */
        .abstract-avatar { width: 60px; height: 60px; border-radius: 8px; overflow: hidden; border: 2px solid #1a1a1a; flex-shrink: 0; }
        
        /* 門診室遮罩 */
        #clinic-overlay {
            display: none; position: fixed; inset: 0; background: rgba(0, 0, 0, 0.95);
            backdrop-filter: blur(15px); z-index: 1000; flex-direction: column;
        }
        #game-arena { position: relative; width: 100%; height: 60vh; }
        .mini-patient { position: absolute; cursor: pointer; transition: transform 0.2s; }
        .mini-patient:active { transform: scale(0.9); }

        .timer-track { width: 100%; height: 15px; background: #222; }
        #timer-bar { width: 100%; height: 100%; background: #ff4757; }
    </style>
</head>
<body class="p-4 pb-64">

    <div id="clinic-overlay">
        <div class="timer-track"><div id="timer-bar"></div></div>
        <div class="text-center mt-10">
            <h2 id="order-hint" class="text-yellow-400 text-5xl font-black mb-4 uppercase italic"></h2>
            <p id="game-status" class="text-white font-bold text-lg italic tracking-widest"></p>
        </div>
        <div id="game-arena"></div>
    </div>

    <div class="max-w-6xl mx-auto">
        <header class="mb-8 flex justify-between items-center">
            <div class="bg-black text-white px-4 py-2 italic font-black text-2xl">PPTH DATABASE</div>
            <div class="text-right">
                <p class="font-bold text-slate-800">Department of Diagnostic Medicine</p>
                <p class="text-xs font-bold text-blue-600 uppercase">Case Load: 15 Patients</p>
            </div>
        </header>

        <div id="grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"></div>
    </div>

    <div class="fixed bottom-0 left-0 w-full p-6 flex items-end space-x-4 bg-white/80 backdrop-blur-md border-t-4 border-black z-50">
        <div id="dialogue-box" class="flex-1 bg-white p-5 h-32 overflow-y-auto border-l-8 border-blue-600 shadow-inner">
            <span id="speaker-name" class="font-black text-blue-600 text-xl uppercase tracking-tighter">DR. HOUSE:</span><br>
            <p id="main-message" class="text-slate-800 font-bold mt-1 text-lg italic">「病歷都在這了。選一個，看看誰在撒謊。」</p>
        </div>
        <div class="flex flex-col items-center">
            <div class="abstract-avatar" style="width:80px; height:80px;">
                <div id="av-top" class="h-[40%] bg-slate-700"></div>
                <div id="av-bottom" class="h-[60%] bg-slate-400"></div>
            </div>
            <span class="font-black text-[10px] mt-1 uppercase">Gregory House</span>
        </div>
    </div>

    <script>
        const patientsData = [
            { name: "白雪", age: 18, symptom: "食用不明蘋果後陷入深度昏睡，呼吸緩慢。", top: "#FFD700", bottom: "#0056b3", cases: [{d:"機磷中毒", h:"蘋果裡的農藥。"}, {d:"清潔障礙", h:"闖入民宅打掃？"}] },
            { name: "貝兒", age: 24, symptom: "長期與野獸同居，出現幻聽與情感依附。", top: "#FDE047", bottom: "#EAB308", cases: [{d:"斯德哥爾摩", h:"典型的心理補償。"}, {d:"麥角鹼中毒", h:"看見家具會說話？"}] },
            { name: "木蘭", age: 20, symptom: "退伍後出現幻視與嚴重的身份認知偏差。", top: "#EF4444", bottom: "#1E293B", cases: [{d:"PTSD", h:"戰場壓力的幻覺。"}, {d:"鉛中毒", h:"盔甲太劣質。"}] },
            { name: "仙杜瑞拉", age: 19, symptom: "雙腳劇痛且呼吸困難，長期吸入大量粉塵。", top: "#BAE6FD", bottom: "#38BDF8", cases: [{d:"矽肺症", h:"煙囪灰吸太多。"}, {d:"睡眠剝奪", h:"大腦強制關機。"}] },
            { name: "奧蘿拉", age: 16, symptom: "受傷後陷入長期昏迷，對外界刺激無反應。", top: "#F9A8D4", bottom: "#F472B6", cases: [{d:"KLS", h:"腦幹功能紊亂。"}, {d:"敗血性休克", h:"針頭有毒。"}] },
            { name: "黑魔女", age: 45, symptom: "情緒極端不穩定，頭部有巨大的異常骨質增生。", top: "#1a2b3c", bottom: "#6a0572", cases: [{d:"巨大皮角", h:"這角得鋸了。"}, {d:"人格障礙", h:"太過邊緣。"}] },
            { name: "烏蘇拉", age: 50, symptom: "極度肥胖，且伴隨皮膚病變與消化道寄生蟲。", top: "#551a8b", bottom: "#000000", cases: [{d:"代謝症候群", h:"心臟快撐不住。"}, {d:"寄生胎", h:"那是雙胞胎。"}] },
            { name: "壞皇后", age: 40, symptom: "長期攝取化學藥劑，對鏡子有病態幻覺。", top: "#7b1c1c", bottom: "#000000", cases: [{d:"軀體變形", h:"病態外表執著。"}, {d:"精神分裂", h:"跟牆壁對話？"}] },
            { name: "賈方", age: 48, symptom: "四肢異常纖長，伴隨嚴重的權力妄想症。", top: "#303030", bottom: "#8b0000", cases: [{d:"馬凡氏症", h:"手指長是缺陷。"}, {d:"自大妄想", h:"他想當精靈。"}] },
            { name: "寶嘉康蒂", age: 18, symptom: "聲稱能與自然對話，心跳過緩且體溫偏低。", top: "#D2B48C", bottom: "#8B4513", cases: [{d:"顳葉受損", h:"妳聽見風的聲音？"}, {d:"低體溫症", h:"別再跳河了。"}] },
            { name: "蒂安納", age: 22, symptom: "急性胃腸炎，曾與兩棲類動物近距離接觸。", top: "#BEF264", bottom: "#166534", cases: [{d:"沙門氏菌", h:"別跟青蛙接吻。"}, {d:"慢性疲勞", h:"妳太累了。"}] },
            { name: "莫阿娜", age: 16, symptom: "大面積曝曬傷，電解質失衡且幻視海洋。", top: "#EF4444", bottom: "#F97316", cases: [{d:"海洋弧菌", h:"傷口感染了。"}, {d:"脫水", h:"妳在大海漂太久。"}] },
            { name: "茉莉", age: 20, symptom: "胸悶、咳嗽，且與大型掠食動物親密接觸。", top: "#2DD4BF", bottom: "#134E4A", cases: [{d:"弓形蟲", h:"老虎身上有寄生蟲。"}, {d:"缺鐵性貧血", h:"臉色發青。"}] },
            { name: "拉雅", age: 25, symptom: "肌肉痙攣，身上有陳舊刀傷，極度不信任人。", top: "#1E293B", bottom: "#713F12", cases: [{d:"破傷風", h:"被古劍砍傷。"}, {d:"信任缺失", h:"這不是末日是病。"}] },
            { name: "梅利達", age: 17, symptom: "肩關節習慣性脫臼，家族史有集體中毒。", top: "#EA580C", bottom: "#1E3A8A", cases: [{d:"關節磨損", h:"每天射箭的後果。"}, {d:"紅髮症候群", h:"對麻醉沒反應。"}] }
        ];

        const team = [
            { n: "CAMERON", t: "#10b981", m: "卡麥蓉幫你接走了一個！" },
            { n: "CHASE", t: "#3b82f6", m: "柴絲替你簽了報告。" },
            { n: "FOREMAN", t: "#4b5563", m: "佛曼默默處理了一個。" },
            { n: "WILSON", t: "#f43f5e", m: "威爾森清空了全部！" }
        ];

        let gameTimer;
        let targetOrder = [];
        let currentStep = 0;

        function setUI(n, c, m) {
            document.getElementById('speaker-name').innerText = `DR. ${n}:`;
            document.getElementById('speaker-name').style.color = c;
            document.getElementById('main-message').innerHTML = m;
        }

        // --- 門診小遊戲 ---
        function startClinicGame() {
            const overlay = document.getElementById('clinic-overlay');
            const arena = document.getElementById('game-arena');
            const timerBar = document.getElementById('timer-bar');
            overlay.style.display = 'flex';
            arena.innerHTML = '';
            currentStep = 0;

            const shuffle = [...patientsData].sort(() => 0.5 - Math.random()).slice(0, 3);
            targetOrder = shuffle;
            document.getElementById('order-hint').innerText = targetOrder.map(p => p.name).join(" > ");
            
            // 隨機救援
            let statusText = "科蒂在盯著你...";
            if(Math.random() < 0.40) {
                const helper = team[Math.floor(Math.random() * team.length)];
                statusText = helper.m;
                if(helper.n === "WILSON") { setTimeout(winGame, 800); } else { currentStep++; }
            }
            document.getElementById('game-status').innerText = statusText;

            // 渲染純色塊頭像（亂數散佈）
            shuffle.forEach(p => {
                const pDiv = document.createElement('div');
                pDiv.className = "mini-patient";
                pDiv.style.left = (Math.random() * 80 + 5) + "%";
                pDiv.style.top = (Math.random() * 70 + 5) + "%";
                pDiv.innerHTML = `
                    <div class="abstract-avatar" style="width:70px; height:70px; border:4px solid white; background:white;">
                        <div class="h-[40%]" style="background:${p.top}"></div>
                        <div class="h-[60%]" style="background:${p.bottom}"></div>
                    </div>
                    <div class="text-white text-xs font-black text-center mt-1 bg-black/50">${p.name}</div>
                `;
                pDiv.onclick = () => {
                    if (p.name === targetOrder[currentStep].name) {
                        pDiv.style.opacity = '0.2';
                        pDiv.style.pointerEvents = 'none';
                        currentStep++;
                        if (currentStep >= 3) winGame();
                    }
                };
                arena.appendChild(pDiv);
            });

            timerBar.style.transition = 'none';
            timerBar.style.width = '100%';
            setTimeout(() => {
                timerBar.style.transition = 'width 5s linear';
                timerBar.style.width = '0%';
            }, 50);

            gameTimer = setTimeout(loseGame, 5000);
        }

        function winGame() {
            clearTimeout(gameTimer);
            document.getElementById('clinic-overlay').style.display = 'none';
            setUI('HOUSE', '#2563eb', "「門診結束。雖然過程很蠢，但至少車位保住了。」");
        }

        function loseGame() {
            document.getElementById('clinic-overlay').style.display = 'none';
            setUI('CUDDY', '#dc2626', "「豪斯！你太慢了！我要把你的車位移到 E 區！」");
        }

        // --- 渲染主畫面列表 ---
        const grid = document.getElementById('grid');
        patientsData.forEach(p => {
            const card = document.createElement('div');
            card.className = "module-card p-4 flex items-center space-x-4";
            card.innerHTML = `
                <div class="abstract-avatar">
                    <div class="h-[40%]" style="background:${p.top}"></div>
                    <div class="h-[60%]" style="background:${p.bottom}"></div>
                </div>
                <div class="flex-1 overflow-hidden">
                    <div class="flex justify-between items-center">
                        <span class="font-black text-slate-900">${p.name}</span>
                        <span class="text-[10px] font-bold bg-black text-white px-1">AGE: ${p.age}</span>
                    </div>
                    <p class="text-[11px] font-bold text-slate-500 mt-1 leading-tight truncate">
                        <span class="text-red-600">主訴:</span> ${p.symptom}
                    </p>
                </div>
            `;
            card.onclick = () => {
                if (Math.random() < 0.12) {
                    startClinicGame();
                } else {
                    const c = p.cases[Math.floor(Math.random() * p.cases.length)];
                    setUI('HOUSE', '#2563eb', `<span class="text-red-600 font-black">[最終診斷: ${c.d}]</span><br>「${c.h}」`);
                }
            };
            grid.appendChild(card);
        });
    </script>
</body>
</html>
