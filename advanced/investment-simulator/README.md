# Simulated Investment Platform

<p align="center">
  <img src="https://images.unsplash.com/photo-1611974789855-9c2a0a7236a3?w=600&q=80" alt="Investment Trading" width="600">
</p>

## Overview

**Difficulty:** Advanced  
**Estimated Time:** 30–40 hours  
**Prerequisites:** Node.js, React, WebSockets, Financial APIs, Chart.js/D3.js, Real-time data processing  

A **Simulated Investment Platform** is a comprehensive trading simulator that allows users to practice investing with real market data without financial risk. Users can:

- Trade stocks, cryptocurrencies, forex, and other assets
- View real-time market data and price charts
- Build and manage investment portfolios
- Compete with other traders on leaderboards
- Learn technical analysis with indicators and patterns
- Backtest trading strategies
- Receive educational content and market news

This project covers financial API integration, real-time data streaming with WebSockets, complex charting with candlesticks and technical indicators, portfolio calculations, risk analysis, and building a scalable trading platform similar to Robinhood, E*TRADE, or TradingView.

---

## Features

### Must Have

- User authentication and investor profiles
- Virtual cash balance ($10,000 starting capital)
- Search for stocks/assets with autocomplete
- Real-time price quotes and updates
- Buy and sell orders (market orders)
- Portfolio overview (holdings, P&L, allocation)
- Transaction history
- Interactive price charts with:
  - Multiple timeframes (1D, 5D, 1M, 3M, 1Y, All)
  - Candlestick charts
  - Volume bars
  - Moving averages (SMA, EMA)
- Watchlist functionality
- Leaderboard (top investors by return %)
- Responsive design

### Nice to Have

- Multiple asset classes (stocks, crypto, forex, commodities)
- Advanced order types (limit, stop-loss, stop-limit)
- Technical indicators (RSI, MACD, Bollinger Bands, Fibonacci)
- Drawing tools on charts (trend lines, support/resistance)
- News feed integrated with assets
- Sentiment analysis of news
- Price alerts and notifications
- Paper trading competitions with prizes
- Social trading (follow and copy successful traders)
- Strategy backtesting engine
- Risk analysis (Sharpe ratio, max drawdown)
- Dividend tracking
- Options trading simulator
- Margin trading simulation
- Portfolio rebalancing suggestions
- Tax-loss harvesting calculator
- Educational academy with courses
- Mobile app (React Native)
- Real-time chat and discussion forums

---

## Learning Objectives

By building this project, you will learn:

- **Financial API integration** (Alpha Vantage, Finnhub, Yahoo Finance, Polygon.io)
- **Real-time data streaming** with WebSockets
- **Financial charting** with candlesticks, volume, and technical indicators
- **Portfolio management** calculations (P&L, returns, allocation)
- **Technical analysis** implementation (RSI, MACD, moving averages)
- **Market data processing** and normalization
- **WebSocket** connection management and reconnection
- **Complex state management** for real-time data
- **Performance optimization** for high-frequency data updates
- **Financial calculations** (compound returns, risk metrics)
- **Caching strategies** for market data
- **Rate limiting** and API quota management
- **Database design** for financial data
- **Testing strategies** for financial applications

---

## Project Concepts

This project simulates a real trading platform and covers:

- **Market Data Streaming:** Real-time price updates via WebSocket
- **Order Execution:** Buy/sell logic with price calculation
- **Portfolio Accounting:** Track positions, P&L, realized/unrealized gains
- **Technical Analysis:** Calculate and display indicators
- **Risk Management:** Position sizing, stop-losses, portfolio diversification
- **Market Simulation:** Handle market hours, after-hours trading, holidays
- **Data Normalization:** Handle different API formats consistently

---

## Step-by-Step Implementation

### Step 1: Financial API Integration

Set up connections to market data providers:

