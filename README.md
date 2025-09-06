<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบติดตามอุณหภูมิและความชื้น โรงพยาบาล</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
    <style>
        :root {
            --ward7-color: #ff6384;
            --ward6-color: #36a2eb;
            --ward5-color: #4bc0c0;
            --ward4-color: #ffcd56;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f8f9fa;
            padding-top: 20px;
        }
        
        .card {
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            margin-bottom: 20px;
            transition: transform 0.3s;
        }
        
        .card:hover {
            transform: translateY(-5px);
        }
        
        .alert-badge {
            position: absolute;
            top: -5px;
            right: -5px;
        }
        
        .ward-card {
            border-left: 5px solid;
        }
        
        .ward-7 {
            border-left-color: var(--ward7-color);
        }
        
        .ward-6 {
            border-left-color: var(--ward6-color);
        }
        
        .ward-5 {
            border-left-color: var(--ward5-color);
        }
        
        .ward-4 {
            border-left-color: var(--ward4-color);
        }
        
        .notification-panel {
            max-height: 400px;
            overflow-y: auto;
        }
        
        .device-status {
            padding: 5px 10px;
            border-radius: 20px;
            font-size: 0.85rem;
            font-weight: 500;
        }
        
        .status-active {
            background-color: #d4edda;
            color: #155724;
        }
        
        .status-inactive {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        .status-maintenance {
            background-color: #fff3cd;
            color: #856404;
        }
        
        .header-bar {
            background: linear-gradient(135deg, #0d6efd 0%, #0a58ca 100%);
            color: white;
            border-radius: 10px;
            padding: 15px;
            margin-bottom: 20px;
        }
        
        @media (max-width: 768px) {
            .header-bar h1 {
                font-size: 1.5rem;
            }
            
            .header-bar p {
                font-size: 0.9rem;
            }
            
            .btn-group .btn {
                font-size: 0.8rem;
                padding: 5px 8px;
            }
        }
    </style>
</head>
<body>
    <div class="container-fluid">
        <!-- Header -->
        <div class="row">
            <div class="col-md-12">
                <div class="header-bar d-flex justify-content-between align-items-center">
                    <div>
                        <h1><i class="bi bi-clipboard2-pulse"></i> ระบบติดตามอุณหภูมิและความชื้น</h1>
                        <p class="mb-0">โรงพยาบาล - แสดงข้อมูลวอร์ด 7, 6, 5, 4</p>
                    </div>
                    <div>
                        <button class="btn btn-light me-2" id="share-btn">
                            <i class="bi bi-share"></i> แชร์
                        </button>
                        <button class="btn btn-light" id="help-btn">
                            <i class="bi bi-question-circle"></i> ช่วยเหลือ
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div class="row">
            <!-- Main Content -->
            <div class="col-lg-9">
                <!-- Ward Selection -->
                <div class="row mb-4">
                    <div class="col-md-12">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">เลือกวอร์ดที่ต้องการ monitor</h5>
                                <div class="btn-group" role="group">
                                    <button type="button" class="btn btn-outline-danger active" data-ward="7">วอร์ด 7</button>
                                    <button type="button" class="btn btn-outline-primary" data-ward="6">วอร์ด 6</button>
                                    <button type="button" class="btn btn-outline-info" data-ward="5">วอร์ด 5</button>
                                    <button type="button" class="btn btn-outline-warning" data-ward="4">วอร์ด 4</button>
                                </div>
                                <div class="mt-2">
                                    <small class="text-muted">อัปเดตล่าสุด: <span id="last-update-time">กำลังโหลด...</span></small>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Charts -->
                <div class="row mb-4">
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">อุณหภูมิ (°C) - วอร์ด <span id="chart-ward">7</span></h5>
                                <canvas id="temperatureChart" height="250"></canvas>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">ความชื้น (%) - วอร์ด <span id="chart-ward-humid">7</span></h5>
                                <canvas id="humidityChart" height="250"></canvas>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Statistics -->
                <div class="row mb-4">
                    <div class="col-md-12">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">สถิติวอร์ด <span id="current-ward">7</span></h5>
                                <div class="row text-center">
                                    <div class="col-md-3">
                                        <div class="card ward-card ward-7">
                                            <div class="card-body">
                                                <h6>อุณหภูมิเฉลี่ย</h6>
                                                <h3 id="avg-temp">25.3°C</h3>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-md-3">
                                        <div class="card ward-card ward-7">
                                            <div class="card-body">
                                                <h6>ความชื้นเฉลี่ย</h6>
                                                <h3 id="avg-humid">65.2%</h3>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-md-3">
                                        <div class="card ward-card ward-7">
                                            <div class="card-body">
                                                <h6>อุณหภูมิสูงสุด</h6>
                                                <h3 id="max-temp">28.7°C</h3>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-md-3">
                                        <div class="card ward-card ward-7">
                                            <div class="card-body">
                                                <h6>อุณหภูมิต่ำสุด</h6>
                                                <h3 id="min-temp">23.1°C</h3>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Device Status -->
                <div class="row mb-4">
                    <div class="col-md-12">
                        <div class="card">
                            <div class="card-body">
                                <div class="d-flex justify-content-between align-items-center">
                                    <h5 class="card-title">สถานะอุปกรณ์</h5>
                                    <div>
                                        <button class="btn btn-sm btn-outline-secondary me-2" id="refresh-btn">
                                            <i class="bi bi-arrow-clockwise"></i> รีเฟรช
                                        </button>
                                        <button class="btn btn-sm btn-outline-primary" id="export-btn">
                                            <i class="bi bi-download"></i> ส่งออก Excel
                                        </button>
                                    </div>
                                </div>
                                <div class="table-responsive">
                                    <table class="table table-hover">
                                        <thead>
                                            <tr>
                                                <th>เลขเครื่อง</th>
                                                <th>ชื่อเครื่อง</th>
                                                <th>ประเภทเครื่อง</th>
                                                <th>สถานะการใช้งาน</th>
                                                <th>วันที่ตรวจสอบล่าสุด</th>
                                                <th>การดำเนินการ</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr>
                                                <td>D-701</td>
                                                <td>เซ็นเซอร์วอร์ด 7-A</td>
                                                <td>เซ็นเซอร์อุณหภูมิ</td>
                                                <td><span class="device-status status-active">ใช้งานได้</span></td>
                                                <td>2025-06-15 14:30</td>
                                                <td>
                                                    <button class="btn btn-sm btn-info">ดูรายละเอียด</button>
                                                </td>
                                            </tr>
                                            <tr>
                                                <td>D-702</td>
                                                <td>เซ็นเซอร์วอร์ด 7-B</td>
                                                <td>เซ็นเซอร์ความชื้น</td>
                                                <td><span class="device-status status-maintenance">กำลังซ่อมบำรุง</span></td>
                                                <td>2025-06-14 09:15</td>
                                                <td>
                                                    <button class="btn btn-sm btn-info">ดูรายละเอียด</button>
                                                </td>
                                            </tr>
                                            <tr>
                                                <td>D-601</td>
                                                <td>เซ็นเซอร์วอร์ด 6-A</td>
                                                <td>เซ็นเซอร์อุณหภูมิ</td>
                                                <td><span class="device-status status-active">ใช้งานได้</span></td>
                                                <td>2025-06-16 10:45</td>
                                                <td>
                                                    <button class="btn btn-sm btn-info">ดูรายละเอียด</button>
                                                </td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Sidebar -->
            <div class="col-lg-3">
                <!-- Notifications -->
                <div class="card mb-4">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <h5 class="card-title">การแจ้งเตือน</h5>
                            <span class="badge bg-danger">3</span>
                        </div>
                        <div class="notification-panel">
                            <div class="alert alert-warning alert-dismissible fade show" role="alert">
                                <strong>วอร์ด 7:</strong> อุณหภูมิสูงเกินค่า (28.7°C)
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                                <div class="text-muted small">10 นาที ที่แล้ว</div>
                            </div>
                            <div class="alert alert-info alert-dismissible fade show" role="alert">
                                <strong>วอร์ด 5:</strong> ความชื้นต่ำกว่าค่า (45.2%)
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                                <div class="text-muted small">1 ชั่วโมง ที่แล้ว</div>
                            </div>
                            <div class="alert alert-danger alert-dismissible fade show" role="alert">
                                <strong>วอร์ด 6:</strong> เซ็นเซอร์ D-602 offline
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                                <div class="text-muted small">2 ชั่วโมง ที่แล้ว</div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Alert Settings -->
                <div class="card mb-4">
                    <div class="card-body">
                        <h5 class="card-title">ตั้งค่าการแจ้งเตือน</h5>
                        <form>
                            <div class="mb-3">
                                <label for="email" class="form-label">อีเมลสำหรับแจ้งเตือน</label>
                                <input type="email" class="form-control" id="email" value="hospital@example.com">
                            </div>
                            <div class="mb-3">
                                <label for="max-temp-alert" class="form-label">อุณหภูมิสูงสุดที่ (°C)</label>
                                <input type="number" class="form-control" id="max-temp-alert" value="28">
                            </div>
                            <div class="mb-3">
                                <label for="min-temp-alert" class="form-label">อุณหภูมิต่ำสุดที่ (°C)</label>
                                <input type="number" class="form-control" id="min-temp-alert" value="22">
                            </div>
                            <button type="submit" class="btn btn-primary w-100">บันทึกการตั้งค่า</button>
                        </form>
                    </div>
                </div>

                <!-- Latest Update -->
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">อัปเดตล่าสุด</h5>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                วอร์ด 7
                                <span class="text-muted">2 นาที ที่แล้ว</span>
                            </li>
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                วอร์ด 6
                                <span class="text-muted">2 นาที ที่แล้ว</span>
                            </li>
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                วอร์ด 5
                                <span class="text-muted">2 นาที ที่แล้ว</span>
                            </li>
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                วอร์ด 4
                                <span class="text-muted">2 นาที ที่แล้ว</span>
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                โภชนาการ
                                <span class="text-muted">2 นาที ที่แล้ว</span>
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                <optgroup>
                                <span class="text-muted">2 นาที ที่แล้ว</span>
                                </optgroup>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>

        <!-- Footer -->
        <div class="row mt-4">
            <div class="col-md-12 text-center">
                <p class="text-muted">ระบบติดตามอุณหภูมิและความชื้นภายในโรงพยาบาล &copy; 2025</p>
            </div>
        </div>
    </div>

    <!-- Help Modal -->
    <div class="modal fade" id="helpModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">คำแนะนำการใช้งาน</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <h6>การดูข้อมูล</h6>
                    <p>เลือกวอร์ดที่ต้องการดูข้อมูลโดยใช้ปุ่มเลือกวอร์ดด้านบน</p>
                    
                    <h6>การแจ้งเตือน</h6>
                    <p>ระบบจะแจ้งเตือนเมื่ออุณหภูมิหรือความชื้นเกินค่าที่กำหนด</p>
                    
                    <h6>การส่งออกข้อมูล</h6>
                    <p>กดปุ่ม "ส่งออก Excel" เพื่อดาวน์โหลดข้อมูลในรูปแบบ Excel</p>
                    
                    <h6>การแชร์</h6>
                    <p>กดปุ่ม "แชร์" เพื่อคัดลอกลิงก์ไปยังคลิปบอร์ดและส่งให้ผู้อื่น</p>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">ปิด</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        // ข้อมูลตัวอย่าง
        const wardData = {
            7: {
                temperature: [24.5, 24.7, 25.0, 25.3, 25.6, 25.8, 26.0, 25.8, 25.5, 25.3],
                humidity: [65, 64, 65, 66, 65, 64, 65, 66, 65, 64],
                timestamps: ['10:00', '10:10', '10:20', '10:30', '10:40', '10:50', '11:00', '11:10', '11:20', '11:30'],
                stats: {
                    avgTemp: 25.3,
                    avgHumid: 65.2,
                    maxTemp: 26.0,
                    minTemp: 24.5
                }
            },
            6: {
                temperature: [23.8, 23.9, 24.1, 24.3, 24.5, 24.7, 24.9, 24.7, 24.5, 24.3],
                humidity: [68, 67, 68, 69, 68, 67, 68, 69, 68, 67],
                timestamps: ['10:00', '10:10', '10:20', '10:30', '10:40', '10:50', '11:00', '11:10', '11:20', '11:30'],
                stats: {
                    avgTemp: 24.5,
                    avgHumid: 68.2,
                    maxTemp: 24.9,
                    minTemp: 23.8
                }
            },
            5: {
                temperature: [25.2, 25.4, 25.7, 25.9, 26.1, 26.3, 26.5, 26.3, 26.0, 25.8],
                humidity: [62, 61, 62, 63, 62, 61, 62, 63, 62, 61],
                timestamps: ['10:00', '10:10', '10:20', '10:30', '10:40', '10:50', '11:00', '11:10', '11:20', '11:30'],
                stats: {
                    avgTemp: 25.9,
                    avgHumid: 62.2,
                    maxTemp: 26.5,
                    minTemp: 25.2
                }
            },
            4: {
                temperature: [24.0, 24.2, 24.4, 24.6, 24.8, 25.0, 25.2, 25.0, 24.8, 24.6],
                humidity: [70, 69, 70, 71, 70, 69, 70, 71, 70, 69],
                timestamps: ['10:00', '10:10', '10:20', '10:30', '10:40', '10:50', '11:00', '11:10', '11:20', '11:30'],
                stats: {
                    avgTemp: 24.7,
                    avgHumid: 70.2,
                    maxTemp: 25.2,
                    minTemp: 24.0
                }
            }
        };

        // ตัวแปรสำหรับเก็บกราฟ
        let tempChart, humidChart;
        let currentWard = 7;

        // ฟังก์ชันสำหรับอัปเดตกราฟ
        function updateCharts(ward) {
            const data = wardData[ward];
            
            // อัปเดตกราฟอุณหภูมิ
            tempChart.data.labels = data.timestamps;
            tempChart.data.datasets[0].data = data.temperature;
            tempChart.data.datasets[0].borderColor = getWardColor(ward);
            tempChart.data.datasets[0].backgroundColor = getWardColor(ward, 0.1);
            tempChart.update();
            
            // อัปเดตกราฟความชื้น
            humidChart.data.labels = data.timestamps;
            humidChart.data.datasets[0].data = data.humidity;
            humidChart.data.datasets[0].borderColor = getWardColor(ward);
            humidChart.data.datasets[0].backgroundColor = getWardColor(ward, 0.1);
            humidChart.update();
            
            // อัปเดตสถิติ
            document.getElementById('avg-temp').textContent = data.stats.avgTemp.toFixed(1) + '°C';
            document.getElementById('avg-humid').textContent = data.stats.avgHumid.toFixed(1) + '%';
            document.getElementById('max-temp').textContent = data.stats.maxTemp.toFixed(1) + '°C';
            document.getElementById('min-temp').textContent = data.stats.minTemp.toFixed(1) + '°C';
            document.getElementById('current-ward').textContent = ward;
            document.getElementById('chart-ward').textContent = ward;
            document.getElementById('chart-ward-humid').textContent = ward;
            
            // อัปเดตการแสดงผลการ์ด
            document.querySelectorAll('.ward-card').forEach(card => {
                card.className = card.className.replace(/ward-\d/g, '');
                card.classList.add('ward-' + ward);
            });
        }

        // ฟังก์ชันสำหรับส่งออก Excel
        function exportToExcel() {
            const data = wardData[currentWard];
            const worksheet = XLSX.utils.aoa_to_sheet([
                ['ข้อมูลวอร์ด ' + currentWard],
                [],
                ['เวลา', 'อุณหภูมิ (°C)', 'ความชื้น (%))'],
                ...data.timestamps.map((time, i) => [time, data.temperature[i], data.humidity[i]])
            ]);
            
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, 'วอร์ด ' + currentWard);
            XLSX.writeFile(workbook, `ward_${currentWard}_data.xlsx`);
        }

        // ฟังก์ชันสำหรับแชร์ลิงก์
        function shareLink() {
            const url = window.location.href;
            navigator.clipboard.writeText(url).then(() => {
                alert('คัดลอกลิงก์ไปยังคลิปบอร์ดแล้ว สามารถแชร์ให้ดูได้แล้ว');
            }).catch(err => {
                console.error('Failed to copy: ', err);
            });
        }

        // ฟังก์ชันสำหรับสีของวอร์ด
        function getWardColor(ward, opacity = 1) {
            const colors = {
                7: `rgba(255, 99, 132, ${opacity})`,
                6: `rgba(54, 162, 235, ${opacity})`,
                5: `rgba(75, 192, 192, ${opacity})`,
                4: `rgba(255, 205, 86, ${opacity})`
            };
            return colors[ward];
        }

        // ฟังก์ชันสำหรับอัปเดตเวลาล่าสุด
        function updateLastUpdateTime() {
            const now = new Date();
            const timeString = now.toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit', second:'2-digit'});
            document.getElementById('last-update-time').textContent = timeString;
        }

        // เริ่มต้นเมื่อหน้าเว็บโหลดเสร็จ
        document.addEventListener('DOMContentLoaded', function() {
            // สร้างกราฟอุณหภูมิ
            const tempCtx = document.getElementById('temperatureChart').getContext('2d');
            tempChart = new Chart(tempCtx, {
                type: 'line',
                data: {
                    labels: wardData[7].timestamps,
                    datasets: [{
                        label: 'อุณหภูมิ (°C)',
                        data: wardData[7].temperature,
                        borderColor: getWardColor(7),
                        backgroundColor: getWardColor(7, 0.1),
                        borderWidth: 2,
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: false,
                            title: {
                                display: true,
                                text: 'อุณหภูมิ (°C)'
                            }
                        }
                    }
                }
            });

            // สร้างกราฟความชื้น
            const humidCtx = document.getElementById('humidityChart').getContext('2d');
            humidChart = new Chart(humidCtx, {
                type: 'line',
                data: {
                    labels: wardData[7].timestamps,
                    datasets: [{
                        label: 'ความชื้น (%)',
                        data: wardData[7].humidity,
                        borderColor: getWardColor(7),
                        backgroundColor: getWardColor(7, 0.1),
                        borderWidth: 2,
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: false,
                            title: {
                                display: true,
                                text: 'ความชื้น (%)'
                            }
                        }
                    }
                }
            });

            // การเปลี่ยนวอร์ด
            document.querySelectorAll('[data-ward]').forEach(button => {
                button.addEventListener('click', function() {
                    const ward = this.getAttribute('data-ward');
                    currentWard = ward;
                    
                    // อัปเดต UI
                    document.querySelectorAll('[data-ward]').forEach(btn => {
                        btn.classList.remove('active');
                    });
                    this.classList.add('active');
                    
                    // อัปเดตกราฟและข้อมูล
                    updateCharts(ward);
                });
            });

            // การส่งออก Excel
            document.getElementById('export-btn').addEventListener('click', exportToExcel);

            // การแชร์ลิงก์
            document.getElementById('share-btn').addEventListener('click', shareLink);

            // การแสดงคำช่วยเหลือ
            document.getElementById('help-btn').addEventListener('click', function() {
                const helpModal = new bootstrap.Modal(document.getElementById('helpModal'));
                helpModal.show();
            });

            // การรีเฟรชข้อมูล
            document.getElementById('refresh-btn').addEventListener('click', function() {
                this.classList.add('spinning');
                setTimeout(() => {
                    this.classList.remove('spinning');
                    updateLastUpdateTime();
                }, 1000);
            });

            // จำลองการอัปเดตข้อมูลแบบเรียลไทม์
            setInterval(() => {
                // เพิ่มข้อมูลใหม่แบบสุ่ม
                const newTime = new Date().toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'});
                const newTemp = wardData[currentWard].temperature[wardData[currentWard].temperature.length - 1] + (Math.random() - 0.5);
                const newHumid = wardData[currentWard].humidity[wardData[currentWard].humidity.length - 1] + (Math.random() - 0.5);
                
                // จำกัดข้อมูลให้แสดงแค่ 10 ค่า
                if (wardData[currentWard].timestamps.length >= 10) {
                    wardData[currentWard].timestamps.shift();
                    wardData[currentWard].temperature.shift();
                    wardData[currentWard].humidity.shift();
                }
                
                // เพิ่มข้อมูลใหม่
                wardData[currentWard].timestamps.push(newTime);
                wardData[currentWard].temperature.push(parseFloat(newTemp.toFixed(1)));
                wardData[currentWard].humidity.push(parseFloat(newHumid.toFixed(1)));
                
                // อัปเดตสถิติ
                const temps = wardData[currentWard].temperature;
                wardData[currentWard].stats = {
                    avgTemp: parseFloat((temps.reduce((a, b) => a + b, 0) / temps.length).toFixed(1)),
                    avgHumid: parseFloat((wardData[currentWard].humidity.reduce((a, b) => a + b, 0) / wardData[currentWard].humidity.length).toFixed(1)),
                    maxTemp: parseFloat(Math.max(...temps).toFixed(1)),
                    minTemp: parseFloat(Math.min(...temps).toFixed(1))
                };
                
                // อัปเดตกราฟ
                updateCharts(currentWard);
                updateLastUpdateTime();
                
                // ตรวจสอบการแจ้งเตือน
                if (newTemp > 28) {
                    // ในระบบจริงจะส่งอีเมลแจ้งเตือนที่นี่
                    console.log(`อุณหภูมิสูงเกินกำหนดในวอร์ด ${currentWard}: ${newTemp.toFixed(1)}°C`);
                }
            }, 10000); // อัปเดตทุก 10 วินาที

            // อัปเดตเวลาล่าสุดเมื่อโหลดหน้า
            updateLastUpdateTime();
        });
    </script>
</body>
</html>
