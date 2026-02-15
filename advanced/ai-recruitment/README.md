# AI-Powered Recruitment Platform

<p align="center">
  <img src="https://images.unsplash.com/photo-1454165804606-c3d57bc86b40?w=600&q=80" alt="AI Recruitment" width="600">
</p>

## Overview

**Difficulty:** Advanced  
**Estimated Time:** 35–45 hours  
**Prerequisites:** Python/Node.js, NLP, Machine Learning, OCR, Video Processing, React, MongoDB/PostgreSQL  

An **AI-Powered Recruitment Platform** revolutionizes the hiring process by automating resume screening, candidate matching, and interview assessments using artificial intelligence. The system can:

- Parse and extract information from resumes (PDF, DOCX, TXT)
- Use NLP to identify skills, experience, and qualifications
- Match candidates to job openings using ML algorithms
- Conduct automated video interviews with AI analysis
- Detect bias in job descriptions and hiring decisions
- Provide recruiter dashboards with candidate insights
- Generate automated candidate reports
- Track the entire hiring pipeline (ATS - Applicant Tracking System)

This project covers Natural Language Processing, Machine Learning, Computer Vision, OCR, video analysis, speech recognition, and building a complete recruitment workflow similar to platforms like Lever, Greenhouse, or HireVue.

---

## Features

### Must Have

- User roles (recruiter, hiring manager, candidate)
- Resume upload (PDF, DOCX, TXT)
- Automatic resume parsing and information extraction
- Job posting creation with requirements
- Candidate search and filtering
- AI-powered candidate-job matching
- Applicant tracking system (ATS) workflow:
  - Applied → Screening → Interview → Offer → Hired/Rejected
- Candidate profile with extracted information
- Email notifications for status updates
- Interview scheduling
- Recruiter dashboard with analytics
- Responsive design

### Nice to Have

- Video interview recording and upload
- AI video interview analysis:
  - Facial expression analysis
  - Speech-to-text transcription
  - Sentiment analysis of responses
  - Confidence scoring
- Automated interview questions generation
- Bias detection in job descriptions
- Diversity and inclusion metrics
- Candidate ranking algorithms
- Skills gap analysis
- Salary recommendations based on market data
- Reference checking automation
- Background check integration
- Multi-language support (i18n)
- Chrome extension for LinkedIn sourcing
- Candidate relationship management (CRM)
- Interview scorecards with rubrics
- Offer letter generation
- Onboarding checklist
- Integration with job boards (Indeed, LinkedIn)
- Chatbot for candidate Q&A
- Mobile app

---

## Learning Objectives

By building this project, you will learn:

- **Natural Language Processing (NLP)** with spaCy, NLTK, or Hugging Face
- **Named Entity Recognition (NER)** for information extraction
- **Machine Learning** for candidate-job matching
- **OCR (Optical Character Recognition)** with Tesseract or AWS Textract
- **Resume parsing** from multiple formats
- **Text classification** and similarity algorithms
- **Speech recognition** with Google Speech-to-Text or Whisper
- **Computer vision** for facial analysis
- **Sentiment analysis** of text and audio
- **Bias detection** algorithms
- **Recommendation systems** for candidate matching
- **Video processing** with FFmpeg
- **File upload** and storage (AWS S3, Cloudinary)
- **Workflow automation** (state machines)
- **Data privacy** and compliance (GDPR, EEOC)

---

## Project Concepts

This project simulates a modern AI recruitment system:

- **Resume Parsing:** Extract structured data from unstructured documents
- **Skill Matching:** Use TF-IDF, word embeddings, or semantic similarity
- **Candidate Scoring:** Rank candidates based on multiple factors
- **Bias Detection:** Identify discriminatory language or patterns
- **Video Analysis:** Assess non-verbal communication and speech
- **Workflow Management:** Track candidates through hiring stages
- **Analytics:** Provide insights on hiring metrics (time-to-hire, diversity, etc.)

---

## Step-by-Step Implementation

