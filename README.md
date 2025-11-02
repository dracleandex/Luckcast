import React, { useEffect, useState } from "react";

// LuckCast - Daily Luck Score // Single-file React component (Next.js / Vercel friendly) // Tailwind CSS assumed available in the project (but styling will still be readable without it)

export default function LuckCast() { const todayKey = new Date().toISOString().slice(0, 10); // YYYY-MM-DD const storageKey = luckcast_leaderboard_${todayKey}; const rollKey = luckcast_roll_${todayKey};

const [handle, setHandle] = useState(""); const [score, setScore] = useState<number | null>(null); const [message, setMessage] = useState(""); const [leaderboard, setLeaderboard] = useState<Array<{ handle: string; score: number; time: string }>>([]); const [hasRolled, setHasRolled] = useState(false); const [copied, setCopied] = useState(false);

const funnyMessages = [ (s: number) => 🍀 You rolled ${s}/100 — The luck gods winked at you 👀, (s: number) => ✨ ${s}/100 — Fortune says: maybe buy a lottery ticket... maybe., (s: number) => 😅 ${s}/100 — Hmm. Today is a cautious optimism day., (s: number) => 🔥 ${s}/100 — Legendary! The stars aligned for you., (s: number) => 😬 ${s}/100 — Could be worse. Could be Monday-level worse., (s: number) => 🤷 ${s}/100 — Luck is like weather: unpredictable., (s: number) => 🎉 ${s}/100 — Big mood. Big vibes. Go shine!, (s: number) => 😂 ${s}/100 — The universe had a chuckle. You were in the punchline., ];

useEffect(() => { // load leaderboard from localStorage for today try { const raw = localStorage.getItem(storageKey); if (raw) setLeaderboard(JSON.parse(raw)); } catch (e) { console.error("Failed to load leaderboard", e); }

// check if user already rolled (by handle)
try {
  const rolled = localStorage.getItem(rollKey);
  if (rolled) {
    const parsed = JSON.parse(rolled);
    if (parsed && parsed.handle && parsed.score !== undefined) {
      setHandle(parsed.handle);
      setScore(parsed.score);
      setHasRolled(true);
      setMessage(makeMessage(parsed.score));
    }
  }
} catch (e) {
  console.error("Failed to load roll", e);
}

}, []);

function makeMessage(s: number) { const idx = s % funnyMessages.length; return funnyMessagesidx; }

function rollLuck() { if (!handle || handle.trim().length < 2) { alert("Please enter a handle (e.g. @yourname) before you roll — it's how we show the leaderboard."); return; }

if (hasRolled) return;

const s = Math.floor(Math.random() * 100) + 1; // 1-100
setScore(s);
setMessage(makeMessage(s));
setHasRolled(true);

const time = new Date().toLocaleTimeString();
const entry = { handle: handle.trim(), score: s, time };

const next = [entry, ...leaderboard].slice(0, 50); // keep most recent 50
setLeaderboard(next);

// persist to localStorage
try {
  localStorage.setItem(storageKey, JSON.stringify(next));
  localStorage.setItem(rollKey, JSON.stringify(entry));
} catch (e) {
  console.error("Failed to persist leaderboard", e);
}

}

function resetToday() { // Debug helper: clears today's leaderboard and rolls. (In production you might hide this.) if (!confirm("Reset today's leaderboard and your roll?")) return; localStorage.removeItem(storageKey); localStorage.removeItem(rollKey); setLeaderboard([]); setHasRolled(false); setScore(null); setMessage(""); }

function copyShareText() { if (!score) return; const txt = I got ${score}/100 on LuckCast today! 🍀 #LuckCast; navigator.clipboard.writeText(txt).then(() => { setCopied(true); setTimeout(() => setCopied(false), 1800); }); }

const topToday = [...leaderboard].sort((a, b) => b.score - a.score).slice(0, 10);

return ( <div className="min-h-screen flex items-center justify-center p-6 bg-gradient-to-br from-slate-50 to-amber-50"> <div className="max-w-xl w-full bg-white/95 backdrop-blur-md rounded-2xl shadow-2xl p-6"> <header className="flex items-center justify-between mb-4"> <h1 className="text-2xl font-extrabold">🎲 LuckCast</h1> <div className="text-sm text-slate-500">Daily Luck Score • {todayKey}</div> </header>

<p className="text-slate-700 mb-4">Tap once per day, see your luck, and flex on the leaderboard. Be kind — luck is random 😉</p>

    <label className="block mb-3">
      <span className="text-sm text-slate-600">Your Farcaster handle</span>
      <input
        aria-label="handle"
        value={handle}
        onChange={(e) => setHandle(e.target.value)}
        className="mt-1 block w-full rounded-lg border-gray-200 shadow-sm p-3"
        placeholder="@yourhandle"
        disabled={hasRolled}
      />
    </label>

    <div className="flex items-center gap-3 mb-4">
      <button
        onClick={rollLuck}
        className={`flex-1 py-3 rounded-xl font-semibold shadow-md transition-all ${
          hasRolled ? "bg-gray-200 text-gray-700 cursor-default" : "bg-amber-400 hover:bg-amber-500 text-white"
        }`}>
        {hasRolled ? "You already rolled today" : "Check my luck 🍀"}
      </button>

      <button
        onClick={copyShareText}
        disabled={!score}
        className={`px-4 py-3 rounded-xl border ${!score ? "opacity-50 cursor-not-allowed" : "hover:bg-gray-100"}`}>
        {copied ? "Copied!" : "Share"}
      </button>
    </div>

    {score !== null && (
      <div className="mb-4 p-4 rounded-lg border border-dashed">
        <div className="text-4xl font-extrabold">{score}/100</div>
        <div className="text-slate-600 mt-1">{message}</div>
      </div>
    )}

    <section className="mb-4">
      <h2 className="text-lg font-bold mb-2">Top Luck Today</h2>
      {topToday.length === 0 ? (
        <div className="text-slate-500">No rolls yet — be the first to check your luck!</div>
      ) : (
        <ol className="space-y-2">
          {topToday.map((p, i) => (
            <li key={`${p.handle}-${i}`} className="flex items-center justify-between p-2 rounded-md bg-gradient-to-r from-white to-amber-50">
              <div>
                <div className="font-semibold">{p.handle}</div>
                <div className="text-xs text-slate-500">{p.time}</div>
              </div>
              <div className="text-xl font-extrabold">{p.score}</div>
            </li>
          ))}
        </ol>
      )}
    </section>

    <footer className="flex items-center justify-between text-xs text-slate-500">
      <div>Made with ❤️ for Warpcast</div>
      <div className="space-x-2">
        <button onClick={() => alert("Invite friends: share your LuckCast result on Warpcast!")}>How it works</button>
        <button onClick={resetToday} className="ml-2">Reset (debug)</button>
      </div>
    </footer>
  </div>
</div>

); }

