# AI-Powered Chatbot Assistant

<p align="center">
  <img src="https://images.unsplash.com/photo-1531746790731-6c087fecd65a?w=600&q=80" alt="AI Chatbot" width="600">
</p>

**Level:** Advanced  
**Estimated Time:** 3-4 weeks  
**Prerequisites:** Python, Machine Learning basics, API integration, NLP fundamentals

---

## Description

Build an intelligent chatbot assistant that can understand natural language, respond to user queries, learn from interactions, and integrate with various messaging platforms. This advanced project combines Natural Language Processing (NLP), machine learning, API integration, and conversation management to create a truly interactive AI assistant.

The chatbot will use state-of-the-art NLP models to understand user intent, extract relevant entities, maintain conversation context, and provide intelligent responses. It can be integrated with external APIs for real-time data, support voice commands, and connect to popular messaging platforms like Telegram, WhatsApp, or Discord.

---

## Core Features

### Must Have
- Natural Language Understanding (NLU) with intent recognition
- Entity extraction from user messages
- Context-aware conversation management
- Integration with at least one messaging platform (Telegram/Discord)
- Response generation based on intents
- Fallback handling for unknown queries
- Basic sentiment analysis
- Conversation history storage
- Training data management

### Nice to Have
- Multi-language support
- Voice command support (speech-to-text)
- Custom voice responses (text-to-speech)
- Integration with external APIs (weather, news, etc.)
- Learning from user feedback
- Customizable personality and tone
- Rich media responses (images, cards, buttons)
- User authentication and personalization
- Conversation analytics dashboard
- A/B testing for responses

---

## Learning Objectives

By building this project, you will learn:

- **Natural Language Processing**: Understanding text analysis and language models
- **Intent Classification**: Training ML models to recognize user intentions
- **Entity Recognition**: Extracting key information from text
- **Dialogue Management**: Maintaining conversation state and context
- **Transformer Models**: Working with BERT, GPT, or similar architectures
- **API Integration**: Connecting external services for real-time data
- **Messaging Platform APIs**: Telegram Bot API, Discord.py, WhatsApp Business API
- **Machine Learning Pipeline**: Training, evaluating, and deploying ML models
- **Context Management**: Tracking conversation history and user preferences

---

## Implementation Guide

### Step 1: Set Up the Project Structure

Create a well-organized project structure:

```
ai-chatbot/
├── src/
│   ├── nlp/
│   │   ├── intent_classifier.py
│   │   ├── entity_extractor.py
│   │   └── sentiment_analyzer.py
│   ├── dialogue/
│   │   ├── context_manager.py
│   │   └── response_generator.py
│   ├── integrations/
│   │   ├── telegram_bot.py
│   │   ├── discord_bot.py
│   │   └── api_clients.py
│   ├── models/
│   │   └── trained_models/
│   └── utils/
│       ├── config.py
│       └── database.py
├── data/
│   ├── training/
│   │   ├── intents.json
│   │   └── entities.json
│   └── conversations/
├── tests/
├── requirements.txt
├── .env
└── main.py
```

### Step 2: Choose Your NLP Framework

**Option A: Rasa (Recommended for beginners)**
```python
# Install Rasa
pip install rasa

# Initialize Rasa project
rasa init
```

**Option B: Custom with Transformers**
```python
from transformers import pipeline, AutoTokenizer, AutoModelForSequenceClassification

# Load pre-trained model
classifier = pipeline("sentiment-analysis")
```

**Option C: spaCy + Custom Models**
```python
import spacy
nlp = spacy.load("en_core_web_sm")
```

### Step 3: Implement Intent Classification

Create a system to understand user intentions:

```python
import json
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
import pickle

class IntentClassifier:
    def __init__(self):
        self.vectorizer = TfidfVectorizer()
        self.classifier = MultinomialNB()
        self.intents = {}
        
    def load_training_data(self, filepath):
        """Load intents from JSON file"""
        with open(filepath, 'r') as f:
            data = json.load(f)
            self.intents = data['intents']
        
        # Prepare training data
        texts = []
        labels = []
        for intent in self.intents:
            for pattern in intent['patterns']:
                texts.append(pattern.lower())
                labels.append(intent['tag'])
        
        return texts, labels
    
    def train(self, texts, labels):
        """Train the intent classifier"""
        X = self.vectorizer.fit_transform(texts)
        self.classifier.fit(X, labels)
        
    def predict_intent(self, user_input):
        """Predict intent from user input"""
        X = self.vectorizer.transform([user_input.lower()])
        intent = self.classifier.predict(X)[0]
        confidence = max(self.classifier.predict_proba(X)[0])
        return intent, confidence
    
    def save_model(self, filepath):
        """Save trained model"""
        with open(filepath, 'wb') as f:
            pickle.dump({
                'vectorizer': self.vectorizer,
                'classifier': self.classifier,
                'intents': self.intents
            }, f)
```

### Step 4: Define Training Data Structure

Create `data/training/intents.json`:

```json
{
  "intents": [
    {
      "tag": "greeting",
      "patterns": [
        "Hi", "Hello", "Hey", "Good morning",
        "Good afternoon", "What's up", "Howdy"
      ],
      "responses": [
        "Hello! How can I help you today?",
        "Hi there! What can I do for you?",
        "Hey! How may I assist you?"
      ]
    },
    {
      "tag": "goodbye",
      "patterns": [
        "Bye", "Goodbye", "See you", "Talk to you later",
        "Have a nice day", "Catch you later"
      ],
      "responses": [
        "Goodbye! Have a great day!",
        "See you later!",
        "Bye! Feel free to come back anytime."
      ]
    },
    {
      "tag": "weather",
      "patterns": [
        "What's the weather like?",
        "How's the weather today?",
        "Is it raining?",
        "Will it rain today?",
        "Weather forecast"
      ],
      "responses": [
        "Let me check the weather for you.",
        "I'll get the current weather information."
      ],
      "context": ["weather_api"]
    },
    {
      "tag": "help",
      "patterns": [
        "Help", "I need help", "Can you help me?",
        "What can you do?", "How do you work?"
      ],
      "responses": [
        "I can help you with weather information, answer questions, and have conversations. What do you need?",
        "I'm here to assist! I can provide information, answer questions, and chat with you."
      ]
    }
  ]
}
```

### Step 5: Implement Context Management

```python
from datetime import datetime
from collections import defaultdict

class ContextManager:
    def __init__(self):
        self.contexts = defaultdict(dict)
        self.conversation_history = defaultdict(list)
    
    def set_context(self, user_id, key, value):
        """Store context for a user"""
        self.contexts[user_id][key] = {
            'value': value,
            'timestamp': datetime.now()
        }
    
    def get_context(self, user_id, key):
        """Retrieve context for a user"""
        if key in self.contexts[user_id]:
            return self.contexts[user_id][key]['value']
        return None
    
    def add_to_history(self, user_id, message, is_user=True):
        """Add message to conversation history"""
        self.conversation_history[user_id].append({
            'message': message,
            'is_user': is_user,
            'timestamp': datetime.now()
        })
    
    def get_history(self, user_id, limit=10):
        """Get recent conversation history"""
        return self.conversation_history[user_id][-limit:]
    
    def clear_context(self, user_id):
        """Clear context for a user"""
        if user_id in self.contexts:
            del self.contexts[user_id]
```

### Step 6: Create Response Generator

```python
import random
import requests

class ResponseGenerator:
    def __init__(self, intent_classifier, context_manager):
        self.intent_classifier = intent_classifier
        self.context_manager = context_manager
    
    def generate_response(self, user_id, user_input):
        """Generate appropriate response based on intent"""
        # Classify intent
        intent, confidence = self.intent_classifier.predict_intent(user_input)
        
        # Store in history
        self.context_manager.add_to_history(user_id, user_input, is_user=True)
        
        # Handle low confidence
        if confidence < 0.5:
            response = "I'm not sure I understand. Could you rephrase that?"
        else:
            # Find intent data
            intent_data = next(
                (i for i in self.intent_classifier.intents if i['tag'] == intent),
                None
            )
            
            if intent_data:
                # Check if requires API call
                if 'context' in intent_data and 'weather_api' in intent_data['context']:
                    response = self.get_weather_response(user_id)
                else:
                    response = random.choice(intent_data['responses'])
            else:
                response = "I'm not sure how to respond to that."
        
        # Store response in history
        self.context_manager.add_to_history(user_id, response, is_user=False)
        
        return response
    
    def get_weather_response(self, user_id):
        """Get weather information from API"""
        location = self.context_manager.get_context(user_id, 'location')
        if not location:
            location = "New York"  # Default
        
        try:
            # Example with OpenWeatherMap API
            api_key = "YOUR_API_KEY"
            url = f"http://api.openweathermap.org/data/2.5/weather?q={location}&appid={api_key}&units=metric"
            response = requests.get(url)
            data = response.json()
            
            if response.status_code == 200:
                temp = data['main']['temp']
                description = data['weather'][0]['description']
                return f"The weather in {location} is {description} with a temperature of {temp}°C."
            else:
                return "Sorry, I couldn't fetch the weather information right now."
        except Exception as e:
            return "An error occurred while fetching weather data."
```

### Step 7: Integrate with Telegram

```python
import os
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

class TelegramBot:
    def __init__(self, token, response_generator):
        self.token = token
        self.response_generator = response_generator
        self.app = Application.builder().token(token).build()
        self.setup_handlers()
    
    def setup_handlers(self):
        """Setup command and message handlers"""
        self.app.add_handler(CommandHandler("start", self.start_command))
        self.app.add_handler(CommandHandler("help", self.help_command))
        self.app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, self.handle_message))
    
    async def start_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Handle /start command"""
        await update.message.reply_text(
            "Hello! I'm your AI assistant. How can I help you today?"
        )
    
    async def help_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Handle /help command"""
        await update.message.reply_text(
            "I can help you with:\n"
            "- Weather information\n"
            "- General questions\n"
            "- Conversations\n\n"
            "Just type your message and I'll respond!"
        )
    
    async def handle_message(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Handle incoming messages"""
        user_id = update.effective_user.id
        user_input = update.message.text
        
        # Generate response
        response = self.response_generator.generate_response(str(user_id), user_input)
        
        # Send response
        await update.message.reply_text(response)
    
    def run(self):
        """Start the bot"""
        print("Bot is running...")
        self.app.run_polling()
```

