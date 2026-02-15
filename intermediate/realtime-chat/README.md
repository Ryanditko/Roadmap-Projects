# Real-time Chat Application

<p align="center">
  <img src="https://images.unsplash.com/photo-1611746869696-d09bce200020?w=600&q=80" alt="Real-time Chat" width="600">
</p>

**Level:** Intermediate  
**Estimated Time:** 2-3 weeks  
**Prerequisites:** WebSockets, real-time communication, authentication, database design

---

## Description

Build a real-time chat application with instant messaging, group chats, file sharing, and online status indicators. This project teaches you about WebSocket communication, real-time data synchronization, message persistence, and building responsive chat interfaces.

---

## Core Features

### Must Have

- User registration and authentication
- One-on-one messaging
- Real-time message delivery
- Message history persistence
- Online/offline status indicators
- Typing indicators
- Message timestamps
- Emoji support
- Responsive chat UI

### Nice to Have

- Group chats/channels
- File and image sharing
- Message read receipts
- Push notifications
- Voice messages
- Video calls (WebRTC)
- Message search
- User profiles
- Dark mode
- Message reactions
- Thread replies
- @mentions

---

## Learning Objectives

- **WebSocket Communication**: Real-time bidirectional communication
- **Socket.io**: Event-based messaging
- **Message Persistence**: Storing chat history
- **Real-time Updates**: Synchronizing state across clients
- **Authentication**: Securing WebSocket connections
- **File Upload**: Handling media in chat
- **Presence System**: Tracking online users
- **Scalability**: Handling multiple concurrent connections

---

## Technology Stack

### Backend

- **Node.js + Express**: Server framework
- **Socket.io**: WebSocket library
- **MongoDB**: Message storage
- **Redis**: Presence and session management
- **JWT**: Authentication
- **Multer**: File uploads

### Frontend

- **React**: UI library
- **Socket.io-client**: WebSocket client
- **Tailwind CSS**: Styling
- **React Query**: Data management

---

## Implementation Guide

### Step 1: Set Up WebSocket Server

```javascript
// server.js
const express = require('express');
const http = require('http');
const socketio = require('socket.io');
const jwt = require('jsonwebtoken');

const app = express();
const server = http.createServer(app);
const io = socketio(server, {
  cors: {
    origin: process.env.CLIENT_URL,
    credentials: true
  }
});

// Socket authentication middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication error'));
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.userId = decoded.id;
    next();
  } catch (error) {
    next(new Error('Authentication error'));
  }
});

// Connection handler
io.on('connection', (socket) => {
  console.log(`User connected: ${socket.userId}`);
  
  // Join user's personal room
  socket.join(socket.userId);
  
  // Handle private message
  socket.on('private_message', async (data) => {
    const { recipientId, message } = data;
    
    // Save message to database
    const savedMessage = await Message.create({
      sender: socket.userId,
      recipient: recipientId,
      content: message,
      timestamp: new Date()
    });
    
    // Send to recipient
    io.to(recipientId).emit('new_message', savedMessage);
    
    // Send confirmation to sender
    socket.emit('message_sent', savedMessage);
  });
  
  // Handle typing indicator
  socket.on('typing', (recipientId) => {
    io.to(recipientId).emit('user_typing', socket.userId);
  });
  
  socket.on('stop_typing', (recipientId) => {
    io.to(recipientId).emit('user_stop_typing', socket.userId);
  });
  
  // Handle disconnect
  socket.on('disconnect', () => {
    console.log(`User disconnected: ${socket.userId}`);
  });
});

server.listen(5000, () => {
  console.log('Server running on port 5000');
});
```

### Step 2: Create Chat Interface