### Step 1: Resume Parsing Service

Extract information from resumes using OCR and NLP:

```python
# services/resume_parser.py
import re
import spacy
import pdfplumber
from docx import Document
import pytesseract
from PIL import Image
import io

class ResumeParser:
    def __init__(self):
        self.nlp = spacy.load("en_core_web_lg")
        self.skills_database = self.load_skills_database()
    
    def load_skills_database(self):
        # Load comprehensive skills database
        return {
            'programming': ['Python', 'JavaScript', 'Java', 'C++', 'Ruby', 'Go', 'Rust', 'TypeScript'],
            'frameworks': ['React', 'Angular', 'Vue', 'Django', 'Flask', 'Express', 'Spring'],
            'databases': ['MongoDB', 'PostgreSQL', 'MySQL', 'Redis', 'Elasticsearch', 'Cassandra'],
            'cloud': ['AWS', 'Azure', 'GCP', 'Docker', 'Kubernetes', 'Terraform'],
            'soft_skills': ['Leadership', 'Communication', 'Teamwork', 'Problem Solving', 'Critical Thinking']
        }
    
    def parse_resume(self, file_path, file_type):
        """Main parsing method"""
        try:
            # Extract text based on file type
            if file_type == 'pdf':
                text = self.extract_text_from_pdf(file_path)
            elif file_type == 'docx':
                text = self.extract_text_from_docx(file_path)
            elif file_type == 'txt':
                with open(file_path, 'r', encoding='utf-8') as f:
                    text = f.read()
            else:
                raise ValueError(f"Unsupported file type: {file_type}")
            
            # Parse the text
            parsed_data = self.extract_information(text)
            
            return {
                'success': True,
                'data': parsed_data,
                'raw_text': text
            }
            
        except Exception as e:
            return {
                'success': False,
                'error': str(e)
            }
    
    def extract_text_from_pdf(self, file_path):
        """Extract text from PDF using pdfplumber"""
        text = ""
        with pdfplumber.open(file_path) as pdf:
            for page in pdf.pages:
                extracted = page.extract_text()
                if extracted:
                    text += extracted + "\n"
        
        # If no text found, try OCR
        if not text.strip():
            text = self.ocr_pdf(file_path)
        
        return text
    
    def ocr_pdf(self, file_path):
        """OCR for scanned PDFs"""
        from pdf2image import convert_from_path
        
        images = convert_from_path(file_path)
        text = ""
        
        for image in images:
            text += pytesseract.image_to_string(image) + "\n"
        
        return text
    
    def extract_text_from_docx(self, file_path):
        """Extract text from DOCX"""
        doc = Document(file_path)
        text = "\n".join([para.text for para in doc.paragraphs])
        return text
    
    def extract_information(self, text):
        """Extract structured information from text"""
        doc = self.nlp(text)
        
        extracted = {
            'name': self.extract_name(doc, text),
            'email': self.extract_email(text),
            'phone': self.extract_phone(text),
            'skills': self.extract_skills(text, doc),
            'experience': self.extract_experience(text, doc),
            'education': self.extract_education(text, doc),
            'certifications': self.extract_certifications(text),
            'languages': self.extract_languages(text),
            'summary': self.extract_summary(text)
        }
        
        return extracted
    
    def extract_name(self, doc, text):
        """Extract candidate name using NER"""
        # Try to find name in first few lines
        lines = text.split('\n')[:5]
        
        for line in lines:
            line_doc = self.nlp(line)
            for ent in line_doc.ents:
                if ent.label_ == 'PERSON':
                    return ent.text
        
        # Fallback: first capitalized words
        for line in lines:
            words = line.split()
            if len(words) >= 2:
                if words[0][0].isupper() and words[1][0].isupper():
                    return f"{words[0]} {words[1]}"
        
        return None
    
    def extract_email(self, text):
        """Extract email using regex"""
        email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
        match = re.search(email_pattern, text)
        return match.group(0) if match else None
    
    def extract_phone(self, text):
        """Extract phone number using regex"""
        phone_patterns = [
            r'\+?1?\s*\(?(\d{3})\)?[-.\s]?(\d{3})[-.\s]?(\d{4})',
            r'\+\d{1,3}\s?\(?\d{1,4}\)?[\s.-]?\d{1,4}[\s.-]?\d{1,9}'
        ]
        
        for pattern in phone_patterns:
            match = re.search(pattern, text)
            if match:
                return match.group(0)
        
        return None
    
    def extract_skills(self, text, doc):
        """Extract skills by matching against skills database"""
        found_skills = {
            'technical': [],
            'soft': []
        }
        
        text_lower = text.lower()
        
        # Match technical skills
        for category, skills in self.skills_database.items():
            if category != 'soft_skills':
                for skill in skills:
                    if skill.lower() in text_lower:
                        found_skills['technical'].append(skill)
        
        # Match soft skills
        for skill in self.skills_database['soft_skills']:
            if skill.lower() in text_lower:
                found_skills['soft'].append(skill)
        
        # Remove duplicates
        found_skills['technical'] = list(set(found_skills['technical']))
        found_skills['soft'] = list(set(found_skills['soft']))
        
        return found_skills
    
    def extract_experience(self, text, doc):
        """Extract work experience"""
        experience = []
        
        # Look for experience section
        exp_section = self.find_section(text, ['experience', 'work history', 'employment'])
        
        if exp_section:
            # Parse job entries
            job_pattern = r'([A-Z][^,\n]+)\s*,?\s*([\d]{4})\s*[-–]\s*([\d]{4}|Present|Current)'
            matches = re.finditer(job_pattern, exp_section, re.MULTILINE)
            
            for match in matches:
                experience.append({
                    'title': match.group(1).strip(),
                    'start_year': int(match.group(2)),
                    'end_year': match.group(3) if match.group(3) in ['Present', 'Current'] else int(match.group(3))
                })
        
        return experience
    
    def extract_education(self, text, doc):
        """Extract education information"""
        education = []
        
        # Look for education section
        edu_section = self.find_section(text, ['education', 'academic', 'qualifications'])
        
        if edu_section:
            # Common degree patterns
            degree_patterns = [
                r'(Bachelor|Master|PhD|Associate|B\.S\.|M\.S\.|MBA|B\.A\.|M\.A\.)',
                r'(BS|MS|BA|MA|Ph\.D\.)'
            ]
            
            for pattern in degree_patterns:
                matches = re.finditer(pattern, edu_section, re.IGNORECASE)
                for match in matches:
                    # Try to find associated year and institution
                    context = edu_section[max(0, match.start()-100):min(len(edu_section), match.end()+100)]
                    year_match = re.search(r'(19|20)\d{2}', context)
                    
                    education.append({
                        'degree': match.group(0),
                        'year': int(year_match.group(0)) if year_match else None
                    })
        
        return education
    
    def extract_certifications(self, text):
        """Extract certifications"""
        certifications = []
        
        cert_section = self.find_section(text, ['certifications', 'certificates', 'licenses'])
        
        if cert_section:
            # Common certifications
            common_certs = ['AWS', 'Azure', 'PMP', 'CISSP', 'CPA', 'Six Sigma', 'Scrum Master']
            
            for cert in common_certs:
                if cert.lower() in cert_section.lower():
                    certifications.append(cert)
        
        return certifications
    
    def extract_languages(self, text):
        """Extract language proficiency"""
        languages = []
        
        lang_section = self.find_section(text, ['languages', 'language proficiency'])
        
        if lang_section:
            common_languages = ['English', 'Spanish', 'French', 'German', 'Chinese', 'Japanese', 
                              'Portuguese', 'Russian', 'Arabic', 'Hindi']
            
            for lang in common_languages:
                if lang.lower() in lang_section.lower():
                    languages.append(lang)
        
        return languages
    
    def extract_summary(self, text):
        """Extract professional summary"""
        summary_section = self.find_section(text, ['summary', 'profile', 'objective', 'about'])
        
        if summary_section:
            # Take first 200 words
            words = summary_section.split()[:200]
            return ' '.join(words)
        
        return None
    
    def find_section(self, text, section_headers):
        """Find a section in the resume by headers"""
        text_lower = text.lower()
        
        for header in section_headers:
            pattern = rf'\n\s*{header}\s*\n(.*?)(?=\n\s*[A-Z][a-z]+\s*\n|\Z)'
            match = re.search(pattern, text_lower, re.DOTALL | re.IGNORECASE)
            
            if match:
                return match.group(1)
        
        return None

# Initialize parser
resume_parser = ResumeParser()
```

