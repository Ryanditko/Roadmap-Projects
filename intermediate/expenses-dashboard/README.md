# Expenses Dashboard

<p align="center">
  <img src="https://images.unsplash.com/photo-1554224155-6726b3ff858f?w=600" alt="Expenses Dashboard">
</p>

## Overview

**Difficulty:** Intermediate  
**Estimated Time:** 10–12 hours  
**Prerequisites:** HTML, CSS, JavaScript, Chart.js, localStorage or backend basics  

An **Expenses Dashboard** is a personal finance tracking application that allows users to:

- Monitor spending
- Categorize expenses
- Visualize financial data
- Generate reports

This project focuses on data visualization, financial calculations, state management, and UI/UX for data-heavy interfaces.

---

## Features

### Must Have

- Add expenses (amount, category, description, date)
- Display expenses in a list/table
- Category-based organization (food, transport, utilities, etc.)
- Filter by:
  - Date range
  - Category
  - Amount
- Total expenses calculation
- Charts showing spending by category
- Edit and delete expenses
- Responsive mobile design

### Nice to Have

- Monthly / yearly comparison charts
- Budget limits per category with alerts
- Recurring expense templates
- Search functionality
- Export to CSV / PDF
- Receipt image upload
- Multi-currency support
- Expense sharing / splitting
- Dark mode
- Offline support with sync

---

## Learning Objectives

By building this project, you will learn:

- Data visualization with Chart.js (or D3.js)
- Date manipulation using native Date API or date-fns
- CRUD operations for financial data
- Filtering and sorting large datasets efficiently
- Data persistence:
  - localStorage (frontend-only)
  - or database integration
- Financial calculations:
  - Totals
  - Averages
  - Percentages

---

## Project Concepts

This project simulates a real-world financial system and covers:

- State management for dynamic UI updates
- Data normalization for categories
- Performance when handling large lists
- Component-based UI thinking
- Dashboard design patterns

---

## Suggested Data Structure

```js
{
  id: string,
  amount: number,
  category: string,
  description: string,
  date: string,
  createdAt: string,
  updatedAt: string
}


Plan how to store and organize expense data:

```javascript
// Expense object structure
const expense = {
    id: 'exp_1234567890',
    amount: 45.50,
    category: 'food',
    description: 'Grocery shopping',
    date: '2024-02-14T10:30:00',
    paymentMethod: 'credit_card',
    tags: ['groceries', 'weekly'],
    receipt: 'base64_image_or_url',
    createdAt: '2024-02-14T10:30:00',
    updatedAt: '2024-02-14T10:30:00'
};

