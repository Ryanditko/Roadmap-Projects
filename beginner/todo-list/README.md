# To-Do List

<p align="center">
  <img src="https://images.unsplash.com/photo-1484480974693-6ca0a78fb36b?w=600&q=80" alt="Task Management" width="600">
</p>

**Level:** Beginner  
**Estimated Time:** 4-6 hours  
**Prerequisites:** Basic programming knowledge, understanding of data structures

---

## Description

Build a task management application that helps users organize their daily activities. Users can add new tasks, mark them as complete, edit existing tasks, and delete them. The application should persist data locally so tasks remain available between sessions.

This project is perfect for learning CRUD operations (Create, Read, Update, Delete) and understanding how to store data locally.

---

## Core Features

### Must Have
-  Add new tasks with a description
-  Display all tasks in a list
-  Mark tasks as completed (with visual indication)
-  Delete individual tasks
-  Edit existing task descriptions
-  Persist data locally (localStorage, file, or database)

### Nice to Have
-  Filter tasks by status (All, Active, Completed)
-  Clear all completed tasks at once
-  Count of remaining tasks
-  Undo delete action

---

## Learning Objectives

By building this project, you will learn:

- **CRUD Operations**: Master Create, Read, Update, Delete functionality
- **Data Structures**: Work with arrays and objects to manage tasks
- **State Management**: Track application state and updates
- **Local Storage**: Persist data between sessions
- **Event Handling**: Respond to user interactions
- **DOM Manipulation**: Update UI dynamically (for web)

---

## Implementation Guide

### Step 1: Design the Data Structure

Each task should be an object with these properties:

```javascript
{
  id: 1,                    // Unique identifier
  text: "Buy groceries",    // Task description
  completed: false,         // Completion status
  createdAt: "2026-02-14"   // Creation timestamp (optional)
}
```

### Step 2: Plan Core Functions

You'll need these main functions:

1. **addTask(text)**: Create a new task and add to list
2. **getTasks()**: Retrieve all tasks
3. **updateTask(id, updates)**: Modify existing task
4. **deleteTask(id)**: Remove task from list
5. **toggleComplete(id)**: Mark task as done/undone
6. **saveTasks()**: Persist tasks to storage
7. **loadTasks()**: Load tasks on app start

### Step 3: Choose Your Storage Method

**Web (localStorage):**
```javascript
// Save tasks
localStorage.setItem('tasks', JSON.stringify(tasks));

// Load tasks
const tasks = JSON.parse(localStorage.getItem('tasks')) || [];
```

**Python (JSON file):**
```python
import json

# Save tasks
with open('tasks.json', 'w') as f:
    json.dump(tasks, f)

# Load tasks
with open('tasks.json', 'r') as f:
    tasks = json.load(f)
```

### Step 4: Build the User Interface

**Web Version:**
- Input field for new tasks
- Add button
- List container for tasks
- Checkbox for completion status
- Edit and delete buttons per task

**CLI Version:**
- Menu with numbered options
- Input prompts for task details
- Display tasks with numbers
- Simple text-based interface

---

## Technology Suggestions

### Web Development
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: CSS Grid/Flexbox for layout
- **Storage**: localStorage API
- **Alternative**: React, Vue, or Angular for practice

### Python
- **GUI**: Tkinter, PyQt, or Kivy
- **Storage**: JSON file or SQLite database
- **Web**: Flask + Jinja2 templates

### Other Languages
- **Java**: Swing/JavaFX for GUI
- **C#**: WinForms or WPF
- **Node.js**: Electron for desktop app

---

## Example Code Snippets

### JavaScript (Vanilla)

```javascript
// Task Manager Class
class TodoList {
  constructor() {
    this.tasks = this.loadTasks();
  }

  addTask(text) {
    const task = {
      id: Date.now(),
      text: text,
      completed: false,
      createdAt: new Date().toISOString()
    };
    this.tasks.push(task);
    this.saveTasks();
    return task;
  }

  toggleComplete(id) {
    const task = this.tasks.find(t => t.id === id);
    if (task) {
      task.completed = !task.completed;
      this.saveTasks();
    }
  }

  deleteTask(id) {
    this.tasks = this.tasks.filter(t => t.id !== id);
    this.saveTasks();
  }

  saveTasks() {
    localStorage.setItem('tasks', JSON.stringify(this.tasks));
  }

  loadTasks() {
    const stored = localStorage.getItem('tasks');
    return stored ? JSON.parse(stored) : [];
  }
}
```

