<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Absensi Siswa dengan QR Code</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        #qr-reader-results {
            margin-top: 20px;
            padding: 10px;
            border-radius: 8px;
        }
        .success {
            background-color: #d1fae5;
            color: #065f46;
            border: 1px solid #6ee7b7;
        }
        .error {
            background-color: #fee2e2;
            color: #991b1b;
            border: 1px solid #fca5a5;
        }
        .info {
            background-color: #dbeafe;
            color: #1e40af;
            border: 1px solid #93c5fd;
        }
        .modal {
            display: none; /* Hidden by default */
            position: fixed; /* Stay in place */
            z-index: 100; /* Sit on top */
            left: 0;
            top: 0;
            width: 100%; /* Full width */
            height: 100%; /* Full height */
            overflow: auto; /* Enable scroll if needed */
            background-color: rgba(0,0,0,0.5); /* Black w/ opacity */
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 24px;
            border: 1px solid #888;
            width: 90%;
            max-width: 500px;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
        }
        .close-button {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div id="app" class="container mx-auto p-4 md:p-6 max-w-7xl">
        <!-- Notification Bar -->
        <div id="notification-bar" class="hidden fixed top-5 right-5 text-white p-4 rounded-lg shadow-lg z-[101]"></div>

        <!-- Header -->
        <header class="bg-white shadow-md rounded-xl p-6 mb-6 flex justify-between items-center">
            <div>
                <h1 class="text-3xl font-bold text-gray-800">Sistem Absensi QR Code</h1>
                <p class="text-gray-500">Manajemen kehadiran siswa secara modern dan efisien.</p>
            </div>
            <div id="view-switcher" class="flex space-x-2">
                 <button id="show-admin-view-btn" class="bg-blue-600 text-white px-4 py-2 rounded-lg shadow hover:bg-blue-700 transition duration-300">Tampilan Admin</button>
                 <button id="show-student-view-btn" class="bg-green-600 text-white px-4 py-2 rounded-lg shadow hover:bg-green-700 transition duration-300">Tampilan Siswa</button>
            </div>
        </header>

        <!-- Loading Spinner -->
        <div id="loading-spinner" class="hidden fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50">
            <div class="animate-spin rounded-full h-32 w-32 border-t-2 border-b-2 border-white"></div>
        </div>

        <!-- Admin View -->
        <div id="admin-view" class="hidden">
            <!-- Admin Login -->
            <div id="admin-login-section" class="bg-white p-8 rounded-xl shadow-lg max-w-md mx-auto">
                <h2 class="text-2xl font-bold mb-6 text-center text-gray-700">Login Admin</h2>
                <div class="space-y-4">
                    <input type="password" id="admin-password" placeholder="Masukkan Kata Sandi" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <button id="admin-login-btn" class="w-full bg-blue-600 text-white py-2 rounded-lg shadow hover:bg-blue-700 transition duration-300">Masuk</button>
                </div>
                 <p id="login-error" class="text-red-500 text-sm mt-2 text-center"></p>
            </div>
            
            <!-- Admin Dashboard -->
            <div id="admin-dashboard" class="hidden">
                <!-- Stats Cards -->
                <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
                    <div class="bg-white p-4 rounded-xl shadow-lg text-center">
                        <h3 class="text-gray-500 text-sm font-medium">Total Siswa</h3>
                        <p id="total-students-stat" class="text-3xl font-bold">0</p>
                    </div>
                    <div class="bg-white p-4 rounded-xl shadow-lg text-center">
                        <h3 class="text-gray-500 text-sm font-medium">Hadir Hari Ini</h3>
                        <p id="attendance-percentage-stat" class="text-3xl font-bold">0%</p>
                    </div>
                    <div class="bg-white p-4 rounded-xl shadow-lg text-center">
                        <h3 class="text-gray-500 text-sm font-medium">Terlambat</h3>
                        <p id="late-students-stat" class="text-3xl font-bold">0</p>
                    </div>
                    <div class="bg-white p-4 rounded-xl shadow-lg text-center">
                        <h3 class="text-gray-500 text-sm font-medium">Pulang Cepat</h3>
                        <p id="early-leave-stat" class="text-3xl font-bold">0</p>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- Left Column -->
                    <div class="lg:col-span-1 space-y-6">
                        <!-- General Time Settings -->
                        <div class="bg-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-xl font-bold mb-4 border-b pb-2">Pengaturan Waktu Umum</h2>
                            <div class="space-y-3">
                                <div>
                                    <label for="general-start-time" class="block text-sm font-medium text-gray-700">Jam Masuk Umum</label>
                                    <input type="time" id="general-start-time" class="mt-1 w-full px-4 py-2 border rounded-lg">
                                </div>
                                <div>
                                    <label for="general-end-time" class="block text-sm font-medium text-gray-700">Jam Pulang Umum</label>
                                    <input type="time" id="general-end-time" class="mt-1 w-full px-4 py-2 border rounded-lg">
                                </div>
                                <button id="save-general-settings-btn" class="w-full bg-teal-500 text-white py-2 rounded-lg hover:bg-teal-600">Simpan Waktu Umum</button>
                            </div>
                        </div>
                        
                        <!-- Class Management -->
                        <div class="bg-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-xl font-bold mb-4 border-b pb-2">Manajemen Kelas</h2>
                            <div class="flex space-x-2">
                                <input type="text" id="class-name-input" placeholder="Nama Kelas Baru" class="w-full px-4 py-2 border rounded-lg">
                                <button id="add-class-btn" class="bg-indigo-500 text-white px-4 py-2 rounded-lg hover:bg-indigo-600">Tambah</button>
                            </div>
                            <div id="class-list" class="mt-4 space-y-2"></div>
                        </div>

                        <!-- Student Management -->
                        <div class="bg-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-xl font-bold mb-4 border-b pb-2">Manajemen Siswa</h2>
                            <div class="space-y-3">
                                <input type="text" id="student-name" placeholder="Nama Siswa" class="w-full px-4 py-2 border rounded-lg">
                                <input type="text" id="student-id" placeholder="NIS / ID Siswa" class="w-full px-4 py-2 border rounded-lg">
                                <select id="student-class-selector" class="w-full px-4 py-2 border rounded-lg bg-white">
                                    <option value="">Pilih Kelas</option>
                                </select>
                                <button id="add-student-btn" class="w-full bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600">Tambah Siswa</button>
                            </div>
                            <div class="mt-6 border-t pt-4">
                                <h3 class="font-semibold mb-2">Tambah Siswa Massal</h3>
                                <select id="bulk-student-class-selector" class="w-full px-4 py-2 border rounded-lg bg-white mb-2">
                                    <option value="">Pilih Kelas Untuk Siswa Massal</option>
                                </select>
                                <textarea id="bulk-student-input" rows="5" class="w-full px-4 py-2 border rounded-lg" placeholder="Format: NIS,Nama (satu siswa per baris)"></textarea>
                                <button id="bulk-add-student-btn" class="w-full mt-2 bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600">Tambah Massal</button>
                            </div>
                        </div>

                        <!-- QR Code Generator -->
                        <div class="bg-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-xl font-bold mb-4 border-b pb-2">Buat QR Code Kelas</h2>
                            <div class="space-y-3">
                                <select id="class-selector-qr" class="w-full px-4 py-2 border rounded-lg bg-white">
                                    <option value="">Pilih Kelas</option>
                                </select>
                                <button id="generate-qr-btn" class="w-full bg-green-500 text-white py-2 rounded-lg hover:bg-green-600">Buat QR Code Harian</button>
                            </div>
                            <div id="qrcode-container" class="mt-4 p-4 border rounded-lg flex justify-center bg-gray-50 hidden">
                                 <div id="qrcode"></div>
                            </div>
                            <p id="qr-message" class="text-center mt-2 text-sm text-gray-600"></p>
                        </div>
                    </div>
                    <!-- Right Column -->
                    <div class="lg:col-span-2 space-y-6">
                         <div class="bg-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-xl font-bold mb-4 border-b pb-2">Laporan & Daftar Siswa</h2>
                            <div class="flex space-x-2 mb-4">
                                <button id="show-students-tab" class="px-4 py-2 rounded-lg bg-blue-100 text-blue-800 font-semibold">Daftar Siswa</button>
                                <button id="show-reports-tab" class="px-4 py-2 rounded-lg bg-gray-100 text-gray-800">Laporan Kehadiran</button>
                            </div>

                            <!-- Student List Table -->
                            <div id="student-list-content">
                                <div class="overflow-x-auto">
                                    <table class="w-full text-left">
                                        <thead class="bg-gray-50">
                                            <tr>
                                                <th class="p-3">Nama</th>
                                                <th class="p-3">NIS/ID</th>
                                                <th class="p-3">Kelas</th>
                                                <th class="p-3">Aksi</th>
                                            </tr>
                                        </thead>
                                        <tbody id="student-list-table"></tbody>
                                    </table>
                                </div>
                            </div>

                            <!-- Attendance Reports -->
                            <div id="reports-content" class="hidden">
                                 <div class="flex flex-wrap items-center gap-2 mb-4">
                                    <input type="date" id="report-date" class="border rounded-lg px-3 py-2">
                                    <select id="class-selector-report" class="border rounded-lg px-3 py-2 bg-white">
                                        <option value="">Pilih Kelas</option>
                                    </select>
                                    <button id="export-report-btn" class="bg-green-600 text-white px-4 py-2 rounded-lg shadow hover:bg-green-700 transition duration-300">Export CSV</button>
                                 </div>
                                 <div class="overflow-x-auto">
                                    <table class="w-full text-left">
                                        <thead class="bg-gray-50">
                                            <tr>
                                                <th class="p-3">Nama</th>
                                                <th class="p-3">Datang</th>
                                                <th class="p-3">Pulang</th>
                                                <th class="p-3">Status</th>
                                                <th class="p-3">Keterangan</th>
                                            </tr>
                                        </thead>
                                        <tbody id="attendance-report-table"></tbody>
                                    </table>
                                 </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Student View -->
        <div id="student-view" class="bg-white p-6 md:p-8 rounded-xl shadow-lg max-w-2xl mx-auto text-center">
            <h2 class="text-2xl font-bold mb-2 text-gray-700">Absensi Siswa</h2>
            <p class="text-center text-gray-500 mb-6">Pilih jenis absensi, lalu arahkan kamera ke QR Code Anda.</p>
            
            <div id="student-action-buttons" class="flex space-x-4 justify-center mb-6">
                <button id="check-in-btn" class="w-full bg-green-600 text-white py-3 rounded-lg shadow hover:bg-green-700 transition duration-300 text-lg font-semibold">Absen Datang</button>
                <button id="check-out-btn" class="w-full bg-red-600 text-white py-3 rounded-lg shadow hover:bg-red-700 transition duration-300 text-lg font-semibold">Absen Pulang</button>
            </div>

            <div id="qr-reader-container" class="hidden">
                 <p id="scan-instruction" class="text-center text-gray-600 font-medium mb-4"></p>
                <div id="qr-reader" class="w-full border-2 border-dashed border-gray-300 rounded-lg overflow-hidden" style="max-width: 500px; margin: 0 auto;"></div>
                <button id="cancel-scan-btn" class="mt-4 bg-gray-500 text-white px-4 py-2 rounded-lg hover:bg-gray-600">Batal</button>
            </div>
            <div id="qr-reader-results" class="text-center font-medium mt-4"></div>
        </div>
    </div>
    
    <!-- Student ID Modal -->
    <div id="student-id-modal" class="modal">
        <div class="modal-content">
            <span class="close-button" id="close-modal-btn">&times;</span>
            <h3 class="text-xl font-bold mb-4">Konfirmasi Kehadiran</h3>
            <p class="mb-4 text-gray-600">Silakan masukkan NIS/ID Anda untuk mencatat kehadiran.</p>
            <input type="text" id="modal-student-id-input" placeholder="Masukkan NIS / ID Siswa" class="w-full px-4 py-2 border rounded-lg mb-4">
            <button id="submit-attendance-btn" class="w-full bg-green-600 text-white py-2 rounded-lg hover:bg-green-700">Kirim Absensi</button>
        </div>
    </div>
    
    <!-- Class Settings Modal -->
    <div id="class-settings-modal" class="modal">
        <div class="modal-content">
             <span class="close-button" id="close-class-modal-btn">&times;</span>
             <h3 class="text-xl font-bold mb-4">Pengaturan Waktu Kelas: <span id="modal-class-name"></span></h3>
             <input type="hidden" id="modal-class-id">
             <div class="space-y-3">
                <label for="modal-start-time" class="block text-sm font-medium text-gray-700">Jam Masuk</label>
                <input type="time" id="modal-start-time" class="w-full px-4 py-2 border rounded-lg">
                <label for="modal-end-time" class="block text-sm font-medium text-gray-700">Jam Pulang</label>
                <input type="time" id="modal-end-time" class="w-full px-4 py-2 border rounded-lg">
                <button id="save-class-settings-btn" class="w-full bg-indigo-500 text-white py-2 rounded-lg hover:bg-indigo-600">Simpan Pengaturan</button>
            </div>
        </div>
    </div>

    <!-- Student QR Code Modal -->
    <div id="student-qr-modal" class="modal">
        <div class="modal-content text-center">
            <span class="close-button" id="close-student-qr-modal-btn">&times;</span>
            <h3 class="text-xl font-bold mb-2">QR Code Absensi untuk</h3>
            <p id="modal-qr-student-name" class="text-lg font-medium mb-4"></p>
            <div id="student-qrcode-display" class="p-4 border rounded-lg inline-block bg-gray-50"></div>
            <p class="mt-4 text-sm text-gray-600">Simpan atau cetak QR Code ini untuk absensi.</p>
        </div>
    </div>


    <script type="module">
        // Import Firebase modules
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, getDocs, onSnapshot, query, where, doc, deleteDoc, setDoc, getDoc, writeBatch, updateDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // --- PRE-CONFIGURED GLOBAL VARIABLES ---
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-absensi-app';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : { apiKey: "DEMO_API_KEY", authDomain: "DEMO_AUTH_DOMAIN", projectId: "DEMO_PROJECT_ID" };
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // --- FIREBASE INITIALIZATION ---
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        
        // Firestore collection references
        const studentsCollection = collection(db, `/artifacts/${appId}/public/data/students`);
        const attendanceCollection = collection(db, `/artifacts/${appId}/public/data/attendance`);
        const classesCollection = collection(db, `/artifacts/${appId}/public/data/classes`);
        const settingsCollection = collection(db, `/artifacts/${appId}/public/data/settings`);
        
        // --- STATE MANAGEMENT ---
        let currentUser = null;
        let qrCodeData = null;
        let html5QrCode;
        let allStudents = [];
        let allClassSettings = [];
        let scanMode = null; // 'checkIn' or 'checkOut'
        let generalSettings = { startTime: '07:00', endTime: '15:00' };
       
        const ADMIN_PASSWORD = "admin123";
        const QR_SECRET = "RAHASIA_GURUMU";

        // --- DOM ELEMENTS ---
        const adminView = document.getElementById('admin-view');
        const studentView = document.getElementById('student-view');
        const showAdminBtn = document.getElementById('show-admin-view-btn');
        const showStudentBtn = document.getElementById('show-student-view-btn');
        const adminLoginSection = document.getElementById('admin-login-section');
        const adminDashboard = document.getElementById('admin-dashboard');
        const adminLoginBtn = document.getElementById('admin-login-btn');
        const adminPasswordInput = document.getElementById('admin-password');
        const loginError = document.getElementById('login-error');
        const loadingSpinner = document.getElementById('loading-spinner');
        
        // Admin Dashboard elements
        const addStudentBtn = document.getElementById('add-student-btn');
        const studentNameInput = document.getElementById('student-name');
        const studentIdInput = document.getElementById('student-id');
        const studentClassSelector = document.getElementById('student-class-selector');
        const studentListTable = document.getElementById('student-list-table');
        const bulkStudentInput = document.getElementById('bulk-student-input');
        const bulkAddStudentBtn = document.getElementById('bulk-add-student-btn');
        const bulkStudentClassSelector = document.getElementById('bulk-student-class-selector');
        const classNameInput = document.getElementById('class-name-input');
        const addClassBtn = document.getElementById('add-class-btn');
        const classListDiv = document.getElementById('class-list');
        const earlyLeaveStat = document.getElementById('early-leave-stat');
        const saveGeneralSettingsBtn = document.getElementById('save-general-settings-btn');
        const generalStartTimeInput = document.getElementById('general-start-time');
        const generalEndTimeInput = document.getElementById('general-end-time');

        // QR Code elements
        const generateQrBtn = document.getElementById('generate-qr-btn');
        const classSelectorQr = document.getElementById('class-selector-qr');
        const qrcodeContainer = document.getElementById('qrcode-container');
        const qrcodeDiv = document.getElementById('qrcode');
        const qrMessage = document.getElementById('qr-message');
        
        // Report elements
        const showStudentsTab = document.getElementById('show-students-tab');
        const showReportsTab = document.getElementById('show-reports-tab');
        const studentListContent = document.getElementById('student-list-content');
        const reportsContent = document.getElementById('reports-content');
        const reportDateInput = document.getElementById('report-date');
        const classSelectorReport = document.getElementById('class-selector-report');
        const attendanceReportTable = document.getElementById('attendance-report-table');
        const exportReportBtn = document.getElementById('export-report-btn');
        
        // Student View elements
        const qrReader = document.getElementById('qr-reader');
        const qrReaderResults = document.getElementById('qr-reader-results');
        const qrReaderContainer = document.getElementById('qr-reader-container');
        const studentActionButtons = document.getElementById('student-action-buttons');
        const checkInBtn = document.getElementById('check-in-btn');
        const checkOutBtn = document.getElementById('check-out-btn');
        const cancelScanBtn = document.getElementById('cancel-scan-btn');
        const scanInstruction = document.getElementById('scan-instruction');


        // Modal elements
        const studentIdModal = document.getElementById('student-id-modal');
        const closeModalBtn = document.getElementById('close-modal-btn');
        const submitAttendanceBtn = document.getElementById('submit-attendance-btn');
        const modalStudentIdInput = document.getElementById('modal-student-id-input');
        const notificationBar = document.getElementById('notification-bar');
        const classSettingsModal = document.getElementById('class-settings-modal');
        const closeClassModalBtn = document.getElementById('close-class-modal-btn');
        const saveClassSettingsBtn = document.getElementById('save-class-settings-btn');
        const studentQrModal = document.getElementById('student-qr-modal');
        const closeStudentQrModalBtn = document.getElementById('close-student-qr-modal-btn');
        const modalQrStudentName = document.getElementById('modal-qr-student-name');
        const studentQrcodeDisplay = document.getElementById('student-qrcode-display');


        // --- FUNCTIONS ---
        
        const showNotification = (message, type = 'success') => {
            notificationBar.textContent = message;
            notificationBar.className = 'fixed top-5 right-5 p-4 rounded-lg shadow-lg z-[101] text-white'; // Reset classes
            if (type === 'success') {
                notificationBar.classList.add('bg-green-600');
            } else { // error
                notificationBar.classList.add('bg-red-500');
            }
            notificationBar.classList.remove('hidden');
            setTimeout(() => {
                notificationBar.classList.add('hidden');
            }, 3000);
        };

        const showLoading = (show) => {
            loadingSpinner.style.display = show ? 'flex' : 'none';
        };

        const switchView = (view) => {
            adminView.classList.add('hidden');
            studentView.classList.add('hidden');
            stopScanner();
            if (view === 'admin') {
                adminView.classList.remove('hidden');
                showAdminBtn.classList.replace('bg-blue-600', 'bg-blue-800');
                showStudentBtn.classList.replace('bg-green-800', 'bg-green-600');
            } else {
                studentView.classList.remove('hidden');
                resetStudentView();
                showStudentBtn.classList.replace('bg-green-600', 'bg-green-800');
                showAdminBtn.classList.replace('bg-blue-800', 'bg-blue-600');
            }
        };

        const handleAdminLogin = async () => {
            if (adminPasswordInput.value === ADMIN_PASSWORD) {
                adminLoginSection.classList.add('hidden');
                adminDashboard.classList.remove('hidden');
                loadGeneralSettings();
                listenForClasses();
                listenForStudents();
            } else {
                loginError.textContent = 'Kata sandi salah!';
            }
        };

        const loadGeneralSettings = async () => {
            try {
                const docRef = doc(settingsCollection, 'general');
                const docSnap = await getDoc(docRef);
                if (docSnap.exists()) {
                    generalSettings = docSnap.data();
                }
                generalStartTimeInput.value = generalSettings.startTime;
                generalEndTimeInput.value = generalSettings.endTime;
            } catch (error) {
                console.error("Error loading general settings:", error);
            }
        };

        const saveGeneralSettings = async () => {
            const settings = {
                startTime: generalStartTimeInput.value,
                endTime: generalEndTimeInput.value
            };
            showLoading(true);
            try {
                const docRef = doc(settingsCollection, 'general');
                await setDoc(docRef, settings);
                generalSettings = settings; // Update local state
                showNotification('Pengaturan waktu umum berhasil disimpan.');
            } catch (error) {
                console.error("Error saving general settings: ", error);
                showNotification('Gagal menyimpan pengaturan.', 'error');
            }
            showLoading(false);
        };

        const addStudent = async () => {
            const name = studentNameInput.value.trim();
            const id = studentIdInput.value.trim();
            const studentClass = studentClassSelector.value;
            if (!name || !id || !studentClass) {
                showNotification('Semua kolom harus diisi!', 'error');
                return;
            }
            showLoading(true);
            try {
                const studentDocRef = doc(db, `/artifacts/${appId}/public/data/students`, id);
                await setDoc(studentDocRef, { name, studentId: id, class: studentClass });
                studentNameInput.value = '';
                studentIdInput.value = '';
                studentClassSelector.value = '';
                showNotification('Siswa berhasil ditambahkan!');
            } catch (error) {
                console.error("Error adding student: ", error);
                showNotification('Gagal menambahkan siswa. Silakan coba lagi.', 'error');
            }
            showLoading(false);
        };

        const addStudentsInBulk = async () => {
            const selectedClass = bulkStudentClassSelector.value;
            const bulkInput = bulkStudentInput.value.trim();
            if (!selectedClass) {
                showNotification('Pilih kelas terlebih dahulu untuk tambah massal!', 'error');
                return;
            }
            if (!bulkInput) return;
            const lines = bulkInput.split('\n').filter(line => line.trim() !== '');
            if (lines.length === 0) return;

            showLoading(true);
            try {
                const batch = writeBatch(db);
                let count = 0;
                for (const line of lines) {
                    const [id, name] = line.split(',').map(s => s.trim());
                    if (id && name) {
                        const studentDocRef = doc(db, `/artifacts/${appId}/public/data/students`, id);
                        batch.set(studentDocRef, { studentId: id, name, class: selectedClass });
                        count++;
                    }
                }
                await batch.commit();
                showNotification(`${count} siswa berhasil ditambahkan ke kelas ${selectedClass}!`);
                bulkStudentInput.value = '';
                bulkStudentClassSelector.value = '';
            } catch (error) {
                console.error("Error adding students in bulk: ", error);
                showNotification('Gagal menambahkan siswa massal.', 'error');
            }
            showLoading(false);
        };
        
        const deleteStudent = async (studentId) => {
            showLoading(true);
            try {
                await deleteDoc(doc(db, `/artifacts/${appId}/public/data/students`, studentId));
                showNotification('Siswa berhasil dihapus.');
            } catch (error) {
                console.error("Error deleting student: ", error);
                showNotification('Gagal menghapus siswa.', 'error');
            }
            showLoading(false);
        };
        
        const listenForClasses = () => {
             onSnapshot(classesCollection, (snapshot) => {
                const classes = [];
                snapshot.forEach(doc => {
                    classes.push({ id: doc.id, ...doc.data() });
                });
                allClassSettings = classes;
                renderClassList(classes);
                populateClassSelectors(classes);
            });
        }

        const addClass = async () => {
            const className = classNameInput.value.trim();
            if (!className) return;
            if (allClassSettings.some(c => c.className.toLowerCase() === className.toLowerCase())) {
                showNotification('Nama kelas sudah ada!', 'error');
                return;
            }
            try {
                await addDoc(classesCollection, {
                    className: className,
                    startTime: generalSettings.startTime,
                    endTime: generalSettings.endTime
                });
                classNameInput.value = '';
                showNotification(`Kelas ${className} berhasil ditambahkan.`);
            } catch (error) {
                console.error("Error adding class:", error);
                showNotification('Gagal menambahkan kelas.', 'error');
            }
        };
        
        const deleteClass = async (classId) => {
             showLoading(true);
            try {
                await deleteDoc(doc(db, `/artifacts/${appId}/public/data/classes`, classId));
                showNotification('Kelas berhasil dihapus.');
            } catch (error) {
                console.error("Error deleting class: ", error);
                showNotification('Gagal menghapus kelas.', 'error');
            }
            showLoading(false);
        };

        const renderClassList = (classes) => {
            classListDiv.innerHTML = '';
            classes.sort((a,b) => a.className.localeCompare(b.className)).forEach(c => {
                const classEl = document.createElement('div');
                classEl.className = 'flex justify-between items-center bg-gray-50 p-2 rounded-lg';
                classEl.innerHTML = `
                    <span class="font-medium">${c.className}</span>
                    <div>
                        <button class="text-sm text-indigo-600 hover:text-indigo-800 font-semibold mr-2" data-id="${c.id}">Atur</button>
                        <button class="text-sm text-red-500 hover:text-red-700 font-semibold" data-id="${c.id}">Hapus</button>
                    </div>
                `;
                classListDiv.appendChild(classEl);
            });

            classListDiv.querySelectorAll('button').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    const classId = e.target.dataset.id;
                    if (e.target.textContent === 'Hapus') {
                        deleteClass(classId);
                    } else if (e.target.textContent === 'Atur') {
                        openClassSettingsModal(classId);
                    }
                })
            })
        };

        const openClassSettingsModal = (classId) => {
            const classData = allClassSettings.find(c => c.id === classId);
            if (!classData) return;
            document.getElementById('modal-class-name').textContent = classData.className;
            document.getElementById('modal-class-id').value = classId;
            document.getElementById('modal-start-time').value = classData.startTime;
            document.getElementById('modal-end-time').value = classData.endTime;
            classSettingsModal.style.display = 'flex';
        };

        const saveClassSettings = async () => {
            const classId = document.getElementById('modal-class-id').value;
            const newSettings = {
                startTime: document.getElementById('modal-start-time').value,
                endTime: document.getElementById('modal-end-time').value,
            };
            showLoading(true);
            try {
                const classDocRef = doc(db, `/artifacts/${appId}/public/data/classes`, classId);
                await updateDoc(classDocRef, newSettings);
                showNotification("Pengaturan kelas berhasil disimpan!");
                classSettingsModal.style.display = 'none';
            } catch (error) {
                console.error("Error saving class settings: ", error);
                showNotification("Gagal menyimpan pengaturan.", "error");
            }
            showLoading(false);
        }

        const listenForStudents = () => {
            onSnapshot(studentsCollection, (snapshot) => {
                const students = [];
                snapshot.forEach(doc => students.push({ id: doc.id, ...doc.data() }));
                allStudents = students;
                renderStudentList(students);
                renderFullDayReport();
                updateDashboardStats();
            });
        };

        const renderStudentList = (students) => {
            studentListTable.innerHTML = '';
            if (students.length === 0) {
                 studentListTable.innerHTML = '<tr><td colspan="4" class="text-center p-4 text-gray-500">Belum ada data siswa.</td></tr>';
                 return;
            }
            students.sort((a, b) => a.name.localeCompare(b.name)).forEach(student => {
                const row = document.createElement('tr');
                row.className = 'border-b';
                row.innerHTML = `
                    <td class="p-3">${student.name}</td>
                    <td class="p-3">${student.studentId}</td>
                    <td class="p-3">${student.class}</td>
                    <td class="p-3">
                        <button class="text-blue-500 hover:text-blue-700 font-semibold text-sm mr-2" data-type="generate-qr" data-student-id="${student.studentId}" data-student-name="${student.name}" data-student-class="${student.class}">Buat QR</button>
                        <button class="text-red-500 hover:text-red-700 font-semibold text-sm" data-type="delete" data-id="${student.studentId}">Hapus</button>
                    </td>
                `;
                studentListTable.appendChild(row);
            });
        };

        const generateStudentQRCode = (student) => {
            const data = {
                studentId: student.id,
                secret: QR_SECRET
            };
            studentQrcodeDisplay.innerHTML = ''; 
            new QRCode(studentQrcodeDisplay, { text: JSON.stringify(data), width: 250, height: 250 });
            modalQrStudentName.textContent = student.name;
            studentQrModal.style.display = 'flex';
        };

        const populateClassSelectors = (classes) => {
            const options = '<option value="">Pilih Kelas</option>' + classes.map(c => `<option value="${c.className}">${c.className}</option>`).join('');
            studentClassSelector.innerHTML = options;
            bulkStudentClassSelector.innerHTML = '<option value="">Pilih Kelas Untuk Siswa Massal</option>' + classes.map(c => `<option value="${c.className}">${c.className}</option>`).join('');
            classSelectorQr.innerHTML = options;
            classSelectorReport.innerHTML = '<option value="">Pilih Kelas</option>' + classes.map(c => `<option value="${c.className}">${c.className}</option>`).join('');
        };

        const generateClassQRCode = () => {
            const selectedClass = classSelectorQr.value;
            if (!selectedClass) {
                showNotification('Pilih kelas terlebih dahulu!', 'error');
                return;
            }
            const data = {
                class: selectedClass,
                secret: QR_SECRET,
                date: new Date().toISOString().split('T')[0]
            };
            qrcodeDiv.innerHTML = ''; 
            new QRCode(qrcodeDiv, { text: JSON.stringify(data), width: 200, height: 200 });
            qrcodeContainer.classList.remove('hidden');
            qrMessage.textContent = `QR Code untuk absensi kelas ${selectedClass} hari ini.`;
        };
        
        const renderFullDayReport = async () => {
            const reportDate = reportDateInput.value;
            const selectedClass = classSelectorReport.value;
            
            showLoading(true);
            const studentsToDisplay = selectedClass ? allStudents.filter(s => s.class === selectedClass) : allStudents;
            
            const q = query(attendanceCollection, where('date', '==', reportDate));
            const attendanceSnapshot = await getDocs(q);
            const attendanceRecordsMap = new Map();
            attendanceSnapshot.forEach(doc => {
                const data = doc.data();
                attendanceRecordsMap.set(data.studentId, data);
            });

            attendanceReportTable.innerHTML = '';
            if (studentsToDisplay.length === 0 && selectedClass) {
                 attendanceReportTable.innerHTML = '<tr><td colspan="5" class="text-center p-4 text-gray-500">Tidak ada siswa di kelas ini.</td></tr>';
                 showLoading(false);
                 return;
            }

            studentsToDisplay.sort((a,b) => a.name.localeCompare(b.name)).forEach(student => {
                const record = attendanceRecordsMap.get(student.studentId);
                const docId = `${reportDate}_${student.studentId}`;
                const checkIn = record?.checkInTime ? new Date(record.checkInTime).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }) : '-';
                const checkOut = record?.checkOutTime ? new Date(record.checkOutTime).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }) : '-';
                const currentStatus = record?.presenceStatus || 'Alpa';
                const notes = record?.notes || '-';
                
                const statusOptions = ['Hadir', 'Izin', 'Sakit', 'Alpa'];
                const statusSelect = `
                    <select class="status-select bg-gray-100 border border-gray-300 text-sm rounded-lg p-1 w-full" 
                            data-doc-id="${docId}" 
                            data-student-id="${student.studentId}" 
                            data-student-name="${student.name}" 
                            data-student-class="${student.class}">
                        ${statusOptions.map(s => `<option value="${s}" ${currentStatus === s ? 'selected' : ''}>${s}</option>`).join('')}
                    </select>
                `;

                const row = document.createElement('tr');
                row.className = 'border-b';
                row.innerHTML = `
                    <td class="p-3">${student.name}</td>
                    <td class="p-3">${checkIn}</td>
                    <td class="p-3">${checkOut}</td>
                    <td class="p-3">${statusSelect}</td>
                    <td class="p-3">${notes}</td>
                `;
                attendanceReportTable.appendChild(row);
            });

            updateDashboardStats(attendanceRecordsMap);
            showLoading(false);
        };
        
        const exportReportToCSV = async () => {
            const reportDate = reportDateInput.value;
            const selectedClass = classSelectorReport.value;
            
            if (!selectedClass) {
                showNotification('Pilih kelas terlebih dahulu untuk mengeksport laporan.', 'error');
                return;
            }

            showLoading(true);

            const studentsInClass = allStudents.filter(s => s.class === selectedClass);
            
            const q = query(attendanceCollection, where('date', '==', reportDate), where('class', '==', selectedClass));
            const attendanceSnapshot = await getDocs(q);
            const attendanceRecordsMap = new Map();
            attendanceSnapshot.forEach(doc => {
                const data = doc.data();
                attendanceRecordsMap.set(data.studentId, data);
            });

            let csvContent = "data:text/csv;charset=utf-8,";
            const headers = ["NIS", "Nama", "Kelas", "Datang", "Pulang", "Status", "Keterangan"];
            csvContent += headers.join(",") + "\r\n";

            studentsInClass.sort((a,b) => a.name.localeCompare(b.name)).forEach(student => {
                const record = attendanceRecordsMap.get(student.studentId);
                const nis = `"${student.studentId}"`;
                const name = `"${student.name}"`;
                const studentClass = `"${student.class}"`;
                const checkIn = record?.checkInTime ? `"${new Date(record.checkInTime).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })}"` : '""';
                const checkOut = record?.checkOutTime ? `"${new Date(record.checkOutTime).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })}"` : '""';
                const status = `"${record?.presenceStatus || 'Alpa'}"`;
                const notes = `"${record?.notes || ''}"`;
                
                const row = [nis, name, studentClass, checkIn, checkOut, status, notes].join(",");
                csvContent += row + "\r\n";
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            const fileName = `laporan_kehadiran_${selectedClass.replace(/\s+/g, '_')}_${reportDate}.csv`;
            link.setAttribute("download", fileName);
            document.body.appendChild(link);

            link.click();
            document.body.removeChild(link);

            showLoading(false);
            showNotification('Laporan berhasil dieksport!');
        };
        
        attendanceReportTable.addEventListener('change', async (e) => {
            if (e.target.classList.contains('status-select')) {
                const select = e.target;
                const { docId, studentId, studentName, studentClass } = select.dataset;
                const newStatus = select.value;
                const reportDate = reportDateInput.value;

                showLoading(true);
                const attendanceDocRef = doc(db, `/artifacts/${appId}/public/data/attendance`, docId);
                try {
                    await setDoc(attendanceDocRef, {
                        presenceStatus: newStatus,
                        studentId, studentName, class: studentClass, date: reportDate,
                    }, { merge: true });
                    showNotification('Status berhasil diperbarui.');
                    renderFullDayReport();
                } catch(error) {
                    console.error("Error updating status:", error);
                    showNotification('Gagal memperbarui status.', 'error');
                }
                showLoading(false);
            }
        });


        const updateDashboardStats = (attendanceRecordsMap = new Map()) => {
            const totalStudents = allStudents.length;
            const presentTodayCount = Array.from(attendanceRecordsMap.values()).filter(r => r.presenceStatus === 'Hadir').length;
            
            let lateCount = 0;
            let earlyLeaveCount = 0;
            attendanceRecordsMap.forEach(record => {
                if (record.notes?.includes('Terlambat')) lateCount++;
                if (record.notes?.includes('Pulang Cepat')) earlyLeaveCount++;
            });
            
            const attendancePercentage = totalStudents > 0 ? ((presentTodayCount / totalStudents) * 100).toFixed(0) : 0;

            document.getElementById('total-students-stat').textContent = totalStudents;
            document.getElementById('attendance-percentage-stat').textContent = `${attendancePercentage}%`;
            document.getElementById('late-students-stat').textContent = lateCount;
            earlyLeaveStat.textContent = earlyLeaveCount;
        };

        // --- Student View Functions ---
        const prepareScanner = (mode) => {
            scanMode = mode;
            studentActionButtons.classList.add('hidden');
            qrReaderContainer.classList.remove('hidden');
            scanInstruction.textContent = `Pindai QR Code untuk ${mode === 'checkIn' ? 'Absen Datang' : 'Absen Pulang'}`;
            startScanner();
        };
        
        const startScanner = () => {
            if (!html5QrCode) html5QrCode = new Html5Qrcode("qr-reader");
            if (html5QrCode.isScanning) return;
            html5QrCode.start(
                { facingMode: "environment" }, { fps: 10, qrbox: { width: 250, height: 250 }},
                onScanSuccess, (error) => {}
            ).catch(err => {
                qrReaderResults.textContent = "Gagal memulai kamera.";
                qrReaderResults.className = 'error';
                resetStudentView();
            });
        };

        const stopScanner = async () => {
            if (html5QrCode && html5QrCode.isScanning) {
                try { await html5QrCode.stop(); } catch(err) { console.error("Scanner stop failed", err); }
            }
        };
        
        const resetStudentView = () => {
            scanMode = null;
            qrReaderContainer.classList.add('hidden');
            studentActionButtons.classList.remove('hidden');
            qrReaderResults.textContent = '';
            qrReaderResults.className = '';
        };

        const onScanSuccess = async (decodedText, decodedResult) => {
            await stopScanner();
            try {
                const data = JSON.parse(decodedText);
                if (data.secret !== QR_SECRET) throw new Error('QR Code tidak valid.');
                
                if (data.studentId) { // Student QR
                    qrReaderResults.textContent = `QR Code terdeteksi. Memproses...`;
                    qrReaderResults.className = 'info';
                    submitAttendance(data.studentId);
                } 
                else if (data.class && data.date) { // Class QR
                    const today = new Date().toISOString().split('T')[0];
                    if (data.date !== today) throw new Error('QR Code ini bukan untuk absensi hari ini.');
                    qrCodeData = data;
                    modalStudentIdInput.value = '';
                    studentIdModal.style.display = 'flex';
                } 
                else {
                    throw new Error('Format QR Code tidak dikenali.');
                }
            } catch (error) {
                qrReaderResults.textContent = error.message || 'QR Code tidak dapat dibaca.';
                qrReaderResults.className = 'error';
                setTimeout(resetStudentView, 3000);
            }
        };
        
        const submitAttendance = async (studentIdFromQr = null) => {
            const studentId = studentIdFromQr || modalStudentIdInput.value.trim();
            if (!studentId) {
                showNotification('ID Siswa harus diisi.', 'error');
                return;
            }
            if (!scanMode) {
                 showNotification('Mode absensi tidak valid.', 'error');
                 resetStudentView();
                 return;
            }
            
            showLoading(true);
            studentIdModal.style.display = 'none';
            
            const today = new Date().toISOString().split('T')[0];
            const attendanceDocId = `${today}_${studentId}`;
            const attendanceDocRef = doc(db, `/artifacts/${appId}/public/data/attendance`, attendanceDocId);

            try {
                const studentDoc = await getDoc(doc(studentsCollection, studentId));
                if (!studentDoc.exists()) throw new Error(`Siswa dengan ID ${studentId} tidak ditemukan.`);
                const studentData = studentDoc.data();

                const classSetting = allClassSettings.find(c => c.className === studentData.class);
                if (!classSetting) throw new Error('Pengaturan untuk kelas ini tidak ditemukan.');

                const attendanceDoc = await getDoc(attendanceDocRef);
                const currentTime = Date.now();
                let notes = attendanceDoc.exists() && attendanceDoc.data().notes ? attendanceDoc.data().notes.split(', ').filter(Boolean) : [];

                if (scanMode === 'checkIn') {
                    if (attendanceDoc.exists() && attendanceDoc.data().checkInTime) throw new Error('Anda sudah absen datang hari ini.');
                    
                    const [hours, minutes] = classSetting.startTime.split(':');
                    const deadline = new Date();
                    deadline.setHours(hours, minutes, 0, 0);
                    if (currentTime > deadline.getTime()) notes.push('Terlambat');

                    await setDoc(attendanceDocRef, {
                        studentId, studentName: studentData.name, class: studentData.class, date: today,
                        checkInTime: currentTime,
                        presenceStatus: 'Hadir',
                        notes: notes.join(', ')
                    }, { merge: true });
                    qrReaderResults.textContent = `Absen datang berhasil untuk ${studentData.name}!`;

                } else if (scanMode === 'checkOut') {
                    if (!attendanceDoc.exists() || !attendanceDoc.data().checkInTime) throw new Error('Anda harus absen datang terlebih dahulu.');
                    if (attendanceDoc.data().checkOutTime) throw new Error('Anda sudah absen pulang hari ini.');

                    const [endHours, endMinutes] = classSetting.endTime.split(':');
                    const minDepartureTime = new Date();
                    minDepartureTime.setHours(endHours, endMinutes, 0, 0);
                    if (currentTime < minDepartureTime.getTime()) {
                        if(!notes.includes('Pulang Cepat')) notes.push('Pulang Cepat');
                    }

                    await updateDoc(attendanceDocRef, { checkOutTime: currentTime, notes: notes.join(', ') });
                    qrReaderResults.textContent = `Absen pulang berhasil untuk ${studentData.name}!`;
                }
                
                qrReaderResults.className = 'success';
                
            } catch (error) {
                 qrReaderResults.textContent = error.message;
                 qrReaderResults.className = 'error';
            } finally {
                showLoading(false);
                setTimeout(resetStudentView, 3000);
            }
        };

        // --- INITIALIZATION & EVENT LISTENERS ---
        onAuthStateChanged(auth, (user) => { currentUser = user; });

        const authenticate = async () => {
            showLoading(true);
            try {
                if (initialAuthToken) await signInWithCustomToken(auth, initialAuthToken);
                else await signInAnonymously(auth);
            } catch (error) {
                console.error("Authentication failed:", error);
            }
            showLoading(false);
        };
        
        showAdminBtn.addEventListener('click', () => switchView('admin'));
        showStudentBtn.addEventListener('click', () => switchView('student'));
        adminLoginBtn.addEventListener('click', handleAdminLogin);
        saveGeneralSettingsBtn.addEventListener('click', saveGeneralSettings);
        adminPasswordInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') handleAdminLogin(); });
        addStudentBtn.addEventListener('click', addStudent);
        bulkAddStudentBtn.addEventListener('click', addStudentsInBulk);
        addClassBtn.addEventListener('click', addClass);
        generateQrBtn.addEventListener('click', generateClassQRCode);
        reportDateInput.addEventListener('change', renderFullDayReport);
        classSelectorReport.addEventListener('change', renderFullDayReport);
        exportReportBtn.addEventListener('click', exportReportToCSV);
        
        studentListTable.addEventListener('click', (e) => {
            const target = e.target;
            if (target.tagName === 'BUTTON') {
                const type = target.dataset.type;
                if (type === 'delete') {
                    deleteStudent(target.dataset.id);
                } else if (type === 'generate-qr') {
                    generateStudentQRCode({ id: target.dataset.studentId, name: target.dataset.studentName, class: target.dataset.studentClass });
                }
            }
        });

        showStudentsTab.addEventListener('click', () => {
            studentListContent.classList.remove('hidden');
            reportsContent.classList.add('hidden');
            showStudentsTab.classList.add('bg-blue-100', 'text-blue-800', 'font-semibold');
            showStudentsTab.classList.remove('bg-gray-100', 'text-gray-800');
            showReportsTab.classList.add('bg-gray-100', 'text-gray-800');
            showReportsTab.classList.remove('bg-blue-100', 'text-blue-800', 'font-semibold');
        });
        showReportsTab.addEventListener('click', () => {
            reportsContent.classList.remove('hidden');
            studentListContent.classList.add('hidden');
            showReportsTab.classList.add('bg-blue-100', 'text-blue-800', 'font-semibold');
            showReportsTab.classList.remove('bg-gray-100', 'text-gray-800');
            showStudentsTab.classList.add('bg-gray-100', 'text-gray-800');
            showStudentsTab.classList.remove('bg-blue-100', 'text-blue-800', 'font-semibold');
        });

        checkInBtn.addEventListener('click', () => prepareScanner('checkIn'));
        checkOutBtn.addEventListener('click', () => prepareScanner('checkOut'));
        cancelScanBtn.addEventListener('click', () => {
            stopScanner();
            resetStudentView();
        });

        closeModalBtn.addEventListener('click', () => { 
            studentIdModal.style.display = 'none'; 
            resetStudentView(); 
        });
        submitAttendanceBtn.addEventListener('click', () => submitAttendance());
        modalStudentIdInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') submitAttendance(); });

        closeClassModalBtn.addEventListener('click', () => { classSettingsModal.style.display = 'none'; });
        saveClassSettingsBtn.addEventListener('click', saveClassSettings);
        closeStudentQrModalBtn.addEventListener('click', () => { studentQrModal.style.display = 'none'; });
        
        window.onload = async () => {
            await authenticate();
            switchView('student');
            reportDateInput.value = new Date().toISOString().split('T')[0];
        };

    </script>
</body>
</html>

