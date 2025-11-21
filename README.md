# SIPERSIS-v4.0
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SIPERSIS 3.0: Sistem Sekolah Terintegrasi</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Inter Font and Lucide Icons -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        /* Dark Theme Styling */
        :root {
            --bg-dark: #111827; /* Gray-900 */
            --bg-card: #1f2937; /* Gray-800 */
            --text-light: #f3f4f6; /* Gray-100 */
            --primary-color: #3b82f6; /* Blue-500 */
            --accent-color: #10b981; /* Emerald-500 for positive */
            --danger-color: #ef4444; /* Red-500 for critical */
            --panic-color: #f97316; /* Orange-500 for panic */
        }
        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--bg-dark);
            color: var(--text-light);
            /* Add padding bottom to account for the fixed panic button */
            padding-bottom: 7rem; 
        }
        .card {
            background-color: var(--bg-card);
            border-radius: 0.75rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.2), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            transition: all 0.3s ease;
        }
        .sidebar-item:hover {
            background-color: #374151; /* Gray-700 hover */
        }
        .sidebar-item.active {
            background-color: var(--primary-color);
            color: var(--text-light);
            box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.3);
        }
        .status-badge { @apply px-3 py-1 text-xs font-semibold rounded-full; }
        .status-Menunggu { @apply bg-yellow-800 text-yellow-100; }
        .status-Diproses { @apply bg-blue-800 text-blue-100; }
        .status-Selesai { @apply bg-green-800 text-green-100; }
        .status-Apresiasi { @apply bg-accent-color text-white; } 

        /* Animation for Motivation Text */
        @keyframes fadeInMove {
            0% { opacity: 0; transform: translateY(10px); }
            100% { opacity: 1; transform: translateY(0); }
        }
        .motivational-text {
            animation: fadeInMove 1s ease-out;
        }

        /* Responsive Chat UI */
        #ai-chat-box {
            height: 60vh;
            max-height: 500px;
        }

        /* Custom scrollbar for dark theme */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: var(--bg-card); }
        ::-webkit-scrollbar-thumb { background: #4b5563; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #6b7280; }

        /* Mood Selection Styles */
        .mood-item {
            @apply p-4 rounded-xl cursor-pointer transition transform hover:scale-105 border-4 border-transparent;
        }
        .mood-item.selected {
            @apply border-primary-color bg-gray-700 shadow-lg;
        }
    </style>
</head>
<body class="min-h-screen flex antialiased">

    <!-- Sidebar (Navigasi) -->
    <aside class="w-64 flex-shrink-0 card m-4 p-4 space-y-4 hidden md:block">
        <div class="text-xl font-bold mb-6 text-primary-color">SIPERSIS 3.0</div>
        <nav class="space-y-2" id="sidebar-nav">
            <!-- Nav Items Loaded by JS -->
        </nav>
    </aside>

    <!-- Main Content Area -->
    <main class="flex-1 p-4 md:p-8 overflow-y-auto">
        
        <!-- Mobile Header (for Nav) -->
        <header class="md:hidden flex justify-between items-center mb-4">
            <h1 class="text-2xl font-bold text-primary-color">SIPERSIS</h1>
            <button id="mobile-menu-btn" class="p-2 rounded-lg bg-gray-700 hover:bg-gray-600">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path></svg>
            </button>
        </header>

        <!-- Main View Container -->
        <div id="main-view-container" class="space-y-8">
            <!-- Content will be loaded here by JavaScript -->
        </div>

        <!-- Notification Message Container -->
        <div id="message-container" class="fixed bottom-4 right-4 z-20"></div>

    </main>
    
    <!-- Modal for Mobile Menu -->
    <div id="mobile-menu-modal" class="fixed inset-0 bg-black bg-opacity-75 z-50 hidden md:hidden">
        <div class="w-64 bg-gray-800 h-full p-6 space-y-4">
            <div class="flex justify-between items-center mb-6">
                <div class="text-xl font-bold text-primary-color">Menu SIPERSIS</div>
                <button onclick="toggleMobileMenu()" class="p-2 rounded-full hover:bg-gray-700">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/24/24/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>
            <nav class="space-y-2" id="mobile-sidebar-nav">
                <!-- Nav Items Loaded by JS -->
            </nav>
        </div>
    </div>
    
    <!-- Emergency Button (Fixed Footer) -->
    <div class="fixed bottom-0 left-0 right-0 p-4 bg-gray-900 bg-opacity-95 shadow-2xl z-40">
        <button id="panic-button" onclick="triggerPanic()" 
                class="w-full md:w-auto md:max-w-xs mx-auto block bg-panic-color text-white py-3 rounded-xl font-extrabold text-lg transition transform hover:scale-[1.02] active:scale-95 shadow-xl ring-4 ring-panic-color ring-opacity-50">
            <i data-lucide="Zap" class="inline-block w-6 h-6 mr-3"></i> EMERGENCY: PANIC BUTTON
        </button>
    </div>


    <script>
        // Global State & Constants
        let currentView = 'dashboard';
        let currentReportFilter = 'all'; // Filter for staff report check
        
        const APP_STATE = {
            // Mock Student Data
            userId: 'student-12345',
            nisn: '1234567890',
            name: 'Andi Siswa',
            class_name: 'XI IPA 2',
            role: 'student' // 'student' or 'staff' - use 'staff' to see admin features
        };

        // Static data keys for localStorage
        const REPORTS = 'sipersis_reports';
        const POSITIVE_REPORTS = 'sipersis_positive_reports';
        const AI_CHAT_HISTORY = 'sipersis_ai_chat';
        const ACADEMIC_GRADES = 'sipersis_grades'; // NEW: For Academic Reporting
        const MOOD_HISTORY = 'sipersis_mood_history'; // NEW: For Emotional Check-In
        const LIBRARY_CATALOG = 'sipersis_library_catalog'; // NEW: For Digital Library
        const MOTIVATION_QUOTES = [
            "Mimpikan masa depanmu, tapi fokuslah pada hari ini.",
            "Kegagalan adalah kesempatan untuk memulai lagi dengan lebih cerdas.",
            "Disiplin adalah jembatan antara tujuan dan pencapaian.",
            "Berani bermimpi, berani bertindak. Semangat!",
            "Jangan pernah berhenti belajar, karena hidup tak pernah berhenti mengajarkan.",
        ];
        const MOOD_OPTIONS = [
            { key: 'good', label: 'Luar Biasa!', emoji: '😊', color: 'text-green-500' },
            { key: 'ok', label: 'Cukup Baik', emoji: '🙂', color: 'text-primary-color' },
            { key: 'stressed', label: 'Merasa Tertekan', emoji: '😟', color: 'text-yellow-500' },
            { key: 'sad', label: 'Sedih/Buruk', emoji: '😔', color: 'text-danger-color' },
        ];
        
        // Mock Subject Data
        const MOCK_SUBJECTS = ['Matematika', 'Fisika', 'Sejarah', 'Bahasa Inggris', 'Biologi'];

        // --- Utility Functions (Simulasi Backend/DB with localStorage) ---

        const db = {
            get: (key, defaultValue = []) => {
                const data = localStorage.getItem(key);
                if (!data) {
                    if (key === ACADEMIC_GRADES) return generateMockGrades(); // NEW: Initial mock grades
                    if (key === LIBRARY_CATALOG) return generateMockLibraryCatalog(); // NEW: Initial mock library
                    return defaultValue;
                }
                return JSON.parse(data) || defaultValue;
            },
            set: (key, value) => localStorage.setItem(key, JSON.stringify(value)),
        };
        
        // NEW: Initial Mock Grade Data Generator
        function generateMockGrades() {
            const grades = [];
            MOCK_SUBJECTS.forEach(subject => {
                grades.push({
                    subject: subject,
                    type: 'Tugas Harian',
                    score: Math.floor(Math.random() * 20) + 75,
                    date: new Date(2025, Math.floor(Math.random() * 4) + 7, Math.floor(Math.random() * 28) + 1).toISOString().substring(0, 10),
                    studentNisn: APP_STATE.nisn
                });
                grades.push({
                    subject: subject,
                    type: 'Kuis',
                    score: Math.floor(Math.random() * 20) + 70,
                    date: new Date(2025, Math.floor(Math.random() * 4) + 7, Math.floor(Math.random() * 28) + 1).toISOString().substring(0, 10),
                    studentNisn: APP_STATE.nisn
                });
                grades.push({
                    subject: subject,
                    type: 'Ujian Tengah Semester',
                    score: Math.floor(Math.random() * 25) + 65,
                    date: new Date(2025, 10, 15).toISOString().substring(0, 10),
                    studentNisn: APP_STATE.nisn
                });
            });
            return grades;
        }

        // NEW: Initial Mock Library Catalog Generator
        function generateMockLibraryCatalog() {
            return [
                { id: 1, title: 'Fisika Dasar: Mekanika Kuantum', author: 'Dr. Ahmad Dani', available: 5, total: 10, category: 'Sains' },
                { id: 2, title: 'Novel: Laskar Pelangi', author: 'Andrea Hirata', available: 12, total: 15, category: 'Fiksi' },
                { id: 3, title: 'Sejarah Dunia Jilid 1', author: 'Prof. Yuniarti', available: 2, total: 5, category: 'Sosial' },
                { id: 4, title: 'Kumpulan Soal SBMPTN 2024', author: 'Tim Edukasi', available: 0, total: 8, category: 'Ujian' },
            ];
        }

        function showMessage(type, message) {
            const container = document.getElementById('message-container');
            const color = type === 'success' ? 'bg-green-500' : type === 'error' ? 'bg-danger-color' : type === 'panic' ? 'bg-panic-color' : 'bg-primary-color';
            const icon = type === 'success' ? 'CheckCircle' : type === 'error' ? 'XCircle' : type === 'panic' ? 'AlertTriangle' : 'Info';
            
            const alertHtml = `
                <div class="${color} text-white p-4 rounded-lg shadow-xl mb-3 flex items-center transform transition duration-500 ease-out translate-y-0 opacity-100" style="min-width: 250px;">
                    <i data-lucide="${icon}" class="w-5 h-5 mr-3"></i>
                    <p class="text-sm font-medium">${message}</p>
                </div>
            `;
            const alertElement = document.createElement('div');
            alertElement.innerHTML = alertHtml;
            container.appendChild(alertElement);

            lucide.createIcons(); 

            // Auto-remove after 5 seconds
            setTimeout(() => {
                alertElement.classList.add('opacity-0', 'translate-y-4');
                setTimeout(() => alertElement.remove(), 500);
            }, 5000);
        }

        function formatDate(isoString) {
            if (!isoString) return '-';
            const date = new Date(isoString);
            return date.toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric', hour: '2-digit', minute: '2-digit' });
        }
        
        // --- Emergency Button Logic ---
        function triggerPanic() {
            if (!confirm("ANDA YAKIN INGIN MENGAKTIFKAN TOMBOL DARURAT? Tindakan ini akan mengirim notifikasi lokasi segera ke Guru Piket dan Keamanan.")) {
                return;
            }

            showMessage('panic', '⚠️ TOMBOL DARURAT AKTIF! Lokasi Anda sedang dikirim ke tim keamanan.');
            
            // 1. Dapatkan lokasi (Simulasi GPS)
            const mockLocation = "Lingkungan Sekolah: Gedung A, Lantai 2, dekat Ruang UKS";
            
            // 2. Kirim Notifikasi (Simulasi)
            const timestamp = new Date().toLocaleTimeString('id-ID');
            console.warn(`[PANIC ALERT] Waktu: ${timestamp}, Pelapor: ${APP_STATE.name}, Lokasi: ${mockLocation}`);
            
            // 3. Tampilkan pesan berhasil
            showModal('panic-modal', `
                <div class="p-6 bg-gray-700 rounded-lg shadow-2xl border-t-8 border-panic-color space-y-4">
                    <h3 class="text-3xl font-bold text-panic-color">DARURAT TERKIRIM!</h3>
                    <p class="text-gray-200">Tim keamanan dan Guru Piket telah menerima notifikasi darurat Anda.</p>
                    <p class="text-sm text-gray-400">Lokasi Terkirim (Simulasi): <span class="font-mono">${mockLocation}</span></p>
                    <p class="text-sm font-semibold mt-4">TETAP TENANG dan tunggu bantuan tiba. Jangan tinggalkan lokasi jika tidak aman.</p>
                    <button onclick="closeModal('panic-modal')" class="mt-4 w-full bg-primary-color text-white py-2 rounded-lg hover:bg-blue-600">Saya Mengerti</button>
                </div>
            `);
        }


        // --- Renderer Functions ---

        const navItems = [
            { id: 'dashboard', name: 'Dashboard Utama', icon: 'LayoutDashboard', role: 'student' },
            { id: 'report-check', name: 'Cek Laporan', icon: 'ListChecks', role: 'staff', color: 'text-primary-color' },
            { id: 'academic-report', name: 'Pelaporan Akademik', icon: 'BookOpenCheck', role: 'staff', color: 'text-accent-color' }, // NEW
            { id: 'quick-report', name: 'Pelaporan Cepat', icon: 'Bolt', role: 'student', color: 'text-danger-color' },
            { id: 'positive-report', name: 'Laporan Prestasi', icon: 'Award', role: 'student', color: 'text-accent-color' },
            { id: 'emotional-check', name: 'Emotional Check-In', icon: 'Smile', role: 'student', color: 'text-yellow-500' }, // NEW
            { id: 'digital-library', name: 'Perpustakaan Digital', icon: 'Library', role: 'student', color: 'text-purple-400' }, // NEW
            { id: 'ai-consultation', name: 'Konsultasi AI', icon: 'Bot', role: 'student' },
            { id: 'e-learning', name: 'E-Learning & Tugas', icon: 'GraduationCap', role: 'student' },
            { id: 'announcement', name: 'Pengumuman Sekolah', icon: 'BellRing', role: 'student' },
            { id: 'calendar', name: 'Kalender Kegiatan', icon: 'Calendar', role: 'student' },
        ];
        
        function renderNavigation(containerId) {
            const container = document.getElementById(containerId);
            // Filter nav items based on user role simulation
            const itemsToShow = navItems.filter(item => 
                (APP_STATE.role === 'staff' && item.role === 'staff') || // Staff sees staff features
                (APP_STATE.role === 'student' && item.role !== 'staff') || // Student sees student features
                (item.id === 'dashboard') // Everyone sees dashboard
            );

            container.innerHTML = itemsToShow.map(item => `
                <a href="#" onclick="changeView('${item.id}')" 
                   class="sidebar-item flex items-center p-3 font-medium rounded-lg transition ${item.id === currentView ? 'active' : 'text-gray-300'}">
                    <i data-lucide="${item.icon}" class="w-5 h-5 mr-3 ${item.color || ''}"></i>
                    ${item.name}
                </a>
            `).join('');
            lucide.createIcons();
        }

        function changeView(viewId) {
            currentView = viewId;
            document.getElementById('mobile-menu-modal').classList.add('hidden');
            renderMainView();
            renderNavigation('sidebar-nav');
            renderNavigation('mobile-sidebar-nav'); 
            window.scrollTo(0, 0); 
        }

        function toggleMobileMenu() {
            document.getElementById('mobile-menu-modal').classList.toggle('hidden');
        }
        
        // --- View: Academic Reporting (Staff/Teacher Feature) ---

        function renderAcademicReporting() {
            const grades = db.get(ACADEMIC_GRADES).filter(g => g.studentNisn === APP_STATE.nisn);
            
            // Rekap Nilai per Mata Pelajaran
            const subjectRecap = MOCK_SUBJECTS.reduce((acc, subject) => {
                const subjectGrades = grades.filter(g => g.subject === subject);
                const totalScore = subjectGrades.reduce((sum, g) => sum + g.score, 0);
                const average = subjectGrades.length > 0 ? (totalScore / subjectGrades.length).toFixed(1) : 'N/A';
                acc[subject] = { average, count: subjectGrades.length };
                return acc;
            }, {});

            return `
                <div class="space-y-8">
                    <h2 class="text-3xl font-bold text-accent-color flex items-center">
                        <i data-lucide="BookOpenCheck" class="w-7 h-7 mr-3"></i> Pelaporan Akademik & Nilai
                    </h2>
                    <p class="text-gray-400 italic">Hanya dapat diakses oleh staf/guru. Melacak perkembangan nilai siswa.</p>

                    <!-- Mock Input Nilai Tugas/Ujian -->
                    <div class="card p-6">
                        <h3 class="text-xl font-semibold mb-4 border-b border-gray-700 pb-2">1. Input Nilai Baru (Simulasi Staf)</h3>
                        <form id="grade-input-form" class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div>
                                <label for="grade-subject" class="block text-sm font-medium mb-1">Mata Pelajaran</label>
                                <select id="grade-subject" class="w-full px-3 py-2 rounded-lg bg-gray-700 border border-gray-600">
                                    ${MOCK_SUBJECTS.map(s => `<option value="${s}">${s}</option>`).join('')}
                                </select>
                            </div>
                            <div>
                                <label for="grade-type" class="block text-sm font-medium mb-1">Jenis Nilai</label>
                                <select id="grade-type" class="w-full px-3 py-2 rounded-lg bg-gray-700 border border-gray-600">
                                    <option value="Tugas Harian">Tugas Harian</option>
                                    <option value="Kuis">Kuis</option>
                                    <option value="Ujian Tengah Semester">UTS</option>
                                    <option value="Ujian Akhir Semester">UAS</option>
                                </select>
                            </div>
                            <div>
                                <label for="grade-score" class="block text-sm font-medium mb-1">Nilai (0-100)</label>
                                <input type="number" id="grade-score" required min="0" max="100" class="w-full px-3 py-2 rounded-lg bg-gray-700 border border-gray-600">
                            </div>
                            <div class="md:col-span-3 pt-2">
                                <button type="submit" class="w-full bg-accent-color text-white py-2 rounded-lg font-semibold hover:bg-emerald-600 transition">
                                    <i data-lucide="Save" class="inline-block w-5 h-5 mr-2"></i> Simpan Nilai
                                </button>
                            </div>
