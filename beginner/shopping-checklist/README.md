# Shopping Checklist - Smart Grocery Manager

<p align="center">
  <img src="https://images.unsplash.com/photo-1542838132-92c53300491e?w=600&q=80" alt="Shopping List" width="600">
</p>

**Level:** Beginner | **Time:** 3-5 hours | **Prerequisites:** HTML, CSS, JavaScript basics

## Description

Build a comprehensive shopping checklist application with categories, quantities, prices, budget tracking, and smart suggestions. Learn CRUD operations, localStorage persistence, filtering, and creating an intuitive user interface for everyday tasks.

## Core Features

### Must Have
- Add/edit/delete shopping items
- Mark items as purchased with checkbox
- Category organization (Produce, Dairy, Meat, etc.)
- Quantity and unit tracking
- Search and filter items
- Sort by category, name, or priority
- Data persistence with localStorage
- Clear completed items
- Total item count display

### Nice to Have
- Budget tracker with price per item
- Shopping history and frequent items
- Smart suggestions based on history
- Share list via link or QR code
- Print-friendly view
- Dark mode toggle
- Barcode scanner integration
- Multi-list support (different stores)
- Collaborative lists (multiple users)
- Recipe-to-list converter

## Learning Objectives

- **CRUD Operations**: Create, Read, Update, Delete functionality
- **localStorage**: Persist data in browser storage
- **Array Methods**: Filter, map, reduce, sort operations
- **Event Delegation**: Efficient event handling
- **State Management**: Track and update application state
- **UI/UX Design**: Create intuitive interfaces

## Implementation Guide

### Step 1: HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Shopping Checklist</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>🛒 Shopping Checklist</h1>
            <div class="stats">
                <span id="totalItems">0 items</span>
                <span id="totalPrice">$0.00</span>
            </div>
        </header>

        <div class="add-item-section">
            <input type="text" id="itemName" placeholder="Item name" required>
            <input type="number" id="itemQuantity" placeholder="Qty" value="1" min="1">
            <select id="itemCategory">
                <option value="produce">🥬 Produce</option>
                <option value="dairy">🥛 Dairy</option>
                <option value="meat">🥩 Meat</option>
                <option value="bakery">🍞 Bakery</option>
                <option value="beverages">☕ Beverages</option>
                <option value="snacks">🍿 Snacks</option>
                <option value="frozen">❄️ Frozen</option>
                <option value="household">🧹 Household</option>
                <option value="personal">🧴 Personal Care</option>
                <option value="other">📦 Other</option>
            </select>
            <input type="number" id="itemPrice" placeholder="Price ($)" step="0.01" min="0">
            <button id="addBtn" class="btn-add">Add Item</button>
        </div>

        <div class="controls">
            <input type="text" id="searchInput" placeholder="🔍 Search items...">
            <select id="filterCategory">
                <option value="all">All Categories</option>
                <option value="produce">Produce</option>
                <option value="dairy">Dairy</option>
                <option value="meat">Meat</option>
                <option value="bakery">Bakery</option>
                <option value="beverages">Beverages</option>
                <option value="snacks">Snacks</option>
                <option value="frozen">Frozen</option>
                <option value="household">Household</option>
                <option value="personal">Personal Care</option>
                <option value="other">Other</option>
            </select>
            <select id="sortBy">
                <option value="name">Sort by Name</option>
                <option value="category">Sort by Category</option>
                <option value="price">Sort by Price</option>
            </select>
            <button id="clearCompleted" class="btn-clear">Clear Completed</button>
        </div>

        <div id="shoppingList" class="shopping-list"></div>

        <div class="summary">
            <h3>Shopping Summary</h3>
            <div class="summary-grid">
                <div class="summary-item">
                    <span>Total Items:</span>
                    <strong id="summaryTotal">0</strong>
                </div>
                <div class="summary-item">
                    <span>Completed:</span>
                    <strong id="summaryCompleted">0</strong>
                </div>
                <div class="summary-item">
                    <span>Remaining:</span>
                    <strong id="summaryRemaining">0</strong>
                </div>
                <div class="summary-item">
                    <span>Total Cost:</span>
                    <strong id="summaryCost">$0.00</strong>
                </div>
            </div>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
