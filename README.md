<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planificador Personal Inteligente</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6; /* bg-gray-100 */
            color: #1f2937; /* text-gray-800 */
        }
        .hidden { display: none; }
        .completed {
            text-decoration: line-through;
            color: #6b7280; /* text-gray-500 */
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .task-item, .auth-form, .card, .alert-message {
            animation: fadeIn 0.4s ease-out;
        }
        .loader {
            border: 4px solid #e5e7eb;
            border-top: 4px solid #4f46e5; /* indigo-600 */
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .category-badge {
            display: inline-block;
            padding: 0.2rem 0.6rem;
            font-size: 0.75rem;
            font-weight: 500;
            border-radius: 9999px;
            text-transform: uppercase;
        }
        .modal {
            animation: fadeIn 0.2s ease-out;
        }
        /* Indicadores de Urgencia */
        .task-item.overdue {
            background-color: #fee2e2; /* red-100 */
            border-left: 5px solid #ef4444; /* red-500 */
        }
        .task-item.today {
            background-color: #ffedd5; /* orange-100 */
            border-left: 5px solid #f97316; /* orange-500 */
        }
    </style>
</head>
<body class="antialiased">

    <div id="loader" class="loader hidden"></div>

    <!-- Modal de Edición de Tarea -->
    <div id="edit-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full card">
            <h3 class="text-2xl font-bold mb-6 text-gray-800">Editar Tarea</h3>
            <form id="edit-task-form" class="space-y-4">
                <div>
                    <label for="edit-task-text" class="block mb-2 text-sm font-medium text-gray-600">Nombre de la Tarea</label>
                    <input type="text" id="edit-task-text" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                     <div>
                        <label for="edit-task-date" class="block mb-2 text-sm font-medium text-gray-600">Fecha</label>
                        <input type="date" id="edit-task-date" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                    </div>
                    <div>
                        <label for="edit-task-time" class="block mb-2 text-sm font-medium text-gray-600">Hora</label>
                        <input type="time" id="edit-task-time" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                     <div>
                        <label for="edit-task-category" class="block mb-2 text-sm font-medium text-gray-600">Categoría</label>
                        <select id="edit-task-category" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="Personal">Personal</option>
                            <option value="Trabajo">Trabajo</option>
                            <option value="Escuela">Escuela</option>
                            <option value="Otro">Otro</option>
                        </select>
                    </div>
                </div>
                <div class="flex justify-end gap-4 pt-4">
                    <button type="button" id="cancel-edit-btn" class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-2 px-6 rounded-lg transition">Cancelar</button>
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg transition">Guardar Cambios</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal de Confirmación para Borrar -->
    <div id="confirm-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-8 rounded-2xl shadow-xl max-w-sm w-full text-center card">
            <h3 class="text-xl font-bold mb-4 text-gray-800">¿Confirmar Eliminación?</h3>
            <p class="text-gray-600 mb-6">Esta acción no se puede deshacer. ¿Estás seguro?</p>
            <div class="flex justify-center gap-4">
                <button id="cancel-delete-btn" class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-2 px-6 rounded-lg transition">Cancelar</button>
                <button id="confirm-delete-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-lg transition">Eliminar</button>
            </div>
        </div>
    </div>

    <!-- Contenedor de Autenticación -->
    <div id="auth-container" class="container mx-auto max-w-md p-4 sm:p-8 mt-10">
        <div id="logout-success-message" class="hidden alert-message bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded-lg relative mb-6 text-center">
            <span class="block sm:inline">¡Sesión cerrada exitosamente!</span>
        </div>
        <div class="auth-form bg-white p-6 sm:p-8 rounded-2xl shadow-xl">
            <h2 class="text-3xl font-bold text-center text-indigo-600 mb-8">Iniciar Sesión</h2>
            <form id="login-form">
                <div class="mb-4">
                    <label for="login-email" class="block mb-2 text-sm font-medium text-gray-600">Email</label>
                    <input type="email" id="login-email" autocomplete="username" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <div class="mb-6">
                    <label for="login-password" class="block mb-2 text-sm font-medium text-gray-600">Contraseña</label>
                    <input type="password" id="login-password" autocomplete="current-password" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg shadow-md transition-transform transform hover:scale-105">Entrar</button>
            </form>
            <p id="auth-error" class="text-red-500 text-center mt-4"></p>
            <p class="text-center text-sm text-gray-500 mt-6">
                Para registrar una cuenta nueva, por favor contacte al administrador.
            </p>
        </div>
    </div>

    <!-- Contenedor del Planificador -->
    <div id="planner-container" class="hidden container mx-auto p-4 md:p-8 max-w-5xl">
         <header class="flex flex-col sm:flex-row justify-between items-center mb-8 gap-4">
            <div>
                <h1 class="text-3xl md:text-5xl font-bold text-indigo-600">Mi Planificador</h1>
                <p id="user-email" class="text-gray-500 mt-2 text-center sm:text-left"></p>
            </div>
            <button id="logout-btn" class="w-full sm:w-auto bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg shadow-md transition">Cerrar Sesión</button>
        </header>

        <div id="task-success-message" class="hidden alert-message bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded-lg relative mb-6 text-center">
             <span class="block sm:inline">¡Tarea añadida exitosamente!</span>
        </div>

        <!-- Dashboard -->
        <div id="dashboard-container" class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="card bg-white p-6 rounded-2xl shadow-xl">
                <h3 class="text-lg font-semibold text-gray-500 text-center">Próxima Tarea</h3>
                <div id="dashboard-next-task" class="text-center mt-2">
                    <p class="text-gray-400">¡Ninguna tarea pendiente!</p>
                </div>
            </div>
            <div class="card bg-white p-6 rounded-2xl shadow-xl text-center">
                <h3 class="text-lg font-semibold text-gray-500">Total Pendientes</h3>
                <p id="dashboard-pending" class="text-4xl font-bold text-indigo-600">0</p>
            </div>
            <div class="card bg-white p-6 rounded-2xl shadow-xl">
                <h3 class="text-lg font-semibold text-gray-500 text-center">Vencidas (<span id="dashboard-overdue-count">0</span>)</h3>
                <div id="dashboard-overdue" class="text-center mt-2">
                     <p class="text-gray-400">¡Ninguna tarea vencida!</p>
                </div>
            </div>
        </div>

        <div class="card bg-white p-6 rounded-2xl shadow-xl mb-8">
            <h2 class="text-2xl font-semibold mb-4 text-gray-800">Añadir Nueva Tarea</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <input type="text" id="taskInput" placeholder="¿Qué necesitas hacer?" class="md:col-span-2 w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <input type="date" id="dateInput" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <input type="time" id="timeInput" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
            </div>
             <select id="categoryInput" class="mt-4 w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <option value="Personal">Personal</option>
                <option value="Trabajo">Trabajo</option>
                <option value="Escuela">Escuela</option>
                <option value="Otro">Otro</option>
            </select>
             <button id="addTaskBtn" class="mt-4 w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg shadow-md transition">Añadir Tarea</button>
        </div>
        
        <div id="task-list-container" class="space-y-8"></div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, doc, updateDoc, deleteDoc } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-firestore.js";

        const firebaseConfig = {
          apiKey: "AIzaSyA_LRcHCkClvlHeqDPTSKfGa5gY2uiuZ5E",
          authDomain: "mi-planificador-privado.firebaseapp.com",
          projectId: "mi-planificador-privado",
          storageBucket: "mi-planificador-privado.firebasestorage.app",
          messagingSenderId: "792700686473",
          appId: "1:792700686473:web:8d1f41076f12fa0103f658",
          measurementId: "G-H5V6YERMVP"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        // --- DOM Elements ---
        const loader = document.getElementById('loader');
        const authContainer = document.getElementById('auth-container');
        const plannerContainer = document.getElementById('planner-container');
        const userEmailDisplay = document.getElementById('user-email');
        const authError = document.getElementById('auth-error');
        const loginForm = document.getElementById('login-form');
        const logoutBtn = document.getElementById('logout-btn');
        const logoutSuccessMessage = document.getElementById('logout-success-message');
        const taskSuccessMessage = document.getElementById('task-success-message');
        // Modals
        const confirmModal = document.getElementById('confirm-modal');
        const cancelDeleteBtn = document.getElementById('cancel-delete-btn');
        const confirmDeleteBtn = document.getElementById('confirm-delete-btn');
        const editModal = document.getElementById('edit-modal');
        const editTaskForm = document.getElementById('edit-task-form');
        const cancelEditBtn = document.getElementById('cancel-edit-btn');
        const editTaskText = document.getElementById('edit-task-text');
        const editTaskDate = document.getElementById('edit-task-date');
        const editTaskTime = document.getElementById('edit-task-time');
        const editTaskCategory = document.getElementById('edit-task-category');
        // Planner
        const taskInput = document.getElementById('taskInput');
        const dateInput = document.getElementById('dateInput');
        const timeInput = document.getElementById('timeInput');
        const categoryInput = document.getElementById('categoryInput');
        const addTaskBtn = document.getElementById('addTaskBtn');
        const taskListContainer = document.getElementById('task-list-container');

        let allTasks = [];
        let unsubscribeTasks;
        let taskToDeleteId = null;
        let taskToEditId = null;
        let dashboardInterval = null;

        // --- Auth Logic ---
        onAuthStateChanged(auth, user => {
            loader.classList.remove('hidden');
            authError.textContent = '';
            if (user) {
                authContainer.classList.add('hidden');
                plannerContainer.classList.remove('hidden');
                userEmailDisplay.textContent = `Sesión iniciada como: ${user.email}`;
                loadTasks(user.uid);
                if (dashboardInterval) clearInterval(dashboardInterval);
                dashboardInterval = setInterval(() => {
                    renderTasks(allTasks); // Re-render tasks to update countdowns
                    updateDashboard(allTasks);
                }, 60000); // Update every minute
            } else {
                plannerContainer.classList.add('hidden');
                authContainer.classList.remove('hidden');
                clearTasksUI();
                if(unsubscribeTasks) unsubscribeTasks();
                if (dashboardInterval) clearInterval(dashboardInterval);
            }
            loader.classList.add('hidden');
        });

        loginForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            logoutSuccessMessage.classList.add('hidden');
            const email = document.getElementById('login-email').value;
            const password = document.getElementById('login-password').value;
            authError.textContent = '';
            loader.classList.remove('hidden');
            try {
                await signInWithEmailAndPassword(auth, email, password);
            } catch (error) {
                switch (error.code) {
                    case 'auth/user-not-found': case 'auth/wrong-password': case 'auth/invalid-credential':
                        authError.textContent = 'El correo o la contraseña son incorrectos.'; break;
                    case 'auth/user-disabled':
                        authError.textContent = 'Su cuenta ha sido deshabilitada.'; break;
                    default:
                        authError.textContent = 'Ocurrió un error inesperado.'; break;
                }
            } finally {
                loader.classList.add('hidden');
            }
        });

        logoutBtn.addEventListener('click', () => {
            signOut(auth).then(() => {
                document.getElementById('login-email').value = '';
                document.getElementById('login-password').value = '';
                logoutSuccessMessage.classList.remove('hidden');
                setTimeout(() => { logoutSuccessMessage.classList.add('hidden'); }, 3000);
            });
        });

        // --- Modal Logic ---
        function showConfirmModal(taskId) {
            taskToDeleteId = taskId;
            confirmModal.classList.remove('hidden');
        }
        function hideConfirmModal() {
            taskToDeleteId = null;
            confirmModal.classList.add('hidden');
        }
        cancelDeleteBtn.addEventListener('click', hideConfirmModal);
        confirmDeleteBtn.addEventListener('click', async () => {
            if (taskToDeleteId) {
                await deleteTask(taskToDeleteId);
                hideConfirmModal();
            }
        });

        function showEditModal(taskId) {
            taskToEditId = taskId;
            const task = allTasks.find(t => t.id === taskId);
            if(task) {
                editTaskText.value = task.text;
                editTaskDate.value = task.date;
                editTaskTime.value = task.time || '';
                editTaskCategory.value = task.category;
                editModal.classList.remove('hidden');
            }
        }
        function hideEditModal() {
            taskToEditId = null;
            editModal.classList.add('hidden');
        }
        cancelEditBtn.addEventListener('click', hideEditModal);
        editTaskForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            if(taskToEditId) {
                const updatedData = { text: editTaskText.value, date: editTaskDate.value, time: editTaskTime.value, category: editTaskCategory.value };
                await updateTask(taskToEditId, updatedData);
                hideEditModal();
            }
        });

        // --- Planner Logic ---
        function setSmartDefaults() {
            dateInput.value = new Date().toISOString().split('T')[0];
            const now = new Date();
            const nextHour = (now.getHours() + 1) % 24;
            timeInput.value = `${String(nextHour).padStart(2, '0')}:00`;
        }
        
        async function loadTasks(userId) {
            const tasksCollection = collection(db, "users", userId, "tasks");
            const q = query(tasksCollection);
            if (unsubscribeTasks) unsubscribeTasks();
            
            unsubscribeTasks = onSnapshot(q, (snapshot) => {
                allTasks = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderTasks(allTasks);
                updateDashboard(allTasks);
            }, (error) => { console.error("Error fetching tasks:", error); });
        }
        
        function renderTasks(tasks) {
            taskListContainer.innerHTML = '';
            const groupedTasks = tasks.reduce((acc, task) => {
                const category = task.category || 'Otro';
                if (!acc[category]) acc[category] = [];
                acc[category].push(task);
                return acc;
            }, {});

            const categories = ['Personal', 'Trabajo', 'Escuela', 'Otro'];
            categories.forEach(category => {
                if (groupedTasks[category] && groupedTasks[category].length > 0) {
                    const tasksInCategory = groupedTasks[category];
                    tasksInCategory.sort((a, b) => new Date(`${a.date}T${a.time || '00:00'}`) - new Date(`${b.date}T${b.time || '00:00'}`));

                    const categorySection = document.createElement('div');
                    categorySection.className = 'card bg-white p-6 rounded-2xl shadow-xl';
                    
                    let sectionHTML = `<h3 class="text-2xl font-semibold mb-4 border-b-2 border-gray-200 pb-2 text-gray-700">${category}</h3>`;
                    const upcomingList = tasksInCategory.filter(t => !t.completed).map(createTaskElement).join('');
                    const completedList = tasksInCategory.filter(t => t.completed).map(createTaskElement).join('');

                    if (upcomingList) sectionHTML += `<ul class="space-y-3">${upcomingList}</ul>`;
                    if (completedList) sectionHTML += `<h4 class="text-lg font-semibold mt-6 mb-3 text-gray-500">Completadas</h4><ul class="space-y-3">${completedList}</ul>`;
                    
                    categorySection.innerHTML = sectionHTML;
                    taskListContainer.appendChild(categorySection);
                }
            });

            taskListContainer.querySelectorAll('.task-checkbox').forEach(box => box.addEventListener('change', (e) => toggleTaskStatus(e.target.closest('li').dataset.id, e.target.checked)));
            taskListContainer.querySelectorAll('.edit-btn').forEach(btn => btn.addEventListener('click', (e) => showEditModal(e.target.closest('li').dataset.id)));
            taskListContainer.querySelectorAll('.delete-btn').forEach(btn => btn.addEventListener('click', (e) => showConfirmModal(e.target.closest('li').dataset.id)));
        }
        
        function formatTimeDifference(targetDate) {
            const now = new Date();
            const diff = targetDate - now;
            const isPast = diff < 0;
            const absDiff = Math.abs(diff);

            const days = Math.floor(absDiff / (1000 * 60 * 60 * 24));
            const hours = Math.floor((absDiff / (1000 * 60 * 60)) % 24);
            const minutes = Math.floor((absDiff / (1000 * 60)) % 60);

            const prefix = isPast ? 'Hace' : 'En';
            
            if (days > 1) return `${prefix} ${days} días`;
            if (days === 1) return isPast ? 'Hace 1 día' : 'Mañana';
            if (hours > 0) return `${prefix} ${hours}h y ${minutes}m`;
            if (minutes > 1) return `${prefix} ${minutes} minutos`;
            return isPast ? 'Venció hace un momento' : 'Vence en menos de un minuto';
        }

        function updateDashboard(tasks) {
            const pendingTasks = tasks.filter(t => !t.completed);
            const now = new Date();

            const upcomingTasks = pendingTasks
                .map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59'}`) }))
                .filter(t => t.dateTime > now)
                .sort((a, b) => a.dateTime - b.dateTime);

            const nextTaskDisplay = document.getElementById('dashboard-next-task');
            if (upcomingTasks.length > 0) {
                const nextTask = upcomingTasks[0];
                nextTaskDisplay.innerHTML = `<p class="font-bold text-lg text-indigo-600 break-words">${nextTask.text}</p><p class="text-gray-500 text-sm mt-1">${formatTimeDifference(nextTask.dateTime)}</p>`;
            } else {
                nextTaskDisplay.innerHTML = `<p class="text-gray-400">¡Ninguna tarea pendiente!</p>`;
            }
            
            document.getElementById('dashboard-pending').textContent = pendingTasks.length;

            const overdueTasks = pendingTasks
                .map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59'}`) }))
                .filter(t => t.dateTime < now)
                .sort((a, b) => a.dateTime - b.dateTime);
            
            document.getElementById('dashboard-overdue-count').textContent = overdueTasks.length;
            const overdueDisplay = document.getElementById('dashboard-overdue');
            if(overdueTasks.length > 0) {
                const oldestOverdue = overdueTasks[0];
                overdueDisplay.innerHTML = `<p class="font-bold text-lg text-red-500 break-words">${oldestOverdue.text}</p><p class="text-gray-500 text-sm mt-1">La más antigua venció ${formatTimeDifference(oldestOverdue.dateTime).toLowerCase()}</p>`;
            } else {
                overdueDisplay.innerHTML = `<p class="text-gray-400">¡Ninguna tarea vencida!</p>`;
            }
        }
        
        function createTaskElement(task) {
            const categoryColors = {'Personal': 'bg-blue-100 text-blue-800','Trabajo': 'bg-purple-100 text-purple-800','Escuela': 'bg-green-100 text-green-800','Otro': 'bg-gray-200 text-gray-800'};
            
            const taskDateTime = new Date(`${task.date}T${task.time || '23:59'}`);
            let formattedTime = '';
            if (task.time) {
                const [hours, minutes] = task.time.split(':');
                const h = parseInt(hours, 10); const ampm = h >= 12 ? 'PM' : 'AM';
                const formattedHour = h % 12 || 12; formattedTime = ` a las ${formattedHour}:${minutes} ${ampm}`;
            }

            const now = new Date();
            const today = new Date(); today.setHours(0, 0, 0, 0);
            const taskDateOnly = new Date(task.date + 'T00:00:00');
            
            let urgencyClass = '';
            let timeDiffHtml = '';

            if (!task.completed) {
                if (taskDateTime < now) urgencyClass = 'overdue';
                else if (taskDateOnly.getTime() === today.getTime()) urgencyClass = 'today';
                
                const timeDiffText = formatTimeDifference(taskDateTime);
                let timeDiffColor = 'text-gray-500';
                if(urgencyClass === 'overdue') timeDiffColor = 'text-red-600';
                else if (urgencyClass === 'today') timeDiffColor = 'text-orange-600';
                timeDiffHtml = `<span class="font-semibold ${timeDiffColor}">(${timeDiffText})</span>`;
            }

            return `<li class="task-item ${urgencyClass} bg-white p-4 rounded-lg flex items-center justify-between shadow-sm" data-id="${task.id}"><div class="flex items-center gap-4 flex-grow min-w-0"><input type="checkbox" ${task.completed ? 'checked' : ''} class="task-checkbox h-6 w-6 rounded border-gray-300 text-indigo-600 cursor-pointer focus:ring-indigo-500 flex-shrink-0"><div><span class="task-text font-medium break-words ${task.completed ? 'completed' : ''}">${task.text}</span><div class="text-sm text-gray-500 flex items-center flex-wrap gap-x-2 gap-y-1 mt-1"><span>${taskDateOnly.toLocaleDateString('es-ES', { weekday: 'long', day: 'numeric', month: 'long' })}${formattedTime}</span><span class="category-badge ${categoryColors[task.category] || categoryColors['Otro']}">${task.category}</span>${timeDiffHtml}</div></div></div><div class="flex items-center flex-shrink-0 ml-2"><button class="edit-btn text-gray-400 hover:text-indigo-500 transition p-1"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path></svg></button><button class="delete-btn text-gray-400 hover:text-red-500 transition p-1"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg></button></div></li>`;
        }

        function showTaskSuccessMessage() {
            taskSuccessMessage.classList.remove('hidden');
            setTimeout(() => { taskSuccessMessage.classList.add('hidden'); }, 3000);
        }

        // --- CRUD Functions ---
        async function handleAddTask() {
            const text = taskInput.value.trim(); const date = dateInput.value;
            const time = timeInput.value; const category = categoryInput.value;
            const user = auth.currentUser;
            if (text && date && user) {
                await addDoc(collection(db, "users", user.uid, "tasks"), { text, date, time, category, completed: false });
                taskInput.value = '';
                taskInput.focus();
                setSmartDefaults();
                showTaskSuccessMessage();
            }
        }
        async function toggleTaskStatus(taskId, isCompleted) {
            const user = auth.currentUser;
            if (user) await updateDoc(doc(db, "users", user.uid, "tasks", taskId), { completed: isCompleted });
        }
        async function updateTask(taskId, updatedData) {
            const user = auth.currentUser;
            if(user) await updateDoc(doc(db, "users", user.uid, "tasks", taskId), updatedData);
        }
        async function deleteTask(taskId) {
            const user = auth.currentUser;
            if (user) await deleteDoc(doc(db, "users", user.uid, "tasks", taskId));
        }
        
        function clearTasksUI() {
             taskListContainer.innerHTML = '';
             document.getElementById('dashboard-pending').textContent = 0;
             const nextTaskDisplay = document.getElementById('dashboard-next-task');
             nextTaskDisplay.innerHTML = `<p class="text-gray-400">¡Ninguna tarea pendiente!</p>`;
             const overdueDisplay = document.getElementById('dashboard-overdue');
             overdueDisplay.innerHTML = `<p class="text-gray-400">¡Ninguna tarea vencida!</p>`;
             document.getElementById('dashboard-overdue-count').textContent = 0;
        }

        addTaskBtn.addEventListener('click', handleAddTask);
        taskInput.addEventListener('keypress', (e) => { if(e.key === 'Enter') handleAddTask(); });

        setSmartDefaults();
    </script>
</body>
</html>

