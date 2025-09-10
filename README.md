<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planificador Personal Avanzado</title>
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
        .task-item, .auth-form, .card, #logout-success-message {
            animation: fadeIn 0.4s ease-out;
        }
        /* Estilos para el spinner de carga */
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
            margin-left: -20px;
            margin-top: -20px;
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
        /* Estilos para el modal */
        #confirm-modal {
            animation: fadeIn 0.2s ease-out;
        }
    </style>
</head>
<body class="antialiased">

    <div id="loader" class="loader hidden"></div>

    <!-- Modal de Confirmación para Borrar -->
    <div id="confirm-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4">
        <div class="bg-white p-8 rounded-2xl shadow-xl max-w-sm w-full text-center card">
            <h3 class="text-xl font-bold mb-4 text-gray-800">¿Confirmar Eliminación?</h3>
            <p class="text-gray-600 mb-6">Esta acción no se puede deshacer. ¿Estás seguro de que quieres eliminar esta tarea?</p>
            <div class="flex justify-center gap-4">
                <button id="cancel-delete-btn" class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-2 px-6 rounded-lg transition">Cancelar</button>
                <button id="confirm-delete-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-lg transition">Eliminar</button>
            </div>
        </div>
    </div>

    <!-- Contenedor de Autenticación -->
    <div id="auth-container" class="container mx-auto max-w-md p-8 mt-10">
        <div id="logout-success-message" class="hidden bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded-lg relative mb-6 text-center">
            <span class="block sm:inline">¡Sesión cerrada exitosamente!</span>
        </div>
        <div class="auth-form bg-white p-8 rounded-2xl shadow-xl">
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
         <header class="flex flex-wrap justify-between items-center mb-8 gap-4">
            <div>
                <h1 class="text-4xl md:text-5xl font-bold text-indigo-600">Mi Planificador</h1>
                <p id="user-email" class="text-gray-500 mt-2"></p>
            </div>
            <button id="logout-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg shadow-md transition">Cerrar Sesión</button>
        </header>

        <!-- Dashboard -->
        <div id="dashboard-container" class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="card bg-white p-6 rounded-2xl shadow-xl text-center">
                <h3 class="text-lg font-semibold text-gray-500">Tareas para Hoy</h3>
                <p id="dashboard-today" class="text-4xl font-bold text-indigo-600">0</p>
            </div>
            <div class="card bg-white p-6 rounded-2xl shadow-xl text-center">
                <h3 class="text-lg font-semibold text-gray-500">Total Pendientes</h3>
                <p id="dashboard-pending" class="text-4xl font-bold text-indigo-600">0</p>
            </div>
            <div class="card bg-white p-6 rounded-2xl shadow-xl text-center">
                <h3 class="text-lg font-semibold text-gray-500">Vencidas</h3>
                <p id="dashboard-overdue" class="text-4xl font-bold text-red-500">0</p>
            </div>
        </div>

        <div class="card bg-white p-6 rounded-2xl shadow-xl mb-8">
            <h2 class="text-2xl font-semibold mb-4 text-gray-800">Añadir Nueva Tarea</h2>
            <div class="grid grid-cols-1 md:grid-cols-5 gap-4">
                <input type="text" id="taskInput" placeholder="¿Qué necesitas hacer?" class="md:col-span-2 w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <input type="date" id="dateInput" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <input type="time" id="timeInput" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <select id="categoryInput" class="w-full bg-gray-100 text-gray-800 rounded-lg px-4 py-3 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    <option value="Personal">Personal</option>
                    <option value="Trabajo">Trabajo</option>
                    <option value="Escuela">Escuela</option>
                    <option value="Otro">Otro</option>
                </select>
            </div>
             <button id="addTaskBtn" class="mt-4 w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg shadow-md transition">Añadir Tarea</button>
        </div>
        
        <div id="task-list-container" class="space-y-8">
            <!-- Las categorías y tareas se insertarán aquí dinámicamente -->
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-app.js";
        import { 
            getAuth,
            signInWithEmailAndPassword, 
            onAuthStateChanged,
            signOut
        } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-auth.js";
        import { 
            getFirestore, 
            collection, 
            addDoc, 
            query, 
            onSnapshot,
            doc,
            updateDoc,
            deleteDoc
        } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-firestore.js";

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

        // --- Elementos del DOM ---
        const loader = document.getElementById('loader');
        const authContainer = document.getElementById('auth-container');
        const plannerContainer = document.getElementById('planner-container');
        const userEmailDisplay = document.getElementById('user-email');
        const authError = document.getElementById('auth-error');
        const loginForm = document.getElementById('login-form');
        const logoutBtn = document.getElementById('logout-btn');
        const logoutSuccessMessage = document.getElementById('logout-success-message');
        // Modal
        const confirmModal = document.getElementById('confirm-modal');
        const cancelDeleteBtn = document.getElementById('cancel-delete-btn');
        const confirmDeleteBtn = document.getElementById('confirm-delete-btn');
        // Planificador
        const taskInput = document.getElementById('taskInput');
        const dateInput = document.getElementById('dateInput');
        const timeInput = document.getElementById('timeInput');
        const categoryInput = document.getElementById('categoryInput');
        const addTaskBtn = document.getElementById('addTaskBtn');
        const taskListContainer = document.getElementById('task-list-container');

        let unsubscribeTasks;
        let taskToDeleteId = null;

        // --- Lógica de Autenticación ---
        onAuthStateChanged(auth, user => {
            loader.classList.remove('hidden');
            authError.textContent = '';
            if (user) {
                authContainer.classList.add('hidden');
                plannerContainer.classList.remove('hidden');
                userEmailDisplay.textContent = `Sesión iniciada como: ${user.email}`;
                loadTasks(user.uid);
            } else {
                plannerContainer.classList.add('hidden');
                authContainer.classList.remove('hidden');
                clearTasksUI();
                if(unsubscribeTasks) unsubscribeTasks();
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
                    case 'auth/user-not-found':
                    case 'auth/wrong-password':
                    case 'auth/invalid-credential':
                        authError.textContent = 'El correo electrónico o la contraseña son incorrectos.';
                        break;
                    case 'auth/user-disabled':
                        authError.textContent = 'Su cuenta ha sido deshabilitada.';
                        break;
                    default:
                        authError.textContent = 'Ocurrió un error inesperado. Por favor, inténtelo de nuevo.';
                        break;
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
                setTimeout(() => {
                    logoutSuccessMessage.classList.add('hidden');
                }, 3000);
            });
        });

        // --- Lógica del Modal de Confirmación ---
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
                const user = auth.currentUser;
                if (user) {
                    await deleteDoc(doc(db, "users", user.uid, "tasks", taskToDeleteId));
                }
                hideConfirmModal();
            }
        });

        // --- Lógica del Planificador ---
        dateInput.value = new Date().toISOString().split('T')[0];
        
        async function loadTasks(userId) {
            const tasksCollection = collection(db, "users", userId, "tasks");
            const q = query(tasksCollection);
            if (unsubscribeTasks) unsubscribeTasks();
            
            unsubscribeTasks = onSnapshot(q, (snapshot) => {
                const tasks = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderTasks(tasks);
                updateDashboard(tasks);
            }, (error) => {
                console.error("Error en el listener de Firestore:", error);
            });
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
                if (groupedTasks[category]) {
                    const tasksInCategory = groupedTasks[category];
                    tasksInCategory.sort((a, b) => new Date(`${a.date}T${a.time || '00:00'}`) - new Date(`${b.date}T${b.time || '00:00'}`));

                    const categorySection = document.createElement('div');
                    categorySection.className = 'card bg-white p-6 rounded-2xl shadow-xl';
                    
                    let sectionHTML = `<h3 class="text-2xl font-semibold mb-4 border-b-2 border-gray-200 pb-2 text-gray-700">${category}</h3>`;
                    const upcomingList = tasksInCategory.filter(t => !t.completed).map(createTaskElement).join('');
                    const completedList = tasksInCategory.filter(t => t.completed).map(createTaskElement).join('');

                    if (upcomingList) sectionHTML += `<ul class="space-y-3">${upcomingList}</ul>`;
                    if (completedList) sectionHTML += `<h4 class="text-lg font-semibold mt-6 mb-3 text-gray-500">Completadas</h4><ul class="space-y-3">${completedList}</ul>`;
                    
                    if (!upcomingList && !completedList) {
                        sectionHTML += `<p class="text-gray-500">No hay tareas en esta categoría.</p>`;
                    }

                    categorySection.innerHTML = sectionHTML;
                    taskListContainer.appendChild(categorySection);
                }
            });

            // Re-attach event listeners
            taskListContainer.querySelectorAll('.task-checkbox').forEach(box => {
                box.addEventListener('change', (e) => {
                    const li = e.target.closest('li');
                    toggleTaskStatus(li.dataset.id, e.target.checked);
                });
            });
            taskListContainer.querySelectorAll('.delete-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    const li = e.target.closest('li');
                    showConfirmModal(li.dataset.id);
                });
            });
        }
        
        function createTaskElement(task) {
            const categoryColors = {'Personal': 'bg-blue-100 text-blue-800','Trabajo': 'bg-purple-100 text-purple-800','Escuela': 'bg-green-100 text-green-800','Otro': 'bg-gray-200 text-gray-800'};
            let formattedTime = '';
            if (task.time) {
                const [hours, minutes] = task.time.split(':');
                const h = parseInt(hours, 10);
                const ampm = h >= 12 ? 'PM' : 'AM';
                const formattedHour = h % 12 || 12;
                formattedTime = ` a las ${formattedHour}:${minutes} ${ampm}`;
            }
            return `<li class="task-item bg-gray-50 p-4 rounded-lg flex items-center justify-between shadow-sm" data-id="${task.id}"><div class="flex items-center gap-4"><input type="checkbox" ${task.completed ? 'checked' : ''} class="task-checkbox h-6 w-6 rounded border-gray-300 text-indigo-600 cursor-pointer focus:ring-indigo-500"><div><span class="task-text font-medium ${task.completed ? 'completed' : ''}">${task.text}</span><div class="text-sm text-gray-500 flex items-center flex-wrap gap-2 mt-1"><span>${new Date(task.date + 'T00:00:00').toLocaleDateString('es-ES', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}${formattedTime}</span><span class="category-badge ${categoryColors[task.category] || categoryColors['Otro']}">${task.category}</span></div></div></div><button class="delete-btn text-gray-400 hover:text-red-500 transition"><svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg></button></li>`;
        }

        function updateDashboard(tasks) {
            const pendingTasks = tasks.filter(t => !t.completed);
            const today = new Date();
            today.setHours(0, 0, 0, 0);

            const todayTasks = pendingTasks.filter(task => {
                const taskDate = new Date(task.date + 'T00:00:00');
                return taskDate.getTime() === today.getTime();
            }).length;

            const overdueTasks = pendingTasks.filter(task => {
                const taskDate = new Date(task.date + 'T00:00:00');
                return taskDate < today;
            }).length;

            document.getElementById('dashboard-today').textContent = todayTasks;
            document.getElementById('dashboard-pending').textContent = pendingTasks.length;
            document.getElementById('dashboard-overdue').textContent = overdueTasks;
        }

        async function handleAddTask() {
            const text = taskInput.value.trim();
            const date = dateInput.value;
            const time = timeInput.value;
            const category = categoryInput.value;
            const user = auth.currentUser;

            if (text && date && user) {
                await addDoc(collection(db, "users", user.uid, "tasks"), { text, date, time, category, completed: false });
                taskInput.value = '';
            }
        }
        
        async function toggleTaskStatus(taskId, isCompleted) {
            const user = auth.currentUser;
            if (user) {
                const taskRef = doc(db, "users", user.uid, "tasks", taskId);
                await updateDoc(taskRef, { completed: isCompleted });
            }
        }
        
        function clearTasksUI() {
             taskListContainer.innerHTML = '';
             document.getElementById('dashboard-today').textContent = 0;
             document.getElementById('dashboard-pending').textContent = 0;
             document.getElementById('dashboard-overdue').textContent = 0;
        }

        addTaskBtn.addEventListener('click', handleAddTask);
        taskInput.addEventListener('keypress', (e) => { if(e.key === 'Enter') handleAddTask(); });
    </script>
</body>
</html>

