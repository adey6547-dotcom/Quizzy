

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mastery Quiz & Certification</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        :root { --primary: #2563eb; --success: #16a34a; --dark: #1e293b; --light: #f8fafc; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #e2e8f0; margin: 0; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        .container { background: white; width: 90%; max-width: 800px; padding: 2rem; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); text-align: center; }
        .hidden { display: none; }
        .btn { background: var(--primary); color: white; border: none; padding: 12px 24px; border-radius: 8px; cursor: pointer; font-size: 1rem; margin-top: 1rem; transition: 0.3s; }
        .btn:hover { opacity: 0.9; transform: translateY(-2px); }
        .qr-section img { width: 200px; border: 5px solid #fff; border-radius: 10px; margin-bottom: 1rem; }
        .quiz-header { display: flex; justify-content: space-between; margin-bottom: 2rem; font-weight: bold; color: var(--dark); }
        .option-btn { display: block; width: 100%; text-align: left; padding: 15px; margin: 10px 0; border: 1px solid #cbd5e1; border-radius: 8px; cursor: pointer; transition: background 0.2s; }
        .option-btn:hover { background: #f1f5f9; }
        .certificate-box { border: 10px double var(--primary); padding: 40px; background: #fff; position: relative; }
        #timer { color: #dc2626; }
    </style>
</head>
<body>

    <div class="container" id="app">
        <!-- Section 1: Payment Gate -->
        <div id="payment-view">
            <h1>Unlock the Certification Quiz</h1>
            <p>Pay ₹1.00 to start your examination and receive a verified certificate.</p>
            <div class="qr-section">
                <!-- Generating a dynamic UPI QR Link -->
                <img src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=upi://pay?pa=adey6547@axl&pn=AkashDey&am=1&cu=INR" alt="UPI QR Code">
                <p><strong>UPI ID:</strong> adey6547@axl</p>
            </div>
            <button class="btn" onclick="startQuiz()">I Have Paid - Start Quiz</button>
        </div>

        <!-- Section 2: Quiz Interface -->
        <div id="quiz-view" class="hidden">
            <div class="quiz-header">
                <span>Question: <span id="q-count">1</span>/10</span>
                <span>Time Left: <span id="timer">30</span>s</span>
            </div>
            <h2 id="question-text">Loading question...</h2>
            <div id="options-container"></div>
        </div>

        <!-- Section 3: Report Card & Certificate -->
        <div id="result-view" class="hidden">
            <h1>Quiz Report Card</h1>
            <div id="report-stats" style="margin-bottom: 2rem; font-size: 1.2rem;"></div>
            
            <div id="certificate" class="certificate-box">
                <h2 style="color: var(--primary); font-size: 2rem;">CERTIFICATE OF EXCELLENCE</h2>
                <p>This is to certify that</p>
                <h3 style="text-decoration: underline;">The Participant</h3>
                <p>has successfully completed the <strong>Academic Core Subject Quiz</strong></p>
                <p>with a score of <span id="final-score-disp"></span>%</p>
                <p><small>Issued on: <span id="issue-date"></span></small></p>
            </div>
            <button class="btn" onclick="downloadPDF()">Download Certificate (PDF)</button>
        </div>
    </div>

    <script>
        const questions = [
            { q: "What is the 'brain' of the computer?", a: ["CPU", "RAM", "Hard Drive", "GPU"], c: 0 },
            { q: "What does HTML stand for?", a: ["HyperText Markup Language", "HighText Machine Language", "Hyperlink Text Mode", "None"], c: 0 },
            { q: "What is the value of Pi (π) to two decimal places?", a: ["3.12", "3.14", "3.16", "3.18"], c: 1 },
            { q: "What is the smallest prime number?", a: ["0", "1", "2", "3"], c: 2 },
            { q: "What is the unit of Force?", a: ["Joule", "Watt", "Newton", "Pascal"], c: 2 },
            { q: "What is the chemical symbol for Gold?", a: ["Ag", "Fe", "Au", "Pb"], c: 2 },
            { q: "What is the pH of pure water?", a: ["5", "7", "9", "11"], c: 1 },
            { q: "What is the 'Powerhouse of the cell'?", a: ["Nucleus", "Mitochondria", "Ribosome", "Wall"], c: 1 },
            { q: "Which planet is known as the Red Planet?", a: ["Venus", "Mars", "Jupiter", "Saturn"], c: 1 },
            { q: "What is the derivative of x^n?", a: ["nx", "n(x-1)", "nx^(n-1)", "x^(n+1)"], c: 2 }
        ];

        let currentIdx = 0;
        let score = 0;
        let timeLeft = 30;
        let timerId;

        function startQuiz() {
            document.getElementById('payment-view').classList.add('hidden');
            document.getElementById('quiz-view').classList.remove('hidden');
            loadQuestion();
            startTimer();
        }

        function startTimer() {
            timerId = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = timeLeft;
                if (timeLeft <= 0) nextQuestion(false);
            }, 1000);
        }

        function loadQuestion() {
            const q = questions[currentIdx];
            document.getElementById('q-count').innerText = currentIdx + 1;
            document.getElementById('question-text').innerText = q.q;
            const container = document.getElementById('options-container');
            container.innerHTML = '';
            q.a.forEach((opt, i) => {
                const btn = document.createElement('button');
                btn.className = 'option-btn';
                btn.innerText = opt;
                btn.onclick = () => checkAnswer(i);
                container.appendChild(btn);
            });
        }

        function checkAnswer(selectedIdx) {
            if (selectedIdx === questions[currentIdx].c) score++;
            nextQuestion();
        }

        function nextQuestion() {
            currentIdx++;
            if (currentIdx < questions.length) {
                timeLeft = 30;
                loadQuestion();
            } else {
                showResults();
            }
        }

        function showResults() {
            clearInterval(timerId);
            document.getElementById('quiz-view').classList.add('hidden');
            document.getElementById('result-view').classList.remove('hidden');
            
            const percent = (score / questions.length) * 100;
            document.getElementById('report-stats').innerHTML = `Correct Answers: ${score} <br> Grade: ${percent >= 50 ? 'PASS' : 'FAIL'}`;
            document.getElementById('final-score-disp').innerText = percent;
            document.getElementById('issue-date').innerText = new Date().toLocaleDateString();
        }

        function downloadPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('l', 'mm', 'a4');
            const cert = document.getElementById('certificate');
            doc.text("CERTIFICATE OF COMPLETION", 105, 40, { align: "center" });
            doc.text("Score: " + score + "/10", 105, 60, { align: "center" });
            doc.save("Quiz_Certificate.pdf");
        }
    </script>
</body>
</html>
