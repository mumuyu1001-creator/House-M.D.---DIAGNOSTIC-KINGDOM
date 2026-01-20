<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>HOUSE M.D. MOBILE</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@500;900&display=swap" rel="stylesheet">
    <style>
        body { 
            font-family: 'Noto Sans TC', sans-serif; background: #cbd5e1;
            background-image: url('https://i.redd.it/qvqqhsd1fci71.png');
            background-size: cover; background-position: center; background-attachment: fixed;
            margin: 0; overflow: hidden; /* 防止主頁面隨意晃動 */
        }
        body::before { content: ""; position: fixed; inset: 0; background: rgba(255, 255, 255, 0.4); backdrop-filter: blur(12px); z-index: -1; }
        
        /* 滾動容器 */
        .scroll-container { height: 100vh; overflow-y: auto; padding-bottom: 160px; -webkit-overflow-scrolling: touch; }

        .module-card { 
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1); cursor: pointer; 
            background: rgba(255, 255, 255, 0.95); border-radius: 8px; 
            border: 2px solid #1e293b; box-shadow: 3px 3px 0px 0px #1e293b;
            height: 80px; padding: 10px;
            display: flex; align-items: center; gap: 10px;
        }
        
        .abstract-avatar { 
            width: 48px; height: 48px; border-radius: 50%; overflow: hidden; 
            border: 2px solid #1e293b; flex-shrink: 0; position: relative;
        }
        .avatar-top { height: 45%; width: 100%; }
        .avatar-bottom { height: 55%; width: 100%; }

        /* 豪斯頭像 */
        .house-face { background: #64748b; height: 45%; } 
        .house-suit { background: #1e293b; height: 55%; } 
        .house-eyes { position: absolute; top: 35%; left: 0; width: 100%; height: 3px; background: rgba(0,0,0,0.2); }

        .pill-needed { animation: vibrate 0.2s infinite; border-color: #ef4444 !important; background: #fee2e2 !important; }
        @keyframes vibrate { 0% { transform: translate(0); } 25% { transform: translate(2px, -1px); } 75% { transform: translate(-2px, 1px); } 100% { transform: translate(0); } }
        
        #clinic-overlay { display: none; position: fixed; inset: 0; background: rgba(15, 23, 42, 0.98); z-index: 1000; flex-direction: column; align-items: center; justify-content: center; }
        #game-arena { position: relative; width: 100%; height: 60vh; }
        #timer-bar { position: fixed; top: 0; left: 0; width: 100%; height: 8px; background: #fbbf24; }
    </style>
</head>
<body>

    <div id="clinic-overlay">
        <div id="timer-bar"></div>
        <div class="text-center px-4 mb-4">
            <h3 id="order-hint" class="text-yellow-400 text-xl font-black italic mb-2"></h3>
            <p id="game-status" class="text-white text-xs font-bold italic bg-slate-800/80 px-4 py-1 rounded-full inline-block"></p>
        </div>
        <div id="game-arena"></div>
    </div>

    <div class="scroll-container p-3">
        <header class="mb-4 flex justify-between items-center bg-white/95 p-3 rounded-lg border-b-2 border-slate-900 shadow-lg">
            <div>
                <h1 class="font-black text-xl italic uppercase text-slate-900 leading-none">PPTH APP</h1>
                <p id="leg-status" class="text-[8px] font-bold text-blue-600 uppercase tracking-widest italic">Pain: OK</p>
            </div>
            <div class="text-right">
                <p id="case-counter" class="font-black text-2xl italic text-slate-900">0/0</p>
            </div>
        </header>

        <div id="grid" class="grid grid-cols-1 gap-3"></div>
    </div>

    <div class="fixed bottom-0 left-0 w-full p-2 flex items-end space-x-2 bg-white/95 backdrop-blur-xl border-t border-slate-900 z-50">
        <div id="dialogue-box" class="flex-1 bg-slate-100 p-3 h-28 overflow-y-auto border-l-4 border-blue-600 rounded-r shadow-inner">
            <span id="speaker-name" class="font-black text-blue-700 text-sm italic uppercase">G. House, M.D.</span><br>
            <p id="main-message" class="text-slate-800 font-bold mt-1 text-sm italic leading-tight">「App 版上線了。別把手機摔了，那不屬於保險範圍。」</p>
        </div>

        <div class="flex flex-col items-center">
            <div onclick="takePill()" id="pill-bottle" class="w-12 h-6 bg-slate-100 border-2 border-slate-800 rounded-full relative flex items-center justify-center cursor-pointer active:scale-90 transition-all">
                <div class="absolute top-0 left-1/2 -translate-x-1/2 w-[1px] h-full bg-slate-300"></div>
                <span id="pill-text" class="z-10 font-black text-[8px] text-slate-900 uppercase">VIC 0/2</span>
            </div>
            <span class="text-[7px] font-black mt-1 text-slate-400">MEDS</span>
        </div>

        <div class="shrink-0">
            <div class="abstract-avatar" style="width: 50px; height: 50px; border-width: 2px;">
                <div class="house-face"></div>
                <div class="house-suit"></div>
                <div class="house-eyes"></div>
            </div>
        </div>
    </div>

    <script>
        // 數據與邏輯同前 (修正了莫阿娜斷行問題)
        const patientsData = [
            { name: "白雪", age: 18, symptom: "食用不明蘋果後深度昏睡。", top: "#FFD700", bottom: "#0056b3", cases: [{d:"有機磷中毒", h:"農藥中毒，需要阿托品。"}, {d:"清潔障礙", h:"強迫打掃是神經受損。"}] },
            { name: "貝兒", age: 24, symptom: "長期與野獸同居，出現幻聽。", top: "#FDE047", bottom: "#EAB308", cases: [{d:"麥角鹼中毒", h:"家具會說話是幻覺。"}, {d:"斯德哥爾摩", h:"典型的心理補償。"}] },
            { name: "木蘭", age: 20, symptom: "身份認知偏差與幻視。", top: "#EF4444", bottom: "#1E293B", cases: [{d:"PTSD", h:"戰場壓力的視覺體現。"}, {d:"鉛中毒", h:"盔甲鉛超標。"}] },
            { name: "仙杜瑞拉", age: 19, symptom: "雙腳劇痛，長期吸入粉塵。", top: "#BAE6FD", bottom: "#38BDF8", cases: [{d:"矽肺症", h:"肺纖維化。"}, {d:"細菌感染", h:"玻璃鞋磨破了腳。"}] },
            { name: "奧蘿拉", age: 16, symptom: "受傷後陷入長期昏迷。", top: "#F9A8D4", bottom: "#F472B6", cases: [{d:"KLS", h:"腦幹功能紊亂。"}, {d:"敗血性休克", h:"針頭不潔。"}] },
            { name: "黑魔女", age: 45, symptom: "情緒不穩，頭部異常增生。", top: "#1a2b3c", bottom: "#6a0572", cases: [{d:"巨大皮角", h:"這角壓迫到神經了。"}, {d:"人格障礙", h:"腎上腺素失控。"}] },
            { name: "烏蘇拉", age: 50, symptom: "極度肥胖與消化道寄生蟲。", top: "#551a8b", bottom: "#000000", cases: [{d:"代謝症候群", h:"心臟快撐不住了。"}, {d:"多肢畸形", h:"觸手是發育異常。"}] },
            { name: "壞皇后", age: 40, symptom: "長期攝取藥劑，有病態幻覺。", top: "#7b1c1c", bottom: "#000000", cases: [{d:"軀體變形", h:"對外貌的病態執著。"}, {d:"汞中毒", h:"美白霜害的。"}] },
            { name: "賈方", age: 48, symptom: "四肢纖長，權力妄想。", top: "#303030", bottom: "#8b0000", cases: [{d:"馬凡氏症", h:"主動脈隨時會爆。"}, {d:"香料中毒", h:"腦血管炎。"}] },
            { name: "寶嘉康蒂", age: 18, symptom: "與自然對話，心跳過緩。", top: "#D2B48C", bottom: "#8B4513", cases: [{d:"顳葉受損", h:"聽見風聲是大腦放電。"}, {d:"低體溫症", h:"別跳河了。"}] },
            { name: "蒂安納", age: 22, symptom: "急性胃腸炎，接觸兩棲類。", top: "#BEF264", bottom: "#166534", cases: [{d:"沙門氏菌", h:"別跟青蛙接吻。"}, {d:"慢性疲勞", h:"免疫系統崩潰。"}] },
            { name: "莫阿娜", age: 16, symptom: "曝曬傷，幻視海洋。", top: "#EF4444", bottom: "#F97316", cases: [{d:"海洋弧菌", h:"細菌吞噬傷口。"}, {d:"熱衰竭", h:"脫水導致神智不清。"}] },
            { name: "艾霞", age: 17, symptom: "視覺先兆與聽覺幻覺。", top: "#a855f7", bottom: "#581c87", cases: [{d:"顳葉癲癇", h:"氣泡是視覺先兆。"}, {d:"聯覺症", h:"神經迴路混亂。"}] },
            { name: "茉莉", age: 20, symptom: "胸悶、與猛獸親密接觸。", top: "#2DD4BF", bottom: "#134E4A", cases: [{d:"弓形蟲", h:"老虎寄生蟲入肺。"}, {d:"缺鐵性貧血", h:"飛毯是低血氧。"}] },
            { name: "拉雅", age: 25, symptom: "肌肉痙攣，不信任人。", top: "#1E293B", bottom: "#713F12", cases: [{d:"破傷風", h:"舊傷口感染。"}, {d:"心理創傷", h:"防禦心過強。"}] },
            { name: "梅利達", age: 17, symptom: "肩關節脫臼，家史中毒。", top: "#EA580C", bottom: "#1E3A8A", cases: [{d:"關節磨損", h:"射箭職業病。"}, {d:"普利昂感染", h:"變熊是吃了腦組織。"}] },
            { name: "安娜", age: 18, symptom: "頭撞後隨處入睡。", top: "#1e3a8a", bottom: "#0ea5e9", cases: [{d:"腦損傷", h:"前額葉失控。"}, {d:"發作性睡病", h:"睡眠斷電。"}] },
            { name: "船長", age: 45, symptom: "幻肢痛，恐懼滴答聲。", top: "#991b1b", bottom: "#1e293b", cases: [{d:"幻肢痛", h:"大腦想念那隻手。"}, {d:"PTSD", h:"心理觸發點。"}] },
            { name: "庫伊拉", age: 55, symptom: "色素異常，聽力受損。", top: "#f8fafc", bottom: "#020617", cases: [{d:"瓦登伯革", h:"遺傳性色素異常。"}, {d:"化學中毒", h:"染劑毀了情緒。"}] },
            { name: "野獸", age: 21, symptom: "多毛、暴怒。", top: "#1e3a8a", bottom: "#fbbf24", cases: [{d:"多發性腺瘤", h:"腫瘤導致體型失控。"}, {d:"狼人症", h:"罕見基因表現。"}] }
        ];

        const team = [
            { n: "FOREMAN", m: "佛曼不屑地幫了你。", c: "#6366f1", p: 1 },
            { n: "CAMERON", m: "卡麥蓉接走一個病人。", c: "#10b981", p: 1 },
            { n: "CHASE", m: "柴斯搞定了報告。", c: "#f59e0b", p: 1 },
            { n: "WILSON", m: "威爾森清空了門診！", c: "#ec4899", p: 3 }
        ];

        let caseCount = 0; let nextPillAt = 4; let pillNeeded = false; let pillClicks = 0;
        let clinicOrder = []; let clinicStep = 0; let clinicTimer;

        function init() {
            const grid = document.getElementById('grid');
            patientsData.forEach(p => {
                const card = document.createElement('div');
                card.className = "module-card";
                card.innerHTML = `
                    <div class="abstract-avatar" style="background:${p.bottom}"><div class="avatar-top" style="background:${p.top}"></div></div>
                    <div class="flex-1 min-w-0">
                        <div class="flex items-baseline gap-2"><span class="font-black text-xs text-slate-900">${p.name}</span><span class="text-[8px] font-bold text-blue-600 bg-blue-50 px-1 rounded">${p.age}歲</span></div>
                        <p class="text-[9px] font-medium text-slate-500 line-clamp-2 mt-1">${p.symptom}</p>
                    </div>
                `;
                card.onclick = () => {
                    if (pillNeeded) { setMsg("HOUSE", "#ef4444", "「先吃藥！別廢話！」"); return; }
                    if (Math.random() < 0.2) { startClinic(); return; } 
                    const c = p.cases[Math.floor(Math.random() * p.cases.length)];
                    setMsg("G. HOUSE, M.D.", "#2563eb", `[${c.d}] ${c.h}`);
                    caseCount++; if (caseCount >= nextPillAt) triggerPill();
                    updateUI();
                };
                grid.appendChild(card);
            });
            updateUI();
        }

        function triggerPill() { 
            pillNeeded = true; document.getElementById('pill-bottle').classList.add('pill-needed');
            document.getElementById('leg-status').innerText = "Pain: CRITICAL";
            document.getElementById('leg-status').classList.replace('text-blue-600', 'text-red-600');
        }

        function takePill() {
            if (!pillNeeded) return;
            pillClicks++; document.getElementById('pill-text').innerText = `VIC ${pillClicks}/2`;
            if (pillClicks >= 2) {
                pillNeeded = false; pillClicks = 0; caseCount = 0; nextPillAt = 3 + Math.floor(Math.random()*2);
                document.getElementById('pill-bottle').classList.remove('pill-needed');
                document.getElementById('pill-text').innerText = `VIC 0/2`;
                document.getElementById('leg-status').innerText = "Pain: OK";
                document.getElementById('leg-status').classList.replace('text-red-600', 'text-blue-600');
                setMsg("HOUSE", "#2563eb", "「...清醒多了。」"); updateUI();
            }
        }

        function startClinic() {
            const overlay = document.getElementById('clinic-overlay');
            overlay.style.display = 'flex'; clinicStep = 0;
            const arena = document.getElementById('game-arena'); arena.innerHTML = '';
            const shuffle = [...patientsData].sort(() => 0.5 - Math.random()).slice(0, 3);
            clinicOrder = shuffle;
            document.getElementById('order-hint').innerText = shuffle.map(p => p.name).join(" > ");
            const helper = Math.random() < 0.4 ? team[Math.floor(Math.random()*team.length)] : null;
            document.getElementById('game-status').innerText = helper ? helper.m : "Cuddy: 5s!";
            if (helper && helper.n === "WILSON") { setTimeout(() => endClinic(true), 1000); return; }
            shuffle.forEach((p, idx) => {
                const pDiv = document.createElement('div'); pDiv.className = "absolute text-center";
                pDiv.style.left = (15 + Math.random()*60)+"%"; pDiv.style.top = (15 + Math.random()*60)+"%";
                pDiv.innerHTML = `<div class="abstract-avatar" style="width:60px; height:60px; background:${p.bottom}"><div class="avatar-top" style="background:${p.top}"></div></div><div class="text-white text-[10px] font-bold">${p.name}</div>`;
                if (helper && helper.p === 1 && idx === 0) { pDiv.style.opacity = '0.3'; clinicStep = 1; }
                pDiv.onclick = () => { if (p.name === clinicOrder[clinicStep].name) { pDiv.style.opacity = '0.2'; clinicStep++; if (clinicStep >= 3) endClinic(true); } };
                arena.appendChild(pDiv);
            });
            const tb = document.getElementById('timer-bar'); tb.style.transition = 'none'; tb.style.width = '100%';
            setTimeout(() => { tb.style.transition = 'width 5s linear'; tb.style.width = '0%'; }, 50);
            clinicTimer = setTimeout(() => endClinic(false), 5000);
        }

        function endClinic(win) {
            clearTimeout(clinicTimer); document.getElementById('clinic-overlay').style.display = 'none';
            if (win) { setMsg("TEAM", "#10b981", "門診處理完畢。"); caseCount++; updateUI(); }
            else { setMsg("CUDDY", "#ef4444", "豪斯！你又超時了！"); }
        }

        function setMsg(n, c, m) { document.getElementById('speaker-name').innerText = n; document.getElementById('speaker-name').style.color = c; document.getElementById('main-message').innerText = m; }
        function updateUI() { document.getElementById('case-counter').innerText = pillNeeded ? "MEDS" : `${caseCount}/${nextPillAt}`; }
        window.onload = init;
    </script>
</body>
</html>
