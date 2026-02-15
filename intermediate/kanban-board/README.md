# Kanban Board - Drag & Drop Task Management

**Level:** Intermediate | **Time:** 1-2 weeks

## Description
Visual project management tool with drag-and-drop columns, task cards, priority levels, and filtering. Learn DOM manipulation, drag-and-drop API, and state management.

## Core Features
- Drag-and-drop tasks between columns
- Create/edit/delete tasks
- Priority levels (Low/Medium/High)
- Task categories/tags
- Search and filter
- Progress tracking
- localStorage persistence

## Implementation Guide

```javascript
class KanbanBoard {
    constructor() {
        this.tasks = JSON.parse(localStorage.getItem('kanban-tasks')) || [];
        this.init();
    }
    
    init() {
        this.render();
        this.setupDragAndDrop();
        this.setupEventListeners();
    }
    
    setupDragAndDrop() {
        const columns = document.querySelectorAll('.column');
        
        columns.forEach(column => {
            column.addEventListener('dragover', (e) => {
                e.preventDefault();
                column.classList.add('drag-over');
            });
            
            column.addEventListener('dragleave', () => {
                column.classList.remove('drag-over');
            });
            
            column.addEventListener('drop', (e) => {
                e.preventDefault();
                column.classList.remove('drag-over');
                
                const taskId = e.dataTransfer.getData('text/plain');
                const newStatus = column.dataset.status;
                
                this.updateTaskStatus(taskId, newStatus);
            });
        });
    }
    
    createTask(title, description, priority) {
        const task = {
            id: Date.now().toString(),
            title,
            description,
            priority,
            status: 'todo',
            createdAt: new Date().toISOString()
        };
        
        this.tasks.push(task);
        this.save();
        this.render();
    }
    
    updateTaskStatus(taskId, newStatus) {
        const task = this.tasks.find(t => t.id === taskId);
        if (task) {
            task.status = newStatus;
            this.save();
            this.render();
        }
    }
    
    deleteTask(taskId) {
        this.tasks = this.tasks.filter(t => t.id !== taskId);
        this.save();
        this.render();
    }
    
    renderTask(task) {
        return `
            <div class="task" draggable="true" data-id="${task.id}">
                <div class="task-header">
                    <h4>${task.title}</h4>
                    <span class="priority ${task.priority}">${task.priority}</span>
                </div>
                <p>${task.description}</p>
                <button onclick="kanban.deleteTask('${task.id}')">Delete</button>
            </div>
        `;
    }
    
    render() {
        ['todo', 'inprogress', 'done'].forEach(status => {
            const column = document.querySelector(`[data-status="${status}"]`);
            const tasks = this.tasks.filter(t => t.status === status);
            column.querySelector('.tasks').innerHTML = tasks.map(t => this.renderTask(t)).join('');
            
            // Setup drag events for tasks
            column.querySelectorAll('.task').forEach(taskEl => {
                taskEl.addEventListener('dragstart', (e) => {
                    e.dataTransfer.setData('text/plain', taskEl.dataset.id);
                    taskEl.classList.add('dragging');
                });
                
                taskEl.addEventListener('dragend', (e) => {
                    taskEl.classList.remove('dragging');
                });
            });
        });
    }
    
    save() {
        localStorage.setItem('kanban-tasks', JSON.stringify(this.tasks));
    }
}

const kanban = new KanbanBoard();
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Kanban Board</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; background: #f5f5f5; padding: 20px; }
        .board { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
        .column {
            background: white;
            border-radius: 10px;
            padding: 20px;
            min-height: 500px;
        }
        .column.drag-over { background: #e3f2fd; }
        .column-header { font-size: 20px; font-weight: 600; margin-bottom: 20px; }
        .tasks { display: flex; flex-direction: column; gap: 15px; }
        .task {
            background: #fff;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            padding: 15px;
            cursor: move;
        }
        .task.dragging { opacity: 0.5; }
        .priority { padding: 4px 8px; border-radius: 4px; font-size: 12px; }
        .priority.high { background: #ffebee; color: #c62828; }
        .priority.medium { background: #fff3e0; color: #e65100; }
        .priority.low { background: #e8f5e9; color: #2e7d32; }
    </style>
</head>
<body>
    <h1>Kanban Board</h1>
    <div class="board">
        <div class="column" data-status="todo">
            <h3 class="column-header">To Do</h3>
            <div class="tasks"></div>
        </div>
        <div class="column" data-status="inprogress">
            <h3 class="column-header">In Progress</h3>
            <div class="tasks"></div>
        </div>
        <div class="column" data-status="done">
            <h3 class="column-header">Done</h3>
            <div class="tasks"></div>
        </div>
    </div>
</body>
</html>
```

## Technology Stack
- HTML5 Drag and Drop API
- CSS Grid/Flexbox
- Vanilla JavaScript
- localStorage

## Testing Checklist
- [ ] Tasks drag between columns
- [ ] Visual feedback during drag
- [ ] Tasks persist after reload
- [ ] Priority colors display
- [ ] Delete functionality works
- [ ] Mobile responsive

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
