<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard IoT Lengkap</title>

    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <style>
        body {
            font-family: Arial;
            text-align: center;
            background: #f0f2f5;
        }

        /* JAM */
        #datetime {
            position: fixed;
            top: 10px;
            right: 20px;
            font-size: 14px;
            font-weight: bold;
            background: #007bff;
            color: white;
            padding: 8px 12px;
            border-radius: 10px;
        }

        /* CARD */
        .card {
            display: inline-block;
            padding: 20px;
            border-radius: 15px;
            margin: 10px;
            width: 150px;
            background: white;
            transition: 0.5s;
        }

        .nilai {
            font-size: 25px;
            font-weight: bold;
        }

        /* BUTTON */
        .btn {
            padding: 12px 20px;
            margin: 10px;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            color: white;
        }

        .off {
            background: gray;
        }

        .on {
            background: green;
        }

        /* GRAFIK */
        .grafik-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            flex-wrap: wrap;
        }

            .grafik-container canvas {
                max-width: 300px;
            }

        /* TABEL */
        table {
            margin: 30px auto;
            border-collapse: collapse;
            width: 85%;
            background: white;
        }

        th, td {
            border: 1px solid #ccc;
            padding: 8px;
        }

        th {
            background: #007bff;
            color: white;
        }
    </style>
</head>

