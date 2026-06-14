<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>能带演变与仿真</title>
    <style>
        * { margin:0; padding:0; box-sizing:border-box; }
        body {
            background: #0a0e17;
            font-family: 'Segoe UI', system-ui, 'PingFang SC', sans-serif;
            padding: 20px;
            color: #e2e8f0;
            display: flex; justify-content: center;
        }
        .main-container { width:100%; max-width:1400px; display:flex; flex-direction:column; gap:16px; }
        .header {
            background:#111827; padding:14px 24px; border-radius:12px;
            border:1px solid #1e293b; display:flex; justify-content:space-between; align-items:center;
        }
        .header h1 {
            font-size:1.5rem; font-weight:700;
            background: linear-gradient(135deg, #38bdf8, #a78bfa);
            background-clip: text; -webkit-background-clip: text;
            -webkit-text-fill-color: transparent; color: transparent;
        }
        .badge { background:#0f172a; border:1px solid #334155; padding:6px 14px; border-radius:6px; font-size:0.85rem; color:#94a3b8; }
        .control-panel {
            background:#111827; border:1px solid #1e293b; border-radius:12px; padding:16px 20px;
            display:grid; grid-template-columns: repeat(auto-fit, minmax(300px,1fr)); gap:24px;
        }
        .panel-section { background:#0f172a; border-radius:8px; padding:14px; border:1px solid #1e293b; }
        .panel-section h3 { font-size:0.9rem; font-weight:600; color:#38bdf8; margin-bottom:12px; border-left:3px solid #38bdf8; padding-left:8px; }
        .control-row { display:flex; flex-wrap:wrap; gap:12px; align-items:flex-end; }
        .control-item { flex:1 1 120px; display:flex; flex-direction:column; gap:4px; }
        .control-item label {
            font-size:0.7rem; color:#64748b; text-transform:uppercase; letter-spacing:0.5px;
            display:flex; justify-content:space-between; align-items:baseline;
        }
        .doping-value { font-weight:600; color:#cbd5e1; font-size:0.8rem; }
        select, input[type="range"] {
            background:#1e293b; border:1px solid #334155; color:#e2e8f0;
            padding:7px 8px; border-radius:6px; font-size:0.8rem; outline:none; font-family:inherit;
        }
        input[type="range"] { appearance:none; -webkit-appearance:none; height:5px; padding:0; background:#334155; cursor:pointer; border-radius:3px; }
        input[type="range"]::-moz-range-track { height:5px; background:#334155; border-radius:3px; border:none; }
        input[type="range"]::-moz-range-thumb { width:18px; height:18px; border-radius:50%; background:#38bdf8; border:none; cursor:pointer; }
        input[type="range"]::-webkit-slider-thumb { -webkit-appearance:none; appearance:none; width:18px; height:18px; border-radius:50%; background:#38bdf8; border:none; margin-top:-6.5px; }
        input[type="range"]::-webkit-slider-runnable-track { height:5px; background:#334155; border-radius:3px; }
        input[type="range"]:disabled { opacity:0.4; cursor:not-allowed; }
        .btn-group { display:flex; gap:6px; }
        .btn { background:#1e293b; border:1px solid #334155; color:#cbd5e1; padding:6px 12px; border-radius:6px; font-size:0.75rem; cursor:pointer; transition:0.2s; }
        .btn.active { background:#38bdf8; color:#0f172a; font-weight:bold; border-color:#38bdf8; }
        .canvas-grid { display:grid; grid-template-columns: 1fr 1fr; gap:16px; }
        @media (max-width:900px) { .canvas-grid { grid-template-columns:1fr; } }
        .canvas-card {
            background:#0f172a; border-radius:12px; border:1px solid #1e293b;
            overflow:hidden; display:flex; flex-direction:column;
        }
        .canvas-card .card-label {
            padding:8px 14px; font-size:0.75rem; font-weight:600; color:#94a3b8;
            background:#111827; border-bottom:1px solid #1e293b; display:flex; justify-content:space-between;
        }
        .canvas-wrap { flex:1; min-height:420px; position:relative; }
        canvas { display:block; width:100%; height:100%; }
        .info-bar { display:flex; justify-content:space-between; padding:6px 14px; font-size:0.7rem; color:#64748b; background:#0f172a; border-top:1px solid #1e293b; }
        .warning-text { color:#facc15; font-size:0.75rem; text-align:center; margin-top:8px; padding:6px; background:rgba(250,204,21,0.1); border-radius:6px; }
        .param-table { width:100%; font-size:0.75rem; border-collapse:collapse; }
        .param-table td { padding:4px 6px; border-bottom:1px solid #1e293b; color:#cbd5e1; }
        .param-table td:first-child { width:40%; }
    </style>
</head>
<body>
    <div class="main-container">
        <div class="header">
            <h1>⚡ 能带演变与仿真</h1>
            <div class="badge" id="junction-badge">同质/异质结</div>
        </div>
        <div class="control-panel" id="controlPanel"></div>
        <div class="canvas-grid">
            <div class="canvas-card">
                <div class="card-label">📊 能带演变图 <span id="band-state-text">独立状态</span></div>
                <div class="canvas-wrap"><canvas id="bandCanvas"></canvas></div>
                <div class="info-bar" id="band-info">势垒 | 耗尽宽度</div>
            </div>
            <div class="canvas-card">
                <div class="card-label">📈 I‑V 特性曲线 <span id="iv-state-text">工作点</span></div>
                <div class="canvas-wrap"><canvas id="ivCanvas"></canvas></div>
                <div class="info-bar" id="iv-info">电压/电流</div>
            </div>
        </div>
        <div class="panel-section" style="margin-top:0;">
            <h3>📋 实时演算参数</h3>
            <table class="param-table" id="paramTable">
                <tr><td>左侧材料</td><td id="p-leftMat">-</td><td>右侧材料</td><td id="p-rightMat">-</td></tr>
                <tr><td>左侧 Eg</td><td id="p-leftEg">-</td><td>右侧 Eg</td><td id="p-rightEg">-</td></tr>
                <tr><td>左侧 χ</td><td id="p-leftChi">-</td><td>右侧 χ</td><td id="p-rightChi">-</td></tr>
                <tr><td>平衡 Ef (avg)</td><td id="p-ef-avg" colspan="3">-</td></tr>
                <tr><td>内建电势 Vbi</td><td id="p-Vbi">-</td><td>接触电势差 Vd</td><td id="p-Vd">-</td></tr>
                <tr><td>势垒高度</td><td id="p-barrier">-</td><td>耗尽层宽度 W</td><td id="p-W">-</td></tr>
            </table>
        </div>
    </div>
    <script>
    (function () {
        const mats = {
            'Si':   { eg:1.12, chi:4.05, nc:2.8e19, nv:1.0e19, eps:11.7, ni300:1.5e10, mun:1350, mup:480 },
            'Ge':   { eg:0.66, chi:3.65, nc:1.04e19, nv:6.0e18, eps:16.0, ni300:2.4e13, mun:3900, mup:1900 },
            'GaAs': { eg:1.42, chi:4.45, nc:4.7e17, nv:7.0e18, eps:13.1, ni300:1.8e6, mun:8000, mup:400 }
        };
        const q = 1.602e-19, k = 1.38e-23, eps0 = 8.854e-12, k_eV = 8.617333e-5;

        const state = {
            leftMat: 'GaAs', leftType: 'p', leftDoping: 1e16,
            rightMat: 'Si', rightType: 'n', rightDoping: 1e16,
            temp: 300, bias: 0.0, stage: 0.0, speed: 1.0
        };
        const particles = [];
        const isHomojunction = () => state.leftMat === state.rightMat;

        function formatDoping(val) {
            if (val >= 1e18) return (val/1e18).toFixed(1)+'×10¹⁸';
            if (val >= 1e17) return (val/1e17).toFixed(1)+'×10¹⁷';
            if (val >= 1e16) return (val/1e16).toFixed(1)+'×10¹⁶';
            if (val >= 1e15) return (val/1e15).toFixed(1)+'×10¹⁵';
            return (val/1e14).toFixed(1)+'×10¹⁴';
        }

        function calcPhysics() {
            const L = mats[state.leftMat], R = mats[state.rightMat];
            const Vt = (k * state.temp) / q;
            const computeEf = (mat, type, doping) => {
                const midGap = -mat.chi - mat.eg/2;
                const shift = type==='n' ? Vt*Math.log(mat.nc/doping)*0.6 : -Vt*Math.log(mat.nv/doping)*0.6;
                return midGap + shift;
            };
            const ef1 = computeEf(L, state.leftType, state.leftDoping);
            const ef2 = computeEf(R, state.rightType, state.rightDoping);
            const Vd = ef1 - ef2;
            const Vtotal = Math.abs(Vd - state.bias);
            const eps1 = L.eps*eps0, eps2 = R.eps*eps0;
            const n1 = state.leftDoping*1e6, n2 = state.rightDoping*1e6;
            const V1 = Vtotal * (eps2*n2) / (eps1*n1 + eps2*n2);
            const V2 = Vtotal * (eps1*n1) / (eps1*n1 + eps2*n2);
            const x1 = Math.sqrt((2*eps1*V1)/(q*n1))*1e9;
            const x2 = Math.sqrt((2*eps2*V2)/(q*n2))*1e9;
            const W = x1 + x2;
            const Vbi = isHomojunction()
                ? (k_eV*state.temp)*Math.log(state.leftDoping*state.rightDoping /
                    Math.pow(L.ni300*Math.pow(state.temp/300,1.5)*Math.exp(-L.eg/(2*k_eV*state.temp)+L.eg/(2*k_eV*300)),2))
                : Vd;
            return { L, R, ef1, ef2, Vd, Vbi, V1, V2, x1, x2, W };
        }

        function updateParticles(dt) {
            const ph = calcPhysics();
            const rate = 5*(1+Math.abs(ph.Vd - state.bias)*0.3)*state.speed;
            if (particles.length < 70 && Math.random() < rate*dt) {
                const left = Math.random()<0.5;
                particles.push({
                    left, type: left ? (state.leftType==='p'?'h':'e') : (state.rightType==='p'?'h':'e'),
                    x_rel:Math.random(), y_off:(Math.random()-0.5)*20,
                    vx:0.003+Math.random()*0.008, life:1.0
                });
            }
            for (let i=particles.length-1; i>=0; i--) {
                const p = particles[i];
                let drive = 0;
                if (state.stage>=0.98) drive = state.bias>0 ? state.bias*0.25 : state.bias*0.02;
                const mv = p.vx + drive*0.005;
                if (p.type==='e') p.x_rel += mv*state.speed*dt*8;
                else p.x_rel -= mv*state.speed*dt*8;
                if (p.x_rel>1.1) p.x_rel=0;
                if (p.x_rel<-0.1) p.x_rel=1;
                p.life -= dt*0.2;
                if (p.life<=0) particles.splice(i,1);
            }
        }

        function drawBand(canvas, ctx) {
            const dpr = window.devicePixelRatio||1;
            const rect = canvas.parentElement.getBoundingClientRect();
            const cw = rect.width, ch = Math.max(420, rect.height);
            canvas.width = cw*dpr; canvas.height = ch*dpr;
            canvas.style.width = cw+'px'; canvas.style.height = ch+'px';
            ctx.setTransform(dpr,0,0,dpr,0,0);
            ctx.clearRect(0,0,cw,ch);
            const midX = cw/2;
            const ph = calcPhysics();
            const { L, R, Vd, V1, V2, x1, x2, W, ef1, ef2 } = ph;
            const E0_L = 0;
            const ΔV_total = Vd - state.bias;
            const signΔ = ΔV_total >= 0 ? 1 : -1;
            const E0_R = E0_L - ΔV_total * state.stage;
            const totalVis = Math.min(150, cw*0.4);
            const dwL = W>0 ? (totalVis*(x1/W))*state.stage : 0;
            const dwR = W>0 ? (totalVis*(x2/W))*state.stage : 0;
            const E0_int = E0_L - signΔ * V1 * state.stage;
            const getE0 = (x) => {
                if (x < midX - dwL) return E0_L;
                if (x > midX + dwR) return E0_R;
                if (x < midX) {
                    const t = (x - (midX - dwL)) / (dwL || 1);
                    return E0_L + (E0_int - E0_L) * t * t;
                } else {
                    const t = ((midX + dwR) - x) / (dwR || 1);
                    return E0_R + (E0_int - E0_R) * t * t;
                }
            };
            const eMin = Math.min(-L.chi-L.eg, -R.chi-R.eg)-0.3;
            const eMax = Math.max(E0_L, E0_R)+0.3;
            const scale = ch*0.7/(eMax-eMin);
            const toY = (energy) => ch*0.8 - (energy-eMin)*scale;

            // 网格
            ctx.strokeStyle='#1e293b'; ctx.lineWidth=0.5;
            for(let i=0;i<=8;i++){ const y=ch*0.1+i*(ch*0.7)/8; ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(cw,y); ctx.stroke(); }

            // 真空能级
            ctx.strokeStyle='#ffffff'; ctx.lineWidth=2.5; ctx.setLineDash([8,4]);
            ctx.beginPath();
            for(let x=0;x<cw;x+=3){ let y=toY(getE0(x)); x===0?ctx.moveTo(x,y):ctx.lineTo(x,y); }
            ctx.stroke(); ctx.setLineDash([]);

            // 费米能级
            const ef_avg = (ef1 + ef2)/2;
            const base_efL = ef1 + (ef_avg - ef1) * state.stage;
            const base_efR = ef2 + (ef_avg - ef2) * state.stage;
            let efL, efR;
            if (state.stage >= 1.0 && Math.abs(state.bias) > 0.001) {
                efL = base_efL - state.bias/2;
                efR = base_efR + state.bias/2;
            } else {
                efL = base_efL;
                efR = base_efR;
            }
            const isEquilibrium = (state.stage >= 1.0 && Math.abs(state.bias) < 0.001);

            ctx.strokeStyle='#3b82f6'; ctx.lineWidth=1.8;
            if (isEquilibrium) {
                ctx.setLineDash([6,4]);
                const yEf = toY(ef_avg);
                ctx.beginPath(); ctx.moveTo(0,yEf); ctx.lineTo(cw,yEf); ctx.stroke();
                ctx.setLineDash([]);
            } else {
                ctx.setLineDash([6,4]);
                ctx.beginPath(); ctx.moveTo(0, toY(efL)); ctx.lineTo(midX, toY(efL)); ctx.stroke();
                ctx.beginPath(); ctx.moveTo(midX, toY(efR)); ctx.lineTo(cw, toY(efR)); ctx.stroke();
                ctx.setLineDash([]);
            }

            // 导带与价带
            const drawEcEv = (side) => {
                const mat = side==='L'?L:R;
                const startX = side==='L'?0:midX;
                const endX = side==='L'?midX:cw;
                ctx.lineWidth=3.2;
                ctx.strokeStyle='#ef4444'; ctx.beginPath();
                for(let x=startX;x<=endX;x+=3){ const e0=getE0(x); const ec=e0-mat.chi; const y=toY(ec); x===startX?ctx.moveTo(x,y):ctx.lineTo(x,y); }
                ctx.stroke();
                ctx.strokeStyle='#10b981'; ctx.beginPath();
                for(let x=startX;x<=endX;x+=3){ const e0=getE0(x); const ev=e0-mat.chi-mat.eg; const y=toY(ev); x===startX?ctx.moveTo(x,y):ctx.lineTo(x,y); }
                ctx.stroke();
            };
            drawEcEv('L'); drawEcEv('R');
            if (isHomojunction()) {
                ctx.lineWidth=3.2;
                ctx.strokeStyle='#ef4444'; ctx.beginPath(); ctx.moveTo(midX-2, toY(getE0(midX-2)-L.chi)); ctx.lineTo(midX+2, toY(getE0(midX+2)-R.chi)); ctx.stroke();
                ctx.strokeStyle='#10b981'; ctx.beginPath(); ctx.moveTo(midX-2, toY(getE0(midX-2)-L.chi-L.eg)); ctx.lineTo(midX+2, toY(getE0(midX+2)-R.chi-R.eg)); ctx.stroke();
            }

            // 耗尽层标记
            if (dwL>0||dwR>0) {
                const leftEdge=midX-dwL, rightEdge=midX+dwR;
                ctx.fillStyle='rgba(255,180,100,0.12)'; ctx.fillRect(leftEdge,0,dwL+dwR,ch);
                ctx.strokeStyle='#f59e0b'; ctx.lineWidth=1.5; ctx.setLineDash([5,5]);
                ctx.beginPath(); ctx.moveTo(leftEdge,0); ctx.lineTo(leftEdge,ch); ctx.stroke();
                ctx.beginPath(); ctx.moveTo(rightEdge,0); ctx.lineTo(rightEdge,ch); ctx.stroke();
                ctx.setLineDash([]);
                ctx.fillStyle='#f59e0b'; ctx.font='bold 11px sans-serif'; ctx.textAlign='center';
                ctx.fillText('耗尽层', (leftEdge+rightEdge)/2, ch*0.12);
                ctx.fillText(`${(W*state.stage).toFixed(1)} nm`, (leftEdge+rightEdge)/2, ch*0.12+14);
            }

            // 界面虚线
            ctx.strokeStyle='rgba(255,255,255,0.2)'; ctx.lineWidth=1; ctx.setLineDash([5,5]);
            ctx.beginPath(); ctx.moveTo(midX,0); ctx.lineTo(midX,ch); ctx.stroke(); ctx.setLineDash([]);

            // 粒子 (含动态标签)
            particles.forEach(p => {
                const px = p.left ? p.x_rel*midX : midX + p.x_rel*(cw-midX);
                const e0 = getE0(px);
                const mat = px<midX ? L : R;
                const ec = e0 - mat.chi;
                const ev = e0 - mat.chi - mat.eg;
                const py = p.type==='e' ? toY(ec)-5 : toY(ev)+5;
                ctx.shadowBlur = 6;
                ctx.shadowColor = p.type==='e'?'#60a5fa':'#f43f5e';
                ctx.fillStyle = p.type==='e'?'#60a5fa':'#f43f5e';
                ctx.beginPath(); ctx.arc(px,py,3.5,0,Math.PI*2); ctx.fill();
                ctx.shadowBlur = 0;
                ctx.font = 'bold 9px sans-serif';
                ctx.fillStyle = p.type==='e' ? '#60a5fa' : '#f43f5e';
                ctx.textAlign = 'start';
                ctx.textBaseline = 'middle';
                ctx.fillText(p.type==='e' ? 'e⁻' : 'h⁺', px + 6, py);
            });

            // 文字标注
            ctx.fillStyle='#94a3b8'; ctx.font='10px sans-serif';
            ctx.textAlign = 'start';
            ctx.fillText('真空能级 E₀', 15, toY(E0_L)-10);
            if (isEquilibrium) {
                ctx.fillText('费米能级 Ef (平衡)', 15, toY(ef_avg)-10);
            } else if (state.stage < 1.0) {
                ctx.fillText('左侧 Ef', 15, toY(efL)-10);
                ctx.textAlign = 'end';
                ctx.fillText('右侧 Ef', cw-15, toY(efR)-10);
                ctx.textAlign = 'start';
            } else {
                ctx.fillText('准费米 Ef(L)', 15, toY(efL)-10);
                ctx.textAlign = 'end';
                ctx.fillText('准费米 Ef(R)', cw-15, toY(efR)-10);
                ctx.textAlign = 'start';
            }
            ctx.fillText('导带 Ec', 15, toY(E0_L - L.chi)-10);
            ctx.fillText('价带 Ev', 15, toY(E0_L - L.chi - L.eg)+14);

            document.getElementById('band-info').innerHTML =
                `势垒≈${Math.abs(Vd-state.bias).toFixed(2)}eV | 耗尽层≈${(W*state.stage).toFixed(1)}nm | ${isHomojunction()?'同质结':'异质结'}`;
            const bandStateEl = document.getElementById('band-state-text');
            if (state.stage === 0) bandStateEl.textContent = '独立状态';
            else if (state.stage < 1) bandStateEl.textContent = '接触过渡中';
            else if (Math.abs(state.bias) < 0.001) bandStateEl.textContent = '平衡建立';
            else bandStateEl.textContent = '偏压状态';

            updateParamPanel(ph, ef_avg);
        }

        function updateParamPanel(ph, efAvg) {
            document.getElementById('p-leftMat').textContent = state.leftMat;
            document.getElementById('p-rightMat').textContent = state.rightMat;
            document.getElementById('p-leftEg').textContent = ph.L.eg.toFixed(2)+' eV';
            document.getElementById('p-rightEg').textContent = ph.R.eg.toFixed(2)+' eV';
            document.getElementById('p-leftChi').textContent = ph.L.chi.toFixed(2)+' eV';
            document.getElementById('p-rightChi').textContent = ph.R.chi.toFixed(2)+' eV';
            document.getElementById('p-ef-avg').textContent = efAvg.toFixed(3)+' eV';
            document.getElementById('p-Vbi').textContent = ph.Vbi.toFixed(3)+' V';
            document.getElementById('p-Vd').textContent = ph.Vd.toFixed(3)+' V';
            document.getElementById('p-barrier').textContent = Math.abs(ph.Vd-state.bias).toFixed(3)+' eV';
            document.getElementById('p-W').textContent = (ph.W*state.stage).toFixed(2)+' nm';
        }

        function drawIv(canvas, ctx) {
            const dpr = window.devicePixelRatio||1;
            const rect = canvas.parentElement.getBoundingClientRect();
            const cw = rect.width, ch = Math.max(420, rect.height);
            canvas.width = cw*dpr; canvas.height = ch*dpr;
            canvas.style.width = cw+'px'; canvas.style.height = ch+'px';
            ctx.setTransform(dpr,0,0,dpr,0,0);
            ctx.clearRect(0,0,cw,ch);
            const pL=60, pR=30, pT=30, pB=50;
            const plotW = cw-pL-pR, plotH = ch-pT-pB;
            const toX = v => pL + (v+2)/4*plotW;

            const Vt_vis = 0.12;
            const I0 = 0.005;
            const mapScale = 80;

            ctx.strokeStyle='#38bdf8'; ctx.lineWidth=2.5;
            ctx.beginPath();
            let first = true;
            for (let v=-2; v<=2; v+=0.05) {
                let cur = (v>0) ? I0*(Math.exp(v/Vt_vis)-1) : -I0;
                const y = pT+plotH/2 - cur*mapScale;
                if (first) { ctx.moveTo(toX(v),y); first=false; } else ctx.lineTo(toX(v),y);
            }
            ctx.stroke();

            if (state.stage>0.9) {
                const v = state.bias;
                let cur = (v>0) ? I0*(Math.exp(v/Vt_vis)-1) : -I0;
                const x = toX(v), y = pT+plotH/2 - cur*mapScale;
                ctx.fillStyle='#facc15'; ctx.shadowBlur=8; ctx.shadowColor='#facc15';
                ctx.beginPath(); ctx.arc(x,y,6,0,Math.PI*2); ctx.fill(); ctx.shadowBlur=0;
            }

            ctx.strokeStyle='#1e293b'; ctx.lineWidth=0.5;
            for(let i=0;i<=6;i++){ const y=pT+i*plotH/6; ctx.beginPath(); ctx.moveTo(pL,y); ctx.lineTo(cw-pR,y); ctx.stroke(); }
            ctx.strokeStyle='#334155';
            ctx.beginPath(); ctx.moveTo(pL, pT+plotH/2); ctx.lineTo(cw-pR, pT+plotH/2); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(toX(0), pT); ctx.lineTo(toX(0), ch-pB); ctx.stroke();
            ctx.fillStyle='#94a3b8'; ctx.font='10px sans-serif'; ctx.textAlign='center';
            ctx.fillText('电压 [V]', cw/2, ch-12);
            document.getElementById('iv-info').innerHTML = `示意模式 | 偏压 ${state.bias.toFixed(2)}V`;
        }

        function setSpeed(speed, activeId) {
            state.speed = speed;
            ['sp05','sp1','sp2'].forEach(id => {
                const btn = document.getElementById(id);
                if(btn) btn.classList.toggle('active', id===activeId);
            });
        }

        function buildControls() {
            const container = document.getElementById('controlPanel');
            container.innerHTML = `
            <div class="panel-section"><h3>🧬 左侧半导体</h3><div class="control-row">
                <div class="control-item"><label>材料</label><select id="leftMat">${Object.keys(mats).map(m=>`<option value="${m}" ${m===state.leftMat?'selected':''}>${m}</option>`).join('')}</select></div>
                <div class="control-item"><label>类型</label><select id="leftType"><option value="p" ${state.leftType==='p'?'selected':''}>P型</option><option value="n" ${state.leftType==='n'?'selected':''}>N型</option></select></div>
                <div class="control-item"><label>掺杂 <span class="doping-value" id="leftDopingDisp">1.0×10¹⁶ cm⁻³</span></label><input type="range" id="leftDoping" min="14" max="18" value="16" step="0.2"></div>
            </div></div>
            <div class="panel-section"><h3>🧬 右侧半导体</h3><div class="control-row">
                <div class="control-item"><label>材料</label><select id="rightMat">${Object.keys(mats).map(m=>`<option value="${m}" ${m===state.rightMat?'selected':''}>${m}</option>`).join('')}</select></div>
                <div class="control-item"><label>类型</label><select id="rightType"><option value="n" ${state.rightType==='n'?'selected':''}>N型</option><option value="p" ${state.rightType==='p'?'selected':''}>P型</option></select></div>
                <div class="control-item"><label>掺杂 <span class="doping-value" id="rightDopingDisp">1.0×10¹⁶ cm⁻³</span></label><input type="range" id="rightDoping" min="14" max="18" value="16" step="0.2"></div>
            </div></div>
            <div class="panel-section"><h3>🌡️ 环境与演化控制</h3><div class="control-row">
                <div class="control-item"><label>温度 (K) <span id="tempVal">300</span></label><input type="range" id="temp" min="250" max="400" value="300" step="5"></div>
                <div class="control-item"><label>演化进度 <span id="stageVal">0%</span></label><input type="range" id="stage" min="0" max="1" step="0.01" value="0"></div>
                <div class="control-item"><label>外加偏压 V <span id="biasVal">0.00 V</span></label><input type="range" id="bias" min="-2" max="2" value="0" step="0.04" disabled></div>
                <div class="control-item"><label>流速</label><div class="btn-group">
                    <span class="btn" id="sp05">0.5×</span><span class="btn active" id="sp1">1×</span><span class="btn" id="sp2">2×</span></div></div>
            </div>
            <div class="warning-text" id="warningText">⚠️ 演变未达到100%前系统处于独立/接触中状态，外加电压锁定为0。完全建立平衡后方可加压。</div>`;

            const bind = (id, prop, transform, displayId) => {
                document.getElementById(id).addEventListener('input', function() {
                    const val = transform ? transform(this.value) : this.value;
                    state[prop] = val;
                    if (displayId && typeof val === 'number') {
                        document.getElementById(displayId).textContent = formatDoping(val) + ' cm⁻³';
                    }
                    if (id==='stage') {
                        const stageVal = parseFloat(val);
                        document.getElementById('stageVal').textContent = Math.round(stageVal*100)+'%';
                        const biasSlider = document.getElementById('bias');
                        if (stageVal<1.0) {
                            biasSlider.disabled = true;
                            state.bias = 0;
                            document.getElementById('bias').value = 0;
                            document.getElementById('biasVal').textContent = '0.00 V';
                            document.getElementById('warningText').style.display = 'block';
                        } else {
                            biasSlider.disabled = false;
                            document.getElementById('warningText').style.display = 'none';
                        }
                    }
                    if (id==='bias') document.getElementById('biasVal').textContent = parseFloat(val).toFixed(2)+' V';
                    if (id==='temp') document.getElementById('tempVal').textContent = val;
                    updateBadge();
                });
            };
            bind('leftMat','leftMat'); bind('leftType','leftType');
            bind('leftDoping','leftDoping', v=>Math.pow(10,v), 'leftDopingDisp');
            bind('rightMat','rightMat'); bind('rightType','rightType');
            bind('rightDoping','rightDoping', v=>Math.pow(10,v), 'rightDopingDisp');
            bind('temp','temp',parseInt); bind('stage','stage',parseFloat); bind('bias','bias',parseFloat);

            document.getElementById('sp05').addEventListener('click', ()=>setSpeed(0.5,'sp05'));
            document.getElementById('sp1').addEventListener('click', ()=>setSpeed(1.0,'sp1'));
            document.getElementById('sp2').addEventListener('click', ()=>setSpeed(2.0,'sp2'));

            function updateBadge() {
                document.getElementById('junction-badge').textContent = `${state.leftType}${state.rightType==='p'?'P':'N'} ${isHomojunction()?'同质结':'异质结'}`;
            }
            updateBadge();
        }

        const bandCanvas = document.getElementById('bandCanvas');
        const bandCtx = bandCanvas.getContext('2d');
        const ivCanvas = document.getElementById('ivCanvas');
        const ivCtx = ivCanvas.getContext('2d');
        let lastTime = performance.now();

        function animate(now) {
            const dt = Math.min(0.1, (now-lastTime)/1000);
            lastTime = now;
            updateParticles(dt);
            drawBand(bandCanvas, bandCtx);
            drawIv(ivCanvas, ivCtx);
            requestAnimationFrame(animate);
        }

        window.onload = () => {
            buildControls();
            requestAnimationFrame(animate);
        };
    })();
    </script>
</body>
</html>
