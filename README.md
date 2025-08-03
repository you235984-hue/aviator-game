// Aviator-Style Crash Game with Leaderboard & Daily Rewards (React + Tailwind + Framer Motion)

import React, { useState, useEffect, useRef } from "react";
import { motion } from "framer-motion";
import { Button } from "@/components/ui/button"; // You need to create this file manually

const getRandomCrash = () => parseFloat((Math.random() * 5 + 1.5).toFixed(2));

export default function AviatorGame() {
  const [multiplier, setMultiplier] = useState(1.0);
  const [isRunning, setIsRunning] = useState(false);
  const [crashPoint, setCrashPoint] = useState(null);
  const [hasCashedOut, setHasCashedOut] = useState(false);
  const [balance, setBalance] = useState(1000);
  const [bet, setBet] = useState(100);
  const [autoCashOut, setAutoCashOut] = useState(2.0);
  const [cashOutValue, setCashOutValue] = useState(null);
  const [history, setHistory] = useState([]);
  const [players, setPlayers] = useState([]);
  const [leaderboard, setLeaderboard] = useState([]);
  const [dailyClaimed, setDailyClaimed] = useState(false);
  const intervalRef = useRef(null);

  const claimDailyReward = () => {
    if (!dailyClaimed) {
      setBalance(prev => prev + 100);
      setDailyClaimed(true);
    }
  };

  const startGame = () => {
    if (isRunning) return;
    setHasCashedOut(false);
    const crash = getRandomCrash();
    setCrashPoint(crash);
    setMultiplier(1.0);
    setIsRunning(true);
    setCashOutValue(null);
    setBalance(prev => prev - bet);
    generateSimulatedPlayers(crash);
  };

  const cashOut = () => {
    if (!isRunning || hasCashedOut) return;
    setHasCashedOut(true);
    const winnings = parseFloat((bet * multiplier).toFixed(2));
    setBalance(prev => prev + winnings);
    setCashOutValue(winnings);
    updateLeaderboard(winnings);
  };

  const generateSimulatedPlayers = crash => {
    const fakePlayers = Array.from({ length: 5 }, (_, i) => {
      const fakeBet = Math.floor(Math.random() * 300 + 50);
      const fakeCashOut = parseFloat((Math.random() * (crash - 0.5) + 1.0).toFixed(2));
      const won = fakeCashOut < crash;
      return {
        name: `Player${i + 1}`,
        bet: fakeBet,
        cashOutAt: fakeCashOut,
        result: won ? (fakeBet * fakeCashOut).toFixed(2) : 0,
      };
    });
    setPlayers(fakePlayers);
  };

  const updateLeaderboard = winnings => {
    setLeaderboard(prev => {
      const updated = [...prev, { player: "You", winnings }];
      return updated.sort((a, b) => b.winnings - a.winnings).slice(0, 5);
    });
  };

  useEffect(() => {
    if (isRunning) {
      intervalRef.current = setInterval(() => {
        setMultiplier(prev => {
          const next = parseFloat((prev + 0.05).toFixed(2));
          if (!hasCashedOut && autoCashOut && next >= autoCashOut) cashOut();
          if (next >= crashPoint) {
            clearInterval(intervalRef.current);
            setIsRunning(false);
            if (!hasCashedOut) setCashOutValue(0);
            setHistory(prev => [crashPoint, ...prev.slice(0, 9)]);
          }
          return next;
        });
      }, 100);
    }
    return () => clearInterval(intervalRef.current);
  }, [isRunning, crashPoint]);

  return (
    <div className="h-full bg-gradient-to-b from-blue-200 to-blue-500 p-6 flex flex-col items-center">
      <h1 className="text-4xl font-bold text-white mb-4">🚀 Aviator Crash Game</h1>

      <div className="mb-2 text-white">Balance: ₹{balance.toFixed(2)}</div>
      <div className="mb-2 text-white">Bet: ₹{bet} | Auto Cash-Out at: {autoCashOut}x</div>

      <div className="flex gap-2 mb-4">
        <input type="number" value={autoCashOut} step="0.1" min="1" onChange={e => setAutoCashOut(parseFloat(e.target.value))} className="p-1 rounded" />
        {!isRunning && (
          <Button onClick={startGame} className="bg-green-600 hover:bg-green-700 text-white">Place Bet</Button>
        )}
        {isRunning && !hasCashedOut && (
          <Button onClick={cashOut} className="bg-yellow-500 hover:bg-yellow-600 text-white">Cash Out</Button>
        )}
        <Button onClick={claimDailyReward} disabled={dailyClaimed} className="bg-purple-600 hover:bg-purple-700 text-white">
          {dailyClaimed ? "Reward Claimed" : "Claim Daily ₹100"}
        </Button>
      </div>

      <div className="w-full max-w-xl h-40 bg-white rounded-xl shadow-inner overflow-hidden mb-4 relative">
        <motion.div
          className="absolute top-1/2 left-10 text-xl font-mono text-red-600"
          animate={{ x: isRunning ? [0, 350] : 0 }}
          transition={{ duration: crashPoint ?? 5, ease: "linear" }}
        >
          🚀 {multiplier.toFixed(2)}x
        </motion.div>
      </div>

      {cashOutValue !== null && (
        <div className="mb-4 bg-white p-2 rounded-xl shadow text-center font-semibold">
          {cashOutValue > 0 ? `✅ You won ₹${cashOutValue}` : "💥 Crashed! You lost your bet."}
        </div>
      )}

      <div className="flex flex-col md:flex-row gap-6 w-full max-w-5xl mt-4">
        <div className="flex-1 bg-white p-4 rounded shadow">
          <h2 className="text-lg font-bold mb-2">📉 Crash History</h2>
          <div className="flex gap-2 flex-wrap">
            {history.map((val, idx) => (
              <span key={idx} className="bg-blue-100 px-2 py-1 rounded text-sm">{val}x</span>
            ))}
          </div>
        </div>

        <div className="flex-1 bg-white p-4 rounded shadow">
          <h2 className="text-lg font-bold mb-2">👥 Simulated Players</h2>
          <table className="text-sm w-full">
            <thead>
              <tr><th>Name</th><th>Bet</th><th>Cash Out</th><th>Winnings</th></tr>
            </thead>
            <tbody>
              {players.map((p, i) => (
                <tr key={i} className="text-center">
                  <td>{p.name}</td><td>₹{p.bet}</td><td>{p.cashOutAt}x</td><td>₹{p.result}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>

        <div className="flex-1 bg-white p-4 rounded shadow">
          <h2 className="text-lg font-bold mb-2">🏆 Leaderboard</h2>
          <ol className="list-decimal pl-5">
            {leaderboard.map((entry, i) => (
              <li key={i}>{entry.player}: ₹{entry.winnings.toFixed(2)}</li>
            ))}
          </ol>
        </div>
      </div>
    </div>
  );
}