### Step 2: Candidate-Job Matching Algorithm

Use ML to match candidates to jobs:

```python
# services/matching_service.py
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

class CandidateMatchingService:
    def __init__(self):
        self.vectorizer = TfidfVectorizer(
            max_features=500,
            stop_words='english',
            ngram_range=(1, 2)
        )
    
    def calculate_match_score(self, candidate, job):
        """Calculate match score between candidate and job"""
        
        # Extract features
        candidate_text = self.create_candidate_text(candidate)
        job_text = self.create_job_text(job)
        
        # Vectorize
        vectors = self.vectorizer.fit_transform([candidate_text, job_text])
        
        # Calculate cosine similarity
        similarity = cosine_similarity(vectors[0:1], vectors[1:2])[0][0]
        
        # Calculate component scores
        skills_score = self.calculate_skills_match(candidate, job)
        experience_score = self.calculate_experience_match(candidate, job)
        education_score = self.calculate_education_match(candidate, job)
        
        # Weighted average
        final_score = (
            similarity * 0.4 +
            skills_score * 0.35 +
            experience_score * 0.15 +
            education_score * 0.10
        ) * 100
        
        return {
            'overall_score': round(final_score, 2),
            'skills_score': round(skills_score * 100, 2),
            'experience_score': round(experience_score * 100, 2),
            'education_score': round(education_score * 100, 2),
            'text_similarity': round(similarity * 100, 2),
            'breakdown': self.generate_breakdown(candidate, job)
        }
    
    def create_candidate_text(self, candidate):
        """Create text representation of candidate"""
        parts = []
        
        if candidate.get('skills'):
            technical = ' '.join(candidate['skills'].get('technical', []))
            soft = ' '.join(candidate['skills'].get('soft', []))
            parts.append(technical + ' ' + soft)
        
        if candidate.get('experience'):
            exp_text = ' '.join([e.get('title', '') for e in candidate['experience']])
            parts.append(exp_text)
        
        if candidate.get('education'):
            edu_text = ' '.join([e.get('degree', '') for e in candidate['education']])
            parts.append(edu_text)
        
        if candidate.get('summary'):
            parts.append(candidate['summary'])
        
        return ' '.join(parts)
    
    def create_job_text(self, job):
        """Create text representation of job"""
        parts = [
            job.get('title', ''),
            job.get('description', ''),
            ' '.join(job.get('required_skills', [])),
            ' '.join(job.get('preferred_skills', []))
        ]
        
        return ' '.join(parts)
    
    def calculate_skills_match(self, candidate, job):
        """Calculate skills match percentage"""
        candidate_skills = set([s.lower() for s in 
                               candidate.get('skills', {}).get('technical', [])])
        
        required_skills = set([s.lower() for s in job.get('required_skills', [])])
        preferred_skills = set([s.lower() for s in job.get('preferred_skills', [])])
        
        if not required_skills:
            return 0.5  # Default if no requirements
        
        # Calculate matches
        required_matches = len(candidate_skills & required_skills)
        preferred_matches = len(candidate_skills & preferred_skills)
        
        # Score: 70% for required, 30% for preferred
        required_score = required_matches / len(required_skills) if required_skills else 1.0
        preferred_score = preferred_matches / len(preferred_skills) if preferred_skills else 0.5
        
        return (required_score * 0.7) + (preferred_score * 0.3)
    
    def calculate_experience_match(self, candidate, job):
        """Calculate experience match"""
        candidate_experience = candidate.get('experience', [])
        
        if not candidate_experience:
            return 0.3
        
        total_years = 0
        for exp in candidate_experience:
            start = exp.get('start_year')
            end = exp.get('end_year')
            
            if start:
                if end and isinstance(end, int):
                    total_years += end - start
                else:
                    # Current job
                    total_years += 2024 - start
        
        required_years = job.get('required_experience_years', 0)
        
        if required_years == 0:
            return 1.0
        
        # Score based on how close to requirement
        if total_years >= required_years:
            return 1.0
        elif total_years >= required_years * 0.7:
            return 0.8
        elif total_years >= required_years * 0.5:
            return 0.6
        else:
            return total_years / required_years
    
    def calculate_education_match(self, candidate, job):
        """Calculate education match"""
        candidate_education = candidate.get('education', [])
        required_education = job.get('required_education', '')
        
        if not required_education:
            return 1.0
        
        if not candidate_education:
            return 0.0
        
        # Simple degree level matching
        degree_hierarchy = {
            'associate': 1,
            'bachelor': 2,
            'master': 3,
            'phd': 4,
            'doctorate': 4
        }
        
        required_level = 0
        for degree, level in degree_hierarchy.items():
            if degree in required_education.lower():
                required_level = level
                break
        
        candidate_level = 0
        for edu in candidate_education:
            degree = edu.get('degree', '').lower()
            for deg_name, level in degree_hierarchy.items():
                if deg_name in degree:
                    candidate_level = max(candidate_level, level)
        
        if candidate_level >= required_level:
            return 1.0
        elif candidate_level == required_level - 1:
            return 0.7
        else:
            return 0.4
    
    def generate_breakdown(self, candidate, job):
        """Generate detailed match breakdown"""
        candidate_skills = set([s.lower() for s in 
                               candidate.get('skills', {}).get('technical', [])])
        required_skills = set([s.lower() for s in job.get('required_skills', [])])
        preferred_skills = set([s.lower() for s in job.get('preferred_skills', [])])
        
        return {
            'matched_required_skills': list(candidate_skills & required_skills),
            'missing_required_skills': list(required_skills - candidate_skills),
            'matched_preferred_skills': list(candidate_skills & preferred_skills),
            'missing_preferred_skills': list(preferred_skills - candidate_skills)
        }
    
    def rank_candidates(self, candidates, job):
        """Rank all candidates for a job"""
        scored_candidates = []
        
        for candidate in candidates:
            score_data = self.calculate_match_score(candidate, job)
            scored_candidates.append({
                'candidate_id': candidate.get('_id'),
                'name': candidate.get('name'),
                'email': candidate.get('email'),
                'score': score_data['overall_score'],
                'breakdown': score_data
            })
        
        # Sort by score descending
        scored_candidates.sort(key=lambda x: x['score'], reverse=True)
        
        return scored_candidates

# Initialize matching service
matching_service = CandidateMatchingService()
```