// Categories structure
const categories = {
    food: { name: 'Food & Dining', icon: '🍔', color: '#FF6384', budget: 500 },
    transport: { name: 'Transportation', icon: '🚗', color: '#36A2EB', budget: 200 },
    utilities: { name: 'Utilities', icon: '💡', color: '#FFCE56', budget: 150 },
    entertainment: { name: 'Entertainment', icon: '🎬', color: '#4BC0C0', budget: 100 },
    healthcare: { name: 'Healthcare', icon: '🏥', color: '#9966FF', budget: 200 },
    shopping: { name: 'Shopping', icon: '🛍️', color: '#FF9F40', budget: 300 },
    other: { name: 'Other', icon: '📦', color: '#C9CBCF', budget: 100 }
};
```

### Step 2: Create the Dashboard Layout

Build the HTML structure with responsive grid:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Expenses Dashboard</title>
    <link rel="stylesheet" href="styles.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <div class="app">
        <!-- Sidebar -->
        <aside class="sidebar">
            <h1>Expenses</h1>
            <nav>
                <a href="#" class="nav-link active" data-view="dashboard">Dashboard</a>
                <a href="#" class="nav-link" data-view="expenses">All Expenses</a>
                <a href="#" class="nav-link" data-view="categories">Categories</a>
                <a href="#" class="nav-link" data-view="reports">Reports</a>
            </nav>
        </aside>
        
        <!-- Main Content -->
        <main class="main-content">
            <!-- Header -->
            <header class="header">
                <div class="header-left">
                    <h2 id="pageTitle">Dashboard</h2>
                </div>
                <div class="header-right">
                    <button class="btn-primary" id="addExpenseBtn">+ Add Expense</button>
                </div>
            </header>
            
            <!-- Dashboard View -->
            <div id="dashboardView" class="view active">
                <!-- Summary Cards -->
                <div class="summary-cards">
                    <div class="card">
                        <div class="card-icon">💰</div>
                        <div class="card-content">
                            <p class="card-label">Total Expenses</p>
                            <h3 class="card-value" id="totalExpenses">$0.00</h3>
                            <small class="card-change">This month</small>
                        </div>
                    </div>
                    
                    <div class="card">
                        <div class="card-icon">📊</div>
                        <div class="card-content">
                            <p class="card-label">Average per Day</p>
                            <h3 class="card-value" id="avgPerDay">$0.00</h3>
                            <small class="card-change">Last 30 days</small>
                        </div>
                    </div>
                    
                    <div class="card">
                        <div class="card-icon">🎯</div>
                        <div class="card-content">
                            <p class="card-label">Budget Status</p>
                            <h3 class="card-value" id="budgetStatus">On track</h3>
                            <small class="card-change">75% used</small>
                        </div>
                    </div>
                    
                    <div class="card">
                        <div class="card-icon">📈</div>
                        <div class="card-content">
                            <p class="card-label">Top Category</p>
                            <h3 class="card-value" id="topCategory">Food</h3>
                            <small class="card-change">$450 this month</small>
                        </div>
                    </div>
                </div>
                
                <!-- Charts -->
                <div class="charts-grid">
                    <div class="chart-container">
                        <h3>Spending by Category</h3>
                        <canvas id="categoryChart"></canvas>
                    </div>
                    
                    <div class="chart-container">
                        <h3>Monthly Trend</h3>
                        <canvas id="trendChart"></canvas>
                    </div>
                </div>
                
                <!-- Recent Expenses -->
                <div class="recent-expenses">
                    <h3>Recent Expenses</h3>
                    <div id="recentExpensesList"></div>
                </div>
            </div>
            
            <!-- Expenses View -->
            <div id="expensesView" class="view">
                <!-- Filters -->
                <div class="filters">
                    <input type="text" id="searchExpense" placeholder="Search expenses...">
                    <select id="filterCategory">
                        <option value="">All Categories</option>
                    </select>
                    <input type="date" id="filterDateFrom">
                    <input type="date" id="filterDateTo">
                    <button id="clearFilters">Clear</button>
                </div>
                
                <!-- Expenses Table -->
                <div class="expenses-table">
                    <table id="expensesTable">
                        <thead>
                            <tr>
                                <th>Date</th>
                                <th>Description</th>
                                <th>Category</th>
                                <th>Amount</th>
                                <th>Actions</th>
                            </tr>
                        </thead>
                        <tbody id="expensesTableBody"></tbody>
                    </table>
                </div>
            </div>
        </main>
    </div>
    
    <!-- Add/Edit Expense Modal -->
    <div id="expenseModal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h2 id="modalTitle">Add Expense</h2>
                <button class="close-btn" id="closeModal">&times;</button>
            </div>
            <form id="expenseForm">
                <div class="form-group">
                    <label>Amount</label>
                    <input type="number" id="expenseAmount" step="0.01" required>
                </div>
                
                <div class="form-group">
                    <label>Category</label>
                    <select id="expenseCategory" required>
                        <option value="">Select category</option>
                    </select>
                </div>
                
                <div class="form-group">
                    <label>Description</label>
                    <input type="text" id="expenseDescription" required>
                </div>
                
                <div class="form-group">
                    <label>Date</label>
                    <input type="date" id="expenseDate" required>
                </div>
                
                <div class="form-group">
                    <label>Payment Method</label>
                    <select id="paymentMethod">
                        <option value="cash">Cash</option>
                        <option value="credit_card">Credit Card</option>
                        <option value="debit_card">Debit Card</option>
                        <option value="bank_transfer">Bank Transfer</option>
                    </select>
                </div>
                
                <div class="form-actions">
                    <button type="button" class="btn-secondary" id="cancelBtn">Cancel</button>
                    <button type="submit" class="btn-primary">Save Expense</button>
                </div>
            </form>
        </div>
    </div>
    
    <script src="script.js"></script>
</body>
</html>
```

### Step 3: Implement Core JavaScript Logic

Create the expense management system:

