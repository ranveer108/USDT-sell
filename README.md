import React, { useEffect, useState } from "react";
import { GoogleOAuthProvider, GoogleLogin } from "@react-oauth/google";
import axios from "axios";

const BACKEND_URL = "https://your-railway-url"; // Replace this with your backend URL
const WALLET_ADDRESS = "TSrAqB2LBMHrwJPf7aJDz2EuULRpGve8gi";

export default function App() {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(null);
  const [usdtAmount, setUsdtAmount] = useState(0);
  const [price, setPrice] = useState(null);
  const [transactions, setTransactions] = useState([]);

  const fetchPrice = async () => {
    try {
      const res = await axios.get("https://api.coingecko.com/api/v3/simple/price?ids=tether&vs_currencies=usd");
      setPrice(res.data.tether.usd);
    } catch (e) {
      console.error("Error fetching price", e);
    }
  };

  const fetchTransactions = async () => {
    try {
      const res = await axios.get(`${BACKEND_URL}/admin/transactions`, {
        headers: { Authorization: token },
      });
      setTransactions(res.data);
    } catch (e) {
      console.error("Error fetching transactions");
    }
  };

  const handleSell = async () => {
    if (!price || usdtAmount <= 0) return alert("Enter amount");
    const usdValue = (price * usdtAmount).toFixed(2);
    try {
      await axios.post(
        `${BACKEND_URL}/sell`,
        { usdtAmount, usdValue },
        { headers: { Authorization: token } }
      );
      alert("Sell request submitted!");
      setUsdtAmount(0);
      if (user?.email === "ranveers4593@gmail.com") fetchTransactions();
    } catch (err) {
      alert("Error submitting sell");
    }
  };

  useEffect(() => {
    fetchPrice();
    const interval = setInterval(fetchPrice, 30000);
    return () => clearInterval(interval);
  }, []);

  return (
    <GoogleOAuthProvider clientId="YOUR_GOOGLE_CLIENT_ID">
      <div className="p-4 max-w-xl mx-auto">
        <h1 className="text-3xl font-bold mb-4">USDT Selle</h1>

        {!user ? (
          <GoogleLogin
            onSuccess={(res) => {
              setToken(res.credential);
              const decoded = JSON.parse(atob(res.credential.split(".")[1]));
              setUser(decoded);
              if (decoded.email === "ranveers4593@gmail.com") fetchTransactions();
            }}
            onError={() => alert("Login Failed")}
          />
        ) : (
          <div>
            <p className="mb-2">Welcome, {user.name} ({user.email})</p>

            <div className="mb-4">
              <p className="font-medium">Live USDT Price: ${price || "..."}</p>
              <p>Send USDT to: <span className="font-mono">{WALLET_ADDRESS}</span></p>
              <img
                src="https://chart.googleapis.com/chart?chs=200x200&cht=qr&chl=TSrAqB2LBMHrwJPf7aJDz2EuULRpGve8gi"
                alt="QR Code"
                className="my-2"
              />
            </div>

            <div className="flex gap-2 mb-4">
              <input
                type="number"
                className="border p-2 flex-1"
                value={usdtAmount}
                onChange={(e) => setUsdtAmount(e.target.value)}
                placeholder="Enter USDT to sell"
              />
              <button className="bg-blue-500 text-white p-2 rounded" onClick={handleSell}>
                Sell
              </button>
            </div>

            {user.email === "ranveers4593@gmail.com" && (
              <div>
                <h2 className="text-xl font-semibold mt-6 mb-2">Admin Dashboard</h2>
                <table className="w-full border">
                  <thead>
                    <tr>
                      <th className="border p-2">User</th>
                      <th className="border p-2">Amount</th>
                      <th className="border p-2">USD</th>
                      <th className="border p-2">Time</th>
                    </tr>
                  </thead>
                  <tbody>
                    {transactions.map((t) => (
                      <tr key={t.id}>
                        <td className="border p-2">{t.email}</td>
                        <td className="border p-2">{t.usdtAmount}</td>
                        <td className="border p-2">${t.usdValue}</td>
                        <td className="border p-2">{new Date(t.timestamp).toLocaleString()}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}
          </div>
        )}
      </div>
    </GoogleOAuthProvider>
  );
}
