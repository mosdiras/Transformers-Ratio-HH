ทีมงานหม้อแปลง กฟส.หห
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>เครื่องมือคำนวณและประเมินผล TTR Ratio</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #f4f7f6;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .card {
            background: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 500px;
        }
        h2 {
            margin-top: 0;
            color: #333;
            text-align: center;
            font-size: 1.5rem;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            color: #555;
            font-weight: 600;
        }
        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 1rem;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 1rem;
            font-weight: bold;
            cursor: pointer;
            margin-top: 10px;
            transition: background 0.3s;
        }
        button:hover {
            background-color: #0056b3;
        }
        .result-box {
            margin-top: 20px;
            padding: 15px;
            border-radius: 8px;
            display: none;
        }
        .result-box.normal {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        .result-box.warning {
            background-color: #fff3cd;
            color: #856404;
            border: 1px solid #ffeeba;
        }
        .result-box.critical {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        .result-item {
            margin-bottom: 8px;
            font-size: 0.95rem;
        }
        .status-tag {
            font-weight: bold;
            font-size: 1.1rem;
        }
    </style>
</head>
<body>

<div class="card">
    <h2>คำนวณและประเมินผล TTR Test</h2>
    
    <div class="form-group">
        <label>แรงดันไฟฝั่งแรงสูงตาม Nameplate (HV Volts):</label>
        <input type="number" id="hvVolt" placeholder="เช่น 22000" step="any">
    </div>

    <div class="form-group">
        <label>แรงดันไฟฝั่งแรงต่ำตาม Nameplate (LV Volts):</label>
        <input type="number" id="lvVolt" placeholder="เช่น 400" step="any">
    </div>

    <div class="form-group">
        <label>รูปแบบการต่อสายขดลวดแรงต่ำ (LV Connection):</label>
        <select id="lvConnection">
            <option value="star">Star (Y) - คำนวณ V_LV / √3</option>
            <option value="delta">Delta (D) - คำนวณ V_LV ตรง</option>
        </select>
    </div>

    <div class="form-group">
        <label>ค่า Ratio ที่วัดได้จริงจากเครื่อง (Measured Ratio):</label>
        <input type="number" id="measuredRatio" placeholder="เช่น 95.677" step="any">
    </div>

    <button onclick="calculateTTR()">วิเคราะห์และประเมินผล</button>

    <div id="result" class="result-box">
        <div class="result-item">อัตราส่วนมาตรฐาน (Calculated Ratio): <strong id="resCalculated">-</strong></div>
        <div class="result-item">เปอร์เซ็นต์ความคลาดเคลื่อน (% Error): <strong id="resError">-</strong></div>
        <div class="result-item">สถานะการประเมิน: <span id="resStatus" class="status-tag">-</span></div>
        <div class="result-item" id="resAdvice" style="margin-top: 10px; font-size: 0.85rem;"></div>
    </div>
</div>

<script>
function calculateTTR() {
    const hvVolt = parseFloat(document.getElementById('hvVolt').value);
    const lvVolt = parseFloat(document.getElementById('lvVolt').value);
    const lvConnection = document.getElementById('lvConnection').value;
    const measuredRatio = parseFloat(document.getElementById('measuredRatio').value);

    if (isNaN(hvVolt) || isNaN(lvVolt) || isNaN(measuredRatio)) {
        alert('กรุณากรอกข้อมูลตัวเลขให้ครบทุกช่อง');
        return;
    }

    // คำนวณ Phase Voltage ฝั่ง LV
    let lvPhaseVolt = lvVolt;
    if (lvConnection === 'star') {
        lvPhaseVolt = lvVolt / Math.sqrt(3);
    }

    // คำนวณ Calculated Ratio
    const calculatedRatio = hvVolt / lvPhaseVolt;

    // คำนวณ % Error
    const percentError = Math.abs((measuredRatio - calculatedRatio) / calculatedRatio) * 100;

    // แสดงผลตัวเลข
    document.getElementById('resCalculated').innerText = calculatedRatio.toFixed(3);
    document.getElementById('resError').innerText = percentError.toFixed(2) + ' %';

    const resultBox = document.getElementById('result');
    const statusSpan = document.getElementById('resStatus');
    const adviceDiv = document.getElementById('resAdvice');

    // ลบ Class เดิมออก
    resultBox.className = 'result-box';

    // ประเมินผลตามเกณฑ์มาตรฐาน IEEE / IEC
    if (percentError <= 0.5) {
        resultBox.classList.add('normal');
        statusSpan.innerText = 'PASS (ปกติ)';
        adviceDiv.innerText = 'ค่า Ratio อยู่ในเกณฑ์มาตรฐาน (ไม่เกิน ±0.5%) หม้อแปลงอยู่ในสภาพพร้อมใช้งาน';
    } else if (percentError <= 1.0) {
        resultBox.classList.add('warning');
        statusSpan.innerText = 'WARNING (เริ่มมีความผิดปกติ)';
        adviceDiv.innerText = 'ค่าความคลาดเคลื่อนอยู่ระหว่าง 0.5% - 1.0% ควรตรวจสอบตำแหน่ง Tap Changer หรือความสะอาดจุดเชื่อมต่อ';
    } else {
        resultBox.classList.add('critical');
        statusSpan.innerText = 'CRITICAL / FAULT (วิกฤต)';
        adviceDiv.innerText = 'ค่าความคลาดเคลื่อนเกิน ±0.5% (หรือเกิน 1%) ควรตรวจสอบการคีบสายวัด สลับขั้ว หรือตรวจเช็กขดลวดหม้อแปลงภายใน';
    }

    resultBox.style.display = 'block';
}
</script>

</body>
</html>
# Transformers-Ratio-HH
