<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>6학년 영어 복습 게임 - 우주 비행기 단어 격추!</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Firebase SDK (v8 호환 모드 라이브러리를 통해 전역 firebase 정의 오류 완벽히 해결) -->
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-firestore.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2 family=Jua&family=Nanum+Gothic:wght@400;700;800&display=swap');
        body {
            font-family: 'Nanum Gothic', sans-serif;
            background-color: #0b0f19;
            user-select: none;
        }
        .font-jua {
            font-family: 'Jua', sans-serif;
        }
        /* 우주 배경 애니메이션 효과 */
        .stars {
            background: radial-gradient(white, rgba(255,255,255,.2) 2px, transparent 40px),
                        radial-gradient(white, rgba(255,255,255,.15) 1px, transparent 30px),
                        radial-gradient(white, rgba(255,255,255,.1) 2px, transparent 40px);
            background-size: 550px 550px, 350px 350px, 250px 250px;
            background-position: 0 0, 40px 60px, 130px 270px;
            animation: starScroll 20s linear infinite;
        }
        @keyframes starScroll {
            from { background-position: 0 0, 40px 60px, 130px 270px; }
            to { background-position: 0 1000px, 40px 1060px, 130px 1270px; }
        }
        /* 캔버스 터치 영역 최적화 */
        canvas {
            touch-action: none;
        }
    </style>
