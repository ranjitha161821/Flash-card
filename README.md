from flask import Flask, request, jsonify
from datetime import datetime
import re

app = Flask(__name__)

# In-memory storage for flashcards (in a real system, use a database)
flashcards = []

# Subject classification rules
SUBJECT_KEYWORDS = {
    "Physics": ["newton", "force", "mass", "acceleration", "velocity", "energy", "quantum", "thermodynamics"],
    "Chemistry": ["atom", "molecule", "reaction", "chemical", "periodic table", "bond", "organic"],
    "Biology": ["cell", "photosynthesis", "dna", "organism", "evolution", "ecosystem", "mitochondria"],
    "Mathematics": ["equation", "algebra", "calculus", "derivative", "integral", "matrix", "probability"],
    "History": ["war", "century", "empire", "revolution", "ancient", "medieval", "colonial"],
    "Computer Science": ["programming", "algorithm", "database", "python", "java", "binary", "compiler"]
}

def classify_subject(text):
    text = text.lower()
    subject_scores = {subject: 0 for subject in SUBJECT_KEYWORDS}
    
    for subject, keywords in SUBJECT_KEYWORDS.items():
        for keyword in keywords:
            if re.search(r'\b' + re.escape(keyword) + r'\b', text):
                subject_scores[subject] += 1
                
    # Get the subject with the highest score
    best_subject = max(subject_scores.items(), key=lambda x: x[1])[0]
    return best_subject if subject_scores[best_subject] > 0 else "General"

@app.route('/flashcard', methods=['POST'])
def add_flashcard():
    data = request.json
    
    # Validate input
    if not all(key in data for key in ['student_id', 'question', 'answer']):
        return jsonify({"error": "Missing required fields"}), 400
    
    # Classify the subject
    question = data['question']
    answer = data['answer']
    combined_text = f"{question} {answer}"
    subject = classify_subject(combined_text)
    
    # Create and store the flashcard
    flashcard = {
        "student_id": data['student_id'],
        "question": question,
        "answer": answer,
        "subject": subject,
        "created_at": datetime.now().isoformat()
    }
    flashcards.append(flashcard)
    
    return jsonify({
        "message": "Flashcard added successfully",
        "subject": subject
    }), 201

if __name__ == '__main__':
    app.run(debug=True)


from transformers import pipeline

classifier = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")

def classify_subject_ml(text):
    candidate_labels = ["Physics", "Chemistry", "Biology", "Mathematics", "History", "Computer Science"]
    result = classifier(text, candidate_labels)
    return result['labels'][0]




    @app.route('/flashcards', methods=['GET'])
def get_flashcards():
    student_id = request.args.get('student_id')
    subject = request.args.get('subject')
    
    filtered = [fc for fc in flashcards if fc['student_id'] == student_id]
    if subject:
        filtered = [fc for fc in filtered if fc['subject'].lower() == subject.lower()]
    
    return jsonify(filtered), 200



    

