import React, { useState, useEffect } from 'react';
import { 
  MapPin, Wallet, Leaf, Scan, Navigation, Info, X, CheckCircle2, 
  ChevronRight, Search, MessageSquare, Sparkles, Send, Loader2,
  Newspaper, ShoppingBag, Gift, Ticket, User, ArrowRight, Home,
  Utensils, Gamepad2, Droplets, Trophy, Heart, Coffee, Bike, Camera,
  History, ShieldCheck, Headset, Crown, CreditCard
} from 'lucide-react';

// --- Gemini API 參數設定 ---
const apiKey = ""; 
const GEMINI_MODEL = "gemini-2.5-flash-preview-09-2025";

const App = () => {
  const [activeTab, setActiveTab] = useState('home'); 
  const [selectedStation, setSelectedStation] = useState(null);
  const [mapFilter, setMapFilter] = useState('all'); 
  const [showScanSuccess, setShowScanSuccess] = useState(false);
  const [isScanning, setIsScanning] = useState(false); 
  const [packoin, setPackoin] = useState(1280);
  
  // --- 電子雞遊戲狀態 ---
  const [petStats, setPetStats] = useState({
    hunger: 70,
    happiness: 80,
    health: 90,
    exp: 45,
    level: 2,
    status: 'Happy'
  });
  const [gameMsg, setGameMsg] = useState('哈囉！今天也要一起愛地球喔！');

  // AI 相關狀態
  const [showAiAssistant, setShowAiAssistant] = useState(false);
  const [aiInput, setAiInput] = useState('');
  const [chatHistory, setChatHistory] = useState([
    { role: 'ai', text: '您好！我是您的 ✨ PackAge+ 減碳助手。想知道如何更有效利用 PACKOIN，或是查詢最近的回收技巧嗎？' }
  ]);
  const [isAiTyping, setIsAiTyping] = useState(false);

  // 模擬站點數據
  const stations = [
    { id: 1, name: '全家便利商店 - 台北松山店', distance: '250m', type: 'both', bags: 5, returns: 12, lat: 45, lng: 35 },
    { id: 2, name: '7-11 - 興雅門市', distance: '450m', type: 'return', bags: 0, returns: 8, lat: 60, lng: 55 },
    { id: 3, name: '誠品書店 - 信義店', distance: '1.2km', type: 'rent', bags: 15, returns: 0, lat: 30, lng: 75 },
    { id: 4, name: '萊爾富 - 北市旗艦店', distance: '850m', type: 'both', bags: 3, returns: 20, lat: 20, lng: 40 },
    { id: 5, name: '里仁有機 - 南京店', distance: '1.5km', type: 'rent', bags: 8, returns: 0, lat: 75, lng: 25 },
  ];

  // --- 遊戲邏輯 ---
  const interactPet = (type) => {
    setPetStats(prev => {
      let newStats = { ...prev };
      if (type === 'feed') {
        newStats.hunger = Math.min(100, prev.hunger + 10);
        setGameMsg('嗯～這份有機飼料真好吃！');
      } else if (type === 'play') {
        newStats.happiness = Math.min(100, prev.happiness + 15);
        newStats.exp = prev.exp + 5;
        setGameMsg('耶！最喜歡跟你一起玩了！');
      } else if (type === 'clean') {
        newStats.health = Math.min(100, prev.health + 10);
        setGameMsg('感覺好清爽，謝謝你！');
      }
      return newStats;
    });
    setTimeout(() => setGameMsg('哈囉！今天也要一起愛地球喔！'), 3000);
  };

  // --- 掃描成功觸發邏輯 ---
  useEffect(() => {
    if (isScanning) {
      const timer = setTimeout(() => {
        setIsScanning(false);
        handleReturnAction();
      }, 2500); 
      return () => clearTimeout(timer);
    }
  }, [isScanning]);

  const handleReturnAction = () => {
    setShowScanSuccess(true);
    setPackoin(prev => prev + 50);
    setPetStats(prev => ({ ...prev, exp: prev.exp + 20 }));
    setTimeout(() => {
      setShowScanSuccess(false);
      setSelectedStation(null);
      setActiveTab('mall'); 
    }, 2000);
  };

  // --- Gemini API ---
  const callGemini = async (userPrompt, systemPrompt = "") => {
    const url = `https://generativelanguage.googleapis.com/v1beta/models/${GEMINI_MODEL}:generateContent?key=${apiKey}`;
    const payload = {
      contents: [{ parts: [{ text: userPrompt }] }],
      systemInstruction: { parts: [{ text: systemPrompt }] }
    };
    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
      const data = await response.json();
      return data.candidates?.[0]?.content?.parts?.[0]?.text;
    } catch (error) {
      return "暫時無法連接伺服器";
    }
  };

  const handleSendMessage = async () => {
    if (!aiInput.trim()) return;
    setChatHistory(prev => [...prev, { role: 'user', text: aiInput }]);
    setAiInput('');
    setIsAiTyping(true);
    const result = await callGemini(aiInput, "你是 PackAge+ AI 助手。");
    setChatHistory(prev => [...prev, { role: 'ai', text: result }]);
    setIsAiTyping(false);
  };

  return (
    <div className="flex flex-col h-screen bg-slate-50 font-sans text-slate-800 max-w-md mx-auto overflow-hidden shadow-2xl border-x relative">
      
      {/* Top Header */}
      <header className="px-6 pt-12 pb-4 bg-white flex justify-between items-end border-b z-10 shadow-sm">
        <div>
          <h1 className="text-2xl font-black text-[#49B5A8] tracking-tight">PackAge+</h1>
          <p className="text-[10px] text-slate-400 font-bold tracking-[0.15em] mb-1">循環包材 永續生活</p>
        </div>
        <button onClick={() => setShowAiAssistant(true)} className="bg-emerald-50 text-emerald-600 p-2 rounded-2xl border border-emerald-100 hover:bg-emerald-100 transition-all flex items-center gap-2 shadow-sm">
          <Sparkles size={18} />
          <span className="text-xs font-bold">AI 助手</span>
        </button>
      </header>

      {/* Main Content Area */}
      <main className="flex-1 overflow-y-auto relative bg-slate-50">
        
        {/* VIEW: HOME (主頁) */}
        {activeTab === 'home' && (
          <div className="p-6 space-y-8 animate-in fade-in duration-500 pb-20">
            {/* 遊戲區 */}
            <section className="bg-white rounded-[2.5rem] p-6 shadow-sm border border-slate-100 overflow-hidden relative">
              <div className="flex justify-between items-start mb-4">
                <div className="flex items-center gap-2 bg-emerald-50 px-3 py-1 rounded-full border border-emerald-100">
                  <Trophy size={14} className="text-emerald-500" />
                  <span className="text-[10px] font-black text-emerald-600 uppercase">LV.{petStats.level} 成長中</span>
                </div>
                <div className="bg-slate-50 px-2 py-1 rounded-lg text-[10px] font-bold text-slate-400 flex items-center gap-1">
                  <Heart size={10} fill="#f43f5e" className="text-[#f43f5e]" /> {petStats.happiness}%
                </div>
              </div>
              <div className="py-8 flex flex-col items-center justify-center relative">
                <div className="w-32 h-32 bg-emerald-50 rounded-full flex items-center justify-center animate-bounce duration-[2000ms]">
                  <img src={`https://api.dicebear.com/7.x/bottts/svg?seed=EcoPet&backgroundColor=b6e3f4`} alt="Eco Pet" className="w-20 h-20" />
                </div>
                <div className="absolute -top-2 right-4 bg-[#49B5A8] text-white p-3 rounded-2xl rounded-bl-none shadow-lg text-[11px] font-bold max-w-[140px]">{gameMsg}</div>
                <div className="w-48 bg-slate-100 h-2 rounded-full mt-6 overflow-hidden"><div className="bg-emerald-400 h-full transition-all duration-500" style={{ width: `${petStats.exp}%` }}></div></div>
              </div>
              <div className="flex justify-around gap-2 mt-4 pt-4 border-t border-slate-50">
                <button onClick={() => interactPet('feed')} className="flex flex-col items-center gap-1 group"><div className="p-3 bg-amber-50 text-amber-500 rounded-2xl shadow-sm"><Utensils size={20} /></div><span className="text-[10px] font-bold text-slate-400">餵食</span></button>
                <button onClick={() => interactPet('play')} className="flex flex-col items-center gap-1 group"><div className="p-3 bg-blue-50 text-blue-500 rounded-2xl shadow-sm"><Gamepad2 size={20} /></div><span className="text-[10px] font-bold text-slate-400">玩耍</span></button>
                <button onClick={() => interactPet('clean')} className="flex flex-col items-center gap-1 group"><div className="p-3 bg-cyan-50 text-cyan-500 rounded-2xl shadow-sm"><Droplets size={20} /></div><span className="text-[10px] font-bold text-slate-400">清潔</span></button>
              </div>
            </section>
            
            <section>
              <h3 className="font-black text-slate-900 mb-4 flex items-center gap-2 px-2"><Leaf size={18} className="text-[#49B5A8]" />永續專欄</h3>
              <div className="grid grid-cols-2 gap-4 px-2">
                <div className="bg-white p-4 rounded-3xl border border-slate-100 shadow-sm hover:border-[#49B5A8] transition-all"><h4 className="font-black text-sm text-slate-800">減碳名人榜</h4><p className="text-[10px] text-slate-400">看看本月誰是最佳代言人！</p></div>
                <div className="bg-white p-4 rounded-3xl border border-slate-100 shadow-sm hover:border-[#49B5A8] transition-all"><h4 className="font-black text-sm text-slate-800">包材小知識</h4><p className="text-[10px] text-slate-400">循環幾次能打平碳足跡？</p></div>
              </div>
            </section>

            <section className="px-2 pb-10">
              <div className="flex justify-between items-center mb-4 px-1">
                <h3 className="font-black text-slate-900 flex items-center gap-2"><Newspaper size={18} className="text-[#49B5A8]" />環保報導</h3>
                <button className="text-[10px] font-black text-[#49B5A8]">查看全部</button>
              </div>
              <div className="space-y-4">
                {[
                  { title: 'PackAge+ 攜手 全家 FamilyMart 打造全台最大回收鏈', tag: '最新消息', img: 'https://images.unsplash.com/photo-1532996122724-e3c354a0b15b?auto=format&fit=crop&w=300&q=80' },
                  { title: '企業禮品提袋還能這樣玩？「客製循環提袋」3大特色', tag: '最新消息', img: 'https://images.unsplash.com/photo-1605600611280-1a1b392444b0?auto=format&fit=crop&w=300&q=80' },
                  { title: '展覽環保提袋變成永續行銷利器！品牌客製提袋秘訣', tag: '市場趨勢', img: 'https://images.unsplash.com/photo-1591339731633-aa24959966ca?auto=format&fit=crop&w=300&q=80' },
                  { title: 'TNFD 是什麼？了解 TNFD 採用好處及台灣科技業案例', tag: '市場趨勢', img: 'https://images.unsplash.com/photo-1451187580459-43490279c0fa?auto=format&fit=crop&w=300&q=80' },
                  { title: '2024 年網購包裝減量怎麼做？選擇循環包裝一舉兩得！', tag: '永續生活', img: 'https://images.unsplash.com/photo-1589939705384-5185138a047a?auto=format&fit=crop&w=300&q=80' },
                  { title: '循環袋、循環箱真的環保？看懂配客嘉 PLUS 循環圈', tag: '永續新知', img: 'https://images.unsplash.com/photo-1536939459926-301728717817?auto=format&fit=crop&w=300&q=80' },
                ].map((news, i) => (
                  <div key={i} className="bg-white rounded-3xl overflow-hidden border border-slate-100 shadow-sm flex items-center hover:border-emerald-100 transition-all cursor-pointer">
                    <img src={news.img} className="w-24 h-24 object-cover" alt="news" />
                    <div className="p-4 flex-1">
                      <div className="mb-1"><span className="text-[8px] font-black text-[#49B5A8] bg-emerald-50 px-2 py-0.5 rounded-full">{news.tag}</span></div>
                      <h4 className="text-[11px] font-black text-slate-800 leading-snug">{news.title}</h4>
                    </div>
                  </div>
                ))}
              </div>
            </section>
          </div>
        )}

        {/* VIEW: MAP (地圖) */}
        {activeTab === 'map' && (
          <div className="h-full relative bg-slate-100 flex flex-col">
            <div className="flex-1 relative bg-slate-200 overflow-hidden">
               <svg className="absolute inset-0 w-full h-full opacity-30" viewBox="0 0 100 100" preserveAspectRatio="none">
                 <path d="M0 20 Q50 25 100 20 M20 0 Q25 50 20 100 M80 0 Q75 50 80 100 M0 80 Q50 75 100 80" stroke="#94a3b8" fill="none" strokeWidth="0.5" />
               </svg>

               {stations.filter(s => {
                 if (mapFilter === 'all') return true;
                 if (mapFilter === 'rent') return s.type === 'rent' || s.type === 'both';
                 if (mapFilter === 'return') return s.type === 'return' || s.type === 'both';
                 return true;
               }).map(s => (
                <button key={s.id} onClick={() => setSelectedStation(s)} className={`absolute transition-all duration-300 transform -translate-x-1/2 -translate-y-1/2 ${selectedStation?.id === s.id ? 'scale-125 z-20' : 'z-10'}`} style={{ left: `${s.lat}%`, top: `${s.lng}%` }}>
                  <div className={`p-2 rounded-full shadow-lg border-2 ${(s.type === 'rent' || s.type === 'both') ? 'bg-[#49B5A8] border-white shadow-emerald-100' : 'bg-blue-500 border-white shadow-blue-100'}`}><MapPin size={20} className="text-white" /></div>
                </button>
              ))}

              <div className="absolute top-4 left-4 right-4 space-y-2 z-30">
                {/* Filter Switcher */}
                <div className="bg-white/90 backdrop-blur-md p-1 rounded-2xl shadow-xl flex gap-1 border border-white">
                  {['all', 'rent', 'return'].map((f) => (
                    <button key={f} onClick={() => setMapFilter(f)} className={`flex-1 py-2 text-[10px] font-black rounded-xl ${mapFilter === f ? 'bg-slate-900 text-white shadow-md' : 'text-slate-400 hover:bg-slate-50'}`}>{f === 'all' ? '全部顯示' : f === 'rent' ? '我要租借' : '我要歸還'}</button>
                  ))}
                </div>
                {/* Search Bar - 新增於此 */}
                <div className="bg-white rounded-xl shadow-md p-3 flex items-center gap-3 border border-slate-50">
                  <Search size={16} className="text-slate-400" />
                  <input type="text" placeholder="搜尋附近合作夥伴..." className="bg-transparent outline-none text-xs w-full font-medium" />
                </div>
              </div>
            </div>

            {selectedStation && (
              <div className="bg-white rounded-t-[2.5rem] p-6 shadow-[0_-10px_30px_rgba(0,0,0,0.1)] animate-in slide-in-from-bottom-20 z-40 relative">
                <div className="flex justify-between items-start mb-4">
                  <div className="flex-1">
                    <span className={`text-[9px] font-black px-2 py-0.5 rounded-full ${selectedStation.type === 'rent' || selectedStation.type === 'both' ? 'bg-emerald-50 text-emerald-600' : 'bg-blue-50 text-blue-600'}`}>{selectedStation.type === 'both' ? '雙效合作店' : selectedStation.type === 'rent' ? '專業租借點' : '便利歸還點'}</span>
                    <h3 className="text-xl font-black text-slate-900 mt-1">{selectedStation.name}</h3>
                  </div>
                  <button onClick={() => setSelectedStation(null)} className="p-2 bg-slate-50 rounded-full"><X size={18} className="text-slate-400" /></button>
                </div>
                <div className="flex gap-2">
                   <button className="flex-1 bg-slate-900 text-white py-4 rounded-2xl font-black flex items-center justify-center gap-2 hover:opacity-90"><Navigation size={18} />導航</button>
                   <button onClick={() => setIsScanning(true)} className="flex-[2] bg-[#49B5A8] text-white py-4 rounded-2xl font-black flex items-center justify-center gap-2 hover:opacity-90"><Scan size={18} />確認租還動作</button>
                </div>
              </div>
            )}
          </div>
        )}

        {/* VIEW: SCANNER MOCKUP */}
        {isScanning && (
          <div className="absolute inset-0 bg-black z-[80] animate-in fade-in duration-300 flex flex-col items-center justify-center text-white p-6 overflow-hidden">
            <div className="relative w-72 h-72">
              <div className="absolute top-0 left-0 w-8 h-8 border-t-4 border-l-4 border-[#49B5A8] rounded-tl-lg"></div>
              <div className="absolute top-0 right-0 w-8 h-8 border-t-4 border-r-4 border-[#49B5A8] rounded-tr-lg"></div>
              <div className="absolute bottom-0 left-0 w-8 h-8 border-b-4 border-l-4 border-[#49B5A8] rounded-bl-lg"></div>
              <div className="absolute bottom-0 right-0 w-8 h-8 border-b-4 border-r-4 border-[#49B5A8] rounded-br-lg"></div>
              <div className="absolute top-0 left-0 w-full h-1 bg-[#49B5A8] shadow-[0_0_15px_#49B5A8] animate-bounce-slow"></div>
              <div className="absolute inset-4 opacity-20 flex items-center justify-center"><Scan size={120} strokeWidth={1} /></div>
            </div>
            <div className="mt-12 text-center space-y-2">
              <div className="flex items-center justify-center gap-2"><Loader2 className="animate-spin text-[#49B5A8]" size={20} /><h3 className="text-lg font-black tracking-tight">正在識別 QR Code...</h3></div>
              <p className="text-xs text-slate-400 font-bold tracking-wide">請對準循環袋/箱上的掃描碼</p>
            </div>
            <button onClick={() => setIsScanning(false)} className="absolute bottom-12 bg-white/10 p-4 rounded-full backdrop-blur-md border border-white/10"><X size={24} /></button>
          </div>
        )}

        {/* VIEW: MALL (商城) */}
        {activeTab === 'mall' && (
          <div className="p-6 space-y-6 animate-in fade-in pb-20">
            <div className="bg-white rounded-3xl p-6 border border-slate-100 shadow-sm flex justify-between items-center">
              <div><p className="text-xs font-bold text-slate-400 mb-1 font-sans">可用 PACKOIN</p><span className="text-3xl font-black text-slate-900">{packoin.toLocaleString()}</span></div>
              <Wallet className="text-amber-500" size={32} />
            </div>
            <div className="space-y-3 px-2">
               <h3 className="font-black text-slate-900">熱門兌換</h3>
               {[
                 { title: '全家大杯美式咖啡', price: 800, icon: '☕' },
                 { title: '旋轉拍賣 $30 抵用券', price: 500, icon: '🚚' },
               ].map((item, i) => (
                 <div key={i} className="bg-white p-4 rounded-3xl flex justify-between items-center border border-slate-100 hover:border-amber-100 transition-all">
                   <div className="flex items-center gap-3">
                     <span className="text-2xl">{item.icon}</span>
                     <div><p className="font-bold text-slate-800 text-sm">{item.title}</p><p className="text-[10px] text-amber-500 font-bold">{item.price} P+</p></div>
                   </div>
                   <button className="bg-slate-900 text-white px-4 py-2 rounded-xl text-[10px] font-bold">兌換</button>
                 </div>
               ))}
            </div>
          </div>
        )}

        {/* VIEW: INFO (個人) */}
        {activeTab === 'info' && (
          <div className="p-6 space-y-6 animate-in fade-in pb-20 overflow-y-auto">
             {/* Profile Card */}
             <div className="bg-white rounded-[2.5rem] p-8 text-center border border-slate-100 shadow-sm relative overflow-hidden">
               <div className="absolute top-0 right-0 w-24 h-24 bg-emerald-50 rounded-full -mr-8 -mt-8 opacity-50"></div>
               <div className="w-20 h-20 bg-slate-100 rounded-full mx-auto mb-4 overflow-hidden border-4 border-white shadow-lg relative z-10">
                 <img src="https://api.dicebear.com/7.x/avataaars/svg?seed=Felix" alt="avatar" />
               </div>
               <h3 className="text-2xl font-black text-slate-900 relative z-10">周小明</h3>
               <p className="text-slate-400 text-xs mb-6 font-bold tracking-tight relative z-10">加入 PackAge+ 第 124 天</p>
               <div className="bg-emerald-900 text-white rounded-3xl p-5 flex items-center justify-between text-left shadow-lg relative z-10 group active:scale-95 transition-all">
                 <div className="flex items-center gap-4">
                    <div className="bg-emerald-400/20 p-2 rounded-xl"><Crown size={24} className="text-emerald-300" /></div>
                    <div><p className="text-[10px] font-bold text-emerald-300 uppercase tracking-widest mb-0.5">專業賣家方案</p><p className="text-sm font-black">Unlimited Pro 方案</p></div>
                 </div>
                 <ChevronRight size={18} className="text-emerald-400" />
               </div>
             </div>

             <div className="space-y-3 px-2">
               <button className="w-full bg-white p-5 rounded-3xl flex justify-between items-center text-sm font-black border border-slate-100 hover:border-[#49B5A8] hover:bg-emerald-50/20 transition-all shadow-sm">
                 <div className="flex items-center gap-4"><div className="bg-amber-50 text-amber-500 p-2.5 rounded-2xl"><CreditCard size={20} /></div><span className="text-slate-700">訂閱方案管理</span></div>
                 <div className="flex items-center gap-2"><span className="text-[10px] font-bold text-amber-500 bg-amber-50 px-2 py-0.5 rounded-full">續訂中</span><ChevronRight size={18} className="text-slate-300" /></div>
               </button>
               <button className="w-full bg-white p-5 rounded-3xl flex justify-between items-center text-sm font-black border border-slate-100 hover:border-[#49B5A8] hover:bg-emerald-50/20 transition-all shadow-sm">
                 <div className="flex items-center gap-4"><div className="bg-blue-50 text-blue-500 p-2.5 rounded-2xl"><History size={20} /></div><span className="text-slate-700">租還歷史紀錄</span></div>
                 <div className="flex items-center gap-2"><span className="text-[10px] font-bold text-slate-300">本月 12 次</span><ChevronRight size={18} className="text-slate-300" /></div>
               </button>
               <button className="w-full bg-white p-5 rounded-3xl flex justify-between items-center text-sm font-black border border-slate-100 hover:border-[#49B5A8] hover:bg-emerald-50/20 transition-all shadow-sm">
                 <div className="flex items-center gap-4"><div className="bg-slate-50 text-slate-500 p-2.5 rounded-2xl"><ShieldCheck size={20} /></div><span className="text-slate-700">帳號與隱私設定</span></div>
                 <ChevronRight size={18} className="text-slate-300" />
               </button>
               <button className="w-full bg-white p-5 rounded-3xl flex justify-between items-center text-sm font-black border border-slate-100 hover:border-[#49B5A8] hover:bg-emerald-50/20 transition-all shadow-sm">
                 <div className="flex items-center gap-4"><div className="bg-emerald-50 text-emerald-500 p-2.5 rounded-2xl"><Headset size={20} /></div><span className="text-slate-700">幫助與客服中心</span></div>
                 <div className="flex items-center gap-2"><div className="w-2 h-2 bg-rose-500 rounded-full animate-pulse"></div><ChevronRight size={18} className="text-slate-300" /></div>
               </button>
             </div>
          </div>
        )}

        {/* AI Assistant */}
        {showAiAssistant && (
          <div className="absolute inset-0 bg-white z-[90] flex flex-col animate-in slide-in-from-bottom-full duration-300">
            <div className="px-6 pt-12 pb-4 flex justify-between items-center border-b">
              <div className="flex items-center gap-2"><Sparkles className="text-emerald-500" size={20} /><h2 className="text-lg font-black">PackAge+ AI</h2></div>
              <button onClick={() => setShowAiAssistant(false)} className="p-2 bg-slate-50 rounded-full"><X size={20} /></button>
            </div>
            <div className="flex-1 overflow-y-auto p-6 space-y-4 bg-slate-50">
              {chatHistory.map((msg, i) => (
                <div key={i} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                  <div className={`max-w-[85%] p-4 rounded-2xl text-xs font-medium shadow-sm ${msg.role === 'user' ? 'bg-slate-900 text-white rounded-tr-none' : 'bg-white border border-slate-100 rounded-tl-none'}`}>{msg.text}</div>
                </div>
              ))}
            </div>
            <div className="p-4 bg-white border-t flex gap-2">
              <input type="text" value={aiInput} onChange={(e) => setAiInput(e.target.value)} onKeyPress={(e) => e.key === 'Enter' && handleSendMessage()} placeholder="詢問 PackAge+ 永續資訊..." className="flex-1 bg-slate-100 px-4 py-3 rounded-2xl text-xs outline-none font-medium" />
              <button onClick={handleSendMessage} className="bg-emerald-500 text-white p-3 rounded-2xl shadow-lg active:scale-95 transition-all"><Send size={18} /></button>
            </div>
          </div>
        )}

        {/* Scan Success Overlay */}
        {showScanSuccess && (
          <div className="absolute inset-0 bg-[#49B5A8]/95 backdrop-blur-sm z-[100] flex flex-col items-center justify-center text-white animate-in zoom-in-95">
            <CheckCircle2 size={80} className="mb-4" />
            <h2 className="text-3xl font-black mb-2 tracking-tight">歸還成功！</h2>
            <p className="font-bold opacity-80 mb-6 tracking-wider">PACKOIN +50</p>
            <div className="bg-white/20 px-6 py-3 rounded-full font-black text-sm flex items-center gap-2 border border-white/20 shadow-xl">
               <Leaf size={18} /> 小精靈獲得了大量成長值！
            </div>
          </div>
        )}
      </main>

      {/* Bottom Navigation */}
      <nav className="bg-white border-t px-6 py-4 flex justify-between items-center z-10 shadow-[0_-5px_15px_rgba(0,0,0,0.05)]">
        <button onClick={() => { setActiveTab('home'); setIsScanning(false); }} className={`flex flex-col items-center gap-1 transition-all ${activeTab === 'home' && !isScanning ? 'text-slate-900 scale-110' : 'text-slate-300'}`}>
          <Home size={22} fill={activeTab === 'home' && !isScanning ? 'currentColor' : 'none'} />
          <span className="text-[9px] font-black uppercase tracking-widest">主頁</span>
        </button>
        <button onClick={() => { setActiveTab('map'); setIsScanning(false); }} className={`flex flex-col items-center gap-1 transition-all ${activeTab === 'map' && !isScanning ? 'text-slate-900 scale-110' : 'text-slate-300'}`}>
          <MapPin size={22} fill={activeTab === 'map' && !isScanning ? 'currentColor' : 'none'} />
          <span className="text-[9px] font-black uppercase tracking-widest">地圖</span>
        </button>
        <div className="relative -top-8">
          <button onClick={() => setIsScanning(true)} className={`w-14 h-14 rounded-2xl shadow-xl flex items-center justify-center border-4 border-white active:scale-90 transition-all ${isScanning ? 'bg-rose-500 text-white' : 'bg-slate-900 text-white'}`}>
            <Scan size={24} />
          </button>
        </div>
        <button onClick={() => { setActiveTab('mall'); setIsScanning(false); }} className={`flex flex-col items-center gap-1 transition-all ${activeTab === 'mall' && !isScanning ? 'text-[#49B5A8] scale-110' : 'text-slate-300'}`}>
          <ShoppingBag size={22} fill={activeTab === 'mall' && !isScanning ? 'currentColor' : 'none'} />
          <span className="text-[9px] font-black uppercase tracking-widest">商城</span>
        </button>
        <button onClick={() => { setActiveTab('info'); setIsScanning(false); }} className={`flex flex-col items-center gap-1 transition-all ${activeTab === 'info' && !isScanning ? 'text-slate-900 scale-110' : 'text-slate-300'}`}>
          <User size={22} fill={activeTab === 'info' && !isScanning ? 'currentColor' : 'none'} />
          <span className="text-[9px] font-black uppercase tracking-widest">個人</span>
        </button>
      </nav>

      <style dangerouslySetInnerHTML={{ __html: `
        @keyframes bounce-slow {
          0%, 100% { transform: translateY(0); }
          50% { transform: translateY(280px); }
        }
        .animate-bounce-slow {
          animation: bounce-slow 2s infinite ease-in-out;
        }
        main::-webkit-scrollbar { display: none; }
      `}} />
    </div>
  );
};

export default App;
