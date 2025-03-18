layout: post
title: AP Computer Science Principles
permalink: classes/ap/csp/quizz
---

<div class="quiz-page" style="font-family: Arial, sans-serif; background: linear-gradient(to bottom, #E3F2FD, #BBDEFB); min-height: 100vh; padding: 20px; box-sizing: border-box;">
  <h1 style="color: #1E88E5; text-align: center; font-size: 3em; animation: fadeIn 1s;">AP CSP Quiz</h1>

  <!-- User Name Input -->
  <div style="text-align: center; margin-bottom: 20px;">
    <label for="nameInput" style="color: #1565C0; font-size: 1.2em;">Enter Your Name:</label>
    <input id="nameInput" type="text" placeholder="Your Name" style="padding: 10px; font-size: 1em; margin-top: 10px; border-radius: 10px; border: 1px solid #1565C0;">
  </div>

  <form id="quiz-form" style="max-width: 700px; margin: 20px auto; background-color: #E3F2FD; padding: 20px; border-radius: 15px; box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.2); animation: slideIn 2s;">
    <div id="quiz-questions">
      <!-- Placeholder for dynamically injected quiz questions -->
    </div>
    <button type="submit" style="background: linear-gradient(45deg, #1E88E5, #42A5F5); border: none; color: white; padding: 15px 30px; font-size: 1.5em; border-radius: 50px; cursor: pointer; box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.3); transition: transform 0.2s, box-shadow 0.2s; margin-top: 20px;">
      Submit Quiz
    </button>
  </form>

  <p id="quiz-result" style="color: #1565C0; font-size: 1.5em; text-align: center; margin-top: 20px;"></p>

  <!-- Leaderboard Section -->
  <div id="leaderboard" style="max-width: 700px; margin: 40px auto; background: #1E88E5; padding: 20px; border-radius: 15px; box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.2); animation: fadeIn 1s;">
    <h2 style="color: #E3F2FD; text-align: center; font-size: 2em;">Leaderboard</h2>
    <ul id="leaderboard-list" style="list-style-type: none; padding: 0; font-size: 1.2em; color: #0D47A1; text-align: center;"></ul>
  </div>

  <div style="text-align: center; margin-top: 20px;">
    <button onclick="returnToHome()" style="background: linear-gradient(45deg, #1E88E5, #42A5F5); border: none; color: white; padding: 10px 20px; font-size: 1.2em; border-radius: 30px; cursor: pointer; box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.3); transition: transform 0.2s, box-shadow 0.2s;">Return to Home</button>
  </div>
</div>

<script>
  let questions = [];

  // Fetch quiz questions from the backend
  async function fetchQuestions() {
    try {
      const response = await fetch('http://localhost:5000/api/quiz/csp');
      if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);

      const data = await response.json();
      if (!Array.isArray(data) || data.length === 0) throw new Error("No questions found.");

      questions = data;
      const quizContainer = document.getElementById('quiz-questions');
      quizContainer.innerHTML = '';

      data.forEach((question, index) => {
        const questionDiv = document.createElement('div');
        questionDiv.className = 'quiz-question';
        questionDiv.style = "margin-bottom: 20px; padding: 15px; background-color: #E3F2FD; border-radius: 10px;";
        questionDiv.innerHTML = `
          <p style="color: #1E88E5; font-size: 1.2em; font-weight: bold;">${index + 1}. ${question.text}</p>
          ${question.options.map((option, idx) => `
            <label style="display: block; margin: 5px 0; color: #0D47A1;">
              <input type="radio" name="q${index}" value="${option}" style="margin-right: 10px;">${option}
            </label>
          `).join('')}
        `;
        quizContainer.appendChild(questionDiv);
      });
    } catch (error) {
      console.error('Error fetching questions:', error);
      document.getElementById('quiz-questions').innerHTML = `<p style="color: red;">Failed to load questions.</p>`;
    }
  }

  // Submit the quiz and display the score
  async function submitQuiz(event) {
    event.preventDefault();

    const nameInput = document.getElementById('nameInput');
    const userName = nameInput.value.trim();

    if (!userName) {
      alert("Please enter your name before submitting the quiz.");
      return;
    }

    const formData = new FormData(document.getElementById('quiz-form'));
    const answers = questions.map((q, index) => ({ questionId: q.id, answer: formData.get(`q${index}`) }));
    if (answers.some(a => !a.answer)) return alert("Please answer all questions.");

    try {
      const response = await fetch('http://localhost:5000/api/quiz/csp/submit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: userName, answers })
      });
      const result = await response.json();

      const resultElement = document.getElementById('quiz-result');
      resultElement.textContent = `${userName}, your score: ${result.score}/10`;
      resultElement.style.animation = "popIn 1s";
      fetchLeaderboard();
    } catch (error) {
      console.error('Error submitting quiz:', error);
    }
  }

  async function fetchLeaderboard() {
    try {
      const response = await fetch('http://localhost:5000/api/leaderboard/csp');
      if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);

      const data = await response.json();
      const leaderboardList = document.getElementById('leaderboard-list');
      leaderboardList.innerHTML = '';

      data.forEach((entry, index) => {
        const li = document.createElement('li');
        li.textContent = `${index + 1}. ${entry.name} - Score: ${entry.score}`;
        leaderboardList.appendChild(li);
      });
    } catch (error) {
      console.error('Error fetching leaderboard:', error);
      document.getElementById('leaderboard-list').innerHTML = `<li style="color: red;">Failed to load leaderboard.</li>`;
    }
  }

  function returnToHome() {
    if (confirm("Are you sure you want to leave?")) {
      window.location.href = '{{site.baseurl}}/classes/ap/csp/home';
    }
  }

  document.getElementById('quiz-form').addEventListener('submit', submitQuiz);
  fetchQuestions();
  fetchLeaderboard();
</script>

<style>
  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  @keyframes popIn {
    0% { transform: scale(0.5); opacity: 0; }
    100% { transform: scale(1); opacity: 1; }
  }
  button:hover {
    transform: scale(1.05);
    box-shadow: 0px 6px 15px rgba(0, 0, 0, 0.4);
  }
</style>



# test
