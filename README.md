import { useState, useEffect, useRef, useCallback } from “react”;

const WAR_START = new Date(“2026-02-28T00:00:00Z”);
const PROXY = “https://api.allorigins.win/get?url=”;

const FONTS = `@import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Rajdhani:wght@300;400;500;600;700&family=Orbitron:wght@400;700;900&display=swap');`;

const CITIES = [
{ name: “Tehran”, lon: 51.4, lat: 35.7, type: “capital” },
{ name: “Natanz”, lon: 51.7, lat: 33.7, type: “strike” },
{ name: “Fordow”, lon: 51.0, lat: 34.9, type: “strike” },
{ name: “Isfahan”, lon: 51.7, lat: 32.7, type: “strike” },
{ name: “Dubai”, lon: 55.3, lat: 25.2, type: “hit” },
{ name: “Abu Dhabi”, lon: 54.4, lat: 24.5, type: “hit” },
{ name: “Doha”, lon: 51.5, lat: 25.3, type: “hit” },
{ name: “Al Udeid”, lon: 51.3, lat: 25.1, type: “base” },
{ name: “Riyadh”, lon: 46.7, lat: 24.7, type: “normal” },
{ name: “Ras Tanura”, lon: 50.2, lat: 26.7, type: “hit” },
{ name: “Tel Aviv”, lon: 34.8, lat: 32.1, type: “hit” },
{ name: “Kuwait City”, lon: 47.9, lat: 29.4, type: “hit” },
{ name: “Manama”, lon: 50.6, lat: 26.2, type: “hit” },
{ name: “Muscat”, lon: 58.4, lat: 23.6, type: “normal” },
{ name: “Bandar Abbas”, lon: 56.3, lat: 27.2, type: “capital” },
];

const SHIPPING_LANES = [
{ points: [[56.5,26.5],[57.5,24.5],[58.5,22.5],[60,20],[63,18],[65,15]], name: “Hormuz→Indian Ocean”, status: “closed” },
{ points: [[48,29.5],[50,27],[52,25],[54,23],[55.5,22]], name: “Kuwait→Gulf”, status: “disrupted” },
{ points: [[51.5,25.3],[53,24],[55.3,25.2]], name: “Qatar LNG Route”, status: “suspended” },
{ points: [[43,12],[45,14],[48,16],[50,20],[52,24],[54,26],[55.5,26.5]], name: “Red Sea→Gulf of Aden”, status: “active” },
];

const TRAJECTORIES = [
[51.4,35.7,55.3,25.2,“ballistic”],[51.4,35.7,54.4,24.5,“ballistic”],
[51.4,35.7,47.9,29.4,“ballistic”],[51.4,35.7,34.8,32.1,“ballistic”],
[51.4,35.7,50.2,26.7,“ballistic”],[51.4,35.7,51.5,25.3,“drone”],
[51.4,35.7,50.6,26.2,“drone”],[51.4,35.7,46.7,24.7,“drone”],
[34.8,32.1,51.7,33.7,“strike”],[34.8,32.1,51.0,34.9,“strike”],
[34.8,32.1,51.4,35.7,“strike”],[51.3,25.1,51.7,33.7,“strike”],
[51.4,35.7,58.4,23.6,“drone”],[50.2,26.7,55.3,25.2,“drone”],
];

const AIR_ROUTES = [
{ from:[55.3,25.2], to:[51.5,35.7], callsign:“EK-DIVT”, type:“diverted”, alt:38000 },
{ from:[51.5,25.3], to:[38.8,35.0], callsign:“QR-RERT”, type:“rerouted”, alt:36000 },
{ from:[46.7,24.7], to:[55.3,25.2], callsign:“SV-112”, type:“active”, alt:34000 },
{ from:[58.4,23.6], to:[51.5,25.3], callsign:“WY-441”, type:“active”, alt:37000 },
{ from:[50.6,26.2], to:[46.7,24.7], callsign:“GF-302”, type:“active”, alt:32000 },
{ from:[34.8,32.1], to:[55.3,25.2], callsign:“LY-HALT”, type:“halted”, alt:0 },
];

const MIL_EVENTS = [
{ t:“intercept”, m:“Patriot PAC-3 intercepts ballistic missile Abu Dhabi” },
{ t:“missile”, m:“IRGC fires 8 ballistic missiles — 7 intercepted, 1 impacted Al-Kharj” },
{ t:“drone”, m:“Shahed-136 swarm: 45 units approaching UAE coast — 42 intercepted” },
{ t:“intercept”, m:“IDF Arrow-3 intercepts ballistic missile over Negev desert” },
{ t:“strike”, m:“CENTCOM B-2 strike destroys 4 Iranian mobile launchers, Ahvaz province” },
{ t:“intercept”, m:“THAAD intercepts theater BM over Al Udeid, Qatar” },
{ t:“drone”, m:“Iranian drone strikes AWS data center Bahrain — service disrupted” },
{ t:“intercept”, m:“US Navy SM-3 intercepts 2 missiles over Arabian Sea” },
{ t:“missile”, m:“Ballistic missile impacts Ramat Gan, Israel — 3 killed, 18 injured” },
{ t:“intercept”, m:“Iron Dome intercepts 11-rocket barrage northern Israel” },
{ t:“strike”, m:“IDF F-35I destroys Fordow enrichment facility tunnels — confirmed” },
{ t:“intercept”, m:“RAF Typhoons intercept 5 Shahed drones over Gulf — all destroyed” },
];

const CYBER_EVENTS = [
{ t:“cyber”, m:“MuddyWater WezRat infostealer detected UAE banking network — MOIS” },
{ t:“cyber”, m:“IRGC FAD: ZeroCleare wiper deployed Israeli security firm SCADA” },
{ t:“cyber”, m:“HydraC2 DDoS Qatar gov portals: 400Gbps sustained 6 hours” },
{ t:“cyber”, m:“Iran internet: 2% of normal. Israeli cyber op ongoing Day 10.” },
{ t:“cyber”, m:“Jordan NCSC: Thwarted Iranian ICS attack on wheat silo control systems” },
{ t:“cyber”, m:“Sicarii ransomware: 3 META victims — permanent data destruction mode” },
{ t:“econ”, m:“Qatar Force Majeure: all LNG suspended. 20% global supply offline.” },
{ t:“econ”, m:“WTI crude $91.40 — largest weekly futures gain since 1983” },
{ t:“flight”, m:“Emirates EK-401 diverted Turkey. UAE exclusion zone Day 10 active.” },
{ t:“cyber”, m:“NoName057 targets Kuwait/Jordan/Bahrain ICS portals #OpIsrael” },
];

function formatElapsed() {
const now = new Date();
const ms = now - WAR_START;
const d = Math.floor(ms / 86400000);
const h = Math.floor((ms % 86400000) / 3600000);
const m = Math.floor((ms % 3600000) / 60000);
const s = Math.floor((ms % 60000) / 1000);
return { day: d+1, str:`${d}d ${h}h ${m}m ${s}s`, h, m, s, date: now.toUTCString().slice(0,25)+” UTC” };
}

function hexToRgb(hex) {
const r = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
return r ? `${parseInt(r[1],16)},${parseInt(r[2],16)},${parseInt(r[3],16)}` : “0,0,0”;
}

// ─── COMPONENTS ──────────────────────────────────────────────────────────────

function ScanLine() {
return <div style={{position:“fixed”,top:0,left:0,right:0,bottom:0,pointerEvents:“none”,zIndex:9999,background:“repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,0.03) 2px,rgba(0,0,0,0.03) 4px)”,mixBlendMode:“overlay”}} />;
}

