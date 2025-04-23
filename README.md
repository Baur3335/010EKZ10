# 010EKZ10
010EKZ10
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Управление поездками</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 600px;
      margin: 20px auto;
      background-color: #f2f2f2;
      padding: 20px;
      border-radius: 10px;
    }
    select, input, textarea, button {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      font-size: 16px;
    }
    .status {
      margin-top: 30px;
      padding: 15px;
      background-color: #e2e8f0;
      border-radius: 5px;
    }
    .status-item {
      margin-bottom: 15px;
      padding: 10px;
      background-color: white;
      border-radius: 5px;
    }
    .badge {
      padding: 3px 8px;
      border-radius: 5px;
      font-size: 14px;
      color: white;
    }
    .badge-green {
      background-color: #28a745;
    }
    .badge-gray {
      background-color: #6c757d;
    }
    @media (max-width: 600px) {
      body {
        padding: 10px;
        font-size: 14px;
      }
      select, input, textarea, button {
        font-size: 14px;
      }
    }
  </style>
</head>
<body>

  <h2>Назначить поездку</h2>

  <label for="driver">Выберите водителя:</label>
  <select id="driver">
    <option value="">-- Выберите --</option>
    <option value="Иванов">Иванов</option>
    <option value="Петров">Петров</option>
    <option value="Сидоров">Сидоров</option>
  </select>

  <label for="vehicle">Выберите машину:</label>
  <select id="vehicle">
    <option value="">-- Выберите --</option>
    <option value="КАМАЗ 123">КАМАЗ 123</option>
    <option value="Газель 456">Газель 456</option>
    <option value="MAN 789">MAN 789</option>
  </select>

  <label for="time">Время в пути (в минутах):</label>
  <input type="number" id="time" placeholder="Например, 30">

  <label for="comment">Комментарий:</label>
  <textarea id="comment" rows="3" placeholder="Ваш комментарий..."></textarea>

  <button onclick="startTrip()">Отправить</button>

  <div class="status">
    <h3>В пути:</h3>
    <div id="activeTrips"></div>
  </div>

  <div class="status">
    <h3>История поездок:</h3>
    <div id="tripHistory"></div>
  </div>

  <button onclick="exportToExcel()">Экспорт в Excel</button>

  <script src="https://cdn.jsdelivr.net/npm/excellentexport@3.4.3/dist/excellentexport.min.js"></script>
  <script>
    let trips = [];

    function saveTrips() {
      localStorage.setItem('trips', JSON.stringify(trips));
    }

    function loadTrips() {
      const saved = localStorage.getItem('trips');
      if (saved) {
        trips = JSON.parse(saved);
        trips.forEach(trip => {
          if (trip.status === 'в пути') {
            renderTrip(trip);
          } else {
            addToHistory(trip);
          }
        });
      }
    }

    function startTrip() {
      const driverSelect = document.getElementById("driver");
      const vehicleSelect = document.getElementById("vehicle");
      const driver = driverSelect.value;
      const vehicle = vehicleSelect.value;
      const time = parseInt(document.getElementById("time").value);
      const comment = document.getElementById("comment").value;

      if (!driver || !vehicle || !time) {
        alert("Пожалуйста, выберите водителя, машину и укажите время.");
        return;
      }

      const tripId = Date.now();
      const now = new Date();
      const createdAt = now.toISOString();

      const trip = {
        id: tripId,
        driver,
        vehicle,
        time,
        comment,
        status: 'в пути',
        createdAt
      };

      trips.push(trip);
      saveTrips();
      renderTrip(trip);

      driverSelect.querySelector(`option[value="${driver}"]`).remove();
      vehicleSelect.querySelector(`option[value="${vehicle}"]`).remove();

      driverSelect.value = "";
      vehicleSelect.value = "";
      document.getElementById("time").value = "";
      document.getElementById("comment").value = "";
    }

    function renderTrip(trip) {
      const activeTrips = document.getElementById("activeTrips");

      const tripDiv = document.createElement("div");
      tripDiv.classList.add("status-item");
      tripDiv.id = "trip-" + trip.id;
      tripDiv.innerHTML = `
        <strong>${trip.driver}</strong> на <strong>${trip.vehicle}</strong><br>
        Комментарий: ${trip.comment}<br>
        Статус: <span class="badge badge-green" id="status-${trip.id}">В пути</span><br>
        Осталось: <span id="timer-${trip.id}">00:00</span><br>
        <div>Создано: ${new Date(trip.createdAt).toLocaleString()}</div>
        <button onclick="cancelTrip(${trip.id})">Отменить</button>
      `;

      activeTrips.appendChild(tripDiv);

      let secondsLeft = trip.time * 60;
      const interval = setInterval(() => {
        const timerEl = document.getElementById("timer-" + trip.id);
        if (!timerEl) return clearInterval(interval);

        let min = Math.floor(secondsLeft / 60);
        let sec = secondsLeft % 60;
        timerEl.innerText = `${String(min).padStart(2, '0')}:${String(sec).padStart(2, '0')}`;

        if (secondsLeft <= 0) {
          clearInterval(interval);
          completeTrip(trip.id);
        }

        secondsLeft--;
      }, 1000);
    }

    function cancelTrip(tripId) {
      const confirmed = confirm("Вы уверены,
