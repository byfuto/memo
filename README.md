<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Webデザイナー向けAIメモ帳</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: "Inter", sans-serif;
            background-color: #f0f2f5; /* Light gray background */
        }
        /* Custom scrollbar for better aesthetics */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #e2e8f0; /* Tailwind gray-200 */
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb {
            background: #94a3b8; /* Tailwind gray-400 */
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #64748b; /* Tailwind gray-500 */
        }

        /* Content editable styling */
        [contenteditable]:focus {
            outline: none;
        }
        [contenteditable] img {
            max-width: 100%;
            height: auto;
            display: block; /* Prevents extra space below image */
            margin: 8px 0;
            border-radius: 8px;
        }
        [contenteditable] ul {
            list-style-type: disc;
            padding-left: 24px;
        }
        [contenteditable] ol {
            list-style-type: decimal;
            padding-left: 24px;
        }
        /* Basic heading styles for readability */
        [contenteditable] h1 { font-size: 2em; font-weight: bold; margin-top: 1em; margin-bottom: 0.5em; }
        [contenteditable] h2 { font-size: 1.5em; font-weight: bold; margin-top: 1em; margin-bottom: 0.5em; }
        [contenteditable] h3 { font-size: 1.2em; font-weight: bold; margin-top: 1em; margin-bottom: 0.5em; }
        [contenteditable] p { margin-bottom: 0.5em; }

        /* Modal specific styles */
        .modal {
            display: none; /* Hidden by default */
            position: fixed; /* Stay in place */
            z-index: 1000; /* Sit on top */
            left: 0;
            top: 0;
            width: 100%; /* Full width */
            height: 100%; /* Full height */
            overflow: auto; /* Enable scroll if needed */
            background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
            backdrop-filter: blur(5px); /* Blur background */
        }
        .modal-content {
            background-color: #fefefe;
            margin: 15% auto; /* 15% from the top and centered */
            padding: 20px;
            border: 1px solid #888;
            width: 80%; /* Could be more or less, depending on screen size */
            max-width: 600px;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            animation: fadeIn 0.3s ease-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .close-button {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
        }
        .close-button:hover,
        .close-button:focus {
            color: #000;
            text-decoration: none;
            cursor: pointer;
        }
    </style>
</head>
<body class="flex h-screen overflow-hidden">
    <!-- Main application container -->
    <div class="flex flex-1">
        <!-- Sidebar for notes list -->
        <div class="w-1/4 bg-white border-r border-gray-200 flex flex-col shadow-lg">
            <div class="p-4 border-b border-gray-200">
                <h1 class="text-2xl font-bold text-blue-600 mb-4">AIメモ帳</h1>
                <input type="text" id="search-input" placeholder="キーワード検索..." class="w-full p-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200">
                <button id="new-note-btn" class="mt-4 w-full bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded-md shadow-md transition duration-200">
                    新しいメモを作成
                </button>
            </div>
            <div id="notes-list" class="flex-1 overflow-y-auto p-4 space-y-2">
                <!-- Notes will be loaded here -->
            </div>
        </div>

        <!-- Main content area for note editing -->
        <div class="flex-1 flex flex-col bg-gray-50 p-6 overflow-hidden">
            <div class="bg-white rounded-lg shadow-xl flex-1 flex flex-col overflow-hidden">
                <!-- Note title input -->
                <div class="p-4 border-b border-gray-200">
                    <input type="text" id="note-title" placeholder="メモのタイトル" class="w-full text-3xl font-bold text-gray-800 focus:outline-none focus:ring-0">
                </div>

                <!-- Toolbar for text formatting and file management -->
                <div class="flex items-center space-x-2 p-4 border-b border-gray-200 bg-gray-50">
                    <button id="bold-btn" class="p-2 rounded-md hover:bg-gray-200 transition duration-200" title="太字">
                        <strong>B</strong>
                    </button>
                    <button id="italic-btn" class="p-2 rounded-md hover:bg-gray-200 transition duration-200" title="斜体">
                        <em>I</em>
                    </button>
                    <button id="ul-btn" class="p-2 rounded-md hover:bg-gray-200 transition duration-200" title="箇条書き">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path></svg>
                    </button>
                    <button id="ol-btn" class="p-2 rounded-md hover:bg-gray-200 transition duration-200" title="番号付きリスト">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h.01M3 6h.01M3 14h.01M3 18h.01M7 6h10l4 4-4 4H7l-4-4 4-4z"></path></svg>
                    </button>
                    <input type="file" id="image-upload" accept="image/*" class="hidden">
                    <button id="add-image-btn" class="p-2 rounded-md hover:bg-gray-200 transition duration-200" title="画像を挿入">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                    </button>

                    <!-- AI features -->
                    <div class="ml-auto flex space-x-2">
                        <button id="generate-tags-btn" class="bg-purple-500 hover:bg-purple-600 text-white font-semibold py-1.5 px-3 rounded-md shadow-md transition duration-200 text-sm" title="AIでタグを生成">
                            タグ生成 (AI)
                        </button>
                        <button id="summarize-btn" class="bg-green-500 hover:bg-green-600 text-white font-semibold py-1.5 px-3 rounded-md shadow-md transition duration-200 text-sm" title="AIで要約">
                            要約 (AI)
                        </button>
                    </div>
                </div>

                <!-- Note content editable area -->
                <div id="note-content" contenteditable="true" class="flex-1 p-4 overflow-y-auto text-gray-700 leading-relaxed focus:outline-none">
                    <!-- Note content will be edited here -->
                </div>

                <!-- Tags display area -->
                <div id="tags-display" class="p-4 border-t border-gray-200 bg-gray-50 flex flex-wrap gap-2">
                    <!-- Generated tags will appear here -->
                </div>

                <!-- Action buttons -->
                <div class="p-4 border-t border-gray-200 bg-gray-100 flex justify-end space-x-3">
                    <button id="save-note-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded-md shadow-md transition duration-200">
                        保存
                    </button>
                    <button id="delete-note-btn" class="bg-red-500 hover:bg-red-600 text-white font-semibold py-2 px-4 rounded-md shadow-md transition duration-200">
                        削除
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Loading Modal -->
    <div id="loading-modal" class="modal">
        <div class="modal-content text-center">
            <div class="flex justify-center items-center">
                <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500"></div>
            </div>
            <p class="mt-4 text-lg text-gray-700">AIが処理中です...</p>
        </div>
    </div>

    <!-- Summary Modal -->
    <div id="summary-modal" class="modal">
        <div class="modal-content">
            <span class="close-button" id="close-summary-modal">&times;</span>
            <h2 class="text-xl font-bold mb-4">AIによる要約</h2>
            <div id="summary-content" class="text-gray-700 max-h-96 overflow-y-auto p-2 border border-gray-200 rounded-md bg-gray-50">
                <!-- Summary will be displayed here -->
            </div>
            <div class="mt-4 flex justify-end space-x-2">
                <button id="copy-summary-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-1.5 px-3 rounded-md shadow-md transition duration-200 text-sm">
                    要約をコピー
                </button>
            </div>
        </div>
    </div>

    <script>
        // Define __app_id for consistency, though not strictly used for local-only app
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'web-designer-notepad-default';

        // IndexedDB setup
        const DB_NAME = 'WebDesignerNotepadDB';
        const DB_VERSION = 1;
        const STORE_NAME = 'notes';
        let db;

        // UI Elements
        const notesList = document.getElementById('notes-list');
        const searchInput = document.getElementById('search-input');
        const newNoteBtn = document.getElementById('new-note-btn');
        const noteTitleInput = document.getElementById('note-title');
        const noteContentDiv = document.getElementById('note-content');
        const saveNoteBtn = document.getElementById('save-note-btn');
        const deleteNoteBtn = document.getElementById('delete-note-btn');
        const boldBtn = document.getElementById('bold-btn');
        const italicBtn = document.getElementById('italic-btn');
        const ulBtn = document.getElementById('ul-btn');
        const olBtn = document.getElementById('ol-btn');
        const addImageBtn = document.getElementById('add-image-btn');
        const imageUploadInput = document.getElementById('image-upload');
        const tagsDisplay = document.getElementById('tags-display');
        const generateTagsBtn = document.getElementById('generate-tags-btn');
        const summarizeBtn = document.getElementById('summarize-btn');

        // Modals
        const loadingModal = document.getElementById('loading-modal');
        const summaryModal = document.getElementById('summary-modal');
        const closeSummaryModalBtn = document.getElementById('close-summary-modal');
        const summaryContentDiv = document.getElementById('summary-content');
        const copySummaryBtn = document.getElementById('copy-summary-btn');

        let currentNoteId = null; // Stores the ID of the currently active note

        // --- IndexedDB Functions ---

        /**
         * Initializes the IndexedDB database.
         * @returns {Promise<IDBDatabase>} A promise that resolves with the database instance.
         */
        function openDatabase() {
            return new Promise((resolve, reject) => {
                const request = indexedDB.open(DB_NAME, DB_VERSION);

                request.onupgradeneeded = (event) => {
                    db = event.target.result;
                    if (!db.objectStoreNames.contains(STORE_NAME)) {
                        db.createObjectStore(STORE_NAME, { keyPath: 'id', autoIncrement: true });
                    }
                };

                request.onsuccess = (event) => {
                    db = event.target.result;
                    resolve(db);
                };

                request.onerror = (event) => {
                    console.error('IndexedDB error:', event.target.error);
                    reject(event.target.error);
                };
            });
        }

        /**
         * Adds a new note to the database.
         * @param {object} note - The note object to add.
         * @returns {Promise<number>} A promise that resolves with the ID of the new note.
         */
        function addNote(note) {
            return new Promise((resolve, reject) => {
                const transaction = db.transaction([STORE_NAME], 'readwrite');
                const store = transaction.objectStore(STORE_NAME);
                const request = store.add(note);

                request.onsuccess = (event) => resolve(event.target.result);
                request.onerror = (event) => reject(event.target.error);
            });
        }

        /**
         * Retrieves all notes from the database.
         * @returns {Promise<Array<object>>} A promise that resolves with an array of note objects.
         */
        function getNotes() {
            return new Promise((resolve, reject) => {
                const transaction = db.transaction([STORE_NAME], 'readonly');
                const store = transaction.objectStore(STORE_NAME);
                const request = store.getAll();

                request.onsuccess = (event) => {
                    // Sort notes by lastModifiedDate in descending order
                    const sortedNotes = event.target.result.sort((a, b) => {
                        return new Date(b.lastModifiedDate) - new Date(a.lastModifiedDate);
                    });
                    resolve(sortedNotes);
                };
                request.onerror = (event) => reject(event.target.error);
            });
        }

        /**
         * Updates an existing note in the database.
         * @param {object} note - The note object to update (must contain 'id').
         * @returns {Promise<void>} A promise that resolves when the update is complete.
         */
        function updateNote(note) {
            return new Promise((resolve, reject) => {
                const transaction = db.transaction([STORE_NAME], 'readwrite');
                const store = transaction.objectStore(STORE_NAME);
                const request = store.put(note);

                request.onsuccess = () => resolve();
                request.onerror = (event) => reject(event.target.error);
            });
        }

        /**
         * Deletes a note from the database.
         * @param {number} id - The ID of the note to delete.
         * @returns {Promise<void>} A promise that resolves when the deletion is complete.
         */
        function deleteNote(id) {
            return new Promise((resolve, reject) => {
                const transaction = db.transaction([STORE_NAME], 'readwrite');
                const store = transaction.objectStore(STORE_NAME);
                const request = store.delete(id);

                request.onsuccess = () => resolve();
                request.onerror = (event) => reject(event.target.error);
            });
        }

        // --- UI Rendering & Event Handlers ---

        /**
         * Displays notes in the sidebar.
         * @param {Array<object>} notes - An array of note objects to display.
         * @param {string} [searchTerm=''] - The current search term for highlighting.
         */
        function displayNotes(notes, searchTerm = '') {
            notesList.innerHTML = '';
            if (notes.length === 0) {
                notesList.innerHTML = '<p class="text-gray-500 text-center mt-8">メモがありません。</p>';
                return;
            }

            notes.forEach(note => {
                const noteElement = document.createElement('div');
                noteElement.className = 'note-item bg-gray-100 hover:bg-gray-200 p-3 rounded-md cursor-pointer transition duration-200';
                noteElement.dataset.id = note.id;

                let title = note.title || '無題のメモ';
                if (searchTerm) {
                    const regex = new RegExp(`(${searchTerm})`, 'gi');
                    title = title.replace(regex, '<mark class="bg-yellow-200 rounded">$1</mark>');
                }

                noteElement.innerHTML = `
                    <h3 class="font-semibold text-gray-800 truncate">${title}</h3>
                    <p class="text-xs text-gray-500">${new Date(note.lastModifiedDate).toLocaleString()}</p>
                `;
                noteElement.addEventListener('click', () => loadNote(note.id));
                notesList.appendChild(noteElement);
            });
        }

        /**
         * Loads a specific note into the editor.
         * @param {number} id - The ID of the note to load.
         */
        async function loadNote(id) {
            try {
                const transaction = db.transaction([STORE_NAME], 'readonly');
                const store = transaction.objectStore(STORE_NAME);
                const request = store.get(id);

                request.onsuccess = (event) => {
                    const note = event.target.result;
                    if (note) {
                        currentNoteId = note.id;
                        noteTitleInput.value = note.title || '';
                        // Apply URL linking when displaying content
                        noteContentDiv.innerHTML = autoLinkUrls(note.content || '');
                        displayTags(note.tags || []); // Display existing tags
                        deleteNoteBtn.classList.remove('hidden'); // Show delete button for existing notes
                        saveNoteBtn.textContent = '更新'; // Change save button text to update
                        // Highlight the selected note in the sidebar
                        document.querySelectorAll('.note-item').forEach(item => {
                            item.classList.remove('bg-blue-100', 'border-l-4', 'border-blue-500');
                        });
                        const selectedItem = document.querySelector(`.note-item[data-id="${id}"]`);
                        if (selectedItem) {
                            selectedItem.classList.add('bg-blue-100', 'border-l-4', 'border-blue-500');
                        }
                    }
                };
                request.onerror = (event) => console.error('Error loading note:', event.target.error);
            } catch (error) {
                console.error('Failed to load note:', error);
            }
        }

        /**
         * Creates a new empty note in the editor.
         */
        function newNote() {
            currentNoteId = null;
            noteTitleInput.value = '';
            noteContentDiv.innerHTML = '';
            tagsDisplay.innerHTML = ''; // Clear tags
            deleteNoteBtn.classList.add('hidden'); // Hide delete button for new notes
            saveNoteBtn.textContent = '保存'; // Change save button text to save
            document.querySelectorAll('.note-item').forEach(item => {
                item.classList.remove('bg-blue-100', 'border-l-4', 'border-blue-500');
            });
            noteTitleInput.focus(); // Focus on title for new note
        }

        /**
         * Saves or updates the current note.
         */
        async function saveOrUpdateNote() {
            const title = noteTitleInput.value.trim();
            const content = noteContentDiv.innerHTML; // Get full HTML content
            const tags = Array.from(tagsDisplay.querySelectorAll('.tag')).map(tagEl => tagEl.textContent.replace(/^#/, '')); // Extract text from tags
            const now = new Date().toISOString();

            const note = {
                title: title || '無題のメモ',
                content: content,
                tags: tags,
                lastModifiedDate: now
            };

            try {
                if (currentNoteId) {
                    note.id = currentNoteId;
                    await updateNote(note);
                    console.log('Note updated:', note.id);
                } else {
                    const newId = await addNote(note);
                    currentNoteId = newId; // Set currentNoteId to the newly added note's ID
                    console.log('Note saved:', newId);
                }
                await refreshNotesList(); // Refresh sidebar after save/update
                loadNote(currentNoteId); // Reload the note to show updated content/tags
            } catch (error) {
                console.error('Failed to save/update note:', error);
                alert('メモの保存に失敗しました。');
            }
        }

        /**
         * Deletes the currently loaded note.
         */
        async function deleteCurrentNote() {
            if (currentNoteId && confirm('このメモを本当に削除しますか？')) {
                try {
                    await deleteNote(currentNoteId);
                    console.log('Note deleted:', currentNoteId);
                    newNote(); // Clear editor after deletion
                    await refreshNotesList(); // Refresh sidebar
                } catch (error) {
                    console.error('Failed to delete note:', error);
                    alert('メモの削除に失敗しました。');
                }
            }
        }

        /**
         * Refreshes the list of notes in the sidebar based on current search input.
         */
        async function refreshNotesList() {
            const allNotes = await getNotes();
            const searchTerm = searchInput.value.toLowerCase();

            const filteredNotes = allNotes.filter(note =>
                (note.title && note.title.toLowerCase().includes(searchTerm)) ||
                (note.content && note.content.toLowerCase().includes(searchTerm)) ||
                (note.tags && note.tags.some(tag => tag.toLowerCase().includes(searchTerm)))
            );
            displayNotes(filteredNotes, searchTerm);
        }

        /**
         * Applies text formatting to the selected text.
         * @param {string} command - The document.execCommand command.
         * @param {string} [value=null] - The value for the command.
         */
        function formatText(command, value = null) {
            document.execCommand(command, false, value);
            noteContentDiv.focus(); // Keep focus on the editable area
        }

        /**
         * Handles image file selection and insertion.
         * @param {Event} event - The change event from the file input.
         */
        function handleImageUpload(event) {
            const file = event.target.files[0];
            if (file && file.type.startsWith('image/')) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    const imageUrl = e.target.result;
                    const img = `<img src="${imageUrl}" alt="Uploaded Image">`;
                    document.execCommand('insertHTML', false, img);
                    noteContentDiv.focus();
                };
                reader.readAsDataURL(file);
            } else {
                alert('画像ファイルを選択してください。');
            }
        }

        /**
         * Displays tags in the tags display area.
         * @param {Array<string>} tags - An array of tag strings.
         */
        function displayTags(tags) {
            tagsDisplay.innerHTML = '';
            tags.forEach(tag => {
                const tagEl = document.createElement('span');
                tagEl.className = 'tag bg-blue-100 text-blue-800 text-xs font-semibold px-2.5 py-0.5 rounded-full mr-2';
                tagEl.textContent = `#${tag}`;
                tagsDisplay.appendChild(tagEl);
            });
        }

        /**
         * Automatically converts URLs in text to clickable links.
         * Note: This is a display-only conversion. Actual storage keeps raw HTML.
         * @param {string} htmlContent - The HTML string to process.
         * @returns {string} The HTML string with auto-linked URLs.
         */
        function autoLinkUrls(htmlContent) {
            // Regex to find URLs (http/https) not already inside <a> tags
            const urlRegex = /(?<!<a[^>]*?>)(https?:\/\/[^\s"<]+)(?![^<]*?<\/a>)/g;
            return htmlContent.replace(urlRegex, '<a href="$1" target="_blank" class="text-blue-600 hover:underline">$1</a>');
        }

        // --- AI Integration (Gemini API) ---
        // IMPORTANT: For actual deployment, ensure your API key is managed securely.
        // For Canvas environment, the apiKey can be left as empty string.
        const GEMINI_API_KEY = ""; // Canvas will provide this at runtime
        const GEMINI_API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${GEMINI_API_KEY}`;

        /**
         * Calls Gemini API to generate tags from note content.
         */
        async function generateTagsFromAI() {
            const content = noteContentDiv.innerText.trim(); // Use innerText to get plain text
            if (!content) {
                alert('タグを生成するにはメモの内容を入力してください。');
                return;
            }

            showLoadingModal();
            try {
                const prompt = `このテキストから関連性の高いタグを5つ、カンマ区切りで生成してください。各タグは単一の単語または短いフレーズで、日本語でお願いします。例: デザイン, UX, コーディング\n\nテキスト:\n"${content}"`;
                const payload = {
                    contents: [{ role: "user", parts: [{ text: prompt }] }],
                    generationConfig: {
                        responseMimeType: "application/json",
                        responseSchema: {
                            type: "ARRAY",
                            items: { "type": "STRING" }
                        }
                    }
                };

                const response = await fetch(GEMINI_API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`API error: ${response.status} ${response.statusText}`);
                }

                const result = await response.json();
                console.log("Gemini Tags Result:", result);

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const jsonString = result.candidates[0].content.parts[0].text;
                    const tags = JSON.parse(jsonString);
                    if (Array.isArray(tags)) {
                        displayTags(tags);
                        // Optionally, save tags to the current note
                        if (currentNoteId) {
                            const transaction = db.transaction([STORE_NAME], 'readwrite');
                            const store = transaction.objectStore(STORE_NAME);
                            const request = store.get(currentNoteId);
                            request.onsuccess = async (event) => {
                                const note = event.target.result;
                                if (note) {
                                    note.tags = tags;
                                    note.lastModifiedDate = new Date().toISOString();
                                    await updateNote(note);
                                    console.log('Tags saved to note.');
                                    refreshNotesList(); // Refresh to update last modified date
                                }
                            };
                        }
                    } else {
                        alert('AIがタグを生成できませんでした。');
                    }
                } else {
                    alert('AIがタグを生成できませんでした。');
                }
            } catch (error) {
                console.error('Error generating tags with AI:', error);
                alert('タグ生成中にエラーが発生しました。');
            } finally {
                hideLoadingModal();
            }
        }

        /**
         * Calls Gemini API to summarize note content.
         */
        async function summarizeNoteWithAI() {
            const content = noteContentDiv.innerText.trim(); // Use innerText for plain text summarization
            if (!content) {
                alert('要約するにはメモの内容を入力してください。');
                return;
            }

            showLoadingModal();
            try {
                const prompt = `以下のテキストを、Webデザイナーが共有や要約に使いやすいように、簡潔な箇条書き形式で要約してください。重要なポイントを3〜5点に絞り、日本語でお願いします。\n\nテキスト:\n"${content}"`;
                const payload = {
                    contents: [{ role: "user", parts: [{ text: prompt }] }]
                };

                const response = await fetch(GEMINI_API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`API error: ${response.status} ${response.statusText}`);
                }

                const result = await response.json();
                console.log("Gemini Summary Result:", result);

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const summaryText = result.candidates[0].content.parts[0].text;
                    summaryContentDiv.innerHTML = summaryText.replace(/\n/g, '<br>'); // Display summary, convert newlines to <br>
                    summaryModal.style.display = 'block'; // Show summary modal
                } else {
                    alert('AIが要約を生成できませんでした。');
                }
            } catch (error) {
                console.error('Error summarizing with AI:', error);
                alert('要約中にエラーが発生しました。');
            } finally {
                hideLoadingModal();
            }
        }

        // --- Modal Functions ---
        function showLoadingModal() {
            loadingModal.style.display = 'block';
        }

        function hideLoadingModal() {
            loadingModal.style.display = 'none';
        }

        // --- Event Listeners ---
        window.onload = async () => {
            await openDatabase();
            await refreshNotesList();
            newNote(); // Start with a new empty note
        };

        newNoteBtn.addEventListener('click', newNote);
        saveNoteBtn.addEventListener('click', saveOrUpdateNote);
        deleteNoteBtn.addEventListener('click', deleteCurrentNote);
        searchInput.addEventListener('input', refreshNotesList);

        // Formatting buttons
        boldBtn.addEventListener('click', () => formatText('bold'));
        italicBtn.addEventListener('click', () => formatText('italic'));
        ulBtn.addEventListener('click', () => formatText('insertUnorderedList'));
        olBtn.addEventListener('click', () => formatText('insertOrderedList'));

        // Image upload
        addImageBtn.addEventListener('click', () => imageUploadInput.click());
        imageUploadInput.addEventListener('change', handleImageUpload);

        // AI buttons
        generateTagsBtn.addEventListener('click', generateTagsFromAI);
        summarizeBtn.addEventListener('click', summarizeNoteWithAI);

        // Summary Modal Close Button
        closeSummaryModalBtn.addEventListener('click', () => {
            summaryModal.style.display = 'none';
        });

        // Copy Summary Button
        copySummaryBtn.addEventListener('click', () => {
            const textToCopy = summaryContentDiv.innerText;
            // Use document.execCommand('copy') for better compatibility in iframes
            const tempTextArea = document.createElement('textarea');
            tempTextArea.value = textToCopy;
            document.body.appendChild(tempTextArea);
            tempTextArea.select();
            try {
                const successful = document.execCommand('copy');
                const msg = successful ? 'コピーしました！' : 'コピーに失敗しました。';
                alert(msg);
            } catch (err) {
                console.error('Copy failed:', err);
                alert('コピーに失敗しました。');
            }
            document.body.removeChild(tempTextArea);
        });

        // Close modals if clicked outside
        window.addEventListener('click', (event) => {
            if (event.target === loadingModal) {
                hideLoadingModal();
            }
            if (event.target === summaryModal) {
                summaryModal.style.display = 'none';
            }
        });

        // Auto-link URLs when content is edited (optional, can be heavy for large notes)
        // A more robust solution might be to do this on display only, as implemented in loadNote.
        // For real-time editing, it might interfere with user input.
        // For this simple app, we'll rely on loadNote for display linking.
    </script>
</body>
</html>