```javascript
class ExpenseTracker {
    constructor() {
        this.expenses = this.loadExpenses();
        this.categories = {
            food: { name: 'Food & Dining', icon: '🍔', color: '#FF6384', budget: 500 },
            transport: { name: 'Transportation', icon: '🚗', color: '#36A2EB', budget: 200 },
            utilities: { name: 'Utilities', icon: '💡', color: '#FFCE56', budget: 150 },
            entertainment: { name: 'Entertainment', icon: '🎬', color: '#4BC0C0', budget: 100 },
            healthcare: { name: 'Healthcare', icon: '🏥', color: '#9966FF', budget: 200 },
            shopping: { name: 'Shopping', icon: '🛍️', color: '#FF9F40', budget: 300 },
            other: { name: 'Other', icon: '📦', color: '#C9CBCF', budget: 100 }
        };
        
        this.currentExpense = null;
        this.charts = {};
        
        this.init();
    }
    
    init() {
        this.setupEventListeners();
        this.populateCategorySelects();
        this.switchView('dashboard');
        this.updateDashboard();
    }
    
    setupEventListeners() {
        // Navigation
        document.querySelectorAll('.nav-link').forEach(link => {
            link.addEventListener('click', (e) => {
                e.preventDefault();
                const view = e.target.dataset.view;
                this.switchView(view);
            });
        });
        
        // Add expense button
        document.getElementById('addExpenseBtn').addEventListener('click', () => {
            this.openExpenseModal();
        });
        
        // Modal close
        document.getElementById('closeModal').addEventListener('click', () => {
            this.closeExpenseModal();
        });
        
        document.getElementById('cancelBtn').addEventListener('click', () => {
            this.closeExpenseModal();
        });
        
        // Form submit
        document.getElementById('expenseForm').addEventListener('submit', (e) => {
            e.preventDefault();
            this.saveExpense();
        });
        
        // Filters
        document.getElementById('searchExpense').addEventListener('input', () => {
            this.filterExpenses();
        });
        
        document.getElementById('filterCategory').addEventListener('change', () => {
            this.filterExpenses();
        });
        
        document.getElementById('filterDateFrom').addEventListener('change', () => {
            this.filterExpenses();
        });
        
        document.getElementById('filterDateTo').addEventListener('change', () => {
            this.filterExpenses();
        });
        
        document.getElementById('clearFilters').addEventListener('click', () => {
            this.clearFilters();
        });
    }
    
    switchView(viewName) {
        // Update navigation
        document.querySelectorAll('.nav-link').forEach(link => {
            link.classList.toggle('active', link.dataset.view === viewName);
        });
        
        // Update views
        document.querySelectorAll('.view').forEach(view => {
            view.classList.remove('active');
        });
        
        document.getElementById(`${viewName}View`).classList.add('active');
        
        // Update page title
        const titles = {
            dashboard: 'Dashboard',
            expenses: 'All Expenses',
            categories: 'Categories',
            reports: 'Reports'
        };
        
        document.getElementById('pageTitle').textContent = titles[viewName];
        
        // Load view-specific data
        if (viewName === 'dashboard') {
            this.updateDashboard();
        } else if (viewName === 'expenses') {
            this.renderExpensesTable();
        }
    }
    
    openExpenseModal(expense = null) {
        this.currentExpense = expense;
        const modal = document.getElementById('expenseModal');
        
        if (expense) {
            document.getElementById('modalTitle').textContent = 'Edit Expense';
            document.getElementById('expenseAmount').value = expense.amount;
            document.getElementById('expenseCategory').value = expense.category;
            document.getElementById('expenseDescription').value = expense.description;
            document.getElementById('expenseDate').value = expense.date.split('T')[0];
            document.getElementById('paymentMethod').value = expense.paymentMethod || 'cash';
        } else {
            document.getElementById('modalTitle').textContent = 'Add Expense';
            document.getElementById('expenseForm').reset();
            document.getElementById('expenseDate').value = new Date().toISOString().split('T')[0];
        }
        
        modal.classList.add('active');
    }
    
    closeExpenseModal() {
        document.getElementById('expenseModal').classList.remove('active');
        this.currentExpense = null;
    }
    
    saveExpense() {
        const amount = parseFloat(document.getElementById('expenseAmount').value);
        const category = document.getElementById('expenseCategory').value;
        const description = document.getElementById('expenseDescription').value;
        const date = new Date(document.getElementById('expenseDate').value).toISOString();
        const paymentMethod = document.getElementById('paymentMethod').value;
        
        if (this.currentExpense) {
            // Edit existing expense
            const index = this.expenses.findIndex(e => e.id === this.currentExpense.id);
            this.expenses[index] = {
                ...this.currentExpense,
                amount,
                category,
                description,
                date,
                paymentMethod,
                updatedAt: new Date().toISOString()
            };
        } else {
            // Add new expense
            const expense = {
                id: 'exp_' + Date.now(),
                amount,
                category,
                description,
                date,
                paymentMethod,
                createdAt: new Date().toISOString(),
                updatedAt: new Date().toISOString()
            };
            
            this.expenses.unshift(expense);
        }
        
        this.saveExpenses();
        this.closeExpenseModal();
        this.updateDashboard();
        this.renderExpensesTable();
    }
    
    deleteExpense(id) {
        if (confirm('Delete this expense?')) {
            this.expenses = this.expenses.filter(e => e.id !== id);
            this.saveExpenses();
            this.updateDashboard();
            this.renderExpensesTable();
        }
    }
    
    updateDashboard() {
        this.updateSummaryCards();
        this.updateCharts();
        this.renderRecentExpenses();
    }
    
    updateSummaryCards() {
        const now = new Date();
        const currentMonth = now.getMonth();
        const currentYear = now.getFullYear();
        
        // Filter expenses for current month
        const monthExpenses = this.expenses.filter(e => {
            const expenseDate = new Date(e.date);
            return expenseDate.getMonth() === currentMonth && 
                   expenseDate.getFullYear() === currentYear;
        });
        
        // Total expenses
        const total = monthExpenses.reduce((sum, e) => sum + e.amount, 0);
        document.getElementById('totalExpenses').textContent = '$' + total.toFixed(2);
        
        // Average per day
        const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate();
        const avgPerDay = total / daysInMonth;
        document.getElementById('avgPerDay').textContent = '$' + avgPerDay.toFixed(2);
        
        // Budget status
        const totalBudget = Object.values(this.categories).reduce((sum, c) => sum + c.budget, 0);
        const budgetPercent = (total / totalBudget * 100).toFixed(0);
        document.getElementById('budgetStatus').textContent = 
            budgetPercent > 100 ? 'Over budget' : 'On track';
        document.querySelector('#budgetStatus + small').textContent = `${budgetPercent}% used`;
        
        // Top category
        const categoryTotals = {};
        monthExpenses.forEach(e => {
            categoryTotals[e.category] = (categoryTotals[e.category] || 0) + e.amount;
        });
        
        const topCategory = Object.entries(categoryTotals)
            .sort((a, b) => b[1] - a[1])[0];
        
        if (topCategory) {
            document.getElementById('topCategory').textContent = 
                this.categories[topCategory[0]].name;
            document.querySelector('#topCategory + small').textContent = 
                `$${topCategory[1].toFixed(2)} this month`;
        }
    }
    
    updateCharts() {
        this.createCategoryChart();
        this.createTrendChart();
    }
    
    createCategoryChart() {
        const now = new Date();
        const currentMonth = now.getMonth();
        const currentYear = now.getFullYear();
        
        const monthExpenses = this.expenses.filter(e => {
            const expenseDate = new Date(e.date);
            return expenseDate.getMonth() === currentMonth && 
                   expenseDate.getFullYear() === currentYear;
        });
        
        const categoryData = {};
        monthExpenses.forEach(e => {
            categoryData[e.category] = (categoryData[e.category] || 0) + e.amount;
        });
        
        const labels = Object.keys(categoryData).map(key => this.categories[key].name);
        const data = Object.values(categoryData);
        const colors = Object.keys(categoryData).map(key => this.categories[key].color);
        
        if (this.charts.category) {
            this.charts.category.destroy();
        }
        
        const ctx = document.getElementById('categoryChart').getContext('2d');
        this.charts.category = new Chart(ctx, {
            type: 'doughnut',
            data: {
                labels: labels,
                datasets: [{
                    data: data,
                    backgroundColor: colors
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: {
                        position: 'bottom'
                    }
                }
            }
        });
    }
    
    createTrendChart() {
        const last6Months = [];
        const now = new Date();
        
        for (let i = 5; i >= 0; i--) {
            const date = new Date(now.getFullYear(), now.getMonth() - i, 1);
            last6Months.push({
                month: date.toLocaleString('default', { month: 'short' }),
                year: date.getFullYear(),
                monthIndex: date.getMonth()
            });
        }
        
        const monthlyTotals = last6Months.map(m => {
            const monthExpenses = this.expenses.filter(e => {
                const expenseDate = new Date(e.date);
                return expenseDate.getMonth() === m.monthIndex && 
                       expenseDate.getFullYear() === m.year;
            });
            
            return monthExpenses.reduce((sum, e) => sum + e.amount, 0);
        });
        
        if (this.charts.trend) {
            this.charts.trend.destroy();
        }
        
        const ctx = document.getElementById('trendChart').getContext('2d');
        this.charts.trend = new Chart(ctx, {
            type: 'line',
            data: {
                labels: last6Months.map(m => m.month),
                datasets: [{
                    label: 'Monthly Expenses',
                    data: monthlyTotals,
                    borderColor: '#667eea',
                    backgroundColor: 'rgba(102, 126, 234, 0.1)',
                    tension: 0.4,
                    fill: true
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: {
                        display: false
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        ticks: {
                            callback: function(value) {
                                return '$' + value;
                            }
                        }
                    }
                }
            }
        });
    }
    
    renderRecentExpenses() {
        const recent = this.expenses.slice(0, 5);
        const html = recent.map(expense => {
            const category = this.categories[expense.category];
            const date = new Date(expense.date).toLocaleDateString();
            
            return `
                <div class="expense-item">
                    <div class="expense-icon">${category.icon}</div>
                    <div class="expense-details">
                        <h4>${expense.description}</h4>
                        <small>${date} • ${category.name}</small>
                    </div>
                    <div class="expense-amount">$${expense.amount.toFixed(2)}</div>
                </div>
            `;
        }).join('');
        
        document.getElementById('recentExpensesList').innerHTML = 
            html || '<p class="empty-state">No expenses yet</p>';
    }
    
    renderExpensesTable() {
        const html = this.expenses.map(expense => {
            const category = this.categories[expense.category];
            const date = new Date(expense.date).toLocaleDateString();
            
            return `
                <tr>
                    <td>${date}</td>
                    <td>${expense.description}</td>
                    <td>
                        <span class="category-badge" style="background: ${category.color}20; color: ${category.color}">
                            ${category.icon} ${category.name}
                        </span>
                    </td>
                    <td><strong>$${expense.amount.toFixed(2)}</strong></td>
                    <td>
                        <button onclick="tracker.openExpenseModal(tracker.expenses.find(e => e.id === '${expense.id}'))">Edit</button>
                        <button onclick="tracker.deleteExpense('${expense.id}')">Delete</button>
                    </td>
                </tr>
            `;
        }).join('');
        
        document.getElementById('expensesTableBody').innerHTML = 
            html || '<tr><td colspan="5" class="empty-state">No expenses yet</td></tr>';
    }
    
    filterExpenses() {
        const search = document.getElementById('searchExpense').value.toLowerCase();
        const category = document.getElementById('filterCategory').value;
        const dateFrom = document.getElementById('filterDateFrom').value;
        const dateTo = document.getElementById('filterDateTo').value;
        
        let filtered = this.expenses;
        
        if (search) {
            filtered = filtered.filter(e => 
                e.description.toLowerCase().includes(search)
            );
        }
        
        if (category) {
            filtered = filtered.filter(e => e.category === category);
        }
        
        if (dateFrom) {
            filtered = filtered.filter(e => new Date(e.date) >= new Date(dateFrom));
        }
        
        if (dateTo) {
            filtered = filtered.filter(e => new Date(e.date) <= new Date(dateTo));
        }
        
        // Render filtered results (simplified)
        console.log('Filtered expenses:', filtered);
    }
    
    clearFilters() {
        document.getElementById('searchExpense').value = '';
        document.getElementById('filterCategory').value = '';
        document.getElementById('filterDateFrom').value = '';
        document.getElementById('filterDateTo').value = '';
        this.renderExpensesTable();
    }
    
    populateCategorySelects() {
        const html = Object.entries(this.categories).map(([key, cat]) => 
            `<option value="${key}">${cat.icon} ${cat.name}</option>`
        ).join('');
        
        document.getElementById('expenseCategory').innerHTML = 
            '<option value="">Select category</option>' + html;
        
        document.getElementById('filterCategory').innerHTML = 
            '<option value="">All Categories</option>' + html;
    }
    
    saveExpenses() {
        localStorage.setItem('expenses', JSON.stringify(this.expenses));
    }
    
    loadExpenses() {
        const saved = localStorage.getItem('expenses');
        return saved ? JSON.parse(saved) : [];
    }
}

// Initialize
const tracker = new ExpenseTracker();
```

