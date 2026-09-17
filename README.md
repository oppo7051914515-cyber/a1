<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ระบบเช็กชื่อเข้าเรียนอัจฉริยะ (Smart Attendance System)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.31/jspdf.plugin.autotable.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Sarabun', sans-serif; -webkit-tap-highlight-color: transparent; }
        
        .crop-container {
            width: 140px;
            height: 186px;
            overflow: hidden;
            position: relative;
            border-radius: 12px;
            cursor: grab;
            user-select: none;
            touch-action: none;
            background-color: #f1f5f9;
        }
        .crop-container:active { cursor: grabbing; }
        .crop-img {
            position: absolute;
            top: 50%;
            left: 50%;
            transform-origin: center center;
            pointer-events: none;
            max-width: none;
        }
        
        .tab-content { transition: opacity 0.2s ease-in-out; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 min-h-screen flex flex-col md:flex-row pb-16 md:pb-0">

    <!-- Top Header (แสดงเฉพาะครู บน Mobile/Tablet) -->
    <header id="mobileHeader" class="md:hidden bg-indigo-900 text-white p-4 sticky top-0 z-30 shadow-md flex justify-between items-center">
        <div class="flex items-center gap-2">
            <div class="p-1.5 bg-indigo-600 rounded-lg">
                <i data-lucide="scan-face" class="w-6 h-6"></i>
            </div>
            <div>
                <h1 class="font-bold text-sm leading-tight">เช็กชื่อเข้าเรียน</h1>
                <span id="mobileCurrentClassText" class="text-[11px] text-indigo-300">วิชา: CS101</span>
            </div>
        </div>
        <button onclick="toggleMobileClassroomDrawer()" class="bg-indigo-800 hover:bg-indigo-700 px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1 border border-indigo-700">
            <i data-lucide="folder" class="w-3.5 h-3.5"></i> เลือกวิชา
        </button>
    </header>

    <!-- Sidebar Menu (แสดงเฉพาะครู บน Laptop/Desktop) -->
    <aside id="desktopSidebar" class="hidden md:flex w-72 bg-indigo-900 text-white p-5 flex-col justify-between shadow-xl flex-shrink-0">
        <div>
            <div class="flex items-center gap-3 mb-6">
                <div class="p-2.5 bg-indigo-600 rounded-xl shadow-lg">
                    <i data-lucide="scan-face" class="w-7 h-7"></i>
                </div>
                <div>
                    <h1 class="font-bold text-base leading-tight">ระบบเช็กชื่อใบหน้า</h1>
                    <span class="text-xs text-indigo-300">Smart Attendance v6.1</span>
                </div>
            </div>

            <!-- กล่องเลือกห้องเรียน -->
            <div class="mb-6 bg-indigo-950/70 p-3.5 rounded-2xl border border-indigo-700/50 space-y-2.5">
                <label class="block text-xs text-indigo-300 font-semibold flex items-center gap-1">
                    <i data-lucide="book-open" class="w-3.5 h-3.5"></i> รายวิชา / ห้องเรียน
                </label>
                <select id="classroomSelect" onchange="changeClassroom()" class="w-full bg-indigo-800 text-white p-2.5 rounded-xl text-sm font-medium focus:outline-none focus:ring-2 focus:ring-indigo-400 border border-indigo-600">
                    <!-- JS Render Options -->
                </select>

                <div class="grid grid-cols-2 gap-1.5 pt-1">
                    <button onclick="openClassroomModal(false)" class="text-xs bg-indigo-600 hover:bg-indigo-500 text-white py-2 px-2 rounded-lg font-medium flex items-center justify-center gap-1 transition">
                        <i data-lucide="plus" class="w-3.5 h-3.5"></i> เพิ่มห้อง
                    </button>
                    <button onclick="openClassroomModal(true)" class="text-xs bg-amber-600 hover:bg-amber-500 text-white py-2 px-2 rounded-lg font-medium flex items-center justify-center gap-1 transition">
                        <i data-lucide="pencil" class="w-3.5 h-3.5"></i> แก้ไขวิชา
                    </button>
                </div>
                <button onclick="deleteCurrentClassroom()" class="w-full text-xs bg-rose-900/60 hover:bg-rose-800 text-rose-200 py-1.5 rounded-lg flex items-center justify-center gap-1 transition">
                    <i data-lucide="trash-2" class="w-3.5 h-3.5"></i> ลบห้องเรียนนี้
                </button>
            </div>

            <!-- เมนูนำทาง -->
            <nav class="space-y-1.5">
                <button onclick="switchTab('session-control')" id="nav-session-control" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium bg-indigo-800 text-white shadow-md transition">
                    <i data-lucide="play-circle" class="w-4 h-4"></i> ควบคุมการเช็กชื่อ (ครู)
                </button>
                <button onclick="switchTab('scan-student')" id="nav-scan-student" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60 transition">
                    <i data-lucide="smartphone" class="w-4 h-4"></i> นักเรียนสแกนเช็กชื่อ
                </button>
                <button onclick="switchTab('edit-time')" id="nav-edit-time" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60 transition">
                    <i data-lucide="clock" class="w-4 h-4"></i> ตารางเวลา / แก้ไข
                </button>
                <button onclick="switchTab('students')" id="nav-students" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60 transition">
                    <i data-lucide="users" class="w-4 h-4"></i> จัดการนักเรียน (CRUD)
                </button>
                <button onclick="switchTab('dashboard')" id="nav-dashboard" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60 transition">
                    <i data-lucide="bar-chart-3" class="w-4 h-4"></i> สถิติ & Export รายงาน
                </button>
            </nav>
        </div>

        <div class="mt-6 pt-4 border-t border-indigo-800/60 text-xs text-indigo-300 text-center font-light">
            ระบบเช็กชื่อเข้าเรียนอัจฉริยะ
        </div>
    </aside>

    <!-- Content Main Area -->
    <main class="flex-1 p-4 md:p-8 overflow-y-auto max-w-7xl mx-auto w-full">

        <!-- Page Header -->
        <div id="mainPageHeader" class="flex flex-col md:flex-row justify-between items-start md:items-center mb-6 pb-4 border-b border-slate-200 gap-3">
            <div>
                <h2 id="pageTitle" class="text-xl md:text-2xl font-bold text-slate-800">⏱️ ควบคุมเปิด/ปิดการเช็กชื่อ (สำหรับครู)</h2>
                <div class="flex flex-wrap items-center gap-2 mt-1 text-xs md:text-sm text-slate-500">
                    <span class="font-bold text-indigo-600 bg-indigo-50 px-2.5 py-0.5 rounded-md border border-indigo-100" id="currentClassText">CS101</span>
                    <span id="headerSubjectText"></span>
                    <span id="headerTeacherText"></span>
                </div>
            </div>
            <div class="flex items-center gap-2 bg-white shadow-sm border border-slate-200 text-indigo-700 px-3.5 py-1.5 rounded-xl text-xs font-semibold">
                <i data-lucide="calendar" class="w-4 h-4 text-indigo-600"></i>
                <span id="liveDateText"></span>
            </div>
        </div>

        <!-- 1. เมนู: ควบคุมเปิด/ปิดการเช็กชื่อ -->
        <section id="tab-session-control" class="tab-content block space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-2 bg-white p-5 md:p-6 rounded-2xl shadow-sm border border-slate-200/80">
                    <h3 class="font-bold text-slate-800 text-base md:text-lg mb-4 flex items-center gap-2">
                        <i data-lucide="sliders" class="w-5 h-5 text-indigo-600"></i> ตั้งค่าคาบเรียนและเปิดสิทธิ์รับเช็กชื่อ (AM/PM)
                    </h3>

                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-6">
                        <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200">
                            <label class="block text-xs font-semibold text-slate-600 mb-1.5">เวลาสิ้นสุด "มาตรงเวลา"</label>
                            <div class="flex gap-2">
                                <input type="time" id="onTimeLimitInput" value="08:30" class="flex-1 bg-white border border-slate-300 rounded-lg p-2 text-sm focus:ring-2 focus:ring-indigo-500 outline-none font-medium">
                                <select id="onTimeAmpm" class="border border-slate-300 rounded-lg px-2.5 bg-white text-sm font-bold text-slate-700">
                                    <option value="AM" selected>AM</option>
                                    <option value="PM">PM</option>
                                </select>
                            </div>
                        </div>
                        <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200">
                            <label class="block text-xs font-semibold text-slate-600 mb-1.5">เวลาสิ้นสุด "เข้าเรียนสาย"</label>
                            <div class="flex gap-2">
                                <input type="time" id="lateLimitInput" value="09:00" class="flex-1 bg-white border border-slate-300 rounded-lg p-2 text-sm focus:ring-2 focus:ring-indigo-500 outline-none font-medium">
                                <select id="lateAmpm" class="border border-slate-300 rounded-lg px-2.5 bg-white text-sm font-bold text-slate-700">
                                    <option value="AM" selected>AM</option>
                                    <option value="PM">PM</option>
                                </select>
                            </div>
                        </div>
                    </div>

                    <div class="p-4 bg-slate-50 rounded-xl border border-slate-200 mb-6 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
                        <div>
                            <span class="text-xs text-slate-500 block font-medium">สถานะระบบเช็กชื่อขณะนี้:</span>
                            <span id="sessionStatusBadge" class="text-xs md:text-sm font-bold inline-flex items-center gap-1.5 mt-1 px-3 py-1 rounded-full bg-slate-200 text-slate-600">
                                <span class="w-2.5 h-2.5 rounded-full bg-slate-400"></span> ปิดรับการเช็กชื่อ (Closed)
                            </span>
                        </div>
                        <div id="sessionTimeStarted" class="text-xs text-slate-500 font-medium">
                            เวลาเริ่มเช็กชื่อ: -
                        </div>
                    </div>

                    <div class="flex flex-col sm:flex-row gap-3">
                        <button onclick="toggleAttendanceSession(true)" id="btnStartSession" class="flex-1 bg-emerald-600 hover:bg-emerald-700 text-white py-3 rounded-xl font-bold text-sm flex items-center justify-center gap-2 shadow-lg transition active:scale-95">
                            <i data-lucide="play" class="w-5 h-5"></i> เริ่มต้นเช็กชื่อ (Start)
                        </button>
                        <button onclick="toggleAttendanceSession(false)" id="btnStopSession" class="flex-1 bg-rose-600 hover:bg-rose-700 text-white py-3 rounded-xl font-bold text-sm flex items-center justify-center gap-2 shadow-lg transition active:scale-95 opacity-50 cursor-not-allowed" disabled>
                            <i data-lucide="square" class="w-5 h-5"></i> ปิดรับเช็กชื่อ (Stop)
                        </button>
                    </div>
                </div>

                <div class="bg-white p-5 md:p-6 rounded-2xl shadow-sm border border-slate-200/80 flex flex-col justify-between">
                    <div>
                        <h3 class="font-bold text-slate-800 mb-4 pb-2 border-b flex items-center gap-2">
                            <i data-lucide="pie-chart" class="w-5 h-5 text-indigo-600"></i> สรุปผลคาบปัจจุบัน
                        </h3>

                        <div class="space-y-3">
                            <div class="flex justify-between items-center p-3 bg-slate-50 rounded-xl">
                                <span class="text-xs font-semibold text-slate-600">นักเรียนทั้งหมด</span>
                                <span id="sessionTotalStudents" class="font-bold text-slate-800">0 คน</span>
                            </div>
                            <div class="flex justify-between items-center p-3 bg-emerald-50 rounded-xl text-emerald-800">
                                <span class="text-xs font-semibold">เช็กชื่อเรียบร้อย</span>
                                <span id="sessionCheckedStudents" class="font-bold text-emerald-600">0 คน</span>
                            </div>
                            <div class="flex justify-between items-center p-3 bg-rose-50 rounded-xl text-rose-800">
                                <span class="text-xs font-semibold">ยังไม่ได้เช็กชื่อ</span>
                                <span id="sessionUncheckedStudents" class="font-bold text-rose-600">0 คน</span>
                            </div>
                        </div>
                    </div>

                    <div class="mt-6 p-3 bg-indigo-50/80 rounded-xl text-xs text-indigo-800 flex items-start gap-2">
                        <i data-lucide="info" class="w-4 h-4 flex-shrink-0 mt-0.5 text-indigo-600"></i>
                        <span>แชร์ลิงก์หรือแสดง QR Code ในเมนูสแกนเพื่อให้เด็กเริ่มเช็กชื่อ</span>
                    </div>
                </div>
            </div>
        </section>

        <!-- 2. เมนู: หน้าเช็กชื่อสำหรับนักเรียนเท่านั้น (Student Scan View) -->
        <section id="tab-scan-student" class="tab-content hidden space-y-6">
            <div id="shareControlBar" class="max-w-md mx-auto bg-indigo-900 text-white p-3.5 rounded-2xl flex items-center justify-between gap-2 shadow-md">
                <div class="flex items-center gap-2 font-semibold text-xs pl-1">
                    <i data-lucide="share-2" class="w-4 h-4 text-indigo-300"></i>
                    <span>แชร์ให้นักเรียนสแกน</span>
                </div>
                <div class="flex gap-1.5">
                    <button onclick="copyStudentLink()" class="bg-indigo-600 hover:bg-indigo-500 px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1 transition">
                        <i data-lucide="copy" class="w-3.5 h-3.5"></i> คัดลอกลิงก์
                    </button>
                    <button onclick="showQRCodeModal()" class="bg-amber-500 hover:bg-amber-400 text-slate-900 px-3 py-1.5 rounded-lg text-xs font-bold flex items-center gap-1 transition">
                        <i data-lucide="qr-code" class="w-3.5 h-3.5"></i> QR Code
                    </button>
                </div>
            </div>

            <!-- การ์ดฟอร์มสแกนสำหรับนักเรียน -->
            <div class="max-w-md mx-auto bg-white p-6 rounded-3xl shadow-xl border border-slate-200 my-auto">
                <div id="studentClosedAlert" class="bg-amber-50 border border-amber-200 p-5 rounded-2xl text-center mb-4">
                    <div class="w-12 h-12 bg-amber-100 text-amber-600 rounded-full flex items-center justify-center mx-auto mb-2">
                        <i data-lucide="lock" class="w-6 h-6"></i>
                    </div>
                    <h4 class="font-bold text-amber-800 text-sm">ยังไม่เปิดรับการเช็กชื่อในขณะนี้</h4>
                    <p class="text-xs text-amber-600 mt-1">กรุณารอครูผู้สอนกดเปิดระบบเช็กชื่อประจำคาบ</p>
                </div>

                <div id="studentScanFormArea" class="space-y-4 opacity-50 pointer-events-none">
                    <div class="text-center mb-2">
                        <span id="studentScanSubjectBadge" class="bg-indigo-100 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full inline-block mb-2">รายวิชา CS101</span>
                        <h3 class="text-xl font-bold text-slate-800">สแกนใบหน้าเข้าเรียน</h3>
                        <p class="text-xs text-slate-500 mt-0.5">เลือกรายชื่อของคุณ และเปิดกล้องถ่ายภาพหน้าตรง</p>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">1. เลือกชื่อ-รหัสนักเรียนของคุณ</label>
                        <select id="studentSelfSelect" class="w-full border border-slate-300 rounded-xl p-3 text-sm font-medium focus:ring-2 focus:ring-indigo-500 outline-none bg-slate-50">
                            <!-- JS Render List -->
                        </select>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">2. สแกนใบหน้ายืนยันตัวตน</label>
                        <div class="relative bg-slate-900 rounded-2xl aspect-square flex items-center justify-center overflow-hidden border-2 border-slate-200 shadow-inner">
                            <video id="videoStudent" class="w-full h-full object-cover hidden" autoplay playsinline></video>
                            <div id="studentCamPlaceholder" class="text-center p-6 text-slate-400">
                                <i data-lucide="camera" class="w-12 h-12 mx-auto mb-2 opacity-40"></i>
                                <p class="text-xs font-medium">กดปุ่ม "เปิดกล้อง" ด้านล่าง</p>
                            </div>
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-2.5 pt-2">
                        <button onclick="startStudentCamera()" class="bg-indigo-600 hover:bg-indigo-700 active:scale-95 text-white py-3 rounded-xl text-xs md:text-sm font-bold flex items-center justify-center gap-1.5 shadow-md transition">
                            <i data-lucide="camera" class="w-4 h-4"></i> เปิดกล้อง
                        </button>
                        <button onclick="confirmStudentSelfScan()" class="bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white py-3 rounded-xl text-xs md:text-sm font-bold flex items-center justify-center gap-1.5 shadow-md transition">
                            <i data-lucide="check-circle" class="w-4 h-4"></i> บันทึกเช็กชื่อ
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- 3. เมนู: กำหนด/แก้ไขเวลาเรียน -->
        <section id="tab-edit-time" class="tab-content hidden space-y-6">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="font-bold text-slate-800 text-base flex items-center gap-2">
                        <i data-lucide="edit-3" class="w-5 h-5 text-indigo-600"></i> รายการบันทึกเช็กชื่อ & แก้ไขย้อนหลัง
                    </h3>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left text-sm text-slate-600 border-collapse">
                        <thead class="bg-slate-100 text-slate-700 uppercase text-xs">
                            <tr>
                                <th class="p-3 rounded-l-xl">วันที่</th>
                                <th class="p-3">รหัส</th>
                                <th class="p-3">ชื่อ-นามสกุล</th>
                                <th class="p-3">เวลา</th>
                                <th class="p-3">สถานะ</th>
                                <th class="p-3 text-center rounded-r-xl">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="attendanceLogsTable" class="divide-y divide-slate-100">
                            <!-- JS Render Logs -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- 4. เมนู: จัดการข้อมูลนักเรียน (CRUD & 2-Inch Photo Cropper) -->
        <section id="tab-students" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- ฟอร์มเพิ่ม/แก้ไขนักเรียน -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                    <h3 id="studentFormTitle" class="font-bold text-slate-800 mb-4 flex items-center gap-2">
                        <i data-lucide="user-plus" class="w-5 h-5 text-indigo-600"></i> เพิ่มข้อมูลนักเรียนใหม่
                    </h3>
                    
                    <form id="studentForm" onsubmit="saveStudent(event)" class="space-y-3.5">
                        <input type="hidden" id="editStudentIdx" value="-1">
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสนักเรียน <span class="text-rose-500">*</span></label>
                            <input type="text" id="studentId" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น 6601001">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อ-นามสกุล <span class="text-rose-500">*</span></label>
                            <input type="text" id="studentName" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น นาย สมชาย ใจดี">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ชั้นเรียน / ห้อง</label>
                            <input type="text" id="studentClassGrade" class="w-full border border-slate-300 rounded-lg p-2 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น ม.4/1">
                        </div>
                        
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">อัปโหลดรูปถ่ายหน้าตรง (ปรับสัดส่วน 2 นิ้วอัตโนมัติ)</label>
                            <input type="file" id="studentPhoto" accept="image/*" onchange="loadStudentPhotoToCropper(event)" class="w-full text-xs text-slate-500 file:mr-3 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-xs file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100 cursor-pointer">
                            
                            <div id="photoCropperArea" class="mt-3 p-3 bg-slate-50 border border-slate-200 rounded-2xl hidden text-center space-y-2">
                                <span class="text-[11px] font-semibold text-indigo-600 block">✨ ปรับรูปขนาด 2 นิ้วอัตโนมัติ (ลากขยับตำแหน่งได้)</span>
                                
                                <div class="flex justify-center">
                                    <div id="cropContainer" class="crop-container border-2 border-indigo-500 shadow-md">
                                        <img id="cropImg" class="crop-img" src="" alt="Crop Area 2 นิ้ว">
                                    </div>
                                </div>

                                <div class="flex items-center justify-center gap-2 pt-1 max-w-[200px] mx-auto">
                                    <i data-lucide="zoom-out" class="w-4 h-4 text-slate-400"></i>
                                    <input type="range" id="zoomSlider" min="0.3" max="1.8" step="0.02" value="0.75" oninput="updateCropImageTransform()" class="w-full accent-indigo-600 cursor-pointer">
                                    <i data-lucide="zoom-in" class="w-4 h-4 text-slate-400"></i>
                                </div>
                            </div>
                        </div>

                        <div class="flex gap-2 pt-2">
                            <button type="submit" class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white py-2.5 rounded-xl text-sm font-semibold transition shadow">
                                บันทึกข้อมูล
                            </button>
                            <button type="button" onclick="resetStudentForm()" class="bg-slate-200 hover:bg-slate-300 text-slate-700 px-4 py-2.5 rounded-xl text-sm font-semibold">
                                ยกเลิก
                            </button>
                        </div>
                    </form>
                </div>

                <!-- ตารางรายชื่อนักเรียน -->
                <div class="lg:col-span-2 bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                    <h3 class="font-bold text-slate-800 mb-4 flex items-center gap-2">
                        <i data-lucide="users" class="w-5 h-5 text-indigo-600"></i> รายชื่อนักเรียนในห้องเรียนนี้
                    </h3>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm text-slate-600 border-collapse">
                            <thead class="bg-slate-100 text-slate-700 uppercase text-xs">
                                <tr>
                                    <th class="p-3 rounded-l-xl text-center">รูปถ่าย 2 นิ้ว</th>
                                    <th class="p-3">รหัส</th>
                                    <th class="p-3">ชื่อ-นามสกุล</th>
                                    <th class="p-3">ชั้นเรียน/ห้อง</th>
                                    <th class="p-3 text-center rounded-r-xl">จัดการ</th>
                                </tr>
                            </thead>
                            <tbody id="studentListTable" class="divide-y divide-slate-100">
                                <!-- JS Render Student List -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </section>

        <!-- 5. เมนู: Dashboard สถิติ -->
        <section id="tab-dashboard" class="tab-content hidden space-y-6">
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                <div class="flex items-center gap-2 text-slate-700 font-bold text-sm">
                    <i data-lucide="filter" class="w-4 h-4 text-indigo-600"></i>
                    <span>กรองช่วงเวลา:</span>
                </div>
                <div class="flex bg-slate-100 p-1 rounded-xl text-xs font-semibold gap-1 w-full sm:w-auto">
                    <button onclick="setDashboardTimeFilter('all')" id="filter-all" class="filter-btn flex-1 sm:flex-initial px-3.5 py-1.5 rounded-lg bg-white text-indigo-600 shadow">ทั้งหมด</button>
                    <button onclick="setDashboardTimeFilter('daily')" id="filter-daily" class="filter-btn flex-1 sm:flex-initial px-3.5 py-1.5 rounded-lg text-slate-600 hover:text-indigo-600">วันนี้</button>
                    <button onclick="setDashboardTimeFilter('weekly')" id="filter-weekly" class="filter-btn flex-1 sm:flex-initial px-3.5 py-1.5 rounded-lg text-slate-600 hover:text-indigo-600">สัปดาห์นี้</button>
                    <button onclick="setDashboardTimeFilter('monthly')" id="filter-monthly" class="filter-btn flex-1 sm:flex-initial px-3.5 py-1.5 rounded-lg text-slate-600 hover:text-indigo-600">เดือนนี้</button>
                </div>
            </div>

            <div id="aiAlertBox" class="bg-amber-50 border-l-4 border-amber-500 p-4 rounded-r-2xl shadow-sm flex items-start gap-3">
                <i data-lucide="alert-triangle" class="w-5 h-5 text-amber-600 flex-shrink-0 mt-0.5"></i>
                <div>
                    <h4 class="font-bold text-amber-800 text-sm">แจ้งเตือนสรุปสถิติการเข้าเรียน</h4>
                    <p id="aiAlertText" class="text-xs text-amber-700 mt-0.5">กำลังคำนวณสถิติการเข้าเรียน...</p>
                </div>
            </div>

            <div class="flex flex-wrap gap-3 justify-between items-center bg-white p-4 rounded-2xl border border-slate-200 shadow-sm">
                <div class="font-bold text-slate-700 text-sm flex items-center gap-2">
                    <i data-lucide="file-spreadsheet" class="w-5 h-5 text-indigo-600"></i> ส่งออกรายงาน
                </div>
                <div class="flex gap-2">
                    <button onclick="exportPDF()" class="bg-rose-600 hover:bg-rose-700 text-white px-3.5 py-2 rounded-xl text-xs font-semibold flex items-center gap-1.5 shadow">
                        <i data-lucide="file-text" class="w-4 h-4"></i> Export PDF
                    </button>
                    <button onclick="exportExcel()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3.5 py-2 rounded-xl text-xs font-semibold flex items-center gap-1.5 shadow">
                        <i data-lucide="sheet" class="w-4 h-4"></i> Export Excel
                    </button>
                </div>
            </div>

            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                <h3 class="font-bold text-slate-800 mb-4 flex items-center gap-2">
                    <i data-lucide="pie-chart" class="w-5 h-5 text-indigo-600"></i> สรุปอัตราการเข้าเรียนรายบุคคล
                </h3>

                <div class="overflow-x-auto">
                    <table class="w-full text-left text-sm text-slate-600 border-collapse">
                        <thead class="bg-slate-100 text-slate-700 uppercase text-xs">
                            <tr>
                                <th class="p-3 rounded-l-xl">รหัส</th>
                                <th class="p-3">ชื่อ-นามสกุล</th>
                                <th class="p-3">ชั้นเรียน</th>
                                <th class="p-3 text-center text-emerald-600">มาตรงเวลา</th>
                                <th class="p-3 text-center text-amber-600">สาย</th>
                                <th class="p-3 text-center text-rose-600">ขาด</th>
                                <th class="p-3 text-center rounded-r-xl">% เข้าเรียน</th>
                            </tr>
                        </thead>
                        <tbody id="dashboardTable" class="divide-y divide-slate-100">
                            <!-- JS Render Dashboard -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

    </main>

    <!-- Bottom Navigation Bar (แสดงเฉพาะครู บนมือถือ) -->
    <nav id="mobileBottomNav" class="md:hidden fixed bottom-0 left-0 right-0 bg-white border-t border-slate-200 flex justify-around items-center p-2 z-40 shadow-lg">
        <button onclick="switchTab('session-control')" id="m-nav-session-control" class="flex flex-col items-center gap-1 text-[11px] font-semibold text-indigo-600">
            <i data-lucide="play-circle" class="w-5 h-5"></i> คุมเช็กชื่อ
        </button>
        <button onclick="switchTab('scan-student')" id="m-nav-scan-student" class="flex flex-col items-center gap-1 text-[11px] font-medium text-slate-400">
            <i data-lucide="smartphone" class="w-5 h-5"></i> นักเรียนสแกน
        </button>
        <button onclick="switchTab('edit-time')" id="m-nav-edit-time" class="flex flex-col items-center gap-1 text-[11px] font-medium text-slate-400">
            <i data-lucide="clock" class="w-5 h-5"></i> ตารางเวลา
        </button>
        <button onclick="switchTab('students')" id="m-nav-students" class="flex flex-col items-center gap-1 text-[11px] font-medium text-slate-400">
            <i data-lucide="users" class="w-5 h-5"></i> นักเรียน
        </button>
        <button onclick="switchTab('dashboard')" id="m-nav-dashboard" class="flex flex-col items-center gap-1 text-[11px] font-medium text-slate-400">
            <i data-lucide="bar-chart-3" class="w-5 h-5"></i> สถิติ
        </button>
    </nav>

    <!-- Mobile Drawer เลือกห้องเรียน -->
    <div id="mobileClassroomDrawer" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex flex-col justify-end">
        <div class="bg-white rounded-t-3xl p-6 space-y-4 max-h-[80vh] overflow-y-auto">
            <div class="flex justify-between items-center pb-2 border-b">
                <h3 class="font-bold text-slate-800 text-base">📂 เลือกห้องเรียน / รายวิชา</h3>
                <button onclick="toggleMobileClassroomDrawer()" class="p-1 bg-slate-100 rounded-full text-slate-500">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>
            <div id="mobileClassroomList" class="space-y-2">
                <!-- JS Render Options -->
            </div>
        </div>
    </div>

    <!-- Modal QR Code -->
    <div id="qrCodeModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center hidden z-50 p-4">
        <div class="bg-white w-full max-w-sm rounded-3xl p-6 text-center shadow-2xl border border-slate-200">
            <h3 class="font-bold text-lg text-slate-800 mb-1">สแกน QR Code เพื่อเช็กชื่อ</h3>
            <p class="text-xs text-slate-500 mb-4">ให้นักเรียนใช้มือถือสแกนเข้าสู่หน้าเช็กชื่อ</p>
            <div id="qrcode" class="flex justify-center p-4 bg-slate-50 rounded-2xl border border-slate-200 mx-auto mb-4"></div>
            <button onclick="closeQRCodeModal()" class="w-full bg-slate-800 hover:bg-slate-900 text-white py-3 rounded-xl text-sm font-semibold">
                ปิดหน้าต่าง
            </button>
        </div>
    </div>

    <!-- Modal เพิ่ม/แก้ไข ห้องเรียน -->
    <div id="classroomModal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm flex items-center justify-center hidden z-50 p-4">
        <div class="bg-white w-full max-w-md rounded-2xl p-6 shadow-2xl border border-slate-200">
            <h3 id="classModalTitle" class="font-bold text-lg text-slate-800 mb-4 flex items-center gap-2">
                <i data-lucide="book-open" class="w-5 h-5 text-indigo-600"></i> เพิ่มห้องเรียน / รายวิชาใหม่
            </h3>
            
            <form id="classModalForm" onsubmit="saveClassroomModal(event)" class="space-y-3.5">
                <input type="hidden" id="classModalIsEdit" value="false">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสห้องเรียน / รหัสวิชา <span class="text-rose-500">*</span></label>
                    <input type="text" id="classCodeInput" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น CS101">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อรายวิชา</label>
                    <input type="text" id="subjectNameInput" class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น พัฒนาเว็บแอปพลิเคชัน">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อครูผู้สอน</label>
                    <input type="text" id="teacherNameInput" class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น อ.กิตติเดช">
                </div>

                <div class="flex gap-2 pt-2">
                    <button type="submit" class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white py-2.5 rounded-xl text-sm font-semibold shadow">
                        บันทึกข้อมูล
                    </button>
                    <button type="button" onclick="closeClassroomModal()" class="bg-slate-200 hover:bg-slate-300 text-slate-700 px-4 py-2.5 rounded-xl text-sm font-semibold">
                        ยกเลิก
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal แก้ไขเวลา -->
    <div id="editLogModal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm flex items-center justify-center hidden z-50 p-4">
        <div class="bg-white w-full max-w-md rounded-2xl p-6 shadow-2xl border border-slate-200">
            <h3 class="font-bold text-lg text-slate-800 mb-4 flex items-center gap-2">
                <i data-lucide="clock" class="w-5 h-5 text-indigo-600"></i> แก้ไขเวลา/สถานะบันทึก
            </h3>
            
            <form id="editLogForm" onsubmit="saveLogEdit(event)" class="space-y-4">
                <input type="hidden" id="modalLogIdx">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อนักเรียน</label>
                    <input type="text" id="modalStudentName" disabled class="w-full bg-slate-100 border border-slate-300 rounded-xl p-2.5 text-sm text-slate-500 font-medium">
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">วันที่</label>
                        <input type="date" id="modalDate" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">เวลา</label>
                        <input type="text" id="modalTime" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="เช่น 08:30:00 AM">
                    </div>
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">สถานะการเข้าเรียน</label>
                    <select id="modalStatus" class="w-full border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none font-medium">
                        <option value="มาตรงเวลา">มาตรงเวลา</option>
                        <option value="สาย">สาย</option>
                        <option value="ขาด">ขาด</option>
                    </select>
                </div>

                <div class="flex gap-2 pt-2">
                    <button type="submit" class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white py-2.5 rounded-xl text-sm font-semibold shadow">
                        บันทึกการแก้ไข
                    </button>
                    <button type="button" onclick="closeEditModal()" class="bg-slate-200 hover:bg-slate-300 text-slate-700 px-4 py-2.5 rounded-xl text-sm font-semibold">
                        ยกเลิก
                    </button>
                </div>
            </form>
        </div>
    </div>

    <script>
        // --- 1. State & Data Store ---
        let currentClassroom = 'CS101';
        let isSessionActive = false;
        let sessionStartTime = null;
        let currentDashboardFilter = 'all';

        let imgPosX = 0, imgPosY = 0;
        let isDraggingImg = false;
        let startX = 0, startY = 0;
        let currentImgScale = 0.75;
        let baseLoadedImg = new Image();

        let db = JSON.parse(localStorage.getItem('smart_attendance_db_v6.1')) || {
            classroomsDetails: {
                'CS101': { subject: 'การพัฒนาเว็บแอปพลิเคชัน', teacher: 'อ.กิตติเดช', room: 'ห้อง 402' },
                'SOC201': { subject: 'สังคมศึกษาและการสอน', teacher: 'อาจารย์พัชรพล', room: 'ห้อง 301' }
            },
            students: {
                'CS101': [
                    { id: '6601001', name: 'นาย กิตติศักดิ์ สมบูรณ์', grade: 'ม.4/1', photo: '' },
                    { id: '6601002', name: 'นางสาว ปริศนา สุขใจ', grade: 'ม.4/1', photo: '' },
                    { id: '6601003', name: 'นาย พัชรพล ต่อวาส', grade: 'ม.4/2', photo: '' }
                ],
                'SOC201': [
                    { id: '6602001', name: 'นาย ณัฐวุฒิ มีสุข', grade: 'ปี 2', photo: '' }
                ]
            },
            attendanceLogs: {
                'CS101': [
                    { date: new Date().toISOString().split('T')[0], id: '6601001', name: 'นาย กิตติศักดิ์ สมบูรณ์', time: '08:15:20 AM', status: 'มาตรงเวลา' }
                ]
            }
        };

        function saveData() {
            localStorage.setItem('smart_attendance_db_v6.1', JSON.stringify(db));
        }

        // Initialize Application
        document.addEventListener("DOMContentLoaded", () => {
            lucide.createIcons();
            document.getElementById('liveDateText').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'long', day: 'numeric' });
            
            // ตรวจสอบพารามิเตอร์ URL ว่าเป็นการสแกนจาก QR Code หรือลิงก์หรือไม่
            const urlParams = new URLSearchParams(window.location.search);
            const classParam = urlParams.get('class');
            
            if (classParam && db.classroomsDetails[classParam]) {
                currentClassroom = classParam;
                
                // ซ่อนเมนูกวนใจทั้งหมดของครูออกทันที ปรับเป็นโหมดนักเรียนเพียวๆ
                document.getElementById('mobileHeader').style.display = 'none';
                document.getElementById('desktopSidebar').style.display = 'none';
                document.getElementById('mobileBottomNav').style.display = 'none';
                document.getElementById('mainPageHeader').style.display = 'none';
                document.getElementById('shareControlBar').style.display = 'none';
                document.body.classList.remove('pb-16', 'md:pb-0');
                
                switchTab('scan-student');
            }

            renderClassroomOptions();
            initCropperEvents();
            renderAll();
        });

        // --- 2. Navigation Control ---
        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.querySelectorAll('.nav-btn').forEach(el => {
                el.classList.remove('bg-indigo-800', 'text-white', 'shadow-md');
                el.classList.add('text-indigo-200');
            });

            document.querySelectorAll('#mobileBottomNav button').forEach(el => {
                el.classList.remove('text-indigo-600', 'font-semibold');
                el.classList.add('text-slate-400', 'font-medium');
            });

            document.getElementById(`tab-${tabId}`).classList.remove('hidden');
            
            const activeNav = document.getElementById(`nav-${tabId}`);
            if (activeNav) activeNav.classList.add('bg-indigo-800', 'text-white', 'shadow-md');

            const mActiveNav = document.getElementById(`m-nav-${tabId}`);
            if (mActiveNav) {
                mActiveNav.classList.remove('text-slate-400', 'font-medium');
                mActiveNav.classList.add('text-indigo-600', 'font-semibold');
            }

            const titles = {
                'session-control': '⏱️ ควบคุมเปิด/ปิดการเช็กชื่อ (สำหรับครู)',
                'scan-student': '📱 นักเรียนสแกนเช็กชื่อเข้าเรียน',
                'edit-time': '✏️ กำหนดและแก้ไขเวลาเรียน',
                'students': '👤 จัดการข้อมูลนักเรียน (CRUD)',
                'dashboard': '📊 สถิติการเข้าเรียน & Export รายงาน'
            };
            document.getElementById('pageTitle').innerText = titles[tabId];
            renderAll();
        }

        function renderAll() {
            renderHeaderClassDetails();
            renderSessionUI();
            renderStudentSelfSelect();
            renderAttendanceLogs();
            renderStudentList();
            renderDashboard();
        }

        // --- 3. Classroom Management ---
        function renderClassroomOptions() {
            const select = document.getElementById('classroomSelect');
            const mobileList = document.getElementById('mobileClassroomList');
            const classKeys = Object.keys(db.classroomsDetails);
            
            if (!classKeys.includes(currentClassroom) && classKeys.length > 0) {
                currentClassroom = classKeys[0];
            }

            select.innerHTML = classKeys.map(code => {
                const info = db.classroomsDetails[code];
                const subjText = info.subject ? ` - ${info.subject}` : '';
                return `<option value="${code}" ${code === currentClassroom ? 'selected' : ''}>${code}${subjText}</option>`;
            }).join('');

            mobileList.innerHTML = classKeys.map(code => {
                const info = db.classroomsDetails[code];
                const active = code === currentClassroom ? 'bg-indigo-50 border-indigo-500 text-indigo-700' : 'bg-slate-50 border-slate-200 text-slate-700';
                return `
                    <button onclick="selectMobileClassroom('${code}')" class="w-full p-3 rounded-xl border text-left font-semibold text-sm flex justify-between items-center ${active}">
                        <span>${code} ${info.subject ? '- '+info.subject : ''}</span>
                        ${code === currentClassroom ? '<i data-lucide="check" class="w-4 h-4 text-indigo-600"></i>' : ''}
                    </button>
                `;
            }).join('');
            lucide.createIcons();
        }

        function selectMobileClassroom(code) {
            currentClassroom = code;
            document.getElementById('classroomSelect').value = code;
            toggleMobileClassroomDrawer();
            changeClassroom();
        }

        function toggleMobileClassroomDrawer() {
            const drawer = document.getElementById('mobileClassroomDrawer');
            drawer.classList.toggle('hidden');
        }

        function renderHeaderClassDetails() {
            document.getElementById('currentClassText').innerText = currentClassroom;
            document.getElementById('mobileCurrentClassText').innerText = `วิชา: ${currentClassroom}`;
            document.getElementById('studentScanSubjectBadge').innerText = `รายวิชา ${currentClassroom}`;
            
            const info = db.classroomsDetails[currentClassroom] || {};
            document.getElementById('headerSubjectText').innerText = info.subject ? `| วิชา: ${info.subject}` : '';
            document.getElementById('headerTeacherText').innerText = info.teacher ? `| ผู้สอน: ${info.teacher}` : '';
        }

        function changeClassroom() {
            currentClassroom = document.getElementById('classroomSelect').value;
            isSessionActive = false;
            renderAll();
        }

        function openClassroomModal(isEdit = false) {
            document.getElementById('classModalIsEdit').value = isEdit ? "true" : "false";
            const titleEl = document.getElementById('classModalTitle');
            const codeInput = document.getElementById('classCodeInput');

            if (isEdit) {
                titleEl.innerHTML = `<i data-lucide="edit" class="w-5 h-5 text-indigo-600"></i> แก้ไขข้อมูลห้องเรียน / วิชา`;
                const info = db.classroomsDetails[currentClassroom] || {};
                codeInput.value = currentClassroom;
                codeInput.disabled = true;
                document.getElementById('subjectNameInput').value = info.subject || '';
                document.getElementById('teacherNameInput').value = info.teacher || '';
            } else {
                titleEl.innerHTML = `<i data-lucide="plus-circle" class="w-5 h-5 text-indigo-600"></i> เพิ่มห้องเรียน / รายวิชาใหม่`;
                codeInput.value = '';
                codeInput.disabled = false;
                document.getElementById('subjectNameInput').value = '';
                document.getElementById('teacherNameInput').value = '';
            }

            document.getElementById('classroomModal').classList.remove('hidden');
            lucide.createIcons();
        }

        function closeClassroomModal() {
            document.getElementById('classroomModal').classList.add('hidden');
        }

        function saveClassroomModal(e) {
            e.preventDefault();
            const isEdit = document.getElementById('classModalIsEdit').value === "true";
            const code = document.getElementById('classCodeInput').value.trim();
            const subject = document.getElementById('subjectNameInput').value.trim();
            const teacher = document.getElementById('teacherNameInput').value.trim();

            if (!code) return;

            if (!isEdit && db.classroomsDetails[code]) {
                alert("❌ รหัสห้องเรียนนี้มีอยู่แล้วในระบบ");
                return;
            }

            db.classroomsDetails[code] = { subject, teacher };
            if (!db.students[code]) db.students[code] = [];
            if (!db.attendanceLogs[code]) db.attendanceLogs[code] = [];

            currentClassroom = code;
            saveData();
            renderClassroomOptions();
            renderAll();
            closeClassroomModal();
        }

        function deleteCurrentClassroom() {
            const classKeys = Object.keys(db.classroomsDetails);
            if (classKeys.length <= 1) {
                alert("⚠️ ต้องมีห้องเรียนอย่างน้อย 1 ห้อง ไม่สามารถลบทั้งหมดได้");
                return;
            }

            if (confirm(`คุณต้องการลบห้องเรียน "${currentClassroom}" และข้อมูลทั้งหมดในห้องนี้หรือไม่?`)) {
                delete db.classroomsDetails[currentClassroom];
                delete db.students[currentClassroom];
                delete db.attendanceLogs[currentClassroom];

                currentClassroom = Object.keys(db.classroomsDetails)[0];
                saveData();
                renderClassroomOptions();
                renderAll();
            }
        }

        // --- 4. Session Control ---
        function toggleAttendanceSession(active) {
            isSessionActive = active;
            const btnStart = document.getElementById('btnStartSession');
            const btnStop = document.getElementById('btnStopSession');
            const badge = document.getElementById('sessionStatusBadge');
            const startedText = document.getElementById('sessionTimeStarted');

            if (active) {
                const now = new Date();
                sessionStartTime = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
                
                badge.className = "text-xs md:text-sm font-bold inline-flex items-center gap-1.5 mt-1 px-3 py-1 rounded-full bg-emerald-100 text-emerald-700 animate-pulse";
                badge.innerHTML = `<span class="w-2.5 h-2.5 rounded-full bg-emerald-500"></span> กำลังเปิดรับเช็กชื่อ (Active)`;
                startedText.innerText = `เวลาเริ่ม: ${sessionStartTime}`;

                btnStart.disabled = true;
                btnStart.classList.add('opacity-50', 'cursor-not-allowed');
                btnStop.disabled = false;
                btnStop.classList.remove('opacity-50', 'cursor-not-allowed');
            } else {
                badge.className = "text-xs md:text-sm font-bold inline-flex items-center gap-1.5 mt-1 px-3 py-1 rounded-full bg-slate-200 text-slate-600";
                badge.innerHTML = `<span class="w-2.5 h-2.5 rounded-full bg-slate-400"></span> ปิดรับการเช็กชื่อ (Closed)`;

                btnStart.disabled = false;
                btnStart.classList.remove('opacity-50', 'cursor-not-allowed');
                btnStop.disabled = true;
                btnStop.classList.add('opacity-50', 'cursor-not-allowed');
            }

            renderStudentScanAccess();
        }

        function renderSessionUI() {
            const students = db.students[currentClassroom] || [];
            const logs = db.attendanceLogs[currentClassroom] || [];
            const todayStr = new Date().toISOString().split('T')[0];
            const todayLogs = logs.filter(l => l.date === todayStr);

            document.getElementById('sessionTotalStudents').innerText = `${students.length} คน`;
            document.getElementById('sessionCheckedStudents').innerText = `${todayLogs.length} คน`;
            document.getElementById('sessionUncheckedStudents').innerText = `${Math.max(0, students.length - todayLogs.length)} คน`;

            renderStudentScanAccess();
        }

        function renderStudentScanAccess() {
            const alertBox = document.getElementById('studentClosedAlert');
            const formArea = document.getElementById('studentScanFormArea');

            if (isSessionActive) {
                alertBox.classList.add('hidden');
                formArea.classList.remove('opacity-50', 'pointer-events-none');
            } else {
                alertBox.classList.remove('hidden');
                formArea.classList.add('opacity-50', 'pointer-events-none');
            }
        }

        // --- 5. Share Links & QR Code ---
        function getStudentPageURL() {
            let baseUrl = window.location.href.split('?')[0].split('#')[0];
            if (baseUrl.startsWith('file://')) {
                baseUrl = 'https://smart-attendance.app/index.html';
            }
            return `${baseUrl}?class=${encodeURIComponent(currentClassroom)}`;
        }

        function copyStudentLink() {
            const url = getStudentPageURL();
            navigator.clipboard.writeText(url).then(() => {
                alert("📋 คัดลอกลิงก์สำหรับนักเรียนเรียบร้อยแล้ว!\n" + url);
            }).catch(() => {
                prompt("คัดลอกลิงก์ด้านล่างนี้ให้นักเรียน:", url);
            });
        }

        function showQRCodeModal() {
            const url = getStudentPageURL();
            const qrContainer = document.getElementById("qrcode");
            qrContainer.innerHTML = "";

            new QRCode(qrContainer, {
                text: url,
                width: 180,
                height: 180,
                colorDark : "#1e1b4b",
                colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });

            document.getElementById('qrCodeModal').classList.remove('hidden');
        }

        function closeQRCodeModal() {
            document.getElementById('qrCodeModal').classList.add('hidden');
        }

        // --- 6. Student Scan ---
        function renderStudentSelfSelect() {
            const students = db.students[currentClassroom] || [];
            const select = document.getElementById('studentSelfSelect');
            if(students.length === 0) {
                select.innerHTML = '<option value="">-- ไม่มีรายชื่อนักเรียน --</option>';
                return;
            }
            select.innerHTML = students.map(s => `<option value="${s.id}">${s.name} ${s.grade ? '['+s.grade+']' : ''} (${s.id})</option>`).join('');
        }

        async function startStudentCamera() {
            if (!isSessionActive) {
                alert("ขณะนี้ครูยังไม่เปิดรับการเช็กชื่อ");
                return;
            }
            const video = document.getElementById('videoStudent');
            const placeholder = document.getElementById('studentCamPlaceholder');

            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                video.classList.remove('hidden');
                placeholder.classList.add('hidden');
            } catch (err) {
                alert("ไม่สามารถเปิดกล้องได้: " + err.message);
            }
        }

        function confirmStudentSelfScan() {
            if (!isSessionActive) {
                alert("❌ ระบบปิดรับการเช็กชื่อในขณะนี้");
                return;
            }

            const studentId = document.getElementById('studentSelfSelect').value;
            if(!studentId) {
                alert("กรุณาเลือกชื่อนักเรียนก่อนเช็กชื่อ");
                return;
            }

            const students = db.students[currentClassroom] || [];
            const student = students.find(s => s.id === studentId);

            const now = new Date();
            const timeStr = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            const dateStr = now.toISOString().split('T')[0];

            let status = "มาตรงเวลา";

            if (!db.attendanceLogs[currentClassroom]) db.attendanceLogs[currentClassroom] = [];

            const existing = db.attendanceLogs[currentClassroom].find(l => l.id === student.id && l.date === dateStr);
            if (existing) {
                alert(`⚠️ ${student.name} เช็กชื่อในวันนี้ไปแล้ว (${existing.time})`);
                return;
            }

            db.attendanceLogs[currentClassroom].unshift({
                date: dateStr,
                id: student.id,
                name: student.name,
                time: timeStr,
                status: status
            });

            saveData();
            renderAll();
            alert(`🎉 เช็กชื่อสำเร็จ! สวัสดี ${student.name}\nสถานะ: ${status} (เวลา: ${timeStr})`);
        }

        // --- 7. Attendance Logs ---
        function renderAttendanceLogs() {
            const logs = db.attendanceLogs[currentClassroom] || [];
            const tbody = document.getElementById('attendanceLogsTable');

            if (logs.length === 0) {
                tbody.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-slate-400">ยังไม่มีรายการเช็กชื่อในระบบ</td></tr>';
                return;
            }

            tbody.innerHTML = logs.map((log, idx) => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3 font-medium">${log.date}</td>
                    <td class="p-3">${log.id}</td>
                    <td class="p-3 font-semibold text-slate-800">${log.name}</td>
                    <td class="p-3 font-mono">${log.time}</td>
                    <td class="p-3">
                        <span class="px-2.5 py-0.5 rounded-full text-xs font-semibold ${getStatusBadgeClass(log.status)}">${log.status}</span>
                    </td>
                    <td class="p-3 text-center">
                        <button onclick="openEditModal(${idx})" class="text-indigo-600 hover:text-indigo-800 p-1 text-xs font-medium flex items-center gap-1 mx-auto">
                            <i data-lucide="pencil" class="w-3.5 h-3.5"></i> แก้ไข
                        </button>
                    </td>
                </tr>
            `).join('');
            lucide.createIcons();
        }

        function getStatusBadgeClass(status) {
            if (status === 'มาตรงเวลา') return 'bg-emerald-100 text-emerald-700';
            if (status === 'สาย') return 'bg-amber-100 text-amber-700';
            return 'bg-rose-100 text-rose-700';
        }

        function openEditModal(idx) {
            const logs = db.attendanceLogs[currentClassroom] || [];
            const log = logs[idx];

            document.getElementById('modalLogIdx').value = idx;
            document.getElementById('modalStudentName').value = `${log.name} (${log.id})`;
            document.getElementById('modalDate').value = log.date;
            document.getElementById('modalTime').value = log.time;
            document.getElementById('modalStatus').value = log.status;

            document.getElementById('editLogModal').classList.remove('hidden');
        }

        function closeEditModal() {
            document.getElementById('editLogModal').classList.add('hidden');
        }

        function saveLogEdit(e) {
            e.preventDefault();
            const idx = document.getElementById('modalLogIdx').value;
            const logs = db.attendanceLogs[currentClassroom];

            logs[idx].date = document.getElementById('modalDate').value;
            logs[idx].time = document.getElementById('modalTime').value;
            logs[idx].status = document.getElementById('modalStatus').value;

            saveData();
            renderAll();
            closeEditModal();
        }

        // --- 8. Student Photo Cropper ---
        function initCropperEvents() {
            const container = document.getElementById('cropContainer');

            container.addEventListener('mousedown', (e) => {
                isDraggingImg = true;
                startX = e.clientX - imgPosX;
                startY = e.clientY - imgPosY;
            });

            window.addEventListener('mousemove', (e) => {
                if (!isDraggingImg) return;
                imgPosX = e.clientX - startX;
                imgPosY = e.clientY - startY;
                updateCropImageTransform();
            });

            window.addEventListener('mouseup', () => { isDraggingImg = false; });

            container.addEventListener('touchstart', (e) => {
                isDraggingImg = true;
                const touch = e.touches[0];
                startX = touch.clientX - imgPosX;
                startY = touch.clientY - imgPosY;
            });

            window.addEventListener('touchmove', (e) => {
                if (!isDraggingImg) return;
                const touch = e.touches[0];
                imgPosX = touch.clientX - startX;
                imgPosY = touch.clientY - startY;
                updateCropImageTransform();
            });

            window.addEventListener('touchend', () => { isDraggingImg = false; });
        }

        function loadStudentPhotoToCropper(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    baseLoadedImg.onload = function() {
                        imgPosX = 0; imgPosY = 0;
                        const scaleW = 140 / baseLoadedImg.width;
                        const scaleH = 186 / baseLoadedImg.height;
                        const autoFitScale = Math.min(scaleW, scaleH) * 0.95;

                        currentImgScale = autoFitScale;
                        document.getElementById('zoomSlider').value = currentImgScale;

                        const cropImgEl = document.getElementById('cropImg');
                        cropImgEl.src = e.target.result;
                        updateCropImageTransform();

                        document.getElementById('photoCropperArea').classList.remove('hidden');
                    };
                    baseLoadedImg.src = e.target.result;
                };
                reader.readAsDataURL(file);
            }
        }

        function updateCropImageTransform() {
            currentImgScale = parseFloat(document.getElementById('zoomSlider').value);
            const cropImgEl = document.getElementById('cropImg');
            cropImgEl.style.transform = `translate(-50%, -50%) translate(${imgPosX}px, ${imgPosY}px) scale(${currentImgScale})`;
        }

        function getCroppedCanvasBase64() {
            if (!baseLoadedImg.src) return '';

            const canvas = document.createElement('canvas');
            canvas.width = 450;
            canvas.height = 600;
            const ctx = canvas.getContext('2d');

            ctx.fillStyle = "#ffffff";
            ctx.fillRect(0, 0, 450, 600);

            ctx.save();
            const scaleMultiplier = 3.2;
            ctx.translate(225 + (imgPosX * scaleMultiplier), 300 + (imgPosY * scaleMultiplier));
            ctx.scale(currentImgScale * scaleMultiplier, currentImgScale * scaleMultiplier);

            ctx.drawImage(baseLoadedImg, -baseLoadedImg.width / 2, -baseLoadedImg.height / 2);
            ctx.restore();

            return canvas.toDataURL('image/jpeg', 0.90);
        }

        function renderStudentList() {
            const students = db.students[currentClassroom] || [];
            const tbody = document.getElementById('studentListTable');

            if (students.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="p-4 text-center text-slate-400">ยังไม่มีนักเรียนในห้องเรียนนี้</td></tr>';
                return;
            }

            tbody.innerHTML = students.map((s, idx) => {
                const avatar = s.photo 
                    ? `<img src="${s.photo}" class="w-10 h-13 object-cover rounded border border-indigo-200 mx-auto shadow-sm" title="รูปถ่าย 2 นิ้ว">`
                    : `<div class="w-10 h-10 bg-indigo-100 text-indigo-600 rounded-full flex items-center justify-center mx-auto text-xs font-bold">${s.name.substring(0,2)}</div>`;

                return `
                    <tr class="hover:bg-slate-50">
                        <td class="p-2 text-center">${avatar}</td>
                        <td class="p-3 font-medium">${s.id}</td>
                        <td class="p-3 font-semibold text-slate-800">${s.name}</td>
                        <td class="p-3 text-slate-500">${s.grade || '-'}</td>
                        <td class="p-3 text-center flex justify-center gap-2 mt-1">
                            <button onclick="editStudent(${idx})" class="text-amber-600 hover:text-amber-800 p-1">
                                <i data-lucide="edit-2" class="w-4 h-4"></i>
                            </button>
                            <button onclick="deleteStudent(${idx})" class="text-rose-600 hover:text-rose-800 p-1">
                                <i data-lucide="trash-2" class="w-4 h-4"></i>
                            </button>
                        </td>
                    </tr>
                `;
            }).join('');
            lucide.createIcons();
        }

        function saveStudent(e) {
            e.preventDefault();
            const idx = parseInt(document.getElementById('editStudentIdx').value);
            const id = document.getElementById('studentId').value.trim();
            const name = document.getElementById('studentName').value.trim();
            const grade = document.getElementById('studentClassGrade').value.trim();

            if (!db.students[currentClassroom]) db.students[currentClassroom] = [];

            let photo = '';
            if (baseLoadedImg.src) {
                photo = getCroppedCanvasBase64();
            } else if (idx !== -1) {
                photo = db.students[currentClassroom][idx].photo || '';
            }

            if (idx === -1) {
                db.students[currentClassroom].push({ id, name, grade, photo });
            } else {
                db.students[currentClassroom][idx] = { id, name, grade, photo };
            }

            saveData();
            resetStudentForm();
            renderAll();
        }

        function editStudent(idx) {
            const student = db.students[currentClassroom][idx];
            document.getElementById('editStudentIdx').value = idx;
            document.getElementById('studentId').value = student.id;
            document.getElementById('studentName').value = student.name;
            document.getElementById('studentClassGrade').value = student.grade || '';
            
            if (student.photo) {
                baseLoadedImg = new Image();
                baseLoadedImg.onload = function() {
                    document.getElementById('cropImg').src = student.photo;
                    document.getElementById('photoCropperArea').classList.remove('hidden');
                    imgPosX = 0; imgPosY = 0; currentImgScale = 0.75;
                    document.getElementById('zoomSlider').value = 0.75;
                    updateCropImageTransform();
                };
                baseLoadedImg.src = student.photo;
            } else {
                document.getElementById('photoCropperArea').classList.add('hidden');
                baseLoadedImg = new Image();
            }

            document.getElementById('studentFormTitle').innerText = "✏️ แก้ไขข้อมูลนักเรียน";
        }

        function deleteStudent(idx) {
            if (confirm("ยืนยันการลบนักเรียนคนนี้ออกจากห้องเรียน?")) {
                db.students[currentClassroom].splice(idx, 1);
                saveData();
                renderAll();
            }
        }

        function resetStudentForm() {
            document.getElementById('studentForm').reset();
            document.getElementById('editStudentIdx').value = "-1";
            document.getElementById('photoCropperArea').classList.add('hidden');
            baseLoadedImg = new Image();
            imgPosX = 0; imgPosY = 0; currentImgScale = 0.75;
            document.getElementById('studentFormTitle').innerText = "➕ เพิ่มข้อมูลนักเรียนใหม่";
        }

        // --- 9. Dashboard Statistics ---
        function setDashboardTimeFilter(type) {
            currentDashboardFilter = type;
            document.querySelectorAll('.filter-btn').forEach(b => {
                b.classList.remove('bg-white', 'text-indigo-600', 'shadow');
                b.classList.add('text-slate-600');
            });
            const activeBtn = document.getElementById(`filter-${type}`);
            if (activeBtn) activeBtn.classList.add('bg-white', 'text-indigo-600', 'shadow');
            renderDashboard();
        }

        function filterLogsByTimeRange(logs, filterType) {
            const now = new Date();
            const todayStr = now.toISOString().split('T')[0];

            if (filterType === 'daily') {
                return logs.filter(l => l.date === todayStr);
            } else if (filterType === 'weekly') {
                const oneWeekAgo = new Date();
                oneWeekAgo.setDate(now.getDate() - 7);
                return logs.filter(l => new Date(l.date) >= oneWeekAgo);
            } else if (filterType === 'monthly') {
                const currentYearMonth = todayStr.substring(0, 7);
                return logs.filter(l => l.date.startsWith(currentYearMonth));
            }
            return logs;
        }

        function renderDashboard() {
            const students = db.students[currentClassroom] || [];
            let logs = db.attendanceLogs[currentClassroom] || [];
            
            logs = filterLogsByTimeRange(logs, currentDashboardFilter);
            const tbody = document.getElementById('dashboardTable');

            if (students.length === 0) {
                tbody.innerHTML = '<tr><td colspan="7" class="p-4 text-center text-slate-400">ไม่มีข้อมูลนักเรียน</td></tr>';
                return;
            }

            let lowAttendanceCount = 0;

            tbody.innerHTML = students.map(s => {
                const studentLogs = logs.filter(l => l.id === s.id);
                const onTime = studentLogs.filter(l => l.status === 'มาตรงเวลา').length;
                const late = studentLogs.filter(l => l.status === 'สาย').length;
                const absent = studentLogs.filter(l => l.status === 'ขาด').length;
                const total = studentLogs.length || 1;
                const rate = Math.round(((onTime + late) / total) * 100);

                if (rate < 80) lowAttendanceCount++;

                return `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3 font-medium">${s.id}</td>
                        <td class="p-3 font-semibold text-slate-800">${s.name}</td>
                        <td class="p-3 text-slate-500">${s.grade || '-'}</td>
                        <td class="p-3 text-center text-emerald-600 font-bold">${onTime}</td>
                        <td class="p-3 text-center text-amber-600 font-bold">${late}</td>
                        <td class="p-3 text-center text-rose-600 font-bold">${absent}</td>
                        <td class="p-3 text-center font-bold ${rate < 80 ? 'text-rose-600' : 'text-emerald-600'}">${rate}%</td>
                    </tr>
                `;
            }).join('');

            const alertText = document.getElementById('aiAlertText');
            if (lowAttendanceCount > 0) {
                alertText.innerHTML = `พบนักเรียนจำนวน <b>${lowAttendanceCount} คน</b> ที่มีอัตราการเข้าเรียนต่ำกว่าเกณฑ์ (80%) ในช่วงเวลานี้`;
            } else {
                alertText.innerHTML = `นักเรียนทุกคนมีสถิติการเข้าเรียนอยู่ในเกณฑ์ปกติ`;
            }
        }

        function exportPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            const info = db.classroomsDetails[currentClassroom] || {};

            doc.setFontSize(16);
            doc.text(`Attendance Summary (${currentDashboardFilter.toUpperCase()}) - ${currentClassroom}`, 14, 15);
            doc.setFontSize(10);
            doc.text(`Subject: ${info.subject || '-'} | Teacher: ${info.teacher || '-'}`, 14, 22);

            const students = db.students[currentClassroom] || [];
            let logs = db.attendanceLogs[currentClassroom] || [];
            logs = filterLogsByTimeRange(logs, currentDashboardFilter);

            const tableData = students.map(s => {
                const studentLogs = logs.filter(l => l.id === s.id);
                const onTime = studentLogs.filter(l => l.status === 'มาตรงเวลา').length;
                const late = studentLogs.filter(l => l.status === 'สาย').length;
                const absent = studentLogs.filter(l => l.status === 'ขาด').length;
                const total = studentLogs.length || 1;
                const rate = Math.round(((onTime + late) / total) * 100);

                return [s.id, s.name, s.grade || '-', onTime, late, absent, `${rate}%`];
            });

            doc.autoTable({
                head: [['Student ID', 'Name', 'Class', 'On Time', 'Late', 'Absent', 'Rate']],
                body: tableData,
                startY: 28,
            });

            doc.save(`Attendance_Report_${currentClassroom}_${currentDashboardFilter}.pdf`);
        }

        function exportExcel() {
            const students = db.students[currentClassroom] || [];
            let logs = db.attendanceLogs[currentClassroom] || [];
            logs = filterLogsByTimeRange(logs, currentDashboardFilter);

            const excelData = students.map(s => {
                const studentLogs = logs.filter(l => l.id === s.id);
                const onTime = studentLogs.filter(l => l.status === 'มาตรงเวลา').length;
                const late = studentLogs.filter(l => l.status === 'สาย').length;
                const absent = studentLogs.filter(l => l.status === 'ขาด').length;
                const total = studentLogs.length || 1;
                const rate = Math.round(((onTime + late) / total) * 100);

                return {
                    'รหัสนักเรียน': s.id,
                    'ชื่อ-นามสกุล': s.name,
                    'ชั้นเรียน': s.grade || '-',
                    'มาตรงเวลา': onTime,
                    'สาย': late,
                    'ขาด': absent,
                    '% เข้าเรียน': `${rate}%`
                };
            });

            const worksheet = XLSX.utils.json_to_sheet(excelData);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "Summary");
            XLSX.writeFile(workbook, `Attendance_Summary_${currentClassroom}_${currentDashboardFilter}.xlsx`);
        }
    </script>
</body>
</html>