### Step 3: Video Interview Analysis

Analyze video interviews using AI:

```python
# services/video_analysis_service.py
import cv2
import numpy as np
from google.cloud import speech_v1p1beta1 as speech
from deepface import DeepFace
import librosa
from transformers import pipeline

class VideoAnalysisService:
    def __init__(self):
        self.speech_client = speech.SpeechClient()
        self.sentiment_analyzer = pipeline("sentiment-analysis")
    
    def analyze_video_interview(self, video_path):
        """Complete video interview analysis"""
        try:
            results = {
                'transcription': self.transcribe_audio(video_path),
                'facial_analysis': self.analyze_facial_expressions(video_path),
                'speech_analysis': self.analyze_speech_patterns(video_path),
                'confidence_score': 0,
                'recommendations': []
            }
            
            # Calculate overall confidence score
            results['confidence_score'] = self.calculate_confidence_score(results)
            
            # Generate recommendations
            results['recommendations'] = self.generate_recommendations(results)
            
            return results
            
        except Exception as e:
            print(f"Error analyzing video: {e}")
            return None
    
    def transcribe_audio(self, video_path):
        """Transcribe audio to text"""
        # Extract audio from video
        audio_path = self.extract_audio(video_path)
        
        with open(audio_path, 'rb') as audio_file:
            content = audio_file.read()
        
        audio = speech.RecognitionAudio(content=content)
        config = speech.RecognitionConfig(
            encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
            sample_rate_hertz=16000,
            language_code="en-US",
            enable_automatic_punctuation=True,
            enable_word_time_offsets=True,
        )
        
        response = self.speech_client.recognize(config=config, audio=audio)
        
        transcription = {
            'full_text': '',
            'segments': [],
            'word_count': 0,
            'filler_words': 0
        }
        
        filler_words = ['um', 'uh', 'like', 'you know', 'sort of', 'kind of']
        
        for result in response.results:
            alternative = result.alternatives[0]
            transcription['full_text'] += alternative.transcript + ' '
            
            # Count filler words
            text_lower = alternative.transcript.lower()
            for filler in filler_words:
                transcription['filler_words'] += text_lower.count(filler)
        
        transcription['word_count'] = len(transcription['full_text'].split())
        
        # Sentiment analysis
        sentiment = self.sentiment_analyzer(transcription['full_text'][:512])
        transcription['sentiment'] = sentiment[0]
        
        return transcription
    
    def extract_audio(self, video_path):
        """Extract audio from video using FFmpeg"""
        import subprocess
        
        audio_path = video_path.replace('.mp4', '.wav')
        
        command = [
            'ffmpeg',
            '-i', video_path,
            '-vn',
            '-acodec', 'pcm_s16le',
            '-ar', '16000',
            '-ac', '1',
            audio_path
        ]
        
        subprocess.run(command, check=True, capture_output=True)
        
        return audio_path
    
    def analyze_facial_expressions(self, video_path):
        """Analyze facial expressions frame by frame"""
        cap = cv2.VideoCapture(video_path)
        
        expressions = {
            'happy': 0,
            'sad': 0,
            'angry': 0,
            'surprise': 0,
            'fear': 0,
            'disgust': 0,
            'neutral': 0
        }
        
        frame_count = 0
        sample_rate = 30  # Analyze every 30th frame
        
        while cap.isOpened():
            ret, frame = cap.read()
            
            if not ret:
                break
            
            frame_count += 1
            
            if frame_count % sample_rate == 0:
                try:
                    # Analyze face
                    analysis = DeepFace.analyze(
                        frame,
                        actions=['emotion'],
                        enforce_detection=False
                    )
                    
                    if analysis:
                        dominant_emotion = analysis[0]['dominant_emotion']
                        expressions[dominant_emotion] += 1
                
                except Exception:
                    # No face detected or error
                    continue
        
        cap.release()
        
        # Calculate percentages
        total = sum(expressions.values())
        if total > 0:
            expression_percentages = {
                k: round((v / total) * 100, 2) 
                for k, v in expressions.items()
            }
        else:
            expression_percentages = expressions
        
        return {
            'expression_distribution': expression_percentages,
            'frames_analyzed': total,
            'dominant_expression': max(expressions, key=expressions.get)
        }
    
    def analyze_speech_patterns(self, video_path):
        """Analyze speech patterns (pace, volume, clarity)"""
        audio_path = self.extract_audio(video_path)
        
        # Load audio
        y, sr = librosa.load(audio_path)
        
        # Calculate features
        tempo, _ = librosa.beat.beat_track(y=y, sr=sr)
        
        # RMS Energy (volume)
        rms = librosa.feature.rms(y=y)[0]
        avg_volume = np.mean(rms)
        
        # Zero crossing rate (clarity)
        zcr = librosa.feature.zero_crossing_rate(y)[0]
        avg_zcr = np.mean(zcr)
        
        # Speech rate (words per minute)
        duration = librosa.get_duration(y=y, sr=sr)
        
        return {
            'tempo': round(float(tempo), 2),
            'average_volume': round(float(avg_volume), 4),
            'clarity_score': round(float(avg_zcr), 4),
            'duration_seconds': round(duration, 2)
        }
    
    def calculate_confidence_score(self, analysis_results):
        """Calculate overall confidence score"""
        score = 0
        
        # Facial expression score (30%)
        facial = analysis_results['facial_analysis']
        positive_emotions = facial['expression_distribution'].get('happy', 0) + \
                          facial['expression_distribution'].get('surprise', 0)
        negative_emotions = facial['expression_distribution'].get('sad', 0) + \
                          facial['expression_distribution'].get('angry', 0) + \
                          facial['expression_distribution'].get('fear', 0)
        
        facial_score = (positive_emotions - negative_emotions) / 100
        score += max(0, min(1, facial_score)) * 30
        
        # Speech clarity score (30%)
        speech = analysis_results['speech_analysis']
        clarity_score = min(1, speech['clarity_score'] * 10)
        score += clarity_score * 30
        
        # Sentiment score (20%)
        sentiment = analysis_results['transcription']['sentiment']
        sentiment_score = sentiment['score'] if sentiment['label'] == 'POSITIVE' else 1 - sentiment['score']
        score += sentiment_score * 20
        
        # Filler words penalty (10%)
        transcription = analysis_results['transcription']
        if transcription['word_count'] > 0:
            filler_ratio = transcription['filler_words'] / transcription['word_count']
            filler_score = max(0, 1 - (filler_ratio * 5))
            score += filler_score * 10
        
        # Speech pace score (10%)
        duration = speech['duration_seconds']
        word_count = transcription['word_count']
        if duration > 0:
            words_per_minute = (word_count / duration) * 60
            # Optimal: 120-150 wpm
            if 120 <= words_per_minute <= 150:
                pace_score = 1.0
            else:
                pace_score = max(0, 1 - abs(words_per_minute - 135) / 135)
            score += pace_score * 10
        
        return round(score, 2)
    
    def generate_recommendations(self, analysis_results):
        """Generate recommendations based on analysis"""
        recommendations = []
        
        # Facial expression recommendations
        facial = analysis_results['facial_analysis']
        if facial['expression_distribution'].get('neutral', 0) > 50:
            recommendations.append("Consider showing more enthusiasm and positive expressions")
        
        # Speech recommendations
        transcription = analysis_results['transcription']
        if transcription['filler_words'] > transcription['word_count'] * 0.05:
            recommendations.append("Try to reduce filler words (um, uh, like)")
        
        # Sentiment recommendations
        sentiment = transcription['sentiment']
        if sentiment['label'] == 'NEGATIVE':
            recommendations.append("Focus on positive and constructive language")
        
        return recommendations

# Initialize video analysis service
video_analysis_service = VideoAnalysisService()
```

