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
        .calendar-day { background-color: white; border-radius: 0.5rem; padding: 0.5rem; min-height: 110px; font-size: 0.875rem; color: #4b5563; transition: background-color 0.2s; cursor: pointer; display: flex; flex-direction: column; }
        .calendar-day:not(.other-month):hover { background-color: #eff6ff; }
        .calendar-day.other-month { background-color: #f9fafb; color: #d1d5db; cursor: default; }
        .calendar-day.is-today .calendar-day-header { color: white; background-color: #4f46e5; border-radius: 9999px; width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; }
        .calendar-day-header { font-weight: 600; text-align: center; margin-bottom: 0.25rem; }
        .priority-flag { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }
        .subtask-container, .notes-container { max-height: 0; overflow: hidden; transition: max-height 0.3s ease-out, padding 0.3s ease-out; padding-top: 0; padding-bottom: 0; }
        .subtask-container.open, .notes-container.open { max-height: 500px; padding-top: 1rem; padding-bottom: 1rem; }
        .category-filter-btn { transition: background-color 0.2s, color 0.2s; }
        .calendar-task-item { font-size: 0.75rem; padding: 2px 6px; border-radius: 4px; margin-top: 4px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        .calendar-tasks-container { flex-grow: 1; overflow: hidden; }
    </style>
</head>
<body class="antialiased">

    <div id="loader" class="loader"></div>

    <div id="add-task-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full">
            <h3 class="text-2xl font-bold mb-6 text-gray-800">Añadir Nueva Tarea</h3>
            <form id="add-task-form" class="space-y-4">
                <input type="text" id="add-task-text" placeholder="¿Qué necesitas hacer?" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <input type="date" id="add-task-date" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                    <input type="time" id="add-task-time" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <select id="add-task-priority" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="low">Prioridad Baja</option><option value="medium" selected>Prioridad Media</option><option value="high">Prioridad Alta</option></select>
                    <select id="add-task-category" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="Personal">Personal</option><option value="Trabajo">Trabajo</option><option value="Escuela">Escuela</option><option value="Otro">Otro</option></select>
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
                 <input type="text" id="edit-task-text" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <select id="edit-task-priority" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="low">Baja</option> <option value="medium">Media</option> <option value="high">Alta</option></select>
                    <select id="edit-task-category" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="Personal">Personal</option> <option value="Trabajo">Trabajo</option> <option value="Escuela">Escuela</option> <option value="Otro">Otro</option></select>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <input type="date" id="edit-task-date" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                    <input type="time" id="edit-task-time" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <textarea id="edit-task-notes" placeholder="Añadir notas..." class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" rows="3"></textarea>
                <div class="flex justify-end gap-4 pt-4">
                    <button type="button" id="cancel-edit-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancelar</button>
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Guardar</button>
                </div>
            </form>
        </div>
    </div>
    
    <div id="confirm-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-8 rounded-2xl shadow-xl max-w-sm w-full text-center">
            <h3 class="text-xl font-bold mb-4">¿Confirmar Eliminación?</h3><p class="text-gray-600 mb-6">Esta acción no se puede deshacer.</p>
            <div class="flex justify-center gap-4">
                <button id="cancel-delete-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancelar</button>
                <button id="confirm-delete-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-lg">Eliminar</button>
            </div>
        </div>
    </div>
    
    <div id="calendar-day-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-md w-full">
            <h3 id="calendar-modal-title" class="text-xl font-bold mb-4 text-gray-800"></h3>
            <div id="calendar-modal-tasks" class="space-y-2 max-h-80 overflow-y-auto"></div>
            <div class="text-right mt-6">
                 <button id="calendar-modal-close-btn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Cerrar</button>
            </div>
        </div>
    </div>

    <div id="auth-container" class="hidden container mx-auto max-w-md p-4 sm:p-8 mt-10 animate-fade-in">
        <div id="logout-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div>
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl">
            <h2 class="text-3xl font-bold text-center text-indigo-600 mb-8">Iniciar Sesión</h2>
            <form id="login-form">
                <input type="email" id="login-email" autocomplete="username" class="w-full bg-gray-100 rounded-lg p-3 border mb-4 focus:outline-none focus:ring-2 focus:ring-indigo-500" required placeholder="Email">
                <input type="password" id="login-password" autocomplete="current-password" class="w-full bg-gray-100 rounded-lg p-3 border mb-6 focus:outline-none focus:ring-2 focus:ring-indigo-500" required placeholder="Contraseña">
                <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg shadow-md">Entrar</button>
            </form>
            <p id="auth-error" class="text-red-500 text-center mt-4"></p>
            <p class="text-center text-sm text-gray-500 mt-6">Para registrar una cuenta, contacte al administrador.</p>
        </div>
    </div>

    <div id="planner-container" class="hidden container mx-auto p-4 md:p-8 max-w-7xl">
        <header class="flex flex-wrap justify-between items-center mb-8 gap-4">
            <div><h1 class="text-3xl md:text-4xl font-bold text-indigo-600">Mi Planificador</h1><p id="current-date" class="text-gray-500 text-sm md:text-base"></p></div>
            <div class="flex items-center gap-3 sm:gap-4"><div class="text-right hidden sm:block"><p id="user-greeting" class="font-semibold text-gray-800"></p><p class="text-xs text-gray-500">¡A organizar tu día!</p></div><div id="user-avatar" class="w-12 h-12 rounded-full bg-indigo-500 flex items-center justify-center text-white font-bold text-xl uppercase"></div><button id="logout-btn" title="Cerrar Sesión" class="bg-gray-200 hover:bg-red-500 hover:text-white text-gray-600 p-2 rounded-full transition-colors duration-200"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"></path></svg></button></div>
        </header>

        <div id="task-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center">Próxima Tarea</h3><div id="dashboard-next-task" class="text-center mt-2"><p class="text-gray-400">¡Ninguna tarea pendiente!</p></div></div>
            <div class="bg-white p-6 rounded-2xl shadow-xl text-center"><h3 class="text-lg font-semibold text-gray-500">Total Pendientes</h3><p id="dashboard-pending" class="text-4xl font-bold text-indigo-600">0</p></div>
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center">Vencidas (<span id="dashboard-overdue-count">0</span>)</h3><div id="dashboard-overdue" class="text-center mt-2"><p class="text-gray-400">¡Ninguna tarea vencida!</p></div></div>
        </div>

        <div class="bg-white p-4 rounded-2xl shadow-xl mb-8 space-y-4">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4"><div class="flex items-center bg-gray-100 rounded-lg p-1"><button id="view-list-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md bg-indigo-600 text-white">Lista</button><button id="view-calendar-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md text-gray-600 hover:bg-gray-200">Calendario</button></div><input type="text" id="searchInput" placeholder="Buscar tareas..." class="w-full bg-gray-100 rounded-lg px-4 py-2 border border-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500"></div>
            <div id="category-filters" class="flex flex-wrap items-center gap-2"><span class="text-sm font-semibold text-gray-600 mr-2">Categorías:</span><button data-category="All" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full">Todos</button><button data-category="Personal" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full">Personal</button><button data-category="Trabajo" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full">Trabajo</button><button data-category="Escuela" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full">Escuela</button><button data-category="Otro" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full">Otro</button></div>
        </div>
        
        <div id="list-view-container"><div id="task-list-container" class="space-y-8"></div></div>
        <div id="calendar-view-container" class="hidden"><div class="bg-white p-6 rounded-2xl shadow-xl"><div class="flex justify-between items-center mb-4"><button id="prev-month-btn" class="p-2 rounded-full hover:bg-gray-100">&lt;</button><h2 id="calendar-month-year" class="text-xl font-bold"></h2><button id="next-month-btn" class="p-2 rounded-full hover:bg-gray-100">&gt;</button></div><div class="grid grid-cols-7 gap-4 text-center font-semibold text-gray-500 text-sm mb-2"><div>Dom</div><div>Lun</div><div>Mar</div><div>Mié</div><div>Jue</div><div>Vie</div><div>Sáb</div></div><div id="calendar-grid"></div></div></div>

        <button id="open-add-task-modal-btn" class="fixed bottom-6 right-6 bg-indigo-600 hover:bg-indigo-700 text-white rounded-full w-14 h-14 flex items-center justify-center shadow-lg transform hover:scale-110 transition-transform"><svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg></button>
        <div id="saving-indicator" class="hidden fixed bottom-5 left-1/2 -translate-x-1/2 bg-gray-800 text-white text-sm py-2 px-4 rounded-full shadow-lg flex items-center gap-2 z-50"><svg class="animate-spin h-4 w-4 text-white" xmlns="http://www.w.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>Guardando...</div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, doc, updateDoc, deleteDoc } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-firestore.js";

        const firebaseConfig = { apiKey: "AIzaSyA_LRcHCkClvlHeqDPTSKfGa5gY2uiuZ5E", authDomain: "mi-planificador-privado.firebaseapp.com", projectId: "mi-planificador-privado", storageBucket: "mi-planificador-privado.firebasestorage.app", messagingSenderId: "792700686473", appId: "1:792700686473:web:8d1f41076f12fa0103f658" };
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let allTasks = [], unsubscribeTasks, taskToDeleteId = null, taskToEditId = null, timerInterval = null, currentView = 'list', currentSearchTerm = '', currentCategoryFilter = 'All', currentCalendarDate = new Date();
        const categoryMap = { 'Personal': 'bg-blue-100 text-blue-800', 'Trabajo': 'bg-purple-100 text-purple-800', 'Escuela': 'bg-green-100 text-green-800', 'Otro': 'bg-gray-200 text-gray-800' };
        const categoryActiveMap = { 'Personal': 'bg-blue-500 text-white', 'Trabajo': 'bg-purple-500 text-white', 'Escuela': 'bg-green-500 text-white', 'Otro': 'bg-gray-500 text-white' };

        const DOMElements = {
            loader: document.getElementById('loader'), savingIndicator: document.getElementById('saving-indicator'), authContainer: document.getElementById('auth-container'), plannerContainer: document.getElementById('planner-container'),
            userGreeting: document.getElementById('user-greeting'), currentDate: document.getElementById('current-date'), userAvatar: document.getElementById('user-avatar'),
            authError: document.getElementById('auth-error'), loginForm: document.getElementById('login-form'), logoutBtn: document.getElementById('logout-btn'),
            logoutSuccessMessage: document.getElementById('logout-success-message'), taskSuccessMessage: document.getElementById('task-success-message'),
            confirmModal: document.getElementById('confirm-modal'), cancelDeleteBtn: document.getElementById('cancel-delete-btn'), confirmDeleteBtn: document.getElementById('confirm-delete-btn'),
            openAddTaskModalBtn: document.getElementById('open-add-task-modal-btn'), addTaskModal: document.getElementById('add-task-modal'), addTaskForm: document.getElementById('add-task-form'),
            cancelAddTaskBtn: document.getElementById('cancel-add-task-btn'), addTaskText: document.getElementById('add-task-text'), addTaskDate: document.getElementById('add-task-date'),
            addTaskTime: document.getElementById('add-task-time'), addTaskCategory: document.getElementById('add-task-category'), addTaskPriority: document.getElementById('add-task-priority'),
            editModal: document.getElementById('edit-modal'), editTaskForm: document.getElementById('edit-task-form'), cancelEditBtn: document.getElementById('cancel-edit-btn'),
            editTaskText: document.getElementById('edit-task-text'), editTaskDate: document.getElementById('edit-task-date'), editTaskTime: document.getElementById('edit-task-time'),
            editTaskCategory: document.getElementById('edit-task-category'), editTaskPriority: document.getElementById('edit-task-priority'), editTaskNotes: document.getElementById('edit-task-notes'),
            taskListContainer: document.getElementById('task-list-container'), viewListBtn: document.getElementById('view-list-btn'), viewCalendarBtn: document.getElementById('view-calendar-btn'),
            listViewContainer: document.getElementById('list-view-container'), calendarViewContainer: document.getElementById('calendar-view-container'),
            searchInput: document.getElementById('searchInput'), categoryFilters: document.getElementById('category-filters'),
            calendarGrid: document.getElementById('calendar-grid'), calendarMonthYear: document.getElementById('calendar-month-year'), prevMonthBtn: document.getElementById('prev-month-btn'), nextMonthBtn: document.getElementById('next-month-btn'),
            calendarDayModal: document.getElementById('calendar-day-modal'), calendarModalTitle: document.getElementById('calendar-modal-title'), calendarModalTasks: document.getElementById('calendar-modal-tasks'), calendarModalCloseBtn: document.getElementById('calendar-modal-close-btn'),
            dashboardNextTask: document.getElementById('dashboard-next-task'), dashboardPending: document.getElementById('dashboard-pending'), dashboardOverdue: document.getElementById('dashboard-overdue'), dashboardOverdueCount: document.getElementById('dashboard-overdue-count'),
        };

        onAuthStateChanged(auth, user => {
            if (user) {
                DOMElements.authContainer.classList.add('hidden'); DOMElements.plannerContainer.classList.remove('hidden');
                const emailName = user.email.split('@')[0]; DOMElements.userGreeting.textContent = `${getGreeting()}, ${emailName}`; DOMElements.userAvatar.textContent = getInitials(user.email);
                const today = new Date(); DOMElements.currentDate.textContent = today.toLocaleDateString('es-ES', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
                loadTasks(user.uid); if (timerInterval) clearInterval(timerInterval); timerInterval = setInterval(updateRelativeTimes, 30000);
            } else {
                DOMElements.plannerContainer.classList.add('hidden'); DOMElements.authContainer.classList.remove('hidden');
                if(unsubscribeTasks) unsubscribeTasks(); if (timerInterval) clearInterval(timerInterval); clearUI();
            }
            DOMElements.loader.classList.add('hidden');
        });

        async function loadTasks(userId) { const tasksCollection = collection(db, "users", userId, "tasks"); if (unsubscribeTasks) unsubscribeTasks(); unsubscribeTasks = onSnapshot(query(tasksCollection), snapshot => { allTasks = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data(), subtasks: doc.data().subtasks || [], notes: doc.data().notes || '' })); refreshDynamicContent(); }); }
        async function crudOperation(action, data) { const user = auth.currentUser; if (!user) return; const { id, ...payload } = data; showSavingIndicator(); try { if (action === 'add') await addDoc(collection(db, "users", user.uid, "tasks"), payload); else if (action === 'update') await updateDoc(doc(db, "users", user.uid, "tasks", id), payload); else if (action === 'delete') await deleteDoc(doc(db, "users", user.uid, "tasks", id)); } catch (error) { console.error("Firestore Error:", error); } finally { setTimeout(hideSavingIndicator, 500); } }
        
        function refreshDynamicContent() { const filteredTasks = getFilteredTasks(); if (currentView === 'list') renderListView(filteredTasks); else renderCalendarView(); updateDashboard(allTasks); updateRelativeTimes(); }
        function getFilteredTasks() { const searchFiltered = allTasks.filter(task => task.text.toLowerCase().includes(currentSearchTerm.toLowerCase())); if (currentCategoryFilter === 'All') return searchFiltered; return searchFiltered.filter(task => task.category === currentCategoryFilter); }

        function renderListView(tasks) {
            DOMElements.taskListContainer.innerHTML = '';
            if (tasks.length === 0) { DOMElements.taskListContainer.innerHTML = `<div class="text-center text-gray-500 py-10"><h3 class="text-xl font-semibold">No hay tareas que mostrar</h3><p>Intenta añadir una tarea o cambia los filtros.</p></div>`; return; }
            const groupedTasks = tasks.reduce((acc, task) => { const category = task.category || 'Otro'; if (!acc[category]) acc[category] = []; acc[category].push(task); return acc; }, {});
            Object.keys(groupedTasks).sort().forEach(category => {
                const tasksInCategory = groupedTasks[category].sort((a, b) => new Date(`${a.date}T${a.time || '00:00'}`) - new Date(`${b.date}T${b.time || '00:00'}`));
                const categorySection = document.createElement('div'); categorySection.className = 'bg-white p-6 rounded-2xl shadow-xl animate-fade-in';
                categorySection.innerHTML = `<h3 class="text-2xl font-semibold mb-4 border-b pb-2">${category}</h3><ul class="space-y-3">${tasksInCategory.map(createTaskElement).join('')}</ul>`;
                DOMElements.taskListContainer.appendChild(categorySection);
            });
            attachDynamicListeners(DOMElements.taskListContainer);
        }
        
        function updateDashboard(tasks) {
            const pendingTasks = tasks.filter(t => !t.completed); const now = new Date();
            const upcomingTasks = pendingTasks.map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59:59'}`) })).filter(t => t.dateTime >= now).sort((a, b) => a.dateTime - b.dateTime);
            if (upcomingTasks.length > 0) { const nextTask = upcomingTasks[0]; DOMElements.dashboardNextTask.innerHTML = `<p class="font-bold text-lg text-indigo-600 break-words">${nextTask.text}</p><p class="relative-time text-sm text-gray-500 mt-1" data-datetime="${nextTask.dateTime.toISOString()}"></p>`; } else { DOMElements.dashboardNextTask.innerHTML = `<p class="text-gray-400">¡Ninguna tarea pendiente!</p>`; }
            DOMElements.dashboardPending.textContent = pendingTasks.length;
            const overdueTasks = pendingTasks.map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59:59'}`) })).filter(t => t.dateTime < now).sort((a, b) => a.dateTime - b.dateTime);
            DOMElements.dashboardOverdueCount.textContent = overdueTasks.length;
            if (overdueTasks.length > 0) { const oldestOverdue = overdueTasks[0]; DOMElements.dashboardOverdue.innerHTML = `<p class="font-bold text-lg text-red-500 break-words">${oldestOverdue.text}</p><p class="relative-time text-sm text-gray-500 mt-1" data-datetime="${oldestOverdue.dateTime.toISOString()}"></p>`; } else { DOMElements.dashboardOverdue.innerHTML = `<p class="text-gray-400">¡Ninguna tarea vencida!</p>`; }
        }

        function createTaskElement(task) { const priorityMap = { high: { color: 'bg-red-500' }, medium: { color: 'bg-yellow-500' }, low: { color: 'bg-green-500' }}; const now = new Date(); const taskDateTime = new Date(`${task.date}T${task.time || '23:59:59'}`); let urgencyClass = !task.completed && taskDateTime < now ? 'overdue' : (!task.completed && now.toDateString() === taskDateTime.toDateString() ? 'today' : ''); let timeDiffHtml = !task.completed ? `<span class="relative-time" data-datetime="${taskDateTime.toISOString()}"></span>` : ''; const subtasksHtml = task.subtasks.map((sub, index) => `<div class="flex items-center gap-2 ml-4"><input type="checkbox" id="subtask-${task.id}-${index}" data-subtask-index="${index}" class="subtask-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500" ${sub.completed ? 'checked' : ''}><label for="subtask-${task.id}-${index}" class="text-sm ${sub.completed ? 'subtask-completed' : ''}">${sub.text}</label></div>`).join(''); const subtaskProgress = task.subtasks.length > 0 ? `(${task.subtasks.filter(s=>s.completed).length}/${task.subtasks.length})` : ''; const categoryClass = categoryMap[task.category] || categoryMap['Otro']; return `<li class="task-item ${urgencyClass} p-4 rounded-lg border border-gray-200" data-id="${task.id}"><div class="flex items-start justify-between"><div class="flex items-start gap-3 flex-grow min-w-0"><input type="checkbox" class="task-checkbox h-5 w-5 rounded border-gray-300 text-indigo-600 flex-shrink-0 mt-1 focus:ring-indigo-500" ${task.completed ? 'checked' : ''}><div class="min-w-0"><div class="flex items-center gap-2 flex-wrap"><span class="priority-flag ${priorityMap[task.priority]?.color || 'bg-gray-400'}" title="Prioridad ${task.priority}"></span><span class="font-medium break-words ${task.completed ? 'completed' : ''}">${task.text}</span><span class="text-xs font-semibold px-2 py-1 rounded-full ${categoryClass}">${task.category}</span></div><div class="text-xs text-gray-500 mt-1 flex items-center gap-2 flex-wrap"><span>${taskDateTime.toLocaleDateString('es-ES', { day: 'numeric', month: 'short' })} ${task.time || ''}</span>${!task.completed ? `<span class="mx-1">•</span> ${timeDiffHtml}` : ''}</div></div></div><div class="flex items-center flex-shrink-0 ml-2"><button class="subtasks-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Subtareas">${subtaskProgress || `<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01"></path></svg>`}</button><button class="notes-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Notas"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg></button><button class="edit-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Editar"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path></svg></button><button class="delete-btn text-gray-500 hover:text-red-500 p-1.5 rounded-full hover:bg-gray-100" title="Eliminar"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg></button></div></div><div class="notes-container ml-8 border-l-2 pl-4 mt-2"><p class="text-sm text-gray-600 whitespace-pre-wrap">${task.notes || 'No hay notas.'}</p></div><div class="subtask-container ml-8 border-l-2 pl-4 mt-2 space-y-2">${subtasksHtml}<div class="flex gap-2 pt-2"><input type="text" class="new-subtask-input bg-gray-100 rounded px-2 py-1 text-sm flex-grow border focus:outline-none focus:ring-1 focus:ring-indigo-500" placeholder="Nueva subtarea..."><button class="add-subtask-btn bg-indigo-500 hover:bg-indigo-600 text-white text-xs px-2.5 rounded">+</button></div></div></li>`; }
        
        function renderCalendarView() {
            DOMElements.calendarGrid.innerHTML = '';
            const m = currentCalendarDate.getMonth(), y = currentCalendarDate.getFullYear();
            DOMElements.calendarMonthYear.textContent = new Date(y, m).toLocaleDateString('es-ES', { month: 'long', year: 'numeric' });
            const firstDay = new Date(y, m, 1).getDay(), daysInMonth = new Date(y, m + 1, 0).getDate();
            for (let i = 0; i < firstDay; i++) DOMElements.calendarGrid.insertAdjacentHTML('beforeend', `<div class="calendar-day other-month"></div>`);
            for (let d = 1; d <= daysInMonth; d++) {
                const dayEl = document.createElement('div');
                dayEl.className = 'calendar-day'; dayEl.dataset.day = d;
                const today = new Date();
                if (d === today.getDate() && m === today.getMonth() && y === today.getFullYear()) dayEl.classList.add('is-today');
                const headerEl = document.createElement('div'); headerEl.className = 'calendar-day-header'; headerEl.textContent = d;
                dayEl.appendChild(headerEl);
                const tasksOnDay = allTasks.filter(t => { const td = new Date(t.date + 'T00:00:00'); return td.getDate() === d && td.getMonth() === m && td.getFullYear() === y; });
                const tasksContainer = document.createElement('div'); tasksContainer.className = 'calendar-tasks-container';
                tasksOnDay.slice(0, 2).forEach(task => {
                    const taskEl = document.createElement('div');
                    taskEl.className = `calendar-task-item ${categoryMap[task.category] || categoryMap['Otro']} pointer-events-none`;
                    taskEl.textContent = task.text;
                    tasksContainer.appendChild(taskEl);
                });
                if (tasksOnDay.length > 2) {
                    const moreEl = document.createElement('div');
                    moreEl.className = 'text-xs text-gray-500 mt-1 pl-1.5 pointer-events-none';
                    moreEl.textContent = `+${tasksOnDay.length - 2} más`;
                    tasksContainer.appendChild(moreEl);
                }
                dayEl.appendChild(tasksContainer);
                DOMElements.calendarGrid.appendChild(dayEl);
            }
        }
        
        function setupEventListeners() { DOMElements.openAddTaskModalBtn.addEventListener('click', showAddTaskModal); DOMElements.cancelAddTaskBtn.addEventListener('click', hideAddTaskModal); DOMElements.addTaskForm.addEventListener('submit', handleAddTask); DOMElements.searchInput.addEventListener('input', e => { currentSearchTerm = e.target.value; refreshDynamicContent(); }); DOMElements.categoryFilters.addEventListener('click', handleCategoryFilter); DOMElements.viewListBtn.addEventListener('click', () => switchView('list')); DOMElements.viewCalendarBtn.addEventListener('click', () => switchView('calendar')); DOMElements.cancelDeleteBtn.addEventListener('click', hideConfirmModal); DOMElements.confirmDeleteBtn.addEventListener('click', () => { if (taskToDeleteId) { crudOperation('delete', { id: taskToDeleteId }); hideConfirmModal(); } }); DOMElements.cancelEditBtn.addEventListener('click', hideEditModal); DOMElements.editTaskForm.addEventListener('submit', handleEditTask); DOMElements.prevMonthBtn.addEventListener('click', () => { currentCalendarDate.setMonth(currentCalendarDate.getMonth() - 1); renderCalendarView(); }); DOMElements.nextMonthBtn.addEventListener('click', () => { currentCalendarDate.setMonth(currentCalendarDate.getMonth() + 1); renderCalendarView(); }); DOMElements.calendarGrid.addEventListener('click', handleCalendarDayClick); DOMElements.calendarModalCloseBtn.addEventListener('click', () => DOMElements.calendarDayModal.classList.add('hidden')); }
        function attachDynamicListeners(container) { container.addEventListener('click', e => { const taskLi = e.target.closest('li.task-item'); if (!taskLi) return; const taskId = taskLi.dataset.id; if (e.target.closest('.task-checkbox')) crudOperation('update', { id: taskId, completed: e.target.closest('.task-checkbox').checked }); else if (e.target.closest('.edit-btn')) showEditModal(taskId); else if (e.target.closest('.delete-btn')) showConfirmModal(taskId); else if (e.target.closest('.notes-btn')) taskLi.querySelector('.notes-container').classList.toggle('open'); else if (e.target.closest('.subtasks-btn')) taskLi.querySelector('.subtask-container').classList.toggle('open'); else if (e.target.closest('.add-subtask-btn')) { const input = taskLi.querySelector('.new-subtask-input'); if (input.value.trim()) handleAddSubtask(taskId, input.value.trim()); input.value = ''; } else if (e.target.closest('.subtask-checkbox')) handleSubtaskToggle(taskId, parseInt(e.target.closest('.subtask-checkbox').dataset.subtaskIndex, 10)); }); }
        
        async function handleAddTask(e) { e.preventDefault(); const { addTaskText, addTaskDate, addTaskTime, addTaskCategory, addTaskPriority } = DOMElements; if (addTaskText.value.trim() && addTaskDate.value) { await crudOperation('add', { text: addTaskText.value.trim(), date: addTaskDate.value, time: addTaskTime.value, category: addTaskCategory.value, priority: addTaskPriority.value, completed: false, notes: '', subtasks: [] }); hideAddTaskModal(); showFlashMessage(DOMElements.taskSuccessMessage, "¡Tarea añadida exitosamente!"); } }
        async function handleEditTask(e) { e.preventDefault(); const task = allTasks.find(t => t.id === taskToEditId); if (task) { await crudOperation('update', { ...task, id: taskToEditId, text: DOMElements.editTaskText.value, date: DOMElements.editTaskDate.value, time: DOMElements.editTaskTime.value, category: DOMElements.editTaskCategory.value, priority: DOMElements.editTaskPriority.value, notes: DOMElements.editTaskNotes.value }); hideEditModal(); } }
        function handleCategoryFilter(e) { const target = e.target.closest('.category-filter-btn'); if (target) { currentCategoryFilter = target.dataset.category; styleCategoryFilters(); refreshDynamicContent(); } }
        function handleAddSubtask(taskId, text) { const task = allTasks.find(t => t.id === taskId); if (task) crudOperation('update', { id: taskId, subtasks: [...task.subtasks, { text, completed: false }] }); }
        function handleSubtaskToggle(taskId, index) { const task = allTasks.find(t => t.id === taskId); if (task) crudOperation('update', { id: taskId, subtasks: task.subtasks.map((s, i) => i === index ? { ...s, completed: !s.completed } : s) }); }
        function handleCalendarDayClick(e) { const dayEl = e.target.closest('.calendar-day:not(.other-month)'); if (!dayEl) return; const day = parseInt(dayEl.dataset.day, 10); const date = new Date(currentCalendarDate.getFullYear(), currentCalendarDate.getMonth(), day); const tasks = allTasks.filter(t => new Date(t.date + 'T00:00:00').toDateString() === date.toDateString()); DOMElements.calendarModalTitle.textContent = `Tareas para ${date.toLocaleDateString('es-ES', { dateStyle: 'full' })}`; DOMElements.calendarModalTasks.innerHTML = tasks.length ? tasks.map(t => `<div class="p-2 rounded-md ${t.completed ? 'bg-gray-100' : 'bg-blue-50'} flex items-center gap-2"><div class="w-2 h-2 rounded-full ${categoryMap[t.category].split(' ')[0]}"></div><span class="${t.completed ? 'completed' : ''}">${t.text}</span></div>`).join('') : '<p class="text-gray-500">No hay tareas para este día.</p>'; DOMElements.calendarDayModal.classList.remove('hidden'); }

        function switchView(view) { const isList = view === 'list'; DOMElements.listViewContainer.classList.toggle('hidden', !isList); DOMElements.categoryFilters.classList.toggle('hidden', !isList); DOMElements.calendarViewContainer.classList.toggle('hidden', isList); DOMElements.viewListBtn.classList.toggle('bg-indigo-600', isList); DOMElements.viewListBtn.classList.toggle('text-white', isList); DOMElements.viewCalendarBtn.classList.toggle('bg-indigo-600', !isList); DOMElements.viewCalendarBtn.classList.toggle('text-white', !isList); refreshDynamicContent(); }
        function showAddTaskModal() { DOMElements.addTaskForm.reset(); setSmartDefaults(); DOMElements.addTaskModal.classList.remove('hidden'); DOMElements.addTaskText.focus(); }
        function hideAddTaskModal() { DOMElements.addTaskModal.classList.add('hidden'); }
        function showEditModal(taskId) { taskToEditId = taskId; const task = allTasks.find(t => t.id === taskId); if(task) { DOMElements.editTaskText.value = task.text; DOMElements.editTaskDate.value = task.date; DOMElements.editTaskTime.value = task.time || ''; DOMElements.editTaskCategory.value = task.category; DOMElements.editTaskPriority.value = task.priority || 'medium'; DOMElements.editTaskNotes.value = task.notes || ''; DOMElements.editModal.classList.remove('hidden'); } }
        function hideEditModal() { DOMElements.editModal.classList.add('hidden'); taskToEditId = null; }
        function showConfirmModal(taskId) { taskToDeleteId = taskId; DOMElements.confirmModal.classList.remove('hidden'); }
        function hideConfirmModal() { taskToDeleteId = null; DOMElements.confirmModal.classList.add('hidden'); }
        function getGreeting() { const h = new Date().getHours(); if (h < 12) return 'Buenos días'; if (h < 19) return 'Buenas tardes'; return 'Buenas noches'; }
        function getInitials(email) { if (!email) return '?'; const p = email.split('@')[0].split(/[._-]/); return p.length > 1 ? (p[0][0] + p[1][0]) : email[0]; }
        function formatTimeDifference(isoDate) { const rtf = new Intl.RelativeTimeFormat('es', { numeric: 'auto' }); const diffSeconds = (new Date(isoDate).getTime() - Date.now()) / 1000; const diffDays = diffSeconds / 86400; if (Math.abs(diffDays) >= 1) return rtf.format(Math.round(diffDays), 'day'); const diffHours = diffSeconds / 3600; if (Math.abs(diffHours) >= 1) return rtf.format(Math.round(diffHours), 'hour'); const diffMinutes = diffSeconds / 60; if (Math.abs(diffMinutes) >= 1) return rtf.format(Math.round(diffMinutes), 'minute'); return 'ahora'; }
        function updateRelativeTimes() { document.querySelectorAll('.relative-time').forEach(el => { const isoDate = el.dataset.datetime; if (isoDate) { el.textContent = formatTimeDifference(isoDate); const diffSeconds = (new Date(isoDate).getTime() - Date.now()) / 1000; el.className = 'relative-time'; if (diffSeconds < 0) el.classList.add('text-red-600', 'font-medium'); else if (diffSeconds < 172800) el.classList.add('text-orange-600', 'font-medium'); else el.classList.add('text-gray-500'); } }); }
        function showFlashMessage(element, message) { element.textContent = message; element.classList.remove('hidden'); setTimeout(() => element.classList.add('hidden'), 3000); }
        function clearUI() { DOMElements.taskListContainer.innerHTML = ''; DOMElements.dashboardPending.textContent = '0'; DOMElements.dashboardNextTask.innerHTML = `<p class="text-gray-400">¡Ninguna tarea pendiente!</p>`; DOMElements.dashboardOverdue.innerHTML = `<p class="text-gray-400">¡Ninguna tarea vencida!</p>`; DOMElements.dashboardOverdueCount.textContent = '0'; }
        function setSmartDefaults() { DOMElements.addTaskDate.value = new Date().toISOString().split('T')[0]; const now = new Date(); DOMElements.addTaskTime.value = `${String((now.getHours() + 1) % 24).padStart(2, '0')}:00`; }
        function showSavingIndicator() { DOMElements.savingIndicator.classList.remove('hidden'); }
        function hideSavingIndicator() { DOMElements.savingIndicator.classList.add('hidden'); }
        function styleCategoryFilters() {
            document.querySelectorAll('.category-filter-btn').forEach(btn => {
                const category = btn.dataset.category;
                btn.className = 'category-filter-btn text-sm font-medium px-3 py-1 rounded-full'; // Reset
                if (category === currentCategoryFilter) {
                    if (category === 'All') btn.classList.add('bg-indigo-600', 'text-white');
                    else btn.className += ` ${categoryActiveMap[category]}`;
                } else {
                    if (category === 'All') btn.classList.add('bg-gray-200', 'text-gray-800');
                    else btn.className += ` ${categoryMap[category]}`;
                }
            });
        }
        
        DOMElements.loginForm.addEventListener('submit', async (e) => { e.preventDefault(); DOMElements.authError.textContent = ''; DOMElements.loader.classList.remove('hidden'); const email = e.target.elements['login-email'].value; const password = e.target.elements['login-password'].value; try { await signInWithEmailAndPassword(auth, email, password); } catch (error) { DOMElements.authError.textContent = 'El correo o la contraseña son incorrectos.'; } finally { DOMElements.loader.classList.add('hidden'); } });
        DOMElements.logoutBtn.addEventListener('click', () => { signOut(auth).then(() => { DOMElements.loginForm.reset(); showFlashMessage(DOMElements.logoutSuccessMessage, "¡Sesión cerrada exitosamente!"); }); });
        
        setupEventListeners();
        styleCategoryFilters();
    </script>
</body>
</html>