### Python

```python
import json
from datetime import datetime

class TodoList:
    def __init__(self, filename='tasks.json'):
        self.filename = filename
        self.tasks = self.load_tasks()
    
    def add_task(self, text):
        task = {
            'id': int(datetime.now().timestamp() * 1000),
            'text': text,
            'completed': False,
            'created_at': datetime.now().isoformat()
        }
        self.tasks.append(task)
        self.save_tasks()
        return task
    
    def toggle_complete(self, task_id):
        for task in self.tasks:
            if task['id'] == task_id:
                task['completed'] = not task['completed']
                self.save_tasks()
                break
    
    def delete_task(self, task_id):
        self.tasks = [t for t in self.tasks if t['id'] != task_id]
        self.save_tasks()
    
    def save_tasks(self):
        with open(self.filename, 'w') as f:
            json.dump(self.tasks, f, indent=2)
    
    def load_tasks(self):
        try:
            with open(self.filename, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return []
```

---

## Testing Checklist

- [ ] Can add new tasks with various text lengths
- [ ] Tasks persist after closing and reopening the app
- [ ] Can mark tasks as completed/incomplete
- [ ] Can edit task descriptions
- [ ] Can delete individual tasks
- [ ] UI updates immediately after each action
- [ ] Handles empty input (shows error or prevents submission)
- [ ] Works with special characters in task text
- [ ] Completed tasks have distinct visual styling

---

## Additional Challenges

### Easy
-  Add task counter (e.g., "3 of 10 tasks completed")
-  Add "Clear completed" button to remove all done tasks
-  Implement filter buttons (All / Active / Completed)

### Medium
-  Add task priority levels (Low, Medium, High) with color coding
-  Add due dates with calendar picker
-  Sort tasks by date, priority, or alphabetically
-  Add task categories/tags with color labels
-  Implement drag-and-drop reordering

### Hard
-  Add subtasks (nested todo items)
-  Implement search and filter functionality
-  Add recurring tasks (daily, weekly, monthly)
-  Create multiple lists/projects
-  Add reminders/notifications
-  Export tasks to CSV or PDF
-  Sync across devices using backend API

---

## Common Pitfalls

 **Not handling edge cases**: Empty input, duplicate tasks, invalid IDs  
 **Poor data persistence**: Not saving immediately after changes  
 **No input validation**: Allowing empty or extremely long task descriptions  
 **Memory leaks**: Not properly cleaning up event listeners (web apps)  
 **No user feedback**: Missing visual confirmation of actions  

---

## UI/UX Best Practices

1. **Visual Feedback**: Show immediate response to user actions
2. **Accessibility**: Use semantic HTML and ARIA labels
3. **Mobile Friendly**: Ensure touch targets are large enough (min 44x44px)
4. **Keyboard Navigation**: Support Tab, Enter, and Delete keys
5. **Empty State**: Show helpful message when no tasks exist
6. **Confirmation**: Ask before deleting tasks (or add undo)

---

## Resources

### Tutorials
- [MDN: Working with localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [JavaScript Array Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [Python JSON Module](https://docs.python.org/3/library/json.html)

### Design Inspiration
- [TodoMVC](https://todomvc.com/) - Todo app implementations in different frameworks
- [Todoist](https://todoist.com/) - Professional task manager
- [Microsoft To Do](https://to-do.microsoft.com/) - Simple and clean design

### Code Examples
- [MDN Todo Example](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/React_todo_list_beginning)

---

## Submission Checklist

Before considering your project complete:

- [ ] All core features are implemented and working
- [ ] Code is organized and well-commented
- [ ] Data persists correctly between sessions
- [ ] UI is intuitive and user-friendly
- [ ] Error handling is in place
- [ ] Project has been tested with various scenarios
- [ ] README includes setup/run instructions

---

## Portfolio Tips

When showcasing this project:

1. **Live Demo**: Deploy web version to GitHub Pages or Vercel
2. **Screenshot/GIF**: Show the app in action
3. **Feature List**: Highlight unique features you added
4. **Code Walkthrough**: Explain interesting parts of your implementation
5. **Lessons Learned**: Share challenges and solutions

---

## License

This project idea is part of the [Project Ideas Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