## Technology Stack

### Frontend
- **React** - UI framework
- **TypeScript** - Type safety
- **Redux Toolkit** - State management
- **React Query** - Server state
- **Material-UI** or **Ant Design** - Component library
- **React Dropzone** - File uploads
- **React Video Player** - Video playback

### Backend
- **Node.js + Express** or **Python + Flask/FastAPI**
- **MongoDB** or **PostgreSQL** - Database
- **Redis** - Caching and job queue
- **Bull** - Background job processing
- **Socket.io** - Real-time notifications

### AI/ML
- **Python** - ML/NLP processing
- **spaCy** or **Hugging Face Transformers** - NLP
- **scikit-learn** - ML algorithms
- **TensorFlow** or **PyTorch** - Deep learning
- **DeepFace** - Facial recognition
- **OpenCV** - Video processing
- **librosa** - Audio analysis
- **Tesseract** or **AWS Textract** - OCR

### Cloud Services
- **AWS S3** or **Google Cloud Storage** - File storage
- **Google Cloud Speech-to-Text** - Transcription
- **Google Cloud Video Intelligence** - Video analysis
- **AWS Lambda** - Serverless functions
- **SendGrid** or **AWS SES** - Email

## Testing Checklist

- [ ] Resume parsing works for PDF, DOCX, TXT
- [ ] Information extraction is accurate (name, email, skills, etc.)
- [ ] Candidate-job matching algorithm ranks correctly
- [ ] Video upload and processing works
- [ ] Speech transcription is accurate
- [ ] Facial expression analysis works
- [ ] Confidence score calculation is reasonable
- [ ] ATS workflow transitions correctly
- [ ] Email notifications sent properly
- [ ] Search and filtering works
- [ ] Recruiter dashboard displays metrics
- [ ] Bias detection flags issues
- [ ] Responsive design works on mobile

