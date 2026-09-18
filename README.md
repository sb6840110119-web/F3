<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ระบบเช็คชื่อป้องกันสแกนซ้ำ & ส่งออกสถิติครู</title>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        * { box-sizing: border-box; font-family: 'Sarabun', sans-serif; }
        body { background-color: #f0f2f5; margin: 0; padding: 15px; display: flex; justify-content: center; }
        .container { width: 100%; max-width: 650px; }
        .card { background: white; padding: 20px; border-radius: 16px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); margin-bottom: 15px; }
        h2 { text-align: center; color: #1a73e8; margin-top: 0; }
        
        .tab-group { display: flex; gap: 8px; margin-bottom: 15px; }
        .tab-btn { flex: 1; padding: 12px 5px; border: none; background: #e4e6eb; border-radius: 10px; font-weight: 600; cursor: pointer; }
        .tab-btn.active { background: #1a73e8; color: white; }

        .form-group { margin-bottom: 12px; }
        .form-group label { display: block; margin-bottom: 5px; font-weight: 600; }
        .form-group input, .form-group select { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 8px; font-size: 1rem; }
        
        .camera-box { width: 100%; aspect-ratio: 4/3; background: #000; border-radius: 12px; overflow: hidden; margin-bottom: 15px; position: relative; }
        video { width: 100%; height: 100%; object-fit: cover; transform: scaleX(-1); }
        canvas { display: none; }
        
        .btn { padding: 10px 16px; border: none; border-radius: 8px; font-size: 0.95rem; font-weight: 600; cursor: pointer; }
        .btn-full { width: 100%; padding: 14px; font-size: 1rem; border-radius: 10px; }
        .btn-blue { background: #1a73e8; color: white; }
        .btn-green { background: #198754; color: white; }
        .btn-danger { background: #dc3545; color: white; }

        .action-bar { display: flex; gap: 8px; margin-bottom: 15px; flex-wrap: wrap; }

        .status-tag { display: inline-block; padding: 3px 8px; border-radius: 12px; font-size: 0.8rem; font-weight: bold; }
        .normal { background: #e6f4ea; color: #137333; }
        .late { background: #fce8e6; color: #c5221f; }

        /* ตารางแสดงผลสถิติ */
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; margin-top: 10px; }
        th, td { padding: 8px 4px; text-align: center; border-bottom: 1px solid #eee; vertical-align: middle; }
        th { background: #f8f9fa; }
        .face-img-td { width: 45px; height: 45px; border-radius: 50%; object-fit: cover; border: 2px solid #1a73e8; }
    </style>
</head>
<body>

<div class="container">
    <div class="tab-group">
        <button class="tab-btn active" id="btnTabCheckin" onclick="showTab('checkinTab', this)">📸 สแกนเช็คชื่อ</button>
        <button class="tab-btn" id="btnTabRegister" onclick="showTab('registerTab', this)">✍️ ลงทะเบียนหน้า</button>
        <button class="tab-btn" id="btnTabReport" onclick="showTab('reportTab', this)">📊 สถิติครู 🔒</button>
    </div>

    <!-- 1. หน้าสแกนเช็คชื่อ -->
    <div id="checkinTab" class="card tab-section">
        <h2>📸 สแกนใบหน้าเช็คชื่อเข้าเรียน</h2>
        <p style="text-align: center; color: #666; font-size: 0.85rem; margin-top: -5px;">ตัดเวลาสายที่ 08:00 น. (สแกนได้คนละ 1 ครั้ง/วัน)</p>

        <div class="form-group">
            <label>เลือกชื่อ-นามสกุล:</label>
            <select id="studentDropdown">
                <option value="">-- เลือกรายชื่อนักเรียน --</option>
            </select>
        </div>

        <div class="camera-box">
            <video id="videoCheckin" autoplay playsinline muted></video>
        </div>

        <button class="btn btn-blue btn-full" onclick="doCheckIn()">📸 ยืนยันสแกนใบหน้า</button>
        <div id="checkinMessage" style="text-align: center; margin-top: 12px;"></div>
    </div>

    <!-- 2. หน้าลงทะเบียนใบหน้า -->
    <div id="registerTab" class="card tab-section" style="display: none;">
        <h2>✍️ ลงทะเบียนใบหน้านักเรียนใหม่</h2>
        
        <div class="form-group">
            <label>ชื่อ-นามสกุล นักเรียน:</label>
            <input type="text" id="regNameInput" placeholder="เช่น นายสมชาย ใจดี">
        </div>

        <div class="camera-box">
            <video id="videoRegister" autoplay playsinline muted></video>
        </div>

        <button class="btn btn-green btn-full" onclick="doRegister()">💾 บันทึกรูปต้นแบบลงระบบ</button>
        <div id="registerMessage" style="text-align: center; margin-top: 12px;"></div>
    </div>

    <!-- 3. หน้าสรุปสถิติครู (ส่งออกเป็น Excel ได้) -->
    <div id="reportTab" class="card tab-section" style="display: none;">
        <h2>📊 สถิติการเข้าเรียนของคุณครู</h2>
        
        <!-- แถบปุ่มจัดการสถิติ -->
        <div class="action-bar">
            <button class="btn btn-green" onclick="exportToExcel()">📥 ส่งออกเป็น Excel (CSV)</button>
            <button class="btn btn-danger" onclick="clearAttendanceHistory()">🗑️ ล้างสถิติต้นวัน</button>
        </div>

        <div style="overflow-x: auto;">
            <table>
                <thead>
                    <tr>
                        <th>ใบหน้า</th>
                        <th>วันที่</th>
                        <th>ชื่อ-นามสกุล</th>
                        <th>เวลา</th>
                        <th>สถานะ</th>
                    </tr>
                </thead>
                <tbody id="reportTableBody">
                    <tr><td colspan="5" style="color:#888;">ยังไม่มีข้อมูลการสแกนวันนี้</td></tr>
                </tbody>
            </table>
        </div>
    </div>

    <!-- Canvas ซ่อนไว้สำหรับจับภาพถ่ายใบหน้า -->
    <canvas id="captureCanvas"></canvas>
</div>

<script>
    let registeredStudents = JSON.parse(localStorage.getItem('faceRegisteredDB')) || [];
    let attendanceRecords = JSON.parse(localStorage.getItem('faceAttendanceDB')) || [];
    
    let activeStream = null;
    let isTeacherUnlocked = false;

    function startCamera(elementId) {
        if (activeStream) activeStream.getTracks().forEach(track => track.stop());
        navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } })
            .then(stream => {
                activeStream = stream;
                document.getElementById(elementId).srcObject = stream;
            })
            .catch(() => alert("⚠️ โปรดอนุญาตให้เข้าถึงกล้องถ่ายรูป"));
    }

    function captureFacePhoto(videoId) {
        const video = document.getElementById(videoId);
        const canvas = document.getElementById('captureCanvas');
        canvas.width = video.videoWidth || 320;
        canvas.height = video.videoHeight || 240;
        const ctx = canvas.getContext('2d');
        
        ctx.translate(canvas.width, 0);
        ctx.scale(-1, 1);
        ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
        
        return canvas.toDataURL('image/jpeg', 0.7);
    }

    function refreshDropdown() {
        const select = document.getElementById('studentDropdown');
        select.innerHTML = '<option value="">-- เลือกรายชื่อนักเรียน --</option>';
        registeredStudents.forEach(s => {
            const opt = document.createElement('option');
            opt.value = s.name;
            opt.innerText = s.name;
            select.appendChild(opt);
        });
    }

    function showTab(tabId, btn) {
        if (tabId === 'reportTab' && !isTeacherUnlocked) {
            const passwordInput = prompt("🔒 กรุณากรอกรหัสผ่านสำหรับคุณครู:");
            if (passwordInput === "123456789") {
                isTeacherUnlocked = true;
                alert("🔓 เข้าสู่ระบบสถิติครูสำเร็จ");
            } else {
                if (passwordInput !== null) alert("❌ รหัสผ่านไม่ถูกต้อง!");
                return;
            }
        }

        document.querySelectorAll('.tab-section').forEach(el => el.style.display = 'none');
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        
        document.getElementById(tabId).style.display = 'block';
        btn.classList.add('active');

        if (tabId === 'checkinTab') {
            refreshDropdown();
            startCamera('videoCheckin');
        } else if (tabId === 'registerTab') {
            startCamera('videoRegister');
        } else if (tabId === 'reportTab') {
            if (activeStream) activeStream.getTracks().forEach(t => t.stop());
            renderReportTable();
        }
    }

    function doRegister() {
        const name = document.getElementById('regNameInput').value.trim();
        if (!name) return alert("กรุณากรอกชื่อ-นามสกุลก่อนบันทึก");
        if (registeredStudents.some(s => s.name === name)) return alert("ชื่อนี้เคยลงทะเบียนไว้แล้วครับ");

        const photoData = captureFacePhoto('videoRegister');
        registeredStudents.push({ name: name, regPhoto: photoData });
        localStorage.setItem('faceRegisteredDB', JSON.stringify(registeredStudents));
        
        document.getElementById('registerMessage').innerHTML = `<div style="color:green; font-weight:bold;">✅ ลงทะเบียนเรียบร้อยแล้ว</div>`;
        document.getElementById('regNameInput').value = '';
        refreshDropdown();
    }

    function doCheckIn() {
        const selectedName = document.getElementById('studentDropdown').value;
        if (!selectedName) return alert("กรุณาเลือกชื่อของคุณก่อนครับ");

        const alreadyChecked = attendanceRecords.some(r => r.name === selectedName);
        if (alreadyChecked) {
            document.getElementById('checkinMessage').innerHTML = `
                <div style="color:#c5221f; font-weight:bold; background:#fce8e6; padding:10px; border-radius:8px;">
                    ❌ คุณ "${selectedName}" ได้ทำการเช็คชื่อไปแล้วก่อนหน้านี้
                </div>`;
            return;
        }

        const currentPhoto = captureFacePhoto('videoCheckin');
        const now = new Date();
        const dateStr = now.toLocaleDateString('th-TH');
        const timeStr = now.toTimeString().split(' ')[0];
        
        const isLate = (now.getHours() > 8) || (now.getHours() === 8 && (now.getMinutes() > 0 || now.getSeconds() > 0));
        const statusText = isLate ? "มาสาย" : "ตรงเวลา";

        attendanceRecords.push({
            date: dateStr,
            name: selectedName,
            time: timeStr,
            status: statusText,
            photo: currentPhoto
        });

        localStorage.setItem('faceAttendanceDB', JSON.stringify(attendanceRecords));

        const badgeClass = isLate ? "late" : "normal";
        document.getElementById('checkinMessage').innerHTML = `
            <div style="color:#1a73e8; font-weight:bold;">✅ สแกนเช็คชื่อสำเร็จ</div>
            <div>${selectedName} (${timeStr} น.)</div>
            <div style="margin-top:5px;"><span class="status-tag ${badgeClass}">${statusText}</span></div>
        `;
    }

    function renderReportTable() {
        const tbody = document.getElementById('reportTableBody');
        tbody.innerHTML = '';

        if (attendanceRecords.length === 0) {
            tbody.innerHTML = '<tr><td colspan="5" style="color:#888;">ยังไม่มีข้อมูลการสแกนวันนี้</td></tr>';
            return;
        }

        [...attendanceRecords].reverse().forEach(r => {
            let tagClass = r.status === "มาสาย" ? "late" : "normal";
            tbody.innerHTML += `
                <tr>
                    <td><img src="${r.photo}" class="face-img-td" alt="ใบหน้า"></td>
                    <td>${r.date || '-'}</td>
                    <td style="text-align:left; font-weight:600;">${r.name}</td>
                    <td>${r.time} น.</td>
                    <td><span class="status-tag ${tagClass}">${r.status}</span></td>
                </tr>
            `;
        });
    }

    // 📥 ฟังก์ชันส่งออกสถิติเป็นไฟล์ Excel / CSV
    function exportToExcel() {
        if (attendanceRecords.length === 0) {
            alert("ยังไม่มีข้อมูลสถิติให้ส่งออกครับ");
            return;
        }

        // หัวตารางไฟล์ CSV
        let csvContent = "\uFEFFลำดับ,วันที่,ชื่อ-นามสกุล,เวลาสแกน,สถานะการเข้าเรียน\n";

        // แปลงข้อมูลเป็นรูปแบบ CSV
        attendanceRecords.forEach((item, index) => {
            csvContent += `${index + 1},"${item.date || '-'}","${item.name}","${item.time} น.","${item.status}"\n`;
        });

        // สร้างไฟล์ดาวน์โหลดอัตโนมัติ
        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.setAttribute("href", url);
        link.setAttribute("download", `รายงานการเช็คชื่อ_${new Date().toISOString().slice(0,10)}.csv`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }

    function clearAttendanceHistory() {
        if (confirm("คุณครูต้องการล้างข้อมูลสถิติการเข้าเรียนทั้งหมดใช่หรือไม่?")) {
            attendanceRecords = [];
            localStorage.removeItem('faceAttendanceDB');
            renderReportTable();
            alert("ล้างข้อมูลสถิติเรียบร้อยแล้ว");
        }
    }

    refreshDropdown();
    startCamera('videoCheckin');
</script>

</body>
</html>