### Step 8: Implement Advanced Entity Extraction

```python
import spacy
from typing import Dict, List

class EntityExtractor:
    def __init__(self):
        self.nlp = spacy.load("en_core_web_sm")
        self.custom_entities = {}
    
    def extract_entities(self, text: str) -> Dict[str, List[str]]:
        """Extract named entities from text"""
        doc = self.nlp(text)
        
        entities = {
            'PERSON': [],
            'ORG': [],
            'GPE': [],  # Countries, cities, states
            'DATE': [],
            'TIME': [],
            'MONEY': [],
            'QUANTITY': []
        }
        
        for ent in doc.ents:
            if ent.label_ in entities:
                entities[ent.label_].append(ent.text)
        
        # Extract custom entities
        custom = self.extract_custom_entities(text)
        entities.update(custom)
        
        return entities
    
    def extract_custom_entities(self, text: str) -> Dict[str, List[str]]:
        """Extract domain-specific entities"""
        import re
        
        entities = {}
        
        # Email extraction
        emails = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', text)
        if emails:
            entities['EMAIL'] = emails
        
        # Phone number extraction
        phones = re.findall(r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b', text)
        if phones:
            entities['PHONE'] = phones
        
        # URL extraction
        urls = re.findall(r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\\(\\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+', text)
        if urls:
            entities['URL'] = urls
        
        return entities
    
    def add_custom_pattern(self, entity_type: str, patterns: List[str]):
        """Add custom entity patterns"""
        from spacy.matcher import Matcher
        
        matcher = Matcher(self.nlp.vocab)
        for pattern in patterns:
            matcher.add(entity_type, [[{"LOWER": word.lower()} for word in pattern.split()]])
        
        self.custom_entities[entity_type] = matcher
```

### Step 9: Implement Sentiment Analysis

```python
from transformers import pipeline
from typing import Dict

class SentimentAnalyzer:
    def __init__(self):
        self.classifier = pipeline(
            "sentiment-analysis",
            model="distilbert-base-uncased-finetuned-sst-2-english"
        )
    
    def analyze(self, text: str) -> Dict[str, float]:
        """Analyze sentiment of text"""
        result = self.classifier(text)[0]
        
        return {
            'label': result['label'],
            'score': result['score'],
            'is_positive': result['label'] == 'POSITIVE',
            'is_negative': result['label'] == 'NEGATIVE'
        }
    
    def get_emotion(self, text: str) -> str:
        """Get emotion from text (requires emotion model)"""
        emotion_classifier = pipeline(
            "text-classification",
            model="j-hartmann/emotion-english-distilroberta-base",
            top_k=None
        )
        
        emotions = emotion_classifier(text)[0]
        # Returns: anger, disgust, fear, joy, neutral, sadness, surprise
        
        top_emotion = max(emotions, key=lambda x: x['score'])
        return top_emotion['label']
```

### Step 10: Create Advanced Dialogue State Tracking

```python
from enum import Enum
from typing import Optional, Dict, Any
from datetime import datetime

class DialogueState(Enum):
    GREETING = "greeting"
    INFORMATION_GATHERING = "information_gathering"
    TASK_EXECUTION = "task_execution"
    CONFIRMATION = "confirmation"
    FAREWELL = "farewell"

class DialogueStateTracker:
    def __init__(self):
        self.current_states = {}  # user_id -> state
        self.slots = {}  # user_id -> slot_values
        self.required_slots = {
            'weather': ['location'],
            'booking': ['date', 'time', 'service'],
            'order': ['product', 'quantity', 'address']
        }
    
    def update_state(self, user_id: str, intent: str, entities: Dict[str, Any]):
        """Update dialogue state based on intent and entities"""
        if intent == 'greeting':
            self.current_states[user_id] = DialogueState.GREETING
            self.slots[user_id] = {}
        
        elif intent in self.required_slots:
            # Check if we need to gather information
            if user_id not in self.slots:
                self.slots[user_id] = {}
            
            # Update slots with extracted entities
            self.update_slots(user_id, intent, entities)
            
            # Check if all required slots are filled
            if self.all_slots_filled(user_id, intent):
                self.current_states[user_id] = DialogueState.TASK_EXECUTION
            else:
                self.current_states[user_id] = DialogueState.INFORMATION_GATHERING
        
        elif intent == 'confirm':
            self.current_states[user_id] = DialogueState.TASK_EXECUTION
        
        elif intent == 'goodbye':
            self.current_states[user_id] = DialogueState.FAREWELL
            self.clear_slots(user_id)
    
    def update_slots(self, user_id: str, intent: str, entities: Dict[str, Any]):
        """Update slot values from entities"""
        if intent not in self.required_slots:
            return
        
        required = self.required_slots[intent]
        
        for slot in required:
            if slot in entities and entities[slot]:
                self.slots[user_id][slot] = entities[slot][0]
    
    def all_slots_filled(self, user_id: str, intent: str) -> bool:
        """Check if all required slots are filled"""
        if intent not in self.required_slots:
            return True
        
        required = self.required_slots[intent]
        user_slots = self.slots.get(user_id, {})
        
        return all(slot in user_slots for slot in required)
    
    def get_missing_slots(self, user_id: str, intent: str) -> list:
        """Get list of missing slots"""
        if intent not in self.required_slots:
            return []
        
        required = self.required_slots[intent]
        user_slots = self.slots.get(user_id, {})
        
        return [slot for slot in required if slot not in user_slots]
    
    def clear_slots(self, user_id: str):
        """Clear all slots for a user"""
        if user_id in self.slots:
            del self.slots[user_id]
    
    def get_state(self, user_id: str) -> Optional[DialogueState]:
        """Get current dialogue state for user"""
        return self.current_states.get(user_id)
```