## Additional Challenges

### Easy Extensions
- Add candidate notes and tags
- Implement interview scheduling calendar
- Create candidate pipelines (kanban view)
- Add email templates
- Export candidate reports to PDF

### Medium Extensions
- Build Chrome extension for LinkedIn sourcing
- Implement chatbot for candidate Q&A
- Add skills assessment tests
- Create offer letter templates
- Integrate with job boards (Indeed, LinkedIn)
- Multi-language support (i18n)

### Hard Extensions
- Advanced bias detection in descriptions
- Predictive analytics (success probability)
- Automated reference checking
- Background check integration
- Salary benchmarking with market data
- Diversity and inclusion analytics
- Interview question generation using GPT
- Real-time video interview with AI proctor
- Mobile app (React Native)

## Common Pitfalls

1. **Resume Parsing Accuracy:** Different formats are challenging. Test extensively with various resume styles.

2. **Data Privacy:** Handle candidate data carefully. Implement GDPR compliance (consent, right to deletion).

3. **Bias in AI:** ML models can perpetuate bias. Regularly audit algorithms for fairness.

4. **Video File Size:** Videos are large. Use compression and cloud storage (S3, Cloudinary).

5. **Real-time Processing:** Video analysis is compute-intensive. Use background jobs (Bull, Celery).

