<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- جدار الحماية: منع تشغيل أي كود خارجي غير آمن -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
    <title>محرك الاختبارات الذكي | آمن ومباشر</title>
    <style>
        /* التصميم الأساسي والآمن */
        :root {
            --bg-color: #f0f2f5;
            --text-color: #333;
            --card-bg: #fff;
            --btn-color: #2563eb;
            --correct-bg: #d1fae5;
            --correct-text: #065f46;
            --wrong-bg: #fee2e2;
            --wrong-text: #991b1b;
        }

        /* الوضع الليلي */
        .dark-mode {
            --bg-color: #111827;
            --text-color: #f3f4f6;
            --card-bg: #1f2937;
            --btn-color: #3b82f6;
            --correct-bg: #064e3b;
            --correct-text: #6ee7b7;
            --wrong-bg: #7f1d1d;
            --wrong-text: #fca5a5;
        }

        body {
            font-family: system-ui, -apple-system, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            transition: background-color 0.3s, color 0.3s;
        }

        .container {
            background-color: var(--card-bg);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            width: 90%;
            max-width: 600px;
            transition: background-color 0.3s;
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 25px;
        }

        .timer {
            font-size: 28px;
            font-weight: bold;
            color: #ef4444;
            font-variant-numeric: tabular-nums;
        }

        .theme-btn {
            background: transparent;
            border: 1px solid var(--text-color);
            color: var(--text-color);
            padding: 8px 16px;
            border-radius: 20px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }

        .theme-btn:hover {
            opacity: 0.7;
        }

        .progress-text {
            color: #6b7280;
            text-align: center;
            margin-bottom: 20px;
        }

        .question-box {
            font-size: 1.4rem;
            font-weight: bold;
            margin-bottom: 25px;
            text-align: right;
            line-height: 1.6;
        }

        .options {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .option {
            background-color: var(--bg-color);
            padding: 15px;
            border-radius: 10px;
            cursor: pointer;
            border: 2px solid transparent;
            transition: all 0.2s;
            text-align: right;
            font-weight: 500;
        }

        .option:hover:not(.disabled) {
            border-color: var(--btn-color);
            transform: translateX(4px);
        }

        .option.correct {
            background-color: var(--correct-bg);
            color: var(--correct-text);
            border-color: var(--correct-text);
            pointer-events: none;
        }

        .option.wrong {
            background-color: var(--wrong-bg);
            color: var(--wrong-text);
            border-color: var(--wrong-text);
            pointer-events: none;
        }

        .option.disabled {
            pointer-events: none;
        }

        #explanation-box {
            margin-top: 20px;
            padding: 15px;
            border-radius: 10px;
            background-color: var(--bg-color);
            border-right: 5px solid var(--btn-color);
            display: none;
            text-align: right;
        }

        #explanation-box.show { display: block; }

        .next-btn {
            width: 100%;
            background-color: var(--btn-color);
            color: white;
            border: none;
            padding: 15px;
            border-radius: 10px;
            font-size: 1.1rem;
            font-weight: bold;
            cursor: pointer;
            margin-top: 20px;
            display: none;
            transition: 0.2s;
        }

        .next-btn.show { display: block; }
        .next-btn:active { transform: scale(0.98); }

        .reset-btn {
            width: 100%;
            background: transparent;
            border: 2px solid var(--btn-color);
            color: var(--btn-color);
            padding: 15px;
            border-radius: 10px;
            font-size: 1.1rem;
            font-weight: bold;
            cursor: pointer;
            margin-top: 10px;
            display: none;
        }

        .reset-btn.show { display: block; }
        
        /* تصحيح الشاشات الصغيرة */
        @media (max-width: 480px) {
            .container { padding: 20px; }
            .question-box { font-size: 1.2rem; }
        }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <div class="timer" id="timerDisplay">01:00:00</div>
        <button class="theme-btn" id="themeBtn" onclick="toggleTheme()">🌙 الوضع الليلي</button>
    </div>

    <div class="progress-text" id="progressText">السؤال 1 من 0</div>
    
    <div class="question-box" id="questionText">انقر "ابدأ الاختبار" لبدء التحدي</div>
    
    <div class="options" id="optionsContainer"></div>

    <div id="explanation-box">
        <strong id="resultIcon"></strong> <span id="explanationText"></span>
    </div>

    <button class="next-btn" id="nextBtn" onclick="nextQuestion()">السؤال التالي ➜</button>
    <button class="reset-btn" id="resetBtn" onclick="location.reload()">🔄 إعادة الاختبار</button>
</div>

