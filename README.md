<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>도서 검색 및 대출 키오스크</title>
    <style>
        body {
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f4f4f4;
            margin: 0;
            padding: 20px;
            min-height: 100vh;
        }
        .container {
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 90%;
            max-width: 850px; /* Increased width for more content */
            text-align: center;
        }
        h1, h2, h3, h4 {
            color: #333;
        }
        .button-group button, .mode-content button, .admin-action-button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 10px 15px;
            margin: 5px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        .button-group button:hover, .mode-content button:hover, .admin-action-button:hover {
            background-color: #0056b3;
        }
        .exit-button { background-color: #6c757d; }
        .exit-button:hover { background-color: #545b62; }
        .delete-button { background-color: #dc3545; }
        .delete-button:hover { background-color: #c82333; }
        .suspend-button { background-color: #ffc107; color: #212529; }
        .suspend-button:hover { background-color: #e0a800; }
        .activate-button { background-color: #28a745; }
        .activate-button:hover { background-color: #218838; }
        .edit-button { background-color: #17a2b8; }
        .edit-button:hover { background-color: #138496; }

        .mode-content {
            margin-top: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 5px;
            background-color: #f9f9f9;
        }
        .hidden { display: none; }
        input[type="text"], input[type="number"], input[type="password"] {
            padding: 8px;
            margin: 5px 0 10px 0;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
            width: calc(100% - 22px); /* Default width */
        }
        .short-input {
             width: 150px;
             display: inline-block;
             margin-right: 10px;
        }
        label {
            display: block;
            margin-top: 10px;
            text-align: left;
            font-weight: bold;
        }
        #searchResults li, #adminBookList li, #adminCardList li {
            list-style: none;
            padding: 10px;
            border-bottom: 1px solid #eee;
            text-align: left;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
        }
        #adminBookList li > span { flex-basis: 75%; }
        #adminBookList li .list-item-actions { flex-basis: 23%; text-align: right;}
        #adminCardList li > span { flex-basis: 65%; }
        #adminCardList li .list-item-actions { flex-basis: 33%; text-align: right;}


        #searchResults li:last-child, #adminBookList li:last-child, #adminCardList li:last-child {
            border-bottom: none;
        }
        .available { color: green; }
        .borrowed { color: red; }
        .overdue { font-weight: bold; color: darkorange; }
        .suspended { color: orange; text-decoration: line-through;}
        .active { color: green; }
        .message { margin-top: 10px; padding: 10px; border-radius: 5px; }
        .success { background-color: #d4edda; color: #155724; }
        .error { background-color: #f8d7da; color: #721c24; }
        .info { background-color: #cce5ff; color: #004085; }
        .admin-section { margin-bottom: 20px; }
        .list-item-actions button {
            margin-left: 3px;
            padding: 5px 8px;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📚 도서관 키오스크</h1>

        <div class="button-group" id="mainMenu">
            <button onclick="showMode('borrowMode')">대출</button>
            <button onclick="showMode('returnMode')">반납</button>
            <button onclick="showMode('searchMode')">검색</button>
            <button onclick="showMode('adminAccessMode')">관리자 모드</button>
        </div>

        <div id="messageArea" class="message hidden"></div>

        <div id="searchMode" class="mode-content hidden">
            <h2>🔎 도서 검색</h2>
            <label for="searchInput">책 이름:</label>
            <input type="text" id="searchInput" placeholder="책 제목을 입력하세요">
            <button onclick="searchBook()">검색</button>
            <ul id="searchResults"></ul>
            <button onclick="goHome()" class="exit-button">메인 메뉴로</button>
        </div>

        <div id="borrowMode" class="mode-content hidden">
            <h2>📖 도서 대출</h2>
            <label for="borrowBookId">책 번호:</label>
            <input type="text" id="borrowBookId" placeholder="대출할 책 번호를 입력하세요">
            <div id="libraryCardInputSection" class="hidden">
                <label for="libraryCardNumberBorrow">대출증 번호:</label>
                <input type="text" id="libraryCardNumberBorrow" placeholder="대출증 번호를 입력하세요">
            </div>
            <button onclick="processBorrow()">대출</button>
            <button onclick="goHome()" class="exit-button">메인 메뉴로</button>
        </div>

        <div id="returnMode" class="mode-content hidden">
            <h2>↪️ 도서 반납</h2>
            <label for="returnBookId">책 번호:</label>
            <input type="text" id="returnBookId" placeholder="반납할 책 번호를 입력하세요">
            <button onclick="returnBook()">반납 확인</button>
            <button onclick="goHome()" class="exit-button">메인 메뉴로</button>
        </div>

        <div id="adminAccessMode" class="mode-content hidden">
            <h2>🔑 관리자 접속</h2>
            <label for="adminPassword">관리자 번호:</label>
            <input type="password" id="adminPassword" placeholder="관리자 번호를 입력하세요">
            <button onclick="accessAdminMode()">접속</button>
            <button onclick="goHome()" class="exit-button">메인 메뉴로</button>
        </div>

        <div id="adminMode" class="mode-content hidden">
            <h2>⚙️ 관리자 모드</h2>
            <div class="admin-section">
                <h3>📚 책 관리 (상태 확인 및 등록/삭제/수정)</h3>
                <label for="bookIdInput">책 번호(ID):</label>
                <input type="text" id="bookIdInput" placeholder="고유 책 번호">
                <label for="bookName">책 이름:</label>
                <input type="text" id="bookName" placeholder="책 이름">
                <label for="bookLocation">책 위치:</label>
                <input type="text" id="bookLocation" placeholder="예: 3층 A-2구역">
                <button onclick="addBook()">신규 책 등록</button>
                <h4>등록된 책 목록:</h4>
                <ul id="adminBookList"></ul>
            </div>
            <hr>
            <div class="admin-section">
                <h3>💳 대출증 관리</h3>
                <label for="libraryCardId">새 대출증 번호:</label>
                <input type="text" id="libraryCardId" placeholder="새 대출증 번호">
                <label for="libraryCardBorrowLimit">최대 대출 가능 권수:</label>
                <input type="number" id="libraryCardBorrowLimit" class="short-input" placeholder="예: 5" min="1" value="5">
                <button onclick="addLibraryCard()">대출증 등록</button>
                <h4>등록된 대출증 목록:</h4>
                <ul id="adminCardList"></ul>
            </div>
            <button onclick="exitAdminMode()" class="exit-button">관리자 모드 종료 (메인 메뉴로)</button>
        </div>
    </div>

    <script>
        const ADMIN_PIN = "1234";
        const DEFAULT_BORROW_LIMIT = 5;
        const DEFAULT_LOAN_PERIOD_DAYS = 14; // 기본 대출 기간 (일)

        // --- Helper Functions ---
        function formatDate(date) {
            const d = new Date(date);
            let month = '' + (d.getMonth() + 1);
            let day = '' + d.getDate();
            const year = d.getFullYear();
            if (month.length < 2) month = '0' + month;
            if (day.length < 2) day = '0' + day;
            return [year, month, day].join('-');
        }

        function addDays(date, days) {
            const result = new Date(date);
            result.setDate(result.getDate() + days);
            return result;
        }
        // --- Data Initialization & Migration ---
        // Book: { id: string, name: string, location: string, status: 'available'|'borrowed', borrowedByCard: string|null, dueDate: string|null }
        let books = JSON.parse(localStorage.getItem('books')) || [
            { id: "B001", name: "자바스크립트 완벽 가이드", location: "1층 B-3", status: "available", borrowedByCard: null, dueDate: null },
            { id: "B002", name: "HTML & CSS 기초", location: "1층 A-1", status: "borrowed", borrowedByCard: "L001", dueDate: formatDate(addDays(new Date(), -5)) }, // Example: borrowed 5 days ago
            { id: "B003", name: "데이터베이스 개론", location: "2층 C-5", status: "available", borrowedByCard: null, dueDate: null }
        ];
        // Card: { id: string, status: 'active'|'suspended', borrowLimit: number, borrowedCount: number, suspensionReason: string|null }
        let rawLibraryCards = JSON.parse(localStorage.getItem('libraryCards')) || [
            { id: "L001", status: "active", borrowLimit: 5, borrowedCount: 1, suspensionReason: null},
            { id: "L002", status: "suspended", borrowLimit: 3, borrowedCount: 0, suspensionReason: "manual"},
            { id: "L003", status: "active", borrowLimit: 7, borrowedCount: 0, suspensionReason: null}
        ];

        let libraryCards = rawLibraryCards.map(card => {
            const borrowedCount = books.filter(b => b.borrowedByCard === card.id && b.status === 'borrowed').length;
            return {
                id: card.id,
                status: card.status || 'active',
                borrowLimit: card.borrowLimit === undefined ? DEFAULT_BORROW_LIMIT : parseInt(card.borrowLimit, 10),
                borrowedCount: borrowedCount, // Always recalculate on load for consistency
                suspensionReason: card.suspensionReason || null
            };
        });
        saveLibraryCards(); // Save potentially migrated/recalculated data

        const modes = ['searchMode', 'borrowMode', 'returnMode', 'adminAccessMode', 'adminMode'];
        const messageArea = document.getElementById('messageArea');
        const searchResultsUl = document.getElementById('searchResults');
        const adminBookListUl = document.getElementById('adminBookList');
        const adminCardListUl = document.getElementById('adminCardList');
        const libraryCardInputSection = document.getElementById('libraryCardInputSection');

        function showMessage(text, type = "info") {
            messageArea.textContent = text;
            messageArea.className = `message ${type}`;
            messageArea.classList.remove('hidden');
            setTimeout(() => { messageArea.classList.add('hidden'); }, 4000);
        }

        function saveBooks() { localStorage.setItem('books', JSON.stringify(books)); }
        function saveLibraryCards() { localStorage.setItem('libraryCards', JSON.stringify(libraryCards)); }

        function showMode(modeId) {
            document.getElementById('mainMenu').classList.add('hidden');
            modes.forEach(id => document.getElementById(id).classList.add('hidden'));
            document.getElementById(modeId).classList.remove('hidden');
            messageArea.classList.add('hidden');

            if (modeId === 'adminMode') {
                checkOverdueBooksAndSuspendCards(); // Check for overdue books on entering admin mode
                renderAdminBookList();
                renderAdminCardList();
            }
            // Clear inputs for specific modes
            ['borrowBookId', 'libraryCardNumberBorrow', 'searchInput', 'returnBookId', 'adminPassword', 'bookIdInput', 'bookName', 'bookLocation', 'libraryCardId'].forEach(inputId => {
                const el = document.getElementById(inputId);
                if (el) el.value = '';
            });
             document.getElementById('libraryCardBorrowLimit').value = DEFAULT_BORROW_LIMIT;


            if (modeId === 'borrowMode') libraryCardInputSection.classList.add('hidden');
            if (modeId === 'searchMode') searchResultsUl.innerHTML = '';
        }

        function goHome() {
            modes.forEach(id => document.getElementById(id).classList.add('hidden'));
            document.getElementById('mainMenu').classList.remove('hidden');
            messageArea.classList.add('hidden');
        }

        // --- Date and Overdue Logic ---
        function checkOverdueBooksAndSuspendCards(cardToCheckId = null) {
            const today = formatDate(new Date());
            let cardSuspendedThisRun = false;

            books.forEach(book => {
                if (book.status === 'borrowed' && book.dueDate && book.dueDate < today) {
                    // Book is overdue
                    const card = libraryCards.find(c => c.id === book.borrowedByCard);
                    if (card && card.status === 'active') {
                        card.status = 'suspended';
                        card.suspensionReason = 'overdue';
                        showMessage(`대출증 ${card.id}이(가) '${book.name}' 책의 반납일(${book.dueDate}) 초과로 자동 정지되었습니다.`, "error");
                        cardSuspendedThisRun = true;
                    }
                }
            });
            if (cardSuspendedThisRun) saveLibraryCards();


            // If a specific card is being checked (e.g., before borrowing)
            if (cardToCheckId) {
                 const card = libraryCards.find(c => c.id === cardToCheckId);
                 if (card && card.status === 'suspended' && card.suspensionReason === 'overdue') {
                     showMessage(`대출증 ${card.id}은(는) 연체된 도서가 있어 정지 상태입니다. 먼저 연체 도서를 반납해주세요.`, "error");
                     return false; // Indicate card is suspended due to overdue
                 }
            }
            return true; // Card is okay or no specific card check
        }


        // --- 검색 모드 ---
        function searchBook() {
            const searchTerm = document.getElementById('searchInput').value.toLowerCase().trim();
            searchResultsUl.innerHTML = '';
            if (!searchTerm) { showMessage("검색어를 입력해주세요.", "error"); return; }
            const results = books.filter(book => book.name.toLowerCase().includes(searchTerm) || book.id.toLowerCase().includes(searchTerm));
            if (results.length === 0) { searchResultsUl.innerHTML = '<li>검색 결과가 없습니다.</li>'; return; }
            results.forEach(book => {
                const listItem = document.createElement('li');
                let bookInfo = `<span>이름: ${book.name} (ID: ${book.id}), 위치: ${book.location}, 상태: <span class="${book.status === 'available' ? 'available' : 'borrowed'}">${book.status === 'available' ? '대출 가능' : '대출 중'}</span>`;
                if (book.status === 'borrowed' && book.dueDate) {
                    const isOverdue = formatDate(new Date()) > book.dueDate;
                    bookInfo += `, 반납 예정일: <span class="${isOverdue ? 'overdue' : ''}">${book.dueDate}</span>`;
                    if(isOverdue) bookInfo += ` <span class="overdue">(연체)</span>`;
                }
                listItem.innerHTML = bookInfo + '</span>';
                searchResultsUl.appendChild(listItem);
            });
        }

        // --- 대출 모드 ---
        let currentBorrowBookInternalId = null; // Using actual book object ID for processing
        function processBorrow() {
            const bookIdInput = document.getElementById('borrowBookId'); // User inputs custom book ID
            const libraryCardNumberInput = document.getElementById('libraryCardNumberBorrow');

            if (!libraryCardInputSection.classList.contains('hidden')) { // 대출증 번호 입력 단계
                const libraryCardId = libraryCardNumberInput.value.trim();
                if (!libraryCardId) { showMessage("대출증 번호를 입력해주세요.", "error"); return; }

                // Check for global overdue status for this card before proceeding
                if (!checkOverdueBooksAndSuspendCards(libraryCardId)) {
                    // Message is shown by checkOverdueBooksAndSuspendCards
                    // Potentially refresh admin list if it was visible
                    if (!document.getElementById('adminMode').classList.contains('hidden')) {
                        renderAdminCardList();
                    }
                    return;
                }


                const card = libraryCards.find(c => c.id === libraryCardId);
                if (!card) { showMessage("등록되지 않은 대출증 번호입니다.", "error"); return; }
                if (card.status === 'suspended') {
                    showMessage(`대출증 ${libraryCardId}은(는) 현재 ${card.suspensionReason === 'overdue' ? '연체로 인해' : ''} 정지 상태입니다. 관리자에게 문의하세요.`, "error");
                    return;
                }
                if (card.borrowedCount >= card.borrowLimit) {
                    showMessage(`대출 한도(${card.borrowLimit}권)를 초과했습니다. 현재 ${card.borrowedCount}권 대출 중입니다.`, "error");
                    return;
                }

                const book = books.find(b => b.id === currentBorrowBookInternalId); // currentBorrowBookInternalId was set in the previous step
                if (book && book.status === 'available') {
                    book.status = 'borrowed';
                    book.borrowedByCard = libraryCardId;
                    book.dueDate = formatDate(addDays(new Date(), DEFAULT_LOAN_PERIOD_DAYS));
                    card.borrowedCount++;
                    saveBooks();
                    saveLibraryCards();
                    showMessage(`책 '${book.name}'(ID: ${book.id})이(가) 대출증 ${libraryCardId}(으)로 대출되었습니다. 반납 예정일: ${book.dueDate}`, "success");
                    bookIdInput.value = '';
                    libraryCardNumberInput.value = '';
                    libraryCardInputSection.classList.add('hidden');
                    currentBorrowBookInternalId = null;
                    setTimeout(goHome, 3000);
                } else {
                    showMessage("오류가 발생했습니다. (책 상태 변경 실패 또는 책 없음)", "error");
                    libraryCardInputSection.classList.add('hidden');
                    currentBorrowBookInternalId = null;
                }
            } else { // 책 번호 입력 단계
                const bookIdToBorrow = bookIdInput.value.trim();
                if (!bookIdToBorrow) { showMessage("책 번호를 입력해주세요.", "error"); return; }

                const bookToBorrow = books.find(b => b.id === bookIdToBorrow);
                if (!bookToBorrow) { showMessage("존재하지 않는 책 번호입니다.", "error"); return; }
                if (bookToBorrow.status === 'borrowed') { showMessage("이미 대출 중인 책입니다.", "error"); return; }

                currentBorrowBookInternalId = bookToBorrow.id; // Store the valid book ID
                libraryCardInputSection.classList.remove('hidden');
                showMessage("대출증 번호를 입력하고 '대출' 버튼을 다시 눌러주세요.", "info");
            }
        }

        // --- 반납 모드 ---
        function returnBook() {
            const bookIdInput = document.getElementById('returnBookId');
            const bookIdToReturn = bookIdInput.value.trim();
            if (!bookIdToReturn) { showMessage("반납할 책 번호를 입력해주세요.", "error"); return; }

            const bookIndex = books.findIndex(b => b.id === bookIdToReturn);
            if (bookIndex === -1) { showMessage("존재하지 않는 책 번호입니다.", "error"); return; }

            const book = books[bookIndex];
            if (book.status === 'available') { showMessage("이미 반납된 (또는 대출되지 않은) 책입니다.", "error"); return; }

            const returningCardId = book.borrowedByCard;
            const wasOverdue = book.dueDate && formatDate(new Date()) > book.dueDate;

            book.status = 'available';
            book.borrowedByCard = null;
            book.dueDate = null;

            if (returningCardId) {
                const card = libraryCards.find(c => c.id === returningCardId);
                if (card) {
                    if (card.borrowedCount > 0) card.borrowedCount--;
                    // If card was suspended ONLY due to this book (or all its overdue books are now returned), and admin didn't manually suspend, consider reactivating.
                    // For simplicity, admin currently needs to manually unsuspend.
                    // However, we can remove 'overdue' as suspensionReason if no other books by this card are overdue.
                    const otherOverdueBooks = books.some(b => b.borrowedByCard === card.id && b.status === 'borrowed' && b.dueDate && formatDate(new Date()) > b.dueDate);
                    if (card.suspensionReason === 'overdue' && !otherOverdueBooks) {
                        // If admin wishes for auto-unsuspend on returning all overdue items:
                        // card.status = 'active';
                        // card.suspensionReason = null;
                        // showMessage(`대출증 ${card.id}의 모든 연체 도서가 반납되어 정지가 해제될 수 있습니다. (현재는 관리자 수동 해제)`, "info");
                    }
                }
            }
            saveBooks();
            saveLibraryCards();
            showMessage(`책 '${book.name}'(ID: ${book.id})이(가) 반납되었습니다. ${wasOverdue ? '(연체 반납)' : ''}`, "success");
            bookIdInput.value = '';
            setTimeout(goHome, 2000);
        }

        // --- 관리자 모드 ---
        function accessAdminMode() {
            const password = document.getElementById('adminPassword').value;
            if (password === ADMIN_PIN) { showMode('adminMode'); }
            else { showMessage("관리자 번호가 올바르지 않습니다.", "error"); }
            document.getElementById('adminPassword').value = '';
        }
        function exitAdminMode() { goHome(); }

        function addBook() {
            const bookId = document.getElementById('bookIdInput').value.trim();
            const name = document.getElementById('bookName').value.trim();
            const location = document.getElementById('bookLocation').value.trim();

            if (!bookId || !name || !location) { showMessage("책 번호, 이름, 위치를 모두 입력해주세요.", "error"); return; }
            if (books.some(b => b.id === bookId)) { showMessage(`책 번호 '${bookId}'는 이미 존재합니다. 다른 번호를 사용해주세요.`, "error"); return; }

            books.push({ id: bookId, name, location, status: 'available', borrowedByCard: null, dueDate: null });
            saveBooks();
            showMessage(`'${name}'(ID: ${bookId}) 책이 등록되었습니다.`, "success");
            document.getElementById('bookIdInput').value = '';
            document.getElementById('bookName').value = '';
            document.getElementById('bookLocation').value = '';
            renderAdminBookList();
        }

        function editBookId(currentBookId) {
            const book = books.find(b => b.id === currentBookId);
            if (!book) { showMessage("책을 찾을 수 없습니다.", "error"); return; }
            if (book.status === 'borrowed') { showMessage("대출 중인 책의 번호는 변경할 수 없습니다. 먼저 반납해주세요.", "error"); return; }

            const newBookId = prompt(`책 '${book.name}'의 새 번호를 입력하세요 (현재: ${currentBookId}):`, currentBookId);
            if (!newBookId || newBookId.trim() === "") { showMessage("새 책 번호가 입력되지 않았습니다.", "info"); return; }
            if (newBookId.trim() === currentBookId) { showMessage("기존 번호와 동일합니다.", "info"); return; }
            if (books.some(b => b.id === newBookId.trim())) { showMessage(`책 번호 '${newBookId.trim()}'는 이미 존재합니다. 다른 번호를 사용해주세요.`, "error"); return; }

            book.id = newBookId.trim();
            saveBooks();
            showMessage(`책 ID가 '${currentBookId}'에서 '${book.id}'(으)로 변경되었습니다.`, "success");
            renderAdminBookList();
        }


        function deleteBook(bookId) {
            const bookIndex = books.findIndex(b => b.id === bookId);
            if (bookIndex === -1) { showMessage("삭제할 책을 찾을 수 없습니다.", "error"); return; }
            if (books[bookIndex].status === 'borrowed') { showMessage(`책 '${books[bookIndex].name}'(은)는 현재 대출 중이므로 삭제할 수 없습니다. 먼저 반납해주세요.`, "error"); return; }
            if (confirm(`정말로 '${books[bookIndex].name}' (ID: ${bookId}) 책을 삭제하시겠습니까?`)) {
                books.splice(bookIndex, 1);
                saveBooks();
                showMessage(`책 (ID: ${bookId})이(가) 삭제되었습니다.`, "success");
                renderAdminBookList();
            }
        }

        function renderAdminBookList() {
            adminBookListUl.innerHTML = '';
            if (books.length === 0) { adminBookListUl.innerHTML = '<li>등록된 책이 없습니다.</li>'; return; }
            books.forEach(book => {
                const listItem = document.createElement('li');
                let statusText = book.status === 'available' ? '<span class="available">대출 가능</span>' : `<span class="borrowed">대출 중 (카드: ${book.borrowedByCard || 'N/A'})</span>`;
                if (book.status === 'borrowed' && book.dueDate) {
                    const isOverdue = formatDate(new Date()) > book.dueDate;
                    statusText += `, 반납일: <span class="${isOverdue ? 'overdue' : ''}">${book.dueDate} ${isOverdue ? '(연체)' : ''}</span>`;
                }
                listItem.innerHTML = `
                    <span>ID: ${book.id}, 이름: ${book.name}, 위치: ${book.location}, 상태: ${statusText}</span>
                    <div class="list-item-actions">
                        <button class="edit-button admin-action-button" onclick="editBookId('${book.id}')" ${book.status === 'borrowed' ? 'disabled title="대출 중인 책은 ID 변경 불가"' : ''}>ID 변경</button>
                        <button class="delete-button admin-action-button" onclick="deleteBook('${book.id}')">삭제</button>
                    </div>
                `;
                adminBookListUl.appendChild(listItem);
            });
        }

        function addLibraryCard() {
            const cardIdInput = document.getElementById('libraryCardId');
            const borrowLimitInput = document.getElementById('libraryCardBorrowLimit');
            const cardId = cardIdInput.value.trim();
            const borrowLimit = parseInt(borrowLimitInput.value, 10) || DEFAULT_BORROW_LIMIT;

            if (!cardId) { showMessage("대출증 번호를 입력해주세요.", "error"); return; }
            if (libraryCards.some(c => c.id === cardId)) { showMessage("이미 등록된 대출증 번호입니다.", "error"); return; }
            if (borrowLimit < 1) { showMessage("최대 대출 가능 권수는 1 이상이어야 합니다.", "error"); return; }

            libraryCards.push({ id: cardId, status: 'active', borrowLimit: borrowLimit, borrowedCount: 0, suspensionReason: null });
            saveLibraryCards();
            showMessage(`대출증 '${cardId}'이(가) 등록되었습니다 (최대 ${borrowLimit}권).`, "success");
            cardIdInput.value = '';
            borrowLimitInput.value = DEFAULT_BORROW_LIMIT;
            renderAdminCardList();
        }

        function editCardBorrowLimit(cardId) {
            const card = libraryCards.find(c => c.id === cardId);
            if (!card) { showMessage("대출증을 찾을 수 없습니다.", "error"); return; }
            const newLimitStr = prompt(`대출증 ${cardId}의 새 대출 한도를 입력하세요 (현재: ${card.borrowLimit}권, 현재 대출: ${card.borrowedCount}권):`, card.borrowLimit);
            if (newLimitStr === null) { showMessage("한도 수정이 취소되었습니다.", "info"); return; }
            const newLimit = parseInt(newLimitStr, 10);
            if (isNaN(newLimit) || newLimit < 0) { // Allow 0 if needed, but usually >= borrowedCount
                showMessage("유효한 숫자를 입력해주세요.", "error"); return;
            }
            if (newLimit < card.borrowedCount) {
                showMessage(`새 한도(${newLimit}권)는 현재 대출 중인 책의 수(${card.borrowedCount}권)보다 적을 수 없습니다.`, "error"); return;
            }
            card.borrowLimit = newLimit;
            saveLibraryCards();
            showMessage(`대출증 ${cardId}의 대출 한도가 ${newLimit}권으로 변경되었습니다.`, "success");
            renderAdminCardList();
        }


        function deleteLibraryCard(cardId) {
            const cardIndex = libraryCards.findIndex(c => c.id === cardId);
            if (cardIndex === -1) { showMessage("삭제할 대출증을 찾을 수 없습니다.", "error"); return; }
            const card = libraryCards[cardIndex];
            if (card.borrowedCount > 0) { showMessage(`대출증 ${cardId}은(는) 현재 ${card.borrowedCount}권의 도서를 대출 중이므로 삭제할 수 없습니다.`, "error"); return; }
            if (confirm(`정말로 대출증 '${cardId}'을(를) 삭제하시겠습니까?`)) {
                libraryCards.splice(cardIndex, 1);
                saveLibraryCards();
                showMessage(`대출증 '${cardId}'이(가) 삭제되었습니다.`, "success");
                renderAdminCardList();
            }
        }

        function toggleLibraryCardStatus(cardId, newStatusManual = null) { // newStatusManual can be 'active' or 'suspended'
            const card = libraryCards.find(c => c.id === cardId);
            if (!card) { showMessage("대출증을 찾을 수 없습니다.", "error"); return; }

            let targetStatus = newStatusManual;
            let reason = newStatusManual === 'suspended' ? 'manual' : null;

            if (!targetStatus) { // If just toggling (old buttons without explicit status)
                 targetStatus = card.status === 'active' ? 'suspended' : 'active';
                 if (targetStatus === 'suspended') reason = 'manual';
            }


            if (targetStatus === 'suspended' && card.borrowedCount > 0 && card.suspensionReason !== 'overdue') {
                 // Allow suspending if reason is 'overdue' even with borrowed books.
                 // Prevent manual suspension if books are out unless already suspended for overdue.
                showMessage(`대출증 ${cardId}은(는) 현재 ${card.borrowedCount}권의 도서를 대출 중이므로 수동으로 정지할 수 없습니다. (연체 정지는 자동)`, "error");
                return;
            }

            card.status = targetStatus;
            card.suspensionReason = reason; // Set to 'manual' or null if activated. 'overdue' is set by system.

            // If activating a card that was suspended for overdue, clear the overdue reason
            // only if no books are currently overdue for this card.
            if (targetStatus === 'active' && card.suspensionReason === 'overdue') {
                const stillOverdue = books.some(b => b.borrowedByCard === card.id && b.status === 'borrowed' && b.dueDate && formatDate(new Date()) > b.dueDate);
                if (stillOverdue) {
                    showMessage(`대출증 ${cardId}은(는) 여전히 연체된 도서가 있습니다. 먼저 모든 연체 도서를 반납해야 자동으로 '연체 정지' 사유가 해제됩니다.`, "error");
                    card.status = 'suspended'; // Revert
                    card.suspensionReason = 'overdue'; // Keep reason
                } else {
                     card.suspensionReason = null; // Clear reason as admin is activating and no more overdue
                }
            } else if (targetStatus === 'active') {
                card.suspensionReason = null; // Clear any reason if activated manually
            }


            saveLibraryCards();
            showMessage(`대출증 '${cardId}'의 상태가 ${card.status === 'active' ? '활성' : '정지'}(으)로 변경되었습니다. ${card.suspensionReason ? `(사유: ${card.suspensionReason})` : ''}`, "success");
            renderAdminCardList();
        }


        function renderAdminCardList() {
            adminCardListUl.innerHTML = '';
            if (libraryCards.length === 0) { adminCardListUl.innerHTML = '<li>등록된 대출증이 없습니다.</li>'; return; }
            libraryCards.forEach(card => {
                const listItem = document.createElement('li');
                const statusClass = card.status === 'active' ? 'active' : 'suspended';
                let statusText = card.status === 'active' ? '활성' : '정지';
                if (card.status === 'suspended' && card.suspensionReason) {
                    statusText += ` (사유: ${card.suspensionReason === 'overdue' ? '연체' : '수동'})`;
                }

                let actionButtons = `
                    <button class="edit-button admin-action-button" onclick="editCardBorrowLimit('${card.id}')">한도수정</button>
                    <button class="delete-button admin-action-button" onclick="deleteLibraryCard('${card.id}')">삭제</button>
                `;
                if (card.status === 'active') {
                    actionButtons += `<button class="suspend-button admin-action-button" onclick="toggleLibraryCardStatus('${card.id}', 'suspended')">정지</button>`;
                } else { // Suspended
                    actionButtons += `<button class="activate-button admin-action-button" onclick="toggleLibraryCardStatus('${card.id}', 'active')">정지해제</button>`;
                }

                listItem.innerHTML = `
                    <span>
                        ID: ${card.id}, 상태: <span class="${statusClass}">${statusText}</span>,
                        한도: ${card.borrowLimit}권, 현재: ${card.borrowedCount}권
                    </span>
                    <div class="list-item-actions">
                        ${actionButtons}
                    </div>
                `;
                adminCardListUl.appendChild(listItem);
            });
        }
    </script>
</body>
</html>