## Resources

### Documentation
- [spaCy NLP](https://spacy.io/)
- [Hugging Face Transformers](https://huggingface.co/transformers/)
- [DeepFace](https://github.com/serengil/deepface)
- [Google Cloud Speech-to-Text](https://cloud.google.com/speech-to-text)

### Tutorials
- [Resume Parsing with Python](https://www.youtube.com/results?search_query=resume+parsing+python+nlp)
- [Building ATS Systems](https://www.youtube.com/results?search_query=applicant+tracking+system+tutorial)

### Design Inspiration
- [Lever](https://www.lever.co/)
- [Greenhouse](https://www.greenhouse.io/)
- [HireVue](https://www.hirevue.com/)

## Submission Checklist

- [ ] Resume parsing and information extraction
- [ ] Candidate-job matching algorithm
- [ ] ATS workflow (Applied → Hired/Rejected)
- [ ] Video interview upload and analysis
- [ ] Speech transcription
- [ ] Facial expression analysis
- [ ] Confidence scoring
- [ ] Recruiter dashboard with analytics
- [ ] Email notifications
- [ ] Search and filtering
- [ ] Responsive design
- [ ] Documentation

## Portfolio Tips

- **Show AI in Action:** Demo video analysis with confidence scores
- **Highlight Accuracy:** "Resume parsing with 95% accuracy across 10,000 resumes"
- **Explain Algorithms:** Detail matching algorithm (TF-IDF, cosine similarity, weighted scoring)
- **Show Fairness:** Describe bias detection and mitigation strategies
- **Compare to Industry:** "Built similar to Lever/Greenhouse ATS architecture"
- **Technical Depth:** Discuss NLP pipeline, model selection, performance optimization

---

This project is part of the [Coding Projects Library](https://github.com/Ryanditko/biblioteca-de-projetos) and is licensed under the MIT License.
