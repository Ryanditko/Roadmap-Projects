# Trello Clone (Task Manager)

<p align="center">
  <img src="https://images.unsplash.com/photo-1484480974693-6ca0a78fb36b?w=600&q=80" alt="Project Management" width="600">
</p>

## Overview

**Difficulty:** Advanced  
**Estimated Time:** 25–30 hours  
**Prerequisites:** React/Vue, Node.js, WebSockets, MongoDB/PostgreSQL, Drag-and-Drop libraries  

A **Trello Clone** is a sophisticated project management tool that allows teams to organize tasks using the Kanban methodology. Users can:

- Create boards, lists, and cards with drag-and-drop functionality
- Collaborate in real-time with team members
- Manage permissions and access control
- Track activity and changes with complete audit logs
- Add rich content (descriptions, checklists, attachments, due dates)
- Receive notifications for updates

This project covers complex drag-and-drop interactions, real-time collaboration with WebSockets, hierarchical data structures, optimistic UI updates, and building a production-ready collaborative tool similar to Trello, Asana, or Monday.com.

---

## Features

### Must Have

- User authentication and profiles
- Board creation and management
- Lists (columns) within boards
- Cards within lists
- Drag-and-drop cards between lists with smooth animations
- Drag-and-drop lists to reorder
- Card details modal with:
  - Title and description (rich text)
  - Comments
  - Checklists with progress tracking
  - Due dates
  - Labels/tags
  - Member assignments
  - Attachments
- Board members management
- Activity log for all actions
- Search and filter functionality
- Responsive design (mobile and desktop)

### Nice to Have

- Real-time collaboration with WebSockets
- Board templates
- Custom backgrounds for boards
- Card cover images
- Archived cards and boards
- Card copying and moving between boards
- Power-ups/integrations (Slack, GitHub, Google Drive)
- Email notifications
- Board sharing with public links
- Calendar view
- Butler-like automations (rules, buttons, commands)
- Time tracking
- Voting on cards
- Card dependencies
- Swimlanes
- Mobile app (React Native/Flutter)
- Offline support with sync
- Advanced search with filters
- Export functionality (JSON, CSV, PDF)

---

## Learning Objectives

By building this project, you will learn:

- **Complex drag-and-drop** with smooth animations
- **Real-time collaboration** with WebSockets and operational transformation
- **Optimistic UI updates** for instant feedback
- Hierarchical data management (workspaces → boards → lists → cards)
- **Conflict resolution** in collaborative editing
- **Permission systems** (RBAC - owner, admin, member, viewer)
- **Rich text editing** with Quill or TipTap
- **File upload** and cloud storage integration
- **Activity logging** and complete audit trails
- **Search and filtering** with Elasticsearch or MongoDB text search
- **State management** at scale (Redux/Zustand/Pinia)
- **Real-time synchronization** patterns and best practices
- **Performance optimization** for large boards
- **Testing strategies** for complex interactive UIs

---

## Project Concepts

This project simulates enterprise collaboration software and covers:

- **Kanban Methodology:** Visual task management with swim lanes
- **Real-time Sync:** Multiple users editing simultaneously without conflicts
- **Optimistic Updates:** UI updates immediately, syncs in background
- **Conflict Resolution:** Last-write-wins or operational transformation (OT)
- **RBAC:** Role-based access control for boards
- **Activity Tracking:** Complete audit log of all changes
- **Event Sourcing:** Store all changes as events for history and undo
- **Microservices Pattern:** Separate services for different concerns

---

## Step-by-Step Implementation

### Step 1: Data Structure Design

Define the hierarchical data model for boards, lists, and cards:

```javascript
// models/Board.js
const mongoose = require('mongoose');

const boardSchema = new mongoose.Schema({
    _id: mongoose.Schema.Types.ObjectId,
    title: { 
        type: String, 
        required: true,
        trim: true,
        maxlength: 512
    },
    description: {
        type: String,
        maxlength: 16384
    },
    background: {
        type: { type: String, enum: ['color', 'image'], default: 'color' },
        value: { type: String, default: '#0079bf' } // hex color or image URL
    },
    ownerId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User', 
        required: true 
    },
    members: [{
        userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        role: { 
            type: String, 
            enum: ['owner', 'admin', 'member', 'viewer'], 
            default: 'member' 
        },
        addedAt: { type: Date, default: Date.now },
        addedBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
    }],
    visibility: { 
        type: String, 
        enum: ['private', 'workspace', 'public'], 
        default: 'private' 
    },
    starred: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    archived: { type: Boolean, default: false },
    closedAt: Date,
    workspace: { type: mongoose.Schema.Types.ObjectId, ref: 'Workspace' },
    prefs: {
        permissionLevel: { 
            type: String, 
            enum: ['private', 'org', 'public'], 
            default: 'private' 
        },
        voting: { type: String, enum: ['disabled', 'members', 'public'], default: 'disabled' },
        comments: { type: String, enum: ['disabled', 'members', 'public'], default: 'members' },
        invitations: { type: String, enum: ['members', 'admins'], default: 'members' },
        selfJoin: { type: Boolean, default: false },
        cardCovers: { type: Boolean, default: true },
        cardAging: { type: String, enum: ['regular', 'pirate'], default: 'regular' }
    },
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now }
}, {
    timestamps: true
});

// Indexes for performance
boardSchema.index({ ownerId: 1 });
boardSchema.index({ 'members.userId': 1 });
boardSchema.index({ workspace: 1 });
boardSchema.index({ createdAt: -1 });

module.exports = mongoose.model('Board', boardSchema);

// models/List.js
const listSchema = new mongoose.Schema({
    _id: mongoose.Schema.Types.ObjectId,
    boardId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'Board', 
        required: true 
    },
    title: { 
        type: String, 
        required: true,
        trim: true,
        maxlength: 512
    },
    position: { 
        type: Number, 
        required: true 
    }, // Fractional indexing for smooth reordering
    archived: { type: Boolean, default: false },
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now }
}, {
    timestamps: true
});

listSchema.index({ boardId: 1, position: 1 });
listSchema.index({ boardId: 1, archived: 1 });

module.exports = mongoose.model('List', listSchema);

// models/Card.js
const cardSchema = new mongoose.Schema({
    _id: mongoose.Schema.Types.ObjectId,
    listId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'List', 
        required: true 
    },
    boardId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'Board', 
        required: true 
    },
    title: { 
        type: String, 
        required: true,
        trim: true,
        maxlength: 16384
    },
    description: {
        type: String,
        maxlength: 16384
    },
    position: { 
        type: Number, 
        required: true 
    },
    
    // Visual elements
    coverImage: {
        url: String,
        color: String,
        brightness: { type: String, enum: ['light', 'dark'], default: 'light' }
    },
    
    // Labels
    labels: [{
        _id: mongoose.Schema.Types.ObjectId,
        name: String,
        color: { 
            type: String, 
            enum: ['green', 'yellow', 'orange', 'red', 'purple', 'blue', 'sky', 'lime', 'pink', 'black']
        }
    }],
    
    // Members
    members: [{ 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User' 
    }],
    
    // Dates
    dueDate: Date,
    dueComplete: { type: Boolean, default: false },
    dueReminder: Date,
    startDate: Date,
    
    // Checklists
    checklists: [{
        _id: mongoose.Schema.Types.ObjectId,
        title: { type: String, required: true },
        position: Number,
        items: [{
            _id: mongoose.Schema.Types.ObjectId,
            name: { type: String, required: true },
            completed: { type: Boolean, default: false },
            completedAt: Date,
            completedBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
            dueDate: Date,
            position: Number
        }]
    }],
    
    // Attachments
    attachments: [{
        _id: mongoose.Schema.Types.ObjectId,
        name: String,
        url: String,
        mimeType: String,
        size: Number,
        uploadedBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        uploadedAt: { type: Date, default: Date.now },
        iscover: { type: Boolean, default: false }
    }],
    
    // Custom fields
    customFields: [{
        fieldId: mongoose.Schema.Types.ObjectId,
        value: mongoose.Schema.Types.Mixed
    }],
    
    // Voting
    votes: [{ 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User' 
    }],
    
    // Meta
    archived: { type: Boolean, default: false },
    subscribed: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    createdBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now }
}, {
    timestamps: true
});

cardSchema.index({ listId: 1, position: 1 });
cardSchema.index({ boardId: 1 });
cardSchema.index({ members: 1 });
cardSchema.index({ dueDate: 1 });
cardSchema.index({ archived: 1 });

// Text search index
cardSchema.index({ title: 'text', description: 'text' });

module.exports = mongoose.model('Card', cardSchema);

// models/Comment.js
const commentSchema = new mongoose.Schema({
    cardId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'Card', 
        required: true 
    },
    userId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User', 
        required: true 
    },
    text: { 
        type: String, 
        required: true,
        maxlength: 16384
    },
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now },
    edited: { type: Boolean, default: false }
}, {
    timestamps: true
});

commentSchema.index({ cardId: 1, createdAt: -1 });

module.exports = mongoose.model('Comment', commentSchema);

// models/Activity.js
const activitySchema = new mongoose.Schema({
    boardId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'Board', 
        required: true 
    },
    userId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User', 
        required: true 
    },
    action: { 
        type: String, 
        required: true,
        enum: [
            'card_created', 'card_updated', 'card_moved', 'card_archived', 'card_deleted',
            'list_created', 'list_updated', 'list_moved', 'list_archived',
            'member_added', 'member_removed', 'member_role_changed',
            'comment_added', 'comment_updated', 'comment_deleted',
            'attachment_added', 'attachment_deleted',
            'checklist_added', 'checklist_item_completed',
            'due_date_added', 'due_date_completed',
            'label_added', 'label_removed',
            'board_updated', 'board_starred'
        ]
    },
    targetType: { 
        type: String, 
        enum: ['card', 'list', 'board', 'member', 'comment', 'checklist']
    },
    targetId: mongoose.Schema.Types.ObjectId,
    metadata: mongoose.Schema.Types.Mixed, // Additional context about the action
    createdAt: { type: Date, default: Date.now, expires: 60 * 60 * 24 * 90 } // Auto-delete after 90 days
});

activitySchema.index({ boardId: 1, createdAt: -1 });
activitySchema.index({ targetId: 1, targetType: 1 });

module.exports = mongoose.model('Activity', activitySchema);
```

### Step 2: Drag and Drop Implementation with React DnD

Build the drag-and-drop interface using @dnd-kit:

```jsx
// components/Board.jsx
import React, { useState, useEffect } from 'react';
import { 
    DndContext, 
    DragOverlay, 
    closestCorners,
    PointerSensor,
    useSensor,
    useSensors 
} from '@dnd-kit/core';
import { 
    SortableContext, 
    horizontalListSortingStrategy,
    arrayMove 
} from '@dnd-kit/sortable';
import List from './List';
import Card from './Card';
import { useBoardStore } from '../stores/boardStore';
import { useWebSocket } from '../hooks/useWebSocket';

function Board({ boardId }) {
    const { 
        board, 
        lists, 
        cards, 
        moveCard, 
        moveList,
        loadBoard 
    } = useBoardStore();
    
    const { socket, emit } = useWebSocket();
    const [activeCard, setActiveCard] = useState(null);
    const [activeList, setActiveList] = useState(null);
    
    const sensors = useSensors(
        useSensor(PointerSensor, {
            activationConstraint: {
                distance: 8, // 8px movement required before drag starts
            },
        })
    );
    
    useEffect(() => {
        loadBoard(boardId);
    }, [boardId]);
    
    // Listen to real-time updates
    useEffect(() => {
        if (!socket) return;
        
        socket.on('card:moved', handleRemoteCardMove);
        socket.on('card:updated', handleRemoteCardUpdate);
        socket.on('list:moved', handleRemoteListMove);
        
        return () => {
            socket.off('card:moved');
            socket.off('card:updated');
            socket.off('list:moved');
        };
    }, [socket]);
    
    const handleDragStart = (event) => {
        const { active } = event;
        
        if (active.data.current.type === 'card') {
            setActiveCard(active.data.current.card);
        } else if (active.data.current.type === 'list') {
            setActiveList(active.data.current.list);
        }
    };
    
    const handleDragEnd = (event) => {
        const { active, over } = event;
        
        if (!over) {
            setActiveCard(null);
            setActiveList(null);
            return;
        }
        
        if (active.data.current.type === 'card') {
            handleCardDragEnd(active, over);
        } else if (active.data.current.type === 'list') {
            handleListDragEnd(active, over);
        }
        
        setActiveCard(null);
        setActiveList(null);
    };
    
    const handleCardDragEnd = (active, over) => {
        const cardId = active.id;
        const sourceListId = active.data.current.listId;
        const targetListId = over.data.current.listId || over.id;
        
        // Calculate new position
        const targetList = lists.find(l => l._id === targetListId);
        const targetCards = cards.filter(c => c.listId === targetListId && c._id !== cardId);
        
        let newPosition;
        if (over.data.current.type === 'card') {
            const overIndex = targetCards.findIndex(c => c._id === over.id);
            const overCard = targetCards[overIndex];
            const prevCard = targetCards[overIndex - 1];
            
            // Fractional indexing for smooth positioning
            if (!prevCard) {
                newPosition = overCard.position / 2;
            } else {
                newPosition = (prevCard.position + overCard.position) / 2;
            }
        } else {
            // Dropped at the end of list
            newPosition = targetCards.length > 0 
                ? targetCards[targetCards.length - 1].position + 1 
                : 1;
        }
        
        // Optimistic update
        moveCard(cardId, sourceListId, targetListId, newPosition);
        
        // Sync with server
        emit('card:move', {
            cardId,
            sourceListId,
            targetListId,
            position: newPosition,
            timestamp: Date.now()
        });
    };
    
    const handleListDragEnd = (active, over) => {
        if (active.id === over.id) return;
        
        const oldIndex = lists.findIndex(l => l._id === active.id);
        const newIndex = lists.findIndex(l => l._id === over.id);
        
        // Reorder lists
        const reorderedLists = arrayMove(lists, oldIndex, newIndex);
        
        // Update positions
        const updates = reorderedLists.map((list, index) => ({
            listId: list._id,
            position: index
        }));
        
        // Optimistic update
        moveList(active.id, newIndex);
        
        // Sync with server
        emit('list:reorder', {
            updates,
            timestamp: Date.now()
        });
    };
    
    const handleDragOver = (event) => {
        const { active, over } = event;
        
        if (!over) return;
        
        // Handle dragging card over different list
        if (active.data.current.type === 'card' && 
            over.data.current.type === 'list' &&
            active.data.current.listId !== over.id) {
            
            // Move card to new list temporarily for visual feedback
            moveCard(active.id, active.data.current.listId, over.id, 0);
        }
    };
    
    const handleRemoteCardMove = (data) => {
        // Don't process our own events
        if (data.userId === socket.id) return;
        
        moveCard(data.cardId, data.sourceListId, data.targetListId, data.position);
    };
    
    const handleRemoteCardUpdate = (data) => {
        if (data.userId === socket.id) return;
        
        // Update card in store
        // Implementation depends on your state management
    };
    
    const handleRemoteListMove = (data) => {
        if (data.userId === socket.id) return;
        
        // Update list positions
        // Implementation depends on your state management
    };
    
    if (!board) {
        return <div className="loading">Loading board...</div>;
    }
    
    return (
        <DndContext
            sensors={sensors}
            collisionDetection={closestCorners}
            onDragStart={handleDragStart}
            onDragEnd={handleDragEnd}
            onDragOver={handleDragOver}
        >
            <div className="board-container" style={{ 
                background: board.background.type === 'color' 
                    ? board.background.value 
                    : `url(${board.background.value})` 
            }}>
                {/* Board Header */}
                <div className="board-header">
                    <div className="board-header-left">
                        <h1 className="board-title">{board.title}</h1>
                        {board.starred.includes(currentUserId) && <span className="star">⭐</span>}
                        <button className="board-visibility">
                            {board.visibility === 'private' ? '🔒' : '🌐'}
                        </button>
                    </div>
                    
                    <div className="board-header-right">
                        <BoardMembers members={board.members} />
                        <button onClick={() => openBoardMenu()}>
                            ⋮ Show Menu
                        </button>
                    </div>
                </div>
                
                {/* Lists */}
                <div className="board-lists">
                    <SortableContext 
                        items={lists.map(l => l._id)}
                        strategy={horizontalListSortingStrategy}
                    >
                        {lists.filter(l => !l.archived).map(list => (
                            <List 
                                key={list._id} 
                                list={list}
                                cards={cards.filter(c => c.listId === list._id && !c.archived)}
                            />
                        ))}
                    </SortableContext>
                    
                    <AddListButton boardId={boardId} />
                </div>
            </div>
            
            {/* Drag Overlay */}
            <DragOverlay>
                {activeCard && <Card card={activeCard} isDragging />}
                {activeList && <List list={activeList} isDragging />}
            </DragOverlay>
        </DndContext>
    );
}

export default Board;

// components/List.jsx
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';
import Card from './Card';

function List({ list, cards, isDragging = false }) {
    const {
        attributes,
        listeners,
        setNodeRef,
        transform,
        transition,
        isDragging: isSortableDragging
    } = useSortable({
        id: list._id,
        data: {
            type: 'list',
            list: list
        }
    });
    
    const style = {
        transform: CSS.Transform.toString(transform),
        transition,
        opacity: isDragging || isSortableDragging ? 0.5 : 1
    };
    
    const completedCards = cards.filter(c => c.dueComplete).length;
    const totalCards = cards.length;
    
    return (
        <div 
            ref={setNodeRef} 
            style={style} 
            className="list"
        >
            <div className="list-header" {...attributes} {...listeners}>
                <h3 className="list-title">{list.title}</h3>
                <span className="list-card-count">{totalCards}</span>
                <button className="list-menu-btn" onClick={(e) => {
                    e.stopPropagation();
                    openListMenu(list._id);
                }}>
                    ⋮
                </button>
            </div>
            
            {totalCards > 0 && (
                <div className="list-progress">
                    <div 
                        className="list-progress-bar" 
                        style={{ width: `${(completedCards / totalCards) * 100}%` }}
                    />
                </div>
            )}
            
            <SortableContext 
                items={cards.map(c => c._id)}
                strategy={verticalListSortingStrategy}
            >
                <div className="list-cards">
                    {cards.map(card => (
                        <Card key={card._id} card={card} />
                    ))}
                </div>
            </SortableContext>
            
            <AddCardButton listId={list._id} />
        </div>
    );
}

export default List;

// components/Card.jsx
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { format } from 'date-fns';

function Card({ card, isDragging = false }) {
    const {
        attributes,
        listeners,
        setNodeRef,
        transform,
        transition
    } = useSortable({
        id: card._id,
        data: {
            type: 'card',
            card: card,
            listId: card.listId
        }
    });
    
    const style = {
        transform: CSS.Transform.toString(transform),
        transition,
        cursor: isDragging ? 'grabbing' : 'grab'
    };
    
    // Calculate checklist progress
    const checklistItems = card.checklists?.reduce((sum, cl) => 
        sum + cl.items.length, 0) || 0;
    const completedItems = card.checklists?.reduce((sum, cl) => 
        sum + cl.items.filter(i => i.completed).length, 0) || 0;
    
    // Check if due date is approaching or overdue
    const isDueSoon = card.dueDate && !card.dueComplete && 
        new Date(card.dueDate) - new Date() < 24 * 60 * 60 * 1000;
    const isOverdue = card.dueDate && !card.dueComplete && 
        new Date(card.dueDate) < new Date();
    
    return (
        <div
            ref={setNodeRef}
            style={style}
            className={`card ${isDragging ? 'dragging' : ''}`}
            {...attributes}
            {...listeners}
            onClick={(e) => {
                if (!isDragging) {
                    openCardModal(card._id);
                }
            }}
        >
            {/* Cover Image */}
            {card.coverImage?.url && (
                <div className="card-cover" style={{
                    backgroundImage: `url(${card.coverImage.url})`
                }} />
            )}
            
            {/* Labels */}
            {card.labels.length > 0 && (
                <div className="card-labels">
                    {card.labels.map(label => (
                        <span 
                            key={label._id} 
                            className={`label label-${label.color}`}
                            title={label.name}
                        >
                            {label.name}
                        </span>
                    ))}
                </div>
            )}
            
            {/* Title */}
            <div className="card-title">{card.title}</div>
            
            {/* Badges */}
            <div className="card-badges">
                {/* Due Date */}
                {card.dueDate && (
                    <span className={`badge badge-due ${
                        card.dueComplete ? 'complete' : 
                        isOverdue ? 'overdue' : 
                        isDueSoon ? 'soon' : ''
                    }`}>
                        <span className="icon">🕐</span>
                        {format(new Date(card.dueDate), 'MMM d')}
                        {card.dueComplete && <span className="icon">✓</span>}
                    </span>
                )}
                
                {/* Description */}
                {card.description && (
                    <span className="badge">
                        <span className="icon">📝</span>
                    </span>
                )}
                
                {/* Comments */}
                {card.commentCount > 0 && (
                    <span className="badge">
                        <span className="icon">💬</span>
                        {card.commentCount}
                    </span>
                )}
                
                {/* Attachments */}
                {card.attachments.length > 0 && (
                    <span className="badge">
                        <span className="icon">📎</span>
                        {card.attachments.length}
                    </span>
                )}
                
                {/* Checklist */}
                {checklistItems > 0 && (
                    <span className={`badge ${
                        completedItems === checklistItems ? 'complete' : ''
                    }`}>
                        <span className="icon">✓</span>
                        {completedItems}/{checklistItems}
                    </span>
                )}
                
                {/* Votes */}
                {card.votes.length > 0 && (
                    <span className="badge">
                        <span className="icon">👍</span>
                        {card.votes.length}
                    </span>
                )}
            </div>
            
            {/* Members */}
            {card.members.length > 0 && (
                <div className="card-members">
                    {card.members.slice(0, 3).map(member => (
                        <Avatar 
                            key={member._id} 
                            user={member} 
                            size="small" 
                        />
                    ))}
                    {card.members.length > 3 && (
                        <span className="member-overflow">
                            +{card.members.length - 3}
                        </span>
                    )}
                </div>
            )}
        </div>
    );
}

export default Card;
```