```

### Step 2: CSS Styling

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}

.container {
    max-width: 1000px;
    margin: 0 auto;
    background: white;
    border-radius: 20px;
    padding: 30px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}

header {
    text-align: center;
    margin-bottom: 30px;
    padding-bottom: 20px;
    border-bottom: 2px solid #e0e0e0;
}

header h1 {
    font-size: 2.5rem;
    color: #333;
    margin-bottom: 10px;
}

.stats {
    display: flex;
    justify-content: center;
    gap: 30px;
    font-size: 1.1rem;
    color: #666;
}

.add-item-section {
    display: grid;
    grid-template-columns: 2fr 0.5fr 1.5fr 0.8fr 1fr;
    gap: 10px;
    margin-bottom: 20px;
}

.add-item-section input,
.add-item-section select,
.add-item-section button {
    padding: 12px;
    border: 2px solid #e0e0e0;
    border-radius: 8px;
    font-size: 14px;
}

.add-item-section input:focus,
.add-item-section select:focus {
    outline: none;
    border-color: #667eea;
}

.btn-add {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    border: none;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.2s;
}

.btn-add:hover {
    transform: translateY(-2px);
}

.controls {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr 1fr;
    gap: 10px;
    margin-bottom: 20px;
}

.controls input,
.controls select,
.controls button {
    padding: 10px;
    border: 2px solid #e0e0e0;
    border-radius: 8px;
    font-size: 14px;
}

.btn-clear {
    background: #e74c3c;
    color: white;
    border: none;
    font-weight: 600;
    cursor: pointer;
}

.btn-clear:hover {
    background: #c0392b;
}

.shopping-list {
    margin-bottom: 30px;
}

.category-section {
    margin-bottom: 20px;
}

.category-header {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    padding: 12px 15px;
    border-radius: 8px;
    font-weight: 600;
    margin-bottom: 10px;
}

.item {
    display: grid;
    grid-template-columns: 40px 2fr 0.5fr 1fr 80px 40px;
    align-items: center;
    gap: 15px;
    padding: 15px;
    background: #f8f9fa;
    border-radius: 8px;
    margin-bottom: 8px;
    transition: all 0.3s;
}

.item:hover {
    background: #e9ecef;
    transform: translateX(5px);
}

.item.completed {
    opacity: 0.6;
}

.item.completed .item-name {
    text-decoration: line-through;
}

.item-checkbox {
    width: 24px;
    height: 24px;
    cursor: pointer;
}

.item-name {
    font-size: 1.1rem;
    font-weight: 500;
    color: #333;
}

.item-quantity {
    color: #666;
    font-size: 0.95rem;
}

.item-category {
    color: #764ba2;
    font-size: 0.9rem;
}

.item-price {
    font-weight: 600;
    color: #667eea;
}

.item-delete {
    background: #e74c3c;
    color: white;
    border: none;
    padding: 8px;
    border-radius: 5px;
    cursor: pointer;
    transition: background 0.3s;
}

.item-delete:hover {
    background: #c0392b;
}

.summary {
    background: #f8f9fa;
    padding: 20px;
    border-radius: 15px;
}

.summary h3 {
    margin-bottom: 15px;
    color: #333;
}

.summary-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 15px;
}

.summary-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 15px;
    background: white;
    border-radius: 10px;
}

.summary-item span {
    color: #666;
    font-size: 0.9rem;
    margin-bottom: 5px;
}

.summary-item strong {
    color: #667eea;
    font-size: 1.5rem;
}

@media (max-width: 768px) {
    .add-item-section {
        grid-template-columns: 1fr;
    }
    
    .controls {
        grid-template-columns: 1fr;
    }
    
    .item {
        grid-template-columns: 30px 1fr 60px;
        gap: 10px;
    }
    
    .item-quantity,
    .item-category {
        display: none;
    }
    
    .summary-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}
```

### Step 3: JavaScript Implementation

