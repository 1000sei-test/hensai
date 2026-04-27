# -import { useState, useEffect } from “react”;

const MONTHS = [“1月”,“2月”,“3月”,“4月”,“5月”,“6月”,“7月”,“8月”,“9月”,“10月”,“11月”,“12月”];

const INITIAL_LOANS = [
{ id: 1, name: “住宅ローン”, amount: 80000, color: “#4f8ef7” },
{ id: 2, name: “カーローン”, amount: 25000, color: “#f77b4f” },
{ id: 3, name: “カードローン”, amount: 15000, color: “#4fcf8e” },
];

function generateKey(loanId, year, month) {
return `${loanId}-${year}-${month}`;
}

export default function RepaymentTracker() {
const today = new Date();
const [year, setYear] = useState(today.getFullYear());
const [loans, setLoans] = useState(INITIAL_LOANS);
const [payments, setPayments] = useState({});
const [showAddLoan, setShowAddLoan] = useState(false);
const [newLoan, setNewLoan] = useState({ name: “”, amount: “” });
const [animating, setAnimating] = useState(null);

// Load from storage
useEffect(() => {
try {
const savedLoans = localStorage.getItem(“rt_loans”);
const savedPayments = localStorage.getItem(“rt_payments”);
if (savedLoans) setLoans(JSON.parse(savedLoans));
if (savedPayments) setPayments(JSON.parse(savedPayments));
} catch(e) {}
}, []);

useEffect(() => {
try {
localStorage.setItem(“rt_loans”, JSON.stringify(loans));
localStorage.setItem(“rt_payments”, JSON.stringify(payments));
} catch(e) {}
}, [loans, payments]);

const togglePayment = (loanId, month) => {
const key = generateKey(loanId, year, month);
const animKey = `${loanId}-${month}`;
setAnimating(animKey);
setTimeout(() => setAnimating(null), 400);
setPayments(prev => ({ …prev, [key]: !prev[key] }));
};

const isPaid = (loanId, month) => !!payments[generateKey(loanId, year, month)];

const addLoan = () => {
if (!newLoan.name.trim() || !newLoan.amount) return;
const colors = [”#e85d8a”,”#a855f7”,”#06b6d4”,”#eab308”,”#f97316”];
const color = colors[loans.length % colors.length];
setLoans(prev => […prev, { id: Date.now(), name: newLoan.name, amount: Number(newLoan.amount), color }]);
setNewLoan({ name: “”, amount: “” });
setShowAddLoan(false);
};

const removeLoan = (id) => setLoans(prev => prev.filter(l => l.id !== id));

const paidCount = (loanId) => MONTHS.filter((_, i) => isPaid(loanId, i + 1)).length;
const totalMonthlyDue = loans.reduce((s, l) => s + l.amount, 0);
const paidThisMonth = loans.filter(l => isPaid(l.id, today.getMonth() + 1)).reduce((s, l) => s + l.amount, 0);
const overallProgress = loans.length === 0 ? 0
: Math.round(loans.reduce((s, l) => s + paidCount(l.id), 0) / (loans.length * 12) * 100);

return (
<div style={{
minHeight: “100vh”,
background: “#0e0e14”,
fontFamily: “‘Noto Sans JP’, ‘Hiragino Sans’, sans-serif”,
color: “#f0f0f5”,
padding: “0 0 80px”,
}}>
<style>{`@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@300;400;500;700&family=Space+Mono:wght@400;700&display=swap'); * { box-sizing: border-box; margin: 0; padding: 0; } ::-webkit-scrollbar { width: 4px; } ::-webkit-scrollbar-track { background: #1a1a24; } ::-webkit-scrollbar-thumb { background: #3a3a54; border-radius: 4px; } .check-btn { width: 40px; height: 40px; border-radius: 10px; border: 2px solid; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 18px; transition: all 0.18s cubic-bezier(.34,1.56,.64,1); background: transparent; flex-shrink: 0; } .check-btn:hover { transform: scale(1.15); } .check-btn.paid { transform: scale(1); } .check-btn.pop { animation: pop 0.38s cubic-bezier(.34,1.56,.64,1); } @keyframes pop { 0% { transform: scale(1); } 50% { transform: scale(1.4); } 100% { transform: scale(1); } } .month-label { font-size: 11px; color: #888; letter-spacing: 0.5px; } .loan-card { background: #16161f; border-radius: 16px; padding: 20px 20px 16px; margin-bottom: 12px; border: 1px solid #26263a; transition: border-color 0.2s; } .loan-card:hover { border-color: #3a3a5a; } .progress-bar-bg { background: #26263a; border-radius: 4px; height: 4px; width: 100%; margin-top: 10px; } .progress-bar-fill { height: 4px; border-radius: 4px; transition: width 0.5s ease; } .tag { display: inline-block; padding: 2px 10px; border-radius: 20px; font-size: 11px; font-weight: 700; letter-spacing: 0.8px; font-family: 'Space Mono', monospace; } input[type=text], input[type=number] { background: #1e1e2e; border: 1px solid #3a3a54; border-radius: 8px; color: #f0f0f5; padding: 10px 14px; font-size: 14px; font-family: inherit; outline: none; width: 100%; transition: border-color 0.2s; } input[type=text]:focus, input[type=number]:focus { border-color: #6366f1; } .btn { border: none; border-radius: 10px; padding: 10px 20px; font-size: 14px; font-weight: 700; cursor: pointer; font-family: inherit; transition: all 0.18s; } .btn:hover { opacity: 0.85; transform: translateY(-1px); } .btn:active { transform: translateY(0); } .header-blur { background: rgba(14,14,20,0.92); backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px); position: sticky; top: 0; z-index: 100; border-bottom: 1px solid #26263a; } .summary-ring { position: relative; display: inline-flex; align-items: center; justify-content: center; } .ring-svg { transform: rotate(-90deg); } .ring-bg { fill: none; stroke: #26263a; stroke-width: 6; } .ring-fg { fill: none; stroke-width: 6; stroke-linecap: round; transition: stroke-dashoffset 0.7s ease; }`}</style>

```
  {/* Header */}
  <div className="header-blur">
    <div style={{ maxWidth: 540, margin: "0 auto", padding: "16px 20px", display: "flex", alignItems: "center", justifyContent: "space-between" }}>
      <div>
        <div style={{ fontSize: 11, color: "#6366f1", fontWeight: 700, letterSpacing: 2, textTransform: "uppercase", fontFamily: "'Space Mono', monospace" }}>返済管理</div>
        <div style={{ fontSize: 20, fontWeight: 700, marginTop: 2 }}>チェックシート</div>
      </div>
      {/* Year selector */}
      <div style={{ display: "flex", alignItems: "center", gap: 8, background: "#1a1a28", borderRadius: 10, padding: "6px 12px" }}>
        <button onClick={() => setYear(y => y - 1)} style={{ background: "none", border: "none", color: "#888", cursor: "pointer", fontSize: 18, lineHeight: 1 }}>‹</button>
        <span style={{ fontFamily: "'Space Mono', monospace", fontWeight: 700, fontSize: 15, minWidth: 42, textAlign: "center" }}>{year}</span>
        <button onClick={() => setYear(y => y + 1)} style={{ background: "none", border: "none", color: "#888", cursor: "pointer", fontSize: 18, lineHeight: 1 }}>›</button>
      </div>
    </div>
  </div>

  <div style={{ maxWidth: 540, margin: "0 auto", padding: "20px 16px" }}>

    {/* Summary card */}
    <div style={{ background: "linear-gradient(135deg, #1a1a2e 0%, #16213e 100%)", border: "1px solid #26264a", borderRadius: 20, padding: "20px", marginBottom: 20, display: "flex", alignItems: "center", gap: 20 }}>
      {/* Ring */}
      <div className="summary-ring">
        <svg width={80} height={80} className="ring-svg">
          <circle className="ring-bg" cx={40} cy={40} r={34}/>
          <circle className="ring-fg" cx={40} cy={40} r={34}
            stroke="#6366f1"
            strokeDasharray={`${2 * Math.PI * 34}`}
            strokeDashoffset={`${2 * Math.PI * 34 * (1 - overallProgress / 100)}`}
          />
        </svg>
        <div style={{ position: "absolute", textAlign: "center" }}>
          <div style={{ fontSize: 15, fontWeight: 700, fontFamily: "'Space Mono', monospace" }}>{overallProgress}%</div>
        </div>
      </div>
      <div style={{ flex: 1 }}>
        <div style={{ color: "#888", fontSize: 12, marginBottom: 4 }}>{year}年 年間進捗</div>
        <div style={{ fontSize: 13, marginBottom: 12 }}>
          <span style={{ color: "#a5b4fc" }}>今月の振込済</span>
          <span style={{ fontFamily: "'Space Mono', monospace", fontWeight: 700, fontSize: 18, marginLeft: 8 }}>¥{paidThisMonth.toLocaleString()}</span>
          <span style={{ color: "#555", fontSize: 12, marginLeft: 4 }}>/ ¥{totalMonthlyDue.toLocaleString()}</span>
        </div>
        <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
          {loans.map(l => (
            <span key={l.id} className="tag" style={{ background: l.color + "22", color: l.color, border: `1px solid ${l.color}44` }}>
              {isPaid(l.id, today.getMonth() + 1) ? "✓" : "–"} {l.name}
            </span>
          ))}
        </div>
      </div>
    </div>

    {/* Loan cards */}
    {loans.map(loan => {
      const paid = paidCount(loan.id);
      const pct = Math.round(paid / 12 * 100);
      return (
        <div key={loan.id} className="loan-card">
          {/* Header row */}
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 14 }}>
            <div>
              <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                <div style={{ width: 10, height: 10, borderRadius: "50%", background: loan.color, flexShrink: 0 }}/>
                <span style={{ fontWeight: 700, fontSize: 15 }}>{loan.name}</span>
              </div>
              <div style={{ fontFamily: "'Space Mono', monospace", color: loan.color, fontSize: 14, fontWeight: 700, marginTop: 3, paddingLeft: 18 }}>
                ¥{loan.amount.toLocaleString()} <span style={{ color: "#555", fontSize: 11, fontWeight: 400 }}>/月</span>
              </div>
            </div>
            <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
              <span style={{ fontFamily: "'Space Mono', monospace", fontSize: 12, color: "#888" }}>{paid}/12</span>
              <button onClick={() => removeLoan(loan.id)} style={{ background: "none", border: "none", color: "#444", cursor: "pointer", fontSize: 18, lineHeight: 1, padding: "2px 4px" }}>×</button>
            </div>
          </div>

          {/* Month grid */}
          <div style={{ display: "grid", gridTemplateColumns: "repeat(6, 1fr)", gap: 6 }}>
            {MONTHS.map((m, i) => {
              const month = i + 1;
              const paid = isPaid(loan.id, month);
              const isThisMonth = year === today.getFullYear() && month === today.getMonth() + 1;
              const animKey = `${loan.id}-${month}`;
              return (
                <div key={month} style={{ display: "flex", flexDirection: "column", alignItems: "center", gap: 4 }}>
                  <div className="month-label" style={{ color: isThisMonth ? loan.color : "#666" }}>{m}</div>
                  <button
                    className={`check-btn ${paid ? "paid" : ""} ${animating === animKey ? "pop" : ""}`}
                    onClick={() => togglePayment(loan.id, month)}
                    style={{
                      borderColor: paid ? loan.color : "#2e2e44",
                      background: paid ? loan.color + "28" : "transparent",
                      color: paid ? loan.color : "#3a3a54",
                      outline: isThisMonth && !paid ? `2px solid ${loan.color}55` : "none",
                      outlineOffset: 2,
                    }}
                    title={`${m}の${loan.name}を${paid ? "未払いに戻す" : "支払済みにする"}`}
                  >
                    {paid ? "✓" : ""}
                  </button>
                </div>
              );
            })}
          </div>

          {/* Progress */}
          <div className="progress-bar-bg">
            <div className="progress-bar-fill" style={{ width: `${pct}%`, background: loan.color }}/>
          </div>
        </div>
      );
    })}

    {/* Add loan */}
    {showAddLoan ? (
      <div style={{ background: "#16161f", border: "1px solid #6366f166", borderRadius: 16, padding: 20, marginBottom: 12 }}>
        <div style={{ fontWeight: 700, marginBottom: 14 }}>ローンを追加</div>
        <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
          <input type="text" placeholder="ローン名（例: 奨学金）" value={newLoan.name} onChange={e => setNewLoan(p => ({ ...p, name: e.target.value }))} />
          <input type="number" placeholder="月々の返済額（円）" value={newLoan.amount} onChange={e => setNewLoan(p => ({ ...p, amount: e.target.value }))} />
          <div style={{ display: "flex", gap: 8 }}>
            <button className="btn" onClick={addLoan} style={{ background: "#6366f1", color: "#fff", flex: 1 }}>追加する</button>
            <button className="btn" onClick={() => setShowAddLoan(false)} style={{ background: "#26263a", color: "#888" }}>キャンセル</button>
          </div>
        </div>
      </div>
    ) : (
      <button className="btn" onClick={() => setShowAddLoan(true)} style={{ width: "100%", background: "#1a1a28", color: "#6366f1", border: "2px dashed #6366f144", fontSize: 14 }}>
        ＋ ローンを追加
      </button>
    )}

    <div style={{ marginTop: 24, textAlign: "center", color: "#333", fontSize: 11, fontFamily: "'Space Mono', monospace" }}>
      データはこのブラウザに保存されます
    </div>
  </div>
</div>
```

);
}
