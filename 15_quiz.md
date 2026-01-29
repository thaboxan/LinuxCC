# Linux Quiz

Test your Linux knowledge with these interactive questions. Type your answer and check if you're correct!

<style>
.quiz-container {
  max-width: 700px;
  margin: 20px auto;
}
.quiz-scoreboard {
  background: linear-gradient(135deg, #2c3e50, #34495e);
  color: white;
  padding: 15px 20px;
  border-radius: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}
.quiz-scoreboard span { font-size: 18px; }
.quiz-reset {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  cursor: pointer;
}
.quiz-reset:hover { background: #c0392b; }
.quiz-topic-selector {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px 20px;
  background: linear-gradient(135deg, #3498db, #2980b9);
  border-radius: 10px;
  color: white;
}
.quiz-topic-selector label {
  font-weight: bold;
  font-size: 16px;
}
.quiz-topic-selector select {
  padding: 10px 15px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  background: white;
  color: #2c3e50;
  cursor: pointer;
  min-width: 180px;
}
.quiz-topic-selector select:focus {
  outline: 2px solid #e95420;
}
#quiz-count {
  background: rgba(255,255,255,0.2);
  padding: 5px 12px;
  border-radius: 15px;
  font-size: 14px;
}
#quiz-box {
  background: #f8f9fa;
  border: 2px solid #e1e4e8;
  border-radius: 12px;
  padding: 25px;
}
.quiz-badge {
  display: inline-block;
  color: white;
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 15px;
  text-transform: uppercase;
}
.quiz-badge.easy { background: #27ae60; }
.quiz-badge.medium { background: #f39c12; }
.quiz-badge.hard { background: #e74c3c; }
#quiz-question {
  color: #2c3e50;
  font-size: 20px;
  margin: 0 0 10px 0;
  line-height: 1.5;
}
#quiz-hint {
  color: #7f8c8d;
  font-size: 14px;
  font-style: italic;
  margin-bottom: 20px;
}
#quiz-input {
  width: 100%;
  padding: 15px;
  font-size: 18px;
  font-family: 'Consolas', 'Monaco', monospace;
  border: 2px solid #ddd;
  border-radius: 8px;
  box-sizing: border-box;
  margin-bottom: 15px;
}
#quiz-input:focus {
  outline: none;
  border-color: #e95420;
  box-shadow: 0 0 0 3px rgba(233, 84, 32, 0.1);
}
.quiz-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}
.quiz-buttons button {
  padding: 12px 24px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}
.quiz-btn-submit {
  background: linear-gradient(135deg, #27ae60, #1e8449);
  color: white;
  flex: 1;
}
.quiz-btn-submit:hover {
  background: linear-gradient(135deg, #2ecc71, #27ae60);
  transform: translateY(-2px);
}
.quiz-btn-view {
  background: linear-gradient(135deg, #3498db, #2980b9);
  color: white;
}
.quiz-btn-view:hover {
  background: linear-gradient(135deg, #5dade2, #3498db);
  transform: translateY(-2px);
}
.quiz-btn-next {
  background: linear-gradient(135deg, #e95420, #c44218);
  color: white;
}
.quiz-btn-next:hover {
  background: linear-gradient(135deg, #eb6b3d, #e95420);
  transform: translateY(-2px);
}
.quiz-feedback {
  margin-top: 20px;
  padding: 15px;
  border-radius: 8px;
  font-size: 16px;
  display: none;
}
.quiz-feedback.correct {
  background: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
  display: block;
}
.quiz-feedback.incorrect {
  background: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  display: block;
}
#quiz-answer {
  margin-top: 15px;
  padding: 15px;
  background: #fff3cd;
  border: 1px solid #ffc107;
  border-radius: 8px;
  display: none;
}
#quiz-answer code {
  background: #2c3e50;
  color: #2ecc71;
  padding: 4px 10px;
  border-radius: 4px;
  font-size: 16px;
}
</style>

<div class="quiz-container">
  <div class="quiz-scoreboard">
    <span>Score: <strong id="quiz-score">0 / 0 (0%)</strong></span>
    <button class="quiz-reset" onclick="resetQuiz()">Reset Quiz</button>
  </div>
  
  <div class="quiz-topic-selector">
    <label for="quiz-topic">Topic:</label>
    <select id="quiz-topic" onchange="changeCategory()">
      <option value="all">All Topics</option>
    </select>
    <span id="quiz-count">50 questions</span>
  </div>
  
  <div id="quiz-box">
    <div id="quiz-category" class="quiz-badge easy"></div>
    <h3 id="quiz-question"></h3>
    <div id="quiz-hint"></div>
    <input type="text" id="quiz-input" placeholder="Type your answer here...">
    <div class="quiz-buttons">
      <button class="quiz-btn-submit" onclick="submitQuizAnswer()">Submit Answer</button>
      <button class="quiz-btn-view" onclick="viewQuizAnswer()">View Answer</button>
      <button class="quiz-btn-next" onclick="loadQuizQuestion()">Next Question</button>
    </div>
    <div id="quiz-feedback" class="quiz-feedback"></div>
    <div id="quiz-answer"></div>
  </div>
</div>

---

## Quiz Categories

| Category | Topics Covered |
|----------|---------------|
| File System | Navigation, paths, directories |
| Files | Copy, move, delete, create |
| Text Processing | grep, sed, awk, wc, head, tail |
| Permissions | chmod, chown, numeric permissions |
| Users | useradd, passwd, groups |
| Processes | ps, kill, top, signals |
| Networking | ping, ip, ss, firewall |
| Services | systemctl, journalctl |
| Packages | dnf, rpm |
| Storage | df, du, mount, lsblk |
| SELinux | getenforce, setenforce, restorecon |
| Containers | podman commands |
| Find | find command options |
| Archives | tar commands |

---

## Tips for Success

1. **Pay attention to case** - Linux commands are case-sensitive
2. **Think about common options** - Many commands accept similar flags (-h, -v, -r)
3. **Remember abbreviations** - Most commands are shortened words (pwd = print working directory)
4. **Practice in a terminal** - The best way to learn is hands-on
5. **Press Enter** - to submit your answer or go to the next question

---

**[Back to Index](README.md)** | **[Practice Labs](14_practice_labs.md)**