### Step 3: Real-Time Collaboration with WebSockets

Implement WebSocket communication for real-time updates:

```javascript
// server/websocketServer.js
const socketIo = require('socket.io');
const jwt = require('jsonwebtoken');
const Board = require('../models/Board');
const Card = require('../models/Card');
const List = require('../models/List');
const Activity = require('../models/Activity');

function initializeWebSocket(server) {
    const io = socketIo(server, {
        cors: {
            origin: process.env.CLIENT_URL,
            credentials: true
        },
        pingTimeout: 60000,
        pingInterval: 25000
    });
    
    // Authentication middleware
    io.use(async (socket, next) => {
        try {
            const token = socket.handshake.auth.token;
            const decoded = jwt.verify(token, process.env.JWT_SECRET);
            socket.userId = decoded.userId;
            socket.username = decoded.username;
            next();
        } catch (error) {
            next(new Error('Authentication error'));
        }
    });
    
    io.on('connection', (socket) => {
        console.log(`User connected: ${socket.username} (${socket.userId})`);
        
        // Join board room
        socket.on('board:join', async ({ boardId }) => {
            try {
                // Verify user has access to board
                const board = await Board.findById(boardId);
                const hasAccess = board.ownerId.equals(socket.userId) || 
                    board.members.some(m => m.userId.equals(socket.userId));
                
                if (!hasAccess) {
                    socket.emit('error', { message: 'Access denied' });
                    return;
                }
                
                socket.join(`board:${boardId}`);
                
                // Notify others
                socket.to(`board:${boardId}`).emit('user:joined', {
                    userId: socket.userId,
                    username: socket.username,
                    timestamp: Date.now()
                });
                
                // Send current active users
                const room = io.sockets.adapter.rooms.get(`board:${boardId}`);
                const activeUsers = Array.from(room || [])
                    .map(socketId => io.sockets.sockets.get(socketId))
                    .filter(s => s.userId !== socket.userId)
                    .map(s => ({ userId: s.userId, username: s.username }));
                
                socket.emit('board:users', activeUsers);
                
                console.log(`User ${socket.username} joined board ${boardId}`);
                
            } catch (error) {
                console.error('Error joining board:', error);
                socket.emit('error', { message: 'Failed to join board' });
            }
        });
        
        // Handle card movement
        socket.on('card:move', async (data) => {
            const { cardId, sourceListId, targetListId, position, timestamp } = data;
            
            try {
                // Update card in database
                const card = await Card.findByIdAndUpdate(
                    cardId,
                    { 
                        listId: targetListId, 
                        position: position,
                        updatedAt: new Date()
                    },
                    { new: true }
                ).populate('members');
                
                // Log activity
                await Activity.create({
                    boardId: card.boardId,
                    userId: socket.userId,
                    action: 'card_moved',
                    targetType: 'card',
                    targetId: cardId,
                    metadata: {
                        cardTitle: card.title,
                        sourceListId,
                        targetListId,
                        position
                    }
                });
                
                // Broadcast to all users in the board except sender
                socket.to(`board:${card.boardId}`).emit('card:moved', {
                    cardId,
                    sourceListId,
                    targetListId,
                    position,
                    userId: socket.userId,
                    username: socket.username,
                    timestamp: Date.now()
                });
                
                // Send confirmation to sender
                socket.emit('card:move:success', { cardId, timestamp });
                
            } catch (error) {
                console.error('Error moving card:', error);
                socket.emit('card:move:error', { 
                    cardId, 
                    message: error.message,
                    timestamp 
                });
            }
        });
        
        // Handle card updates
        socket.on('card:update', async (data) => {
            const { cardId, updates, timestamp } = data;
            
            try {
                const card = await Card.findByIdAndUpdate(
                    cardId,
                    { ...updates, updatedAt: new Date() },
                    { new: true }
                ).populate('members');
                
                // Log activity
                await Activity.create({
                    boardId: card.boardId,
                    userId: socket.userId,
                    action: 'card_updated',
                    targetType: 'card',
                    targetId: cardId,
                    metadata: {
                        cardTitle: card.title,
                        updates: Object.keys(updates)
                    }
                });
                
                socket.to(`board:${card.boardId}`).emit('card:updated', {
                    cardId,
                    updates,
                    card,
                    userId: socket.userId,
                    username: socket.username,
                    timestamp: Date.now()
                });
                
                socket.emit('card:update:success', { cardId, timestamp });
                
            } catch (error) {
                console.error('Error updating card:', error);
                socket.emit('card:update:error', { 
                    cardId, 
                    message: error.message,
                    timestamp 
                });
            }
        });
        
        // Handle list reordering
        socket.on('list:reorder', async (data) => {
            const { updates, timestamp } = data;
            
            try {
                // Bulk update list positions
                const bulkOps = updates.map(({ listId, position }) => ({
                    updateOne: {
                        filter: { _id: listId },
                        update: { position, updatedAt: new Date() }
                    }
                }));
                
                await List.bulkWrite(bulkOps);
                
                // Get board ID from first list
                const firstList = await List.findById(updates[0].listId);
                
                // Log activity
                await Activity.create({
                    boardId: firstList.boardId,
                    userId: socket.userId,
                    action: 'list_moved',
                    targetType: 'list',
                    metadata: { updates }
                });
                
                socket.to(`board:${firstList.boardId}`).emit('list:reordered', {
                    updates,
                    userId: socket.userId,
                    username: socket.username,
                    timestamp: Date.now()
                });
                
                socket.emit('list:reorder:success', { timestamp });
                
            } catch (error) {
                console.error('Error reordering lists:', error);
                socket.emit('list:reorder:error', { 
                    message: error.message,
                    timestamp 
                });
            }
        });
        
        // Handle typing indicator
        socket.on('card:typing', ({ cardId, boardId }) => {
            socket.to(`board:${boardId}`).emit('card:typing', {
                cardId,
                userId: socket.userId,
                username: socket.username
            });
        });
        
        socket.on('card:stop_typing', ({ cardId, boardId }) => {
            socket.to(`board:${boardId}`).emit('card:stop_typing', {
                cardId,
                userId: socket.userId
            });
        });
        
        // Handle disconnect
        socket.on('disconnect', () => {
            console.log(`User disconnected: ${socket.username}`);
            
            // Notify all boards user was in
            const rooms = Array.from(socket.rooms)
                .filter(room => room.startsWith('board:'));
            
            rooms.forEach(room => {
                socket.to(room).emit('user:left', {
                    userId: socket.userId,
                    username: socket.username,
                    timestamp: Date.now()
                });
            });
        });
    });
    
    return io;
}

module.exports = { initializeWebSocket };

// client/hooks/useWebSocket.js
import { useEffect, useState, useCallback } from 'react';
import io from 'socket.io-client';
import { useAuth } from './useAuth';

export function useWebSocket() {
    const [socket, setSocket] = useState(null);
    const [connected, setConnected] = useState(false);
    const { token } = useAuth();
    
    useEffect(() => {
        if (!token) return;
        
        const newSocket = io(process.env.REACT_APP_WS_URL, {
            auth: { token },
            reconnection: true,
            reconnectionDelay: 1000,
            reconnectionDelayMax: 5000,
            reconnectionAttempts: 5
        });
        
        newSocket.on('connect', () => {
            console.log('WebSocket connected');
            setConnected(true);
        });
        
        newSocket.on('disconnect', (reason) => {
            console.log('WebSocket disconnected:', reason);
            setConnected(false);
        });
        
        newSocket.on('error', (error) => {
            console.error('WebSocket error:', error);
        });
        
        setSocket(newSocket);
        
        return () => {
            newSocket.close();
        };
    }, [token]);
    
    const emit = useCallback((event, data) => {
        if (socket && connected) {
            socket.emit(event, data);
        } else {
            console.warn('Socket not connected, event not sent:', event);
        }
    }, [socket, connected]);
    
    const on = useCallback((event, callback) => {
        if (socket) {
            socket.on(event, callback);
        }
    }, [socket]);
    
    const off = useCallback((event, callback) => {
        if (socket) {
            socket.off(event, callback);
        }
    }, [socket]);
    
    return { socket, connected, emit, on, off };
}
```

