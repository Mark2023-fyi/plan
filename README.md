<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planificador Personal Completo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; color: #1f2937; }
        .hidden { display: none; }
        .completed, .subtask-completed { text-decoration: line-through; color: #6b7280; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-fade-in { animation: fadeIn 0.4s ease-out; }
        .loader { border: 4px solid #e5e7eb; border-top: 4px solid #4f46e5; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .modal { animation: fadeIn 0.2s ease-out; }
        .task-item.overdue { background-color: #fee2e2; border-left: 5px solid #ef4444; }
        .task-item.today { background-color: #ffedd5; border-left: 5px solid #f97316; }
        #calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px; }
        .calendar-day { background-color: white; border-radius: 0.5rem; padding: 0.5rem; min-height: 100px; font-size: 0.875rem; color: #4b5563; transition: background-color 0.2s; }
        .calendar-day.other-month { background-color: #f9fafb; color: #d1d5db; }
        .calendar-day.is-today { background-color: #e0e7ff; font-weight: bold; }
        .calendar-day-header { font-weight: 600; text-align: center; margin-bottom: 0.25rem;}
        .task-dot { width: 8px; height: 8px; border-radius: 50%; margin: 1px; display: inline-block; }
        .priority-flag { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }
        .subtask-container, .notes-container { max-height: 0; overflow: hidden; transition: max-height 0.3s ease-out, padding 0.3s ease-out; padding-top: 0; padding-bottom: 0; }
        .subtask-container.open, .notes-container.open { max-height: 500px; padding-top: 1rem; padding-bottom: 1rem; }
        .category-filter-btn.active { background-color: #4f46e5; color: white; }
    </style>
</head>
<body class="antialiased">

    <div id="loader" class="loader"></div>

    <div id="add-task-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full">
            <h3 class="text-2xl font-bold mb-6 text-gray-800">Añadir Nueva Tarea</h3>
            <form id="add-task-form" class="space-y-4">
                <div>
                    <label for="add-task-text" class="block mb-2 text-sm font-medium text-gray-600">Nombre de la Tarea</label>
                    <input type="text" id="add-task-text" placeholder="¿Qué necesitas hacer?" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                        <label for="add-task-date" class="block mb-2 text-sm font-medium text-gray-600">Fecha</label>
                        <input type="date" id="add-task-date" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                    </div>
                    <div>
                        <label for="add-task-time" class="block mb-2 text-sm font-medium text-gray-600">Hora (Opcional)</label>
                        <input type="time" id="add-task-time" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                        <label for="add-task-priority" class="block mb-2 text-sm font-medium text-gray-600">Prioridad</label>
                        <select id="add-task-priority" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="low">Baja</option>
                            <option value="medium" selected>Media</option>
                            <option value="high">Alta</option>
                        </select>
                    </div>
                    <div>
                        <label for="add-task-category" class="block mb-2 text-sm font-medium text-gray-600">Categoría</label>
                        <select id="add-task-category" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="Personal">Personal</option>
                            <option value="Trabajo">Trabajo</option>
                            <option value="Escuela">Escuela</option>
                            <option value="Otro">Otro</option>
                        </select>
                    </div>
                </div>
                <div class="flex justify-end gap-4 pt-4">
                    <button type="button" id="cancel-add-task-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancelar</button>
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Añadir Tarea</button>
                </div>
            </form>
        </div>
    </div>


    <div id="edit-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full">
            <h3 class="text-2xl font-bold mb-6 text-gray-800">Editar Tarea</h3>
            <form id="edit-task-form" class="space-y-4">
                 <div>
                    <label for="edit-task-text" class="block mb-2 text-sm font-medium text-gray-600">Nombre de la Tarea</label>
                    <input type="text" id="edit-task-text" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                     <div>
                        <label for="edit-task-priority" class="block mb-2 text-sm font-medium text-gray-600">Prioridad</label>
                         <select id="edit-task-priority" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="low">Baja</option> <option value="medium">Media</option> <option value="high">Alta</option>
                        </select>
                    </div>
                     <div>
                        <label for="edit-task-category" class="block mb-2 text-sm font-medium text-gray-600">Categoría</label>
                        <select id="edit-task-category" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="Personal">Personal</option> <option value="Trabajo">Trabajo</option> <option value="Escuela">Escuela</option> <option value="Otro">Otro</option>
                        </select>
                    </div>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                     <div>
                        <label for="edit-task-date" class="block mb-2 text-sm font-medium text-gray-600">Fecha</label>
                        <input type="date" id="edit-task-date" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                    </div>
                    <div>
                        <label for="edit-task-time" class="block mb-2 text-sm font-medium text-gray-600">Hora</label>
                        <input type="time" id="edit-task-time" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                </div>
                 <div>
                    <label for="edit-task-notes" class="block mb-2 text-sm font-medium text-gray-600">Notas</label>
                    <textarea id="edit-task-notes" placeholder="Añadir notas..." class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" rows="3"></textarea>
                </div>
                <div class="flex justify-end gap-4 pt-4">
                    <button type="button" id="cancel-edit-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancelar</button>
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Guardar</button>
                </div>
            </form>
        </div>
    </div>
    
    <div id="confirm-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-8 rounded-2xl shadow-xl max-w-sm w-full text-center">
            <h3 class="text-xl font-bold mb-4">¿Confirmar Eliminación?</h3>
            <p class="text-gray-600 mb-6">Esta acción no se puede deshacer.</p>
            <div class="flex justify-center gap-4">
                <button id="cancel-delete-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancelar</button>
                <button id="confirm-delete-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-lg">Eliminar</button>
            </div>
        </div>
    </div>

    <div id="auth-container" class="hidden container mx-auto max-w-md p-4 sm:p-8 mt-10 animate-fade-in">
         <div id="logout-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div>
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl">
            <h2 class="text-3xl font-bold text-center text-indigo-600 mb-8">Iniciar Sesión</h2>
            <form id="login-form">
                <div class="mb-4">
                    <label for="login-email" class="block mb-2 text-sm font-medium text-gray-600">Email</label>
                    <input type="email" id="login-email" autocomplete="username" class="w-full bg-gray-100 rounded-lg p-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <div class="mb-6">
                    <label for="login-password" class="block mb-2 text-sm font-medium text-gray-600">Contraseña</label>
                    <input type="password" id="login-password" autocomplete="current-password" class="w-full bg-gray-100 rounded-lg p-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                </div>
                <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg shadow-md">Entrar</button>
            </form>
            <p id="auth-error" class="text-red-500 text-center mt-4"></p>
            <p class="text-center text-sm text-gray-500 mt-6">Para registrar una cuenta, contacte al administrador.</p>
        </div>
    </div>

    <div id="planner-container" class="hidden container mx-auto p-4 md:p-8 max-w-7xl">
         <header class="flex flex-col sm:flex-row justify-between items-center mb-8 gap-4">
            <div>
                <h1 class="text-3xl md:text-5xl font-bold text-indigo-600">Mi Planificador</h1>
                <p id="user-email" class="text-gray-500 mt-2 text-center sm:text-left"></p>
            </div>
            <button id="logout-btn" class="w-full sm:w-auto bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg shadow-md">Cerrar Sesión</button>
        </header>

        <div id="task-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center">Próxima Tarea</h3><div id="dashboard-next-task" class="text-center mt-2"><p class="text-gray-400">¡Ninguna tarea pendiente!</p></div></div>
            <div class="bg-white p-6 rounded-2xl shadow-xl text-center"><h3 class="text-lg font-semibold text-gray-500">Total Pendientes</h3><p id="dashboard-pending" class="text-4xl font-bold text-indigo-600">0</p></div>
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center">Vencidas (<span id="dashboard-overdue-count">0</span>)</h3><div id="dashboard-overdue" class="text-center mt-2"><p class="text-gray-400">¡Ninguna tarea vencida!</p></div></div>
        </div>

        <div class="bg-white p-4 rounded-2xl shadow-xl mb-8 space-y-4">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="flex items-center bg-gray-100 rounded-lg p-1">
                    <button id="view-list-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md bg-indigo-600 text-white">Lista</button>
                    <button id="view-calendar-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md text-gray-600 hover:bg-gray-200">Calendario</button>
                </div>
                <input type="text" id="searchInput" placeholder="Buscar tareas..." class="w-full bg-gray-100 rounded-lg px-4 py-2 border border-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500">
            </div>
            <div id="category-filters" class="flex flex-wrap items-center gap-2">
                <span class="text-sm font-semibold text-gray-600 mr-2">Categorías:</span>
                <button data-category="All" class="category-filter-btn active text-sm font-medium px-3 py-1 rounded-full bg-gray-200 hover:bg-gray-300">Todos</button>
                <button data-category="Personal" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full bg-gray-200 hover:bg-gray-300">Personal</button>
                <button data-category="Trabajo" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full bg-gray-200 hover:bg-gray-300">Trabajo</button>
                <button data-category="Escuela" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full bg-gray-200 hover:bg-gray-300">Escuela</button>
                <button data-category="Otro" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full bg-gray-200 hover:bg-gray-300">Otro</button>
            </div>
        </div>
        
        <div id="list-view-container">
            <div id="task-list-container" class="space-y-8"></div>
        </div>
        <div id="calendar-view-container" class="hidden">
            <div class="bg-white p-6 rounded-2xl shadow-xl">
                 <div class="flex justify-between items-center mb-4">
                    <button id="prev-month-btn" class="p-2 rounded-full hover:bg-gray-100">&lt;</button>
                    <h2 id="calendar-month-year" class="text-xl font-bold"></h2>
                    <button id="next-month-btn" class="p-2 rounded-full hover:bg-gray-100">&gt;</button>
                </div>
                <div class="grid grid-cols-7 gap-4 text-center font-semibold text-gray-500 text-sm mb-2">
                    <div>Dom</div><div>Lun</div><div>Mar</div><div>Mié</div><div>Jue</div><div>Vie</div><div>Sáb</div>
                </div>
                <div id="calendar-grid"></div>
            </div>
        </div>

        <button id="open-add-task-modal-btn" class="fixed bottom-6 right-6 bg-indigo-600 hover:bg-indigo-700 text-white rounded-full w-14 h-14 flex items-center justify-center shadow-lg transform hover:scale-110 transition-transform">
            <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
        </button>
    </div>

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

        // --- GLOBAL STATE ---
        let allTasks = [];
        let unsubscribeTasks;
        let taskToDeleteId = null;
        let taskToEditId = null;
        let dashboardInterval = null;
        let currentView = 'list';
        let currentSearchTerm = '';
        let currentCategoryFilter = 'All'; // NEW
        let currentCalendarDate = new Date();

        // --- DOM ELEMENTS ---
        const DOMElements = {
            loader: document.getElementById('loader'),
            authContainer: document.getElementById('auth-container'),
            plannerContainer: document.getElementById('planner-container'),
            userEmail: document.getElementById('user-email'),
            authError: document.getElementById('auth-error'),
            loginForm: document.getElementById('login-form'),
            logoutBtn: document.getElementById('logout-btn'),
            logoutSuccessMessage: document.getElementById('logout-success-message'),
            taskSuccessMessage: document.getElementById('task-success-message'),
            confirmModal: document.getElementById('confirm-modal'),
            cancelDeleteBtn: document.getElementById('cancel-delete-btn'),
            confirmDeleteBtn: document.getElementById('confirm-delete-btn'),
            // Add Task Modal
            openAddTaskModalBtn: document.getElementById('open-add-task-modal-btn'),
            addTaskModal: document.getElementById('add-task-modal'),
            addTaskForm: document.getElementById('add-task-form'),
            cancelAddTaskBtn: document.getElementById('cancel-add-task-btn'),
            addTaskText: document.getElementById('add-task-text'),
            addTaskDate: document.getElementById('add-task-date'),
            addTaskTime: document.getElementById('add-task-time'),
            addTaskCategory: document.getElementById('add-task-category'),
            addTaskPriority: document.getElementById('add-task-priority'),
            // Edit Task Modal
            editModal: document.getElementById('edit-modal'),
            editTaskForm: document.getElementById('edit-task-form'),
            cancelEditBtn: document.getElementById('cancel-edit-btn'),
            editTaskText: document.getElementById('edit-task-text'),
            editTaskDate: document.getElementById('edit-task-date'),
            editTaskTime: document.getElementById('edit-task-time'),
            editTaskCategory: document.getElementById('edit-task-category'),
            editTaskPriority: document.getElementById('edit-task-priority'),
            editTaskNotes: document.getElementById('edit-task-notes'),
            // Task List & Filters
            taskListContainer: document.getElementById('task-list-container'),
            viewListBtn: document.getElementById('view-list-btn'),
            viewCalendarBtn: document.getElementById('view-calendar-btn'),
            listViewContainer: document.getElementById('list-view-container'),
            calendarViewContainer: document.getElementById('calendar-view-container'),
            searchInput: document.getElementById('searchInput'),
            categoryFilters: document.getElementById('category-filters'),
            // Calendar
            calendarGrid: document.getElementById('calendar-grid'),
            calendarMonthYear: document.getElementById('calendar-month-year'),
            prevMonthBtn: document.getElementById('prev-month-btn'),
            nextMonthBtn: document.getElementById('next-month-btn'),
            // Dashboard
            dashboardNextTask: document.getElementById('dashboard-next-task'),
            dashboardPending: document.getElementById('dashboard-pending'),
            dashboardOverdue: document.getElementById('dashboard-overdue'),
            dashboardOverdueCount: document.getElementById('dashboard-overdue-count'),
        };

        // --- AUTH LOGIC ---
        onAuthStateChanged(auth, user => {
            if (user) {
                DOMElements.authContainer.classList.add('hidden');
                DOMElements.plannerContainer.classList.remove('hidden');
                DOMElements.userEmail.textContent = `Sesión iniciada como: ${user.email}`;
                loadTasks(user.uid);
                if (dashboardInterval) clearInterval(dashboardInterval);
                dashboardInterval = setInterval(refreshDynamicContent, 5000);
            } else {
                DOMElements.plannerContainer.classList.add('hidden');
                DOMElements.authContainer.classList.remove('hidden');
                if(unsubscribeTasks) unsubscribeTasks();
                if (dashboardInterval) clearInterval(dashboardInterval);
                clearUI();
            }
            DOMElements.loader.classList.add('hidden');
        });

        DOMElements.loginForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            DOMElements.authError.textContent = '';
            DOMElements.loader.classList.remove('hidden');
            const email = e.target.elements['login-email'].value;
            const password = e.target.elements['login-password'].value;
            try {
                await signInWithEmailAndPassword(auth, email, password);
            } catch (error) {
                DOMElements.authError.textContent = 'El correo o la contraseña son incorrectos.';
            } finally {
                DOMElements.loader.classList.add('hidden');
            }
        });

        DOMElements.logoutBtn.addEventListener('click', () => {
            signOut(auth).then(() => {
                DOMElements.loginForm.reset();
                showFlashMessage(DOMElements.logoutSuccessMessage, "¡Sesión cerrada exitosamente!");
            });
        });

        // --- DATA HANDLING (FIRESTORE) ---
        async function loadTasks(userId) {
            const tasksCollection = collection(db, "users", userId, "tasks");
            if (unsubscribeTasks) unsubscribeTasks();
            unsubscribeTasks = onSnapshot(query(tasksCollection), snapshot => {
                allTasks = snapshot.docs.map(doc => ({ 
                    id: doc.id, 
                    ...doc.data(),
                    subtasks: doc.data().subtasks || [],
                    notes: doc.data().notes || ''
                }));
                refreshDynamicContent();
            });
        }
        
        async function crudOperation(action, data) {
            const user = auth.currentUser;
            if (!user) return;
            const { id, ...payload } = data;
            try {
                if (action === 'add') {
                    await addDoc(collection(db, "users", user.uid, "tasks"), payload);
                } else if (action === 'update') {
                    await updateDoc(doc(db, "users", user.uid, "tasks", id), payload);
                } else if (action === 'delete') {
                    await deleteDoc(doc(db, "users", user.uid, "tasks", id));
                }
            } catch (error) {
                console.error("Firestore Error:", error);
            }
        }
        
        // --- RENDERING ---
        function refreshDynamicContent() {
            const filteredTasks = getFilteredTasks();
            if (currentView === 'list') {
                renderListView(filteredTasks);
            } else {
                renderCalendarView();
            }
            updateDashboard(allTasks);
        }

        function getFilteredTasks() {
            const searchFiltered = allTasks.filter(task => 
                task.text.toLowerCase().includes(currentSearchTerm.toLowerCase())
            );
            if (currentCategoryFilter === 'All') {
                return searchFiltered;
            }
            return searchFiltered.filter(task => task.category === currentCategoryFilter);
        }

        function renderListView(tasks) {
            DOMElements.taskListContainer.innerHTML = '';
            const groupedTasks = tasks.reduce((acc, task) => {
                const category = task.category || 'Otro';
                if (!acc[category]) acc[category] = [];
                acc[category].push(task);
                return acc;
            }, {});

            const categories = Object.keys(groupedTasks);
            categories.sort();

            if (tasks.length === 0) {
                 DOMElements.taskListContainer.innerHTML = `<div class="text-center text-gray-500 py-10">
                    <h3 class="text-xl font-semibold">No hay tareas que mostrar</h3>
                    <p>Intenta añadir una tarea o cambia los filtros.</p>
                </div>`;
                return;
            }

            categories.forEach(category => {
                const tasksInCategory = groupedTasks[category];
                tasksInCategory.sort((a, b) => new Date(`${a.date}T${a.time || '00:00'}`) - new Date(`${b.date}T${b.time || '00:00'}`));

                const categorySection = document.createElement('div');
                categorySection.className = 'bg-white p-6 rounded-2xl shadow-xl animate-fade-in';
                
                let sectionHTML = `<h3 class="text-2xl font-semibold mb-4 border-b pb-2">${category}</h3>`;
                const taskList = tasksInCategory.map(createTaskElement).join('');
                sectionHTML += `<ul class="space-y-3">${taskList}</ul>`;
                
                categorySection.innerHTML = sectionHTML;
                DOMElements.taskListContainer.appendChild(categorySection);
            });

            attachDynamicListeners(DOMElements.taskListContainer);
        }
        
        function updateDashboard(tasks) {
            const pendingTasks = tasks.filter(t => !t.completed);
            const now = new Date();

            const upcomingTasks = pendingTasks
                .map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59:59'}`) }))
                .filter(t => t.dateTime >= now)
                .sort((a, b) => a.dateTime - b.dateTime);

            if (upcomingTasks.length > 0) {
                const nextTask = upcomingTasks[0];
                DOMElements.dashboardNextTask.innerHTML = `<p class="font-bold text-lg text-indigo-600 break-words">${nextTask.text}</p><p class="text-sm text-gray-500 mt-1">${formatTimeDifference(nextTask.dateTime)}</p>`;
            } else {
                DOMElements.dashboardNextTask.innerHTML = `<p class="text-gray-400">¡Ninguna tarea pendiente!</p>`;
            }
            
            DOMElements.dashboardPending.textContent = pendingTasks.length;

            const overdueTasks = pendingTasks
                .map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59:59'}`) }))
                .filter(t => t.dateTime < now)
                .sort((a, b) => a.dateTime - b.dateTime);
            
            DOMElements.dashboardOverdueCount.textContent = overdueTasks.length;
            if(overdueTasks.length > 0) {
                const oldestOverdue = overdueTasks[0];
                DOMElements.dashboardOverdue.innerHTML = `<p class="font-bold text-lg text-red-500 break-words">${oldestOverdue.text}</p><p class="text-sm text-gray-500 mt-1">${formatTimeDifference(oldestOverdue.dateTime)}</p>`;
            } else {
                DOMElements.dashboardOverdue.innerHTML = `<p class="text-gray-400">¡Ninguna tarea vencida!</p>`;
            }
        }

        function createTaskElement(task) {
            const priorityMap = { high: { color: 'bg-red-500' }, medium: { color: 'bg-yellow-500' }, low: { color: 'bg-green-500' }};
            const now = new Date();
            const taskDateTime = new Date(`${task.date}T${task.time || '23:59:59'}`);
            let urgencyClass = '';
            if (!task.completed && taskDateTime < now) urgencyClass = 'overdue';
            else if (!task.completed && now.toDateString() === taskDateTime.toDateString()) urgencyClass = 'today';

            let timeDiffHtml = '';
            if (!task.completed) {
                const diffSeconds = (taskDateTime - now) / 1000;
                let colorClass = 'text-gray-500';
                if (diffSeconds < 0) colorClass = 'text-red-600 font-medium';
                else if (diffSeconds < 86400 * 2) colorClass = 'text-orange-600 font-medium';
                timeDiffHtml = `<span class="${colorClass}">${formatTimeDifference(taskDateTime)}</span>`;
            }

            const subtasksHtml = task.subtasks.map((sub, index) => `<div class="flex items-center gap-2 ml-4"><input type="checkbox" id="subtask-${task.id}-${index}" data-subtask-index="${index}" class="subtask-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500" ${sub.completed ? 'checked' : ''}><label for="subtask-${task.id}-${index}" class="text-sm ${sub.completed ? 'subtask-completed' : ''}">${sub.text}</label></div>`).join('');
            const subtaskProgress = task.subtasks.length > 0 ? `(${task.subtasks.filter(s=>s.completed).length}/${task.subtasks.length})` : '';

            return `
                <li class="task-item ${urgencyClass} p-4 rounded-lg border border-gray-200" data-id="${task.id}">
                    <div class="flex items-start justify-between">
                        <div class="flex items-start gap-3 flex-grow min-w-0">
                            <input type="checkbox" class="task-checkbox h-5 w-5 rounded border-gray-300 text-indigo-600 flex-shrink-0 mt-1 focus:ring-indigo-500" ${task.completed ? 'checked' : ''}>
                            <div class="min-w-0">
                                <div class="flex items-center gap-2 flex-wrap">
                                     <span class="priority-flag ${priorityMap[task.priority]?.color || 'bg-gray-400'}" title="Prioridad ${task.priority}"></span>
                                     <span class="font-medium break-words ${task.completed ? 'completed' : ''}">${task.text}</span>
                                </div>
                                <div class="text-xs text-gray-500 mt-1 flex items-center gap-2 flex-wrap">
                                    <span>${taskDateTime.toLocaleDateString('es-ES', { day: 'numeric', month: 'short' })} ${task.time || ''}</span>
                                    ${!task.completed ? `<span class="mx-1">•</span> ${timeDiffHtml}` : ''}
                                </div>
                            </div>
                        </div>
                        <div class="flex items-center flex-shrink-0 ml-2">
                             <button class="subtasks-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Subtareas">${subtaskProgress || `<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01"></path></svg>`}</button>
                             <button class="notes-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Notas"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg></button>
                             <button class="edit-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Editar"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path></svg></button>
                             <button class="delete-btn text-gray-500 hover:text-red-500 p-1.5 rounded-full hover:bg-gray-100" title="Eliminar"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg></button>
                        </div>
                    </div>
                    <div class="notes-container ml-8 border-l-2 pl-4 mt-2"><p class="text-sm text-gray-600 whitespace-pre-wrap">${task.notes || 'No hay notas.'}</p></div>
                    <div class="subtask-container ml-8 border-l-2 pl-4 mt-2 space-y-2">${subtasksHtml}<div class="flex gap-2 pt-2"><input type="text" class="new-subtask-input bg-gray-100 rounded px-2 py-1 text-sm flex-grow border focus:outline-none focus:ring-1 focus:ring-indigo-500" placeholder="Nueva subtarea..."><button class="add-subtask-btn bg-indigo-500 hover:bg-indigo-600 text-white text-xs px-2.5 rounded">+</button></div></div>
                </li>`;
        }
        
        function renderCalendarView() {
            DOMElements.calendarGrid.innerHTML = '';
            const month = currentCalendarDate.getMonth();
            const year = currentCalendarDate.getFullYear();
            DOMElements.calendarMonthYear.textContent = new Date(year, month).toLocaleDateString('es-ES', { month: 'long', year: 'numeric' });
            const firstDayOfMonth = new Date(year, month, 1).getDay();
            const daysInMonth = new Date(year, month + 1, 0).getDate();

            for (let i = 0; i < firstDayOfMonth; i++) {
                DOMElements.calendarGrid.insertAdjacentHTML('beforeend', `<div class="calendar-day other-month"></div>`);
            }
            for (let day = 1; day <= daysInMonth; day++) {
                const dayEl = document.createElement('div');
                dayEl.className = 'calendar-day';
                const today = new Date();
                if (day === today.getDate() && month === today.getMonth() && year === today.getFullYear()) dayEl.classList.add('is-today');
                dayEl.innerHTML = `<div class="calendar-day-header">${day}</div>`;
                const tasksOnDay = allTasks.filter(task => { const d = new Date(task.date + 'T00:00:00'); return d.getDate() === day && d.getMonth() === month && d.getFullYear() === year; });
                const dotsContainer = document.createElement('div');
                dotsContainer.className = 'flex flex-wrap';
                tasksOnDay.slice(0, 9).forEach(task => { dotsContainer.innerHTML += `<span class="task-dot ${task.completed ? 'bg-gray-300' : 'bg-indigo-500'}" title="${task.text}"></span>`; });
                dayEl.appendChild(dotsContainer);
                DOMElements.calendarGrid.appendChild(dayEl);
            }
        }
        
        // --- EVENT LISTENERS ---
        function setupEventListeners() {
            // Add Task
            DOMElements.openAddTaskModalBtn.addEventListener('click', showAddTaskModal);
            DOMElements.cancelAddTaskBtn.addEventListener('click', hideAddTaskModal);
            DOMElements.addTaskForm.addEventListener('submit', handleAddTask);
            
            // Search & Filters
            DOMElements.searchInput.addEventListener('input', e => { currentSearchTerm = e.target.value; refreshDynamicContent(); });
            DOMElements.categoryFilters.addEventListener('click', handleCategoryFilter);
            DOMElements.viewListBtn.addEventListener('click', () => switchView('list'));
            DOMElements.viewCalendarBtn.addEventListener('click', () => switchView('calendar'));
            
            // Modals
            DOMElements.cancelDeleteBtn.addEventListener('click', hideConfirmModal);
            DOMElements.confirmDeleteBtn.addEventListener('click', () => { if (taskToDeleteId) { crudOperation('delete', { id: taskToDeleteId }); hideConfirmModal(); } });
            DOMElements.cancelEditBtn.addEventListener('click', hideEditModal);
            DOMElements.editTaskForm.addEventListener('submit', handleEditTask);

            // Calendar
            DOMElements.prevMonthBtn.addEventListener('click', () => { currentCalendarDate.setMonth(currentCalendarDate.getMonth() - 1); renderCalendarView(); });
            DOMElements.nextMonthBtn.addEventListener('click', () => { currentCalendarDate.setMonth(currentCalendarDate.getMonth() + 1); renderCalendarView(); });
        }
        
        function attachDynamicListeners(container) {
            container.addEventListener('click', e => {
                const target = e.target;
                const taskLi = target.closest('li.task-item');
                if (!taskLi) return;
                const taskId = taskLi.dataset.id;
                
                if (target.closest('.task-checkbox')) { crudOperation('update', { id: taskId, completed: target.closest('.task-checkbox').checked }); }
                else if (target.closest('.edit-btn')) { showEditModal(taskId); }
                else if (target.closest('.delete-btn')) { showConfirmModal(taskId); }
                else if (target.closest('.notes-btn')) { taskLi.querySelector('.notes-container').classList.toggle('open'); }
                else if (target.closest('.subtasks-btn')) { taskLi.querySelector('.subtask-container').classList.toggle('open'); }
                else if (target.closest('.add-subtask-btn')) { const input = taskLi.querySelector('.new-subtask-input'); if (input.value.trim()) handleAddSubtask(taskId, input.value.trim()); input.value = ''; }
                else if (target.closest('.subtask-checkbox')) { const index = parseInt(target.closest('.subtask-checkbox').dataset.subtaskIndex, 10); handleSubtaskToggle(taskId, index); }
            });
        }
        
        // --- HANDLERS ---
        async function handleAddTask(e) {
            e.preventDefault();
            const { addTaskText, addTaskDate, addTaskTime, addTaskCategory, addTaskPriority } = DOMElements;
            if (addTaskText.value.trim() && addTaskDate.value) {
                await crudOperation('add', {
                    text: addTaskText.value.trim(),
                    date: addTaskDate.value,
                    time: addTaskTime.value,
                    category: addTaskCategory.value,
                    priority: addTaskPriority.value,
                    completed: false, notes: '', subtasks: []
                });
                hideAddTaskModal();
                showFlashMessage(DOMElements.taskSuccessMessage, "¡Tarea añadida exitosamente!");
            }
        }

        async function handleEditTask(e) {
            e.preventDefault();
            const taskData = allTasks.find(t => t.id === taskToEditId);
            if (!taskData) return;
            const { editTaskText, editTaskDate, editTaskTime, editTaskCategory, editTaskPriority, editTaskNotes } = DOMElements;
            await crudOperation('update', {
                ...taskData, id: taskToEditId, text: editTaskText.value, date: editTaskDate.value, time: editTaskTime.value,
                category: editTaskCategory.value, priority: editTaskPriority.value, notes: editTaskNotes.value,
            });
            hideEditModal();
        }
        
        function handleCategoryFilter(e) {
            const target = e.target.closest('.category-filter-btn');
            if (!target) return;
            currentCategoryFilter = target.dataset.category;
            document.querySelectorAll('.category-filter-btn').forEach(btn => btn.classList.remove('active'));
            target.classList.add('active');
            refreshDynamicContent();
        }

        function handleAddSubtask(taskId, text) { const task = allTasks.find(t => t.id === taskId); if (task) crudOperation('update', { id: taskId, subtasks: [...task.subtasks, { text, completed: false }] }); }
        function handleSubtaskToggle(taskId, subtaskIndex) { const task = allTasks.find(t => t.id === taskId); if (task) crudOperation('update', { id: taskId, subtasks: task.subtasks.map((s, i) => i === subtaskIndex ? { ...s, completed: !s.completed } : s) }); }
        
        // --- UI & VIEW LOGIC ---
        function switchView(view) {
            currentView = view;
            const isList = view === 'list';
            DOMElements.listViewContainer.classList.toggle('hidden', !isList);
            DOMElements.categoryFilters.classList.toggle('hidden', !isList);
            DOMElements.calendarViewContainer.classList.toggle('hidden', isList);
            DOMElements.viewListBtn.classList.toggle('bg-indigo-600', isList);
            DOMElements.viewListBtn.classList.toggle('text-white', isList);
            DOMElements.viewCalendarBtn.classList.toggle('bg-indigo-600', !isList);
            DOMElements.viewCalendarBtn.classList.toggle('text-white', !isList);
            refreshDynamicContent();
        }

        function showAddTaskModal() { DOMElements.addTaskForm.reset(); setSmartDefaults(); DOMElements.addTaskModal.classList.remove('hidden'); DOMElements.addTaskText.focus(); }
        function hideAddTaskModal() { DOMElements.addTaskModal.classList.add('hidden'); }
        function showEditModal(taskId) {
            taskToEditId = taskId;
            const task = allTasks.find(t => t.id === taskId);
            if(task) {
                DOMElements.editTaskText.value = task.text; DOMElements.editTaskDate.value = task.date; DOMElements.editTaskTime.value = task.time || '';
                DOMElements.editTaskCategory.value = task.category; DOMElements.editTaskPriority.value = task.priority || 'medium'; DOMElements.editTaskNotes.value = task.notes || '';
                DOMElements.editModal.classList.remove('hidden');
            }
        }
        function hideEditModal() { DOMElements.editModal.classList.add('hidden'); taskToEditId = null; }
        function showConfirmModal(taskId) { taskToDeleteId = taskId; DOMElements.confirmModal.classList.remove('hidden'); }
        function hideConfirmModal() { taskToDeleteId = null; DOMElements.confirmModal.classList.add('hidden'); }
        
        // --- UTILITY FUNCTIONS ---
        function formatTimeDifference(taskDate) { const rtf = new Intl.RelativeTimeFormat('es', { numeric: 'auto' }); const diffSeconds = (new Date(taskDate).getTime() - new Date().getTime()) / 1000; const diffDays = diffSeconds / (60 * 60 * 24); if (Math.abs(diffDays) >= 1) return rtf.format(Math.round(diffDays), 'day'); const diffHours = diffSeconds / (60 * 60); if (Math.abs(diffHours) >= 1) return rtf.format(Math.round(diffHours), 'hour'); const diffMinutes = diffSeconds / 60; if (Math.abs(diffMinutes) >= 1) return rtf.format(Math.round(diffMinutes), 'minute'); return 'en unos segundos'; }
        function showFlashMessage(element, message) { element.textContent = message; element.classList.remove('hidden'); setTimeout(() => element.classList.add('hidden'), 3000); }
        function clearUI() { DOMElements.taskListContainer.innerHTML = ''; DOMElements.dashboardPending.textContent = '0'; DOMElements.dashboardNextTask.innerHTML = `<p class="text-gray-400">¡Ninguna tarea pendiente!</p>`; DOMElements.dashboardOverdue.innerHTML = `<p class="text-gray-400">¡Ninguna tarea vencida!</p>`; DOMElements.dashboardOverdueCount.textContent = '0'; }
        function setSmartDefaults() { DOMElements.addTaskDate.value = new Date().toISOString().split('T')[0]; const now = new Date(); const nextHour = (now.getHours() + 1) % 24; DOMElements.addTaskTime.value = `${String(nextHour).padStart(2, '0')}:00`; }

        // Initialize App
        setupEventListeners();

    </script>
</body>
</html>