### Step 11: Implement Learning from Feedback

```python
import json
from datetime import datetime
from typing import List, Dict

class FeedbackLearner:
    def __init__(self, feedback_file='data/feedback.json'):
        self.feedback_file = feedback_file
        self.feedback_data = self.load_feedback()
    
    def load_feedback(self) -> List[Dict]:
        """Load existing feedback data"""
        try:
            with open(self.feedback_file, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return []
    
    def collect_feedback(self, user_id: str, message: str, response: str, 
                        rating: int, comment: str = ""):
        """Collect user feedback on bot response"""
        feedback = {
            'user_id': user_id,
            'message': message,
            'response': response,
            'rating': rating,  # 1-5 scale
            'comment': comment,
            'timestamp': datetime.now().isoformat()
        }
        
        self.feedback_data.append(feedback)
        self.save_feedback()
    
    def save_feedback(self):
        """Save feedback data to file"""
        with open(self.feedback_file, 'w') as f:
            json.dump(self.feedback_data, f, indent=2)
    
    def get_low_rated_responses(self, threshold: int = 3) -> List[Dict]:
        """Get responses with low ratings for improvement"""
        return [f for f in self.feedback_data if f['rating'] <= threshold]
    
    def analyze_feedback(self) -> Dict[str, float]:
        """Analyze feedback metrics"""
        if not self.feedback_data:
            return {}
        
        ratings = [f['rating'] for f in self.feedback_data]
        
        return {
            'average_rating': sum(ratings) / len(ratings),
            'total_feedback': len(self.feedback_data),
            'positive_feedback': len([r for r in ratings if r >= 4]),
            'negative_feedback': len([r for r in ratings if r <= 2]),
            'neutral_feedback': len([r for r in ratings if r == 3])
        }
    
    def retrain_with_feedback(self, intent_classifier):
        """Retrain model with feedback data"""
        # Collect low-rated examples as potential improvements
        low_rated = self.get_low_rated_responses()
        
        # This would require additional implementation to:
        # 1. Identify patterns in low-rated responses
        # 2. Generate new training examples
        # 3. Retrain the model with augmented data
        
        print(f"Found {len(low_rated)} responses that need improvement")
        # Implementation details depend on your ML pipeline
```

### Step 12: Build REST API with FastAPI

```python
from fastapi import FastAPI, HTTPException, BackgroundTasks
from pydantic import BaseModel
from typing import Optional, List
import uvicorn

app = FastAPI(title="AI Chatbot API", version="1.0.0")

# Request/Response models
class ChatRequest(BaseModel):
    user_id: str
    message: str
    context: Optional[dict] = None

class ChatResponse(BaseModel):
    response: str
    intent: str
    confidence: float
    entities: dict
    sentiment: dict

class FeedbackRequest(BaseModel):
    user_id: str
    message: str
    response: str
    rating: int
    comment: Optional[str] = ""

# Initialize bot components (same as before)
intent_classifier = IntentClassifier()
context_manager = ContextManager()
response_generator = ResponseGenerator(intent_classifier, context_manager)
entity_extractor = EntityExtractor()
sentiment_analyzer = SentimentAnalyzer()
feedback_learner = FeedbackLearner()

@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    """Process chat message and return response"""
    try:
        # Extract entities
        entities = entity_extractor.extract_entities(request.message)
        
        # Analyze sentiment
        sentiment = sentiment_analyzer.analyze(request.message)
        
        # Classify intent
        intent, confidence = intent_classifier.predict_intent(request.message)
        
        # Generate response
        response = response_generator.generate_response(
            request.user_id,
            request.message
        )
        
        return ChatResponse(
            response=response,
            intent=intent,
            confidence=confidence,
            entities=entities,
            sentiment=sentiment
        )
    
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/feedback")
async def submit_feedback(request: FeedbackRequest, background_tasks: BackgroundTasks):
    """Submit feedback on bot response"""
    feedback_learner.collect_feedback(
        request.user_id,
        request.message,
        request.response,
        request.rating,
        request.comment
    )
    
    # Optionally retrain model in background
    # background_tasks.add_task(feedback_learner.retrain_with_feedback, intent_classifier)
    
    return {"status": "success", "message": "Feedback recorded"}

@app.get("/analytics")
async def get_analytics():
    """Get chatbot analytics"""
    return feedback_learner.analyze_feedback()

@app.get("/history/{user_id}")
async def get_conversation_history(user_id: str, limit: int = 10):
    """Get conversation history for user"""
    history = context_manager.get_history(user_id, limit)
    return {"user_id": user_id, "history": history}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Step 13: Add Multi-Language Support

```python
from googletrans import Translator
from typing import Dict