```javascript
// services/marketDataService.js
const axios = require('axios');
const WebSocket = require('ws');
const Redis = require('ioredis');

class MarketDataService {
    constructor() {
        this.redis = new Redis(process.env.REDIS_URL);
        this.apiKey = process.env.ALPHA_VANTAGE_API_KEY;
        this.wsConnections = new Map();
        this.subscribers = new Map();
    }
    
    // Get real-time quote
    async getQuote(symbol) {
        try {
            // Check cache first
            const cached = await this.redis.get(`quote:${symbol}`);
            if (cached) {
                return JSON.parse(cached);
            }
            
            // Fetch from API
            const response = await axios.get(
                `https://www.alphavantage.co/query`, {
                    params: {
                        function: 'GLOBAL_QUOTE',
                        symbol: symbol,
                        apikey: this.apiKey
                    }
                }
            );
            
            const data = response.data['Global Quote'];
            
            const quote = {
                symbol: data['01. symbol'],
                price: parseFloat(data['05. price']),
                change: parseFloat(data['09. change']),
                changePercent: parseFloat(data['10. change percent'].replace('%', '')),
                volume: parseInt(data['06. volume']),
                previousClose: parseFloat(data['08. previous close']),
                timestamp: new Date(data['07. latest trading day'])
            };
            
            // Cache for 5 seconds
            await this.redis.setex(
                `quote:${symbol}`, 
                5, 
                JSON.stringify(quote)
            );
            
            return quote;
            
        } catch (error) {
            console.error(`Error fetching quote for ${symbol}:`, error);
            throw error;
        }
    }
    
    // Get historical data
    async getHistoricalData(symbol, interval = 'daily', outputSize = 'compact') {
        try {
            const cacheKey = `history:${symbol}:${interval}:${outputSize}`;
            const cached = await this.redis.get(cacheKey);
            
            if (cached) {
                return JSON.parse(cached);
            }
            
            const functionMap = {
                '1min': 'TIME_SERIES_INTRADAY',
                '5min': 'TIME_SERIES_INTRADAY',
                '15min': 'TIME_SERIES_INTRADAY',
                '30min': 'TIME_SERIES_INTRADAY',
                '60min': 'TIME_SERIES_INTRADAY',
                'daily': 'TIME_SERIES_DAILY',
                'weekly': 'TIME_SERIES_WEEKLY',
                'monthly': 'TIME_SERIES_MONTHLY'
            };
            
            const params = {
                function: functionMap[interval],
                symbol: symbol,
                apikey: this.apiKey,
                outputsize: outputSize
            };
            
            if (interval.includes('min')) {
                params.interval = interval;
            }
            
            const response = await axios.get(
                `https://www.alphavantage.co/query`,
                { params }
            );
            
            const timeSeriesKey = Object.keys(response.data).find(key => 
                key.includes('Time Series')
            );
            
            const rawData = response.data[timeSeriesKey];
            
            // Transform to consistent format
            const historicalData = Object.entries(rawData).map(([date, values]) => ({
                date: new Date(date),
                timestamp: new Date(date).getTime(),
                open: parseFloat(values['1. open']),
                high: parseFloat(values['2. high']),
                low: parseFloat(values['3. low']),
                close: parseFloat(values['4. close']),
                volume: parseInt(values['5. volume'])
            })).sort((a, b) => a.timestamp - b.timestamp);
            
            // Cache for 1 hour
            await this.redis.setex(
                cacheKey,
                3600,
                JSON.stringify(historicalData)
            );
            
            return historicalData;
            
        } catch (error) {
            console.error(`Error fetching historical data for ${symbol}:`, error);
            throw error;
        }
    }
    
    // Search symbols
    async searchSymbols(query) {
        try {
            const response = await axios.get(
                `https://www.alphavantage.co/query`, {
                    params: {
                        function: 'SYMBOL_SEARCH',
                        keywords: query,
                        apikey: this.apiKey
                    }
                }
            );
            
            return response.data.bestMatches.map(match => ({
                symbol: match['1. symbol'],
                name: match['2. name'],
                type: match['3. type'],
                region: match['4. region'],
                currency: match['8. currency']
            }));
            
        } catch (error) {
            console.error(`Error searching symbols for ${query}:`, error);
            throw error;
        }
    }
    
    // WebSocket real-time streaming
    subscribeToRealTimeData(symbol, callback) {
        const wsUrl = `wss://ws.finnhub.io?token=${process.env.FINNHUB_API_KEY}`;
        
        let ws = this.wsConnections.get(symbol);
        
        if (!ws) {
            ws = new WebSocket(wsUrl);
            
            ws.on('open', () => {
                console.log(`WebSocket connected for ${symbol}`);
                ws.send(JSON.stringify({
                    type: 'subscribe',
                    symbol: symbol
                }));
            });
            
            ws.on('message', (data) => {
                const message = JSON.parse(data);
                
                if (message.type === 'trade') {
                    message.data.forEach(trade => {
                        const subscribers = this.subscribers.get(symbol) || [];
                        subscribers.forEach(cb => cb({
                            symbol: trade.s,
                            price: trade.p,
                            volume: trade.v,
                            timestamp: trade.t
                        }));
                    });
                }
            });
            
            ws.on('error', (error) => {
                console.error(`WebSocket error for ${symbol}:`, error);
            });
            
            ws.on('close', () => {
                console.log(`WebSocket closed for ${symbol}`);
                this.wsConnections.delete(symbol);
                
                // Attempt reconnection after 5 seconds
                setTimeout(() => {
                    if (this.subscribers.has(symbol) && this.subscribers.get(symbol).length > 0) {
                        this.subscribeToRealTimeData(symbol, callback);
                    }
                }, 5000);
            });
            
            this.wsConnections.set(symbol, ws);
        }
        
        // Add callback to subscribers
        if (!this.subscribers.has(symbol)) {
            this.subscribers.set(symbol, []);
        }
        this.subscribers.get(symbol).push(callback);
        
        // Return unsubscribe function
        return () => {
            const subs = this.subscribers.get(symbol);
            const index = subs.indexOf(callback);
            if (index > -1) {
                subs.splice(index, 1);
            }
            
            // Close WebSocket if no more subscribers
            if (subs.length === 0) {
                const ws = this.wsConnections.get(symbol);
                if (ws) {
                    ws.send(JSON.stringify({
                        type: 'unsubscribe',
                        symbol: symbol
                    }));
                    ws.close();
                    this.wsConnections.delete(symbol);
                }
                this.subscribers.delete(symbol);
            }
        };
    }
    
    // Calculate technical indicators
    calculateSMA(data, period) {
        const sma = [];
        for (let i = period - 1; i < data.length; i++) {
            const sum = data.slice(i - period + 1, i + 1)
                .reduce((acc, val) => acc + val.close, 0);
            sma.push({
                date: data[i].date,
                value: sum / period
            });
        }
        return sma;
    }
    
    calculateEMA(data, period) {
        const multiplier = 2 / (period + 1);
        const ema = [];
        
        // First EMA is SMA
        const firstSMA = data.slice(0, period)
            .reduce((acc, val) => acc + val.close, 0) / period;
        ema.push({
            date: data[period - 1].date,
            value: firstSMA
        });
        
        // Calculate EMA for remaining data
        for (let i = period; i < data.length; i++) {
            const value = (data[i].close - ema[ema.length - 1].value) * multiplier + 
                         ema[ema.length - 1].value;
            ema.push({
                date: data[i].date,
                value: value
            });
        }
        
        return ema;
    }
    
    calculateRSI(data, period = 14) {
        const changes = [];
        for (let i = 1; i < data.length; i++) {
            changes.push(data[i].close - data[i - 1].close);
        }
        
        const rsi = [];
        
        for (let i = period; i < changes.length; i++) {
            const gains = [];
            const losses = [];
            
            for (let j = i - period; j < i; j++) {
                if (changes[j] > 0) {
                    gains.push(changes[j]);
                } else {
                    losses.push(Math.abs(changes[j]));
                }
            }
            
            const avgGain = gains.reduce((a, b) => a + b, 0) / period;
            const avgLoss = losses.reduce((a, b) => a + b, 0) / period;
            
            const rs = avgGain / (avgLoss || 0.0001); // Avoid division by zero
            const rsiValue = 100 - (100 / (1 + rs));
            
            rsi.push({
                date: data[i + 1].date,
                value: rsiValue
            });
        }
        
        return rsi;
    }
    
    calculateMACD(data, fastPeriod = 12, slowPeriod = 26, signalPeriod = 9) {
        const fastEMA = this.calculateEMA(data, fastPeriod);
        const slowEMA = this.calculateEMA(data, slowPeriod);
        
        // Calculate MACD line
        const macd = [];
        const startIndex = slowPeriod - fastPeriod;
        
        for (let i = 0; i < slowEMA.length; i++) {
            macd.push({
                date: slowEMA[i].date,
                value: fastEMA[i + startIndex].value - slowEMA[i].value
            });
        }
        
        // Calculate signal line (EMA of MACD)
        const signal = this.calculateEMA(
            macd.map(m => ({ close: m.value, date: m.date })),
            signalPeriod
        );
        
        // Calculate histogram
        const histogram = [];
        for (let i = 0; i < signal.length; i++) {
            histogram.push({
                date: signal[i].date,
                value: macd[i + (macd.length - signal.length)].value - signal[i].value
            });
        }
        
        return { macd, signal, histogram };
    }
}

