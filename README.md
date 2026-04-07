import React, { useState, useEffect, useMemo, useRef } from 'react';
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, onSnapshot, query, doc, deleteDoc, addDoc, serverTimestamp, writeBatch, getDocs } from 'firebase/firestore';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { 
  Database, Trash2, Zap, UploadCloud, Loader2, Target, 
  Coins, RefreshCw, FileSpreadsheet, Search, Cpu, CheckCircle2,
  Smartphone, Hash, User, Terminal, AlertTriangle, 
  ShieldCheck, Crosshair, Settings2, Fingerprint,
  Target as TargetIcon, Rocket, Trophy, Eye,
  ShieldAlert, TrendingUp, BarChart3, Eraser, FileText,
  Table as TableIcon, Cat, Wallet, ChevronRight, Activity,
  PieChart, Sparkles, MessageSquare, Send, Bot, Info,
  ShieldQuestion, BrainCircuit, Lightbulb, Gavel, Scan, Sun,
  Ghost, Command, Stars, ExternalLink, ZapOff
} from 'lucide-react';

// --- COMMANDER INFRASTRUCTURE ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'darkside-bb-v4';
const apiKey = ""; 
const SUPABASE_KEY = "sb_publishable_1agyo85-Uxq1VAllYjl1gg_M_utlMbw";
const SUPABASE_URL = "https://nlgecesyjcrrqwiitlyt.supabase.co";
const PROJECT_ID = "nlgecesyjcrrqwiitlyt";

// --- AI NEURAL LINK (GEMINI 2.5 FLASH) ---
const callGemini = async (prompt, systemPrompt = "You are the TITAN MATRIX AI. Accurate, tactical, professional. Response in Thai.") => {
  try {
    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        contents: [{ parts: [{ text: prompt }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] }
      })
    });
    if (!response.ok) return "Link Refused.";
    const data = await response.json();
    return data.candidates?.[0]?.content?.parts?.[0]?.text || "Intelligence not found.";
  } catch (e) { return "Signal lost."; }
};