class MultiLanguageSupport:
    def __init__(self):
        self.translator = Translator()
        self.supported_languages = {
            'en': 'English',
            'es': 'Spanish',
            'fr': 'French',
            'de': 'German',
            'pt': 'Portuguese',
            'it': 'Italian',
            'ru': 'Russian',
            'ja': 'Japanese',
            'zh-cn': 'Chinese (Simplified)'
        }
    
    def detect_language(self, text: str) -> str:
        """Detect language of text"""
        detection = self.translator.detect(text)
        return detection.lang
    
    def translate_to_english(self, text: str, source_lang: str = None) -> str:
        """Translate text to English for processing"""
        if source_lang is None:
            source_lang = self.detect_language(text)
        
        if source_lang == 'en':
            return text
        
        translation = self.translator.translate(text, src=source_lang, dest='en')
        return translation.text
    
    def translate_from_english(self, text: str, target_lang: str) -> str:
        """Translate English response back to user's language"""
        if target_lang == 'en':
            return text
        
        translation = self.translator.translate(text, src='en', dest=target_lang)
        return translation.text
    
    def process_multilingual_message(self, message: str) -> Dict[str, str]:
        """Process message in any language"""
        # Detect language
        source_lang = self.detect_language(message)
        
        # Translate to English for processing
        english_message = self.translate_to_english(message, source_lang)
        
        return {
            'original': message,
            'english': english_message,
            'source_language': source_lang
        }
```

### Step 14: Create Main Application

```python
# main.py
import os
from dotenv import load_dotenv
from src.nlp.intent_classifier import IntentClassifier
from src.nlp.entity_extractor import EntityExtractor
from src.nlp.sentiment_analyzer import SentimentAnalyzer
from src.dialogue.context_manager import ContextManager
from src.dialogue.response_generator import ResponseGenerator
from src.dialogue.state_tracker import DialogueStateTracker
from src.integrations.telegram_bot import TelegramBot
from src.utils.multi_language import MultiLanguageSupport
from src.utils.feedback_learner import FeedbackLearner

def main():
    # Load environment variables
    load_dotenv()
    
    # Initialize components
    print("Initializing AI Chatbot...")
    
    # Load and train intent classifier
    intent_classifier = IntentClassifier()
    texts, labels = intent_classifier.load_training_data('data/training/intents.json')
    intent_classifier.train(texts, labels)
    intent_classifier.save_model('src/models/intent_model.pkl')
    print("✓ Intent classifier trained!")
    
    # Initialize NLP components
    entity_extractor = EntityExtractor()
    sentiment_analyzer = SentimentAnalyzer()
    print("✓ NLP components initialized!")
    
    # Initialize dialogue management
    context_manager = ContextManager()
    state_tracker = DialogueStateTracker()
    print("✓ Dialogue management ready!")
    
    # Initialize response generator
    response_generator = ResponseGenerator(
        intent_classifier,
        context_manager,
        entity_extractor,
        sentiment_analyzer,
        state_tracker
    )
    print("✓ Response generator initialized!")
    
    # Initialize multi-language support
    multi_lang = MultiLanguageSupport()
    print("✓ Multi-language support enabled!")
    
    # Initialize feedback system
    feedback_learner = FeedbackLearner()
    print("✓ Feedback system ready!")
    
    # Get Telegram token
    telegram_token = os.getenv('TELEGRAM_BOT_TOKEN')
    
    if telegram_token:
        # Start Telegram bot
        print("Starting Telegram bot...")
        bot = TelegramBot(telegram_token, response_generator, multi_lang)
        bot.run()
    else:
        print("Error: TELEGRAM_BOT_TOKEN not found in .env file")
        print("Please create a .env file with your Telegram bot token")

if __name__ == "__main__":
    main()
```

### Step 15: Implement Voice Support

```python
import speech_recognition as sr
from gtts import gTTS
import os
from pydub import AudioSegment
from pydub.playback import play

class VoiceInterface:
    def __init__(self):
        self.recognizer = sr.Recognizer()
        self.microphone = sr.Microphone()
    
    def speech_to_text(self, audio_file: str = None) -> str:
        """Convert speech to text"""
        if audio_file:
            # From file
            with sr.AudioFile(audio_file) as source:
                audio = self.recognizer.record(source)
        else:
            # From microphone
            with self.microphone as source:
                self.recognizer.adjust_for_ambient_noise(source)
                print("Listening...")
                audio = self.recognizer.listen(source)
        
        try:
            text = self.recognizer.recognize_google(audio)
            return text
        except sr.UnknownValueError:
            return "Sorry, I couldn't understand that."
        except sr.RequestError as e:
            return f"Error with speech recognition service: {e}"
    
    def text_to_speech(self, text: str, lang: str = 'en', output_file: str = 'response.mp3'):
        """Convert text to speech"""
        tts = gTTS(text=text, lang=lang, slow=False)
        tts.save(output_file)
        return output_file
    
    def play_audio(self, audio_file: str):
        """Play audio file"""
        audio = AudioSegment.from_mp3(audio_file)
        play(audio)
    
    def process_voice_message(self, audio_file: str, chatbot) -> str:
        """Complete voice interaction pipeline"""
        # Convert voice to text
        user_message = self.speech_to_text(audio_file)
        print(f"User said: {user_message}")
        
        # Get bot response
        response = chatbot.generate_response(user_message)
        print(f"Bot responds: {response}")
        
        # Convert response to speech
        audio_response = self.text_to_speech(response)
        
        # Play response
        self.play_audio(audio_response)
        
        return response
