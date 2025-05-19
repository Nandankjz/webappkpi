<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>KPI Kasir LMI 05 - Lengkap + Budget</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
  <style>
    body { background: #0f172a; color: #fff; font-family: Arial; padding: 20px; }
    h2 { margin-top: 30px; }
    input, select, button { padding: 6px; margin: 5px; border-radius: 4px; border: none; }
    button { background: #0ea5e9; color: white; cursor: pointer; }
    table { width: 100%; margin-top: 10px; border-collapse: collapse; }
    th, td { border: 1px solid #334155; padding: 8px; text-align: center; }
    th { background: #1e293b; }
    canvas { background: #1e293b; margin-top: 20px; border-radius: 8px; }
  </style>
</head>
<body>

<h1 style="text-align:center">📊 KPI Kasir LMI 05</h1>

<h2>1. Tambah Kasir</h2>
<input type="text" id="nikKasir" placeholder="NIK Kasir" />
<input type="text" id="namaKasir" placeholder="Nama Kasir" />
<button onclick="tambahKasir()">Tambah Kasir</button>

<h2>2. Hapus Kasir</h2>
<select id="selectKasir"></select>
<button onclick="hapusKasir()">Hapus Kasir</button>

<h2>3. Input Data KPI</h2>
<select id="selectNamaInput"></select>
<input type="date" id="tanggal">
<input type="number" id="pulsa" placeholder="Pulsa">
<input type="number" id="token" placeholder="Token">
<input type="number" id="member" placeholder="New Member">
<input type="number" id="cashback" placeholder="Cashback">
<input type="number" id="selisih" placeholder="Short/Over">
<button onclick="tambahData()">Input Data</button>

<h2>4. Input Budget Bulanan</h2>
<input type="month" id="bulanBudget">
<input type="number" id="budgetPulsa" placeholder="Budget Pulsa">
<input type="number" id="budgetToken" placeholder="Budget Token">
<input type="number" id="budgetMember" placeholder="Budget Member">
<button onclick="simpanBudget()">Simpan Budget</button>

<h2>5. Rekap Harian</h2>
<input type="date" id="filterTanggal" onchange="tampilkanHarian()">
<table id="tabelHarian">
  <thead>
    <tr>
      <th>No</th><th>NIK</th><th>Nama</th><th>Tanggal</th><th>Pulsa</th><th>Token</th><th>Member</th><th>Cashback</th><th>Selisih</th><th>Score</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<h2>6. Rekap Bulanan</h2>
<input type="month" id="bulanFilter" onchange="tampilkanBulanan()">
<button onclick="exportPDF()">Export PDF</button>
<button onclick="exportExcel()">Export Excel</button>
<table id="tabelBulanan">
  <thead>
    <tr><th>Peringkat</th><th>NIK</th><th>Nama</th><th>Pulsa</th><th>Token</th><th>Member</th><th>Cashback</th><th>Selisih</th><th>Score</th></tr>
  </thead>
  <tbody></tbody>
</table>

<h2>7. Grafik Performa Kasir</h2>
<canvas id="grafikKPI" height="100"></canvas>

<script>
let kasirList = JSON.parse(localStorage.getItem('kasirList') || '[]');
let dataKPI = JSON.parse(localStorage.getItem('dataKPI') || '[]');
let budgetList = JSON.parse(localStorage.getItem('budgetList') || '[]');

function simpan() {
  localStorage.setItem('kasirList', JSON.stringify(kasirList));
  localStorage.setItem('dataKPI', JSON.stringify(dataKPI));
  localStorage.setItem('budgetList', JSON.stringify(budgetList));
}

function loadKasir() {
  const selectKasir = document.getElementById("selectKasir");
  const selectInput = document.getElementById("selectNamaInput");
  selectKasir.innerHTML = kasirList.map(k => `<option value="${k.nik}">${k.nik} - ${k.nama}</option>`).join('');
  selectInput.innerHTML = kasirList.map(k => `<option value="${k.nik}">${k.nik} - ${k.nama}</option>`).join('');
}

function tambahKasir() {
  const nik = document.getElementById("nikKasir").value.trim();
  const nama = document.getElementById("namaKasir").value.trim();
  if (!nik || !nama) return alert("Isi NIK dan Nama.");
  if (kasirList.some(k => k.nik === nik)) return alert("NIK sudah terdaftar.");
  kasirList.push({ nik, nama });
  simpan(); loadKasir();
  document.getElementById("nikKasir").value = "";
  document.getElementById("namaKasir").value = "";
}

function hapusKasir() {
  const nik = document.getElementById("selectKasir").value;
  if (!nik) return alert("Pilih kasir.");
  if (confirm(`Hapus kasir NIK ${nik}?`)) {
    kasirList = kasirList.filter(k => k.nik !== nik);
    dataKPI = dataKPI.filter(d => d.nik !== nik);
    simpan(); loadKasir(); tampilkanHarian(); tampilkanBulanan(); updateChart();
  }
}

function tambahData() {
  const nik = document.getElementById("selectNamaInput").value;
  const kasir = kasirList.find(k => k.nik === nik);
  const tanggal = document.getElementById("tanggal").value;
  const pulsa = +document.getElementById("pulsa").value;
  const token = +document.getElementById("token").value;
  const member = +document.getElementById("member").value;
  const cashback = +document.getElementById("cashback").value;
  const selisih = +document.getElementById("selisih").value;
  const score = Math.min(100, member * 50 + token * 30 + pulsa * 20);
  if (!kasir) return alert("Kasir tidak ditemukan.");
  dataKPI.push({ nik, nama: kasir.nama, tanggal, pulsa, token, member, cashback, selisih, score });
  simpan(); tampilkanHarian(); tampilkanBulanan(); updateChart();
}
function tambahData() {
  const nik = document.getElementById("selectNamaInput").value;
  const kasir = kasirList.find(k => k.nik === nik);
  const tanggal = document.getElementById("tanggal").value;
  const pulsa = +document.getElementById("pulsa").value;
  const token = +document.getElementById("token").value;
  const member = +document.getElementById("member").value;
  const cashback = +document.getElementById("cashback").value;
  const selisih = +document.getElementById("selisih").value;
  const score = Math.min(100, member * 50 + token * 30 + pulsa * 20);

  if (!kasir) return alert("Kasir tidak ditemukan.");
  dataKPI.push({ nik, nama: kasir.nama, tanggal, pulsa, token, member, cashback, selisih, score });
  simpan(); tampilkanHarian(); tampilkanBulanan(); updateChart();

  // 🔁 Kosongkan form setelah submit
  document.getElementById("tanggal").value = "";
  document.getElementById("pulsa").value = "";
  document.getElementById("token").value = "";
  document.getElementById("member").value = "";
  document.getElementById("cashback").value = "";
  document.getElementById("selisih").value = "";
}

function simpanBudget() {
  const bulan = document.getElementById("bulanBudget").value;
  const pulsa = +document.getElementById("budgetPulsa").value;
  const token = +document.getElementById("budgetToken").value;
  const member = +document.getElementById("budgetMember").value;
  if (!bulan) return alert("Pilih bulan.");
  const index = budgetList.findIndex(b => b.bulan === bulan);
  const newBudget = { bulan, pulsa, token, member };
  if (index >= 0) budgetList[index] = newBudget;
  else budgetList.push(newBudget);
  simpan();
  alert("Budget disimpan!");
}

function tampilkanHarian() {
  const tbody = document.querySelector("#tabelHarian tbody");
  const filter = document.getElementById("filterTanggal").value;
  tbody.innerHTML = "";
  dataKPI.filter(d => !filter || d.tanggal === filter).forEach((d, i) => {
    tbody.innerHTML += `<tr><td>${i+1}</td><td>${d.nik}</td><td>${d.nama}</td><td>${d.tanggal}</td>
      <td>Rp ${d.pulsa.toLocaleString()}</td><td>Rp ${d.token.toLocaleString()}</td><td>${d.member}</td>
      <td>Rp ${d.cashback.toLocaleString()}</td><td>Rp ${d.selisih.toLocaleString()}</td><td>${d.score}%</td></tr>`;
  });
}

function tampilkanBulanan() {
  const tbody = document.querySelector("#tabelBulanan tbody");
  const bulan = document.getElementById("bulanFilter").value;
  tbody.innerHTML = "";
  const budget = budgetList.find(b => b.bulan === bulan);
  const summary = {};

  dataKPI.forEach(d => {
    if (bulan && !d.tanggal.startsWith(bulan)) return;
    if (!summary[d.nik]) summary[d.nik] = { nama: d.nama, pulsa: 0, token: 0, member: 0, cashback: 0, selisih: 0, skor: [] };
    let s = summary[d.nik];
    s.pulsa += d.pulsa; s.token += d.token; s.member += d.member;
    s.cashback += d.cashback; s.selisih += d.selisih; s.skor.push(d.score);
  });

  const result = Object.entries(summary).map(([nik, v]) => ({
    nik, ...v, rata: v.skor.reduce((a,b)=>a+b,0)/v.skor.length
  })).sort((a,b)=>b.rata-a.rata);

  if (budget) {
    tbody.insertAdjacentHTML("beforebegin", `<tr><td colspan="9"><b>🎯 Budget Bulan Ini</b> | Pulsa: Rp ${budget.pulsa.toLocaleString()}, Token: Rp ${budget.token.toLocaleString()}, Member: ${budget.member}</td></tr>`);
  }

  result.forEach((d, i) => {
    tbody.innerHTML += `<tr><td>${i+1}</td><td>${d.nik}</td><td>${d.nama}</td>
      <td>Rp ${d.pulsa.toLocaleString()}</td><td>Rp ${d.token.toLocaleString()}</td><td>${d.member}</td>
      <td>Rp ${d.cashback.toLocaleString()}</td><td>Rp ${d.selisih.toLocaleString()}</td><td>${d.rata.toFixed(1)}%</td></tr>`;
  });
}

function updateChart() {
  const ctx = document.getElementById("grafikKPI").getContext("2d");
  if (window.myChart) window.myChart.destroy();
  const labels = kasirList.map(k => k.nama);
  const data = labels.map(nama => {
    const skor = dataKPI.filter(d => d.nama === nama).map(d => d.score);
    return skor.length ? (skor.reduce((a,b)=>a+b,0)/skor.length).toFixed(1) : 0;
  });
  window.myChart = new Chart(ctx, {
    type: 'bar',
    data: { labels, datasets: [{ label: "Score", data, backgroundColor: "#38bdf8" }] },
    options: { scales: { y: { beginAtZero: true, max: 100 } } }
  });
}

function exportPDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.text("Rekap Bulanan KPI Kasir", 10, 10);
  doc.autoTable({ html: "#tabelBulanan", startY: 20 });
  doc.save("Rekap-KPI.pdf");
}

function exportExcel() {
  const table = document.getElementById("tabelBulanan");
  const wb = XLSX.utils.table_to_book(table, { sheet: "Rekap KPI" });
  XLSX.writeFile(wb, "Rekap-KPI.xlsx");
}

loadKasir();
tampilkanHarian();
tampilkanBulanan();
updateChart();
</script>
</body>
</html>
