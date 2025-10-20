<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TeacherSchedule Pro</title>
    <style>
        /* Basic Eye-Catching CSS */
        body { font-family: Arial, sans-serif; background: linear-gradient(to right, #4CAF50, #2196F3); color: white; margin: 0; padding: 20px; }
        .container { max-width: 800px; margin: auto; background: rgba(255, 255, 255, 0.9); padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.3); }
        h1, h2 { text-align: center; color: #333; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { padding: 10px; text-align: left; border-bottom: 1px solid #ddd; }
        th { background: #f2f2f2; color: #333; }
        .upcoming { background: #ffeb3b; color: #333; } /* Yellow for upcoming */
        .current { background: #4CAF50; color: white; } /* Green for current */
        button { background: #2196F3; color: white; border: none; padding: 10px; cursor: pointer; border-radius: 5px; }
        button:hover { background: #0b7dda; }
        .notification { display: none; position: fixed; top: 10px; right: 10px; background: #ff5722; color: white; padding: 10px; border-radius: 5px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>TeacherSchedule Pro</h1>
        
        <!-- Admin Section -->
        <h2>Admin: Add/Update Schedule</h2>
        <form id="scheduleForm">
            <label>Teacher Name: <input type="text" id="teacher" required></label><br><br>
            <label>Grade: <input type="text" id="grade" required></label><br><br>
            <label>Subject: <input type="text" id="subject" required></label><br><br>
            <label>Period (e.g., 1): <input type="number" id="period" required></label><br><br>
            <label>Time (e.g., 9:00 AM): <input type="text" id="time" required></label><br><br>
            <label>Classroom: <input type="text" id="room" required></label><br><br>
            <button type="submit">Add Schedule</button>
        </form>
        
        <!-- Teacher Section -->
        <h2>Teacher: View Your Schedule</h2>
        <label>Select Your Name: 
            <select id="teacherSelect">
                <option value="">--Choose--</option>
                <!-- Options populated by JS -->
            </select>
        </label>
        <button onclick="loadSchedule()">Load Schedule</button>
        <table id="scheduleTable">
            <thead>
                <tr>
                    <th>Period</th>
                    <th>Grade</th>
                    <th>Subject</th>
                    <th>Time</th>
                    <th>Room</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody id="scheduleBody">
                <!-- Schedule rows populated by JS -->
            </tbody>
        </table>
        
        <!-- Notification Area -->
        <div id="notification" class="notification">Notification here</div>
    </div>

    <script>
        // JS for Functionality
        let schedules = JSON.parse(localStorage.getItem('schedules')) || [];
        let currentTeacher = '';

        // Populate teacher dropdown
        function updateTeacherList() {
            const select = document.getElementById('teacherSelect');
            select.innerHTML = '<option value="">--Choose--</option>';
            const teachers = [...new Set(schedules.map(s => s.teacher))];
            teachers.forEach(t => {
                const option = document.createElement('option');
                option.value = t;
                option.textContent = t;
                select.appendChild(option);
            });
        }
        updateTeacherList();

        // Add schedule
        document.getElementById('scheduleForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const schedule = {
                teacher: document.getElementById('teacher').value,
                grade: document.getElementById('grade').value,
                subject: document.getElementById('subject').value,
                period: document.getElementById('period').value,
                time: document.getElementById('time').value,
                room: document.getElementById('room').value
            };
            schedules.push(schedule);
            localStorage.setItem('schedules', JSON.stringify(schedules));
            updateTeacherList();
            alert('Schedule added!');
            this.reset();
        });

        // Load schedule for teacher
        function loadSchedule() {
            currentTeacher = document.getElementById('teacherSelect').value;
            if (!currentTeacher) return alert('Select a teacher!');
            const teacherSchedules = schedules.filter(s => s.teacher === currentTeacher);
            const tbody = document.getElementById('scheduleBody');
            tbody.innerHTML = '';
            teacherSchedules.forEach(s => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${s.period}</td>
                    <td>${s.grade}</td>
                    <td>${s.subject}</td>
                    <td>${s.time}</td>
                    <td>${s.room}</td>
                    <td id="status-${s.period}">Upcoming</td>
                `;
                tbody.appendChild(row);
            });
            checkNotifications(); // Start checking
        }

        // Notification logic
        function checkNotifications() {
            if (!currentTeacher) return;
            const now = new Date();
            const currentTime = now.getHours() * 60 + now.getMinutes(); // Minutes since midnight
            schedules.filter(s => s.teacher === currentTeacher).forEach(s => {
                const [hour, minute] = s.time.split(':');
                const periodTime = parseInt(hour) * 60 + parseInt(minute);
                const statusCell = document.getElementById(`status-${s.period}`);
                if (currentTime >= periodTime - 10 && currentTime < periodTime) { // 10 min before
                    statusCell.textContent = 'Upcoming';
                    statusCell.parentElement.classList.add('upcoming');
                    showNotification(`Next class: ${s.grade} ${s.subject} in ${s.room} at ${s.time}`);
                } else if (currentTime >= periodTime && currentTime < periodTime + 60) { // During class
                    statusCell.textContent = 'In Progress';
                    statusCell.parentElement.classList.add('current');
                } else {
                    statusCell.textContent = 'Upcoming';
                    statusCell.parentElement.classList.remove('upcoming', 'current');
                }
            });
        }

        function showNotification(message) {
            const notif = document.getElementById('notification');
            notif.textContent = message;
            notif.style.display = 'block';
            setTimeout(() => notif.style.display = 'none', 5000); // Hide after 5s
            if ('Notification' in window && Notification.permission === 'granted') {
                new Notification('TeacherSchedule Pro', { body: message });
            }
        }

        // Request notification permission
        if ('Notification' in window) {
            Notification.requestPermission();
        }

        // Check every minute
        setInterval(checkNotifications, 60000);
    </script>
</body>
</html>