### Step 4: Style the Dashboard

Create a modern, professional interface:

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    background: #f5f7fa;
    color: #333;
}

.app {
    display: grid;
    grid-template-columns: 250px 1fr;
    min-height: 100vh;
}

/* Sidebar */
.sidebar {
    background: white;
    padding: 30px 0;
    border-right: 1px solid #e0e0e0;
}

.sidebar h1 {
    padding: 0 30px;
    margin-bottom: 30px;
    font-size: 24px;
    color: #667eea;
}

.nav-link {
    display: block;
    padding: 15px 30px;
    color: #666;
    text-decoration: none;
    transition: all 0.3s;
    border-left: 3px solid transparent;
}

.nav-link:hover {
    background: #f5f7fa;
    color: #667eea;
}

.nav-link.active {
    background: #f5f7fa;
    color: #667eea;
    border-left-color: #667eea;
    font-weight: 600;
}

/* Main Content */
.main-content {
    padding: 30px;
}

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 30px;
}

.header h2 {
    font-size: 28px;
    color: #333;
}

.btn-primary {
    padding: 12px 24px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.2s;
}

.btn-primary:hover {
    transform: translateY(-2px);
}

/* Views */
.view {
    display: none;
}

.view.active {
    display: block;
}

/* Summary Cards */
.summary-cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
}