export default function App() {
  const [user, setUser] = useState(null);
  const [orders, setOrders] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [activeTab, setActiveTab] = useState('dashboard');
  const [brand, setBrand] = useState('BB'); // BB, SINGTO
  
  // States
  const [isAiLoading, setIsAiLoading] = useState(false);
  const [aiAnalysis, setAiAnalysis] = useState(null);
  const [riskAssessment, setRiskAssessment] = useState(null);
  const [aiChat, setAiChat] = useState([]);
  const [chatInput, setChatInput] = useState('');
  const chatEndRef = useRef(null);

  // File states
  const [allRawRows, setAllRawRows] = useState([]);
  const [headerOptions, setHeaderOptions] = useState([]);
  const [encoding, setEncoding] = useState('windows-874'); 
  const [currentFileBuffer, setCurrentFileBuffer] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);
  const [logs, setLogs] = useState([]);
  const [mapping, setMapping] = useState({ tracking_number: -1, phone_number: -1, recipient_name: -1, cod_amount: -1 });

  // THEME V33.2: VOID GLOW (ULTRA BRIGHT)
  const theme = useMemo(() => {
    if (brand === 'BB') return { 
      primary: '#e879f9', 
      glow: 'rgba(232,121,249,0.95)', 
      nebula: 'rgba(147,51,234,0.5)', 
      text: 'BB STORE', 
      icon: <Stars size={20} className="text-purple-200"/>,
      table: 'bb_orders_master',
      accent: 'text-purple-300',
      beam: 'linear-gradient(90deg, transparent, #e879f9, #fff, #e879f9, transparent)'
    };
    return { 
      primary: '#fb923c', 
      glow: 'rgba(251,146,60,0.9)', 
      nebula: 'rgba(234,88,12,0.4)', 
      text: 'SINGTO', 
      icon: <Cat size={20} className="text-orange-300"/>,
      table: 'st_orders_master',
      accent: 'text-orange-300',
      beam: 'linear-gradient(90deg, transparent, #fb923c, #fff, #fb923c, transparent)'
    };
  }, [brand]);

  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else { await signInAnonymously(auth); }
      } catch (err) { console.error("Identity Denied"); }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  useEffect(() => {
    if (!user) return;
    // Dynamic table loading
    const colRef = collection(db, 'artifacts', appId, 'public', 'data', theme.table);
    const unsubscribe = onSnapshot(query(colRef), (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setOrders(data.sort((a, b) => (b.timestamp?.seconds || 0) - (a.timestamp?.seconds || 0)));
    });
    return () => unsubscribe();
  }, [user, theme.table]);

  useEffect(() => {
    chatEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [aiChat]);

  // --- 🛠 PRECISION CALCULATION ---
  const stats = useMemo(() => {
    const cleanNumber = (val) => {
      if (val === undefined || val === null || val === '') return 0;
      const cleaned = String(val).replace(/[^0-9.]/g, '');
      const parsed = parseFloat(cleaned);
      return isNaN(parsed) ? 0 : parsed;
    };
    const total = orders.reduce((sum, o) => sum + cleanNumber(o.cod_amount), 0);
    return { total: Math.round(total * 100) / 100, count: orders.length };
  }, [orders]);

  const addLog = (msg, isError = false) => {
    const time = new Date().toLocaleTimeString();
    const safeMsg = typeof msg === 'object' ? JSON.stringify(msg) : String(msg);
    setLogs(prev => [`[${time}] ${isError ? '❌' : '✅'} ${safeMsg}`, ...prev].slice(0, 5));
  };

  // --- 📂 FILE HANDLING ---
  const analyzePayload = (buffer, enc) => {
    setIsProcessing(true);
    try {
      const decoder = new TextDecoder(enc);
      let text = decoder.decode(buffer);
      if (text.charCodeAt(0) === 0xFEFF) text = text.substr(1);
      const sanitizedText = text.replace(/[\u0000-\u0008\u000B\u000C\u000E-\u001F\u007F-\u009F]/g, "");
      const lines = sanitizedText.split(/\r?\n/).filter(line => line.trim() !== '');
      
      if (lines.length < 2) {
        addLog("Protocol insufficient.", true);
        return;
      }

      const sep = lines[0].includes('\t') ? '\t' : (lines[0].includes(';') ? ';' : ',');
      const rows = lines.map(row => row.split(sep).map(c => c.replace(/^"|"$/g, '').trim()));
      setAllRawRows(rows);
      const headerRow = rows[0];
      setHeaderOptions(headerRow);

      const findIdx = (targets) => headerRow.findIndex(h => {
        const cleanH = String(h || '').toLowerCase().replace(/[^a-z0-9ก-ฮ]/g, '');
        return targets.some(t => cleanH.includes(t.toLowerCase().replace(/[^a-z0-9ก-ฮ]/g, '')));
      });

      setMapping({
        tracking_number: findIdx(['tracking', 'เลขพัสดุ', 'pno', 'track']),
        phone_number: findIdx(['phone', 'เบอร์โทร', 'มือถือ', 'tel', 'เบอร์ผู้รับ']),
        recipient_name: findIdx(['consignee', 'ชื่อผู้รับ', 'ชื่อ', 'name']),
        cod_amount: findIdx(['codamount', 'ยอดเงิน', 'ยอดcod', 'price', 'cod'])
      });
      addLog(`Matrix identified ${lines.length - 1} records.`);
    } catch (e) { addLog("Analysis Denied.", true); }
    finally { setIsProcessing(false); }
  };

  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;
    addLog(`Infiltrating: ${file.name}...`);
    const reader = new FileReader();
    reader.onload = (event) => {
      setCurrentFileBuffer(event.target.result);
      analyzePayload(event.target.result, encoding);
    };
    reader.readAsArrayBuffer(file);
  };

  const forceEncoding = (newEnc) => {
    setEncoding(newEnc);
    if (currentFileBuffer) analyzePayload(currentFileBuffer, newEnc);
  };

  const executeStrike = async () => {
    const data = extractedData;
    if (data.length === 0) return;
    setIsProcessing(true);
    addLog(`Authorizing Strike for ${brand}...`);
    try {
      const sourceTag = `TITAN_${brand}_V33`;
      const response = await fetch(`${SUPABASE_URL}/rest/v1/${theme.table}`, {
        method: 'POST',
        headers: { 'apikey': SUPABASE_KEY, 'Authorization': `Bearer ${SUPABASE_KEY}`, 'Content-Type': 'application/json', 'Prefer': 'return=minimal' },
        body: JSON.stringify(data.map(i => ({ ...i, source: sourceTag })))
      });
      if (!response.ok) throw new Error("Vault Denial");
      const colRef = collection(db, 'artifacts', appId, 'public', 'data', theme.table);
      for (const item of data) { await addDoc(colRef, { ...item, source: sourceTag, timestamp: serverTimestamp() }); }
      addLog("Strike Confirmed. Vault Secure.");
      setAllRawRows([]);
      alert(`ภารกิจสำเร็จ: อิมพอร์ต ${data.length} รายการลงคลัง ${brand} เรียบร้อย ✌️🔴🔥`);
    } catch (e) { addLog("Strike Fault.", true); }
    finally { setIsProcessing(false); }
  };

  const extractedData = useMemo(() => {
    if (allRawRows.length < 2) return [];
    return allRawRows.slice(1).map(r => {
      const getVal = (idx) => idx !== -1 ? String(r[idx] || '').trim() : '';
      const codVal = getVal(mapping.cod_amount).replace(/[^0-9.]/g, '');
      return {
        tracking_number: getVal(mapping.tracking_number),
        phone_number: getVal(mapping.phone_number).replace(/[^0-9]/g, ''),
        recipient_name: mapping.recipient_name !== -1 ? getVal(mapping.recipient_name) : 'Unknown',
        cod_amount: parseFloat(codVal) || 0
      };
    }).filter(i => i.phone_number.length >= 9 && i.tracking_number.length > 5);
  }, [allRawRows, mapping]);

  // AI ACTIONS
  const generateStrategicSummary = async () => {
    if (orders.length === 0) return;
    setIsAiLoading(true);
    try {
      const result = await callGemini(`วิเคราะห์คลัง ${brand}: ${orders.length} ออเดอร์, ยอดเงิน ฿${stats.total}. แนะนำยุทธศาสตร์ 3 ข้อสั้นๆ`, "ที่ปรึกษาอาณาจักร");
      setAiAnalysis(String(result));
    } catch (e) { addLog("AI Error", true); }
    finally { setIsAiLoading(false); }
  };

  const runRiskAssessment = async () => {
    if (extractedData.length === 0) return;
    setIsAiLoading(true);
    try {
      const sample = extractedData.slice(0, 15).map(o => o.recipient_name);
      const result = await callGemini(`Assess names: ${JSON.stringify(sample)}. Detect suspicious entries in 2 Thai sentences.`);
      setRiskAssessment(String(result));
    } catch (e) { addLog("AI Error", true); }
    finally { setIsAiLoading(false); }
  };

  const sendChatMessage = async () => {
    if (!chatInput.trim()) return;
    const userMsg = { role: 'user', text: String(chatInput) };
    setAiChat(prev => [...prev, userMsg]);
    setChatInput('');
    setIsAiLoading(true);
    try {
      const result = await callGemini(userMsg.text, `Matrix AI for ${brand}.`);
      setAiChat(prev => [...prev, { role: 'bot', text: String(result) }]);
    } catch (e) { setAiChat(prev => [...prev, { role: 'bot', text: "Signal lost." }]); }
    finally { setIsAiLoading(false); }
  };

  const masterReset = async () => {
    if (!window.confirm("PURGE CURRENT VAULT?")) return;
    setIsProcessing(true);
    try {
      await fetch(`${SUPABASE_URL}/rest/v1/${theme.table}?source=ilike.%TITAN_${brand}%`, {
        method: 'DELETE',
        headers: { 'apikey': SUPABASE_KEY, 'Authorization': `Bearer ${SUPABASE_KEY}` }
      });
      const colRef = collection(db, 'artifacts', appId, 'public', 'data', theme.table);
      const snapshot = await getDocs(query(colRef));
      const batch = writeBatch(db);
      snapshot.docs.forEach(d => batch.delete(d.ref));
      await batch.commit();
      addLog("Vault Cleared.");
    } catch (e) { addLog("Purge Failed.", true); }
    finally { setIsProcessing(false); }
  };

  const isMappingComplete = mapping.tracking_number !== -1 && mapping.phone_number !== -1;

  return (
    <div className="min-h-screen bg-[#010102] text-[#f1f5f9] font-sans antialiased overflow-hidden flex flex-col relative selection:bg-white/10">
      
      {/* VOID GLOW STYLES */}
      <style>{`
        ::-webkit-scrollbar { width: 3px; }
        ::-webkit-scrollbar-thumb { background: #334155; }
        .nebula-vivid { 
          position: absolute; inset: 0; pointer-events: none; 
          background: 
            radial-gradient(circle at 10% 10%, ${theme.nebula}, transparent 50%),
            radial-gradient(circle at 90% 80%, ${theme.glow}, transparent 60%),
            radial-gradient(circle at 50% 50%, rgba(255,255,255,0.15), transparent 70%);
          filter: blur(100px); animation: nebula-pulse 8s infinite alternate; z-index: 0; 
        }
        @keyframes nebula-pulse { 0% { opacity: 0.6; transform: scale(1); } 100% { opacity: 1; transform: scale(1.1); } }
        .glass-kinetic { background: rgba(8, 8, 12, 0.9); backdrop-filter: blur(40px); border: 1px solid rgba(255, 255, 255, 0.15); border-radius: 1rem; transition: all 0.3s; position: relative; overflow: hidden; }
        .glass-kinetic:hover { border-color: ${theme.primary}; box-shadow: 0 0 50px ${theme.glow}; }
        .matrix-text { text-shadow: 0 0 20px ${theme.glow}, 0 0 40px ${theme.glow}; color: #fff; }
        
        .kinetic-beam::after {
          content: ""; position: absolute; inset: -1px;
          background: ${theme.beam};
          background-size: 200% 100%; animation: beam-flow 3s linear infinite; border-radius: 1rem; z-index: -1;
        }
        @keyframes beam-flow { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }
        @keyframes scan { 0% { transform: translateY(-100%); } 100% { transform: translateY(100%); } }
        .laser-scan { position: absolute; inset: 0; height: 120px; background: linear-gradient(to bottom, transparent, rgba(255,255,255,0.1), transparent); animation: scan 4s linear infinite; pointer-events: none; }
      `}</style>

      <div className="nebula-vivid" />

      {/* STRATEGIC HUB BAR */}
      <nav className="h-12 border-b border-[#1e293b] bg-black/80 backdrop-blur-3xl flex items-center justify-between px-4 sm:px-8 z-50 relative">
        <div className="flex items-center gap-4">
           <div className="p-1.5 bg-[#0a0a0a] rounded-lg border border-white/20 shadow-2xl" style={{boxShadow: `0 0 15px ${theme.glow}`}}>{theme.icon}</div>
           <div className="flex flex-col leading-none">
              <span className={`text-[10px] font-black uppercase tracking-[0.4em] ${theme.accent} matrix-text`}>{theme.text} COMMAND</span>
              <span className="text-[7px] font-mono text-white/20 mt-0.5 uppercase tracking-widest">V.33.2 VOID-GLOW</span>
           </div>
        </div>

        <div className="flex items-center gap-2 sm:gap-6">
           <div className="flex gap-1">
              <button onClick={() => window.open('https://docs.google.com/spreadsheets', '_blank')} className="p-2 hover:bg-white/10 rounded-lg transition-all text-[#475569] hover:text-green-400 border border-transparent"><FileSpreadsheet size={16}/></button>
              <button onClick={() => window.open(`https://supabase.com/dashboard/project/${PROJECT_ID}/editor`, '_blank')} className="p-2 hover:bg-white/10 rounded-lg transition-all text-[#475569] hover:text-emerald-400 border border-transparent"><TableIcon size={16}/></button>
           </div>
           <div className="w-[1px] h-5 bg-[#1e293b]"></div>
           <div className="flex gap-1 p-0.5 bg-[#0a0a0a] rounded-lg border border-[#334155]">
              {['dashboard', 'vault', 'ai'].map(t => (
                 <button key={t} onClick={() => setActiveTab(t)} className={`px-4 py-1.5 rounded-md text-[9px] font-black uppercase tracking-widest transition-all ${activeTab === t ? 'bg-[#fff] text-[#000] shadow-2xl' : 'text-[#475569] hover:text-[#fff]'}`}>{t}</button>
              ))}
           </div>
           <button onClick={() => setBrand(brand === 'BB' ? 'SINGTO' : 'BB')} className="p-2 rounded-lg bg-white/5 border border-white/10 hover:scale-110 active:scale-90 transition-all" style={{color: theme.primary}}><RefreshCw size={14}/></button>
        </div>
      </nav>

      {/* MAIN HUD */}
      <main className="flex-1 overflow-y-auto p-4 sm:p-8 z-10 relative">
        <div className="max-w-6xl mx-auto space-y-6 animate-in fade-in duration-1000">
           
           {/* STATS HUD */}
           <div className="grid grid-cols-1 sm:grid-cols-4 gap-6">
              <div className="glass-kinetic p-5 flex flex-col justify-center border-l-2 border-l-white/10 shadow-2xl">
                 <span className="text-[10px] font-bold uppercase tracking-[0.2em] text-[#94a3b8]">Live Armed Units</span>
                 <div className="flex items-baseline gap-2 mt-4">
                    <span className="text-5xl font-light text-[#fff] tracking-tighter">{stats.count}</span>
                    <span className="text-[9px] font-black uppercase text-[#475569]">UNITS</span>
                 </div>
              </div>
              <div className="sm:col-span-2 glass-kinetic p-8 flex items-center justify-between relative overflow-hidden border-l-4 shadow-2xl" style={{borderLeftColor: theme.primary}}>
                 <div className="relative z-10">
                    <span className="text-[11px] font-bold uppercase tracking-[0.6em] text-[#94a3b8]">Net Vault Intake ({brand})</span>
                    <h2 className="text-7xl font-light text-[#fff] tracking-tighter mt-2 matrix-text">฿ {stats.total.toLocaleString(undefined, {minimumFractionDigits: 0})}</h2>
                 </div>
                 <div className="bg-white/5 p-4 rounded-full border border-white/10 group-hover:scale-110 transition-all shadow-[0_0_20px_rgba(255,255,255,0.2)]">
                    <Wallet size={36} className="opacity-100" style={{color: theme.primary}}/>
                 </div>
              </div>
              <div className="glass-kinetic p-6 flex flex-col justify-center items-center gap-4 hover:bg-red-950/20 cursor-pointer transition-all border-l-2 border-l-red-500/50 shadow-2xl" onClick={masterReset}>
                 <Eraser size={24} className="text-red-500 shadow-[0_0_10px_red]"/>
                 <span className="text-[9px] font-black uppercase tracking-[0.3em] text-red-400">Purge Vault</span>
              </div>
           </div>

           {activeTab === 'dashboard' && (
              <div className="grid grid-cols-1 lg:grid-cols-12 gap-8 animate-in slide-in-from-bottom-5 duration-700">
                 
                 <div className="lg:col-span-4 space-y-6">
                    <div className="glass-kinetic p-8 space-y-8 relative overflow-hidden shadow-3xl">
                       <div className="laser-scan" />
                       <div className="flex items-center gap-4 border-b border-white/5 pb-6">
                          <div className="w-1.5 h-6 rounded-full shadow-[0_0_20px_#fff]" style={{backgroundColor: theme.primary}}></div>
                          <h3 className="text-[11px] font-black uppercase tracking-[0.4em] text-white italic">Infiltration Port</h3>
                       </div>

                       <label className="group flex flex-col items-center justify-center w-full h-44 border-2 border-dashed border-[#334155] rounded-3xl cursor-pointer hover:bg-white/[0.05] transition-all bg-black relative touch-manipulation shadow-inner">
                          <UploadCloud className="text-[#475569] group-hover:text-white group-hover:scale-110 transition-all duration-700 shadow-[0_0_20px_#fff]" size={40} />
                          <span className="text-[11px] text-[#fff] font-bold uppercase tracking-widest mt-6 group-hover:text-[#fff] matrix-text">Drop CSV/TXT Payload</span>
                          <input type="file" className="hidden" accept=".csv,.txt" onChange={handleFileUpload} />
                       </label>

                       <div className="space-y-6">
                          <div className="flex justify-center gap-3 bg-black/60 p-2.5 rounded-xl border border-white/5 shadow-inner">
                             {['windows-874', 'utf-8'].map(enc => (
                                <button key={enc} onClick={() => forceEncoding(enc)} className={`px-5 py-2 rounded-lg text-[9px] font-bold uppercase transition-all ${encoding === enc ? 'bg-[#fff] text-[#000] shadow-[0_0_15px_#fff]' : 'text-[#475569] border border-white/5'}`}>{enc === 'windows-874' ? 'Thai' : 'UTF-8'}</button>
                             ))}
                          </div>

                          {headerOptions.length > 0 && (
                             <div className="bg-[#000] p-6 rounded-2xl border border-white/10 space-y-4 shadow-3xl">
                                {['tracking_number', 'phone_number', 'recipient_name', 'cod_amount'].map((key) => (
                                   <div key={key}>
                                      <select className="w-full bg-transparent border-b border-[#1e293b] text-[10px] text-[#94a3b8] py-2.5 outline-none focus:text-[#fff] transition-all cursor-pointer font-bold"
                                           value={mapping[key]} onChange={(e) => setMapping({...mapping, [key]: parseInt(e.target.value)})}>
                                            <option value="-1" className="bg-black">-- LOCK {key.toUpperCase()} --</option>
                                            {headerOptions.map((h, i) => <option key={i} value={i} className="bg-black">Col {i+1}: {String(h || '').substring(0, 15)}</option>)}
                                      </select>
                                   </div>
                                ))}
                             </div>
                          )}

                          <div className={`p-0.5 rounded-[1.3rem] transition-all duration-1000 ${extractedData.length > 0 && isMappingComplete ? 'kinetic-beam' : ''}`}>
                             <button onClick={executeStrike} disabled={extractedData.length === 0 || isProcessing || !isMappingComplete} className={`w-full py-7 rounded-[1.25rem] uppercase tracking-[1.5em] text-sm font-black transition-all ${extractedData.length > 0 && isMappingComplete ? 'bg-[#fff] text-[#000] shadow-[0_0_30px_#fff] hover:scale-[1.02]' : 'bg-[#111] text-[#334155] cursor-not-allowed opacity-40'}`}>
                                {isProcessing ? <Loader2 size={20} className="animate-spin text-black mx-auto" /> : 'EXECUTE STRIKE'}
                             </button>
                          </div>
                       </div>
                    </div>
                    
                    <div className="bg-black/60 p-6 rounded-[2rem] border border-white/10 font-mono text-[9px] h-32 overflow-y-auto scrollbar-none opacity-50 shadow-inner">
                       <div className="space-y-1.5 font-bold tracking-widest text-[#fff]">
                          {logs.map((log, i) => <div key={i} className={`flex gap-3 ${String(log).includes('❌') ? 'text-red-500' : 'text-[#fff]'}`}>{String(log)}</div>)}
                       </div>
                    </div>
                 </div>

                 {/* PREVIEW PANEL */}
                 <div className="lg:col-span-8 space-y-6 flex flex-col">
                    <div className="glass-kinetic p-10 flex items-center justify-between relative overflow-hidden group shadow-3xl border-t-4" style={{borderTopColor: theme.primary}}>
                       <div className="flex flex-col gap-3 relative z-10">
                          <span className="text-[11px] font-black uppercase tracking-[0.8em] text-[#fff]/60 italic text-glow">Expected Strike Yield</span>
                          <h2 className="text-7xl font-light text-[#fff] tracking-tighter matrix-text">฿ {extractedData.reduce((s, i) => s + i.cod_amount, 0).toLocaleString()}</h2>
                       </div>
                       <div className="text-right relative z-10">
                          <span className="text-5xl font-light text-[#fff] group-hover:matrix-text transition-all duration-700">{extractedData.length}</span>
                          <p className="text-[10px] uppercase text-[#fff]/40 font-bold mt-2 italic tracking-[0.3em]">Targets</p>
                       </div>
                    </div>

                    <div className="glass-kinetic rounded-[2.5rem] flex-1 min-h-[550px] flex flex-col overflow-hidden relative border border-white/10 shadow-3xl">
                       <div className="p-8 border-b border-white/5 bg-white/[0.05] flex justify-between items-center relative z-10">
                          <div className="flex items-center gap-4">
                             <Scan size={18} className="text-[#fff] shadow-[0_0_10px_#fff]" />
                             <span className="text-[12px] font-black uppercase tracking-[0.6em] text-white matrix-text">Verification Field</span>
                          </div>
                          <button onClick={runRiskAssessment} disabled={extractedData.length === 0 || isAiLoading} className="text-[10px] font-black uppercase tracking-[0.4em] text-white transition-all bg-[#fff]/10 px-8 py-3 rounded-xl border border-white/30 hover:bg-white/20 shadow-[0_0_20px_rgba(255,255,255,0.2)]">
                             {isAiLoading ? 'Analyzing...' : '✨ AI SCAN'}
                          </button>
                       </div>
                       
                       {riskAssessment && (
                         <div className="p-8 bg-white/[0.03] border-b border-white/10 animate-in slide-in-from-top-6 duration-500 shadow-2xl">
                           <div className="flex items-center gap-4 mb-3">
                             <Sparkles className="text-yellow-400 shadow-[0_0_15px_yellow]" size={16}/>
                             <span className="text-[11px] font-black uppercase text-white/40 tracking-widest">AI Strategic Protocol</span>
                           </div>
                           <p className="text-sm font-medium text-[#cbd5e1] leading-relaxed italic">{String(riskAssessment)}</p>
                         </div>
                       )}

                       <div className="flex-grow max-h-[850px] overflow-y-auto p-10 scrollbar-thin">
                          {extractedData.length > 0 ? (
                             <table className="w-full text-left border-separate border-spacing-y-4">
                                <tbody>
                                   {extractedData.slice(0, 40).map((row, i) => (
                                      <tr key={i} className="group hover:bg-white/[0.03] transition-all shadow-lg rounded-xl">
                                         <td className="py-4 border-b border-white/5 group-hover:border-white/15 transition-colors">
                                            <div className="text-2xl font-medium text-[#fff] uppercase tracking-tight group-hover:matrix-text transition-all">{String(row.recipient_name)}</div>
                                            <div className="text-[10px] font-mono text-[#475569] mt-2 uppercase italic tracking-widest group-hover:text-[#94a3b8]">ID: {String(row.tracking_number)}</div>
                                         </td>
                                         <td className="py-4 border-b border-white/5 text-right">
                                            <span className="text-4xl font-light text-[#fff] tracking-tighter opacity-40 group-hover:opacity-100 group-hover:matrix-text transition-all">฿{Number(row.cod_amount).toLocaleString()}</span>
                                         </td>
                                      </tr>
                                   ))}
                                </tbody>
                             </table>
                          ) : (
                             <div className="flex flex-col items-center justify-center h-[500px] opacity-[0.02] grayscale">
                                <Command size={120} className="text-white shadow-2xl animate-pulse"/>
                                <span className="text-2xl uppercase font-black tracking-[3em] mt-16 italic text-white font-black">NO SIGNAL</span>
                             </div>
                          )}
                       </div>
                    </div>
                 </div>
              </div>
           )}

           {activeTab === 'vault' && (
              <div className="animate-in fade-in duration-1000 space-y-10">
                 <div className="glass-kinetic rounded-[4rem] overflow-hidden shadow-3xl border-white/5 relative">
                    <div className="laser-scan" />
                    <div className="p-12 border-b border-white/10 flex flex-col md:flex-row justify-between items-center gap-10 bg-black/60 relative">
                       <div className="flex items-center gap-10 relative z-10">
                          <Database size={40} style={{color: theme.primary}} className="shadow-2xl opacity-80"/>
                          <h2 className="text-[4rem] font-light text-white uppercase italic tracking-tighter leading-none matrix-glow">{brand} Vault</h2>
                       </div>
                       <div className="relative w-full md:w-[40rem] z-10">
                          <Search className="absolute left-8 top-1/2 -translate-y-1/2 text-[#475569]" size={24} />
                          <input type="text" placeholder="Scan asset hash signature..." className="bg-black border-2 border-white/20 rounded-[2.5rem] py-8 pl-24 pr-10 text-2xl w-full outline-none focus:border-white/50 transition-all font-mono text-[#fff] shadow-inner shadow-[0_0_30px_rgba(255,255,255,0.05)]" value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} />
                       </div>
                    </div>
                    <div className="max-h-[1600px] overflow-y-auto p-12 scrollbar-thin">
                       <table className="w-full text-left border-separate border-spacing-y-6">
                          <tbody className="divide-y divide-white/5">
                             {orders.filter(o => String(o.recipient_name || '').toLowerCase().includes(searchTerm.toLowerCase()) || String(o.phone_number || '').includes(searchTerm) || String(o.tracking_number || '').toLowerCase().includes(searchTerm.toLowerCase())).map(order => (
                                <tr key={order.id} className="group hover:bg-white/[0.04] transition-all relative rounded-2xl shadow-xl">
                                   <td className="p-10 border-b border-white/5">
                                      <div className="text-4xl font-medium text-[#f1f5f9] uppercase tracking-tight group-hover:text-white transition-colors leading-none mb-4">{String(order.recipient_name)}</div>
                                      <div className="flex gap-10 items-center">
                                         <span className="text-[13px] font-mono text-[#475569] tracking-[0.4em] uppercase italic">PK-ID: {String(order.tracking_number)}</span>
                                         <span className="text-[10px] font-mono text-[#334155] uppercase border-l border-[#1e293b] pl-10">SYNC_SECURED</span>
                                      </div>
                                   </td>
                                   <td className="p-10 font-mono text-white/60 text-3xl font-black">{String(order.phone_number)}</td>
                                   <td className="p-10 font-light text-[#fff] text-6xl text-right tracking-tighter leading-none matrix-glow">฿{Number(order.cod_amount || 0).toLocaleString()}</td>
                                   <td className="p-10 text-right">
                                      <button onClick={() => deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', theme.table, order.id))} className="text-[#334155] hover:text-red-700 transition-all hover:scale-150 p-4 rounded-full hover:bg-red-900/10 shadow-2xl">
                                         <Trash2 size={28} />
                                      </button>
                                   </td>
                                </tr>
                             ))}
                          </tbody>
                       </table>
                    </div>
                 </div>
              </div>
           )}

           {activeTab === 'ai' && (
              <div className="grid grid-cols-1 lg:grid-cols-12 gap-10 animate-in slide-in-from-bottom-8 duration-1000 h-[900px]">
                 <div className="lg:col-span-4 h-full">
                    <div className="glass-kinetic p-10 h-full flex flex-col relative overflow-hidden group shadow-3xl border-l-4 border-l-white/20">
                       <div className="relative z-10 space-y-12 flex-grow flex flex-col">
                          <div className="space-y-6">
                             <h3 className="text-5xl font-light text-[#fff] italic tracking-tighter uppercase flex items-center gap-8 matrix-glow">✨ Strategic</h3>
                             <p className="text-[16px] text-white/40 uppercase tracking-[1.2em] font-black italic">Neural Intelligence</p>
                          </div>
                          <button onClick={generateStrategicSummary} disabled={isAiLoading} className={`w-full py-10 rounded-[2.5rem] bg-white text-black text-[13px] font-black uppercase tracking-[0.5em] transition-all shadow-[0_0_50px_#fff] active:scale-95 ${isAiLoading ? 'opacity-20' : 'hover:scale-98'}`}>
                             {isAiLoading ? 'ESTABLISHING LINK...' : 'INITIATE NEURAL FEED'}
                          </button>
                          <div className="flex-grow bg-black/80 border border-white/20 rounded-[2.5rem] p-12 text-[20px] text-[#fff] font-medium leading-relaxed italic shadow-inner overflow-y-auto">
                             {aiAnalysis ? String(aiAnalysis) : 'Awaiting data link from commander...'}
                          </div>
                       </div>
                    </div>
                 </div>
                 <div className="lg:col-span-8 glass-kinetic rounded-[4rem] h-full flex flex-col relative shadow-3xl overflow-hidden border-t-4 border-t-white/10">
                    <div className="p-10 border-b border-white/5 flex items-center justify-between bg-black/60 relative overflow-hidden">
                       <div className="laser-scan" />
                       <div className="flex items-center gap-8 relative z-10">
                          <div className="p-4 rounded-3xl bg-white/5 text-white shadow-2xl transition-all" style={{boxShadow: `0 0 20px ${theme.glow}`}}><Bot size={32}/></div>
                          <h3 className="font-black text-2xl uppercase italic tracking-[0.5em] text-[#fff] matrix-glow">TITAN OVERSEER</h3>
                       </div>
                    </div>
                    <div className="flex-grow overflow-y-auto p-12 space-y-14 scrollbar-thin">
                       {aiChat.length === 0 && <div className="h-full flex flex-col items-center justify-center opacity-[0.03] space-y-12"><MessageSquare size={200}/><p className="text-3xl uppercase font-black tracking-[4em] text-white italic font-black">SILENCE PROTOCOL</p></div>}
                       {aiChat.map((msg, i) => (
                          <div key={i} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                             <div className={`max-w-[85%] p-10 rounded-[3.5rem] text-xl font-medium leading-relaxed shadow-3xl ${msg.role === 'user' ? 'bg-[#fff] text-[#000] scale-105 shadow-[0_0_30px_rgba(255,255,255,0.4)]' : 'bg-black/60 text-[#cbd5e1] border border-white/10 shadow-inner'}`}>{String(msg.text)}</div>
                          </div>
                       ))}
                       <div ref={chatEndRef} />
                    </div>
                    <div className="p-12 pt-0 relative z-10">
                       <div className="relative group">
                          <input type="text" className="w-full bg-black border border-white/10 rounded-[3rem] py-10 pl-14 pr-32 text-2xl font-mono text-[#fff] focus:border-white/30 outline-none transition-all placeholder:text-[#1e293b] shadow-2xl" placeholder="Send pulse to Matrix Core..." value={chatInput} onChange={(e) => setChatInput(e.target.value)} onKeyPress={(e) => e.key === 'Enter' && sendChatMessage()}/>
                          <button onClick={sendChatMessage} disabled={isAiLoading || !chatInput.trim()} className="absolute right-10 top-1/2 -translate-y-1/2 text-[#475569] hover:text-[#fff] transition-all p-4 rounded-full hover:bg-white/5 shadow-2xl"><Send size={32}/></button>
                       </div>
                    </div>
                 </div>
              </div>
           )}
        </div>
      </main>
      <footer className="h-16 flex items-center justify-center border-t border-white/10 bg-[#000] z-50">
         <span className="text-[12px] font-black uppercase tracking-[8em] text-white/5 translate-x-[4em]">TITAN NEBULA COMMAND // V.33.2 VOID-GLOW</span>
      </footer>
    </div>
  );
}
