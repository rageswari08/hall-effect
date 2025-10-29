# hall-effect
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hall Sensor Live Bar Graph</title>

  <!-- Chart.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    body {
      background-color: #0d1117;
      color: #e6edf3;
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 30px;
    }

    button {
      background-color: #238636;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 8px;
      font-size: 16px;
      cursor: pointer;
    }

    button:hover {
      background-color: #2ea043;
    }

    canvas {
      margin-top: 40px;
      background-color: #161b22;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(255,255,255,0.1);
    }
  </style>
</head>
<body>
  <h1>Hall Effect Sensor Live Bar Graph</h1>
  <button id="connectBtn">🔌 Connect to Arduino</button>
  <canvas id="sensorChart" width="400" height="200"></canvas>

  <script>
    const connectBtn = document.getElementById("connectBtn");
    const ctx = document.getElementById("sensorChart").getContext("2d");
    let port, reader;

    // Chart.js setup
    const sensorChart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['Sensor Value'],
        datasets: [{
          label: 'Magnetic Field Strength (Analog Value)',
          data: [0],
          backgroundColor: ['#2ea043']
        }]
      },
      options: {
        scales: {
          y: {
            min: 0,
            max: 1023,
            title: {
              display: true,
              text: 'Sensor Output (0 - 1023)',
              color: '#e6edf3'
            },
            ticks: { color: '#e6edf3' }
          },
          x: { ticks: { color: '#e6edf3' } }
        },
        plugins: {
          legend: { labels: { color: '#e6edf3' } }
        }
      }
    });

    async function connectSerial() {
      try {
        port = await navigator.serial.requestPort();
        await port.open({ baudRate: 9600 });

        const decoder = new TextDecoderStream();
        port.readable.pipeTo(decoder.writable);
        const inputStream = decoder.readable;
        reader = inputStream.getReader();

        alert("Connected! Receiving data...");

        while (true) {
          const { value, done } = await reader.read();
          if (done) break;
          if (value) {
            const sensorValue = parseInt(value.trim());
            if (!isNaN(sensorValue)) {
              // Update the chart
              sensorChart.data.datasets[0].data[0] = sensorValue;
              sensorChart.update();
            }
          }
        }
      } catch (err) {
        console.error("Error:", err);
        alert("⚠️ Connection failed or closed.");
      }
    }

    connectBtn.addEventListener("click", connectSerial);
  </script>
</body>
</html>