.card {
    background: white;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    display: flex;
    gap: 15px;
}

.card-icon {
    font-size: 40px;
}

.card-label {
    color: #666;
    font-size: 14px;
    margin-bottom: 8px;
}

.card-value {
    font-size: 24px;
    font-weight: bold;
    color: #333;
    margin-bottom: 4px;
}

.card-change {
    color: #999;
    font-size: 12px;
}

/* Charts */
.charts-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
}

.chart-container {
    background: white;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.chart-container h3 {
    margin-bottom: 20px;
    color: #333;
    font-size: 18px;
}

.chart-container canvas {
    max-height: 300px;
}

/* Recent Expenses */
.recent-expenses {
    background: white;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.recent-expenses h3 {
    margin-bottom: 20px;
    color: #333;
}

.expense-item {
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 15px;
    border-bottom: 1px solid #f0f0f0;
}

.expense-item:last-child {
    border-bottom: none;
}

.expense-icon {
    font-size: 32px;
}

.expense-details {
    flex: 1;
}

.expense-details h4 {
    font-size: 16px;
    color: #333;
    margin-bottom: 4px;
}

.expense-details small {
    color: #999;
    font-size: 13px;
}

.expense-amount {
    font-size: 18px;
    font-weight: bold;
    color: #667eea;
}

/* Filters */
.filters {
    background: white;
    padding: 20px;
    border-radius: 12px;
    margin-bottom: 20px;
    display: flex;
    gap: 15px;
    flex-wrap: wrap;
}

.filters input,
.filters select,
.filters button {
    padding: 10px 15px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    font-size: 14px;
}

/* Table */
.expenses-table {
    background: white;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

table {
    width: 100%;
    border-collapse: collapse;
}

thead {
    background: #f8f9fa;
}

th {
    text-align: left;
    padding: 15px 20px;
    font-weight: 600;
    color: #666;
    font-size: 14px;
}

td {
    padding: 15px 20px;
    border-top: 1px solid #f0f0f0;
}

.category-badge {
    display: inline-block;
    padding: 4px 12px;
    border-radius: 20px;
    font-size: 13px;
    font-weight: 500;
}

/* Modal */
.modal {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0,0,0,0.5);
    align-items: center;
    justify-content: center;
    z-index: 1000;
}

.modal.active {
    display: flex;
}

.modal-content {
    background: white;
    border-radius: 12px;
    width: 90%;
    max-width: 500px;
    max-height: 90vh;
    overflow-y: auto;
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 25px;
    border-bottom: 1px solid #e0e0e0;
}

.close-btn {
    font-size: 28px;
    border: none;
    background: none;
    cursor: pointer;
    color: #999;
}

.form-group {
    padding: 0 25px;
    margin: 20px 0;
}

.form-group label {
    display: block;
    margin-bottom: 8px;
    font-weight: 600;
    color: #333;
}

.form-group input,
.form-group select {
    width: 100%;
    padding: 12px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    font-size: 14px;
}

.form-actions {
    display: flex;
    gap: 10px;
    padding: 25px;
    border-top: 1px solid #e0e0e0;
}

.btn-secondary {
    flex: 1;
    padding: 12px;
    background: #f5f7fa;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
}

.form-actions .btn-primary {
    flex: 1;
}

/* Responsive */
@media (max-width: 768px) {
    .app {
        grid-template-columns: 1fr;
    }
    
    .sidebar {
        display: none;
    }
    
    .summary-cards {
        grid-template-columns: 1fr;
    }
    
    .charts-grid {
        grid-template-columns: 1fr;
    }
}
```

## Technology Suggestions

### Frontend

- **HTML5** with semantic structure
- **CSS3** with Grid and Flexbox
- **Vanilla JavaScript** or **React** for complex state
- **Chart.js** for visualizations
- **date-fns** or **Day.js** for date manipulation

### Backend (Optional)

- **Node.js + Express** with MongoDB
- **Python + Flask** with PostgreSQL
- **Firebase** for real-time sync
- **Supabase** for backend-as-a-service

### Mobile

- **React Native** with Chart.js alternative
- **Flutter** with fl_chart package

## Testing Checklist

- [ ] Expenses can be added, edited, deleted
- [ ] Charts update in real-time when data changes
- [ ] Filters work correctly (category, date range, search)
- [ ] Total calculations are accurate
- [ ] Budget warnings appear when limits exceeded
- [ ] Data persists across sessions
- [ ] Responsive design works on mobile
- [ ] Date handling works across time zones
- [ ] Export functionality generates correct files

## Additional Challenges

### Easy Extensions

- Add expense templates for recurring items
- Implement dark mode
- Create print-friendly views
- Add keyboard shortcuts

### Medium Extensions

- Build receipt scanning with OCR
- Add budget goals with progress bars
- Create expense sharing/splitting features
- Implement multi-currency support
- Add recurring expense automation
- Build expense categories customization

### Hard Extensions

- Create bank account integration via API
- Add machine learning for expense categorization
- Build collaborative budgets for households
- Implement investment tracking
- Create financial forecasting
- Add tax calculation and reporting

## Common Pitfalls

1. **Date Handling:** Be careful with time zones and date parsing. Use ISO strings consistently.

2. **Decimal Precision:** Use `toFixed(2)` for currency display but store as numbers to avoid rounding errors.

3. **Chart Performance:** Destroy old Chart.js instances before creating new ones to prevent memory leaks.

4. **Data Validation:** Always validate amounts are positive numbers and dates are valid.

## Resources

### Documentation

- [Chart.js Documentation](https://www.chartjs.org/docs/)
- [localStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [Date API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)

### Tutorials

- [Building Financial Dashboards](https://www.youtube.com/results?search_query=expense+tracker+javascript)
- [Chart.js Tutorial](https://www.chartjs.org/docs/latest/getting-started/)

### Design Inspiration

- [Mint](https://mint.intuit.com/)
- [YNAB](https://www.youneedabudget.com/)
- [Splitwise](https://www.splitwise.com/)

## Submission Checklist

- [ ] Full CRUD operations for expenses
- [ ] Interactive charts with Chart.js
- [ ] Category-based organization
- [ ] Filtering and search functionality
- [ ] Summary statistics and calculations
- [ ] Responsive mobile design
- [ ] Data persistence
- [ ] Clean, professional UI
- [ ] Code well-documented

## Portfolio Tips

- **Show Data Visualization:** Screenshots of charts with real data
- **Demonstrate Insights:** Explain how your dashboard helps users understand spending patterns
- **Highlight Performance:** Mention optimizations for handling thousands of expenses
- **Add Real Features:** Implement at least one advanced feature (OCR, budgets, recurring)
- **Compare to Apps:** Explain how your implementation differs from Mint or YNAB

---

This project is part of the [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library) and is licensed under the MIT License.