module.exports = new MarketDataService();
```

### Step 2: Portfolio Management System

Implement buy/sell logic and portfolio tracking:

```javascript
// services/portfolioService.js
const Portfolio = require('../models/Portfolio');
const Transaction = require('../models/Transaction');
const Position = require('../models/Position');
const marketDataService = require('./marketDataService');

class PortfolioService {
    async createPortfolio(userId, initialBalance = 10000) {
        try {
            const portfolio = await Portfolio.create({
                userId,
                cash: initialBalance,
                initialBalance,
                totalValue: initialBalance,
                totalReturn: 0,
                totalReturnPercent: 0
            });
            
            return portfolio;
        } catch (error) {
            console.error('Error creating portfolio:', error);
            throw error;
        }
    }
    
    async getPortfolio(userId) {
        try {
            let portfolio = await Portfolio.findOne({ userId })
                .populate('positions.symbol');
            
            if (!portfolio) {
                portfolio = await this.createPortfolio(userId);
            }
            
            // Update portfolio values with current prices
            await this.updatePortfolioValues(portfolio);
            
            return portfolio;
        } catch (error) {
            console.error('Error getting portfolio:', error);
            throw error;
        }
    }
    
    async buyStock(userId, symbol, quantity, orderType = 'market', limitPrice = null) {
        try {
            const portfolio = await Portfolio.findOne({ userId });
            
            if (!portfolio) {
                throw new Error('Portfolio not found');
            }
            
            // Get current price
            const quote = await marketDataService.getQuote(symbol);
            const price = orderType === 'market' ? quote.price : limitPrice;
            
            if (!price) {
                throw new Error('Invalid price');
            }
            
            const totalCost = price * quantity;
            
            // Check if user has enough cash
            if (portfolio.cash < totalCost) {
                throw new Error('Insufficient funds');
            }
            
            // Update or create position
            let position = await Position.findOne({
                portfolioId: portfolio._id,
                symbol: symbol
            });
            
            if (position) {
                // Update existing position (average cost)
                const newTotalShares = position.shares + quantity;
                const newAvgCost = (
                    (position.averageCost * position.shares) + 
                    (price * quantity)
                ) / newTotalShares;
                
                position.shares = newTotalShares;
                position.averageCost = newAvgCost;
                await position.save();
            } else {
                // Create new position
                position = await Position.create({
                    portfolioId: portfolio._id,
                    symbol: symbol,
                    shares: quantity,
                    averageCost: price
                });
            }
            
            // Deduct cash
            portfolio.cash -= totalCost;
            await portfolio.save();
            
            // Record transaction
            const transaction = await Transaction.create({
                userId: userId,
                portfolioId: portfolio._id,
                type: 'buy',
                symbol: symbol,
                quantity: quantity,
                price: price,
                total: totalCost,
                orderType: orderType,
                status: 'completed'
            });
            
            // Update portfolio values
            await this.updatePortfolioValues(portfolio);
            
            return {
                transaction,
                position,
                portfolio
            };
            
        } catch (error) {
            console.error('Error buying stock:', error);
            throw error;
        }
    }
    
