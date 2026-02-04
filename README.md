<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>تحليل طلبات المتجر</title>

<!-- CSV Parser -->
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

<!-- jsPDF -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.29/jspdf.plugin.autotable.min.js"></script>

<style>
body {
    font-family: Tahoma, Arial, sans-serif;
    background: #fff;
    color: #000;
    padding: 40px;
}

.header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 30px;
}

.logo {
    width: 140px;
    border: 1px solid #000;
    padding: 8px;
}

h1 {
    margin: 0;
    font-size: 32px;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 30px;
}

th, td {
    border: 1px solid #000;
    padding: 12px;
    text-align: center;
}

th {
    background: #000;
    color: #fff;
}

.summary {
    margin-top: 20px;
    font-size: 18px;
    font-weight: bold;
}

.export {
    margin-top: 30px;
    text-align: center;
}

button {
    background: #000;
    color: #fff;
    border: none;
    padding: 12px 25px;
    font-size: 16px;
    cursor: pointer;
}

button:hover {
    opacity: 0.85;
}
</style>
</head>

<body>

<!-- الهيدر -->
<div class="header">
    <h1>تحليل طلبات المتجر</h1>
    <img src="https://c.top4top.io/p_3394eg59k1.png" class="logo" alt="Logo">
</div>

<!-- رفع الملف -->
<input type="file" id="fileInput" accept=".csv">

<!-- الجدول -->
<table id="reportTable">
<thead>
<tr>
    <th>اسم المنتج</th>
    <th>الكمية المباعة</th>
    <th>SKU</th>
</tr>
</thead>
<tbody id="tableBody"></tbody>
</table>

<!-- الإجمالي -->
<div class="summary" id="summary">
إجمالي الكمية المباعة: 0
</div>

<!-- زر التصدير -->
<div class="export">
    <button onclick="exportPDF()">تصدير التقرير PDF</button>
</div>

<script>
document.getElementById("fileInput").addEventListener("change", function(e) {
    const file = e.target.files[0];
    if (!file) return;

    Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: function(results) {
            analyze(results.data);
        }
    });
});

function analyze(data) {
    const products = {};

    data.forEach(row => {
        const rawField = row["اسماء المنتجات مع SKU"];
        if (!rawField) return;

        const items = rawField.split(/\n|,/);

        items.forEach(text => {
            const skuMatch = text.match(/SKU\s*:\s*([A-Z0-9]+)/i);
            const sku = skuMatch ? skuMatch[1] : "-";

            let name = text
                .replace(/\(.*?\)/g, "")
                .replace(/-\s*\d+.*$/g, "")
                .trim();

            const key = sku + "|" + name;

            if (!products[key]) {
                products[key] = { name, sku, qty: 0 };
            }
            products[key].qty += 1;
        });
    });

    render(products);
}

function render(products) {
    const body = document.getElementById("tableBody");
    body.innerHTML = "";
    let total = 0;

    Object.values(products).forEach(p => {
        total += p.qty;
        body.innerHTML += `
            <tr>
                <td>${p.name}</td>
                <td>${p.qty}</td>
                <td>${p.sku}</td>
            </tr>
        `;
    });

    document.getElementById("summary").innerText =
        "إجمالي الكمية المباعة: " + total;
}

// تصدير PDF
function exportPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF("p", "pt");

    doc.setFontSize(18);
    doc.text("تحليل طلبات المتجر", 400, 40, { align: "right" });

    doc.setFontSize(12);
    doc.text(document.getElementById("summary").innerText, 400, 65, { align: "right" });

    doc.autoTable({
        html: "#reportTable",
        startY: 90,
        styles: { halign: "center" },
        headStyles: { fillColor: [0, 0, 0] },
        theme: "grid"
    });

    doc.save("store-orders-report.pdf");
}
</script>

</body>
</html>
