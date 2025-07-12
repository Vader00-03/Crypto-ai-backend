Backend: AI Crypto Trader (Python + FastAPI + Binance)

from fastapi import FastAPI, HTTPException from pydantic import BaseModel import requests import hmac import hashlib import time import os

app = FastAPI()

ENV keys (you should set these securely in production)

BINANCE_API_KEY = os.getenv("BINANCE_API_KEY", "your_api_key") BINANCE_SECRET_KEY = os.getenv("BINANCE_SECRET_KEY", "your_secret_key") BASE_URL = "https://api.binance.com"

class TradeRequest(BaseModel): symbol: str  # Example: "BTCUSDT" amount_usd: float  # Example: 10.0 strategy: str  # "scalping", "trend", or "news"

@app.post("/trade") def place_trade(data: TradeRequest): # 1. Convert USD to quantity of coin to buy price_data = requests.get(f"{BASE_URL}/api/v3/ticker/price", params={"symbol": data.symbol}).json() current_price = float(price_data['price']) qty = round(data.amount_usd / current_price, 6)

# 2. Prepare signed Binance request
timestamp = int(time.time() * 1000)
query = f"symbol={data.symbol}&side=BUY&type=MARKET&quantity={qty}&timestamp={timestamp}"
signature = hmac.new(BINANCE_SECRET_KEY.encode(), query.encode(), hashlib.sha256).hexdigest()

headers = {"X-MBX-APIKEY": BINANCE_API_KEY}
full_url = f"{BASE_URL}/api/v3/order?{query}&signature={signature}"

# 3. Place order
response = requests.post(full_url, headers=headers)
if response.status_code != 200:
    raise HTTPException(status_code=400, detail=response.json())

return {"status": "Trade executed", "data": response.json()}

@app.get("/ping") def ping(): return {"message": "Crypto AI backend is live."}

