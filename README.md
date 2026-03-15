<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>حساب الشامل للدخل والمصروفات</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap');

        body {
            font-family: 'Cairo', sans-serif;
            background-color: #f4f7f6;
            margin: 20px;
            color: #333;
        }

        .container {
            max-width: 1100px;
            margin: auto;
            background: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }

        h1 {
            text-align: center;
            color: #2c3e50;
        }

        /* قسم المدخلات */
        .input-section {
            background: #ecf0f1;
            padding: 20px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            align-items: end;
        }

        .input-group {
            display: flex;
            flex-direction: column;
        }

        label {
            margin-bottom: 5px;
            font-weight: bold;
        }

        input, select {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }

        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
        }

        .btn-add-product { background-color: #3498db; color: white; margin-top: 5px; }
        .btn-submit { background-color: #27ae60; color: white; width: 100%; grid-column: 1 / -1; }
        .btn-print { background-color: #e67e22; color: white; margin-top: 20px; }

        /* الجدول */
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: center;
        }

        th {
            background-color: #2c3e50;
            color: white;
        }

        tr:nth-child(even) { background-color: #f9f9f9; }

        /* الملخص السفلي */
        .summary {
            margin-top: 30px;
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            background: #2c3e50;
            color: white;
            padding: 20px;
            border-radius: 8px;
        }

        .summary-item { text-align: center; }
        .summary-item span { display: block; font-size: 1.5em; font-weight: bold; color: #f1c40f; }

        /* التنسيق للطباعة */
        @media print {
            .input-section, .btn-print, .btn-add-product { display: none; }
            body { background: white; margin: 0; }
            .container { box-shadow: none; width: 100%; max-width: 100%; }
        }
    </style>
</head>
<body>

<div class="container" id="reportArea">
    <h1>حساب الشامل للدخل ومصروفات المشروع</h1>

    <!-- قسم الإدخال -->
    <div class="input-section">
        <div class="input-group">
            <label>يوم العمل</label>
            <select id="day">
                <option>الأحد</option>
                <option>الأثنين</option>
                <option>الثلاثاء</option>
                <option>الأربعاء</option>
                <option>الخميس</option>
                <option>الجمعة</option>
                <option>السبت</option>
            </select>
        </div>

        <div class="input-group">
            <label>اسم المنتج</label>
            <input type="text" id="pName" placeholder="مثلاً: قميص">
        </div>

        <div class="input-group">
            <label>الكمية</label>
            <input type="number" id="pQty" value="1">
        </div>

        <div class="input-group">
            <label>تكلفة القطعة الواحدة</label>
            <input type="number" id="pCost" value="0">
            <button class="btn-add-product" onclick="tempAddProduct()">+ أضف المنتج للعملية</button>
        </div>

        <div class="input-group">
            <label>إجمالي سعر البيع (لليوم)</label>
            <input type="number" id="totalSales" placeholder="0">
        </div>

        <div id="tempProducts" style="grid-column: 1 / -1; font-size: 0.9em; color: #2980b9;">
            المنتجات المضافة حالياً: <span id="productListText">لا يوجد</span>
        </div>

        <button class="btn-submit" onclick="addToTable()">إضافة البيانات للجدول</button>
    </div>

    <!-- الجدول -->
    <table id="mainTable">
        <thead>
            <tr>
                <th>يوم العمل</th>
                <th>المنتجات (النوع والعدد)</th>
                <th>تكلفة المنتجات</th>
                <th>إجمالي سعر البيع</th>
                <th>مجموع التكلفة</th>
                <th>صافي الربح</th>
                <th>نسبة الربح %</th>
            </tr>
        </thead>
        <tbody id="tableBody">
            <!-- البيانات ستظهر هنا -->
        </tbody>
    </table>

    <!-- الملخص -->
    <div class="summary">
        <div class="summary-item">أيام العمل<span id="sumDays">0</span></div>
        <div class="summary-item">إجمالي التكلفة<span id="sumCost">0</span></div>
        <div class="summary-item">إجمالي الربح<span id="sumProfit">0</span></div>
        <div class="summary-item">نسبة الربح من التكلفة<span id="sumRate">0%</span></div>
    </div>

    <button class="btn-print" onclick="window.print()">طباعة الجدول PDF</button>
</div>

<script>
    let currentProducts = [];

    // إضافة منتج للقائمة المؤقتة قبل ترحيله للجدول
    function tempAddProduct() {
        const name = document.getElementById('pName').value;
        const qty = parseFloat(document.getElementById('pQty').value);
        const cost = parseFloat(document.getElementById('pCost').value);

        if(name === "" || isNaN(qty) || isNaN(cost)) {
            alert("يرجى ملء بيانات المنتج بشكل صحيح");
            return;
        }

        currentProducts.push({ name, qty, cost, total: qty * cost });
        updateTempList();
        
        // مسح الخانات للمنتج التالي
        document.getElementById('pName').value = "";
        document.getElementById('pQty').value = "1";
        document.getElementById('pCost').value = "0";
    }

    function updateTempList() {
        const text = currentProducts.map(p => `${p.name} (${p.qty})`).join('، ');
        document.getElementById('productListText').innerText = text || "لا يوجد";
    }

    // ترحيل البيانات للجدول الرئيسي
    function addToTable() {
        const day = document.getElementById('day').value;
        const totalSales = parseFloat(document.getElementById('totalSales').value);

        if (currentProducts.length === 0 || isNaN(totalSales)) {
            alert("يرجى إضافة منتج واحد على الأقل وإدخال سعر البيع");
            return;
        }

        let totalDayCost = currentProducts.reduce((sum, p) => sum + p.total, 0);
        let productsDetails = currentProducts.map(p => `${p.name} (${p.qty})`).join('<br>');
        let costsDetails = currentProducts.map(p => `${p.total}`).join('<br>');
        
        let netProfit = totalSales - totalDayCost;
        let profitPercent = (totalDayCost !== 0) ? (netProfit / totalDayCost * 100).toFixed(1) : 0;

        const tableBody = document.getElementById('tableBody');
        const row = `<tr>
            <td>${day}</td>
            <td>${productsDetails}</td>
            <td>${costsDetails}</td>
            <td>${totalSales}</td>
            <td>${totalDayCost}</td>
            <td style="color: ${netProfit >= 0 ? 'green' : 'red'}">${netProfit}</td>
            <td>${profitPercent}%</td>
        </tr>`;

        tableBody.innerHTML += row;

        // إعادة ضبط
        currentProducts = [];
        updateTempList();
        document.getElementById('totalSales').value = "";
        calculateFinalSummary();
    }

    // حساب الإجماليات في الأسفل
    function calculateFinalSummary() {
        const table = document.getElementById('mainTable');
        let totalCost = 0;
        let totalProfit = 0;
        let rowCount = table.rows.length - 1;

        for (let i = 1; i < table.rows.length; i++) {
            totalCost += parseFloat(table.rows[i].cells[4].innerText);
            totalProfit += parseFloat(table.rows[i].cells[5].innerText);
        }

        let totalRate = (totalCost !== 0) ? (totalProfit / totalCost * 100).toFixed(1) : 0;

        document.getElementById('sumDays').innerText = rowCount;
        document.getElementById('sumCost').innerText = totalCost;
        document.getElementById('sumProfit').innerText = totalProfit;
        document.getElementById('sumRate').innerText = totalRate + "%";
    }
</script>

</body>
</html>