### Step 4: Card Details Modal

Create a comprehensive card editing interface:

```jsx
// components/CardModal.jsx
import React, { useState, useEffect } from 'react';
import { useEditor, EditorContent } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';
import Placeholder from '@tiptap/extension-placeholder';
import Link from '@tiptap/extension-link';
import { format } from 'date-fns';

function CardModal({ cardId, onClose }) {
    const [card, setCard] = useState(null);
    const [comments, setComments] = useState([]);
    const [newComment, setNewComment] = useState('');
    const [loading, setLoading] = useState(true);
    const [activeTab, setActiveTab] = useState('details');
    
    const editor = useEditor({
        extensions: [
            StarterKit,
            Placeholder.configure({
                placeholder: 'Add a more detailed description...'
            }),
            Link.configure({
                openOnClick: false
            })
        ],
        content: card?.description || '',
        onUpdate: ({ editor }) => {
            debouncedUpdateDescription(editor.getHTML());
        }
    });
    
    useEffect(() => {
        loadCard();
        loadComments();
    }, [cardId]);
    
    const loadCard = async () => {
        try {
            const response = await fetch(`/api/cards/${cardId}`);
            const data = await response.json();
            setCard(data);
            if (editor) {
                editor.commands.setContent(data.description || '');
            }
            setLoading(false);
        } catch (error) {
            console.error('Error loading card:', error);
        }
    };
    
    const loadComments = async () => {
        try {
            const response = await fetch(`/api/cards/${cardId}/comments`);
            const data = await response.json();
            setComments(data);
        } catch (error) {
            console.error('Error loading comments:', error);
        }
    };
    
    const updateCard = async (updates) => {
        try {
            await fetch(`/api/cards/${cardId}`, {
                method: 'PATCH',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(updates)
            });
            
            setCard(prev => ({ ...prev, ...updates }));
        } catch (error) {
            console.error('Error updating card:', error);
        }
    };
    
    const debouncedUpdateDescription = debounce((description) => {
        updateCard({ description });
    }, 1000);
    
    const addComment = async () => {
        if (!newComment.trim()) return;
        
        try {
            const response = await fetch(`/api/cards/${cardId}/comments`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ text: newComment })
            });
            
            const comment = await response.json();
            setComments(prev => [...prev, comment]);
            setNewComment('');
        } catch (error) {
            console.error('Error adding comment:', error);
        }
    };
    
    const toggleChecklist Item = async (checklistId, itemId) => {
        const checklist = card.checklists.find(cl => cl._id === checklistId);
        const item = checklist.items.find(i => i._id === itemId);
        
        item.completed = !item.completed;
        item.completedAt = item.completed ? new Date() : null;
        
        await updateCard({ checklists: card.checklists });
    };
    
    if (loading) {
        return <div className="modal-loading">Loading...</div>;
    }
    
    return (
        <div className="modal-overlay" onClick={onClose}>
            <div className="card-modal" onClick={e => e.stopPropagation()}>
                {/* Header */}
                <div className="modal-header">
                    <div className="modal-header-icon">📋</div>
                    <input
                        type="text"
                        className="card-title-input"
                        value={card.title}
                        onChange={(e) => updateCard({ title: e.target.value })}
                    />
                    <button className="close-btn" onClick={onClose}>×</button>
                </div>
                
                <div className="modal-body">
                    <div className="modal-main">
                        {/* Members */}
                        {card.members.length > 0 && (
                            <div className="section">
                                <h4>Members</h4>
                                <div className="members-list">
                                    {card.members.map(member => (
                                        <div key={member._id} className="member-item">
                                            <Avatar user={member} />
                                            <span>{member.username}</span>
                                            <button onClick={() => removeMember(member._id)}>
                                                ×
                                            </button>
                                        </div>
                                    ))}
                                </div>
                            </div>
                        )}
                        
                        {/* Labels */}
                        {card.labels.length > 0 && (
                            <div className="section">
                                <h4>Labels</h4>
                                <div className="labels-list">
                                    {card.labels.map(label => (
                                        <span
                                            key={label._id}
                                            className={`label label-${label.color}`}
                                        >
                                            {label.name}
                                            <button onClick={() => removeLabel(label._id)}>
                                                ×
                                            </button>
                                        </span>
                                    ))}
                                </div>
                            </div>
                        )}
                        
                        {/* Dates */}
                        {(card.startDate || card.dueDate) && (
                            <div className="section">
                                <h4>Dates</h4>
                                <div className="dates-container">
                                    {card.startDate && (
                                        <div>Start: {format(new Date(card.startDate), 'MMM d, yyyy')}</div>
                                    )}
                                    {card.dueDate && (
                                        <div className="due-date-container">
                                            <input
                                                type="checkbox"
                                                checked={card.dueComplete}
                                                onChange={(e) => updateCard({ dueComplete: e.target.checked })}
                                            />
                                            <span>Due: {format(new Date(card.dueDate), 'MMM d, yyyy h:mm a')}</span>
                                            {card.dueComplete && <span className="complete-badge">Complete</span>}
                                        </div>
                                    )}
                                </div>
                            </div>
                        )}
                        
                        {/* Description */}
                        <div className="section">
                            <div className="section-header">
                                <h4>📝 Description</h4>
                            </div>
                            <div className="editor-container">
                                <EditorContent editor={editor} />
                            </div>
                        </div>
                        
                        {/* Checklists */}
                        {card.checklists.map((checklist, idx) => (
                            <div key={checklist._id} className="section">
                                <div className="section-header">
                                    <h4>✓ {checklist.title}</h4>
                                    <span className="checklist-progress">
                                        {checklist.items.filter(i => i.completed).length}/{checklist.items.length}
                                    </span>
                                </div>
                                
                                <div className="progress-bar-container">
                                    <div 
                                        className="progress-bar-fill"
                                        style={{
                                            width: `${(checklist.items.filter(i => i.completed).length / checklist.items.length) * 100}%`
                                        }}
                                    />
                                </div>
                                
                                <div className="checklist-items">
                                    {checklist.items.map(item => (
                                        <div key={item._id} className="checklist-item">
                                            <input
                                                type="checkbox"
                                                checked={item.completed}
                                                onChange={() => toggleChecklistItem(checklist._id, item._id)}
                                            />
                                            <span className={item.completed ? 'completed' : ''}>
                                                {item.name}
                                            </span>
                                        </div>
                                    ))}
                                </div>
                            </div>
                        ))}
                        
                        {/* Attachments */}
                        {card.attachments.length > 0 && (
                            <div className="section">
                                <h4>📎 Attachments</h4>
                                <div className="attachments-list">
                                    {card.attachments.map(attachment => (
                                        <div key={attachment._id} className="attachment-item">
                                            {attachment.mimeType.startsWith('image/') ? (
                                                <img src={attachment.url} alt={attachment.name} />
                                            ) : (
                                                <div className="attachment-icon">📄</div>
                                            )}
                                            <div className="attachment-info">
                                                <div className="attachment-name">{attachment.name}</div>
                                                <div className="attachment-meta">
                                                    {formatFileSize(attachment.size)} • 
                                                    {format(new Date(attachment.uploadedAt), 'MMM d')}
                                                </div>
                                            </div>
                                        </div>
                                    ))}
                                </div>
                            </div>
                        )}
                        
                        {/* Activity/Comments */}
                        <div className="section">
                            <h4>💬 Activity</h4>
                            
                            <div className="comment-form">
                                <Avatar user={currentUser} />
                                <textarea
                                    value={newComment}
                                    onChange={(e) => setNewComment(e.target.value)}
                                    placeholder="Write a comment..."
                                    rows={3}
                                />
                                <button onClick={addComment} className="btn-primary">
                                    Save
                                </button>
                            </div>
                            
                            <div className="comments-list">
                                {comments.map(comment => (
                                    <div key={comment._id} className="comment-item">
                                        <Avatar user={comment.user} />
                                        <div className="comment-content">
                                            <div className="comment-header">
                                                <strong>{comment.user.username}</strong>
                                                <span className="comment-date">
                                                    {format(new Date(comment.createdAt), 'MMM d \'at\' h:mm a')}
                                                </span>
                                            </div>
                                            <div className="comment-text">{comment.text}</div>
                                        </div>
                                    </div>
                                ))}
                            </div>
                        </div>
                    </div>
                    
                    {/* Sidebar */}
                    <div className="modal-sidebar">
                        <h4>Add to card</h4>
                        <button className="sidebar-btn" onClick={() => openMemberSelector()}>
                            <span className="icon">👤</span>
                            Members
                        </button>
                        <button className="sidebar-btn" onClick={() => openLabelSelector()}>
                            <span className="icon">🏷️</span>
                            Labels
                        </button>
                        <button className="sidebar-btn" onClick={() => openChecklistCreator()}>
                            <span className="icon">✓</span>
                            Checklist
                        </button>
                        <button className="sidebar-btn" onClick={() => openDatePicker()}>
                            <span className="icon">📅</span>
                            Dates
                        </button>
                        <button className="sidebar-btn" onClick={() => openAttachmentUploader()}>
                            <span className="icon">📎</span>
                            Attachment
                        </button>
                        <button className="sidebar-btn" onClick={() => openCoverSelector()}>
                            <span className="icon">🖼️</span>
                            Cover
                        </button>
                        <button className="sidebar-btn" onClick={() => openCustomFields()}>
                            <span className="icon">⚙️</span>
                            Custom Fields
                        </button>
                        
                        <hr />
                        
                        <h4>Actions</h4>
                        <button className="sidebar-btn" onClick={() => moveCard()}>
                            <span className="icon">➡️</span>
                            Move
                        </button>
                        <button className="sidebar-btn" onClick={() => copyCard()}>
                            <span className="icon">📋</span>
                            Copy
                        </button>
                        <button className="sidebar-btn" onClick={() => makeTemplate()}>
                            <span className="icon">📄</span>
                            Make Template
                        </button>
                        <button className="sidebar-btn" onClick={() => archiveCard()}>
                            <span className="icon">📦</span>
                            Archive
                        </button>
                        <button className="sidebar-btn danger" onClick={() => deleteCard()}>
                            <span className="icon">🗑️</span>
                            Delete
                        </button>
                    </div>
                </div>
            </div>
        </div>
    );
}

export default CardModal;
```

