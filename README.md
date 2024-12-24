from flask import Flask, request, jsonify
import sqlite3
from datetime import datetime

app = Flask(__name__)

# Initialize database
def create_database():
    conn = sqlite3.connect("mood_tracker.db")
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS mood_entries (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            mood TEXT,
            reason TEXT,
            timestamp TEXT
        )
    ''')
    conn.commit()
    conn.close()

create_database()

# Add mood entry
@app.route("/api/moods", methods=["POST"])
def add_mood():
    data = request.json
    mood = data.get("mood")
    reason = data.get("note", "")
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    conn = sqlite3.connect("mood_tracker.db")
    cursor = conn.cursor()
    cursor.execute("INSERT INTO mood_entries (mood, reason, timestamp) VALUES (?, ?, ?)", (mood, reason, timestamp))
    conn.commit()
    conn.close()
    return jsonify({"message": "Mood added successfully!"}), 201

# Get all mood entries
@app.route("/api/moods", methods=["GET"])
def get_moods():
    conn = sqlite3.connect("mood_tracker.db")
    cursor = conn.cursor()
    cursor.execute("SELECT mood, reason, timestamp FROM mood_entries ORDER BY timestamp DESC")
    entries = cursor.fetchall()
    conn.close()
    return jsonify([{"mood": row[0], "reason": row[1], "timestamp": row[2]} for row in entries])

if __name__ == "__main__":
    app.run(debug=True)
