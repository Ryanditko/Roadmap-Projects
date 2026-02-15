# Markdown Notes Editor

**Level:** Intermediate | **Time:** 1 week

## Description
Real-time markdown editor with live preview, syntax highlighting, export functionality, and note management. Learn markdown parsing, code syntax highlighting, and file handling.

## Core Features
- Split-pane editor with live preview
- Markdown syntax support
- Syntax highlighting for code blocks
- Save/load notes
- Export to HTML/PDF
- Search notes
- Dark mode

## Implementation Guide

```javascript
class MarkdownEditor {
    constructor() {
        this.notes = JSON.parse(localStorage.getItem('md-notes')) || [];
        this.currentNote = null;
        this.init();
    }
    
    init() {
        this.editor = document.getElementById('editor');
        this.preview = document.getElementById('preview');
        
        this.editor.addEventListener('input', () => this.updatePreview());
        this.loadNotes();
    }
    
    updatePreview() {
        const markdown = this.editor.value;
        const html = this.parseMarkdown(markdown);
        this.preview.innerHTML = html;
        this.highlightCode();
    }
    
    parseMarkdown(text) {
        // Headers
        text = text.replace(/^### (.*$)/gim, '<h3>$1</h3>');
        text = text.replace(/^## (.*$)/gim, '<h2>$1</h2>');
        text = text.replace(/^# (.*$)/gim, '<h1>$1</h1>');
        
        // Bold
        text = text.replace(/\*\*(.*?)\*\*/gim, '<strong>$1</strong>');
        
        // Italic
        text = text.replace(/\*(.*?)\*/gim, '<em>$1</em>');
        
        // Links
        text = text.replace(/\[(.*?)\]\((.*?)\)/gim, '<a href="$2">$1</a>');
        
        // Images
        text = text.replace(/!\[(.*?)\]\((.*?)\)/gim, '<img src="$2" alt="$1">');
        
        // Code blocks
        text = text.replace(/```(\w+)?\n([\s\S]*?)```/gim, (match, lang, code) => {
            return `<pre><code class="language-${lang || 'plaintext'}">${this.escapeHtml(code)}</code></pre>`;
        });
        
        // Inline code
        text = text.replace(/`([^`]+)`/gim, '<code>$1</code>');
        
        // Lists
        text = text.replace(/^\* (.*)$/gim, '<li>$1</li>');
        text = text.replace(/(<li>.*<\/li>)/s, '<ul>$1</ul>');
        
        // Line breaks
        text = text.replace(/\n\n/g, '</p><p>');
        text = '<p>' + text + '</p>';
        
        return text;
    }
    
    escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }
    
    highlightCode() {
        if (typeof Prism !== 'undefined') {
            Prism.highlightAll();
        }
    }
    
    saveNote() {
        const title = prompt('Note title:');
        if (!title) return;
        
        const note = {
            id: Date.now().toString(),
            title,
            content: this.editor.value,
            createdAt: new Date().toISOString()
        };
        
        this.notes.push(note);
        localStorage.setItem('md-notes', JSON.stringify(this.notes));
        this.loadNotes();
    }
    
    loadNote(noteId) {
        const note = this.notes.find(n => n.id === noteId);
        if (note) {
            this.currentNote = note;
            this.editor.value = note.content;
            this.updatePreview();
        }
    }
    
    exportHTML() {
        const html = this.preview.innerHTML;
        const blob = new Blob([html], { type: 'text/html' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'note.html';
        a.click();
    }
    
    loadNotes() {
        const notesList = document.getElementById('notes-list');
        notesList.innerHTML = this.notes.map(note => `
            <div class="note-item" onclick="editor.loadNote('${note.id}')">
                <h4>${note.title}</h4>
                <small>${new Date(note.createdAt).toLocaleDateString()}</small>
            </div>
        `).join('');
    }
}

const editor = new MarkdownEditor();
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Markdown Editor</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Segoe UI', sans-serif; display: flex; height: 100vh; }
        .sidebar {
            width: 250px;
            background: #2c3e50;
            color: white;
            padding: 20px;
            overflow-y: auto;
        }
        .main { flex: 1; display: grid; grid-template-columns: 1fr 1fr; }
        .editor-pane, .preview-pane { padding: 20px; overflow-y: auto; }
        #editor {
            width: 100%;
            height: 100%;
            border: none;
            font-family: 'Courier New', monospace;
            font-size: 14px;
            resize: none;
        }
        #preview { line-height: 1.6; }
        .note-item {
            background: #34495e;
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 5px;
            cursor: pointer;
        }
        .note-item:hover { background: #3d566e; }
    </style>
</head>
<body>
    <div class="sidebar">
        <h2>Notes</h2>
        <button onclick="editor.saveNote()">Save Note</button>
        <div id="notes-list"></div>
    </div>
    <div class="main">
        <div class="editor-pane">
            <textarea id="editor" placeholder="Write your markdown here..."></textarea>
        </div>
        <div class="preview-pane">
            <div id="preview"></div>
        </div>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
</body>
</html>
```

## Technology Stack
- Markdown parsing (custom or marked.js)
- Prism.js for syntax highlighting
- localStorage
- File API for export

## Testing Checklist
- [ ] Real-time preview updates
- [ ] All markdown syntax renders
- [ ] Code highlighting works
- [ ] Save/load functionality
- [ ] Export to HTML works
- [ ] Search filters notes

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
