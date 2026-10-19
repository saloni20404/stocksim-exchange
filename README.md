STOCKISM EXCHANGE

A real-time stock exchange simulator: live-updating prices, market and limit orders, candlestick charts, and portfolio tracking. Paper-trade with $100,000 of virtual cash.

Live demo: [https://stocksim-exchange.vercel.app/](https://stocksim-exchange.vercel.app/)

Note: The backend runs on a free tier and sleeps after inactivity. The first load can take about a minute, and the navbar shows "Offline" until the server wakes up.

Everything here is simulated. The companies are made up, prices come from a random-walk model, and the cash is just a number in a database. Nothing is connected to a real market, broker, or payment system.


Features
-25 simulated stocks across 5 sectors (Technology, Finance, Energy, Healthcare, Consumer)
-Prices update every second using Geometric Brownian Motion (GBM), streamed to the browser over WebSockets
-Market and limit orders with an order-matching engine (cash checks, weighted average buy price)
-1-minute OHLCV candles stored in the database and shown in a candlestick chart with volume
-Portfolio view with per-holding and total P&L
-Live ticker, sector filters, search and sorting on the dashboard

Tech Stack
Layer	Tech
-Backend	FastAPI, SQLAlchemy (async), SQLite (aiosqlite), APScheduler, NumPy
-Realtime	WebSockets (topic-based connection manager)
-Frontend	Next.js, React, TypeScript, Tailwind CSS, Zustand, lightweight-charts
-Deploy	Vercel (frontend), Render with Docker (backend)


How It Works
A scheduler calls tick() every second. Each stock's next price is S * exp((mu - 0.5*sigma^2) + sigma * Z) with Z ~ N(0,1).
The new prices are held in memory and broadcast to all clients on /ws/market. They are saved to the database every 10 seconds, and a 1-minute candle is saved every 60 ticks.
Orders go through POST /orders. Market orders fill at the current price. Limit orders fill immediately if the price condition is met, otherwise they stay pending and are checked on each tick.
Fills update the cash balance, the holding, and the trade history, and are broadcast on /ws/trades.

API (summary)
Method	Endpoint	Description
GET	/stocks	All stocks with live prices
GET	/stocks/{symbol}	Single stock details
GET	/stocks/{symbol}/history	OHLCV candles
POST	/orders	Place a market or limit order
GET	/orders	List orders (optional status filter)
DELETE	/orders/{id}	Cancel a pending order
GET	/portfolio	Holdings, cash, P&L
GET	/portfolio/trades	Trade history
WS	/ws/market, /ws/trades	Live price and trade streams

Interactive docs are available at /docs when the backend is running.

Run Locally

You need Python 3.10+ (3.12 recommended) and Node.js 18+.

Backend

bash
cd backend
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --host 127.0.0.1 --port 8000

Check http://127.0.0.1:8000/health, or open http://127.0.0.1:8000/docs.

Frontend (new terminal)

bash
cd frontend
npm install
npm run dev

Open http://localhost:3000. By default the frontend talks to http://localhost:8000. To point it elsewhere, set:

NEXT_PUBLIC_API_URL=https://your-backend-url
NEXT_PUBLIC_WS_URL=wss://your-backend-url

To reset all data, stop the backend and delete backend/stockmarket.db.

Deployment
Backend: Render Web Service, root directory backend, Docker runtime, env var PORT=8000
Frontend: Vercel, root directory frontend, with NEXT_PUBLIC_API_URL and NEXT_PUBLIC_WS_URL set to the Render URL (https://... and wss://...)
Known Limitations
Single shared portfolio: there is no authentication or per-user accounts
The order book in the UI is generated for display and is not built from real orders
SQLite data resets on redeploy on the free hosting tier