</head>
<body class="h-screen w-screen overflow-hidden flex flex-col justify-between text-white relative">
    
    <!-- 우주 배경 효과 -->
    <div class="absolute inset-0 stars pointer-events-none z-0"></div>

    <!-- 메인 컨테이너 -->
    <div class="relative z-10 flex-grow flex flex-col items-center justify-center p-4 w-full max-w-4xl mx-auto h-full overflow-y-auto">
        
        <!-- 1. 로그인/학생 정보 입력 화면 -->
        <div id="login-screen" class="w-full max-w-md bg-slate-900/95 border-2 border-indigo-500 rounded-2xl p-6 shadow-2xl shadow-indigo-500/20 backdrop-blur-sm my-auto">
            <div class="text-center mb-6">
                <span class="bg-indigo-600 text-xs px-3 py-1 rounded-full font-bold uppercase tracking-wider">천재교과서 (김태은) 6학년 영어 복습</span>
                <h1 class="text-3xl font-jua text-indigo-400 mt-2 mb-1">우주 단어 격추 게임</h1>
                <p class="text-sm text-slate-400">Space Vocab Defender</p>
            </div>

            <form id="login-form" class="space-y-4" onsubmit="startGame(event)">
                <!-- 학급 선택 (1~5반 제한) -->
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-sm text-slate-300 mb-1 font-bold">학반 (Class)</label>
                        <select id="student-class" required class="w-full bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-indigo-500">
                            <option value="" disabled selected>선택하세요</option>
                            <option value="1">1반</option>
                            <option value="2">2반</option>
                            <option value="3">3반</option>
                            <option value="4">4반</option>
                            <option value="5">5반</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm text-slate-300 mb-1 font-bold">번호 (Number)</label>
                        <select id="student-number" required class="w-full bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-indigo-500">
                            <option value="" disabled selected>선택하세요</option>
                            <!-- 1번부터 25번까지 스크립트로 동적 생성 -->
                        </select>
                    </div>
                </div>

                <!-- 이름 입력 (실명 작성 유도) -->
                <div>
                    <label class="block text-sm text-slate-300 mb-1 font-bold">이름 (Name)</label>
                    <input type="text" id="student-name" required placeholder="본인의 '실명'을 한글로 입력하세요" class="w-full bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-white placeholder-slate-500 focus:outline-none focus:border-indigo-500">
                    <p class="text-[11px] text-yellow-400/80 mt-1"><i class="fa-solid fa-triangle-exclamation mr-1"></i>선생님께서 학습 확인을 하실 수 있도록 반드시 실명을 입력하세요.</p>
                </div>

                <!-- 게임 안내 및 조작법 -->
                <div class="bg-slate-800/80 p-3 rounded-lg border border-slate-700 text-xs text-slate-300 space-y-1">
                    <p class="font-bold text-yellow-400 text-center mb-1"><i class="fa-solid fa-circle-info"></i> 게임 단계별 3라운드 구성 (각 라운드 10문제, 총 30문제)</p>
                    <p>🥇 <strong>Round 1 (1~10번):</strong> 천천히 하강하는 객관식 우주선 격추! (보기 자동 섞임)</p>
                    <p>🥈 <strong>Round 2 (11~20번):</strong> 더 빠르게 지그재그로 움직이는 기동 격추!</p>
                    <p>🥉 <strong>Round 3 (21~30번):</strong> 주관식 철자 격추! 스펠링 순서대로 요격해요!</p>
                </div>

                <div class="space-y-2">
                    <button type="submit" class="w-full py-3 bg-gradient-to-r from-indigo-500 to-purple-600 hover:from-indigo-600 hover:to-purple-700 text-white font-jua text-xl rounded-xl transition duration-200 transform hover:scale-[1.02] active:scale-95 shadow-lg shadow-indigo-500/30">
                        게임 출격하기! <i class="fa-solid fa-rocket ml-1"></i>
                    </button>
                    <!-- 교사용 학습 현황판 전용 진입 버튼 -->
                    <button type="button" onclick="openTeacherDashboard()" class="w-full py-2.5 bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold rounded-xl transition duration-200 border border-slate-700 flex items-center justify-center gap-2 text-sm shadow-md">
                        <i class="fa-solid fa-user-tie text-emerald-400"></i> 교사용 실시간 학습 현황판 바로가기
                    </button>
                </div>
            </form>
        </div>

        <!-- 2. 게임 플레이 화면 (처음엔 숨김) -->
        <div id="game-screen" class="hidden w-full h-full flex flex-col justify-between relative">
            
            <!-- 라운드 예고 오버레이 (라운드 시작할 때 등장) -->
            <div id="round-transition-overlay" class="absolute inset-0 bg-slate-950/95 z-40 hidden flex flex-col items-center justify-center text-center p-6 rounded-2xl border-2 border-indigo-500">
                <span class="text-xs text-yellow-400 font-bold uppercase tracking-widest px-3 py-1 bg-yellow-400/10 rounded-full border border-yellow-400/30 mb-2 animate-pulse">SYSTEM UPDATE</span>
                <h2 id="round-title" class="text-5xl font-jua text-indigo-400 tracking-wider">ROUND 1</h2>
                <div class="w-16 h-1 bg-indigo-500 my-4 rounded"></div>
                <p id="round-desc" class="text-slate-300 font-bold mb-6 max-w-sm leading-relaxed text-sm md:text-base">천천히 다가오는 올바른 단어를 격추하세요!</p>
                <div class="text-xs text-slate-500 animate-pulse">잠시 후 라운드가 시작됩니다...</div>
            </div>

            <!-- 상단 정보 헤더 -->
            <div class="bg-slate-900/80 border border-slate-800 rounded-xl p-3 flex justify-between items-center text-xs md:text-sm">
                <div class="flex items-center space-x-2 md:space-x-3">
                    <span id="display-info" class="font-bold text-indigo-400">6-1 1번 홍길동</span>
                    <span class="text-slate-500">|</span>
                    <span class="text-yellow-400 font-bold">Score: <span id="display-score">0</span></span>
                    <span class="text-slate-500">|</span>
                    <span id="display-round" class="bg-indigo-950 border border-indigo-800 text-indigo-300 text-xs px-2.5 py-0.5 rounded-full font-bold">ROUND 1</span>
                    <span class="text-slate-500">|</span>
                    <span class="text-emerald-400 font-bold">문제: <span id="display-progress">1/30</span></span>
                </div>
                <!-- 하트 라이프 -->
                <div class="flex items-center space-x-1" id="heart-container">
                    <i class="fa-solid fa-heart text-red-500"></i>
                    <i class="fa-solid fa-heart text-red-500"></i>
                    <i class="fa-solid fa-heart text-red-500"></i>
                    <i class="fa-solid fa-heart text-red-500"></i>
                    <i class="fa-solid fa-heart text-red-500"></i>
                </div>
            </div>

            <!-- 문제 정보 및 힌트 카드 -->
            <div class="my-2 bg-gradient-to-r from-blue-950/90 to-slate-900/90 border border-blue-500/50 rounded-xl p-3 text-center">
                <div class="text-[11px] text-blue-400 font-bold uppercase tracking-wider mb-1" id="quiz-unit-title">주제 1. 학년 묻고 답하기</div>
                
                <!-- 퀴즈 문장 및 스펠링 상황 영역 -->
                <div class="flex flex-col items-center justify-center space-y-2 py-1">
                    <div class="text-lg md:text-xl font-bold tracking-wide text-white leading-relaxed" id="quiz-sentence">
                        What <span class="text-yellow-400 border-b-2 border-dashed border-yellow-400 px-4">?</span> are you in?
                    </div>
                    <!-- Round 3용 스펠링 조합 실시간 디스플레이 -->
                    <div id="spelling-display-container" class="hidden flex items-center space-x-2 bg-slate-950/60 px-4 py-1.5 rounded-lg border border-emerald-500/30">
                        <span class="text-xs text-emerald-400 font-bold mr-2"><i class="fa-solid fa-keyboard"></i> 조합 현황:</span>
                        <div id="spelling-word" class="flex space-x-1 text-lg md:text-xl font-black"></div>
                    </div>
                </div>

                <!-- 힌트 버튼 및 안내 -->
                <div class="mt-1 flex justify-center items-center gap-2">
                    <button onclick="revealHint()" class="bg-indigo-900/50 hover:bg-indigo-800 text-indigo-300 border border-indigo-700 text-xs px-2 py-1 rounded-md transition flex items-center gap-1">
                        <i class="fa-solid fa-lightbulb text-yellow-300"></i> 힌트 보기
                    </button>
                    <span id="quiz-hint" class="text-xs text-slate-400 bg-slate-950/60 px-2 py-1 rounded hidden">
                        힌트: '학년'을 영어로 무엇이라고 할까요? (g로 시작하는 단어)
                    </span>
                </div>
            </div>

            <!-- 메인 게임 캔버스 영역 -->
            <div class="flex-grow relative bg-slate-950/60 border border-slate-800 rounded-xl overflow-hidden flex items-center justify-center">
                <canvas id="gameCanvas" class="w-full h-full block"></canvas>
                
                <!-- 게임 오버 / 일시 정지 오버레이 (필요 시 화면 위에 띄움) -->
                <div id="game-paused" class="absolute inset-0 bg-slate-950/80 hidden flex flex-col items-center justify-center space-y-4 z-20">
                    <p class="text-2xl font-bold">잠시 멈춤</p>
                    <button onclick="resumeGame()" class="px-6 py-2 bg-indigo-600 rounded-lg hover:bg-indigo-700">계속하기</button>
                </div>
            </div>

            <!-- 모바일/터치 조작 컨트롤 패널 -->
            <div class="mt-2 grid grid-cols-12 gap-2 max-w-lg mx-auto w-full">
                <div class="col-span-4 flex gap-2">
                    <button id="btn-left" class="flex-1 py-4 bg-slate-800 active:bg-slate-700 border border-slate-700 rounded-xl flex items-center justify-center text-xl select-none">
                        <i class="fa-solid fa-arrow-left"></i>
                    </button>
                    <button id="btn-right" class="flex-1 py-4 bg-slate-800 active:bg-slate-700 border border-slate-700 rounded-xl flex items-center justify-center text-xl select-none">
                        <i class="fa-solid fa-arrow-right"></i>
                    </button>
                </div>
                <div class="col-span-8">
                    <button id="btn-shoot" class="w-full py-4 bg-red-600 active:bg-red-700 border border-red-500 rounded-xl font-jua text-lg tracking-wider flex items-center justify-center gap-2 select-none shadow-lg shadow-red-900/30">
                        <i class="fa-solid fa-crosshairs text-xl animate-pulse"></i> 빔 발사 (SPACE)
                    </button>
                </div>
            </div>
        </div>

        <!-- 3. 결과 리포트 화면 (처음엔 숨김) -->
        <div id="result-screen" class="hidden w-full max-w-lg bg-slate-900/95 border-2 border-emerald-500 rounded-2xl p-6 shadow-2xl shadow-emerald-500/20 backdrop-blur-sm text-center my-auto">
            <div class="mb-4">
                <span class="bg-emerald-600 text-xs px-3 py-1 rounded-full font-bold uppercase tracking-wider">Mission Accomplished!</span>
                <h2 class="text-3xl font-jua text-emerald-400 mt-2">영어 정복 성공!</h2>
            </div>

            <!-- 학생 정보 및 성적 카드 -->
            <div class="bg-slate-800 border border-slate-700 rounded-xl p-4 my-4 space-y-2 text-left">
                <div class="flex justify-between border-b border-slate-700 pb-2">
                    <span class="text-slate-400">참여자 정보:</span>
                    <span id="result-student" class="font-bold text-white">6-1 1번 홍길동</span>
                </div>
                <div class="flex justify-between items-center pt-1">
                    <span class="text-slate-400">획득 점수:</span>
                    <span id="result-score" class="text-2xl font-jua text-yellow-400">0 점</span>
                </div>
                <div class="flex justify-between items-center">
                    <span class="text-slate-400">맞춘 문제 수:</span>
                    <span id="result-correct" class="text-emerald-400 font-bold">0 / 30</span>
                </div>
                <div class="text-xs text-center mt-2 border-t border-slate-700/50 pt-2" id="result-firebase-status">
                    <i class="fa-solid fa-cloud-arrow-up animate-pulse mr-1"></i> 서버에 학습 결과를 자동 전송하고 있습니다...
                </div>
                <div class="text-xs text-slate-400 text-center mt-2" id="result-feedback">
                    훌륭해요! 6학년 핵심 표현을 마스터했군요!
                </div>
            </div>

            <!-- 리포트 복습 영역 -->
            <div class="text-left mb-6">
                <p class="text-sm font-bold text-indigo-400 mb-2"><i class="fa-solid fa-book-open mr-1"></i> 오늘 공부한 핵심 주제 목록:</p>
                <div class="bg-slate-950 rounded-lg p-3 max-h-32 overflow-y-auto text-xs space-y-2 text-slate-300 border border-slate-800">
                    <p>✔ <strong>1. 학년 묻고 답하기:</strong> What <em>grade</em> are you in? / I'm in the <em>sixth</em> grade.</p>
                    <p>✔ <strong>2. 아픈 곳 묻고 답하기:</strong> What's <em>wrong</em>? / I have a <em>cold</em>/<em>headache</em>. / Take some medicine.</p>
                    <p>✔ <strong>3. 날짜 및 생일:</strong> When is your birthday? / It's <em>on</em> June third.</p>
                    <p>✔ <strong>4. 비교하기:</strong> Which one is <em>faster</em>? / The horse is <em>faster</em> than the dog.</p>
                    <p>✔ <strong>5. 계획 묻고 답하기:</strong> What are you <em>going</em> to do? / I'm going to <em>take</em> a robot class.</p>
                    <p>✔ <strong>6. 외모나 옷차림:</strong> What does she <em>look</em> like? / She has <em>long</em> straight hair.</p>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-2">
                <button onclick="restartGame()" class="py-3 bg-slate-700 hover:bg-slate-600 text-white font-bold rounded-xl transition duration-200 text-sm">
                    다시 하기 <i class="fa-solid fa-rotate-right ml-1"></i>
                </button>
                <button onclick="exportResult()" class="py-3 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-xl transition duration-200 text-sm">
                    결과 복사 <i class="fa-solid fa-copy ml-1"></i>
                </button>
                <!-- 결과창에서도 동일하게 비밀번호 입력을 유도하도록 연동 -->
                <button onclick="openTeacherDashboard()" class="py-3 bg-emerald-600 hover:bg-emerald-700 text-white font-bold rounded-xl transition duration-200 text-sm shadow-md shadow-emerald-900/40">
                    현황판 <i class="fa-solid fa-user-tie ml-1"></i>
                </button>
            </div>
        </div>

        <!-- 4. 교사 전용 대시보드 화면 (비밀번호: qwert) -->
        <div id="teacher-screen" class="hidden w-full max-w-3xl bg-slate-900/95 border-2 border-emerald-500 rounded-2xl p-6 shadow-2xl backdrop-blur-sm my-auto text-left">
            <div class="flex justify-between items-center mb-4 pb-2 border-b border-slate-800">
                <div>
                    <h2 class="text-2xl font-jua text-emerald-400 flex items-center gap-2">
                        <i class="fa-solid fa-user-tie"></i> 6학년 영어 학습 현황판
                    </h2>
                    <p class="text-xs text-slate-400 mt-0.5">실시간으로 전송된 학생들의 복습 결과를 모니터링합니다. (비밀번호 보호 모드)</p>
                </div>
                <button onclick="closeTeacherDashboard()" class="text-slate-400 hover:text-white px-3 py-1 bg-slate-800 rounded-lg text-xs transition">
                    나가기 <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <!-- 필터 및 제어 카드 -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4">
                <div class="bg-slate-950 p-3 rounded-xl border border-slate-800 flex flex-col justify-center">
                    <span class="text-xs text-slate-500 font-bold mb-1">학반 필터 (Class Filter)</span>
                    <select id="teacher-class-filter" onchange="filterTeacherRecords()" class="bg-slate-900 border border-slate-700 rounded px-2 py-1 text-xs focus:outline-none focus:border-emerald-500">
                        <option value="all">전체 학반 보기</option>
                        <option value="1">1반</option>
                        <option value="2">2반</option>
                        <option value="3">3반</option>
                        <option value="4">4반</option>
                        <option value="5">5반</option>
                    </select>
                </div>
                <div class="bg-slate-950 p-3 rounded-xl border border-slate-800 text-center flex flex-col justify-center">
                    <span class="text-xs text-slate-500 font-bold">총 완료 인원</span>
                    <span id="stats-total-students" class="text-2xl font-jua text-emerald-400 mt-1">0 명</span>
                </div>
                <div class="bg-slate-950 p-3 rounded-xl border border-slate-800 text-center flex flex-col justify-center">
                    <span class="text-xs text-slate-500 font-bold">평균 획득 점수</span>
                    <span id="stats-avg-score" class="text-2xl font-jua text-yellow-400 mt-1">0 점</span>
                </div>
            </div>

            <!-- 학생 기록 목록 테이블 -->
            <div class="bg-slate-950 rounded-xl border border-slate-800 overflow-hidden flex flex-col">
                <div class="max-h-72 overflow-y-auto">
                    <table class="w-full text-xs text-left text-slate-300">
                        <thead class="bg-slate-900 text-slate-400 sticky top-0 uppercase text-[11px] border-b border-slate-800">
                            <tr>
                                <th class="px-4 py-3 text-center">반</th>
                                <th class="px-4 py-3 text-center">번호</th>
                                <th class="px-4 py-3">이름</th>
                                <th class="px-4 py-3 text-right">점수</th>
                                <th class="px-4 py-3 text-center">정답수</th>
                                <th class="px-4 py-3 text-center">제출 일시</th>
                            </tr>
                        </thead>
                        <tbody id="student-records-body">
                            <!-- 실시간 데이터 삽입 구역 -->
                            <tr>
                                <td colspan="6" class="text-center py-8 text-slate-500 text-xs">
                                    <i class="fa-solid fa-triangle-exclamation mr-1 animate-pulse"></i> 등록된 학습 결과 데이터가 아직 없거나 불러오는 중입니다.
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <div class="mt-4 flex justify-between items-center text-xs text-slate-500">
                <span class="italic"><i class="fa-solid fa-circle-nodes text-emerald-500"></i> Cloud Sync Enabled</span>
                <button onclick="clearAllDataDemoPrompt()" class="text-red-400/80 hover:text-red-400 text-[10px] underline">
                    현황판 전체 초기화 (보안 인증 필요)
                </button>
            </div>
        </div>

    </div>

    <!-- 푸터 -->
    <footer class="relative z-10 py-2 text-center text-[10px] text-slate-600 border-t border-slate-900 bg-slate-950/80">
        천재교과서(김태은) 초등 영어 6학년 (2022년 개정 교육과정) | Developed for Smart Classroom
    </footer>

    <!-- 알림 메시지 모달 (alert 대체용) -->
    <div id="toast-message" class="fixed top-5 left-1/2 transform -translate-x-1/2 z-50 bg-slate-900 border border-indigo-500 text-white px-6 py-3 rounded-xl shadow-2xl opacity-0 pointer-events-none transition-all duration-300 text-center text-sm font-bold flex items-center gap-2">
        <i class="fa-solid fa-circle-check text-indigo-400"></i> <span id="toast-text">알림 메시지</span>
    </div>

    <script>
        // ----------------------------------------------------
        // [1] Firebase 및 Firestore 설정 & 안전 가동법 (규칙 1, 3 준수)
        // ----------------------------------------------------
        const firebaseConfig = {
            apiKey: "", // 자동 환경 주입용
            authDomain: "canvas-project-auth.firebaseapp.com",
            projectId: "canvas-project-auth",
            storageBucket: "canvas-project-auth.appspot.com",
            messagingSenderId: "1234567890",
            appId: "1:1234567890:web:123456abc"
        };
        
        let app, db, auth;
        let dbUser = null;
        const appId = typeof __app_id !== 'undefined' ? __app_id : '6th-grade-english-game';

        // 1-1. Firebase 초기화 안전 확인 및 세션 리스너 가동
        try {
            const finalConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : firebaseConfig;
            
            app = firebase.initializeApp(finalConfig);
            db = firebase.firestore(app);
            auth = firebase.auth(app);
            
            const initAuth = async () => {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    try {
                        await auth.signInWithCustomToken(__initial_auth_token);
                    } catch (e) {
                        await auth.signInAnonymously();
                    }
                } else {
                    await auth.signInAnonymously();
                }
            };
            initAuth();

            auth.onAuthStateChanged((user) => {
                if (user) {
                    dbUser = user;
                    attachDashboardListener();
                }
            });
        } catch (error) {
            console.error("Firebase Initialization Failure: ", error);
        }

        // ----------------------------------------------------
        // [2] 교과서 기반 30문항 대규모 핵심 문제 은행 데이터 (각 라운드 10문제)
        // ----------------------------------------------------
        const quizBank = [
            // ==================== [ROUND 1] 1~10번 (Easy - 객관식, 느린 직하강, 보기 랜덤 배치) ====================
            {
                unit: "주제 1. 학년 묻고 답하기",
                sentence: "What ________ are you in?",
                hint: "'학년'을 뜻하는 대화 단어를 고르세요. g로 시작합니다.",
                answer: "grade",
                options: ["grade", "class", "school", "date"]
            },
            {
                unit: "주제 1. 학년 묻고 답하기",
                sentence: "I'm in the ________ grade. (6학년)",
                hint: "여섯 번째를 뜻하는 서수를 사용해요.",
                answer: "sixth",
                options: ["sixth", "six", "second", "fifth"]
            },
            {
                unit: "주제 2. 아픈 곳 묻고 답하기",
                sentence: "A: What's ________?<br>B: I have a headache.",
                hint: "어디가 불편한지, 무슨 일이 있는지 물을 때 쓰는 정형적인 문장입니다.",
                answer: "wrong",
                options: ["wrong", "matter", "look", "date"]
            },
            {
                unit: "주제 2. 아픈 곳 묻고 답하기",
                sentence: "A: What's wrong?<br>B: I have a ________. (감기)",
                hint: "으슬으슬 춥고 기침이 날 때 걸리는 대중적인 건강 이상 표현입니다.",
                answer: "cold",
                options: ["cold", "fever", "headache", "stomachache"]
            },
            {
                unit: "주제 2. 아픈 곳 묻고 답하기",
                sentence: "A: I have a headache.<br>B: Take ________ medicine and rest.",
                hint: "셀 수 없는 명사나 약간의 명사 앞에 권유하는 수식 형용사입니다.",
                answer: "some",
                options: ["some", "any", "many", "a"]
            },
            {
                unit: "주제 3. 날짜 및 생일 묻고 답하기",
                sentence: "A: When is your birthday?<br>B: It's ________ May fifth.",
                hint: "특정한 날짜나 요일 앞에는 반드시 이 전치사 'on'을 씁니다.",
                answer: "on",
                options: ["on", "in", "at", "to"]
            },
            {
                unit: "주제 3. 날짜 및 행사 일정",
                sentence: "A: When is the school market?<br>B: It's on May ________. (5일)",
                hint: "날짜를 표기할 때는 '다섯 번째'를 뜻하는 서수 단어가 와야 합니다.",
                answer: "fifth",
                options: ["fifth", "five", "fiftieth", "fifteen"]
            },
            {
                unit: "주제 4. 비교하기",
                sentence: "Which bike is ________? (더 빠른)",
                hint: "fast의 비교급 형용사 형태를 완성해보세요.",
                answer: "faster",
                options: ["faster", "fast", "fastest", "more fast"]
            },
            {
                unit: "주제 5. 계획 묻고 답하기",
                sentence: "What are you ________ to do this Saturday?",
                hint: "'~할 예정이다' 계획을 물어볼 때 be동사 뒤에 쓰여요.",
                answer: "going",
                options: ["going", "doing", "planning", "wanting"]
            },
            {
                unit: "주제 6. 외모나 옷차림 묻고 답하기",
                sentence: "What ________ she look like?",
                hint: "3인칭 단수 주어 she를 수식하며 생김새를 물을 때의 조동사입니다.",
                answer: "does",
                options: ["does", "do", "is", "are"]
            },

            // ==================== [ROUND 2] 11~20번 (Medium - 객관식, 빠른 속도 + 좌우 지그재그 회피) ====================
            {
                unit: "주제 1. 학년 묻고 답하기",
                sentence: "A: Are you in the fifth ________?<br>B: Yes, I am.",
                hint: "'5학년' 상태를 질문하는 짝 단어입니다.",
                answer: "grade",
                options: ["grade", "class", "year", "semester"]
            },
            {
                unit: "주제 2. 아픈 곳 묻고 답하기",
                sentence: "I have a ________. (두통)",
                hint: "머리가 아플 때 쓰는 증상 표기입니다.",
                answer: "headache",
                options: ["headache", "cold", "fever", "toothache"]
            },
            {
                unit: "주제 2. 아픈 곳 묻고 답하기",
                sentence: "You should go and ________ a doctor.",
                hint: "'의사를 만나 진찰을 받아보다'할 때 어울리는 자연스러운 기본 동사입니다.",
                answer: "see",
                options: ["see", "look", "watch", "meet"]
            },
            {
                unit: "주제 3. 날짜 및 생일 묻고 답하기",
                sentence: "A: When is the school festival?<br>B: It's ________ October first.",
                hint: "교과서 속 생일이나 행사의 날짜를 말할 때 쓰는 핵심 전치사입니다.",
                answer: "on",
                options: ["on", "at", "in", "by"]
            },
            {
                unit: "주제 3. 날짜 및 행사 일정",
                sentence: "A: When is the soccer game?<br>B: It's on July ________. (2일)",
                hint: "날짜 2일은 기수가 아닌 '두 번째' 서수 단어로 정답을 표기합니다.",
                answer: "second",
                options: ["second", "two", "first", "third"]
            },
            {
                unit: "주제 4. 비교하기",
                sentence: "The cheetah is faster ________ the hippo.",
                hint: "'~보다'라는 비교 구문의 기준 전치사/접속사 단어입니다.",
                answer: "than",
                options: ["than", "then", "that", "this"]
            },
            {
                unit: "주제 4. 비교하기",
                sentence: "Which building is ________? (더 높은)",
                hint: "높은 빌딩을 나타내는 tall의 비교급 버전입니다.",
                answer: "taller",
                options: ["taller", "tall", "tallest", "short"]
            },
            {
                unit: "주제 5. 계획 묻고 답하기",
                sentence: "I'm going to ________ a book club.",
                hint: "동아리나 클럽 활동 등에 '가입하다/참여하다' 동사입니다.",
                answer: "join",
                options: ["join", "take", "play", "see"]
            },
            {
                unit: "주제 6. 외모나 옷차림 묻고 답하기",
                sentence: "She has ________ straight hair.",
                hint: "머리가 긴 생머리(straight hair)라고 설명할 때 들어갈 수식 형용사입니다.",
                answer: "long",
                options: ["long", "short", "tall", "big"]
            },
            {
                unit: "주제 6. 외모나 옷차림 묻고 답하기",
                sentence: "He is ________ a green cap.",
                hint: "모자를 착용하고 있는 역동적인 동작 진행형 표현입니다.",
                answer: "wearing",
                options: ["wearing", "putting", "taking", "doing"]
            },

            // ==================== [ROUND 3] 21~30번 (Hard - 주관식 알파벳 격추 모드, 점진적 출현) ====================
            {
                unit: "주제 1. 학년 묻고 답하기",
                sentence: "What ________ are you in? (학년)",
                hint: "'학년'의 영어 스펠링 5글자(g-r-a-d-e)를 순서대로 격추하세요!",
                answer: "grade"
            },
            {
                unit: "주제 1. 학년 묻고 답하기",
                sentence: "I'm in the ________ grade. (6학년)",
                hint: "여섯 번째를 의미하는 서수 5글자(s-i-x-t-h)를 순서대로 요격하세요!",
                answer: "sixth"
            },
            {
                unit: "주제 2. 아픈 곳 묻고 답하기",
                sentence: "I have a ________. (열)",
                hint: "'열'이 있음을 뜻하는 5글자(f-e-v-e-r) 단어의 철자를 조준해 보세요!",
                answer: "fever"
            },
            {
                unit: "주제 3. 날짜 및 생일 묻고 답하기",
                sentence: "A: When is the soccer game?<br>B: It's ________ June fourth.",
                hint: "날짜 대답 시 교과서에 사용되는 전치사 2글자(o-n)를 차례로 격추하세요!",
                answer: "on"
            },
            {
                unit: "주제 3. 날짜 및 행사 일정",
                sentence: "A: When is the market?<br>B: It's on September ________. (3일)",
                hint: "날짜 3일을 의미하는 서수 5글자(t-h-i-r-d)를 순서대로 사격하세요!",
                answer: "third"
            },
            {
                unit: "주제 4. 비교하기",
                sentence: "The bear is bigger ________ the fox.",
                hint: "'~보다'의 뜻을 가진 비교 단어 4글자(t-h-a-n)를 연속 사격하세요!",
                answer: "than"
            },
            {
                unit: "주제 4. 비교하기",
                sentence: "Which card is ________? (더 작은)",
                hint: "small(작은)의 비교급 형태인 7글자(s-m-a-l-l-e-r)를 만드세요!",
                answer: "smaller"
            },
            {
                unit: "주제 5. 계획 묻고 답하기",
                sentence: "I'm ________ to take a robot class.",
                hint: "~할 예정이다라는 표현의 5글자(g-o-i-n-g) 철자를 조합하세요!",
                answer: "going"
            },
            {
                unit: "주제 6. 외모나 옷차림 묻고 답하기",
                sentence: "What does he ________ like?",
                hint: "생김새를 묘사할 때 쓰는 동사 4글자(l-o-o-k) 단어입니다.",
                answer: "look"
            },
            {
                unit: "주제 6. 외모나 옷차림 묻고 답하기",
                sentence: "She is ________ a blue skirt.",
                hint: "옷차림을 나타내는 7글자(w-e-a-r-i-n-g) 철자를 완성하세요!",
                answer: "wearing"
            }
        ];

        // ----------------------------------------------------
        // [3] 전역 상태 및 변수 선언
        // ----------------------------------------------------
        let score = 0;
        let lives = 5;
        let currentQuizIndex = 0;
        let currentRound = 1; 
        let collectedSpelling = ""; 
        let isGameRunning = false;
        let canvas, ctx;
        let animationFrameId;

        let localRecords = [];

        // 게임 엔티티 선언
        let player;
        let bullets = [];
        let enemies = [];
        let particles = [];

        // 키 조작 상태
        const keys = { left: false, right: false };

        // ----------------------------------------------------
        // [4] 초기화 및 화면 제어
        // ----------------------------------------------------
        window.onload = function() {
            const numSelect = document.getElementById('student-number');
            for (let i = 1; i <= 25; i++) {
                const opt = document.createElement('option');
                opt.value = i;
                opt.textContent = `${i}번`;
                numSelect.appendChild(opt);
            }

            setupMobileControls();
        };

        // 토스트 알림창
        function showToast(text) {
            const toast = document.getElementById('toast-message');
            const toastText = document.getElementById('toast-text');
            toastText.textContent = text;
            toast.style.opacity = '1';
            setTimeout(() => {
                toast.style.opacity = '0';
            }, 2500);
        }

        // 배열 셔플 (정답 비행기의 예측 불가능한 위치를 위해 도입)
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        // 라운드 구간 연산기
        function determineRound(index) {
            if (index < 10) return 1;
            if (index < 20) return 2;
            return 3;
        }

        // 게임 플레이 가동 트리거
        function startGame(event) {
            event.preventDefault();

            studentInfo.class = document.getElementById('student-class').value;
            studentInfo.number = document.getElementById('student-number').value;
            studentInfo.name = document.getElementById('student-name').value.trim();

            if (!studentInfo.class || !studentInfo.number || !studentInfo.name) {
                showToast("학생 신상 정보를 모두 빠짐없이 선택/입력해주세요!");
                return;
            }
            if (studentInfo.name.length < 2) {
                showToast("선생님 확인을 위해 정확한 이름을 입력해 주세요 (2글자 이상)!");
                return;
            }

            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('game-screen').classList.remove('hidden');

            document.getElementById('display-info').textContent = `6학년 ${studentInfo.class}반 ${studentInfo.number}번 ${studentInfo.name}`;
            document.getElementById('display-score').textContent = score;
            document.getElementById('display-progress').textContent = `${currentQuizIndex + 1} / ${quizBank.length}`;

            initCanvas();
            initGameEngine();
        }

        // 게임 재시작
        function restartGame() {
            score = 0;
            lives = 5;
            currentQuizIndex = 0;
            currentRound = 1;
            collectedSpelling = "";
            bullets = [];
            enemies = [];
            particles = [];

            document.getElementById('result-screen').classList.add('hidden');
            document.getElementById('game-screen').classList.remove('hidden');
            
            updateLivesUI();
            document.getElementById('display-score').textContent = score;
            document.getElementById('display-progress').textContent = `${currentQuizIndex + 1} / ${quizBank.length}`;

            initCanvas();
            initGameEngine();
        }

        // 학습 결과 복사용 클립보드 매니저
        function exportResult() {
            const textToCopy = `[6학년 영어 우주복습 결과 리포트]\n학급: 6학년 ${studentInfo.class}반 ${studentInfo.number}번\n이름: ${studentInfo.name}\n최종 점수: ${score}점\n맞춘 문항: ${currentQuizIndex}개 / ${quizBank.length}개\n우주 영어 단어 대모험을 수료했습니다!`;
            
            const tempTextArea = document.createElement("textarea");
            tempTextArea.value = textToCopy;
            document.body.appendChild(tempTextArea);
            tempTextArea.select();
            
            try {
                document.execCommand('copy');
                showToast("학습 성적이 클립보드에 안전하게 복사되었습니다!");
            } catch (err) {
                showToast("복사 실패! 브라우저 권한을 검토하세요.");
            }
            document.body.removeChild(tempTextArea);
        }

        // ----------------------------------------------------
        // [5] 모바일 최적화 컨트롤 바인딩
        // ----------------------------------------------------
        function setupMobileControls() {
            const btnLeft = document.getElementById('btn-left');
            const btnRight = document.getElementById('btn-right');
            const btnShoot = document.getElementById('btn-shoot');

            btnLeft.addEventListener('touchstart', (e) => { e.preventDefault(); keys.left = true; });
            btnLeft.addEventListener('touchend', (e) => { e.preventDefault(); keys.left = false; });
            btnLeft.addEventListener('mousedown', () => { keys.left = true; });
            btnLeft.addEventListener('mouseup', () => { keys.left = false; });
            btnLeft.addEventListener('mouseleave', () => { keys.left = false; });

            btnRight.addEventListener('touchstart', (e) => { e.preventDefault(); keys.right = true; });
            btnRight.addEventListener('touchend', (e) => { e.preventDefault(); keys.right = false; });
            btnRight.addEventListener('mousedown', () => { keys.right = true; });
            btnRight.addEventListener('mouseup', () => { keys.right = false; });
            btnRight.addEventListener('mouseleave', () => { keys.right = false; });

            btnShoot.addEventListener('touchstart', (e) => { 
                e.preventDefault(); 
                if (isGameRunning) fireBullet(); 
            });
            btnShoot.addEventListener('mousedown', () => { 
                if (isGameRunning) fireBullet(); 
            });

            window.addEventListener('keydown', (e) => {
                if (!isGameRunning) return;
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = true;
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = true;
                if (e.key === ' ' || e.key === 'Spacebar') {
                    e.preventDefault();
                    fireBullet();
                }
            });

            window.addEventListener('keyup', (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = false;
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = false;
            });
        }

        // ----------------------------------------------------
        // [6] 문제 출제 및 힌트 연동
        // ----------------------------------------------------
        function loadQuiz(index) {
            if (index >= quizBank.length) {
                endGame(true);
                return;
            }

            const quiz = quizBank[index];
            const nextRound = determineRound(index);

            // 라운드 승급 오버레이 효과
            if (nextRound !== currentRound) {
                currentRound = nextRound;
                triggerRoundTransition(currentRound);
                return;
            }

            const rdDisplay = document.getElementById('display-round');
            rdDisplay.textContent = `ROUND ${currentRound}`;
            document.getElementById('display-progress').textContent = `${currentQuizIndex + 1} / ${quizBank.length}`;

            if (currentRound === 1) {
                rdDisplay.className = "bg-green-950 border border-green-800 text-green-300 text-[10px] md:text-xs px-2.5 py-0.5 rounded-full font-bold";
            } else if (currentRound === 2) {
                rdDisplay.className = "bg-orange-950 border border-orange-800 text-orange-300 text-[10px] md:text-xs px-2.5 py-0.5 rounded-full font-bold animate-pulse";
            } else {
                rdDisplay.className = "bg-red-950 border border-red-800 text-red-300 text-[10px] md:text-xs px-2.5 py-0.5 rounded-full font-bold animate-bounce";
            }

            document.getElementById('quiz-unit-title').textContent = `[Round ${currentRound}] ${quiz.unit}`;
            
            const spellingContainer = document.getElementById('spelling-display-container');
            if (currentRound === 3) {
                spellingContainer.classList.remove('hidden');
                collectedSpelling = ""; 
                updateSpellingDisplay(quiz.answer);
                document.getElementById('quiz-sentence').innerHTML = quiz.sentence.replace("________", `<span class="text-emerald-400 border-b-2 border-dashed border-emerald-400 px-4">?</span>`);
            } else {
                spellingContainer.classList.add('hidden');
                document.getElementById('quiz-sentence').innerHTML = quiz.sentence.replace("________", `<span class="text-yellow-400 border-b-2 border-dashed border-yellow-400 px-4">?</span>`);
            }
            
            const hintBox = document.getElementById('quiz-hint');
            hintBox.textContent = `💡 힌트: ${quiz.hint}`;
            hintBox.classList.add('hidden');

            spawnQuizEnemies(quiz);
        }

        function updateSpellingDisplay(answer) {
            const spellingWordDiv = document.getElementById('spelling-word');
            spellingWordDiv.innerHTML = "";
            for (let i = 0; i < answer.length; i++) {
                const charSpan = document.createElement('span');
                if (i < collectedSpelling.length) {
                    charSpan.textContent = collectedSpelling[i].toUpperCase();
                    charSpan.className = "text-emerald-400 border-b-2 border-emerald-400 px-1.5 md:px-2 animate-bounce";
                } else {
                    charSpan.textContent = "_";
                    charSpan.className = "text-slate-600 border-b-2 border-dashed border-slate-700 px-1.5 md:px-2";
                }
                spellingWordDiv.appendChild(charSpan);
            }
        }

        function triggerRoundTransition(round) {
            isGameRunning = false;
            if (animationFrameId) cancelAnimationFrame(animationFrameId);

            const overlay = document.getElementById('round-transition-overlay');
            const title = document.getElementById('round-title');
            const desc = document.getElementById('round-desc');

            overlay.classList.remove('hidden');
            title.textContent = `ROUND ${round}`;

            if (round === 2) {
                desc.innerHTML = "<span class='text-orange-400 font-extrabold'>SPEED UP! (속도 증가 & 무작위 셔플)</span><br>외계인 우주선 하강 속도가 빨라지고, 지그재그 회피 기동을 시도합니다! 정답 보기도 지능적으로 뒤섞입니다.";
            } else if (round === 3) {
                desc.innerHTML = "<span class='text-red-400 font-extrabold'>SPELLING SHOOTER! (스펠링 주관식)</span><br>객관식 보기가 완전히 사라집니다! 영어 단어의 <span class='text-emerald-400 underline'>스펠링 순서대로</span> 날아오는 단일 알파벳을 격추해 문장을 채워주세요!";
            }

            setTimeout(() => {
                overlay.classList.add('hidden');
                isGameRunning = true;
                loadQuiz(currentQuizIndex);
                gameLoop();
            }, 3000);
        }

        function revealHint() {
            const hintBox = document.getElementById('quiz-hint');
            hintBox.classList.toggle('hidden');
        }

        function updateLivesUI() {
            const container = document.getElementById('heart-container');
            container.innerHTML = '';
            for (let i = 0; i < 5; i++) {
                const heart = document.createElement('i');
                if (i < lives) {
                    heart.className = 'fa-solid fa-heart text-red-500 text-xs md:text-sm transition-transform scale-110';
                } else {
                    heart.className = 'fa-regular fa-heart text-slate-600 text-xs md:text-sm';
                }
                container.appendChild(heart);
            }
        }

        // ----------------------------------------------------
        // [7] 캔버스 엔진 & 물리 충돌 루프
        // ----------------------------------------------------
        function initCanvas() {
            canvas = document.getElementById('gameCanvas');
            ctx = canvas.getContext('2d');

            const rect = canvas.parentNode.getBoundingClientRect();
            canvas.width = rect.width;
            canvas.height = rect.height;

            window.addEventListener('resize', onCanvasResize);
        }

        function onCanvasResize() {
            if (!canvas) return;
            const rect = canvas.parentNode.getBoundingClientRect();
            canvas.width = rect.width;
            canvas.height = rect.height;
            if (player) {
                player.y = canvas.height - 50;
                player.x = Math.max(30, Math.min(canvas.width - 30, player.x));
            }
        }

        function initGameEngine() {
            isGameRunning = true;
            lives = 5;
            updateLivesUI();

            player = {
                x: canvas.width / 2,
                y: canvas.height - 60,
                width: 50,
                height: 45,
                speed: 6.5,
                color: '#38bdf8'
            };

            bullets = [];
            enemies = [];
            particles = [];

            loadQuiz(currentQuizIndex);

            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            gameLoop();
        }

        function getRandomAlphabet(excludeChar) {
            const alphabets = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
            const filtered = alphabets.replace(excludeChar.toUpperCase(), "");
            return filtered[Math.floor(Math.random() * filtered.length)];
        }

        function spawnQuizEnemies(quiz) {
            enemies = [];

            if (currentRound === 3) {
                // Round 3 주관식 알파벳 저격
                const count = 5;
                const sliceWidth = canvas.width / count;
                const nextChar = quiz.answer[collectedSpelling.length].toUpperCase();
                const correctSlot = Math.floor(Math.random() * count);

                for (let i = 0; i < count; i++) {
                    const isCorrect = (i === correctSlot);
                    const letter = isCorrect ? nextChar : getRandomAlphabet(nextChar);

                    enemies.push({
                        x: sliceWidth * i + sliceWidth / 2,
                        y: -40 - (Math.random() * 60),
                        width: 55,
                        height: 45,
                        text: letter,
                        isCorrect: isCorrect,
                        speed: 1.1 + Math.random() * 0.4,
                        pulse: Math.random() * Math.PI,
                        color: isCorrect ? '#10b981' : `hsl(${(i * 360 / count)}, 65%, 55%)`
                    });
                }
            } else {
                // Round 1 & 2 4지선다 기동 (셔플 로직 탑재!)
                const count = quiz.options.length;
                const sliceWidth = canvas.width / count;
                const speedMultiplier = (currentRound === 2) ? 1.7 : 1.0; 

                // 보기 순서를 완벽하게 섞음 (무작위 셔플)
                const shuffledOptions = shuffleArray([...quiz.options]);

                for (let i = 0; i < count; i++) {
                    const text = shuffledOptions[i];
                    const isCorrect = (text === quiz.answer);

                    enemies.push({
                        x: sliceWidth * i + sliceWidth / 2,
                        y: -40 - (Math.random() * 60),
                        width: 65,
                        height: 50,
                        text: text,
                        isCorrect: isCorrect,
                        speed: (0.45 + Math.random() * 0.4) * speedMultiplier,
                        pulse: Math.random() * Math.PI,
                        color: `hsl(${(i * 360 / count)}, 80%, 65%)`
                    });
                }
            }
        }

        function fireBullet() {
            bullets.push({
                x: player.x,
                y: player.y - 20,
                width: 5,
                height: 15,
                speed: 13,
                color: '#f43f5e'
            });
            createExplosion(player.x, player.y - 20, '#f43f5e', 4);
        }

        function createExplosion(x, y, color, count = 10) {
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: x,
                    y: y,
                    vx: (Math.random() - 0.5) * 8,
                    vy: (Math.random() - 0.5) * 8,
                    size: Math.random() * 4 + 1,
                    alpha: 1,
                    decay: 0.02 + Math.random() * 0.03,
                    color: color
                });
            }
        }

        function gameLoop() {
            if (!isGameRunning) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            updatePlayer();
            drawPlayer();

            updateEnemies();
            drawEnemies();

            updateBullets();
            drawBullets();

            checkCollisions();

            updateParticles();
            drawParticles();

            animationFrameId = requestAnimationFrame(gameLoop);
        }

        function updatePlayer() {
            if (keys.left) player.x -= player.speed;
            if (keys.right) player.x += player.speed;

            if (player.x < player.width / 2) player.x = player.width / 2;
            if (player.x > canvas.width - player.width / 2) player.x = canvas.width - player.width / 2;
        }

        function drawPlayer() {
            ctx.save();
            ctx.translate(player.x, player.y);

            const thrustHeight = Math.sin(Date.now() / 50) * 8 + 12;
            const grad = ctx.createLinearGradient(0, 20, 0, 20 + thrustHeight);
            grad.addColorStop(0, '#f97316');
            grad.addColorStop(1, 'transparent');
            ctx.fillStyle = grad;
            ctx.beginPath();
            ctx.moveTo(-10, 15);
            ctx.lineTo(0, 20 + thrustHeight);
            ctx.lineTo(10, 15);
            ctx.fill();

            ctx.shadowBlur = 15;
            ctx.shadowColor = '#06b6d4';

            ctx.fillStyle = '#1e293b';
            ctx.strokeStyle = '#06b6d4';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(0, -10);
            ctx.lineTo(-25, 15);
            ctx.lineTo(-10, 10);
            ctx.lineTo(0, 15);
            ctx.lineTo(10, 10);
            ctx.lineTo(25, 15);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            ctx.fillStyle = '#0284c7';
            ctx.beginPath();
            ctx.moveTo(0, -25);
            ctx.lineTo(-12, 10);
            ctx.lineTo(12, 10);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            ctx.fillStyle = '#eab308';
            ctx.beginPath();
            ctx.arc(0, -5, 6, 0, Math.PI * 2);
            ctx.fill();

            ctx.restore();
        }

        function updateBullets() {
            for (let i = bullets.length - 1; i >= 0; i--) {
                bullets[i].y -= bullets[i].speed;
                if (bullets[i].y < 0) {
                    bullets.splice(i, 1);
                }
            }
        }

        function drawBullets() {
            for (let b of bullets) {
                ctx.save();
                ctx.shadowBlur = 10;
                ctx.shadowColor = b.color;
                ctx.fillStyle = b.color;
                ctx.beginPath();
                ctx.arc(b.x, b.y, b.width / 2, Math.PI, 0);
                ctx.lineTo(b.x + b.width / 2, b.y + b.height);
                ctx.lineTo(b.x - b.width / 2, b.y + b.height);
                ctx.closePath();
                ctx.fill();
                ctx.restore();
            }
        }

        function updateEnemies() {
            let allPassedOrHit = true;

            for (let e of enemies) {
                e.y += e.speed;
                e.pulse += 0.04;

                if (currentRound === 2) {
                    e.x += Math.sin(e.pulse) * 1.9;
                }

                if (e.y < canvas.height + 50) {
                    allPassedOrHit = false;
                }
            }

            if (allPassedOrHit && enemies.length > 0) {
                if (currentRound === 3) {
                    spawnQuizEnemies(quizBank[currentQuizIndex]);
                } else {
                    handleWrongAnswer("모든 우주선을 무사 통과시켰습니다!");
                }
            }
        }

        function drawEnemies() {
            for (let e of enemies) {
                if (e.y < -50 || e.y > canvas.height + 50) continue;

                ctx.save();
                ctx.translate(e.x, e.y);

                const offset = Math.sin(e.pulse) * 4;
                ctx.translate(0, offset);

                ctx.shadowBlur = 15;
                ctx.shadowColor = e.color;

                ctx.fillStyle = '#475569';
                ctx.beginPath();
                ctx.ellipse(0, 5, e.width / 2, e.height / 3, 0, 0, Math.PI * 2);
                ctx.fill();

                ctx.fillStyle = 'rgba(255, 255, 255, 0.15)';
                ctx.strokeStyle = e.color;
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(0, -3, e.width / 3, Math.PI, 0);
                ctx.closePath();
                ctx.fill();
                ctx.stroke();

                ctx.fillStyle = '#1e293b';
                ctx.beginPath();
                ctx.ellipse(0, 5, e.width / 2 + 5, e.height / 5, 0, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                ctx.fillStyle = e.color;
                ctx.beginPath();
                ctx.arc(-15, 6, 3, 0, Math.PI * 2);
                ctx.arc(0, 9, 3, 0, Math.PI * 2);
                ctx.arc(15, 6, 3, 0, Math.PI * 2);
                ctx.fill();

                ctx.shadowBlur = 0;
                ctx.fillStyle = '#ffffff';
                ctx.font = 'bold 15px Nanum Gothic';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                
                const txtWidth = ctx.measureText(e.text).width;
                ctx.fillStyle = 'rgba(15, 23, 42, 0.9)';
                ctx.fillRect(-txtWidth/2 - 8, -35, txtWidth + 16, 22);
                ctx.strokeStyle = e.color;
                ctx.lineWidth = 1;
                ctx.strokeRect(-txtWidth/2 - 8, -35, txtWidth + 16, 22);

                ctx.fillStyle = '#ffffff';
                ctx.fillText(currentRound === 3 ? e.text.toUpperCase() : e.text, 0, -23);

                ctx.restore();
            }
        }

        function checkCollisions() {
            for (let bIdx = bullets.length - 1; bIdx >= 0; bIdx--) {
                const b = bullets[bIdx];

                for (let eIdx = enemies.length - 1; eIdx >= 0; eIdx--) {
                    const e = enemies[eIdx];

                    if (b.x > e.x - e.width / 2 && b.x < e.x + e.width / 2 &&
                        b.y > e.y - e.height / 2 && b.y < e.y + e.height / 2) {
                        
                        createExplosion(e.x, e.y, e.color, 12);
                        bullets.splice(bIdx, 1);

                        if (currentRound === 3) {
                            handleSpellingHit(e);
                        } else {
                            if (e.isCorrect) {
                                handleCorrectAnswer();
                            } else {
                                handleWrongAnswer(`'${e.text}'은(는) 틀린 오답입니다!`);
                            }
                        }
                        break;
                    }
                }
            }

            for (let e of enemies) {
                const dist = Math.hypot(player.x - e.x, player.y - e.y);
                if (dist < 34) {
                    createExplosion(e.x, e.y, '#f43f5e', 18);
                    handleWrongAnswer("우주선 충돌 피해를 입었습니다!");
                    break;
                }
            }
        }

        function handleSpellingHit(hitEnemy) {
            const quiz = quizBank[currentQuizIndex];
            const targetChar = quiz.answer[collectedSpelling.length].toUpperCase();

            if (hitEnemy.text === targetChar) {
                collectedSpelling += targetChar.toLowerCase();
                showToast(`알파벳 [ ${targetChar} ] 격추 성공! 👍`);
                updateSpellingDisplay(quiz.answer);

                if (collectedSpelling === quiz.answer.toLowerCase()) {
                    handleCorrectAnswer();
                } else {
                    spawnQuizEnemies(quiz);
                }
            } else {
                handleWrongAnswer(`순서 오류! (정답: ${targetChar}, 저격: ${hitEnemy.text}) 알파벳 조합이 리셋됩니다.`);
                collectedSpelling = "";
                updateSpellingDisplay(quiz.answer);
                spawnQuizEnemies(quiz);
            }
        }

        function updateParticles() {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.alpha -= p.decay;
                if (p.alpha <= 0) {
                    particles.splice(i, 1);
                }
            }
        }

        function drawParticles() {
            for (let p of particles) {
                ctx.save();
                ctx.globalAlpha = p.alpha;
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }
        }

        // ----------------------------------------------------
        // [8] 결과 기록 및 대시보드 백엔드 연동 (규칙 1, 2 준수)
        // ----------------------------------------------------
        let studentInfo = { class: "", number: "", name: "" };

        function handleCorrectAnswer() {
            score += 10;
            document.getElementById('display-score').textContent = score;
            showToast(`정답 격파 성공! (+10점) 🎉`);
            
            enemies = [];
            currentQuizIndex++;
            
            setTimeout(() => {
                loadQuiz(currentQuizIndex);
            }, 800);
        }

        function handleWrongAnswer(reasonMessage) {
            lives--;
            updateLivesUI();
            showToast(`위기! ${reasonMessage} (하트 -1)`);

            if (lives <= 0) {
                endGame(false);
                return;
            }

            if (currentQuizIndex < quizBank.length) {
                spawnQuizEnemies(quizBank[currentQuizIndex]);
            }
        }

        function endGame(isCleared) {
            isGameRunning = false;
            if (animationFrameId) cancelAnimationFrame(animationFrameId);

            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('result-screen').classList.remove('hidden');

            document.getElementById('result-student').textContent = `6학년 ${studentInfo.class}반 ${studentInfo.number}번 ${studentInfo.name}`;
            document.getElementById('result-score').textContent = `${score} 점`;
            document.getElementById('result-correct').textContent = `${currentQuizIndex} / ${quizBank.length}`;

            const feedback = document.getElementById('result-feedback');
            if (isCleared && score >= 250) {
                feedback.innerHTML = "🏆 <strong class='text-emerald-400'>완전무결!</strong> 6학년 2022 개정 영어 마스터 레벨에 도달하셨습니다! 선생님께 자랑해 보세요.";
            } else if (score >= 150) {
                feedback.innerHTML = "✨ <strong class='text-blue-400'>대단한 실력!</strong> 영어를 아주 성실하게 복습했군요. 다음 수업 시간의 에이스는 바로 당신입니다!";
            } else {
                feedback.innerHTML = "✍ <strong class='text-yellow-400'>노력의 찬사!</strong> 포기하지 않고 끝까지 플레이한 모습이 멋집니다. 전구 힌트를 사용해 백점에 다시 도전해 보아요!";
            }

            saveScoreToCloud();
        }

        async function saveScoreToCloud() {
            const statusIndicator = document.getElementById('result-firebase-status');
            if (!db || !dbUser) {
                statusIndicator.innerHTML = "<i class='fa-solid fa-circle-exclamation text-yellow-400 mr-1'></i> 로컬 오프라인 모드로 결과가 기기 메모리에 임시 캐싱되었습니다.";
                saveToLocalStorageFallback();
                return;
            }

            try {
                const recordRef = db.doc(`artifacts/${appId}/public/data/studentRecords/${studentInfo.class}-${studentInfo.number}`);
                
                await recordRef.set({
                    class: parseInt(studentInfo.class),
                    number: parseInt(studentInfo.number),
                    name: studentInfo.name,
                    score: score,
                    correctCount: currentQuizIndex,
                    timestamp: firebase.firestore.FieldValue.serverTimestamp()
                });

                statusIndicator.innerHTML = "<i class='fa-solid fa-cloud-check text-emerald-400 mr-1'></i> 학습 데이터가 교사용 현황판 서버에 성공적으로 전송 완료되었습니다!";
            } catch (error) {
                console.error("Cloud Save Error: ", error);
                statusIndicator.innerHTML = "<i class='fa-solid fa-circle-exclamation text-red-400 mr-1'></i> 서버 임시 접속 원활치 않음. (결과 보드 복사를 이용해 주세요)";
                saveToLocalStorageFallback();
            }
        }

        function saveToLocalStorageFallback() {
            try {
                let records = JSON.parse(localStorage.getItem('english_offline_records') || '[]');
                const newRecord = {
                    class: parseInt(studentInfo.class),
                    number: parseInt(studentInfo.number),
                    name: studentInfo.name,
                    score: score,
                    correctCount: currentQuizIndex,
                    timestamp: new Date().toISOString()
                };
                records = records.filter(r => !(r.class === newRecord.class && r.number === newRecord.number));
                records.push(newRecord);
                localStorage.setItem('english_offline_records', JSON.stringify(records));
            } catch (e) {
                console.warn("Storage write blocked inside iframe sandbox.");
            }
        }

        // ----------------------------------------------------
        // [9] 교사용 전용 비밀번호 잠금 대시보드 시스템 (보안 강화)
        // ----------------------------------------------------
        function openTeacherDashboard() {
            const password = prompt("교사용 실시간 현황판 보안 확인\n비밀번호를 입력하십시오.");
            if (password === null) return; 
            if (password !== "qwert") {
                showToast("비밀번호가 일치하지 않습니다! 🔒");
                return;
            }

            showToast("보안 승인 완료. 교사용 현황판을 엽니다.");
            
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('result-screen').classList.add('hidden');
            document.getElementById('teacher-screen').classList.remove('hidden');

            renderRecords(localRecords);
        }

        function closeTeacherDashboard() {
            document.getElementById('teacher-screen').classList.add('hidden');
            if (score > 0 || lives <= 0) {
                document.getElementById('result-screen').classList.remove('hidden');
            } else {
                document.getElementById('login-screen').classList.remove('hidden');
            }
        }

        function attachDashboardListener() {
            if (!db) return;

            try {
                const recordsCollection = db.collection(`artifacts/${appId}/public/data/studentRecords`);
                
                recordsCollection.onSnapshot((snapshot) => {
                    const fetched = [];
                    snapshot.forEach((doc) => {
                        fetched.push(doc.data());
                    });
                    
                    localRecords = fetched;
                    renderRecords(fetched);
                }, (error) => {
                    console.warn("Firestore Read Restrict or Offline. Falling back to Local Cache.");
                    loadLocalFallbackRecords();
                });
            } catch (error) {
                loadLocalFallbackRecords();
            }
        }

        function loadLocalFallbackRecords() {
            try {
                const records = JSON.parse(localStorage.getItem('english_offline_records') || '[]');
                localRecords = records;
                renderRecords(records);
            } catch (e) {
                renderRecords([]);
            }
        }

        function filterTeacherRecords() {
            const filterVal = document.getElementById('teacher-class-filter').value;
            if (filterVal === "all") {
                renderRecords(localRecords);
            } else {
                const filtered = localRecords.filter(r => r.class === parseInt(filterVal));
                renderRecords(filtered);
            }
        }

        function renderRecords(recordsList) {
            const tbody = document.getElementById('student-records-body');
            const totalStudentsEl = document.getElementById('stats-total-students');
            const avgScoreEl = document.getElementById('stats-avg-score');

            if (!tbody) return;

            const sortedRecords = [...recordsList].sort((a, b) => {
                if (a.class !== b.class) return a.class - b.class;
                return a.number - b.number;
            });

            if (sortedRecords.length === 0) {
                tbody.innerHTML = `
                    <tr>
                        <td colspan="6" class="text-center py-8 text-slate-500 text-xs">
                            <i class="fa-solid fa-triangle-exclamation mr-1"></i> 등록된 학생 데이터가 없습니다. 학생들이 먼저 게임을 끝내야 합니다.
                        </td>
                    </tr>
                `;
                totalStudentsEl.textContent = "0 명";
                avgScoreEl.textContent = "0 점";
                return;
            }

            const total = sortedRecords.length;
            const sum = sortedRecords.reduce((acc, r) => acc + (r.score || 0), 0);
            const avg = Math.round(sum / total);

            totalStudentsEl.textContent = `${total} 명`;
            avgScoreEl.textContent = `${avg} 점`;

            let html = "";
            sortedRecords.forEach((r) => {
                let timeStr = "-";
                if (r.timestamp) {
                    const d = r.timestamp.toDate ? r.timestamp.toDate() : new Date(r.timestamp);
                    timeStr = `${d.getMonth() + 1}/${d.getDate()} ${d.getHours().toString().padStart(2, '0')}:${d.getMinutes().toString().padStart(2, '0')}`;
                }

                html += `
                    <tr class="border-b border-slate-900 hover:bg-slate-900/40 transition">
                        <td class="px-4 py-3 text-center font-bold text-indigo-400">${r.class}반</td>
                        <td class="px-4 py-3 text-center text-slate-400">${r.number}번</td>
                        <td class="px-4 py-3 font-bold text-white">${r.name}</td>
                        <td class="px-4 py-3 text-right font-black text-yellow-400 text-sm">${r.score}점</td>
                        <td class="px-4 py-3 text-center text-emerald-400 font-bold">${r.correctCount}개</td>
                        <td class="px-4 py-3 text-center text-slate-500 text-[10px]">${timeStr}</td>
                    </tr>
                `;
            });

            tbody.innerHTML = html;
        }

        async function clearAllDataDemoPrompt() {
            const pass = prompt("현황판 데이터를 일괄 초기화하시겠습니까? 초기화 암호를 입력하세요.");
            if (pass !== "qwert") {
                showToast("초기화 승인이 거부되었습니다.");
                return;
            }

            try {
                if (db) {
                    const recordsSnapshot = await db.collection(`artifacts/${appId}/public/data/studentRecords`).get();
                    const batch = db.batch();
                    recordsSnapshot.forEach((doc) => {
                        batch.delete(doc.ref);
                    });
                    await batch.commit();
                }
                localStorage.removeItem('english_offline_records');
                showToast("학습 성적 기록 데이터베이스가 완전 초기화되었습니다!");
                localRecords = [];
                renderRecords([]);
            } catch (error) {
                showToast("초기화 도중 오류가 발생했습니다.");
            }
        }
    </script>
</body>
</html># review1-6_v3
