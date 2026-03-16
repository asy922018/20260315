<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>보어 원자 모형 학습기 - 확대 기능</title>
    <style>
        :root {
            --bg-color: #f0f2f5;
            --card-bg: #ffffff;
            --text-main: #2c3e50;
            --color-metal: #ffcccc;
            --color-nonmetal: #d1e9ff;
            --color-metalloid: #d4f1be;
            --primary-blue: #3498db;
            --electron-inner: #27ae60;
            --electron-outer: #e67e22;
        }

        body {
            font-family: 'Pretendard', -apple-system, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            margin: 0;
        }

        .container {
            display: flex;
            gap: 30px;
            max-width: 1300px;
            width: 100%;
            justify-content: center;
            align-items: flex-start;
            flex-wrap: wrap;
        }

        /* 주기율표 */
        .periodic-table {
            display: grid;
            grid-template-columns: repeat(8, 75px);
            grid-gap: 10px;
            background: white;
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.05);
        }

        .element-card {
            width: 75px;
            height: 95px;
            border-radius: 12px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
            position: relative;
            border: 1px solid rgba(0,0,0,0.03);
        }

        .element-card:hover {
            transform: translateY(-5px) scale(1.05);
            box-shadow: 0 8px 20px rgba(0,0,0,0.1);
            filter: brightness(0.95);
        }

        .element-card.active {
            outline: 4px solid var(--primary-blue);
            z-index: 2;
        }

        .metal { background-color: var(--color-metal); }
        .nonmetal { background-color: var(--color-nonmetal); }
        .metalloid { background-color: var(--color-metalloid); }

        .element-card .num { font-size: 11px; position: absolute; top: 6px; left: 8px; font-weight: bold; opacity: 0.7; }
        .element-card .symbol { font-size: 26px; font-weight: 800; }
        .element-card .name { font-size: 12px; margin-top: 4px; font-weight: 500; }

        /* 상세 패널 */
        .detail-panel {
            background: var(--card-bg);
            padding: 30px;
            border-radius: 25px;
            box-shadow: 0 15px 40px rgba(0,0,0,0.1);
            width: 420px;
            text-align: center;
            position: relative;
        }

        .info-header h2 { margin: 0; font-size: 40px; color: var(--primary-blue); }
        .type-badge {
            display: inline-block; padding: 5px 15px; border-radius: 20px;
            font-size: 13px; font-weight: bold; margin-top: 10px;
        }

        .info-grid {
            display: grid; grid-template-columns: 1fr 1fr; gap: 12px;
            margin: 25px 0; font-size: 15px;
        }
        .info-item { background: #f8f9fa; padding: 15px; border-radius: 15px; border: 1px solid #eee; }

        /* 보어 모형 SVG 영역 */
        .model-container {
            position: relative;
            background: #fff;
            border-radius: 20px;
            padding: 10px;
            border: 1px solid #f0f0f0;
        }

        .zoom-btn {
            position: absolute; top: 10px; right: 10px;
            background: #eee; border: none; border-radius: 50%;
            width: 40px; height: 40px; cursor: pointer;
            display: flex; align-items: center; justify-content: center;
            transition: background 0.2s; font-size: 20px;
        }
        .zoom-btn:hover { background: #ddd; }

        /* 확대 모달(Modal) */
        .modal {
            display: none; position: fixed; top: 0; left: 0;
            width: 100%; height: 100%; background: rgba(0,0,0,0.85);
            z-index: 1000; justify-content: center; align-items: center;
            backdrop-filter: blur(5px);
        }
        .modal-content {
            background: white; padding: 40px; border-radius: 30px;
            position: relative; text-align: center;
            max-width: 90vw; max-height: 90vh;
        }
        .close-btn {
            position: absolute; top: 20px; right: 20px;
            font-size: 30px; cursor: pointer; color: #333;
        }

        .legend {
            display: flex; justify-content: center; gap: 15px; margin-top: 20px; font-size: 13px;
        }
        .dot { width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 5px; }

        .empty { visibility: hidden; }
    </style>
</head>
<body>

    <h1 style="margin-bottom: 5px;">인터랙티브 보어 원자 모형 학습</h1>
    <p style="color: #666; margin-bottom: 30px;">원소를 선택하여 전자 배치와 화학적 성질을 학습하세요.</p>

    <div class="container">
        <!-- 주기율표 -->
        <div style="display: flex; flex-direction: column; gap: 20px;">
            <div class="periodic-table" id="periodicTable"></div>
            
            <div style="display: flex; justify-content: center; gap: 25px; background: white; padding: 15px; border-radius: 15px; font-size: 14px; box-shadow: 0 4px 10px rgba(0,0,0,0.05);">
                <span><div style="width:15px; height:15px; background:var(--color-metal); display:inline-block; vertical-align:middle; border-radius:3px; margin-right:5px;"></div> 금속</span>
                <span><div style="width:15px; height:15px; background:var(--color-nonmetal); display:inline-block; vertical-align:middle; border-radius:3px; margin-right:5px;"></div> 비금속</span>
                <span><div style="width:15px; height:15px; background:var(--color-metalloid); display:inline-block; vertical-align:middle; border-radius:3px; margin-right:5px;"></div> 준금속</span>
            </div>
        </div>

        <!-- 상세 패널 -->
        <div class="detail-panel">
            <div class="info-header">
                <h2 id="displaySymbol">H</h2>
                <div id="displayName" style="font-size: 18px; font-weight: 600;">수소</div>
                <div id="displayType" class="type-badge">분류</div>
            </div>

            <div class="info-grid">
                <div class="info-item"><b>원자 번호</b><br><span id="infoNum" style="font-size: 20px; color: #333;">1</span></div>
                <div class="info-item"><b>전자 배치</b><br><span id="infoConfig" style="font-size: 20px; color: #333;">1</span></div>
            </div>

            <div class="model-container">
                <button class="zoom-btn" onclick="openModal()" title="확대 보기">🔍</button>
                <svg id="main-svg" width="350" height="350"></svg>
            </div>

            <div class="legend">
                <span><div class="dot" style="background:var(--electron-inner)"></div> 안쪽 전자</span>
                <span><div class="dot" style="background:var(--electron-outer)"></div> 최외각 전자</span>
            </div>
        </div>
    </div>

    <!-- 확대 모달 -->
    <div id="zoomModal" class="modal" onclick="closeModal()">
        <div class="modal-content" onclick="event.stopPropagation()">
            <span class="close-btn" onclick="closeModal()">&times;</span>
            <h2 id="modalTitle" style="margin-top: 0; color: var(--primary-blue);">원자 모형 확대</h2>
            <svg id="modal-svg" width="600" height="600"></svg>
            <div id="modalInfo" style="margin-top: 20px; font-size: 18px; color: #555;"></div>
        </div>
    </div>

    <script>
        const elements = [
            { n: 1, s: "H", name: "수소", group: 1, period: 1, config: [1], type: "nonmetal" },
            { n: 0, s: "", empty: true }, { n: 0, s: "", empty: true }, { n: 0, s: "", empty: true },
            { n: 0, s: "", empty: true }, { n: 0, s: "", empty: true }, { n: 0, s: "", empty: true },
            { n: 2, s: "He", name: "헬륨", group: 18, period: 1, config: [2], type: "nonmetal" },
            
            { n: 3, s: "Li", name: "리튬", group: 1, period: 2, config: [2, 1], type: "metal" },
            { n: 4, s: "Be", name: "베릴륨", group: 2, period: 2, config: [2, 2], type: "metal" },
            { n: 5, s: "B", name: "붕소", group: 13, period: 2, config: [2, 3], type: "metalloid" },
            { n: 6, s: "C", name: "탄소", group: 14, period: 2, config: [2, 4], type: "nonmetal" },
            { n: 7, s: "N", name: "질소", group: 15, period: 2, config: [2, 5], type: "nonmetal" },
            { n: 8, s: "O", name: "산소", group: 16, period: 2, config: [2, 6], type: "nonmetal" },
            { n: 9, s: "F", name: "플루오린", group: 17, period: 2, config: [2, 7], type: "nonmetal" },
            { n: 10, s: "Ne", name: "네온", group: 18, period: 2, config: [2, 8], type: "nonmetal" },

            { n: 11, s: "Na", name: "나트륨", group: 1, period: 3, config: [2, 8, 1], type: "metal" },
            { n: 12, s: "Mg", name: "마그네슘", group: 2, period: 3, config: [2, 8, 2], type: "metal" },
            { n: 13, s: "Al", name: "알루미늄", group: 13, period: 3, config: [2, 8, 3], type: "metal" },
            { n: 14, s: "Si", name: "규소", group: 14, period: 3, config: [2, 8, 4], type: "metalloid" },
            { n: 15, s: "P", name: "인", group: 15, period: 3, config: [2, 8, 5], type: "nonmetal" },
            { n: 16, s: "S", name: "황", group: 16, period: 3, config: [2, 8, 6], type: "nonmetal" },
            { n: 17, s: "Cl", name: "염소", group: 17, period: 3, config: [2, 8, 7], type: "nonmetal" },
            { n: 18, s: "Ar", name: "아르곤", group: 18, period: 3, config: [2, 8, 8], type: "nonmetal" },

            { n: 19, s: "K", name: "칼륨", group: 1, period: 4, config: [2, 8, 8, 1], type: "metal" },
            { n: 20, s: "Ca", name: "칼슘", group: 2, period: 4, config: [2, 8, 8, 2], type: "metal" }
        ];

        let currentElement = elements[0];
        const typeMap = { metal: "금속", nonmetal: "비금속", metalloid: "준금속" };

        function init() {
            const table = document.getElementById('periodicTable');
            elements.forEach(el => {
                const div = document.createElement('div');
                if (el.empty) {
                    div.className = 'element-card empty';
                } else {
                    div.className = `element-card ${el.type}`;
                    div.innerHTML = `<span class="num">${el.n}</span><span class="symbol">${el.s}</span><span class="name">${el.name}</span>`;
                    div.onclick = () => selectElement(el, div);
                }
                table.appendChild(div);
            });
            selectElement(elements[0], document.querySelector('.element-card'));
        }

        function selectElement(el, card) {
            currentElement = el;
            document.querySelectorAll('.element-card').forEach(c => c.classList.remove('active'));
            card.classList.add('active');

            document.getElementById('displaySymbol').innerText = el.s;
            document.getElementById('displayName').innerText = el.name;
            document.getElementById('infoNum').innerText = el.n;
            document.getElementById('infoConfig').innerText = el.config.join(' - ');
            
            const badge = document.getElementById('displayType');
            badge.innerText = typeMap[el.type];
            badge.style.backgroundColor = `var(--color-${el.type})`;

            drawBohr(document.getElementById('main-svg'), el, 350);
        }

        function drawBohr(svg, el, size) {
            svg.innerHTML = '';
            const center = size / 2;
            const shellGap = size * 0.12;
            const nucleusRadius = size * 0.05;

            // 원자핵
            const nucleus = createSVG('circle', { cx: center, cy: center, r: nucleusRadius, fill: '#f1c40f' });
            svg.appendChild(nucleus);

            const nText = createSVG('text', { 
                x: center, y: center + (nucleusRadius/3), 
                'text-anchor': 'middle', 'font-size': nucleusRadius, 'font-weight': 'bold' 
            });
            nText.textContent = "+" + el.n;
            svg.appendChild(nText);

            // 전자 껍질
            el.config.forEach((count, idx) => {
                const radius = nucleusRadius + 25 + (idx * shellGap);
                const shell = createSVG('circle', { 
                    cx: center, cy: center, r: radius, 
                    fill: 'none', stroke: '#ddd', 'stroke-width': 1 
                });
                svg.appendChild(shell);

                // 전자
                for (let i = 0; i < count; i++) {
                    const angle = (i * (360 / count) - 90) * (Math.PI / 180);
                    const ex = center + radius * Math.cos(angle);
                    const ey = center + radius * Math.sin(angle);
                    
                    const electron = createSVG('circle', { 
                        cx: ex, cy: ey, r: size * 0.02, 
                        fill: (idx === el.config.length - 1) ? 'var(--electron-outer)' : 'var(--electron-inner)'
                    });
                    
                    // 애니메이션 효과
                    const anim = document.createElementNS("http://www.w3.org/2000/svg", "animate");
                    anim.setAttribute("attributeName", "r");
                    anim.setAttribute("from", "0");
                    anim.setAttribute("to", size * 0.02);
                    anim.setAttribute("dur", "0.3s");
                    electron.appendChild(anim);
                    
                    svg.appendChild(electron);
                }
            });
        }

        function createSVG(type, attrs) {
            const el = document.createElementNS("http://www.w3.org/2000/svg", type);
            for (let key in attrs) el.setAttribute(key, attrs[key]);
            return el;
        }

        function openModal() {
            const modal = document.getElementById('zoomModal');
            modal.style.display = 'flex';
            document.getElementById('modalTitle').innerText = `${currentElement.name}(${currentElement.s}) 원자 구조 확대 관찰`;
            document.getElementById('modalInfo').innerText = `원자 번호: ${currentElement.n} | 전자 배치: ${currentElement.config.join(' - ')}`;
            drawBohr(document.getElementById('modal-svg'), currentElement, 600);
        }

        function closeModal() {
            document.getElementById('zoomModal').style.display = 'none';
        }

        window.onload = init;
    </script>
</body>
</html>