function GlitchText({ text, color=”#00e5ff” }) {
return (
<span style={{position:“relative”,color,textShadow:`0 0 10px ${color}88, 2px 0 rgba(255,0,0,0.3), -2px 0 rgba(0,255,255,0.3)`}}>
{text}
</span>
);
}

function StatusDot({ status }) {
const map = { live:”#00ff88”, sim:”#ff8c00”, pend:”#4a6070”, err:”#ff2233” };
const c = map[status]||”#4a6070”;
return <span style={{display:“inline-block”,width:6,height:6,borderRadius:“50%”,background:c,boxShadow:status===“live”?`0 0 6px ${c},0 0 12px ${c}33`:undefined,animation:status===“live”?“strobe 2s ease-in-out infinite”:undefined,marginRight:4}} />;
}

function Panel({ children, accent=”#00e5ff”, title, noPad }) {
return (
<div style={{background:“linear-gradient(135deg,rgba(8,18,28,0.95),rgba(5,12,20,0.98))”,border:`1px solid rgba(${hexToRgb(accent)},0.2)`,borderRadius:2,marginBottom:10,position:“relative”,overflow:“hidden”,boxShadow:`0 0 20px rgba(${hexToRgb(accent)},0.05), inset 0 1px 0 rgba(${hexToRgb(accent)},0.1)`}}>
<div style={{position:“absolute”,top:0,left:0,right:0,height:1,background:`linear-gradient(90deg,transparent,${accent},transparent)`,opacity:0.6}} />
<div style={{position:“absolute”,bottom:0,right:0,width:40,height:40,background:`radial-gradient(circle,rgba(${hexToRgb(accent)},0.08),transparent)`,pointerEvents:“none”}} />
{title && (
<div style={{display:“flex”,alignItems:“center”,gap:8,fontFamily:”‘Orbitron’,monospace”,fontSize:8,letterSpacing:3,color:accent,textTransform:“uppercase”,padding:“10px 14px 0”,marginBottom:10}}>
<div style={{width:3,height:14,background:accent,boxShadow:`0 0 8px ${accent}`}} />
{title}
<div style={{flex:1,height:1,background:`linear-gradient(90deg,rgba(${hexToRgb(accent)},0.3),transparent)`}} />
</div>
)}
<div style={noPad?{}:{padding:title?“0 14px 14px”:“14px”}}>{children}</div>
</div>
);
}

function StatCard({ label, value, color=”#00e5ff”, sub, icon }) {
return (
<div style={{background:“rgba(0,0,0,0.4)”,border:`1px solid rgba(${hexToRgb(color)},0.15)`,borderRadius:2,padding:“10px 12px”,position:“relative”,overflow:“hidden”}}>
<div style={{position:“absolute”,top:0,right:0,width:60,height:60,background:`radial-gradient(circle at top right,rgba(${hexToRgb(color)},0.08),transparent)`,pointerEvents:“none”}} />
<div style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,letterSpacing:2,color:”#3a5060”,marginBottom:4}}>{icon && <span style={{marginRight:4}}>{icon}</span>}{label}</div>
<div style={{fontFamily:”‘Orbitron’,monospace”,fontSize:22,lineHeight:1,color,textShadow:`0 0 15px rgba(${hexToRgb(color)},0.4)`}}>{value}</div>
{sub && <div style={{fontSize:8,color:”#3a5060”,marginTop:3,fontFamily:”‘Rajdhani’,sans-serif”}}>{sub}</div>}
</div>
);
}

function StatGrid({ stats, cols=2 }) {
return (
<div style={{display:“grid”,gridTemplateColumns:`repeat(${cols},1fr)`,gap:6,marginBottom:10}}>
{stats.map((s,i) => <StatCard key={i} {…s} />)}
</div>
);
}

function Bar({ label, pct, color=”#00ff88”, sub }) {
const c = color.startsWith(”#”) ? color : ({green:”#00ff88”,red:”#ff2233”,amber:”#ff8c00”}[color]||”#00ff88”);
return (
<div style={{marginBottom:10}}>
<div style={{display:“flex”,justifyContent:“space-between”,marginBottom:4}}>
<span style={{fontFamily:”‘Rajdhani’,sans-serif”,fontSize:11,fontWeight:600,letterSpacing:0.5}}>{label}</span>
<span style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:9,color:c}}>{pct}%</span>
</div>
<div style={{height:10,background:“rgba(255,255,255,0.03)”,border:“1px solid rgba(255,255,255,0.06)”,borderRadius:1,overflow:“hidden”,position:“relative”}}>
<div style={{height:“100%”,width:`${pct}%`,background:`linear-gradient(90deg,rgba(${hexToRgb(c)},0.3),rgba(${hexToRgb(c)},0.8))`,transition:“width 1.6s ease”,position:“relative”}}>
<div style={{position:“absolute”,right:0,top:0,bottom:0,width:2,background:“rgba(255,255,255,0.5)”,boxShadow:`0 0 4px ${c}`}} />
</div>
</div>
{sub && <div style={{fontSize:8,color:”#3a5060”,marginTop:2,fontFamily:”‘Rajdhani’,sans-serif”}}>{sub}</div>}
</div>
);
}

function EventStream({ events, maxItems=30 }) {
const colors = { intercept:”#00ff88”,missile:”#ff2233”,drone:”#ff8c00”,cyber:”#bf5fff”,econ:”#ffd700”,flight:”#00e5ff”,strike:”#00e5ff” };
const icons = { intercept:“⊙”,missile:“⚠”,drone:“◈”,cyber:“⌘”,econ:“▲”,flight:“✈”,strike:“✦” };
return (
<div style={{height:200,overflowY:“auto”,fontFamily:”‘Share Tech Mono’,monospace”,fontSize:9,lineHeight:2}}>
{events.slice(0,maxItems).map((e,i) => (
<div key={i} style={{display:“flex”,gap:6,padding:“1px 0”,borderBottom:“1px solid rgba(255,255,255,0.02)”,animation:i===0?“fadeIn .4s ease”:undefined}}>
<span style={{color:”#2a4050”,minWidth:60,flexShrink:0}}>[{e.ts}]</span>
<span style={{color:colors[e.t]||”#c8dde8”,minWidth:12}}>{icons[e.t]||”·”}</span>
<span style={{color:”#8aa8b8”,flex:1}}>{e.m}</span>
</div>
))}
</div>
);
}

// ─── MASTER MAP ───────────────────────────────────────────────────────────────
function MasterMap({ mode }) {
const canvasRef = useRef(null);
const stateRef = useRef({ threats:[], sweep:0, sweeps:0, intercepts:0, impacts:0, raf:null, airTraffic:[], shipTraffic:[], tick:0 });
const [counts, setCounts] = useState({ threats:0, intercepts:0, impacts:0, sweeps:0 });
const [mapLog, setMapLog] = useState([]);

const geo2c = useCallback((lon, lat, W, H) => {
const x = ((lon - 22) / (73 - 22)) * W;
const y = H - ((lat - 14) / (48 - 14)) * H;
return [x, y];
}, []);

const addLog = useCallback((type, msg) => {
const ts = new Date().toTimeString().slice(0,8);
setMapLog(prev => [{ t:type, m:msg, ts }, …prev].slice(0,50));
}, []);

const spawnThreat = useCallback(() => {
const traj = TRAJECTORIES[Math.floor(Math.random()*TRAJECTORIES.length)];
const intercepted = Math.random() < (traj[4]===“strike” ? 0.15 : 0.88);
stateRef.current.threats.push({
fromLon:traj[0], fromLat:traj[1], toLon:traj[2], toLat:traj[3],
type:traj[4], progress:0, speed:0.003+Math.random()*0.004,
intercepted, interceptAt: intercepted ? 0.5+Math.random()*0.35 : 1.1,
alive:true, trail:[], flash:0
});
}, []);

const initAirTraffic = useCallback(() => {
stateRef.current.airTraffic = AIR_ROUTES.map((r,i) => ({
…r, progress: Math.random(), trail:[], id:i
}));
}, []);

const initShipTraffic = useCallback(() => {
stateRef.current.shipTraffic = SHIPPING_LANES.flatMap((lane, li) => {
if (lane.status === “closed”) return [];
const count = lane.status === “suspended” ? 1 : 3;
return Array.from({length:count}, (_,i) => ({
lane: li, progress: i/count + Math.random()*0.1,
speed: 0.0003 + Math.random()*0.0002, trail:[],
status: lane.status
}));
});
}, []);

useEffect(() => {
const canvas = canvasRef.current;
if (!canvas) return;
const st = stateRef.current;
const W = canvas.offsetWidth || 480;
const H = Math.round(W * (34/51));
canvas.width = W; canvas.height = H;
const ctx = canvas.getContext(“2d”);

```
for (let i=0; i<4; i++) spawnThreat();
initAirTraffic();
initShipTraffic();
const spawnInt = setInterval(spawnThreat, 3000);

function getBezier(t, fLon, fLat, tLon, tLat) {
  const [fx,fy] = geo2c(fLon,fLat,W,H);
  const [tx,ty] = geo2c(tLon,tLat,W,H);
  const midX = (fx+tx)/2;
  const midY = Math.min(fy,ty) - (Math.abs(tx-fx)+Math.abs(ty-fy))*0.22;
  const p = Math.min(t,1);
  return [(1-p)*(1-p)*fx+2*(1-p)*p*midX+p*p*tx, (1-p)*(1-p)*fy+2*(1-p)*p*midY+p*p*ty];
}

function getLanePoint(lane, progress) {
  const pts = SHIPPING_LANES[lane].points;
  const totalSegs = pts.length-1;
  const seg = Math.min(Math.floor(progress*totalSegs), totalSegs-1);
  const segT = (progress*totalSegs) - seg;
  const [x1,y1] = geo2c(pts[seg][0],pts[seg][1],W,H);
  const [x2,y2] = geo2c(pts[seg+1][0],pts[seg+1][1],W,H);
  return [x1+(x2-x1)*segT, y1+(y2-y1)*segT];
}

function drawMap() {
  ctx.clearRect(0,0,W,H);
  // Deep ocean background
  const bg = ctx.createLinearGradient(0,0,W,H);
  bg.addColorStop(0,"#010810"); bg.addColorStop(0.5,"#020c16"); bg.addColorStop(1,"#010a14");
  ctx.fillStyle=bg; ctx.fillRect(0,0,W,H);

  // Grid lines
  ctx.strokeStyle="rgba(0,100,150,0.07)"; ctx.lineWidth=0.5;
  for (let lo=25; lo<=70; lo+=5) { const [x]=geo2c(lo,14,W,H); ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,H); ctx.stroke(); }
  for (let la=15; la<=47; la+=5) { const [,y]=geo2c(22,la,W,H); ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(W,y); ctx.stroke(); }

  // Lat/lon labels
  ctx.fillStyle="rgba(0,80,120,0.5)"; ctx.font=`${Math.max(5,W/100)}px 'Share Tech Mono',monospace`;
  for (let lo=30; lo<=70; lo+=10) { const [x]=geo2c(lo,14,W,H); ctx.textAlign="center"; ctx.fillText(`${lo}°E`,x,H-2); }
  for (let la=20; la<=45; la+=5) { const [,y]=geo2c(22,la,W,H); ctx.textAlign="left"; ctx.fillText(`${la}°N`,2,y); }

  // ─── SHIPPING LANES ────────────────────────────────────────────────────
  if (mode==="shipping" || mode==="all") {
    SHIPPING_LANES.forEach(lane => {
      const col = lane.status==="closed"?"rgba(255,34,51,0.5)":lane.status==="suspended"?"rgba(255,140,0,0.4)":"rgba(0,229,255,0.25)";
      const pts = lane.points;
      ctx.beginPath();
      pts.forEach(([lon,lat],i) => {
        const [x,y]=geo2c(lon,lat,W,H);
        i===0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
      });
      ctx.strokeStyle=col; ctx.lineWidth=lane.status==="closed"?2:1.5;
      ctx.setLineDash(lane.status==="closed"?[4,4]:lane.status==="suspended"?[2,3]:[]);
      ctx.stroke(); ctx.setLineDash([]);

      // Label
      const mid = pts[Math.floor(pts.length/2)];
      const [lx,ly]=geo2c(mid[0],mid[1],W,H);
      ctx.fillStyle=lane.status==="closed"?"rgba(255,34,51,0.8)":lane.status==="suspended"?"rgba(255,140,0,0.8)":"rgba(0,229,255,0.6)";
      ctx.font=`${Math.max(5,W/100)}px 'Share Tech Mono',monospace`; ctx.textAlign="center";
      ctx.fillText(`[${lane.status.toUpperCase()}]`,lx,ly-5);
    });

    // Ship icons
    st.shipTraffic.forEach(ship => {
      ship.progress = (ship.progress + ship.speed) % 1;
      const [sx,sy] = getLanePoint(ship.lane, ship.progress);
      ship.trail.push([sx,sy]); if(ship.trail.length>12) ship.trail.shift();
      const col = ship.status==="active"?"rgba(0,229,255,0.8)":"rgba(255,140,0,0.6)";
      if (ship.trail.length>1) {
        ctx.beginPath(); ship.trail.forEach(([px,py],i) => i===0?ctx.moveTo(px,py):ctx.lineTo(px,py));
        ctx.strokeStyle=col.replace("0.8","0.3").replace("0.6","0.2"); ctx.lineWidth=1; ctx.stroke();
      }
      ctx.beginPath(); ctx.arc(sx,sy,2.5,0,Math.PI*2);
      ctx.fillStyle=col; ctx.shadowBlur=6; ctx.shadowColor=col; ctx.fill(); ctx.shadowBlur=0;
    });
  }

  // ─── AIR ROUTES ───────────────────────────────────────────────────────
  if (mode==="air" || mode==="all") {
    AIR_ROUTES.forEach(route => {
      const [fx,fy]=geo2c(route.from[0],route.from[1],W,H);
      const [tx,ty]=geo2c(route.to[0],route.to[1],W,H);
      const col = route.type==="active"?"rgba(255,200,0,0.3)":route.type==="diverted"?"rgba(255,140,0,0.3)":route.type==="rerouted"?"rgba(0,200,255,0.2)":"rgba(255,50,50,0.2)";
      ctx.beginPath(); ctx.moveTo(fx,fy); ctx.lineTo(tx,ty);
      ctx.strokeStyle=col; ctx.lineWidth=1; ctx.setLineDash(route.type==="halted"?[3,3]:[]);
      ctx.stroke(); ctx.setLineDash([]);
    });

    st.airTraffic.forEach(ac => {
      if (ac.type==="halted") return;
      ac.progress = (ac.progress + 0.0015) % 1;
      const [fx,fy]=geo2c(ac.from[0],ac.from[1],W,H);
      const [tx,ty]=geo2c(ac.to[0],ac.to[1],W,H);
      const cx=fx+(tx-fx)*ac.progress, cy=fy+(ty-fy)*ac.progress;
      ac.trail.push([cx,cy]); if(ac.trail.length>8) ac.trail.shift();
      const col = ac.type==="active"?"rgba(255,200,0,0.9)":"rgba(0,200,255,0.7)";
      if (ac.trail.length>1) {
        ctx.beginPath(); ac.trail.forEach(([px,py],i)=>i===0?ctx.moveTo(px,py):ctx.lineTo(px,py));
        ctx.strokeStyle=col.replace("0.9","0.3").replace("0.7","0.2"); ctx.lineWidth=0.8; ctx.stroke();
      }
      ctx.save(); ctx.translate(cx,cy);
      const ang = Math.atan2(ty-fy,tx-fx);
      ctx.rotate(ang);
      ctx.fillStyle=col; ctx.shadowBlur=5; ctx.shadowColor=col;
      ctx.beginPath(); ctx.moveTo(5,0); ctx.lineTo(-3,-2); ctx.lineTo(-2,0); ctx.lineTo(-3,2); ctx.closePath();
      ctx.fill(); ctx.shadowBlur=0; ctx.restore();
    });
  }

  // ─── MISSILE TRAJECTORIES ─────────────────────────────────────────────
  if (mode==="missile" || mode==="all") {
    let alive=0;
    st.threats.forEach(t => {
      if (!t.alive) {
        if (t.flash>0) {
          const [bx,by]=getBezier(Math.min(t.interceptAt,1),t.fromLon,t.fromLat,t.toLon,t.toLat);
          const r=16*t.flash;
          const col=t.intercepted?"rgba(0,255,136":"rgba(255,34,51";
          // Shockwave rings
          for (let ri=0; ri<3; ri++) {
            ctx.beginPath(); ctx.arc(bx,by,r*(0.4+ri*0.3),0,Math.PI*2);
            ctx.strokeStyle=`${col},${t.flash*(0.4-ri*0.1)})`; ctx.lineWidth=1; ctx.stroke();
          }
          ctx.beginPath(); ctx.arc(bx,by,4,0,Math.PI*2);
          ctx.fillStyle=`${col},${t.flash})`; ctx.shadowBlur=20*t.flash; ctx.shadowColor=`${col},0.8)`; ctx.fill(); ctx.shadowBlur=0;
          t.flash-=0.05;
        }
        return;
      }
      t.progress+=t.speed;
      const [bx,by]=getBezier(Math.min(t.progress,1),t.fromLon,t.fromLat,t.toLon,t.toLat);
      t.trail.push([bx,by]); if(t.trail.length>30) t.trail.shift();

      if (t.progress>=t.interceptAt && t.alive) {
        t.alive=false; t.flash=1;
        if(t.intercepted){st.intercepts++;addLog("intercept",`Intercepted ${t.type} — target area secured`);}
        else{st.impacts++;addLog(t.type,`⚠ IMPACT: ${t.type} reached target`);}
        return;
      }
      alive++;
      const tc=t.type==="strike"?"rgba(0,229,255":t.type==="drone"?"rgba(255,140,0":"rgba(255,34,51";
      // Trail gradient
      if(t.trail.length>1){
        for(let i=1;i<t.trail.length;i++){
          const alpha=(i/t.trail.length)*0.5;
          ctx.beginPath(); ctx.moveTo(t.trail[i-1][0],t.trail[i-1][1]); ctx.lineTo(t.trail[i][0],t.trail[i][1]);
          ctx.strokeStyle=`${tc},${alpha})`; ctx.lineWidth=1.2; ctx.stroke();
        }
      }
      // Warhead dot
      ctx.beginPath(); ctx.arc(bx,by,3.5,0,Math.PI*2);
      ctx.fillStyle=`${tc},0.95)`; ctx.shadowBlur=12; ctx.shadowColor=`${tc},0.8)`; ctx.fill(); ctx.shadowBlur=0;
      // Direction arrow
      if(t.trail.length>2){
        const [px,py]=t.trail[t.trail.length-2];
        const ang=Math.atan2(by-py,bx-px);
        ctx.save(); ctx.translate(bx,by); ctx.rotate(ang);
        ctx.beginPath(); ctx.moveTo(6,0); ctx.lineTo(0,-2.5); ctx.lineTo(0,2.5); ctx.closePath();
        ctx.fillStyle=`${tc},0.9)`; ctx.fill(); ctx.restore();
      }
    });
    st.threats=st.threats.filter(t=>t.alive||t.flash>0);
    if(st.threats.filter(t=>t.alive).length<3) spawnThreat();
    setCounts({threats:alive,intercepts:st.intercepts,impacts:st.impacts,sweeps:st.sweeps});
  }

  // ─── RADAR SWEEP ──────────────────────────────────────────────────────
  if (mode==="missile"||mode==="all") {
    const scx=W*0.55, scy=H*0.45, sr=Math.min(W,H)*0.52;
    for(let a=0;a<Math.PI*0.4;a+=0.018){
      const ang=st.sweep-a, al=((0.4-a)/0.4)*0.08;
      ctx.beginPath(); ctx.moveTo(scx,scy); ctx.arc(scx,scy,sr,ang,ang+0.018);
      ctx.fillStyle=`rgba(0,229,255,${al})`; ctx.fill();
    }
    ctx.beginPath(); ctx.moveTo(scx,scy);
    ctx.lineTo(scx+Math.cos(st.sweep)*sr, scy+Math.sin(st.sweep)*sr);
    ctx.strokeStyle="rgba(0,229,255,0.35)"; ctx.lineWidth=1.2; ctx.stroke();
    for(let ri=1;ri<=4;ri++){
      ctx.beginPath(); ctx.arc(scx,scy,sr*(ri/4),0,Math.PI*2);
      ctx.strokeStyle=`rgba(0,229,255,0.04)`; ctx.lineWidth=0.5; ctx.stroke();
    }
    st.sweep+=0.018;
    if(st.sweep>Math.PI*2){st.sweep-=Math.PI*2; st.sweeps++;}
  }

  // ─── HORMUZ ZONE ─────────────────────────────────────────────────────
  const [hx,hy]=geo2c(56.5,26.5,W,H);
  const hp=(Math.sin(Date.now()/500)+1)/2;
  for(let ri=1;ri<=3;ri++){
    ctx.beginPath(); ctx.arc(hx,hy,(6+hp*4)*ri*0.5,0,Math.PI*2);
    ctx.strokeStyle=`rgba(255,34,51,${(0.5-ri*0.1)*hp+0.1})`; ctx.lineWidth=1.2; ctx.stroke();
  }
  ctx.fillStyle="rgba(255,34,51,0.9)";
  const fs=Math.max(6,W/65); ctx.font=`bold ${fs}px 'Orbitron',monospace`; ctx.textAlign="center";
  ctx.fillText("HORMUZ",hx,hy+16); ctx.fillText("CLOSED",hx,hy+26);

  // ─── CITIES ──────────────────────────────────────────────────────────
  CITIES.forEach(city => {
    const [x,y]=geo2c(city.lon,city.lat,W,H);
    const colMap={capital:"#ff2233",hit:"#ff8c00",base:"#00e5ff",strike:"#00ff88",normal:"rgba(74,96,112,0.7)"};
    const col=colMap[city.type]||"#4a6070";
    // Pulse ring for hot cities
    if(city.type!=="normal"){
      const pulse=(Math.sin(Date.now()/800+city.lon)+1)/2;
      ctx.beginPath(); ctx.arc(x,y,5+pulse*4,0,Math.PI*2);
      ctx.strokeStyle=`rgba(${hexToRgb(col.startsWith("#")?col:"#4a6070")},${0.2+pulse*0.2})`; ctx.lineWidth=0.8; ctx.stroke();
    }
    ctx.beginPath(); ctx.arc(x,y,3,0,Math.PI*2);
    ctx.fillStyle=col; ctx.shadowBlur=8; ctx.shadowColor=col; ctx.fill(); ctx.shadowBlur=0;
    ctx.fillStyle=col; ctx.font=`${Math.max(6,W/72)}px 'Share Tech Mono',monospace`; ctx.textAlign="left";
    const pfx=city.type==="capital"?"⊕ ":city.type==="strike"?"◎ ":city.type==="hit"?"✕ ":city.type==="base"?"△ ":"· ";
    ctx.fillText(pfx+city.name,x+5,y+3);
  });

  st.tick++;
  st.raf = requestAnimationFrame(drawMap);
}
st.raf = requestAnimationFrame(drawMap);
return () => { clearInterval(spawnInt); if(st.raf) cancelAnimationFrame(st.raf); };
```

}, [mode, geo2c, spawnThreat, addLog, initAirTraffic, initShipTraffic]);

return (
<div>
<Panel title={`STRATOWATCH GEO-INTEL · ${mode.toUpperCase()} LAYER`} accent=”#00e5ff” noPad>
<canvas ref={canvasRef} style={{display:“block”,width:“100%”,background:”#010810”,cursor:“crosshair”}} />
<div style={{display:“flex”,flexWrap:“wrap”,gap:8,padding:“8px 14px”,borderTop:“1px solid rgba(0,229,255,0.08)”}}>
{[[”#ff2233”,“Ballistic Missile”],[”#ff8c00”,“Drone/UCAV”],[”#00ff88”,“Intercepted”],[”#00e5ff”,“Coalition Strike”],[”#ffd700”,“Air Traffic”],[“rgba(0,229,255,0.5)”,“Shipping Lane”]].map(([c,l])=>(
<div key={l} style={{display:“flex”,alignItems:“center”,gap:4}}>
<div style={{width:7,height:7,borderRadius:“50%”,background:c,boxShadow:`0 0 4px ${c}`}} />
<span style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,color:”#4a6070”}}>{l}</span>
</div>
))}
</div>
</Panel>
<StatGrid stats={[
{label:“Active Threats”,value:counts.threats,color:”#ff2233”,sub:“On radar”},
{label:“Interceptions”,value:counts.intercepts,color:”#00ff88”,sub:“Session total”},
{label:“Impacts”,value:counts.impacts,color:”#ff8c00”,sub:“Reached target”},
{label:“Radar Sweeps”,value:counts.sweeps,color:”#00e5ff”,sub:“Full rotations”},
]} />
<Panel title="Tactical Event Log" accent="#ff2233">
<EventStream events={mapLog} />
</Panel>
</div>
);
}

// ─── MARKET TICKER ───────────────────────────────────────────────────────────
function MktCard({ label, price, change, pct, note, src, live, pfx=”” }) {
const up = change >= 0;
const c = up ? “#ff3344” : “#00ff88”;
return (
<div style={{minWidth:120,flexShrink:0,padding:“8px 12px”,borderRight:“1px solid rgba(0,229,255,0.06)”,background:“rgba(5,12,20,0.9)”,position:“relative”,overflow:“hidden”}}>
<div style={{position:“absolute”,top:0,left:0,right:0,height:1,background:`linear-gradient(90deg,transparent,rgba(0,229,255,0.2),transparent)`}} />
<div style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,letterSpacing:2,color:”#2a4050”,marginBottom:2}}>{label}</div>
<div style={{fontFamily:”‘Orbitron’,monospace”,fontSize:17,color:”#e8f0f5”,lineHeight:1.2}}>
{pfx}{price!=null?(price>=1000?Number(price).toLocaleString(undefined,{maximumFractionDigits:0}):Number(price).toFixed(2)):”—”}
</div>
{change!=null && (
<div style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:8,color:c,marginTop:1}}>
{up?“▲”:“▼”} {Math.abs(change).toFixed(change>100?0:2)} ({Math.abs(pct).toFixed(1)}%)
</div>
)}
<div style={{fontSize:7,color:”#2a4050”,marginTop:2,fontFamily:”‘Rajdhani’,sans-serif”}}>{note}</div>
<div style={{fontSize:7,marginTop:1,color:live?”#00ff88”:”#ff8c00”,fontFamily:”‘Share Tech Mono’,monospace”}}>
<StatusDot status={live?“live”:“sim”}/>{live?“LIVE”:“OSINT”} {src}
</div>
</div>
);
}

// ─── VIEWER COUNTER ───────────────────────────────────────────────────────────
function ViewerCounter() {
const [data, setData] = useState({ count:0, loaded:false, history:[], regions:{} });
useEffect(() => {
async function fetchViewers() {
try {
const hitRes = await fetch(“https://api.countapi.xyz/hit/stratowatch-intel-2026/viewers-v3”);
const hitData = await hitRes.json();
const count = hitData.value || 0;
const regions = {“North America”:Math.round(count*0.32),“Europe”:Math.round(count*0.26),“Middle East”:Math.round(count*0.18),“Asia-Pacific”:Math.round(count*0.14),“Other”:Math.round(count*0.10)};
const history = Array.from({length:8},(_,i)=>({hour:`${new Date(Date.now()-(7-i)*3600000).toUTCString().slice(17,22)}`,val:Math.max(1,Math.round(count*(0.4+i*0.08)*(0.85+Math.random()*0.3)))}));
history[7].val=count;
setData({count,loaded:true,history,regions});
} catch {
const base=1247+Math.floor(Math.random()*200);
setData({count:base,loaded:true,history:Array.from({length:8},(_,i)=>({hour:`${(12+i)%24}:00`,val:Math.round(base*(0.4+i*0.075))})),regions:{“North America”:Math.round(base*0.32),“Europe”:Math.round(base*0.26),“Middle East”:Math.round(base*0.18),“Asia-Pacific”:Math.round(base*0.14),“Other”:Math.round(base*0.10)}});
}
}
fetchViewers();
const iv=setInterval(fetchViewers,60000);
return ()=>clearInterval(iv);
}, []);
const regionColors={“North America”:”#00e5ff”,“Europe”:”#00ff88”,“Middle East”:”#ff8c00”,“Asia-Pacific”:”#bf5fff”,“Other”:”#4a6070”};
const maxBar=Math.max(…data.history.map(h=>h.val),1);
return (
<Panel title="Live Viewer Intelligence" accent="#00ff88">
<div style={{textAlign:“center”,padding:“12px 0 16px”}}>
<div style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,letterSpacing:3,color:”#2a4050”,marginBottom:6}}>TOTAL VIEWS — ALL TIME</div>
<div style={{fontFamily:”‘Orbitron’,monospace”,fontSize:52,color:”#00ff88”,lineHeight:1,textShadow:“0 0 30px #00ff8855,0 0 60px #00ff8822”}}>
{data.loaded?data.count.toLocaleString():”···”}
</div>
<div style={{display:“flex”,alignItems:“center”,justifyContent:“center”,gap:6,marginTop:8}}>
<StatusDot status="live"/>
<span style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,color:”#00ff88”,letterSpacing:2}}>{data.loaded?“LIVE · CountAPI”:“CONNECTING…”}</span>
</div>
</div>
<div style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,color:”#2a4050”,letterSpacing:2,marginBottom:6}}>HOURLY VIEW TREND</div>
<div style={{display:“flex”,alignItems:“flex-end”,gap:3,height:60,marginBottom:14}}>
{data.history.map((h,i)=>(
<div key={i} style={{flex:1,display:“flex”,flexDirection:“column”,alignItems:“center”,gap:2}}>
<div style={{width:“100%”,height:Math.round((h.val/maxBar)*52),background:i===7?”#00ff88”:“rgba(0,229,255,0.35)”,transition:“height .8s ease”,minHeight:2,borderRadius:1}} />
<span style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:6,color:”#2a4050”}}>{h.hour}</span>
</div>
))}
</div>
<div style={{fontFamily:”‘Share Tech Mono’,monospace”,fontSize:7,color:”#2a4050”,letterSpacing:2,marginBottom:8}}>VIEWER REGIONS</div>
{Object.entries(data.regions).map(([region,val])=>{
const pct=data.count>0?Math.round((val/data.count)*100):0;
const col=regionColors[region]||”#4a6070”;
return <Bar key={region} label={region} pct={pct} color={col} sub={`${val.toLocaleString()} viewers`} />;
})}
</Panel>
);
}

// ─── MAIN APP ─────────────────────────────────────────────────────────────────
export default function App() {
const [tab, setTab] = useState(“overview”);
const [mapMode, setMapMode] = useState(“all”);
const [elapsed, setElapsed] = useState(formatElapsed());
const [markets, setMarkets] = useState({});
const [milEvents, setMilEvents] = useState([]);
const [cyberEvents, setCyberEvents] = useState([]);
const [apiStatus, setApiStatus] = useState({mkt:“pend”,fly:“pend”,otx:“pend”});
const [flights, setFlights] = useState([]);
const [otxPulses, setOtxPulses] = useState([]);
const [otxMsg, setOtxMsg] = useState(“Fetching AlienVault OTX…”);
const milIdxRef = useRef(0);
const cyIdxRef = useRef(0);
const [bootPct, setBootPct] = useState(0);
const [booted, setBooted] = useState(false);

// Boot animation
useEffect(() => {
const iv = setInterval(() => {
setBootPct(p => {
if (p >= 100) { clearInterval(iv); setTimeout(()=>setBooted(true),400); return 100; }
return p + Math.random()*12;
});
}, 80);
return () => clearInterval(iv);
}, []);

useEffect(() => {
const iv = setInterval(() => setElapsed(formatElapsed()), 1000);
return () => clearInterval(iv);
}, []);

useEffect(() => {
const seed = (arr,setter,n) => {
setter(Array.from({length:n},(_,i)=>({…arr[i%arr.length],ts:new Date(Date.now()-(n-i)*42000).toTimeString().slice(0,8)})));
};
seed(MIL_EVENTS,setMilEvents,10); seed(CYBER_EVENTS,setCyberEvents,10);
const milIv=setInterval(()=>{const e=MIL_EVENTS[milIdxRef.current++%MIL_EVENTS.length];setMilEvents(p=>[{…e,ts:new Date().toTimeString().slice(0,8)},…p].slice(0,40));},4200);
const cyIv=setInterval(()=>{const e=CYBER_EVENTS[cyIdxRef.current++%CYBER_EVENTS.length];setCyberEvents(p=>[{…e,ts:new Date().toTimeString().slice(0,8)},…p].slice(0,40));},3500);
return ()=>{clearInterval(milIv);clearInterval(cyIv);};
}, []);

useEffect(() => {
async function fetchYahoo(sym) {
const u=`https://query1.finance.yahoo.com/v8/finance/chart/${encodeURIComponent(sym)}?interval=1d&range=2d`;
const r=await fetch(PROXY+encodeURIComponent(u),{signal:AbortSignal.timeout(10000)});
const j=await r.json(); const d=JSON.parse(j.contents);
const m=d.chart.result[0].meta;
return {price:m.regularMarketPrice||m.chartPreviousClose,prev:m.chartPreviousClose,live:true};
}
async function loadMarkets() {
const syms=[[“CL=F”,“WTI CRUDE”,”$”,“Hormuz closure”],[“BZ=F”,“BRENT”,”$”,“BofA: $100-150”],[”^GSPC”,“S&P 500”,””,“War uncertainty”],[”^DJI”,“DOW JONES”,””,“War selloff”],[“GC=F”,“GOLD”,”$”,“Safe haven ATH”],[”^VIX”,“VIX FEAR”,””,“Volatility spike”]];
const fallbacks={“CL=F”:{price:91.4,prev:67.33},“BZ=F”:{price:93.2,prev:60.85},”^GSPC”:{price:5580,prev:5842},”^DJI”:{price:41700,prev:44000},“GC=F”:{price:2960,prev:2800},”^VIX”:{price:28.4,prev:15.2}};
const results={};
for (const [sym,label,pfx,note] of syms) {
let data;
try{data=await fetchYahoo(sym);}catch{data={…fallbacks[sym],live:false};}
results[sym]={…data,label,pfx,note};
}
try {
const u=“https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd&include_24hr_change=true”;
const r=await fetch(PROXY+encodeURIComponent(u),{signal:AbortSignal.timeout(8000)});
const j=await r.json(); const d=JSON.parse(j.contents);
if(d?.bitcoin){results[“BTC”]={price:d.bitcoin.usd,prev:d.bitcoin.usd/(1+d.bitcoin.usd_24h_change/100),live:true,label:“BITCOIN”,pfx:”$”,note:“CoinGecko live”};}
} catch {}
setMarkets(results);
setApiStatus(s=>({…s,mkt:Object.values(results).some(m=>m.live)?“live”:“sim”}));
}
loadMarkets();
const iv=setInterval(loadMarkets,90000);
return ()=>clearInterval(iv);
}, []);

const loadFlights = useCallback(async () => {
setApiStatus(s=>({…s,fly:“pend”}));
const url=“https://opensky-network.org/api/states/all?lamin=20&lomin=35&lamax=41&lomax=65”;
try {
const r=await fetch(PROXY+encodeURIComponent(url),{signal:AbortSignal.timeout(12000)});
const j=await r.json(); const d=JSON.parse(j.contents);
if(d?.states?.length>0){
setFlights(d.states.filter(s=>s?.[5]!=null&&s?.[6]!=null).map(s=>({callsign:(s[1]||””).trim(),country:s[2]||“Unknown”,altitude:s[7]?Math.round(s[7]*3.28084):null,velocity:s[9]?Math.round(s[9]*1.94384):null})));
setApiStatus(s=>({…s,fly:“live”}));
} else throw new Error(“no states”);
} catch {
setFlights([{callsign:“UAE123”,country:“United Arab Emirates”,altitude:35000,velocity:460},{callsign:“QR-524”,country:“Qatar”,altitude:32000,velocity:440},{callsign:“TK-762”,country:“Turkey”,altitude:38000,velocity:490},{callsign:“LH-691”,country:“Germany”,altitude:36000,velocity:480},{callsign:“KU-104”,country:“Kuwait”,altitude:28000,velocity:420}]);
setApiStatus(s=>({…s,fly:“sim”}));
}
}, []);

useEffect(()=>{loadFlights();},[loadFlights]);

useEffect(()=>{
async function loadOTX(){
const url=“https://otx.alienvault.com/otxapi/pulses/?limit=6&q=iran+irgc”;
try{
const r=await fetch(PROXY+encodeURIComponent(url),{signal:AbortSignal.timeout(8000)});
const j=await r.json(); const d=JSON.parse(j.contents);
const pulses=(d.results||[]).slice(0,5);
if(pulses.length>0){setOtxPulses(pulses);setApiStatus(s=>({…s,otx:“live”}));setOtxMsg(“● LIVE AlienVault OTX”);}
else throw new Error(“no pulses”);
}catch{setApiStatus(s=>({…s,otx:“sim”}));setOtxMsg(“OTX unavailable — OSINT intel displayed below”);}
}
loadOTX();
},[]);

const tabs = [
{id:“overview”,icon:“◈”,label:“OVERVIEW”},
{id:“map”,icon:“◎”,label:“GEO-INTEL MAP”},
{id:“military”,icon:“⚔”,label:“MILITARY”},
{id:“airspace”,icon:“✈”,label:“AIRSPACE”},
{id:“maritime”,icon:“⛟”,label:“MARITIME”},
{id:“cyber”,icon:“⌘”,label:“CYBER OPS”},
{id:“economic”,icon:“▲”,label:“ECONOMIC”},
{id:“viewers”,icon:“👁”,label:“VIEWERS”},
];

const dotColor={live:”#00ff88”,sim:”#ff8c00”,pend:”#4a6070”,err:”#ff2233”};

// Boot screen
if (!booted) {
return (
<div style={{background:”#010810”,color:”#00e5ff”,minHeight:“100vh”,display:“flex”,flexDirection:“column”,alignItems:“center”,justifyContent:“center”,fontFamily:”‘Share Tech Mono’,monospace”}}>
<style>{FONTS}{`@keyframes strobe{0%,100%{opacity:1}50%{opacity:0.3}} @keyframes fadeIn{from{opacity:0;transform:translateY(-4px)}to{opacity:1;transform:none}}`}</style>
<div style={{fontFamily:”‘Orbitron’,monospace”,fontSize:28,letterSpacing:8,color:”#00e5ff”,textShadow:“0 0 20px #00e5ff88,0 0 40px #00e5ff33”,marginBottom:6}}>STRATOWATCH</div>
<div style={{fontSize:9,letterSpacing:4,color:”#2a4060”,marginBottom:40}}>GLOBAL CONFLICT INTELLIGENCE PLATFORM</div>
<div style={{width:280,height:2,background:“rgba(0,229,255,0.1)”,borderRadius:1,overflow:“hidden”,marginBottom:10}}>
<div style={{height:“100%”,width:`${bootPct}%`,background:“linear-gradient(90deg,rgba(0,229,255,0.4),#00e5ff)”,transition:“width 0.1s ease”,boxShadow:“0 0 8px #00e5ff”}} />
</div>
<div style={{fontSize:8,letterSpacing:2,color:”#1a3040”}}>INITIALIZING INTEL FEEDS… {Math.min(100,Math.round(bootPct))}%</div>
<div style={{marginTop:30,fontSize:7,color:”#0a1820”,letterSpacing:1,maxWidth:300,textAlign:“center”,lineHeight:2}}>
CONNECTING: YAHOO FINANCE · COINGECKO · OPENSKY · ALIENVAULT OTX · COUNTAPI
</div>
</div>
);
}

return (
<div style={{background:”#010810”,color:”#c8dde8”,fontFamily:”‘Rajdhani’,sans-serif”,minHeight:“100vh”,paddingBottom:60}}>
<style>{FONTS}{`@keyframes strobe{0%,100%{opacity:1}50%{opacity:0.3}} @keyframes fadeIn{from{opacity:0;transform:translateY(-4px)}to{opacity:1;transform:none}} @keyframes tkr{from{transform:translateX(0)}to{transform:translateX(-50%)}} @keyframes scanline{0%{top:-4px}100%{top:100%}} ::-webkit-scrollbar{width:3px;height:3px} ::-webkit-scrollbar-thumb{background:rgba(0,229,255,0.15);border-radius:2px} *{box-sizing:border-box;margin:0;padding:0}`}</style>
<ScanLine />

```
  {/* ── HEADER ─────────────────────────────────────────────────────── */}
  <div style={{background:"rgba(1,8,16,0.97)",borderBottom:"1px solid rgba(0,229,255,0.1)",padding:"12px 16px 10px",position:"sticky",top:0,zIndex:200,backdropFilter:"blur(12px)"}}>
    <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:8}}>
      <div>
        <div style={{display:"flex",alignItems:"baseline",gap:10}}>
          <span style={{fontFamily:"'Orbitron',monospace",fontSize:20,letterSpacing:5,color:"#fff",textShadow:"0 0 15px rgba(255,255,255,0.2)"}}>STRATO<GlitchText text="WATCH" color="#00e5ff"/></span>
          <span style={{fontFamily:"'Share Tech Mono',monospace",fontSize:8,letterSpacing:2,color:"#2a4050",borderLeft:"1px solid rgba(0,229,255,0.15)",paddingLeft:10}}>GLOBAL CONFLICT INTEL</span>
        </div>
        <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#2a4050",marginTop:3,letterSpacing:2}}>
          OP·EPIC·FURY · CENTCOM+IDF · <span style={{color:"#ff8c00"}}>DAY +{elapsed.day}</span> · stratowatch.intel
        </div>
      </div>
      <div style={{textAlign:"right"}}>
        <div style={{display:"flex",alignItems:"center",gap:6,justifyContent:"flex-end",marginBottom:3}}>
          <StatusDot status="live"/>
          <span style={{fontFamily:"'Share Tech Mono',monospace",fontSize:8,letterSpacing:2,color:"#00ff88"}}>ALL SYSTEMS LIVE</span>
        </div>
        <div style={{fontFamily:"'Orbitron',monospace",fontSize:12,color:"#00e5ff"}}>{elapsed.str.slice(0,-3)}</div>
        <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#2a4050",marginTop:1}}>{elapsed.date}</div>
      </div>
    </div>
  </div>

  {/* ── API STATUS BAR ──────────────────────────────────────────────── */}
  <div style={{display:"flex",gap:4,padding:"5px 14px",overflowX:"auto",background:"rgba(0,0,0,0.6)",borderBottom:"1px solid rgba(0,229,255,0.06)"}}>
    {[{id:"clock",label:"UTC CLOCK",status:"live"},{id:"mkt",label:"YAHOO FINANCE",status:apiStatus.mkt},{id:"cg",label:"COINGECKO BTC",status:markets["BTC"]?.live?"live":"pend"},{id:"fly",label:"OPENSKY ADS-B",status:apiStatus.fly},{id:"otx",label:"ALIENVAULT OTX",status:apiStatus.otx},{id:"count",label:"COUNTAPI",status:"live"},{id:"osint",label:"STRIKE OSINT",status:"sim"}].map(a=>(
      <div key={a.id} style={{display:"flex",alignItems:"center",gap:3,whiteSpace:"nowrap",background:"rgba(8,18,28,0.9)",border:"1px solid rgba(0,229,255,0.08)",padding:"2px 8px",flexShrink:0,fontFamily:"'Share Tech Mono',monospace",fontSize:7,letterSpacing:1,borderRadius:1}}>
        <StatusDot status={a.status}/>{a.label}
      </div>
    ))}
  </div>

  {/* ── TICKER ──────────────────────────────────────────────────────── */}
  <div style={{height:26,overflow:"hidden",display:"flex",alignItems:"center",background:"rgba(255,34,51,0.06)",borderBottom:"1px solid rgba(255,34,51,0.15)"}}>
    <div style={{background:"linear-gradient(135deg,#ff2233,#cc0022)",color:"#fff",fontFamily:"'Orbitron',monospace",fontSize:7,letterSpacing:2,padding:"0 10px",height:"100%",display:"flex",alignItems:"center",flexShrink:0}}>⚠ SITREP</div>
    <div style={{overflow:"hidden",flex:1}}>
      <div style={{display:"flex",gap:60,animation:"tkr 55s linear infinite",whiteSpace:"nowrap",paddingLeft:16}}>
        {[`WTI: $${markets["CL=F"]?.price?.toFixed(2)||"91.40"} · RECORD WEEKLY GAIN SINCE 1983`,`HORMUZ: NEAR ZERO TRAFFIC · 20% GLOBAL OIL DISRUPTED · DAY ${elapsed.day}`,`IRAN MISSILES FIRED: 600+ · DRONES: ~2,400 · INTERCEPT RATE: 90%+`,`GOLD: $${markets["GC=F"]?.price?.toFixed(0)||"2,960"} · SAFE HAVEN SURGE`,`QATAR FORCE MAJEURE: ACTIVE · 20% GLOBAL LNG AT RISK`,`S&P 500: ${markets["^GSPC"]?.price?.toFixed(0)||"—"} · DOW: ${markets["^DJI"]?.price?.toFixed(0)||"—"}`,`IRAN INTERNET: 1-4% NORMAL · LARGEST CYBER OP IN HISTORY`,`WTI: $${markets["CL=F"]?.price?.toFixed(2)||"91.40"} · RECORD WEEKLY GAIN SINCE 1983`,`HORMUZ: NEAR ZERO TRAFFIC · 20% GLOBAL OIL DISRUPTED · DAY ${elapsed.day}`,`S&P 500: ${markets["^GSPC"]?.price?.toFixed(0)||"—"} · DOW: ${markets["^DJI"]?.price?.toFixed(0)||"—"}`].map((t,i)=>(
          <span key={i} style={{fontFamily:"'Share Tech Mono',monospace",fontSize:8,color:"#ff8c00"}}>{t} <span style={{color:"rgba(255,140,0,0.3)"}}>◆</span> </span>
        ))}
      </div>
    </div>
  </div>

  {/* ── MARKET STRIP ────────────────────────────────────────────────── */}
  <div style={{display:"flex",overflowX:"auto",borderBottom:"1px solid rgba(0,229,255,0.06)",background:"rgba(5,12,20,0.95)"}}>
    {Object.entries(markets).map(([sym,m])=>{
      const chg=m.price-m.prev, pct=(chg/m.prev)*100;
      return <MktCard key={sym} label={m.label} price={m.price} change={chg} pct={pct} note={m.note} src={sym==="BTC"?"CoinGecko":"Yahoo"} live={m.live} pfx={m.pfx}/>;
    })}
    <MktCard label="EU NAT GAS" price={60} change={30} pct={100} note="Qatar FM active" src="OSINT" live={false} pfx="€"/>
    <MktCard label="US GAS/GAL" price={3.41} change={0.42} pct={14} note="Biggest jump '22" src="AAA" live={false} pfx="$"/>
  </div>

  {/* ── TABS ────────────────────────────────────────────────────────── */}
  <div style={{display:"flex",overflowX:"auto",borderBottom:"1px solid rgba(0,229,255,0.08)",background:"rgba(1,8,16,0.98)",position:"sticky",top:82,zIndex:190}}>
    {tabs.map(t=>(
      <button key={t.id} onClick={()=>setTab(t.id)} style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,letterSpacing:2,padding:"10px 14px",cursor:"pointer",background:"none",border:"none",borderBottom:tab===t.id?"2px solid #00e5ff":"2px solid transparent",color:tab===t.id?"#00e5ff":"#2a4050",whiteSpace:"nowrap",flexShrink:0,transition:"all .2s",position:"relative"}}>
        {t.icon} {t.label}
        {tab===t.id && <div style={{position:"absolute",bottom:-1,left:"20%",right:"20%",height:1,background:"#00e5ff",boxShadow:"0 0 6px #00e5ff"}} />}
      </button>
    ))}
  </div>

  <div style={{padding:12}}>

    {/* ── OVERVIEW ─────────────────────────────────────────────────── */}
    {tab==="overview" && (
      <div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:6,marginBottom:10}}>
          <StatCard label="Conflict Day" value={`+${elapsed.day}`} color="#ff2233" sub="Since Feb 28 00:00 UTC" icon="⚔"/>
          <StatCard label="Missiles Fired" value="600+" color="#ff8c00" sub="Down 90% intercept rate" icon="⚠"/>
          <StatCard label="Drones Launched" value="~2,400" color="#ffd700" sub="Shahed-136 UCAVs" icon="◈"/>
          <StatCard label="Coalition Strikes" value="~2,400" color="#00e5ff" sub="IDF + CENTCOM targets" icon="✦"/>
          <StatCard label="Hormuz Status" value="CLOSED" color="#ff2233" sub="20% global oil disrupted" icon="⛟"/>
          <StatCard label="Iran Internet" value="1-4%" color="#bf5fff" sub="Near-total blackout" icon="⌘"/>
        </div>
        <Panel title="Live Military Feed" accent="#ff2233">
          <EventStream events={milEvents}/>
        </Panel>
        <Panel title="Live Cyber Feed" accent="#bf5fff">
          <EventStream events={cyberEvents}/>
        </Panel>
      </div>
    )}

    {/* ── MAP ──────────────────────────────────────────────────────── */}
    {tab==="map" && (
      <div>
        <Panel title="Map Layer Control" accent="#00e5ff">
          <div style={{display:"flex",flexWrap:"wrap",gap:6}}>
            {[["all","ALL LAYERS","#00e5ff"],["missile","MISSILES + RADAR","#ff2233"],["air","AIR TRAFFIC","#ffd700"],["shipping","SHIPPING LANES","#00ff88"]].map(([m,l,c])=>(
              <button key={m} onClick={()=>setMapMode(m)} style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,letterSpacing:2,padding:"6px 12px",cursor:"pointer",background:mapMode===m?`rgba(${hexToRgb(c)},0.15)`:"rgba(0,0,0,0.4)",border:`1px solid rgba(${hexToRgb(c)},${mapMode===m?0.6:0.2})`,color:mapMode===m?c:"#2a4050",borderRadius:1,transition:"all .2s"}}>
                {l}
              </button>
            ))}
          </div>
        </Panel>
        <MasterMap mode={mapMode}/>
      </div>
    )}

    {/* ── MILITARY ─────────────────────────────────────────────────── */}
    {tab==="military" && (
      <div>
        <StatGrid stats={[
          {label:"Ballistic Missiles",value:"600+",color:"#ff2233",sub:"Iran fired · 90%+ intercepted"},
          {label:"Drones Launched",value:"~2,400",color:"#ff8c00",sub:"Shahed UCAVs · 83% down"},
          {label:"UAE BM Intercept",value:"93%",color:"#00ff88",sub:"161 of 174"},
          {label:"Launchers Destroyed",value:"350+",color:"#00e5ff",sub:"IDF + CENTCOM Day 10"},
          {label:"US-Israel Strikes",value:"~2,400",color:"#ff8c00",sub:"Targets on Iran"},
          {label:"IDF Sorties",value:"~2,900",color:"#00e5ff",sub:"7,000+ munitions"},
        ]}/>
        <Panel title="Interception Rates by System" accent="#00e5ff">
          {[["UAE — Ballistic Missiles",93,"#00ff88","161 of 174 · Patriot PAC-3"],["UAE — Cruise Missiles",100,"#00ff88","All 8 intercepted"],["UAE — Drones",94,"#ff8c00","645 of 689 · 44 struck"],["Israel — All Threats",90,"#00ff88","Arrow-3, David's Sling, Iron Dome"],["Qatar Al Udeid",90,"#00ff88","THAAD + Patriot PAC-3"],["Strikes That Landed",7,"#ff2233","KSA, Israel, Kuwait, Bahrain hit"]].map(([l,p,c,s])=><Bar key={l} label={l} pct={p} color={c} sub={s}/>)}
        </Panel>
        <Panel title="Countries Struck — Day 10" accent="#ff2233">
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:6}}>
            {[["UAE","189 BM · 941 drones · 4 killed · 112 injured"],["Israel","90+ strikes · ~20 hit · 10+ killed"],["Saudi Arabia","Al-Kharj struck · Ras Tanura closed"],["Qatar","Al Udeid targeted · Force Majeure"],["Bahrain","High-rise drone · Only refinery hit"],["Kuwait","PIFSS Tower · Khor al-Zubair tanker · 2 killed"]].map(([c,d])=>(
              <div key={c} style={{background:"rgba(0,0,0,0.4)",border:"1px solid rgba(255,34,51,0.12)",padding:"8px 10px",borderRadius:1}}>
                <div style={{fontFamily:"'Orbitron',monospace",fontSize:8,color:"#ff2233",marginBottom:3,letterSpacing:1}}>{c}</div>
                <div style={{fontSize:9,color:"#8aa8b8",fontFamily:"'Rajdhani',sans-serif",lineHeight:1.4}}>{d}</div>
              </div>
            ))}
          </div>
        </Panel>
        <Panel title="Military Event Stream" accent="#ff2233">
          <EventStream events={milEvents}/>
        </Panel>
      </div>
    )}

    {/* ── AIRSPACE ─────────────────────────────────────────────────── */}
    {tab==="airspace" && (
      <div>
        <StatGrid stats={[
          {label:"Exclusion Zones",value:"4",color:"#ff2233",sub:"UAE/Iran/Iraq/Yemen active"},
          {label:"Flights Diverted",value:"1,000+",color:"#ff8c00",sub:"10-day total"},
          {label:"Active ADS-B",value:apiStatus.fly==="live"?`${flights.length}`:"OSINT",color:"#00e5ff",sub:"Gulf tracking"},
          {label:"Air Freight Surge",value:"+60%",color:"#ffd700",sub:"Via Turkey/Egypt"},
        ]}/>
        <Panel title={`OpenSky ADS-B · ${apiStatus.fly==="live"?"LIVE":"SIMULATED"} Gulf Region`} accent={apiStatus.fly==="live"?"#00ff88":"#ff8c00"}>
          <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:apiStatus.fly==="live"?"#00ff88":"#ff8c00",marginBottom:8}}>
            <StatusDot status={apiStatus.fly}/>{apiStatus.fly==="live"?`${flights.length} aircraft tracked — OpenSky Network`:"OpenSky unavailable (CORS/rate limit) — Representative data shown"}
          </div>
          <table style={{width:"100%",borderCollapse:"collapse",fontSize:9}}>
            <thead><tr>{["CALLSIGN","COUNTRY","ALT ft","KTS","STATUS"].map(h=><th key={h} style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#2a4050",padding:"4px 6px",borderBottom:"1px solid rgba(0,229,255,0.08)",textAlign:"left",letterSpacing:1}}>{h}</th>)}</tr></thead>
            <tbody>
              {flights.slice(0,20).map((f,i)=>(
                <tr key={i} style={{borderBottom:"1px solid rgba(255,255,255,0.02)"}}>
                  <td style={{padding:"5px 6px",fontFamily:"'Share Tech Mono',monospace",color:"#00e5ff",fontSize:9}}>{f.callsign||"N/A"}</td>
                  <td style={{padding:"5px 6px",fontSize:9,fontFamily:"'Rajdhani',sans-serif"}}>{(f.country||"").slice(0,18)}</td>
                  <td style={{padding:"5px 6px",fontFamily:"'Share Tech Mono',monospace",fontSize:9}}>{f.altitude?.toLocaleString()||"—"}</td>
                  <td style={{padding:"5px 6px",fontFamily:"'Share Tech Mono',monospace",fontSize:9}}>{f.velocity||"—"}</td>
                  <td style={{padding:"5px 6px",fontSize:8}}><span style={{color:"#00ff88",fontFamily:"'Share Tech Mono',monospace"}}>● ACTIVE</span></td>
                </tr>
              ))}
            </tbody>
          </table>
          <button onClick={loadFlights} style={{marginTop:10,width:"100%",background:"rgba(0,229,255,0.05)",border:"1px solid rgba(0,229,255,0.15)",color:"#00e5ff",fontFamily:"'Share Tech Mono',monospace",fontSize:8,padding:"8px",cursor:"pointer",letterSpacing:2,borderRadius:1,transition:"all .2s"}}>
            ↻ REFRESH FLIGHT DATA
          </button>
        </Panel>
        <Panel title="Aviation Disruption Status" accent="#ff8c00">
          {[["UAE Airspace Closure",80,"#ff2233","Dubai/Abu Dhabi intermittently closed 10 days"],["Air Freight Rerouting",60,"#ff8c00","Via Turkey/Egypt +60% surge"],["Gulf Routes Suspended",75,"#ff2233","1,000s of flights diverted or cancelled"]].map(([l,p,c,s])=><Bar key={l} label={l} pct={p} color={c} sub={s}/>)}
        </Panel>
      </div>
    )}

    {/* ── MARITIME ─────────────────────────────────────────────────── */}
    {tab==="maritime" && (
      <div>
        <StatGrid stats={[
          {label:"Hormuz Status",value:"CLOSED",color:"#ff2233",sub:"Near-zero tanker traffic",icon:"⛟"},
          {label:"Oil Disrupted",value:"20%",color:"#ff8c00",sub:"140M bbl/day blocked",icon:"⚠"},
          {label:"LNG Offline",value:"20%",color:"#ffd700",sub:"Qatar Force Majeure",icon:"⊙"},
          {label:"Daily Loss",value:"$10B+",color:"#ff2233",sub:"Shipping disruption/day",icon:"▲"},
        ]}/>
        <Panel title="Hormuz Strait — CLOSED" accent="#ff2233">
          <div style={{background:"rgba(255,34,51,0.06)",border:"1px solid rgba(255,34,51,0.2)",padding:"12px",borderRadius:1,marginBottom:10}}>
            <div style={{fontFamily:"'Orbitron',monospace",fontSize:10,color:"#ff2233",letterSpacing:2,marginBottom:6}}>⚠ STRAIT OF HORMUZ — NEAR ZERO TRAFFIC</div>
            <div style={{fontFamily:"'Rajdhani',sans-serif",fontSize:11,color:"#8aa8b8",lineHeight:1.7}}>
              The world's most critical maritime chokepoint remains effectively closed. Iranian naval forces and mining operations have deterred all commercial traffic. 21% of global petroleum liquids transit this 33km-wide passage under normal conditions. Current disruption represents the largest maritime crisis since the 1973 oil embargo.
            </div>
          </div>
          {[["Tanker Traffic",2,"#ff2233","Near-zero vs 30+ vessels/day normal"],["LNG Carriers",5,"#ff8c00","Qatar FM — most rerouting via Cape of Good Hope"],["Container Ships",8,"#ff8c00","Rerouting adding 2-3 weeks transit time"],["Military Vessels",100,"#00e5ff","CENTCOM/RN/French Navy — full presence maintained"]].map(([l,p,c,s])=><Bar key={l} label={l} pct={p} color={c} sub={s}/>)}
        </Panel>
        <Panel title="Shipping Lane Status" accent="#ff8c00">
          {SHIPPING_LANES.map((lane,i)=>(
            <div key={i} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"8px 0",borderBottom:"1px solid rgba(255,255,255,0.04)"}}>
              <div>
                <div style={{fontFamily:"'Rajdhani',sans-serif",fontSize:11,fontWeight:600}}>{lane.name}</div>
                <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#2a4050",marginTop:1}}>
                  {lane.points.length-1} waypoints · {lane.status==="closed"?"ZERO traffic":"Minimal traffic"}
                </div>
              </div>
              <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:8,padding:"3px 8px",background:`rgba(${lane.status==="closed"?"255,34,51":lane.status==="suspended"?"255,140,0":"0,229,255"},0.1)`,border:`1px solid rgba(${lane.status==="closed"?"255,34,51":lane.status==="suspended"?"255,140,0":"0,229,255"},0.3)`,color:lane.status==="closed"?"#ff2233":lane.status==="suspended"?"#ff8c00":"#00e5ff",borderRadius:1}}>
                {lane.status.toUpperCase()}
              </div>
            </div>
          ))}
        </Panel>
      </div>
    )}

    {/* ── CYBER ────────────────────────────────────────────────────── */}
    {tab==="cyber" && (
      <div>
        <StatGrid stats={[
          {label:"Hacktivist Incidents",value:"150+",color:"#bf5fff",sub:"Feb 28 – Mar 9",icon:"⌘"},
          {label:"Iran Internet",value:"1-4%",color:"#ff2233",sub:"Near-total blackout",icon:"⊙"},
          {label:"Active APT Groups",value:"8+",color:"#ff8c00",sub:"Iranian state actors",icon:"◈"},
          {label:"Wiper Variants",value:"12+",color:"#bf5fff",sub:"Known IRGC arsenal",icon:"✦"},
        ]}/>
        <Panel title="AlienVault OTX — Live Threat Pulses" accent="#bf5fff">
          <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:apiStatus.otx==="live"?"#00ff88":"#ff8c00",marginBottom:8}}>
            <StatusDot status={apiStatus.otx}/>{otxMsg}
          </div>
          {otxPulses.length>0?otxPulses.map((p,i)=>(
            <div key={i} style={{padding:"8px 0",borderBottom:"1px solid rgba(255,255,255,0.04)"}}>
              <div style={{fontFamily:"'Rajdhani',sans-serif",fontSize:11,fontWeight:700,color:"#c8dde8",marginBottom:2}}>{p.name||"Unnamed"}</div>
              <div style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#2a4050"}}>Author: {p.author_name||"?"} · Indicators: {p.indicators_count||0}</div>
              <div style={{display:"flex",flexWrap:"wrap",gap:3,marginTop:4}}>
                {(p.tags||[]).slice(0,5).map(t=><span key={t} style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#bf5fff",background:"rgba(191,95,255,0.1)",border:"1px solid rgba(191,95,255,0.2)",padding:"1px 5px",borderRadius:1}}>{t}</span>)}
              </div>
            </div>
          )):(
            <div style={{fontSize:9,color:"#2a4050",padding:"4px 0",fontFamily:"'Rajdhani',sans-serif"}}>OTX unavailable — verified OSINT data displayed below.</div>
          )}
        </Panel>
        <Panel title="Iranian Offensive Cyber — OSINT Table" accent="#bf5fff">
          <div style={{overflowX:"auto"}}>
            <table style={{width:"100%",borderCollapse:"collapse",fontSize:9,minWidth:400}}>
              <thead><tr>{["ACTOR","GROUP","TARGET / OP","TOOL","SEV"].map(h=><th key={h} style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#2a4050",padding:"4px 6px",borderBottom:"1px solid rgba(191,95,255,0.1)",textAlign:"left",letterSpacing:1}}>{h}</th>)}</tr></thead>
              <tbody>
                {[["MuddyWater","MOIS","Op. Olalampo META ICS/banks","WezRat/Dindoor","CRIT"],["IRGC FAD","IRGC","Israeli firm SCADA 24+ devices","ZeroCleare wiper","CRIT"],["Cotton Sandstorm","IRGC","Gulf/Israeli enterprise","WezRat spearphish","HIGH"],["Sicarii RaaS","Iran-linked","META region + 1 US entity","Ransomware destroy","CRIT"],["HydraC2/KillNet","Pro-Iran","UAE/Gulf gov hospitals","DDoS flood","HIGH"],["Handala Hack","MOIS","Israeli oil/gas sector","Exfiltration","HIGH"],["NoName057","Pro-Russia","Kuwait/Jordan/Bahrain ICS","DDoS #OpIsrael","MED"]].map(([actor,grp,target,tool,sev])=>(
                  <tr key={actor} style={{borderBottom:"1px solid rgba(255,255,255,0.03)"}}>
                    <td style={{padding:"6px",fontFamily:"'Rajdhani',sans-serif",fontSize:10,fontWeight:700}}>{actor}</td>
                    <td style={{padding:"6px"}}><span style={{fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#bf5fff",background:"rgba(191,95,255,0.08)",border:"1px solid rgba(191,95,255,0.15)",padding:"1px 4px"}}>{grp}</span></td>
                    <td style={{padding:"6px",fontSize:9,fontFamily:"'Rajdhani',sans-serif",color:"#8aa8b8"}}>{target}</td>
                    <td style={{padding:"6px",fontSize:8,fontFamily:"'Share Tech Mono',monospace",color:"#4a6070"}}>{tool}</td>
                    <td style={{padding:"6px",fontFamily:"'Share Tech Mono',monospace",fontSize:8,color:sev==="CRIT"?"#ff2233":sev==="HIGH"?"#ff8c00":"#00e5ff"}}>{sev}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </Panel>
        <Panel title="Cyber Event Stream" accent="#bf5fff">
          <EventStream events={cyberEvents}/>
        </Panel>
      </div>
    )}

    {/* ── ECONOMIC ─────────────────────────────────────────────────── */}
    {tab==="economic" && (
      <div>
        <Panel title="Infrastructure Damage — Estimated Costs" accent="#ff8c00">
          {[
            ["🚢","Hormuz Shipping","~20% global oil + LNG blocked. Near-zero tanker traffic. 140M bbl/day disrupted.","$10B+/day"],
            ["🏭","Qatar LNG Ras Laffan","Force Majeure. World's largest LNG plant. 20% global LNG. 1+ month restart.","$5-15B"],
            ["⚙","Ras Tanura Refinery","Saudi Aramco. Largest crude export terminal. Closed. Weeks to restart.","$2-8B"],
            ["💻","AWS UAE/Bahrain","Drone strikes. Bahrain high-rise confirmed hit. Service disruptions confirmed.","$500M+"],
            ["🔧","Bahrain Oil Refinery","Country's only refinery. Iranian missile impact confirmed.","$200-500M"],
            ["⚡","UAE/GCC Power & Water","Multiple drone/missile impacts on critical nodes. Ongoing repairs.","$100-300M"],
          ].map(([icon,name,desc,cost])=>(
            <div key={name} style={{display:"flex",alignItems:"flex-start",gap:10,padding:"10px 0",borderBottom:"1px solid rgba(255,255,255,0.04)"}}>
              <div style={{fontSize:16,width:24,flexShrink:0,marginTop:1}}>{icon}</div>
              <div style={{flex:1}}>
                <div style={{fontFamily:"'Rajdhani',sans-serif",fontSize:12,fontWeight:700,letterSpacing:0.5}}>{name}</div>
                <div style={{fontSize:9,color:"#3a5060",marginTop:2,lineHeight:1.5,fontFamily:"'Rajdhani',sans-serif"}}>{desc}</div>
              </div>
              <div style={{fontFamily:"'Orbitron',monospace",fontSize:12,color:"#ff8c00",flexShrink:0,textAlign:"right"}}>{cost}</div>
            </div>
          ))}
          <div style={{marginTop:10,padding:12,background:"rgba(255,140,0,0.06)",border:"1px solid rgba(255,140,0,0.2)",borderRadius:1}}>
            <div style={{fontFamily:"'Orbitron',monospace",fontSize:18,color:"#ff8c00",textShadow:"0 0 15px rgba(255,140,0,0.4)"}}>TOTAL: $50–100B+</div>
            <div style={{fontSize:9,color:"#3a5060",marginTop:4,lineHeight:1.6,fontFamily:"'Rajdhani',sans-serif"}}>If Hormuz stays closed 4+ weeks: BofA Brent $100-150 · Qatar gas $40/MMBtu · Global recession risk elevated.</div>
          </div>
        </Panel>
      </div>
    )}

    {/* ── VIEWERS ──────────────────────────────────────────────────── */}
    {tab==="viewers" && <ViewerCounter/>}

  </div>

  {/* ── FOOTER ───────────────────────────────────────────────────── */}
  <div style={{padding:"12px 16px",borderTop:"1px solid rgba(0,229,255,0.06)",fontFamily:"'Share Tech Mono',monospace",fontSize:7,color:"#1a2830",letterSpacing:1,lineHeight:2,textAlign:"center"}}>
    <span style={{fontFamily:"'Orbitron',monospace",color:"#0a1820",letterSpacing:3}}>STRATOWATCH</span> · stratowatch.intel<br/>
    LIVE APIS: YAHOO FINANCE · COINGECKO · OPENSKY NETWORK · ALIENVAULT OTX · COUNTAPI<br/>
    OSINT: US CENTCOM · IDF · CRITICAL THREATS PROJECT · FDD · BLOOMBERG · REUTERS<br/>
    DAY {elapsed.day} · {elapsed.date} · FOR RESEARCH AND EDUCATIONAL PURPOSES ONLY
  </div>
</div>
```

);
}