    async sellStock(userId, symbol, quantity, orderType = 'market', limitPrice = null) {
        try {
            const portfolio = await Portfolio.findOne({ userId });
            
            if (!portfolio) {
                throw new Error('Portfolio not found');
            }
            
            // Check if user has the position
            const position = await Position.findOne({
                portfolioId: portfolio._id,
                symbol: symbol
            });
            
            if (!position || position.shares < quantity) {
                throw new Error('Insufficient shares');
            }
            
            // Get current price
            const quote = await marketDataService.getQuote(symbol);
            const price = orderType === 'market' ? quote.price : limitPrice;
            
            if (!price) {
                throw new Error('Invalid price');
            }
            
            const totalProceeds = price * quantity;
            
            // Calculate realized P&L
            const costBasis = position.averageCost * quantity;
            const realizedPL = totalProceeds - costBasis;
            
            // Update position
            position.shares -= quantity;
            
            if (position.shares === 0) {
                // Close position
                await Position.deleteOne({ _id: position._id });
            } else {
                await position.save();
            }
            
            // Add cash
            portfolio.cash += totalProceeds;
            portfolio.realizedPL = (portfolio.realizedPL || 0) + realizedPL;
            await portfolio.save();
            
            // Record transaction
            const transaction = await Transaction.create({
                userId: userId,
                portfolioId: portfolio._id,
                type: 'sell',
                symbol: symbol,
                quantity: quantity,
                price: price,
                total: totalProceeds,
                realizedPL: realizedPL,
                orderType: orderType,
                status: 'completed'
            });
            
            // Update portfolio values
            await this.updatePortfolioValues(portfolio);
            
            return {
                transaction,
                portfolio,
                realizedPL
            };
            
        } catch (error) {
            console.error('Error selling stock:', error);
            throw error;
        }
    }
    
