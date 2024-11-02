<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CRM - Topshiriq Bajarish</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.3/xlsx.full.min.js"></script>
    <style>
        /* Dizayn va formatlash */
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f6f9;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
        }
        h1 { color: #4CAF50; }
        form {
            background-color: #fff;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            width: 100%;
            max-width: 500px;
        }
        input, select, button {
            width: 100%;
            padding: 8px;
            margin: 6px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
        }
        button {
            background-color: #4CAF50;
            color: #fff;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
        table {
            width: 100%;
            max-width: 600px;
            background-color: #fff;
            border-collapse: collapse;
            margin-top: 15px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            border-radius: 8px;
            overflow: hidden;
        }
        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th { background-color: #4CAF50; color: white; }
        .overdue { background-color: #f8d7da; color: #721c24; }
        .completed { background-color: #d4edda; color: #155724; }
        .pending { background-color: #fff3cd; color: #856404; }
        .user-info {
            font-size: 18px;
            margin-bottom: 20px;
            color: #333;
        }
    </style>
</head>
<body>
    <h1>Topshiriqlar Bajarish</h1>

    <form id="login-form" style="display: none;">
        <input type="text" id="username" placeholder="Login" required>
        <input type="password" id="password" placeholder="Parol" required>
        <button type="submit">Kirish</button>
    </form>

    <div id="app" style="display: none;">
        <div class="user-info" id="user-info"></div>
        <form id="task-form">
            <input type="text" id="task-name" placeholder="Topshiriq nomi" required>
            <input type="datetime-local" id="task-deadline" required>
            <select id="task-assignee" required>
                <option value="">Foydalanuvchini tanlang</option>
            </select>
            <button type="submit">Topshiriq Qo'shish</button>
        </form>

        <button onclick="downloadExcel()">Excelga yuklab olish</button>

        <table id="task-table">
            <tr>
                <th>Tartib raqami</th>
                <th>Topshiriq</th>
                <th>Muddati</th>
                <th>Beruvchi</th>
                <th>Oluvchi</th>
                <th>Holat</th>
                <th>Bajarilgan vaqti</th>
                <th>Amallar</th>
            </tr>
        </table>

        <button id="logout-button" onclick="logout()">Chiqish</button>
    </div>

    <script>
        const users = [
            { username: 'Bunyod', password: 'admin123', active: true, role: 'Admin', tasks: [] },
            { username: 'Oxun', password: 'admin123', active: true, role: 'Admin', tasks: [] },
            { username: 'Zuxriddin', password: 'zuxriddin123', active: true, role: 'User', tasks: [] },
            { username: 'Faridun', password: 'faridun123', active: true, role: 'User', tasks: [] },
            { username: 'Aktiv', password: 'aktiv123', active: true, role: 'User', tasks: [] },
            { username: 'Aktiv 1', password: 'aktiv1', active: true, role: 'User', tasks: [] }
        ];

        let currentUser = null;
        let tasks = [];

        document.getElementById('login-form').style.display = 'block';
        
        document.getElementById('login-form').addEventListener('submit', function(event) {
            event.preventDefault();
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
            const user = users.find(u => u.username === username && u.password === password);
            if (user) {
                currentUser = user;
                document.getElementById('login-form').style.display = 'none';
                document.getElementById('app').style.display = 'block';
                document.getElementById('user-info').textContent = `Foydalanuvchi: ${currentUser.username} (${currentUser.role})`;
                loadUsers();
                renderTasks();
            } else {
                alert('Noto\'g\'ri login yoki parol!');
            }
        });

        function loadUsers() {
            const assigneeSelect = document.getElementById('task-assignee');
            assigneeSelect.innerHTML = '<option value="">Foydalanuvchini tanlang</option>';
            users.forEach(user => {
                if (user.active) {
                    const option = document.createElement('option');
                    option.value = user.username;
                    option.textContent = user.username;
                    assigneeSelect.appendChild(option);
                }
            });
        }

        function addTask(event) {
            event.preventDefault();
            const taskName = document.getElementById('task-name').value;
            const taskDeadline = document.getElementById('task-deadline').value;
            const taskAssignee = document.getElementById('task-assignee').value;
            const taskGiver = currentUser.username;

            const newTask = { 
                name: taskName, 
                deadline: new Date(taskDeadline), 
                assignee: taskAssignee, 
                giver: taskGiver, 
                completed: false, 
                completedAt: null 
            };
            tasks.push(newTask);
            renderTasks();

            document.getElementById('task-name').value = '';
            document.getElementById('task-deadline').value = '';
            document.getElementById('task-assignee').value = '';
        }

        document.getElementById('task-form').addEventListener('submit', addTask);

        function renderTasks() {
            const table = document.getElementById('task-table');
            table.innerHTML = '<tr><th>Tartib raqami</th><th>Topshiriq</th><th>Muddati</th><th>Beruvchi</th><th>Oluvchi</th><th>Holat</th><th>Bajarilgan vaqti</th><th>Amallar</th></tr>';

            tasks.forEach((task, index) => {
                if (currentUser.role === 'Admin' || task.assignee === currentUser.username) {
                    const newRow = table.insertRow(-1);
                    newRow.classList.toggle('completed', task.completed);
                    newRow.classList.toggle('overdue', new Date() > task.deadline && !task.completed);
                    newRow.classList.toggle('pending', !task.completed && new Date() <= task.deadline);

                    newRow.insertCell(0).textContent = index + 1;
                    newRow.insertCell(1).textContent = task.name;
                    newRow.insertCell(2).textContent = task.deadline.toLocaleString();
                    newRow.insertCell(3).textContent = task.giver;
                    newRow.insertCell(4).textContent = task.assignee;
                    newRow.insertCell(5).textContent = task.completed ? 'Bajarildi' : 'Bajarilmadi';
                    newRow.insertCell(6).textContent = task.completedAt ? task.completedAt.toLocaleString() : '';
                    const actionCell = newRow.insertCell(7);
                    actionCell.innerHTML = `<button onclick="toggleTaskStatus(${index})">${task.completed ? 'Bekor qilish' : 'Tasdiqlash'}</button>`;
                }
            });
        }

        function toggleTaskStatus(index) {
            tasks[index].completed = !tasks[index].completed;
            tasks[index].completedAt = tasks[index].completed ? new Date() : null;
            renderTasks();
        }

        function downloadExcel() {
            const wb = XLSX.utils.book_new();
            const ws_data = [['Topshiriq', 'Muddati', 'Beruvchi', 'Oluvchi', 'Holat', 'Bajarilgan vaqti']];
            tasks.forEach(task => {
                ws_data.push([
                    task.name, 
                    task.deadline.toLocaleString(), 
                    task.giver, 
                    task.assignee, 
                    task.completed ? 'Bajarildi' : 'Bajarilmadi', 
                    task.completedAt ? task.completedAt.toLocaleString() : ''
                ]);
            });

            const ws = XLSX.utils.aoa_to_sheet(ws_data);
            XLSX.utils.book_append_sheet(wb, ws, 'Topshiriqlar');
            XLSX.writeFile(wb, 'Topshiriqlar.xlsx');
        }

        function logout() {
            currentUser = null;
            document.getElementById('login-form').style.display = 'block';
            document.getElementById('app').style.display = 'none';
            document.getElementById('task-table').innerHTML = '';
        }

        renderTasks();
    </script>
</body>
</html>