## Technology Stack

### Frontend

- **React/Vue.js** - UI framework
- **@dnd-kit/core** - Modern drag-and-drop library
- **Socket.io-client** - Real-time communication
- **TipTap** or **Quill** - Rich text editor
- **Zustand/Redux/Pinia** - State management
- **React Query** - Server state management
- **Tailwind CSS** - Styling
- **date-fns** - Date formatting

### Backend

- **Node.js + Express** - API server
- **Socket.io** - WebSocket server
- **MongoDB** with Mongoose - Database
- **AWS S3** or **Cloudinary** - File storage
- **Redis** - Caching and pub/sub for scaling WebSockets
- **JWT** - Authentication
- **Bull** - Job queue for async tasks

### Infrastructure

- **Docker** - Containerization
- **Nginx** - Reverse proxy
- **Redis** - Real-time features and caching
- **Elasticsearch** - Advanced search (optional)
- **AWS/DigitalOcean** - Hosting

## Testing Checklist

- [ ] Drag and drop works smoothly across browsers
- [ ] Cards move between lists correctly
- [ ] Lists can be reordered
- [ ] Real-time updates appear for all connected users
- [ ] No conflicts when multiple users edit simultaneously
- [ ] Optimistic updates work and rollback on error
- [ ] Permissions are enforced (viewer can't edit, etc.)
- [ ] Activity log captures all changes accurately
- [ ] File uploads work correctly
- [ ] Search returns relevant results
- [ ] Mobile responsive design works
- [ ] Offline detection and graceful handling
- [ ] WebSocket reconnection works seamlessly

## Additional Challenges

### Easy Extensions

- Add board backgrounds (images, gradients, colors)
- Implement keyboard shortcuts (N for new card, etc.)
- Create card templates
- Add board templates
- Implement dark mode
- Add card numbering
- Create printable board views

### Medium Extensions

- Add custom fields to cards
- Implement board-level automation rules (Butler-style)
- Create card dependencies (blocking/blocked by)
- Add time tracking per card
- Build calendar view
- Implement board analytics dashboard
- Add Gantt chart view
- Create card voting system
- Build advanced filtering (multiple criteria)
- Add @ mentions in comments
- Implement emoji reactions

### Hard Extensions

- Implement **Operational Transformation** for conflict-free collaborative editing
- Build offline-first functionality with sync queue
- Add video call integration (Jitly/Zoom)
- Create mind map view
- Implement AI-powered task suggestions
- Build custom power-ups/plugins system
- Add advanced automation with triggers, actions, and conditions
- Create mobile apps (React Native/Flutter)
- Implement end-to-end encryption for sensitive boards
- Build API for third-party integrations
- Add version control for cards (git-like)

## Common Pitfalls

1. **Drag Performance:** For boards with 100+ cards, optimize with React.memo, virtualization, or pagination. Use `transform` CSS instead of position changes.

2. **Race Conditions:** Implement optimistic updates with rollback. Use timestamps or vector clocks to resolve conflicts.

3. **Position Conflicts:** Use fractional indexing (e.g., 1.5 between 1 and 2) instead of sequential integers to avoid frequent re-indexing.

4. **WebSocket Reconnection:** Implement exponential backoff and state reconciliation. Send sequence numbers with updates.

5. **Memory Leaks:** Clean up WebSocket event listeners on component unmount. Use WeakMap for storing temporary data.

6. **Touch Devices:** @dnd-kit supports touch, but test thoroughly on mobile. Consider using long-press to initiate drag.

7. **Large Boards:** Implement virtual scrolling for lists with many cards. Lazy-load card details.

## Resources

### Documentation

- [@dnd-kit Documentation](https://docs.dndkit.com/)
- [Socket.io Documentation](https://socket.io/docs/)
- [TipTap Editor](https://tiptap.dev/)
- [MongoDB Best Practices](https://www.mongodb.com/docs/manual/administration/production-notes/)

### Tutorials

- [Building Real-Time Collaboration](https://www.youtube.com/results?search_query=real+time+collaboration+websocket)
- [Trello Clone Tutorial](https://www.youtube.com/results?search_query=trello+clone+react)
- [Drag and Drop with dnd-kit](https://www.youtube.com/results?search_query=dnd-kit+tutorial)

### Design Inspiration

- [Trello](https://trello.com/)
- [Asana](https://asana.com/)
- [Monday.com](https://monday.com/)
- [Notion](https://notion.so/)
- [ClickUp](https://clickup.com/)

### Architecture Patterns

- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Operational Transformation](https://operational-transformation.github.io/)
- [CRDTs](https://crdt.tech/)

## Submission Checklist

- [ ] Full drag-and-drop functionality (cards and lists)
- [ ] Real-time collaboration with WebSockets
- [ ] User authentication and authorization
- [ ] Complete RBAC (role-based access control)
- [ ] Card details with rich editing
- [ ] Comments system with threading
- [ ] File attachments with preview
- [ ] Checklists with progress tracking
- [ ] Labels and due dates
- [ ] Activity logging
- [ ] Search functionality
- [ ] Responsive mobile design
- [ ] Error handling and loading states
- [ ] Comprehensive documentation
- [ ] Unit and integration tests

## Portfolio Tips

- **Show Collaboration:** Create a demo video with multiple browser windows editing the same board simultaneously
- **Highlight Performance:** Mention optimizations like virtualization, debouncing, memoization
- **Explain Architecture:** Include system diagram showing WebSocket flow, database schema, state management
- **Compare to Industry:** "Built with similar architecture to Trello/Asana, supporting real-time collaboration for distributed teams"
- **Showcase Features:** Screenshots/GIFs of drag-and-drop, real-time updates, rich editing, mobile responsiveness
- **Metrics:** "Supports X concurrent users, Y boards, handles Z operations/second"
- **Technical Depth:** Explain operational transformation, conflict resolution, fractional indexing choices
- **Live Demo:** Deploy and include link + test credentials

---

This project is part of the [Coding Projects Library](https://github.com/Ryanditko/biblioteca-de-projetos) and is licensed under the MIT License.