    async updatePortfolioValues(portfolio) {
        try {
            const positions = await Position.find({ portfolioId: portfolio._id });
            
            let totalEquityValue = 0;
            
            // Update each position with current price
            for (const position of positions) {
                const quote = await marketDataService.getQuote(position.symbol);
                
                position.currentPrice = quote.price;
                position.marketValue = quote.price * position.shares;
                position.unrealizedPL = (quote.price - position.averageCost) * position.shares;
                position.unrealizedPLPercent = 
                    ((quote.price - position.averageCost) / position.averageCost) * 100;
                
                await position.save();
                
                totalEquityValue += position.marketValue;
            }
            
            // Update portfolio totals
            portfolio.totalValue = portfolio.cash + totalEquityValue;
            portfolio.totalReturn = portfolio.totalValue - portfolio.initialBalance;
            portfolio.totalReturnPercent = 
                (portfolio.totalReturn / portfolio.initialBalance) * 100;
            
            await portfolio.save();
            
            return portfolio;
            
        } catch (error) {
            console.error('Error updating portfolio values:', error);
            throw error;
        }
    }
    
    async getTransactionHistory(userId, limit = 50) {
        try {
            const transactions = await Transaction.find({ userId })
                .sort({ createdAt: -1 })
                .limit(limit);
            
            return transactions;
        } catch (error) {
            console.error('Error getting transaction history:', error);
            throw error;
        }
    }
    
    async getPortfolioPerformance(userId, period = '1M') {
        try {
            const portfolio = await Portfolio.findOne({ userId });
            
            // Get historical portfolio values
            // This would require storing daily snapshots in a separate collection
            // For simplicity, calculate based on current positions
            
            const positions = await Position.find({ portfolioId: portfolio._id });
            const performance = [];
            
            for (const position of positions) {
                const historicalData = await marketDataService.getHistoricalData(
                    position.symbol,
                    'daily',
                    period === '1M' ? 'compact' : 'full'
                );
                
                // Calculate position value over time
                const positionPerformance = historicalData.map(data => ({
                    date: data.date,
                    value: data.close * position.shares
                }));
                
                performance.push({
                    symbol: position.symbol,
                    data: positionPerformance
                });
            }
            
            return performance;
            
        } catch (error) {
            console.error('Error getting portfolio performance:', error);
            throw error;
        }
    }
    
    // Risk metrics
    async calculateRiskMetrics(userId) {
        try {
            const portfolio = await Portfolio.findOne({ userId });
            const positions = await Position.find({ portfolioId: portfolio._id });
            
            // Calculate portfolio beta, Sharpe ratio, max drawdown, etc.
            // This would require historical data and benchmark comparison
            
            const metrics = {
                portfolioValue: portfolio.totalValue,
                cash: portfolio.cash,
                investedAmount: portfolio.totalValue - portfolio.cash,
                allocatedPercent: ((portfolio.totalValue - portfolio.cash) / portfolio.totalValue) * 100,
                numberOfPositions: positions.length,
                largestPosition: positions.reduce((max, pos) => 
                    pos.marketValue > max.marketValue ? pos : max, 
                    { marketValue: 0 }
                ),
                concentration: positions.map(pos => ({
                    symbol: pos.symbol,
                    allocation: (pos.marketValue / portfolio.totalValue) * 100
                }))
            };
            
            return metrics;
            
        } catch (error) {
            console.error('Error calculating risk metrics:', error);
            throw error;
        }
    }
}