```jsx
// components/ChatWindow.jsx
import React, { useState, useEffect, useRef } from 'react';
import { Send, Paperclip, Smile } from 'lucide-react';
import { useSocket } from '../context/SocketContext';

const ChatWindow = ({ recipientId, recipientName }) => {
  const [messages, setMessages] = useState([]);
  const [inputMessage, setInputMessage] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const messagesEndRef = useRef(null);
  const socket = useSocket();
  
  useEffect(() => {
    // Load message history
    loadMessageHistory();
    
    // Listen for new messages
    socket.on('new_message', handleNewMessage);
    socket.on('user_typing', () => setIsTyping(true));
    socket.on('user_stop_typing', () => setIsTyping(false));
    
    return () => {
      socket.off('new_message', handleNewMessage);
      socket.off('user_typing');
      socket.off('user_stop_typing');
    };
  }, [recipientId]);
  
  useEffect(() => {
    scrollToBottom();
  }, [messages]);
  
  const loadMessageHistory = async () => {
    // Fetch from API
    const response = await fetch(`/api/messages/${recipientId}`);
    const data = await response.json();
    setMessages(data.messages);
  };
  
  const handleNewMessage = (message) => {
    setMessages(prev => [...prev, message]);
  };
  
  const sendMessage = () => {
    if (!inputMessage.trim()) return;
    
    socket.emit('private_message', {
      recipientId,
      message: inputMessage
    });
    
    setInputMessage('');
    socket.emit('stop_typing', recipientId);
  };
  
  const handleInputChange = (e) => {
    setInputMessage(e.target.value);
    
    if (e.target.value) {
      socket.emit('typing', recipientId);
    } else {
      socket.emit('stop_typing', recipientId);
    }
  };
  
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  return (
    <div className="flex flex-col h-full">
      {/* Chat Header */}
      <div className="bg-white border-b p-4 flex items-center">
        <div className="flex-1">
          <h2 className="font-semibold">{recipientName}</h2>
          {isTyping && (
            <p className="text-sm text-gray-500">typing...</p>
          )}
        </div>
      </div>
      
      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((message, index) => (
          <div
            key={index}
            className={`flex ${
              message.sender === socket.userId ? 'justify-end' : 'justify-start'
            }`}
          >
            <div
              className={`max-w-xs lg:max-w-md px-4 py-2 rounded-lg ${
                message.sender === socket.userId
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-200 text-gray-900'
              }`}
            >
              <p>{message.content}</p>
              <p className="text-xs mt-1 opacity-70">
                {new Date(message.timestamp).toLocaleTimeString()}
              </p>
            </div>
          </div>
        ))}
        <div ref={messagesEndRef} />
      </div>
      
      {/* Input */}
      <div className="bg-white border-t p-4">
        <div className="flex items-center gap-2">
          <button className="p-2 hover:bg-gray-100 rounded">
            <Paperclip size={20} />
          </button>
          
          <input
            type="text"
            value={inputMessage}
            onChange={handleInputChange}
            onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
            placeholder="Type a message..."
            className="flex-1 px-4 py-2 border rounded-full focus:outline-none focus:border-blue-500"
          />
          
          <button className="p-2 hover:bg-gray-100 rounded">
            <Smile size={20} />
          </button>
          
          <button
            onClick={sendMessage}
            className="bg-blue-600 text-white p-2 rounded-full hover:bg-blue-700"
          >
            <Send size={20} />
          </button>
        </div>
      </div>
    </div>
  );
};

export default ChatWindow;
```

---

## Testing Checklist

- [ ] Messages delivered in real-time
- [ ] Message history loaded correctly
- [ ] Typing indicators working
- [ ] Online status accurate
- [ ] File uploads functional
- [ ] Group chats working
- [ ] Authentication secure
- [ ] Mobile responsive
- [ ] Notifications working

---

## Additional Challenges

### Easy

- Add emoji picker
- Implement message search
- Add user profiles
- Create dark mode

### Medium

- Build group chat functionality
- Add file sharing
- Implement voice messages
- Create read receipts
- Add message reactions

### Hard

- Implement video calling (WebRTC)
- Build end-to-end encryption
- Create message threading
- Add voice/video rooms
- Implement message translation

---

## Resources

- [Socket.io Documentation](https://socket.io/docs/)
- [WebRTC Guide](https://webrtc.org/)
- [Building Real-time Apps](https://www.youtube.com/results?search_query=socket.io+chat+tutorial)

---

## License

This project idea is part of the [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