```

---

## Technology Stack

### Backend & NLP
- **Python 3.8+**: Core programming language
- **Rasa**: Complete conversational AI framework (optional)
- **Transformers (Hugging Face)**: Pre-trained NLP models (BERT, GPT, DistilBERT)
- **spaCy 3.0+**: Industrial-strength NLP library with entity recognition
- **NLTK**: Natural language toolkit for preprocessing
- **scikit-learn**: Machine learning library for classification
- **TensorFlow/PyTorch**: Deep learning frameworks for custom models

### Messaging Platform Integration
- **python-telegram-bot 20.0+**: Telegram Bot API wrapper
- **discord.py 2.0+**: Discord API wrapper with slash commands
- **twilio**: WhatsApp Business API integration
- **slack-sdk**: Slack Bot integration
- **fastapi-websocket**: Real-time web chat

### Database & Storage
- **MongoDB**: Conversation history and user data storage
- **Redis**: Context caching and session management
- **PostgreSQL**: Analytics and structured data
- **Elasticsearch**: Search and conversation indexing

### Additional Tools
- **FastAPI**: High-performance REST API
- **WebSockets**: Real-time bidirectional communication
- **Docker**: Containerization for deployment
- **pytest**: Comprehensive testing framework
- **SpeechRecognition**: Voice input processing
- **gTTS (Google Text-to-Speech)**: Voice output generation
- **googletrans**: Multi-language translation

---

## Advanced Features Implementation

### Machine Learning Model Training

```python
import torch
from transformers import BertTokenizer, BertForSequenceClassification
from torch.utils.data import DataLoader, TensorDataset
from sklearn.model_selection import train_test_split
import numpy as np

class IntentClassifierBERT:
    def __init__(self, num_labels):
        self.tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
        self.model = BertForSequenceClassification.from_pretrained(
            'bert-base-uncased',
            num_labels=num_labels
        )
        self.device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
        self.model.to(self.device)
    
    def prepare_data(self, texts, labels):
        """Prepare data for BERT training"""
        # Tokenize
        encodings = self.tokenizer(
            texts,
            truncation=True,
            padding=True,
            max_length=128,
            return_tensors='pt'
        )
        
        # Create dataset
        dataset = TensorDataset(
            encodings['input_ids'],
            encodings['attention_mask'],
            torch.tensor(labels)
        )
        
        return dataset
    
    def train(self, train_texts, train_labels, epochs=3, batch_size=16):
        """Train BERT model"""
        from transformers import AdamW
        from tqdm import tqdm
        
        # Prepare data
        dataset = self.prepare_data(train_texts, train_labels)
        dataloader = DataLoader(dataset, batch_size=batch_size, shuffle=True)
        
        # Optimizer
        optimizer = AdamW(self.model.parameters(), lr=2e-5)
        
        # Training loop
        self.model.train()
        for epoch in range(epochs):
            total_loss = 0
            for batch in tqdm(dataloader, desc=f'Epoch {epoch+1}/{epochs}'):
                input_ids, attention_mask, labels = [b.to(self.device) for b in batch]
                
                # Forward pass
                outputs = self.model(
                    input_ids=input_ids,
                    attention_mask=attention_mask,
                    labels=labels
                )
                
                loss = outputs.loss
                total_loss += loss.item()
                
                # Backward pass
                optimizer.zero_grad()
                loss.backward()
                optimizer.step()
            
            avg_loss = total_loss / len(dataloader)
            print(f'Epoch {epoch+1} - Average Loss: {avg_loss:.4f}')
    
    def predict(self, text):
        """Predict intent from text"""
        self.model.eval()
        
        encoding = self.tokenizer(
            text,
            truncation=True,
            padding=True,
            max_length=128,
            return_tensors='pt'
        ).to(self.device)
        
        with torch.no_grad():
            outputs = self.model(**encoding)
            predictions = torch.nn.functional.softmax(outputs.logits, dim=-1)
            confidence, predicted_class = torch.max(predictions, dim=1)
        
        return predicted_class.item(), confidence.item()
    
    def save_model(self, path):
        """Save trained model"""
        self.model.save_pretrained(path)
        self.tokenizer.save_pretrained(path)
    
    def load_model(self, path):
        """Load trained model"""
        self.model = BertForSequenceClassification.from_pretrained(path)
        self.tokenizer = BertTokenizer.from_pretrained(path)
        self.model.to(self.device)
```

### Conversation Analytics Dashboard

```python
from datetime import datetime, timedelta
from collections import defaultdict
import pandas as pd
import matplotlib.pyplot as plt