module.exports = new PortfolioService();
```

### Step 3: Interactive Chart Component

Build candlestick charts with technical indicators:

```jsx
// components/StockChart.jsx
import React, { useState, useEffect, useRef } from 'react';
import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    TimeScale,
    BarElement,
    LineElement,
    PointElement,
    Title,
    Tooltip,
    Legend
} from 'chart.js';
import { Chart } from 'react-chartjs-2';
import 'chartjs-chart-financial';
import { CandlestickController, CandlestickElement } from 'chartjs-chart-financial';
import zoomPlugin from 'chartjs-plugin-zoom';

ChartJS.register(
    CategoryScale,
    LinearScale,
    TimeScale,
    BarElement,
    LineElement,
    PointElement,
    CandlestickController,
    CandlestickElement,
    Title,
    Tooltip,
    Legend,
    zoomPlugin
);

function StockChart({ symbol, interval = '1D', showIndicators = {} }) {
    const [chartData, setChartData] = useState(null);
    const [historicalData, setHistoricalData] = useState([]);
    const [indicators, setIndicators] = useState({});
    const [loading, setLoading] = useState(true);
    const chartRef = useRef(null);
    
    useEffect(() => {
        loadChartData();
    }, [symbol, interval]);
    
    useEffect(() => {
        if (historicalData.length > 0) {
            calculateIndicators();
        }
    }, [historicalData, showIndicators]);
    
    const loadChartData = async () => {
        try {
            setLoading(true);
            
            const response = await fetch(
                `/api/market-data/historical/${symbol}?interval=${interval}`
            );
            const data = await response.json();
            
            setHistoricalData(data);
            prepareChartData(data);
            
            setLoading(false);
        } catch (error) {
            console.error('Error loading chart data:', error);
            setLoading(false);
        }
    };
    
    const prepareChartData = (data) => {
        const candlestickData = data.map(d => ({
            x: d.date,
            o: d.open,
            h: d.high,
            l: d.low,
            c: d.close
        }));
        
        const volumeData = data.map(d => ({
            x: d.date,
            y: d.volume
        }));
        
        const datasets = [
            {
                label: symbol,
                type: 'candlestick',
                data: candlestickData,
                borderColor: {
                    up: '#26a69a',
                    down: '#ef5350',
                    unchanged: '#999'
                },
                backgroundColor: {
                    up: 'rgba(38, 166, 154, 0.5)',
                    down: 'rgba(239, 83, 80, 0.5)',
                    unchanged: 'rgba(153, 153, 153, 0.5)'
                }
            },
            {
                label: 'Volume',
                type: 'bar',
                data: volumeData,
                backgroundColor: 'rgba(100, 149, 237, 0.3)',
                yAxisID: 'volume'
            }
        ];
        
        setChartData({
            datasets: datasets
        });
    };
    
    const calculateIndicators = async () => {
        const newIndicators = {};
        
        if (showIndicators.sma) {
            const response = await fetch(
                `/api/indicators/sma/${symbol}?period=${showIndicators.sma}&interval=${interval}`
            );
            newIndicators.sma = await response.json();
        }
        
        if (showIndicators.ema) {
            const response = await fetch(
                `/api/indicators/ema/${symbol}?period=${showIndicators.ema}&interval=${interval}`
            );
            newIndicators.ema = await response.json();
        }
        
        if (showIndicators.rsi) {
            const response = await fetch(
                `/api/indicators/rsi/${symbol}?period=14&interval=${interval}`
            );
            newIndicators.rsi = await response.json();
        }
        
        if (showIndicators.macd) {
            const response = await fetch(
                `/api/indicators/macd/${symbol}?interval=${interval}`
            );
            newIndicators.macd = await response.json();
        }
        
        setIndicators(newIndicators);
        addIndicatorsToChart(newIndicators);
    };
    
    const addIndicatorsToChart = (indicators) => {
        if (!chartData) return;
        
        const newDatasets = [...chartData.datasets];
        
        if (indicators.sma) {
            newDatasets.push({
                label: `SMA (${showIndicators.sma})`,
                type: 'line',
                data: indicators.sma.map(d => ({ x: d.date, y: d.value })),
                borderColor: 'rgba(255, 159, 64, 1)',
                backgroundColor: 'transparent',
                borderWidth: 2,
                pointRadius: 0
            });
        }
        
        if (indicators.ema) {
            newDatasets.push({
                label: `EMA (${showIndicators.ema})`,
                type: 'line',
                data: indicators.ema.map(d => ({ x: d.date, y: d.value })),
                borderColor: 'rgba(153, 102, 255, 1)',
                backgroundColor: 'transparent',
                borderWidth: 2,
                pointRadius: 0
            });
        }
        
        setChartData({ datasets: newDatasets });
    };
    
    const chartOptions = {
        responsive: true,
        maintainAspectRatio: false,
        interaction: {
            mode: 'index',
            intersect: false,
        },
        plugins: {
            legend: {
                display: true,
                position: 'top',
            },
            tooltip: {
                enabled: true,
                mode: 'index',
                callbacks: {
                    label: function(context) {
                        if (context.dataset.type === 'candlestick') {
                            const data = context.raw;
                            return [
                                `Open: $${data.o.toFixed(2)}`,
                                `High: $${data.h.toFixed(2)}`,
                                `Low: $${data.l.toFixed(2)}`,
                                `Close: $${data.c.toFixed(2)}`
                            ];
                        }
                        return context.dataset.label + ': $' + context.parsed.y.toFixed(2);
                    }
                }
            },
            zoom: {
                zoom: {
                    wheel: {
                        enabled: true,
                    },
                    pinch: {
                        enabled: true
                    },
                    mode: 'x',
                },
                pan: {
                    enabled: true,
                    mode: 'x',
                }
            }
        },
        scales: {
            x: {
                type: 'time',
                time: {
                    unit: interval === '1D' ? 'minute' : 'day',
                    displayFormats: {
                        minute: 'HH:mm',
                        day: 'MMM dd'
                    }
                },
                ticks: {
                    source: 'auto',
                    maxRotation: 0,
                    autoSkip: true
                }
            },
            y: {
                type: 'linear',
                position: 'right',
                ticks: {
                    callback: function(value) {
                        return '$' + value.toFixed(2);
                    }
                }
            },
            volume: {
                type: 'linear',
                position: 'left',
                display: true,
                grid: {
                    display: false
                },
                max: Math.max(...(chartData?.datasets[1]?.data.map(d => d.y) || [1])) * 4,
                ticks: {
                    callback: function(value) {
                        return (value / 1000000).toFixed(1) + 'M';
                    }
                }
            }
        }
    };
    
    if (loading) {
        return <div className="chart-loading">Loading chart...</div>;
    }
    
    return (
        <div className="stock-chart-container">
            <div className="chart-header">
                <h2>{symbol}</h2>
                <div className="chart-controls">
                    <button 
                        className={interval === '1D' ? 'active' : ''}
                        onClick={() => setInterval('1D')}
                    >
                        1D
                    </button>
                    <button 
                        className={interval === '5D' ? 'active' : ''}
                        onClick={() => setInterval('5D')}
                    >
                        5D
                    </button>
                    <button 
                        className={interval === '1M' ? 'active' : ''}
                        onClick={() => setInterval('1M')}
                    >
                        1M
                    </button>
                    <button 
                        className={interval === '3M' ? 'active' : ''}
                        onClick={() => setInterval('3M')}
                    >
                        3M
                    </button>
                    <button 
                        className={interval === '1Y' ? 'active' : ''}
                        onClick={() => setInterval('1Y')}
                    >
                        1Y
                    </button>
                </div>
            </div>
            
            <div className="chart-wrapper" style={{ height: '500px' }}>
                {chartData && (
                    <Chart
                        ref={chartRef}
                        type='candlestick'
                        data={chartData}
                        options={chartOptions}
                    />
                )}
            </div>
            
            {/* RSI Indicator */}
            {indicators.rsi && (
                <div className="indicator-chart" style={{ height: '150px' }}>
                    <Chart
                        type='line'
                        data={{
                            datasets: [{
                                label: 'RSI',
                                data: indicators.rsi.map(d => ({ x: d.date, y: d.value })),
                                borderColor: 'rgba(75, 192, 192, 1)',
                                backgroundColor: 'transparent',
                                borderWidth: 2
                            }]
                        }}
                        options={{
                            responsive: true,
                            maintainAspectRatio: false,
                            scales: {
                                y: {
                                    min: 0,
                                    max: 100,
                                    ticks: {
                                        callback: function(value) {
                                            return value;
                                        }
                                    }
                                }
                            },
                            plugins: {
                                annotation: {
                                    annotations: {
                                        line1: {
                                            type: 'line',
                                            yMin: 70,
                                            yMax: 70,
                                            borderColor: 'red',
                                            borderWidth: 1,
                                            borderDash: [5, 5]
                                        },
                                        line2: {
                                            type: 'line',
                                            yMin: 30,
                                            yMax: 30,
                                            borderColor: 'green',
                                            borderWidth: 1,
                                            borderDash: [5, 5]
                                        }
                                    }
                                }
                            }
                        }}
                    />
                </div>
            )}
        </div>
    );
}