<script>
    /* 1. أسئلتك الخاصة - انسخ هذا القالب واملأه بأسئلتك */
    const questions = [
        {
            question: "ما هي العملة الرسمية للمملكة العربية السعودية؟",
            options: ["الدولار", "الريال السعودي", "الدرهم", "الدينار"],
            correct: 1,
            explanation: "الريال السعودي هو العملة الرسمية للمملكة."
        },
        {
            question: "كم عدد أركان الإسلام؟",
            options: ["3", "4", "5", "6"],
            correct: 2,
            explanation: "أركان الإسلام خمسة: الشهادتان، والصلاة، والزكاة، والصوم، والحج."
        },
        {
            question: "أي من هذه المدن تقع في المملكة العربية السعودية؟",
            options: ["القاهرة", "الرياض", "بيروت", "دمشق"],
            correct: 1,
            explanation: "الرياض هي عاصمة المملكة العربية السعودية."
        }
    ];

    /* 2. إعدادات الاختبار */
    let currentIndex = 0;
    let score = 0;
    let timeLeft = 3600; // 60 دقيقة
    let timerInterval;
    let isAnswered = false;
    let isQuizStarted = false;

    /* 3. التايمر */
    function startTimer() {
        if (isQuizStarted) return;
        isQuizStarted = true;
        
        timerInterval = setInterval(() => {
            timeLeft--;
            const hours = Math.floor(timeLeft / 3600);
            const minutes = Math.floor((timeLeft % 3600) / 60);
            const seconds = timeLeft % 60;
            
            document.getElementById('timerDisplay').innerText = 
                `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
            
            if (timeLeft <= 0) {
                clearInterval(timerInterval);
                endQuiz("انتهى الوقت!");
            }
        }, 1000);
    }

    /* 4. تحميل الأسئلة */
    function loadQuestion() {
        if (currentIndex >= questions.length) {
            endQuiz("انتهى الاختبار!");
            return;
        }

        const q = questions[currentIndex];
        document.getElementById('progressText').innerText = `السؤال ${currentIndex + 1} من ${questions.length}`;
        document.getElementById('questionText').innerText = q.question;
        document.getElementById('explanation-box').classList.remove('show');
        document.getElementById('nextBtn').classList.remove('show');
        isAnswered = false;

        const container = document.getElementById('optionsContainer');
        container.innerHTML = '';
        
        q.options.forEach((text, idx) => {
            const btn = document.createElement('div');
            btn.className = 'option';
            btn.innerText = text;
            btn.onclick = () => checkAnswer(idx, q);
            container.appendChild(btn);
        });
    }

    /* 5. التصحيح */
    function checkAnswer(selectedIdx, q) {
        if (isAnswered) return;
        isAnswered = true;
        startTimer();

        const allOptions = document.querySelectorAll('.option');
        allOptions.forEach((opt, idx) => {
            opt.classList.add('disabled');
            if (idx === q.correct) opt.classList.add('correct');
            else if (idx === selectedIdx && idx !== q.correct) opt.classList.add('wrong');
        });

        const expBox = document.getElementById('explanation-box');
        const expText = document.getElementById('explanationText');
        const resIcon = document.getElementById('resultIcon');

        expBox.classList.add('show');
        if (selectedIdx === q.correct) {
            score++;
            resIcon.innerText = "✅ إجابة صحيحة! ";
            expBox.style.borderRightColor = "#10b981";
            expBox.style.backgroundColor = "var(--correct-bg)";
        } else {
            resIcon.innerText = "❌ إجابة خاطئة! ";
            expBox.style.borderRightColor = "#ef4444";
            expBox.style.backgroundColor = "var(--wrong-bg)";
        }
        expText.innerText = q.explanation;
        
        document.getElementById('nextBtn').classList.add('show');
    }

    /* 6. الانتقال للأسئلة التالية */
    function nextQuestion() {
        currentIndex++;
        loadQuestion();
    }

    /* 7. إنهاء الاختبار */
    function endQuiz(message) {
        clearInterval(timerInterval);
        document.getElementById('questionText').innerHTML = `🏁 ${message}`;
        document.getElementById('optionsContainer').innerHTML = '';
        document.getElementById('explanation-box').classList.remove('show');
        document.getElementById('nextBtn').classList.remove('show');
        document.getElementById('timerDisplay').innerText = "00:00:00";
        
        const percentage = Math.round((score / questions.length) * 100);
        document.getElementById('progressText').innerHTML = `
            <h2 style="margin-bottom:10px;">النتيجة النهائية</h2>
            <p>لقد أجبت على <strong>${score}</strong> من <strong>${questions.length}</strong> بشكل صحيح.</p>
            <p style="font-size:2.5rem; font-weight:bold; color:var(--btn-color); margin:10px 0;">${percentage}%</p>
        `;
        
        document.getElementById('resetBtn').classList.add('show');
    }

    /* 8. تبديل الوضع ليلي/نهاري */
    function toggleTheme() {
        document.body.classList.toggle('dark-mode');
        const btn = document.getElementById('themeBtn');
        if (document.body.classList.contains('dark-mode')) {
            btn.innerText = '☀️ النهاري';
        } else {
            btn.innerText = '🌙 الليلي';
        }
    }

    /* 9. بدء الاختبار */
    loadQuestion();
</script>
</body>
</html>
