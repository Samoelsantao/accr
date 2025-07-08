<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Treinamento Interativo - Classificação de Risco ACCR Araucária</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Neutral Health -->
    <!-- Application Structure Plan: A single-page application designed as an interactive quiz. The structure is sequential: Start Screen -> Quiz View (one question at a time) -> Results Screen. This flow was chosen for effective self-paced learning, providing case-by-case scenarios with immediate feedback to reinforce the ACCR protocol's decision-making process, which is more engaging and practical for training than a static document. -->
    <!-- Visualization & Content Choices: Report Info: 30 case studies from the ACCR protocol. Goal: Train healthcare professionals. Viz/Presentation Method: An interactive quiz interface with multiple-choice buttons representing risk levels. Interaction: Users select a classification and receive immediate visual feedback (correct/incorrect) and a detailed textual justification. A final bar chart (Chart.js) visualizes the overall score. Justification: This interactive format directly simulates the classification task, enhancing knowledge retention. The final chart provides a clear performance summary. Library/Method: Vanilla JS for quiz logic, Tailwind for styling, Chart.js for the result visualization. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .quiz-card {
            transition: all 0.3s ease-in-out;
        }
        .answer-btn {
            transition: all 0.2s ease-in-out;
        }
        .correct {
            background-color: #22c55e !important; /* green-500 */
            color: white !important;
            border-color: #16a34a !important; /* green-600 */
            transform: scale(1.05);
        }
        .incorrect {
            background-color: #ef4444 !important; /* red-500 */
            color: white !important;
            border-color: #dc2626 !important; /* red-600 */
        }
        .feedback {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.5s ease-in-out, opacity 0.5s ease-in-out;
            opacity: 0;
        }
        .feedback.show {
            max-height: 500px;
            opacity: 1;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            margin-left: auto;
            margin-right: auto;
            height: 250px;
            max-height: 300px;
        }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 flex items-center justify-center min-h-screen p-4">

    <main id="app-container" class="w-full max-w-2xl mx-auto">
        
        <!-- Start Screen -->
        <div id="start-screen" class="bg-white p-8 rounded-2xl shadow-lg text-center quiz-card">
            <h1 class="text-2xl md:text-3xl font-bold text-slate-900 mb-2">Treinamento Interativo ACCR</h1>
            <h2 class="text-lg font-semibold text-cyan-700 mb-4">Protocolo de Acolhimento com Classificação de Risco - Araucária/PR</h2>
            <p class="text-slate-600 mb-6">
                Bem-vindo ao treinamento prático do Protocolo de Acolhimento com Classificação de Risco. Você será apresentado a 30 estudos de caso para testar e aprimorar sua habilidade de classificação. O objetivo é fortalecer a tomada de decisão rápida e segura no atendimento de urgência.
            </p>
            <button id="start-btn" class="w-full bg-cyan-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-cyan-700 focus:outline-none focus:ring-2 focus:ring-cyan-500 focus:ring-opacity-50 transition-transform transform hover:scale-105">
                Iniciar Treinamento
            </button>
        </div>

        <!-- Quiz Screen -->
        <div id="quiz-screen" class="hidden bg-white p-6 md:p-8 rounded-2xl shadow-lg quiz-card">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-slate-900">Estudo de Caso</h2>
                <span id="question-counter" class="text-sm font-semibold text-slate-500 bg-slate-200 px-3 py-1 rounded-full"></span>
            </div>
            <p id="question-text" class="text-lg text-slate-700 mb-6 min-h-[100px]"></p>
            
            <div id="answer-buttons" class="grid grid-cols-1 gap-3">
                <!-- Buttons will be dynamically inserted here -->
            </div>

            <div id="feedback" class="feedback mt-6">
                <div class="border-t pt-4">
                    <h3 class="text-lg font-bold text-slate-900 mb-2">Justificativa:</h3>
                    <p id="feedback-text" class="text-slate-600"></p>
                </div>
            </div>

            <button id="next-btn" class="hidden w-full mt-6 bg-cyan-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-cyan-700 focus:outline-none focus:ring-2 focus:ring-cyan-500 focus:ring-opacity-50">
                Próximo Caso
            </button>
        </div>

        <!-- Results Screen -->
        <div id="results-screen" class="hidden bg-white p-8 rounded-2xl shadow-lg text-center quiz-card">
            <h1 class="text-2xl md:text-3xl font-bold text-slate-900 mb-2">Treinamento Concluído!</h1>
            <p class="text-slate-600 mb-4">
                Você finalizou a avaliação dos 30 estudos de caso. Confira seu desempenho abaixo. A prática contínua é a chave para a excelência na classificação de risco.
            </p>
            <p class="text-xl font-medium text-slate-800 mb-6">Sua pontuação: <span id="score-text" class="font-bold"></span></p>
            
            <div class="chart-container mb-6">
                <canvas id="results-chart"></canvas>
            </div>

            <button id="restart-btn" class="w-full bg-cyan-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-cyan-700 focus:outline-none focus:ring-2 focus:ring-cyan-500 focus:ring-opacity-50 transition-transform transform hover:scale-105">
                Refazer Treinamento
            </button>
        </div>

    </main>

    <script>
        const quizData = [
            { question: "Um paciente de 45 anos chega à UPA após uma agressão, apresentando hemorragia exanguinante. Qual a classificação de risco e a conduta a ser adotada?", options: ["Não Urgente", "Pouco Urgente", "Urgente", "Emergência (Vermelho)"], correct: 3, justification: "De acordo com o Quadro 4 de Agressão e a seção 6 do protocolo, Hemorragia exanguinante é um critério para classificação Vermelha (Emergência), exigindo atendimento imediato na sala de estabilização." },
            { question: "Uma criança de 2 anos chega à UPA com choro prolongado e ininterrupto, e a mãe relata que não consegue alimentá-la. Qual a classificação de risco e a conduta recomendada?", options: ["Não Urgente", "Urgente (Amarelo)", "Muito Urgente", "Emergência"], correct: 1, justification: "Conforme o Quadro 9 de Bebê Chorando, 'choro prolongado ou ininterrupto' e 'incapaz de se alimentar' são critérios para classificação Amarela (Urgente), com tempo de espera de até 60 minutos." },
            { question: "Um adulto chega com dor abdominal intensa e sangramento vaginal, com mais de 20 semanas de gravidez. Qual a classificação de risco para este caso?", options: ["Não Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Emergência"], correct: 2, justification: "De acordo com o Quadro 22 de Dor Abdominal em Adulto, dor intensa e sangramento vaginal em gestante acima de 20 semanas são critérios para Muito Urgente (Laranja)." },
            { question: "Um paciente chega com história de queda e apresenta deformidade grosseira em um membro. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Urgente (Amarelo)", "Emergência"], correct: 2, justification: "Conforme o Quadro 47 de Quedas, uma deformidade grosseira é classificada como Urgente (Amarelo), indicando a necessidade de atenção em até 60 minutos." },
            { question: "Um paciente com asma relata estar com saturação de oxigênio muito baixa. Qual é a classificação de risco apropriada?", options: ["Normal", "Pouco Urgente", "Muito Urgente (Laranja)", "Urgente"], correct: 2, justification: "De acordo com o Quadro 7 de Asma, 'Sat O2 muito baixa' é um critério para classificação Laranja (Muito Urgente)." },
            { question: "Um bebê chorando apresenta púrpura e prostração/hipotonia. Qual a classificação de risco indicada?", options: ["Não Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Emergência"], correct: 2, justification: "Conforme o Quadro 9 de Bebê Chorando, púrpura e prostração/hipotonia são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente diabético chega com hipoglicemia e convulsões. Qual a prioridade de atendimento?", options: ["Urgente", "Muito Urgente", "Emergência (Vermelho)", "Pouco Urgente"], correct: 2, justification: "De acordo com o Quadro 11 de Convulsões, hipoglicemia e convulsões são critérios para Emergência (Vermelho), exigindo atendimento imediato." },
            { question: "Uma criança apresenta febre de 39,5°C e erupção cutânea fixa. Qual a classificação de risco e o tempo de espera aproximado?", options: ["Não Urgente", "Pouco Urgente", "Urgente", "Muito Urgente (Laranja)"], correct: 3, justification: "De acordo com o Quadro 30 de Erupção Cutânea e a Tabela 1 de Temperatura Infantil, uma criança com febre de 39,5°C é 'Muito Quente' e erupção cutânea fixa é critério para Muito Urgente (Laranja)." },
            { question: "Um adulto sofreu uma queda e apresenta dor leve e inchaço. Qual a classificação de risco?", options: ["Urgente", "Muito Urgente", "Pouco Urgente (Verde)", "Emergência"], correct: 2, justification: "De acordo com o Quadro 47 de Quedas, 'dor leve recente' e 'inchaço' são critérios para classificação Verde (Pouco Urgente)." },
            { question: "Um paciente chega à UPA com dor de garganta, febre e relata um evento recente. A temperatura aferida é de 38.0°C. Qual a classificação de risco e a conduta?", options: ["Emergência", "Muito Urgente", "Urgente (Amarelo)", "Pouco Urgente"], correct: 2, justification: "De acordo com o Quadro 25 de Dor de Garganta e a Tabela 2 de Temperatura Adulto, 'Febril' (37.5-38.4°C) e 'Evento recente' com dor de garganta se enquadram na classificação Amarela (Urgente)." },
            { question: "Um paciente com alteração do comportamento apresenta alto risco de agredir os outros. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Urgente"], correct: 2, justification: "De acordo com o Quadro 6 de Alteração do Comportamento, 'Alto risco de agredir os outros' é um critério para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente com suspeita de overdose apresenta alteração súbita de consciência e pulso anormal. Qual a classificação de risco e a conduta?", options: ["Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Não Urgente"], correct: 2, justification: "De acordo com o Quadro 38 de Overdose e Envenenamento, 'alteração súbita de consciência' e 'pulso anormal' são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente adulto com diabetes apresenta hiperglicemia com cetose. Qual a classificação de risco?", options: ["Emergência", "Urgente", "Muito Urgente (Laranja)", "Pouco Urgente"], correct: 2, justification: "De acordo com o Quadro 16 de Diabetes, 'hiperglicemia com cetose' é um critério para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente com dor lombar apresenta déficit neurológico novo e incapacidade de andar. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Urgente (Amarelo)", "Emergência"], correct: 2, justification: "De acordo com o Quadro 26 de Dor Lombar, 'déficit neurológico novo' e 'incapaz de andar' são critérios para classificação Amarela (Urgente)." },
            { question: "Um paciente com mal estar em adulto apresenta pulso anormal e sinais de meningismo. Qual a classificação de risco?", options: ["Urgente", "Não Urgente", "Muito Urgente (Laranja)", "Pouco Urgente"], correct: 2, justification: "De acordo com o Quadro 35 de Mal Estar em Adulto, 'pulso anormal' e 'sinais de meningismo' são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Uma criança irritadiça que 'não se alimenta' e tem 'história discordante'. Qual a classificação de risco?", options: ["Normal", "Pouco Urgente", "Urgente (Amarelo)", "Muito Urgente"], correct: 2, justification: "De acordo com o Quadro 13 de Criança Irritadiça, 'não se alimenta' e 'história discordante' são critérios para classificação Amarela (Urgente)." },
            { question: "Um paciente com história de febre e cefaleia, apresentando erupção cutânea fixa. Qual a classificação de risco?", options: ["Não Urgente", "Urgente", "Muito Urgente (Laranja)", "Emergência"], correct: 2, justification: "De acordo com o Quadro 10 de Cefaleia, 'erupção cutânea fixa' e 'criança quente' (ou 'adulto muito quente') são critérios para Laranja (Muito Urgente). A febre indica temperatura elevada." },
            { question: "Um paciente apresenta diarreia e vômitos persistentes com sinais de desidratação. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Urgente (Amarelo)", "Muito Urgente"], correct: 2, justification: "De acordo com o Quadro 17 de Diarreia e/ou Vômitos, 'Sinais de desidratação' e 'Vômitos persistentes' são critérios para classificação Amarela (Urgente)." },
            { question: "Um adulto com dispneia apresenta dor precordial e saturação de O2 muito baixa. Qual a classificação de risco e a conduta?", options: ["Urgente", "Pouco Urgente", "Emergência (Vermelho)", "Não Urgente"], correct: 2, justification: "De acordo com o Quadro 18 de Dispneia em Adulto, 'dor precordial ou cardíaca' e 'Sat O2 muito baixa' são critérios para classificação Vermelha (Emergência)." },
            { question: "Um paciente com doença mental apresenta agitação psicomotora e comportamento conturbador. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Urgente (Amarelo)", "Muito Urgente"], correct: 2, justification: "De acordo com o Quadro 20 de Doença Mental, 'agitação psicomotora' e 'comportamento conturbador' são critérios para classificação Amarela (Urgente)." },
            { question: "Um paciente com ferida apresenta infecção local e inflamação local. Qual a classificação de risco?", options: ["Emergência", "Muito Urgente", "Urgente", "Pouco Urgente (Verde)"], correct: 3, justification: "De acordo com o Quadro 32 de Feridas, 'infecção local' e 'inflamação local' são critérios para classificação Verde (Pouco Urgente)." },
            { question: "Um paciente com dor testicular apresenta cólicas e vômitos persistentes. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Urgente (Amarelo)", "Muito Urgente"], correct: 2, justification: "De acordo com o Quadro 27 de Dor Testicular, 'cólicas' e 'vômitos persistentes' são critérios para classificação Amarela (Urgente)." },
            { question: "Um paciente com mordedura/picada apresenta envenenamento de alta mortalidade e frases entrecortadas. Qual a classificação de risco e a conduta?", options: ["Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Não Urgente"], correct: 2, justification: "De acordo com o Quadro 37 de Mordeduras e Picadas, 'envenenamento de alta mortalidade' e 'frases entrecortadas' são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente chega com problemas urinários, dor intensa e priaprismo. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Urgente"], correct: 2, justification: "De acordo com o Quadro 46 de Problemas Urinários, 'dor intensa' e 'priaprismo' são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente com queimaduras apresenta estridor e lesão por inalação. Qual a classificação de risco e a conduta?", options: ["Muito Urgente", "Urgente", "Emergência (Vermelho)", "Pouco Urgente"], correct: 2, justification: "De acordo com o Quadro 48 de Queimaduras, 'estridor' e 'lesão por inalação' são critérios para classificação Vermelha (Emergência)." },
            { question: "Um paciente com traumatismo cranioencefálico apresenta hemorragia exanguinante e convulsões. Qual a classificação de risco e a conduta?", options: ["Muito Urgente", "Urgente", "Emergência (Vermelho)", "Não Urgente"], correct: 2, justification: "De acordo com o Quadro 49 de Trauma Cranioencefálico, 'hemorragia exanguinante' e 'convulsionando' são critérios para classificação Vermelha (Emergência)." },
            { question: "Um paciente com trauma maior apresenta dispneia aguda e dor intensa. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Muito Urgente (Laranja)", "Urgente"], correct: 2, justification: "De acordo com o Quadro 50 de Trauma Maior, 'dispneia aguda' e 'dor intensa' são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente com hemorragia digestiva apresenta evacuação de sangue vivo ou escurecido e vômitos de sangue. Qual a classificação de risco?", options: ["Não Urgente", "Pouco Urgente", "Urgente", "Muito Urgente (Laranja)"], correct: 3, justification: "De acordo com o Quadro 33 de Hemorragia Digestiva, 'evacuação de sangue vivo ou escurecido' e 'vômitos de sangue' são critérios para classificação Laranja (Muito Urgente)." },
            { question: "Um paciente com problemas em face apresenta inchaço na face e febril (37.8°C). Qual a classificação de risco?", options: ["Não Urgente", "Muito Urgente", "Urgente", "Pouco Urgente (Verde)"], correct: 3, justification: "De acordo com o Quadro 43 de Problemas em Face e a Tabela 2 de Temperatura Adulto, 'inchaço na face' e 'febril' (37.5-38.4°C) são critérios para classificação Verde (Pouco Urgente)." },
            { question: "Um paciente com problemas em olhos apresenta olho vermelho e sensação de corpo estranho. Qual a classificação de risco?", options: ["Urgente", "Muito Urgente", "Emergência", "Pouco Urgente (Verde)"], correct: 3, justification: "De acordo com o Quadro 44 de Problemas em Olhos, 'olho vermelho' e 'sensação de corpo estranho' são critérios para classificação Verde (Pouco Urgente)." }
        ];

        const riskLevels = [
            { name: 'Emergência', color: 'bg-red-500', hover: 'hover:bg-red-600', text: 'text-white' },
            { name: 'Muito Urgente', color: 'bg-orange-500', hover: 'hover:bg-orange-600', text: 'text-white' },
            { name: 'Urgente', color: 'bg-yellow-400', hover: 'hover:bg-yellow-500', text: 'text-slate-900' },
            { name: 'Pouco Urgente', color: 'bg-green-500', hover: 'hover:bg-green-600', text: 'text-white' },
            { name: 'Não Urgente', color: 'bg-blue-500', hover: 'hover:bg-blue-600', text: 'text-white' }
        ];

        const startScreen = document.getElementById('start-screen');
        const quizScreen = document.getElementById('quiz-screen');
        const resultsScreen = document.getElementById('results-screen');
        
        const startBtn = document.getElementById('start-btn');
        const nextBtn = document.getElementById('next-btn');
        const restartBtn = document.getElementById('restart-btn');

        const questionCounter = document.getElementById('question-counter');
        const questionText = document.getElementById('question-text');
        const answerButtonsEl = document.getElementById('answer-buttons');
        const feedbackEl = document.getElementById('feedback');
        const feedbackText = document.getElementById('feedback-text');
        
        const scoreText = document.getElementById('score-text');
        const resultsChartEl = document.getElementById('results-chart');
        let resultsChart;

        let currentQuestionIndex = 0;
        let score = 0;

        function startQuiz() {
            currentQuestionIndex = 0;
            score = 0;
            startScreen.classList.add('hidden');
            resultsScreen.classList.add('hidden');
            quizScreen.classList.remove('hidden');
            nextBtn.classList.add('hidden');
            loadQuestion();
        }

        function loadQuestion() {
            feedbackEl.classList.remove('show');
            const currentQuestion = quizData[currentQuestionIndex];
            questionCounter.innerText = `Caso ${currentQuestionIndex + 1}/${quizData.length}`;
            questionText.innerText = currentQuestion.question;

            answerButtonsEl.innerHTML = '';
            
            const riskMapping = {
                "Emergência (Vermelho)": 0,
                "Muito Urgente (Laranja)": 1,
                "Urgente (Amarelo)": 2,
                "Pouco Urgente (Verde)": 3,
                "Não Urgente (Azul)": 4,
            };

            const allPossibleAnswers = ["Emergência (Vermelho)", "Muito Urgente (Laranja)", "Urgente (Amarelo)", "Pouco Urgente (Verde)", "Não Urgente (Azul)"];

            allPossibleAnswers.forEach((text, index) => {
                const btn = document.createElement('button');
                const riskIndex = riskMapping[text];
                const risk = riskLevels[riskIndex];
                
                btn.innerHTML = text;
                btn.classList.add('answer-btn', 'w-full', 'font-semibold', 'py-3', 'px-4', 'rounded-lg', 'border-2', 'border-transparent', risk.color, risk.hover, risk.text);
                btn.dataset.index = index;
                btn.addEventListener('click', selectAnswer);
                answerButtonsEl.appendChild(btn);
            });
            nextBtn.classList.add('hidden');
        }

        function selectAnswer(e) {
            const selectedBtn = e.target;
            const selectedIndex = Array.from(answerButtonsEl.children).indexOf(selectedBtn);
            const currentQuestion = quizData[currentQuestionIndex];
            const correctIndex = currentQuestion.correct;
            
            const allButtons = answerButtonsEl.querySelectorAll('button');

            if(selectedIndex === correctIndex){
                score++;
                selectedBtn.classList.add('correct');
            } else {
                selectedBtn.classList.add('incorrect');
                allButtons[correctIndex].classList.add('correct');
            }
            
            allButtons.forEach(btn => {
                btn.disabled = true;
                btn.style.cursor = 'not-allowed';
            });
            
            feedbackText.innerText = currentQuestion.justification;
            feedbackEl.classList.add('show');
            nextBtn.classList.remove('hidden');

            if (currentQuestionIndex === quizData.length - 1) {
                nextBtn.innerText = 'Ver Resultados';
            } else {
                nextBtn.innerText = 'Próximo Caso';
            }
        }

        function showNextQuestion() {
            currentQuestionIndex++;
            if (currentQuestionIndex < quizData.length) {
                loadQuestion();
            } else {
                showResults();
            }
        }
        
        function showResults(){
            quizScreen.classList.add('hidden');
            resultsScreen.classList.remove('hidden');
            scoreText.innerText = `${score}/${quizData.length}`;

            const correctAnswers = score;
            const incorrectAnswers = quizData.length - score;

            if(resultsChart) {
                resultsChart.destroy();
            }

            resultsChart = new Chart(resultsChartEl, {
                type: 'bar',
                data: {
                    labels: ['Respostas Corretas', 'Respostas Incorretas'],
                    datasets: [{
                        label: 'Desempenho no Treinamento',
                        data: [correctAnswers, incorrectAnswers],
                        backgroundColor: [
                            'rgba(34, 197, 94, 0.7)',
                            'rgba(239, 68, 68, 0.7)'
                        ],
                        borderColor: [
                            'rgba(22, 163, 74, 1)',
                            'rgba(220, 38, 38, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                stepSize: 1
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        }
                    }
                }
            });
        }

        startBtn.addEventListener('click', startQuiz);
        nextBtn.addEventListener('click', showNextQuestion);
        restartBtn.addEventListener('click', startQuiz);

    </script>
</body>
</html>
