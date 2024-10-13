mkdir tradezella-clone
cd tradezella-clone
mkdir backend
cd backend
npm init -y
npm install express mongoose body-parser cors dotenv
// backend/server.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(bodyParser.json());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log("MongoDB connected"))
  .catch(err => console.error("MongoDB connection error:", err));

// Trade Model
const Trade = mongoose.model('Trade', new mongoose.Schema({
  symbol: { type: String, required: true },
  quantity: { type: Number, required: true },
  price: { type: Number, required: true },
}));

// Routes
app.post('/api/trades', async (req, res) => {
  try {
    const trade = new Trade(req.body);
    await trade.save();
    res.status(201).send(trade);
  } catch (error) {
    res.status(400).send(error);
  }
});

app.get('/api/trades', async (req, res) => {
  try {
    const trades = await Trade.find();
    res.status(200).send(trades);
  } catch (error) {
    res.status(500).send(error);
  }
});

// Start Server
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
MONGODB_URI=your_mongodb_connection_string
cd ..
npx create-react-app frontend
cd frontend
npm install axios
// frontend/src/App.js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const App = () => {
  const [trade, setTrade] = useState({ symbol: '', quantity: '', price: '' });
  const [trades, setTrades] = useState([]);

  useEffect(() => {
    const fetchTrades = async () => {
      const response = await axios.get('http://localhost:5000/api/trades');
      setTrades(response.data);
    };
    fetchTrades();
  }, []);

  const handleChange = (e) => {
    setTrade({ ...trade, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await axios.post('http://localhost:5000/api/trades', trade);
    setTrades([...trades, trade]);
    setTrade({ symbol: '', quantity: '', price: '' });
  };

  return (
    <div>
      <h1>Trade Logger</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="symbol"
          value={trade.symbol}
          onChange={handleChange}
          placeholder="Stock Symbol"
          required
        />
        <input
          type="number"
          name="quantity"
          value={trade.quantity}
          onChange={handleChange}
          placeholder="Quantity"
          required
        />
        <input
          type="number"
          name="price"
          value={trade.price}
          onChange={handleChange}
          placeholder="Price"
          required
        />
        <button type="submit">Add Trade</button>
      </form>
      <ul>
        {trades.map((t, index) => (
          <li key={index}>
            {t.symbol} - {t.quantity} shares at ${t.price}
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;
"proxy": "http://localhost:5000",
cd backend
node server.js
cd frontend
npm start