<body>

    <!-- JAM -->
    <div id="datetime"></div>

    <h2>Dashboard Monitoring IoT</h2>

    <!-- SENSOR -->
    <div id="cardSuhu" class="card">
        <div>Suhu</div>
        <div id="suhu" class="nilai">-- °C</div>
    </div>

    <div class="card">
        <div>Kelembapan</div>
        <div id="kelembapan" class="nilai">-- %</div>
    </div>

    <div class="card">
        <div>Cahaya</div>
        <div id="cahaya" class="nilai">-- lux</div>
    </div>

    <!-- BUTTON -->
    <h3>Kontrol</h3>

    <button id="lampu" class="btn off" onclick="toggle(this)">Lampu OFF</button>
    <button id="kipas" class="btn off" onclick="toggle(this)">Kipas OFF</button>
    <button id="racun" class="btn off" onclick="toggle(this)">Pompa Racun OFF</button>
    <button id="pendingin" class="btn off" onclick="toggle(this)">Pompa Pendingin OFF</button>

    <!-- GRAFIK -->
    <div class="grafik-container">
        <canvas id="chartSuhu"></canvas>
        <canvas id="chartKelembapan"></canvas>
        <canvas id="chartCahaya"></canvas>
    </div>

    <!-- HISTORI SENSOR -->
    <h3>Histori Sensor (30 detik)</h3>
    <table id="tabelSensor">
        <tr>
            <th>Waktu</th>
            <th>Suhu</th>
            <th>Kelembapan</th>
            <th>Cahaya</th>
        </tr>
    </table>

    <!-- HISTORI BUTTON -->
    <h3>Histori Tombol</h3>
    <table id="tabelButton">
        <tr>
            <th>Waktu</th>
            <th>Perangkat</th>
            <th>Status</th>
        </tr>
    </table>

    <script>
        // ===== JAM REAL TIME =====
        function updateJam() {
            let now = new Date();

            let tanggal = now.toLocaleDateString('id-ID', {
                weekday: 'long',
                year: 'numeric',
                month: 'long',
                day: 'numeric'
            });

            let jam = now.toLocaleTimeString('id-ID');

            document.getElementById("datetime").innerHTML = tanggal + "<br>" + jam;
        }
        setInterval(updateJam, 1000);
        updateJam();

        // ===== DATA HISTORI =====
        let historiSensor = [];

        window.onload = function () {

            const chartSuhu = new Chart(document.getElementById('chartSuhu'), {
                type: 'line',
                data: { labels: [], datasets: [{ label: 'Suhu', data: [] }] },
                options: { animation: false }
            });

            const chartKelembapan = new Chart(document.getElementById('chartKelembapan'), {
                type: 'line',
                data: { labels: [], datasets: [{ label: 'Kelembapan', data: [] }] },
                options: { animation: false }
            });

            const chartCahaya = new Chart(document.getElementById('chartCahaya'), {
                type: 'line',
                data: { labels: [], datasets: [{ label: 'Cahaya', data: [] }] },
                options: { animation: false }
            });

            function updateData() {

                let waktu = new Date().toLocaleTimeString();

                let suhu = Math.floor(Math.random() * 11 + 25);
                let kelembapan = Math.floor(Math.random() * 20 + 60);
                let cahaya = Math.floor(Math.random() * 5000);

                document.getElementById("suhu").innerHTML = suhu + " °C";
                document.getElementById("kelembapan").innerHTML = kelembapan + " %";
                document.getElementById("cahaya").innerHTML = cahaya + " lux";

                // warna suhu
                let card = document.getElementById("cardSuhu");
                if (suhu > 30) {
                    card.style.backgroundColor = "red";
                    card.style.color = "white";
                } else if (suhu < 30) {
                    card.style.backgroundColor = "lightblue";
                    card.style.color = "black";
                } else {
                    card.style.backgroundColor = "white";
                    card.style.color = "black";
                }

                // otomatis
                let lampu = document.getElementById("lampu");
                let kipas = document.getElementById("kipas");
                let pendingin = document.getElementById("pendingin");

                suhu > 30 ? setON(kipas, "Kipas") : setOFF(kipas, "Kipas");
                suhu > 30 ? setON(pendingin, "Pompa Pendingin") : setOFF(pendingin, "Pompa Pendingin");
                cahaya < 2500 ? setON(lampu, "Lampu") : setOFF(lampu, "Lampu");

                // grafik
                chartSuhu.data.labels.push(waktu);
                chartSuhu.data.datasets[0].data.push(suhu);

                chartKelembapan.data.labels.push(waktu);
                chartKelembapan.data.datasets[0].data.push(kelembapan);

                chartCahaya.data.labels.push(waktu);
                chartCahaya.data.datasets[0].data.push(cahaya);

                chartSuhu.update();
                chartKelembapan.update();
                chartCahaya.update();

                historiSensor.push({ waktu, suhu, kelembapan, cahaya });
            }

            setInterval(updateData, 5000);
            setInterval(tampilHistori, 30000);

            function tampilHistori() {
                let tabel = document.getElementById("tabelSensor");
                let data = historiSensor[historiSensor.length - 1];

                if (!data) return;

                let row = tabel.insertRow();
                row.insertCell(0).innerHTML = data.waktu;
                row.insertCell(1).innerHTML = data.suhu;
                row.insertCell(2).innerHTML = data.kelembapan;
                row.insertCell(3).innerHTML = data.cahaya;
            }
        };

        // ===== BUTTON =====
        function setON(btn, nama) {
            btn.classList.remove("off");
            btn.classList.add("on");
            btn.innerHTML = nama + " ON";
        }

        function setOFF(btn, nama) {
            btn.classList.remove("on");
            btn.classList.add("off");
            btn.innerHTML = nama + " OFF";
        }

        function toggle(btn) {

            let status;

            if (btn.classList.contains("off")) {
                btn.classList.replace("off", "on");
                btn.innerHTML = btn.innerHTML.replace("OFF", "ON");
                status = "ON";

                if (btn.id === "racun") {
                    alert("⚠️ BAHAYA! Pompa Racun Aktif!");
                }

            } else {
                btn.classList.replace("on", "off");
                btn.innerHTML = btn.innerHTML.replace("ON", "OFF");
                status = "OFF";
            }

            let waktu = new Date().toLocaleTimeString();
            let tabel = document.getElementById("tabelButton");

            let row = tabel.insertRow();
            row.insertCell(0).innerHTML = waktu;
            row.insertCell(1).innerHTML = btn.id;
            row.insertCell(2).innerHTML = status;
        }
    </script>

</body>
</html>