export default StockChart;
```

## Technology Stack

### Frontend
- **React** - UI framework
- **Chart.js** with financial plugin - Candlestick charts
- **Socket.io-client** - Real-time data
- **Redux Toolkit** - State management
- **React Query** - Server state
- **Tailwind CSS** - Styling
- **date-fns** - Date formatting

### Backend
- **Node.js + Express** - API server
- **MongoDB** - Database
- **Redis** - Caching market data
- **Socket.io** - WebSocket server
- **Bull** - Job queue for data updates
- **JWT** - Authentication

### External APIs
- **Alpha Vantage** - Stock data
- **Finnhub** - Real-time WebSocket data
- **Polygon.io** - Alternative data source
- **News API** - Financial news

## Testing Checklist

- [ ] Market data fetching works correctly
- [ ] Real-time price updates via WebSocket
- [ ] Buy/sell orders execute properly
- [ ] Portfolio calculations are accurate
- [ ] Charts display correctly with all timeframes
- [ ] Technical indicators calculate correctly
- [ ] Transaction history persists
- [ ] Leaderboard ranks users accurately
- [ ] Rate limiting prevents API abuse
- [ ] Error handling for API failures
- [ ] Responsive design works on mobile

## Additional Challenges

### Easy Extensions
- Add watchlist with price alerts
- Implement portfolio charts (pie, line)
- Add market news feed
- Create asset comparison tool
- Add export functionality (CSV, PDF)

### Medium Extensions
- Multi-asset trading (crypto, forex)
- Limit and stop-loss orders
- Strategy backtesting engine
- Social features (follow traders, copy trades)
- Advanced charting tools (drawing, patterns)
- Dividend tracking
- Tax reporting

### Hard Extensions
- Options trading simulator
- Margin trading with leverage
- Automated trading bots
- Machine learning price predictions
- Sentiment analysis of news/social media
- High-frequency trading simulation
- Multi-currency support
- Mobile app with React Native

## Common Pitfalls

1. **API Rate Limits:** Cache aggressively. Use Redis with appropriate TTLs.

2. **Real-time Performance:** Throttle WebSocket updates. Don't update UI on every tick.

3. **Financial Calculations:** Use libraries like `decimal.js` for precision. Avoid floating-point errors.

4. **Market Hours:** Handle pre-market, after-hours, weekends, holidays correctly.

5. **Data Quality:** Different APIs return different formats. Normalize data consistently.

## Resources

### Documentation
- [Alpha Vantage API](https://www.alphavantage.co/documentation/)
- [Finnhub API](https://finnhub.io/docs/api)
- [Chart.js Financial](https://github.com/chartjs/chartjs-chart-financial)

### Tutorials
- [Building Trading Platforms](https://www.youtube.com/results?search_query=stock+trading+platform+tutorial)
- [Technical Analysis in JavaScript](https://github.com/anandanand84/technicalindicators)

### Design Inspiration
- [Robinhood](https://robinhood.com/)
- [TradingView](https://www.tradingview.com/)
- [Webull](https://www.webull.com/)

## Submission Checklist

- [ ] Real-time market data integration
- [ ] Buy/sell functionality
- [ ] Portfolio management
- [ ] Interactive candlestick charts
- [ ] Technical indicators (SMA, EMA, RSI, MACD)
- [ ] Transaction history
- [ ] Leaderboard system
- [ ] Responsive design
- [ ] Error handling
- [ ] Documentation

## Portfolio Tips

- **Show Live Data:** Demo with real market data streaming
- **Highlight Performance:** "Handles 100+ WebSocket connections, processes 1000+ ticks/second"
- **Explain Architecture:** System diagram showing data flow, caching strategy
- **Compare to Industry:** "Built similar to Robinhood/E*TRADE architecture"
- **Show Metrics:** P&L calculations, risk metrics, performance charts
- **Technical Depth:** Explain candlestick rendering, indicator algorithms

---

This project is part of the [Coding Projects Library](https://github.com/Ryanditko/biblioteca-de-projetos) and is licensed under the MIT License.