class ConversationAnalytics:
    def __init__(self, context_manager, feedback_learner):
        self.context_manager = context_manager
        self.feedback_learner = feedback_learner
    
    def get_conversation_stats(self) -> dict:
        """Get overall conversation statistics"""
        total_users = len(self.context_manager.conversation_history)
        total_messages = sum(
            len(history) for history in self.context_manager.conversation_history.values()
        )
        
        return {
            'total_users': total_users,
            'total_messages': total_messages,
            'avg_messages_per_user': total_messages / total_users if total_users > 0 else 0
        }
    
    def get_intent_distribution(self) -> dict:
        """Get distribution of intents"""
        intent_counts = defaultdict(int)
        
        for user_history in self.context_manager.conversation_history.values():
            for message in user_history:
                if 'intent' in message:
                    intent_counts[message['intent']] += 1
        
        return dict(intent_counts)
    
    def get_response_time_stats(self) -> dict:
        """Calculate response time statistics"""
        response_times = []
        
        for user_history in self.context_manager.conversation_history.values():
            for i in range(len(user_history) - 1):
                if user_history[i]['is_user'] and not user_history[i+1]['is_user']:
                    time_diff = (
                        user_history[i+1]['timestamp'] - user_history[i]['timestamp']
                    ).total_seconds()
                    response_times.append(time_diff)
        
        if not response_times:
            return {}
        
        return {
            'avg_response_time': sum(response_times) / len(response_times),
            'min_response_time': min(response_times),
            'max_response_time': max(response_times),
            'total_responses': len(response_times)
        }
    
    def get_user_satisfaction(self) -> dict:
        """Get user satisfaction metrics from feedback"""
        feedback_stats = self.feedback_learner.analyze_feedback()
        
        if not feedback_stats:
            return {'satisfaction_rate': 0, 'total_feedback': 0}
        
        total = feedback_stats['total_feedback']
        positive = feedback_stats['positive_feedback']
        
        return {
            'satisfaction_rate': (positive / total * 100) if total > 0 else 0,
            'total_feedback': total,
            'positive': positive,
            'negative': feedback_stats['negative_feedback'],
            'neutral': feedback_stats['neutral_feedback']
        }
    
    def generate_report(self) -> str:
        """Generate comprehensive analytics report"""
        stats = self.get_conversation_stats()
        intents = self.get_intent_distribution()
        response_times = self.get_response_time_stats()
        satisfaction = self.get_user_satisfaction()
        
        report = f"""
        ========================================
        CHATBOT ANALYTICS REPORT
        ========================================
        
        CONVERSATION STATISTICS
        -----------------------
        Total Users: {stats['total_users']}
        Total Messages: {stats['total_messages']}
        Avg Messages/User: {stats['avg_messages_per_user']:.2f}
        
        INTENT DISTRIBUTION
        -------------------
        """
        
        for intent, count in sorted(intents.items(), key=lambda x: x[1], reverse=True):
            report += f"{intent}: {count}\n        "
        
        report += f"""
        
        RESPONSE TIME
        -------------
        Average: {response_times.get('avg_response_time', 0):.2f}s
        Min: {response_times.get('min_response_time', 0):.2f}s
        Max: {response_times.get('max_response_time', 0):.2f}s
        
        USER SATISFACTION
        -----------------
        Satisfaction Rate: {satisfaction['satisfaction_rate']:.1f}%
        Total Feedback: {satisfaction['total_feedback']}
        Positive: {satisfaction.get('positive', 0)}
        Negative: {satisfaction.get('negative', 0)}
        Neutral: {satisfaction.get('neutral', 0)}
        
        ========================================
        """
        
        return report
    
    def plot_intent_distribution(self, save_path='intent_distribution.png'):
        """Create visualization of intent distribution"""
        intents = self.get_intent_distribution()
        
        plt.figure(figsize=(10, 6))
        plt.bar(intents.keys(), intents.values())
        plt.xlabel('Intent')
        plt.ylabel('Count')
        plt.title('Intent Distribution')
        plt.xticks(rotation=45, ha='right')
        plt.tight_layout()
        plt.savefig(save_path)
        plt.close()
```

### Integration with Discord

```python
import discord
from discord.ext import commands
from discord import app_commands

class DiscordBot(commands.Bot):
    def __init__(self, token, response_generator):
        intents = discord.Intents.default()
        intents.message_content = True
        super().__init__(command_prefix='!', intents=intents)
        
        self.token = token
        self.response_generator = response_generator
    
    async def setup_hook(self):
        """Setup slash commands"""
        await self.tree.sync()
    
    async def on_ready(self):
        """Bot is ready"""
        print(f'{self.user} has connected to Discord!')
        print(f'Bot is in {len(self.guilds)} guilds')
    
    async def on_message(self, message):
        """Handle incoming messages"""
        # Ignore bot's own messages
        if message.author == self.user:
            return
        
        # Ignore commands
        if message.content.startswith('!'):
            await self.process_commands(message)
            return
        
        # Generate response
        user_id = str(message.author.id)
        user_input = message.content
        
        async with message.channel.typing():
            response = self.response_generator.generate_response(user_id, user_input)
        
        await message.reply(response)
    
    @app_commands.command(name="chat", description="Chat with the AI assistant")
    async def chat_command(self, interaction: discord.Interaction, message: str):
        """Slash command for chatting"""
        user_id = str(interaction.user.id)
        
        await interaction.response.defer(thinking=True)
        
        response = self.response_generator.generate_response(user_id, message)
        
        await interaction.followup.send(response)
    
    @app_commands.command(name="reset", description="Reset conversation context")
    async def reset_command(self, interaction: discord.Interaction):
        """Reset conversation context"""
        user_id = str(interaction.user.id)
        self.response_generator.context_manager.clear_context(user_id)
        
        await interaction.response.send_message("Conversation context has been reset!")
    
    @app_commands.command(name="stats", description="Get your conversation statistics")
    async def stats_command(self, interaction: discord.Interaction):
        """Get user statistics"""
        user_id = str(interaction.user.id)
        history = self.response_generator.context_manager.get_history(user_id)
        
        stats = f"""
        **Your Conversation Stats**
        Total Messages: {len(history)}
        User Messages: {len([m for m in history if m['is_user']])}
        Bot Responses: {len([m for m in history if not m['is_user']])}
        """
        
        await interaction.response.send_message(stats)
    
    def run_bot(self):
        """Run the Discord bot"""
        self.run(self.token)