```javascript
class ShoppingList {
    constructor() {
        this.items = JSON.parse(localStorage.getItem('shoppingItems')) || [];
        this.initializeElements();
        this.attachEventListeners();
        this.render();
    }

    initializeElements() {
        this.itemName = document.getElementById('itemName');
        this.itemQuantity = document.getElementById('itemQuantity');
        this.itemCategory = document.getElementById('itemCategory');
        this.itemPrice = document.getElementById('itemPrice');
        this.searchInput = document.getElementById('searchInput');
        this.filterCategory = document.getElementById('filterCategory');
        this.sortBy = document.getElementById('sortBy');
        this.shoppingList = document.getElementById('shoppingList');
    }

    attachEventListeners() {
        document.getElementById('addBtn').addEventListener('click', () => this.addItem());
        this.itemName.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') this.addItem();
        });
        this.searchInput.addEventListener('input', () => this.render());
        this.filterCategory.addEventListener('change', () => this.render());
        this.sortBy.addEventListener('change', () => this.render());
        document.getElementById('clearCompleted').addEventListener('click', () => this.clearCompleted());
    }

    addItem() {
        const name = this.itemName.value.trim();
        if (!name) {
            alert('Please enter an item name');
            return;
        }

        const item = {
            id: Date.now().toString(),
            name,
            quantity: parseInt(this.itemQuantity.value) || 1,
            category: this.itemCategory.value,
            price: parseFloat(this.itemPrice.value) || 0,
            completed: false,
            timestamp: new Date().toISOString()
        };

        this.items.push(item);
        this.save();
        this.render();
        this.clearInputs();
    }

    clearInputs() {
        this.itemName.value = '';
        this.itemQuantity.value = '1';
        this.itemPrice.value = '';
        this.itemName.focus();
    }

    toggleItem(id) {
        const item = this.items.find(i => i.id === id);
        if (item) {
            item.completed = !item.completed;
            this.save();
            this.render();
        }
    }

    deleteItem(id) {
        if (confirm('Delete this item?')) {
            this.items = this.items.filter(i => i.id !== id);
            this.save();
            this.render();
        }
    }

    clearCompleted() {
        const completedCount = this.items.filter(i => i.completed).length;
        if (completedCount === 0) {
            alert('No completed items to clear');
            return;
        }
        
        if (confirm(`Clear ${completedCount} completed item(s)?`)) {
            this.items = this.items.filter(i => !i.completed);
            this.save();
            this.render();
        }
    }

    getFilteredItems() {
        let filtered = [...this.items];

        // Search filter
        const search = this.searchInput.value.toLowerCase();
        if (search) {
            filtered = filtered.filter(item => 
                item.name.toLowerCase().includes(search)
            );
        }

        // Category filter
        const category = this.filterCategory.value;
        if (category !== 'all') {
            filtered = filtered.filter(item => item.category === category);
        }

        // Sort
        const sortBy = this.sortBy.value;
        filtered.sort((a, b) => {
            if (sortBy === 'name') {
                return a.name.localeCompare(b.name);
            } else if (sortBy === 'category') {
                return a.category.localeCompare(b.category);
            } else if (sortBy === 'price') {
                return b.price - a.price;
            }
            return 0;
        });

        return filtered;
    }

    render() {
        const filtered = this.getFilteredItems();
        
        // Group by category
        const grouped = {};
        filtered.forEach(item => {
            if (!grouped[item.category]) {
                grouped[item.category] = [];
            }
            grouped[item.category].push(item);
        });

        // Render items
        this.shoppingList.innerHTML = Object.entries(grouped).map(([category, items]) => `
            <div class="category-section">
                <div class="category-header">${this.getCategoryIcon(category)} ${this.getCategoryName(category)}</div>
                ${items.map(item => this.renderItem(item)).join('')}
            </div>
        `).join('');

        // Update summary
        this.updateSummary();
        this.updateStats();
    }

    renderItem(item) {
        return `
            <div class="item ${item.completed ? 'completed' : ''}">
                <input 
                    type="checkbox" 
                    class="item-checkbox"
                    ${item.completed ? 'checked' : ''}
                    onchange="list.toggleItem('${item.id}')"
                >
                <div class="item-name">${item.name}</div>
                <div class="item-quantity">×${item.quantity}</div>
                <div class="item-category">${this.getCategoryName(item.category)}</div>
                <div class="item-price">$${(item.price * item.quantity).toFixed(2)}</div>
                <button class="item-delete" onclick="list.deleteItem('${item.id}')">🗑️</button>
            </div>
        `;
    }

    getCategoryIcon(category) {
        const icons = {
            produce: '🥬', dairy: '🥛', meat: '🥩', bakery: '🍞',
            beverages: '☕', snacks: '🍿', frozen: '❄️', household: '🧹',
            personal: '🧴', other: '📦'
        };
        return icons[category] || '📦';
    }

    getCategoryName(category) {
        return category.charAt(0).toUpperCase() + category.slice(1);
    }

    updateSummary() {
        const total = this.items.length;
        const completed = this.items.filter(i => i.completed).length;
        const remaining = total - completed;
        const totalCost = this.items.reduce((sum, item) => 
            sum + (item.price * item.quantity), 0
        );

        document.getElementById('summaryTotal').textContent = total;
        document.getElementById('summaryCompleted').textContent = completed;
        document.getElementById('summaryRemaining').textContent = remaining;
        document.getElementById('summaryCost').textContent = `$${totalCost.toFixed(2)}`;
    }

    updateStats() {
        const total = this.items.length;
        const totalPrice = this.items.reduce((sum, item) => 
            sum + (item.price * item.quantity), 0
        );

        document.getElementById('totalItems').textContent = `${total} item${total !== 1 ? 's' : ''}`;
        document.getElementById('totalPrice').textContent = `$${totalPrice.toFixed(2)}`;
    }

    save() {
        localStorage.setItem('shoppingItems', JSON.stringify(this.items));
    }
}

// Initialize the shopping list
const list = new ShoppingList();
```

## Technology Stack
- HTML5
- CSS3 (Grid, Flexbox, Gradients)
- Vanilla JavaScript
- localStorage API

## Testing Checklist
- [ ] Can add items with all fields
- [ ] Items persist after page reload
- [ ] Checkbox toggles completed state
- [ ] Delete removes items
- [ ] Search filters correctly
- [ ] Category filter works
- [ ] Sort by name/category/price functions
- [ ] Clear completed removes only completed items
- [ ] Total cost calculates accurately
- [ ] Mobile responsive design

## Additional Challenges

### Easy
- Add item priority levels (high/medium/low)
- Include item images or icons
- Add undo deleted item feature

### Medium
- Implement drag-and-drop reordering
- Add shopping list templates
- Create print-friendly view
- Build barcode scanner with API

### Hard
- Implement multi-user collaborative lists
- Add recipe-to-shopping-list converter
- Create AI-powered smart suggestions
- Build cross-device sync with backend

## Resources
- [localStorage Guide](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [Array Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [CSS Grid Layout](https://css-tricks.com/snippets/css/complete-guide-grid/)

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