```

### Complete Requirements File

```txt
# Core Dependencies
python>=3.8

# NLP & ML
transformers==4.35.0
torch==2.1.0
spacy==3.7.0
nltk==3.8.1
scikit-learn==1.3.0
tensorflow==2.14.0  # Alternative to PyTorch

# Messaging Platforms
python-telegram-bot==20.7
discord.py==2.3.2
twilio==8.10.0
slack-sdk==3.23.0

# API & Web
fastapi==0.104.1
uvicorn==0.24.0
websockets==12.0
httpx==0.25.1
requests==2.31.0

# Database
pymongo==4.6.0
redis==5.0.1
psycopg2-binary==2.9.9
motor==3.3.2  # Async MongoDB

# Voice Processing
SpeechRecognition==3.10.0
gTTS==2.4.0
pydub==0.25.1
pyaudio==0.2.14

# Translation
googletrans==4.0.0rc1

# Data Processing
pandas==2.1.3
numpy==1.26.2

# Visualization
matplotlib==3.8.2
seaborn==0.13.0

# Testing
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0

# Utilities
python-dotenv==1.0.0
pydantic==2.5.0
aiohttp==3.9.1
tqdm==4.66.1

# Deployment
gunicorn==21.2.0
docker==6.1.3
```

---

## Testing Checklist

- [ ] Intent classification accuracy > 85%
- [ ] Context maintained across conversation
- [ ] Fallback responses for unknown queries
- [ ] API integrations working correctly
- [ ] Messaging platform integration functional
- [ ] Response time < 2 seconds
- [ ] Sentiment analysis working
- [ ] Conversation history stored correctly
- [ ] Multi-user support tested
- [ ] Error handling for edge cases

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
- Add more intents and training data
- Implement a simple admin dashboard
- Add emoji support in responses
- Create conversation export feature

### Medium
- Implement sentiment-based response adjustment
- Add voice recognition (speech-to-text)
- Create a web interface using React
- Implement user feedback collection
- Add multi-language support

### Hard
- Fine-tune a GPT model on custom data
- Implement advanced dialogue state tracking
- Create a recommendation system
- Build conversation analytics with ML insights
- Implement A/B testing for responses
- Add memory and long-term context retention
- Create custom TTS with personality
- Implement transfer learning for domain-specific knowledge

---

## Common Pitfalls

**Poor training data quality**: Ensure diverse and representative training examples  
**Overfitting to training data**: Use validation sets and regularization  
**Ignoring context**: Always maintain conversation state  
**Slow response times**: Implement caching and optimize model inference  
**Security vulnerabilities**: Validate inputs and protect API keys  
**Scalability issues**: Design for concurrent users from the start  

---

## Resources

### Documentation
- [Rasa Documentation](https://rasa.com/docs/)
- [Hugging Face Transformers](https://huggingface.co/docs/transformers/)
- [spaCy Documentation](https://spacy.io/usage)
- [Telegram Bot API](https://core.telegram.org/bots/api)

### Tutorials
- [Building Chatbots with Rasa](https://rasa.com/docs/rasa/tutor/)
- [NLP with Python](https://www.nltk.org/book/)
- [Fine-tuning BERT](https://huggingface.co/docs/transformers/training)

### Courses
- [Natural Language Processing Specialization (Coursera)](https://www.coursera.org/specializations/natural-language-processing)
- [Deep Learning for NLP (Stanford CS224N)](http://web.stanford.edu/class/cs224n/)

---

## Deployment Guide

### Option 1: Heroku
```bash
# Create Procfile
echo "worker: python main.py" > Procfile

# Deploy
heroku create your-chatbot
git push heroku main
heroku ps:scale worker=1
```

### Option 2: Docker
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["python", "main.py"]
```

### Option 3: AWS Lambda
Use AWS Lambda with API Gateway for serverless deployment.

---

## Submission Checklist

Before considering your project complete:

- [ ] Code is well-documented with docstrings
- [ ] Unit tests written and passing
- [ ] Intent classifier achieves good accuracy
- [ ] At least one messaging platform integrated
- [ ] External API integration working
- [ ] Error handling implemented
- [ ] README with setup instructions
- [ ] Environment variables documented
- [ ] Demo video or screenshots
- [ ] Deployment instructions included

---

## Portfolio Tips

When showcasing this project:

1. **Demo Video**: Show real conversations with the bot
2. **Metrics**: Display accuracy, response time, and user engagement
3. **Architecture Diagram**: Visualize system components
4. **Code Quality**: Highlight clean architecture and best practices
5. **Scalability**: Discuss how it handles multiple users
6. **Innovation**: Show unique features or integrations

---

## License

This project idea is part of the [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
