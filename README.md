    <?php
    // ✅ Автоматическое создание history.json при отсутствии
    if (!file_exists("history.json")) {
        file_put_contents("history.json", "[]");
    }

    // Обработка POST-запроса
    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
        $newData = [
            'boards' => json_decode($_POST['boards'], true),
            'stateCount' => json_decode($_POST['stateCount'], true),
            'totalCount' => json_decode($_POST['totalCount'], true),
            'orderedBoardIds' => json_decode($_POST['orderedBoardIds'], true),
            'lastUpdate' => time()
        ];

        // Архивируем, но после этого обязательно сохраняем текущие данные в data.json
        date_default_timezone_set("Europe/Warsaw");
        $currentHour = date("H");
        $currentMinute = date("i");
        $currentTimeStr = "$currentHour:$currentMinute";
        $archiveTimes = ['14:20', '22:20'];

        if (in_array($currentTimeStr, $archiveTimes) || isset($_POST['forceArchive'])) {
            $history = json_decode(file_get_contents("history.json"), true);
            $history[] = $newData;
            file_put_contents("history.json", json_encode($history, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
        }

        // ✅ Всегда сохраняем актуальные данные
        file_put_contents("data.json", json_encode($newData, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));

        exit;
    }





    $data = file_exists("data.json") ? json_decode(file_get_contents("data.json"), true) : [
        'boards' => [],
        'stateCount' => ['LS' => 0, 'LB' => 0, 'RB' => 0, 'RS' => 0],
        'totalCount' => ['LS' => 0, 'LB' => 0, 'RB' => 0, 'RS' => 0],
        'orderedBoardIds' => []
    ];

    $langs = [
        "pl" => [
            "title" => "Monitor Tablic",
            "input_label" => "Wprowadź numer tablicy montażowej, stan, wartość i tekst (np. 1LS 84 Test):",
            "reset" => "Resetuj",
            "reset_scheme" => "Resetuj schemat",
            "current_stats" => "🔹 Bieżące statystyki (tablice):",
            "state" => "Stan",
            "current" => "Bieżące",
            "total" => "Łącznie",
            "total_tables" => "🔹ZROBIONO W TEJ ZMIANIE:",
            "table_order" => "🔹 Kolejność tablic:",
            "dark_mode" => "🌙 Tryb ciemny",
            "history_button" => "📂 Historia zapisów",
            "clear_history" => "🗑 Wyczyść archiwum",
            // ✅ Новые ключи для нормы и скорости
            "norm_label" => "Norma (szt):",
            "speed_label" => "Prędkość"
        ],
        "en" => [
            "title" => "Board Monitor",
            "input_label" => "Enter board number, state, value and label (np. 1LS 84 Test):",
            "reset" => "Reset",
            "reset_scheme" => "Reset Scheme",
            "current_stats" => "🔹 Current statistics (boards):",
            "state" => "State",
            "current" => "Current",
            "total" => "Total",
            "total_tables" => "🔹 Total number of registered boards:",
            "table_order" => "🔹 Board order:",
            "dark_mode" => "🌙 Dark Mode",
            "history_button" => "📂 View history",
            "clear_history" => "🗑 Clear archive",
            // ✅ Новые ключи для нормы и скорости
            "norm_label" => "Norm (pcs):",
            "speed_label" => "Speed"
        ]
    ];


    $lang_code = $_GET['lang'] ?? 'pl';
    if (!isset($langs[$lang_code])) $lang_code = 'pl';
    $LANG = $langs[$lang_code];
    ?>
    <!DOCTYPE html>
    <html lang="<?= htmlspecialchars($lang_code) ?>">
    <head>
    <meta charset="UTF-8" />
    <title><?= htmlspecialchars($LANG['title']) ?></title>
    <style>
    #shift-indicator {
    position: fixed;
    top: 15px;
    left: 15px;
    padding: 10px 20px;
    font-size: 35px;
    font-weight: 700;
    border-radius: 14px;
    letter-spacing: 1px;
    backdrop-filter: blur(10px);
    background: linear-gradient(145deg, #e6ebf2, #d3dbe6);
    box-shadow: 0 8px 20px rgba(0,0,0,0.12);
    transition: all 0.3s ease;
}

/* 1 смена */
.shift-1 {
    color: #1e9e55;
}

/* 2 смена */
.shift-2 {
    color: #c63a6a;
}

/* Dark theme */
body.dark #shift-indicator {
    background: linear-gradient(145deg, #1f2430, #2b3242);
    box-shadow: 0 8px 20px rgba(0,0,0,0.6);
}

body.dark .shift-1 {
    color: #00c853;
}

body.dark .shift-2 {
    color: #ff4d8d;
}




        * { box-sizing: border-box; }
body {
    font-family: "Segoe UI", "Inter", monospace;
    margin: 0;
    padding: 20px;
    background: linear-gradient(135deg, #f4f6f9, #e6ebf2);
    color: #1c1f26;
    transition: background 0.4s ease, color 0.4s ease;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
}

        body.dark {
    background: linear-gradient(135deg, #0f1115, #1b1f27);
    color: #e4e7ec;
}
        header, main {
            width: 100%;
            max-width: 900px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        header h1 {
    font-size: 32px;
	 margin-top: -20px;
    font-weight: 800;
    letter-spacing: 2px;
}
body.dark header h1 {
    color: #eaeaea;
    text-shadow: 0 0 8px rgba(0,255,157,0.6);
}


        #clock {
    margin-top: -30px;
    font-size: 70px;
    font-weight: 900;
    letter-spacing: 2px;
    text-shadow: 0 4px 10px rgba(0,0,0,0.15);
}

        .top-right {
            position: absolute;
            right: 0;
            top: 0;
            display: flex;
            gap: 10px;
        }
        button {
    font-size: 16px;
    padding: 8px 16px;
    border-radius: 10px;
    border: none;
    background: linear-gradient(145deg, #dfe4ea, #cfd6df);
    box-shadow: 4px 4px 10px rgba(0,0,0,0.08),
                -4px -4px 10px rgba(255,255,255,0.6);
    cursor: pointer;
    transition: all 0.2s ease;
}

button:hover {
    transform: translateY(-2px);
    box-shadow: 6px 6px 15px rgba(0,0,0,0.12);
}

        body.dark button {
    background: linear-gradient(145deg, #1f2430, #2b3242);
    box-shadow: 4px 4px 10px rgba(0,0,0,0.6),
                -4px -4px 10px rgba(255,255,255,0.05);
    color: white;
}
        label {
            font-size: 1.1em;
            margin-bottom: 8px;
            text-align: center;
        }
        .input-row {
            display: flex;
            gap: 10px;
            margin-bottom: 15px;
            flex-wrap: wrap;
            justify-content: center;
        }
        input#input {
            font-family: monospace;
            font-size: 16px;
            padding: 6px 12px;
            width: 200px;
            max-width: 100%;
            border: 1px solid #ccc;
            border-radius: 10px;
        }
        input#input:focus {
            outline: none;
            border-color: #6666ff;
        }
        .buttons-group {
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
        }
      pre#stats {
    margin-top: 5px;
    background: rgba(255,255,255,0.7);
    backdrop-filter: blur(12px);
    padding: 5px;
    width: 100%;
    max-width: 180%;
    height: 500px;
    overflow-y: auto;
    white-space: pre-wrap;
    font-size: 45px;
	font-weight: 900;
    border-radius: 19px;
    border: 1px solid rgba(0,0,0,0.1);
    box-shadow: 0 15px 40px rgba(0,0,0,0.08);
}

body.dark pre#stats {
    background: rgba(30,35,45,0.75);
    border: 1px solid rgba(255,255,255,0.08);
    box-shadow: 0 15px 40px rgba(0,0,0,0.6);
}
        #board-visual {
            margin-top: 1px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            align-items: center;
        }
        .board-row {
        display: flex;
        gap: 10px;
        flex-wrap: nowrap;
        justify-content: center;
        align-items: flex-start; /* 🟢 Ключевой момент! */
        width: 100%;
    }

        .board-row.top-row {
            justify-content: flex-end;
        }
        .board-row.bottom-row {
            justify-content: flex-start;
        }


    /* Светлая тема — яркие и читаемые цвета */
    .board-tile.LS { background: linear-gradient(145deg, #8ff5c3, #4cd39a); }
.board-tile.LB { background: linear-gradient(145deg, #ffc2d3, #ff7aa2); }
.board-tile.RS { background: linear-gradient(145deg, #42c97f, #1e9e55); }
.board-tile.RB { background: linear-gradient(145deg, #e05380, #b92d5c); }



    /* Тёмная тема — насыщенные цвета */
   body.dark .board-tile.LS { background: linear-gradient(145deg, #00c853, #009624); }
body.dark .board-tile.LB { background: linear-gradient(145deg, #c2185b, #880e4f); }
body.dark .board-tile.RS { background: linear-gradient(145deg, #2e8b57, #1b5e20); }
body.dark .board-tile.RB { background: linear-gradient(145deg, #a31545, #6a0030); }


    /* Оверлей перерыва */
    #break-overlay {
        position: fixed;
        top: 0; left: 0;
        width: 100vw; height: 100vh;
        background: rgba(0, 0, 0, 0.85);
        color: red;
        display: none;
        align-items: center;
        justify-content: center;
        font-size: 100px;
        font-weight: bold;
        z-index: 9999;
        animation: pulse 1s infinite;
    }

    @keyframes pulse {
        0% { transform: scale(1); opacity: 0.9; }
        50% { transform: scale(1.05); opacity: 0.8; }
        100% { transform: scale(1); opacity: 0.9; }
    }

    #break-overlay {
        animation: pulse 2s infinite; /* медленнее и мягче */
    }

    /* цифры как раньше */
    .number-cell {
        width: 120px;          /* ← точное совпадение с плиткой */
        padding: 2px 0;
        font-size: 25px;
        font-weight: bold;
        text-align: center;
        margin: 0;             /* убрать случайные смещения */
        margin-left: 5px;
    }

    /* Taśmowanie: теперь занимает 9 колонок, было 10 */
    .tasmowanie-wide {
        grid-column: span 10;
        font-size: 30px;
        font-weight: 800;
        text-align: center;
        margin-top: -35px;   /* убираем лишнее пространство сверху */
        margin-bottom: 0px; /* уменьшаем промежуток снизу */
    }


    /* Контейнер для цифр — делаем относительным, чтобы absolute работало */
    .board-row-top {
        display: grid;
        grid-template-columns: repeat(12, 1fr);
        width: 100%;
        text-align: center;
        position: relative; /* ← важно для абсолютного позиционирования */
    }

    /* Цифры — свободное перемещение */
    .cell12,
    .cell13,
    .cell14 {
        position: absolute;     /* ← свободное перемещение */
        font-weight: bold;
        font-size: 25px;
        text-align: center;
        cursor: move;           /* курсор показывает возможность движения */
    }

    /* Пример начального положения — можно менять */
    .cell12 { top: -20px; left: 1450px; }
    .cell13 { top: -20px; left: 1300px; }
    .cell14 { top: -20px; left: 1150px; }

    /* Остальные стили */
    .krok-label {
        display: flex;
        align-items: center;
        font-size: 28px;
        font-weight: 900;
        margin-right: 15px;
        padding: 0 10px;
    }

    .krok-top {
        font-size: 30px;
        font-weight: bold;
        writing-mode: vertical-rl;
        transform: rotate(-90deg);
        margin-top: -20px;
    }

    .krok-bottom {
        font-size: 30px;
        font-weight: bold;
        writing-mode: vertical-rl;
        transform: rotate(-90deg);
        margin-top: 5px;
    }

    .number-cell {
        width: 120px;
        margin-left: 3px;
    }

    .bottom-numbers {
        margin-left: 35px;
    }

    .del-btn {
        background: #c0392b;
        color: white;
        border: none;
        border-radius: 4px;
        padding: 2px 5px;
        cursor: pointer;
        font-size: 18px;
    }
    .del-btn:hover {
        background: #e74c3c;
    }

    /* ===== Table colors ===== */
    .ls { background: #7CFF9B; }   /* light green */
    .lb { background: #FF9EB5; }   /* light pink */
    .rs { background: #1E9E55; }   /* dark green */
    .rb { background: #C63A6A; }   /* dark pink */

    /* bottom stripe */
    .bottom-line {
        height: 6px;
        width: 100%;
    }

    /* stripe colors = pair side */
    .line-ls { background: #7CFF9B; }
    .line-lb { background: #FF9EB5; }
    .line-rs { background: #1E9E55; }
    .line-rb { background: #C63A6A; }

    /* cell layout */
    .table-cell {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        border-radius: 6px;
        overflow: hidden;
    }



    .board-tile {
    width: 120px;
    height: 70px;
    position: relative;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    border-radius: 18px;
    overflow: hidden;
    font-weight: 600;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
    box-shadow: 0 6px 20px rgba(0,0,0,0.12);
}

.board-tile:hover {
    transform: translateY(-3px);
}



    .cell-content {
        flex: 1;
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 25px;
    }



    .bottom-line {
        width: 100%;
        height: 6px;
        flex-shrink: 0;              /* 🔑 не сжимается */
    }


    /* цвета полосок */
    .line-ls { background-color: #9be7b3; }  /* светло-зелёный */
    .line-rs { background-color: #2e8b57; }  /* тёмно-зелёный */
    .line-lb { background-color: #f6b3c2; }  /* светло-розовый */
    .line-rb { background-color: #c84c6a; }  /* тёмно-розовый */

    .board-highlight {
    outline: 4px solid #ffd600;
    box-shadow: 0 0 20px rgba(255,214,0,0.7);
}


    .control-panel {
        margin-top: -10px;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 30px;
        flex-wrap: wrap;
    }

    .norm-block,
    .speed-block,
    .search-block {
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 18px;
    }

    #norm {
        width: 80px;
        padding: 4px 6px;
        border-radius: 6px;
        border: 1px solid #aaa;
    }

    #search-value {
        width: 120px;
        padding: 4px 8px;
        border-radius: 20px;
        border: 2px solid #6666ff;
        text-align: center;
        font-weight: bold;
    }

    #search-value:focus {
        outline: none;
        border-color: #00bcd4;
        box-shadow: 0 0 8px #00bcd4;
    }

/* ===== Language Select Premium ===== */

.language-select {
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;

    padding: 6px 36px 6px 14px;
    font-size: 16px;
    font-weight: 700;

    border-radius: 12px;
    border: 1px solid rgba(0,0,0,0.15);

    background: linear-gradient(145deg, #f0f3f8, #d9e0ea);
    color: #1c1f26;

    box-shadow: 4px 4px 12px rgba(0,0,0,0.08),
                -4px -4px 12px rgba(255,255,255,0.6);

    cursor: pointer;
    transition: all 0.2s ease;

    background-image: url("data:image/svg+xml;utf8,<svg fill='%231c1f26' height='20' viewBox='0 0 24 24' width='20' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l5 5 5-5z'/></svg>");
    background-repeat: no-repeat;
    background-position: right 10px center;
    background-size: 16px;
}


body.dark .language-select {
    background: linear-gradient(145deg, #1f2430, #2b3242);
    color: #ffffff;
    border: 1px solid rgba(255,255,255,0.1);

    box-shadow: 4px 4px 12px rgba(0,0,0,0.6),
                -4px -4px 12px rgba(255,255,255,0.05);

    background-image: url("data:image/svg+xml;utf8,<svg fill='%23ffffff' height='20' viewBox='0 0 24 24' width='20' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l5 5 5-5z'/></svg>");
}



/* Dark theme */
body.dark .language-select {
    background: linear-gradient(145deg, #1f2430, #2b3242);
    color: white;
    box-shadow: 4px 4px 10px rgba(0,0,0,0.6),
                -4px -4px 10px rgba(255,255,255,0.05);
}

body.dark .language-select {
    background: #2b3242;
    color: #ffffff;
    border: 1px solid #444;
}

body.dark .language-select option {
    background-color: #2b3242;
    color: #ffffff;
}



    </style>
    </head>
    <body>
    <header>

        <!-- твой header код -->
    </header>
    <div id="break-overlay">
        Przerwa
        <button id="hide-break-overlay" style="
            position:absolute; top:20px; right:20px;
            font-size:25px; padding:5px 10px;">❌</button>
    </div>

    </body>
    <header>
        <h1><?= htmlspecialchars($LANG['title']) ?></h1>
        <div id="clock"></div>
        <div class="top-right">
		<div class="top-right">
    
</div>

        <button onclick="showAdminLogin()">🔑</button>
    <div id="shift-indicator" ></div>
    <div style="display: flex; align-items: center; gap: 5px; margin-bottom: 5px;">
        <select id="lang" class="language-select">
    <option value="pl" <?= $lang_code == 'pl' ? 'selected' : '' ?>>PL</option>
    <option value="en" <?= $lang_code == 'en' ? 'selected' : '' ?>>EN</option>
</select>

    <button onclick="showHistory()"><?= htmlspecialchars($LANG['history_button']) ?></button>
    <button onclick="clearHistory()"><?= htmlspecialchars($LANG['clear_history']) ?></button>


    </div>
        
        </select>
        <button onclick="toggleTheme()"><?= htmlspecialchars($LANG['dark_mode']) ?></button>
    </div>
    </header>

    <main>
        <label for="input"><?= htmlspecialchars($LANG['input_label']) ?></label>
        <div class="input-row">
            <input type="text" id="input" placeholder=" Wpisz np. 1LS 84 " autocomplete="off" />
            <div class="buttons-group">
                <button onclick="resetStats()"><?= htmlspecialchars($LANG['reset']) ?></button>
                <button onclick="resetScheme()"><?= htmlspecialchars($LANG['reset_scheme']) ?></button>
                <button onclick="downloadStats()">💾</button>
            </div>
        </div>
    <div style="margin-top:10px;">
    <div class="control-panel">
        
        <div class="norm-block">
            <label for="norm"><?= htmlspecialchars($LANG['norm_label']) ?></label>
            <input type="number" id="norm" min="1" value="100" />
        </div>

        <div class="speed-block">
            <span id="speed-info"><?= htmlspecialchars($LANG['speed_label']) ?>: 0 pcs/h</span>
        </div>

        <div class="search-block">
            <label for="search-value">🔎</label>
            <input type="text" id="search-value" placeholder="Znaleźć deskę..." />
        </div>

    </div>


        <pre id="stats" aria-live="polite"></pre>
        <div id="board-visual"></div>
    </main>



    <script>

    let breakOverlayEnabled = true;   // ← ВОТ СЮДА
    let manualHideBreak = false;   

    const tables = {
        "1RS": "R31B-82",
        "2LS": "59-81",
        "3RB": "R22S-80",
        "4LS": "58-79",
        "5RS": "R28B-78",
        "6LS": "57-77",
        "7RB": "R23S-76",
        "8LS": "56-75",
        "9RS": "R33B-74",
        "10LS": "55-73",
        "11RB": "R26S-93",
        "12LS": "53-54",

        "1LB": "82-R31B",
        "2LB": "81-59",
        "3LB": "80-R22S",
        "4LB": "79-58",
        "5LB": "78-R28B",
        "6LB": "77-57",
        "7LB": "76-R23S",
        "8LB": "75-56",
        "9LB": "74-R33B",
        "10LB": "73-55",
        "11LS": "93-R26S",
        "12LB": "54-53",
        
        "24LB": "83-60",
        "23LB": "84-90",
        "22LB": "85-61",
        "21LB": "64-R29B",
        "20LB": "65-50",
        "19LS": "92-R21S",
        "18RB": "R25S-51",
        "17LB": "67-R32B",
        "16LB": "68-91",
        "15LB": "69-R35B",
        "14LB": "70-52",
        "13LS": "94-R27S",

        "24LS": "60-83",
        "23LS": "90-84",
        "22LS": "61-85",
        "21RS": "R29B-64",
        "20LS": "50-65",
        "19RB": "R21S-92",
        "18LS": "51-R25S",
        "17RS": "R32B-67",
        "16LS": "91-68",
        "15RS": "R35B-69",
        "14LS": "52-70",
        "13RB": "R27S-94"
    };


    function getPairType(key) {
        const num = key.match(/\d+/)[0];   // 3, 24, 8 ...
        const side = key.slice(-2);        // LB, RB, LS, RS

        const allSides = ["LB", "LS", "RB", "RS"];

        for (const s of allSides) {
            const pairKey = num + s;
            if (pairKey !== key && tables[pairKey]) {
                return s.toLowerCase();
            }
        }
        return null;
    }



    let lastUpdateTime = 0;
    let lastInputKey = null;

    let boards = <?= json_encode($data['boards'], JSON_UNESCAPED_UNICODE) ?>;
    let stateCount = <?= json_encode($data['stateCount'], JSON_UNESCAPED_UNICODE) ?>;
    let totalCount = <?= json_encode($data['totalCount'], JSON_UNESCAPED_UNICODE) ?>;
    let orderedBoardIds = <?= json_encode($data['orderedBoardIds'], JSON_UNESCAPED_UNICODE) ?>;

    let stagePercents = {
        shift1: [0.5, 0.25, 0.25],
        shift2: [0.5, 0.25, 0.25]
    };


    // метки для локализации
    let SPEED_LABEL = "<?= $LANG['speed_label'] ?>";
    let NORM_LABEL = "<?= $LANG['norm_label'] ?>";

    // обновляем подписи на странице
    function updateLabels() {
        document.querySelector('label[for="norm"]');
        updateSpeed();
    }

    // загрузка сохранённой нормы
    document.addEventListener("DOMContentLoaded", function() {
        const savedNorm = localStorage.getItem("normValue");
        if (savedNorm !== null) {
            document.getElementById("norm").value = savedNorm;
        }
        updateLabels();
    });

    // сохраняем норму при изменении
    document.getElementById("norm").addEventListener("input", function() {
        localStorage.setItem("normValue", this.value);
        updateSpeed();
    });

    // загрузка данных с сервера
    function fetchDataAndUpdate() {
        fetch("data.json?_=" + Date.now())
            .then(res => res.json())
            .then(data => {
                if (data.lastUpdate && data.lastUpdate > lastUpdateTime) {
                    lastUpdateTime = data.lastUpdate;

                    boards = data.boards || {};
                    stateCount = data.stateCount || {LS:0,LB:0,RB:0,RS:0};
                    totalCount = data.totalCount || {LS:0,LB:0,RB:0,RS:0};
                    orderedBoardIds = data.orderedBoardIds || [];

                    updateStats();
                    updateSpeed();
                }
            })
            .catch(err => console.error("Ошибка загрузки data.json:", err));
    }

    // вспомогательная функция для выравнивания текста
    function padRight(str, length) {
        str = String(str);
        return str.length >= length ? str : str + ' '.repeat(length - str.length);
    }

    // обновление статистики
    function updateStats() {
        let txt = `<?= $LANG['current_stats'] ?>\n`;
        txt += padRight("<?= $LANG['state'] ?>", 24) + "| " + padRight("<?= $LANG['current'] ?>", 8) + "| " + padRight("<?= $LANG['total'] ?>", 8) + '\n';
        txt += "-".repeat(45) + '\n';

        ["LS","LB","RB","RS"].forEach(s => {
            let label = s;
            if (s === "LS") label += " Lewa  SCHMALL";
            if (s === "LB") label += " Lewa  BREIT ";
            if (s === "RB") label += " Prawa BREIT ";
            if (s === "RS") label += " Prawa SCHMALL";
            txt += padRight(label, 24) + "| " + padRight(stateCount[s] || 0, 8) + "| " + padRight(totalCount[s] || 0, 8) + '\n';
        });

        // 🟢 Новый расчёт
        const totalBoards = Object.values(totalCount).reduce((a,b)=>a+b,0);
        const norm = parseInt(document.getElementById("norm").value, 10) || 0;
        const remaining = norm > 0 ? Math.max(0, norm - totalBoards) : 0;

        // 🟢 Новая строка с остатком
    txt += `\n<?= $LANG['total_tables'] ?> ${totalBoards}`;
    if (norm > 0) txt += `     (pozostało ${remaining})`;


        txt += `\n\n<?= $LANG['table_order'] ?>\n`;

        txt += padRight("ID+State", 10) + padRight("Value", 10) + "Label\n";
        txt += "-".repeat(40) + '\n';


        const sortedIds = Object.keys(boards).sort((a,b)=>parseInt(a)-parseInt(b));
        for (let id of sortedIds) {
        const b = boards[id];
        const row = `${padRight(id+b.state, 10)}${padRight(b.value, 10)}${(b.label||"")}`;
        txt += `${row}   [DEL:${id}]\n`; // добавляем текстовую метку для кнопки
    }


        document.getElementById("stats").textContent = txt;
    // делаем метки [DEL:ID] кликабельными
    const statsEl = document.getElementById("stats");
    statsEl.innerHTML = statsEl.textContent.replace(/\[DEL:(\d+)\]/g, '<button class="del-btn" data-id="$1">🗑</button>');

    // обработчик кликов
    statsEl.querySelectorAll(".del-btn").forEach(btn => {
        btn.onclick = () => {
            const id = btn.dataset.id;
            if (confirm(`❗ Czy na pewno chcesz usunąć tablicę #${id}?`)) {
                deleteBoard(id);
            }
        };
    });

        renderBoardVisual();
        updateSpeed();
    }

    // расчёт требуемой скорости для выполнения нормы
    function getShiftIntervals(now) {
        const y = now.getFullYear();
        const m = now.getMonth();
        const d = now.getDate();

        // первая смена (06:30–14:20) с перерывами
        const shift1 = [
            [new Date(y, m, d, 6, 30), new Date(y, m, d, 9, 40)],
            [new Date(y, m, d, 10, 0), new Date(y, m, d, 12, 10)],
            [new Date(y, m, d, 12, 20), new Date(y, m, d, 14, 20)]
        ];

        // вторая смена (14:30–22:20) с перерывами
        const shift2 = [
            [new Date(y, m, d, 14, 30), new Date(y, m, d, 18, 0)],
            [new Date(y, m, d, 18, 20), new Date(y, m, d, 20, 0)],
            [new Date(y, m, d, 20, 10), new Date(y, m, d, 22, 20)]
        ];

        if (now < shift1[shift1.length - 1][1]) return shift1;
        if (now < shift2[shift2.length - 1][1]) return shift2;

        // после 22:20 — следующая первая смена (на следующий день)
        const shiftNext = [
            [new Date(y, m, d + 1, 6, 30), new Date(y, m, d + 1, 9, 40)],
            [new Date(y, m, d + 1, 10, 0), new Date(y, m, d + 1, 12, 10)],
            [new Date(y, m, d + 1, 12, 20), new Date(y, m, d + 1, 14, 20)]
        ];
        return shiftNext;
    }

    function getStageRemainingHours(now) {
        const y = now.getFullYear();
        const m = now.getMonth();
        const d = now.getDate();
        const min = now.getHours() * 60 + now.getMinutes();

        // 1 zmiana
        if (min >= 390 && min < 580) { // 06:30–09:40
            return (new Date(y, m, d, 9, 40) - now) / 3600000;
        }
        if (min >= 600 && min < 730) { // 10:00–12:10
            return (new Date(y, m, d, 12, 10) - now) / 3600000;
        }
        if (min >= 740 && min < 860) { // 12:20–14:20
            return (new Date(y, m, d, 14, 20) - now) / 3600000;
        }

        // 2 zmiana
        if (min >= 870 && min < 1080) { // 14:30–18:00
            return (new Date(y, m, d, 18, 0) - now) / 3600000;
        }
        if (min >= 1100 && min < 1200) { // 18:20–20:00
            return (new Date(y, m, d, 20, 0) - now) / 3600000;
        }
        if (min >= 1210 && min < 1340) { // 20:10–22:20
            return (new Date(y, m, d, 22, 20) - now) / 3600000;
        }

        return 0;
    }

    function formatHours(hours) {
        const totalMinutes = Math.max(0, Math.floor(hours * 60));
        const h = Math.floor(totalMinutes / 60);
        const m = totalMinutes % 60;
        return `${h}:${m.toString().padStart(2, "0")}`;
    }



    function updateSpeed() {
        const norm = parseInt(document.getElementById("norm").value, 10) || 0;
        if (norm <= 0) return;

        if (isBreakTime()) {
            document.getElementById("speed-info").textContent =
                `${SPEED_LABEL}: przerwa`;
            return;
        }

        const produced = Object.values(totalCount).reduce((a, b) => a + b, 0);
        const now = new Date();

        const stage = getShiftStage(now);
        if (!stage) return;

        const targetQty = Math.round(norm * stage.target);
        const remainingQty = Math.max(0, targetQty - produced);
        const remainingHours = getStageRemainingHours(now);

        const speed = remainingHours > 0
            ? remainingQty / remainingHours
            : 0;

        document.getElementById("speed-info").textContent =
            `${SPEED_LABEL}: ${speed.toFixed(1)} |  `;
    }









    // Визуал нижних таблиц + подписи (KROK, 24–13, 1–12)
    function renderBoardVisual() {
        const container = document.getElementById("board-visual");
        container.innerHTML = '';

        // Главный блок схемы
        const wrapper = document.createElement("div");
        wrapper.style.display = "flex";
        wrapper.style.flexDirection = "row";
        wrapper.style.alignItems = "center";
        wrapper.style.gap = "15px";

        // Левая подпись
        const krokLabel = document.createElement("div");
        krokLabel.textContent = "";
        krokLabel.style.writingMode = "vertical-rl";
        krokLabel.style.transform = "rotate(-90deg)";
        krokLabel.style.fontSize = "22px";
        krokLabel.style.fontWeight = "bold";

        wrapper.appendChild(krokLabel);

        // Контейнер самого визуала
        const block = document.createElement("div");
        block.style.display = "flex";
        block.style.flexDirection = "column";
        block.style.gap = "10px";

        // ВЕРХНИЕ ЦИФРЫ (24 → 13)
    // Верхний ряд цифр и Taśmowanie
    const numbersTop = document.createElement("div");
    numbersTop.className = "board-row-top";

    // TAŚMOWANIE
    const tasmowanie = document.createElement("div");
    tasmowanie.className = "tasmowanie-wide";
    tasmowanie.textContent = "TAŚMOWANIE";
    numbersTop.appendChild(tasmowanie);

    // Справа: 14
    const cell14 = document.createElement("div");
    cell14.className = "cell14";
    cell14.textContent = "KROK 14";
    numbersTop.appendChild(cell14);

    // Справа: 13
    const cell13 = document.createElement("div");
    cell13.className = "cell13";
    cell13.textContent = "KROK 13";
    numbersTop.appendChild(cell13);

    // Справа: 12  ← теперь здесь!
    const cell12 = document.createElement("div");
    cell12.className = "cell12";
    cell12.textContent = "KROK 12";
    numbersTop.appendChild(cell12);







        // Верхний ряд плит
        const topRow = document.createElement("div");
        topRow.className = "board-row top-row";
    const topKrok = document.createElement("div");
    topKrok.className = "krok-label krok-top";

    topKrok.textContent = "";

        // Нижний ряд плит
        const bottomRow = document.createElement("div");
        bottomRow.className = "board-row bottom-row";
        const bottomKrok = document.createElement("div");
    bottomKrok.className = "krok-label krok-bottom";

    bottomKrok.textContent = "";

        // Нижние цифры (1 → 12)
    const numbersBottom = document.createElement("div");
    numbersBottom.className = "board-row bottom-numbers";


    // Нижний ряд цифр (1–11)
    for (let i = 1; i <= 11; i++) {
        const d = document.createElement("div");
        d.className = "number-cell";
        d.textContent = `KROK ${i}`;
        numbersBottom.appendChild(d);
    }


    // Нижние цифры (13–14)



    const bottom = orderedBoardIds.slice(0, 12);   // 1–12 
        const top = orderedBoardIds.slice(12, 24);     // 13–24

        // Заполняем верхний ряд (справа → налево)
        for (let i = top.length - 1; i >= 0; i--) {
            const id = top[i];
            const b = boards[id];
            const key = `${id}${b.state}`;
    const pairType = getPairType(key);

    const div = document.createElement("div");
    div.className = `board-tile ${b.state}`;
    const searchValue = document.getElementById("search-value")?.value?.trim();

    if (searchValue && b.value && b.value.includes(searchValue)) {
        div.classList.add("board-highlight");
    }


    // контент
    const content = document.createElement("div");
    content.className = "cell-content";
    content.textContent = `${key} ${b.label || ''}`;

    // нижняя полоска
    const line = document.createElement("div");
    line.className = "bottom-line line-" + (pairType || b.state.toLowerCase());

    div.appendChild(content);
    div.appendChild(line);

    topRow.appendChild(div);

        }

        // Заполняем нижний ряд (слева → направо)
        for (let id of bottom) {
            const b = boards[id];
            const key = `${id}${b.state}`;
    const pairType = getPairType(key);

    const div = document.createElement("div");
    div.className = `board-tile ${b.state}`;
    const searchValue = document.getElementById("search-value")?.value?.trim();

    if (searchValue && b.value && b.value.includes(searchValue)) {
        div.classList.add("board-highlight");
    }


    // контент
    const content = document.createElement("div");
    content.className = "cell-content";
    content.textContent = `${key} ${b.label || ''}`;

    // нижняя полоска
    const line = document.createElement("div");
    line.className = "bottom-line line-" + (pairType || b.state.toLowerCase());

    div.appendChild(content);
    div.appendChild(line);

    bottomRow.appendChild(div);

        }

        // Сборка структуры
    // --- ВЕРХНИЙ РЯД С KROK ---
    const topWrap = document.createElement("div");
    topWrap.style.display = "flex";
    topWrap.style.alignItems = "center";
    topWrap.style.gap = "10px";


    topWrap.appendChild(topRow);


    // --- НИЖНИЙ РЯД С KROK ---
    const bottomWrap = document.createElement("div");
    bottomWrap.style.display = "flex";
    bottomWrap.style.alignItems = "center";
    bottomWrap.style.gap = "10px";

    bottomWrap.appendChild(bottomKrok); // ← Добавили KROK
    bottomWrap.appendChild(bottomRow);


    // --- Финальная сборка ---
    block.appendChild(numbersTop);
    block.appendChild(topWrap);      // ← верхний ряд с KROK
    block.appendChild(bottomWrap);   // ← нижний ряд с KROK
    block.appendChild(numbersBottom);


        wrapper.appendChild(block);
        container.appendChild(wrapper);
    }


    // головна логіка вводу с контролем повторов
    function handleInput() {
        const val = document.getElementById("input").value.trim();
        const m = val.match(/^(\d+)(LS|LB|RB|RS)\s+([^\s]+)(?:\s+(.+))?$/i);
        if (!m) return;

        const [_, id, stateRaw, value, label] = m;
        const state = stateRaw.toUpperCase();

        const currentKey = `${id}|${state}|${value}|${label || ''}`;

        // 🛑 если повторный ввод подряд — игнорируем
        if (lastInputKey === currentKey) {
            document.getElementById("input").value = "";
            return;
        }

        if (boards[id]) {
            const oldState = boards[id].state;
            if (stateCount[oldState] && stateCount[oldState] > 0) {
                stateCount[oldState]--;
            }
            orderedBoardIds = orderedBoardIds.filter(x => x !== id);
        }

        boards[id] = { state, value, label };
        orderedBoardIds.unshift(id);

        stateCount[state] = (stateCount[state] || 0) + 1;
        totalCount[state] = (totalCount[state] || 0) + 1;

        lastInputKey = currentKey; // сохраняем ключ последнего ввода

        save();
        document.getElementById("input").value = "";
        updateStats();

        // блокировка ввода на выбранное время
        const inputField = document.getElementById("input");
        const lockTime = parseInt(document.getElementById("lock-time").value, 10);
        inputField.disabled = true;
        setTimeout(() => {
            inputField.disabled = false;
            inputField.focus();
        }, lockTime * 1000);
    }



        // сброс статистики
    function resetStats() {
        if (!confirm("Are you sure you want to reset stats?")) return;

        // Сбросить только счётчики, но пересчитать текущие значения на основе boards
        stateCount = { LS: 0, LB: 0, RB: 0, RS: 0 };

        // Пересчёт на основе boards
        for (const id in boards) {
            const b = boards[id];
            if (b && b.state) {
                stateCount[b.state] = (stateCount[b.state] || 0) + 1;
            }
        }

        // totalCount обнуляем как раньше
        totalCount = { LS: 0, LB: 0, RB: 0, RS: 0 };

        save();
        updateStats();
    }

        //сброс схеми досок
    function resetScheme() {
        if (!confirm("Are you sure you want to reset the board layout?")) return;
        boards = {};
        orderedBoardIds = [];
        save();
        updateStats();
    }
    function deleteBoard(id) {
        if (!boards[id]) return;

        const b = boards[id];
        const s = b.state;

        // 🧮 уменьшаем текущие и общие счётчики, если больше 0
        if (stateCount[s] > 0) stateCount[s]--;
        if (totalCount[s] > 0) totalCount[s]--;

        // 🗑 удаляем саму таблицу
        delete boards[id];

        // удаляем из порядка отображения
        orderedBoardIds = orderedBoardIds.filter(x => x !== id);

        // 💾 сохраняем и обновляем экран
        save();
        updateStats();
    }


        //оновлені дані стану з браузера на сервер
    function save() {
        const dataToSend = {
            boards: JSON.stringify(boards),
            stateCount: JSON.stringify(stateCount),     // оставить JSON.stringify — норм
            totalCount: JSON.stringify(totalCount),
            orderedBoardIds: JSON.stringify(orderedBoardIds)
        };

        fetch("", {
            method: "POST",
            headers: {'Content-Type': 'application/x-www-form-urlencoded'},
            body: new URLSearchParams(dataToSend)
        }).then(() => {
            lastUpdateTime = Math.floor(Date.now() / 1000);
        });
    }


        //запись статистики
    function downloadStats() {
        const statsText = document.getElementById("stats").textContent;

        // 2️⃣ Отправляем данные в архив
        const dataToSend = {
            boards: JSON.stringify(boards),
            stateCount: JSON.stringify(stateCount),
            totalCount: JSON.stringify(totalCount),
            orderedBoardIds: JSON.stringify(orderedBoardIds),
            forceArchive: "1"  // 🔑 Условие на сервере для записи в архив
        };

        fetch("", {
            method: "POST",
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
            body: new URLSearchParams(dataToSend)
        }).then(() => {
            console.log("📦 Данные также добавлены в архив.");
        });
    }

        // темний режим
    function toggleTheme() {
        document.body.classList.toggle("dark");
        localStorage.setItem("theme", document.body.classList.contains("dark") ? "dark" : "light");
    }

    window.addEventListener("DOMContentLoaded", () => {
        // применить тему
        if (localStorage.getItem("theme") === "dark") {
            document.body.classList.add("dark");
        }
        // 🔎 Поиск по value
    const searchInput = document.getElementById("search-value");
    if (searchInput) {
        searchInput.addEventListener("input", () => {
            renderBoardVisual();
        });
    }

        let manualHideBreak = false; // флаг, что пользователь скрыл overlay

    document.getElementById("hide-break-overlay").onclick = () => {
        document.getElementById("break-overlay").style.display = "none";
        manualHideBreak = true;
    };


        // применить сохранённое время блокировки
        const savedLockTime = localStorage.getItem("lockTime");
        if (savedLockTime) {
            document.getElementById("lock-time").value = savedLockTime;
        }

        // слушатель изменения таймера — сохраняем в localStorage
        document.getElementById("lock-time").onchange = (e) => {
            localStorage.setItem("lockTime", e.target.value);
        };

        // ✅ Инициализация галочки Break Overlay
        const showBreak = localStorage.getItem("showBreakOverlay") === "on";
        const checkbox = document.getElementById("toggle-break-overlay");
        if (checkbox) {
            checkbox.checked = showBreak;

            // слушатель для сохранения при переключении
            checkbox.onchange = () => {
                localStorage.setItem("showBreakOverlay", checkbox.checked ? "on" : "off");
            };
        }
    });


        //таймер во время записи
    let typingTimer;
    const input = document.getElementById("input");
    input.addEventListener("input", () => {
        clearTimeout(typingTimer);
        typingTimer = setTimeout(() => {
            handleInput();
        }, 1000);
    });
            // Время
    // Время
    let lastResetTime = ""; // предотвращает многократный сброс/сейв в одну минуту

    function updateClock() {
        const now = new Date();
        const hours = now.getHours().toString().padStart(2, "0");
        const minutes = now.getMinutes().toString().padStart(2, "0");
        const timeStr = now.toLocaleTimeString('pl-PL', { hour12: false });

        document.getElementById("clock").textContent = timeStr;

        // Смена
        const totalMinutes = parseInt(hours) * 60 + parseInt(minutes);
        let shiftText = '';

        if (totalMinutes >= 390 && totalMinutes < 870) {
            shiftText = '🌞 1 zmiana ';
        } else if (totalMinutes >= 870 && totalMinutes < 1350) {
            shiftText = '🌕 2 zmiana';
        } else {
            shiftText = '🌙 3 zmiana';
        }

        document.getElementById("shift-indicator").textContent = shiftText;

        // Ключ минуты, например "06:30"
        const currentTimeKey = `${hours}:${minutes}`;

        // ⏰ АВТОСБРОС в 06:30 и 14:30
        if ((currentTimeKey === "06:30" || currentTimeKey === "14:30") && lastResetTime !== currentTimeKey) {
            // Очистить totalCount
            totalCount = { LS: 0, LB: 0, RB: 0, RS: 0 };

            // Пересчитать stateCount по boards
            stateCount = { LS: 0, LB: 0, RB: 0, RS: 0 };
            for (const id in boards) {
                const b = boards[id];
                if (b && b.state) {
                    stateCount[b.state] = (stateCount[b.state] || 0) + 1;
                }
            }

            save();
            updateStats();
            lastResetTime = currentTimeKey;
            console.log("🔁 Автоматический сброс totalCount выполнен в", currentTimeKey);
        }

        // ⏰ АВТОСЕЙВ статистики в 14:20 и 22:20
        if ((currentTimeKey === "14:20" || currentTimeKey === "06:29") && lastResetTime !== "archive-" + currentTimeKey) {
            const dataToSend = {
                boards: JSON.stringify(boards),
                stateCount: JSON.stringify(stateCount),
                totalCount: JSON.stringify(totalCount),
                orderedBoardIds: JSON.stringify(orderedBoardIds),
                forceArchive: "1"
            };

            fetch("", {
                method: "POST",
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                body: new URLSearchParams(dataToSend)
            }).then(() => {
                console.log("📦 Автосейв статистики выполнен в", currentTimeKey);
            });

            lastResetTime = "archive-" + currentTimeKey; // чтобы не повторялось каждую секунду
        }
    }


    setInterval(updateClock, 1000);
    updateClock();
    setInterval(fetchDataAndUpdate, 2000); // обновлять данные каждые 2 секунды

    document.getElementById("lang").onchange = (e) => {
        location.href = "?lang=" + e.target.value;
    };

    updateStats();
    let inactivityTimer;

    function focusInput() {
        const inputField = document.getElementById("input");
        if (document.activeElement !== inputField) {
            inputField.focus();
        }
    }
            // Автофокус
    function resetInactivityTimer() {
        clearTimeout(inactivityTimer);
        inactivityTimer = setTimeout(focusInput, 2000); // 1 секунд
    }

    ['mousemove', 'keydown', 'click', 'touchstart'].forEach(evt =>
        document.addEventListener(evt, resetInactivityTimer)
    );

    resetInactivityTimer();

    function showHistory() {
        fetch("history.json?_=" + Date.now())
            .then(res => res.json())
            .then(data => {
                let html = '';
                data.forEach((entry, index) => {
                    const timeStr = new Date(entry.lastUpdate * 1000).toLocaleString();
                    const totalBoards = Object.values(entry.totalCount || {}).reduce((a, b) => a + b, 0);

                    html += `<div style="margin-bottom:10px; border-bottom:1px solid #555; padding:2px;">
                        <b>📦Zapis #${index + 1} — ${timeStr}</b><br>
                        Desek: ${totalBoards} pcs<br>
                        Bieżące: ${JSON.stringify(entry.stateCount)}<br>
                        Łącznie: ${JSON.stringify(entry.totalCount)}<br>
                        <button onclick="deleteHistory(${index})" 
            style="font-size:12px; padding:2px 6px; border-radius:4px;"> 🗑</button>

                    </div>`;
                });

                document.getElementById('history-content').innerHTML = html;
                document.getElementById('history-modal').style.display = 'block';
            })
            .catch(err => {
                console.error("Ошибка загрузки history.json:", err);
                alert("❌ Ошибка чтения архива.");
            });
    }

    function deleteHistory(index) {
        if (!confirm("❗ Czy chcesz usunąć ten zapis z archiwum?")) return;

        fetch("delete_history.php", {
            method: "POST",
            headers: { "Content-Type": "application/x-www-form-urlencoded" },
            body: "index=" + index
        })
        .then(res => res.json())
        .then(resp => {
            if (resp.success) {
                // обновляем список, чтобы запись исчезла
                showHistory();
            } else {
                alert("❌ Ошибка: " + (resp.error || "неизвестно"));
            }
        })
        .catch(err => {
            alert("⚠️ Ошибка при соединении: " + err);
        });
    }

    let fullHistoryData = []; // хранит весь архив

    function showHistory() {
        fetch("history.json?_=" + Date.now())
            .then(res => res.json())
            .then(data => {
                fullHistoryData = data; // сохраним весь архив
                renderHistory(data);
                document.getElementById('history-modal').style.display = 'block';
            })
            .catch(err => {
                console.error("Ошибка загрузки history.json:", err);
                alert("❌ Ошибка чтения архива.");
            });
    }

    function renderHistory(data) {
        let html = '';
        data.forEach((entry, index) => {
            const timeStr = new Date(entry.lastUpdate * 1000).toLocaleString();
            const dateOnly = new Date(entry.lastUpdate * 1000).toISOString().slice(0,10);
            const totalBoards = Object.values(entry.totalCount || {}).reduce((a, b) => a + b, 0);

            html += `<div style="margin-bottom:10px; border-bottom:1px solid #555; padding:2px;">
                <b>📦 Zapis #${index + 1} — ${timeStr}</b><br>
                Desek: ${totalBoards} pcs<br>
                Bieżące: ${JSON.stringify(entry.stateCount)}<br>
                Łącznie: ${JSON.stringify(entry.totalCount)}<br>
                <button class="small-btn" onclick="deleteHistory(${index})">🗑</button>
            </div>`;
        });

        document.getElementById('history-content').innerHTML = html || "⚠️ Brak zapisów w tym zakresie.";
    }

    function applyHistoryFilter() {
        const start = document.getElementById("filter-start").value;
        const end = document.getElementById("filter-end").value;

        if (!start && !end) {
            renderHistory(fullHistoryData);
            return;
        }

        const filtered = fullHistoryData.filter(entry => {
            const entryDate = new Date(entry.lastUpdate * 1000).toISOString().slice(0,10);
            return (!start || entryDate >= start) && (!end || entryDate <= end);
        });

        renderHistory(filtered);
    }

    function clearHistoryFilter() {
        document.getElementById("filter-start").value = "";
        document.getElementById("filter-end").value = "";
        renderHistory(fullHistoryData);
    }



    function clearHistory() {
        if (!confirm("❗ Czy na pewno chcesz wyczyścić archiwum? Ta czynność jest nieodwracalna.")) return;

        fetch("clear_history.php", {
            method: "POST"
        }).then(res => {
            if (res.ok) {
                alert("✅ Archiwum zostało pomyślnie wyczyszczone..");
                document.getElementById("history-content").textContent = '';
            } else {
                alert("❌ Błąd podczas usuwania archiwum.");
            }
        }).catch(err => {
            alert("⚠️ Błąd połączenia: " + err);
        });
    }

    function showAdminLogin() {
        document.getElementById("admin-login-modal").style.display = "flex";
        document.getElementById("admin-password").value = "";
        document.getElementById("admin-error").style.display = "none";
        document.getElementById("admin-password").focus();
    }

    function closeAdminLogin() {
        document.getElementById("admin-login-modal").style.display = "none";
    }

    function checkAdminPassword() {
        const pass = document.getElementById("admin-password").value.trim();
        if (pass === "PKC") {   // ← твой пароль
            document.getElementById("admin-login-modal").style.display = "none";
            document.getElementById("admin-panel").style.display = "block";
        } else {
            document.getElementById("admin-error").style.display = "block";
        }
    }

    function closeAdminPanel() {
        document.getElementById("admin-panel").style.display = "none";
    }

    window.addEventListener("load", () => {
        const saved = localStorage.getItem("breakOverlayEnabled");

        if (saved !== null) {
            breakOverlayEnabled = saved === "1";
        }

        updateBreakOverlay(); // проверка сразу при загрузке
    });


    window.addEventListener("DOMContentLoaded", () => {

        const savedPercents = localStorage.getItem("stagePercents");

        if (savedPercents) {
            stagePercents = JSON.parse(savedPercents);

            const s1e1 = document.getElementById("s1_e1");
            const s1e2 = document.getElementById("s1_e2");
            const s1e3 = document.getElementById("s1_e3");
            const s2e1 = document.getElementById("s2_e1");
            const s2e2 = document.getElementById("s2_e2");
            const s2e3 = document.getElementById("s2_e3");

            if (s1e1) s1e1.value = stagePercents.shift1[0] * 100;
            if (s1e2) s1e2.value = stagePercents.shift1[1] * 100;
            if (s1e3) s1e3.value = stagePercents.shift1[2] * 100;

            if (s2e1) s2e1.value = stagePercents.shift2[0] * 100;
            if (s2e2) s2e2.value = stagePercents.shift2[1] * 100;
            if (s2e3) s2e3.value = stagePercents.shift2[2] * 100;
        }

    });


    let autosaveTimes = []; // список часов:минут

    function updateAutosaveTimes() {
        const t1 = document.getElementById("autosave-time1").value;
        const t2 = document.getElementById("autosave-time2").value;

        autosaveTimes = [];
        if (t1) autosaveTimes.push(t1);
        if (t2) autosaveTimes.push(t2);

        localStorage.setItem("autosaveTimes", JSON.stringify(autosaveTimes));
        console.log("💾 Saved AutoSave times:", autosaveTimes);
    }

    function saveStagePercents() {
        stagePercents.shift1 = [
            document.getElementById("s1_e1").value / 100,
            document.getElementById("s1_e2").value / 100,
            document.getElementById("s1_e3").value / 100
        ];

        stagePercents.shift2 = [
            document.getElementById("s2_e1").value / 100,
            document.getElementById("s2_e2").value / 100,
            document.getElementById("s2_e3").value / 100
        ];

        localStorage.setItem("stagePercents", JSON.stringify(stagePercents));
        updateSpeed();
    }


    let lastAutosaveMinute = null;

    setInterval(() => {
        const now = new Date();

        // 🔄 обновляем скорость каждую секунду
        updateSpeed();

        // 💾 автосейв — 1 раз в нужную минуту
        const current = now.toTimeString().slice(0,5); // HH:MM

        if (
            autosaveTimes.includes(current) &&
            lastAutosaveMinute !== current
        ) {
            autosaveToArchive();
            lastAutosaveMinute = current;
            console.log("✅ AutoSaved to archive at", current);
        }

    }, 1000); // ⏱ каждую секунду



    // при загрузке подтягиваем сохранённые времена
    window.addEventListener("load", () => {
        const saved = JSON.parse(localStorage.getItem("autosaveTimes") || "[]");
        autosaveTimes = saved;

        if (saved[0]) document.getElementById("autosave-time1").value = saved[0];
        if (saved[1]) document.getElementById("autosave-time2").value = saved[1];
    });

    function autosaveToArchive() {
        const dataToSend = {
            boards: JSON.stringify(boards),
            stateCount: JSON.stringify(stateCount),
            totalCount: JSON.stringify(totalCount),
            orderedBoardIds: JSON.stringify(orderedBoardIds),
            forceArchive: "1"  // 🔑 условие для записи в архив
        };

        fetch("", {
            method: "POST",
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
            body: new URLSearchParams(dataToSend)
        }).then(() => {
            console.log("📦 Автосейв в архив выполнен");
        }).catch(err => {
            console.error("Ошибка автосейва в архив:", err);
        });
    }


    function toggleBreakOverlay() {
        breakOverlayEnabled = document.getElementById("break-toggle").checked;
        localStorage.setItem("breakOverlayEnabled", breakOverlayEnabled ? "1" : "0");
        updateBreakOverlay();
    }



    function isBreakTime() {
        const now = new Date();
        const h = now.getHours();
        const m = now.getMinutes();
        const current = h * 60 + m;

        const breaks = [
            [9*60 + 40, 10*60],     // 09:40–10:00
            [12*60 + 10, 12*60+20], // 12:10–12:20
            [18*60, 18*60+20],      // 18:00–18:20
            [20*60, 20*60+10]       // 20:00–20:10
        ];

        return breaks.some(([start, end]) => current >= start && current < end);
    }

    function updateBreakOverlay() {
        const overlay = document.getElementById("break-overlay");
        if (!overlay) return;

        if (breakOverlayEnabled && isBreakTime()) {
            overlay.style.display = "flex";
        } else {
            overlay.style.display = "none";
        }
    }

    function getShiftStage(now) {
        const min = now.getHours() * 60 + now.getMinutes();
        const isShift1 = min >= 390 && min < 860;
        const p = isShift1 ? stagePercents.shift1 : stagePercents.shift2;

        if (isShift1) {
            if (min < 580) return { stage: 1, target: p[0] };
            if (min < 730) return { stage: 2, target: p[0] + p[1] };
            if (min < 860) return { stage: 3, target: 1 };
        } else {
            if (min < 1080) return { stage: 1, target: p[0] };
            if (min < 1200) return { stage: 2, target: p[0] + p[1] };
            if (min < 1340) return { stage: 3, target: 1 };
        }

        return null;
    }

    window.addEventListener("DOMContentLoaded", () => {

        const saved = localStorage.getItem("breakOverlayEnabled");

        if (saved !== null) {
            breakOverlayEnabled = saved === "1";
        }

        const toggle = document.getElementById("break-toggle");
        if (toggle) {
            toggle.checked = breakOverlayEnabled;
        }

        updateBreakOverlay();
        setInterval(updateBreakOverlay, 30000);

    });





    </script>

    <!-- Модальное окно истории -->
    <!-- ...весь твой HTML ... -->

    <!-- Модальное окно истории -->
    <div id="history-modal" style="display:none; position:fixed; top:0; left:0; width:100vw; height:100vh;
        background:rgba(0,0,0,0.8); color:white; overflow:auto; padding:20px; z-index:2000;">
        
        <button onclick="document.getElementById('history-modal').style.display='none'"
            style="position:absolute; top:10px; right:10px; font-size:20px;">❌</button>

        <h2>📂 Historia zapisów</h2>

        <!-- 🔎 Панель фильтра -->
        <div style="margin-bottom:15px;">
            <label>Od: <input type="date" id="filter-start"></label>
            <label>Do: <input type="date" id="filter-end"></label>
            <button onclick="applyHistoryFilter()">🔎 Filtruj</button>
            <button onclick="clearHistoryFilter()">❌ Reset</button>
        </div>

        <div id="history-content"></div>
    </div>
    <!-- Модальное окно для ввода пароля -->
    <div id="admin-login-modal" style="display:none; position:fixed; top:0; left:0; width:100vw; height:100vh;
        background:rgba(0,0,0,0.8); color:white; z-index:3000; align-items:center; justify-content:center;">

        <div style="background:#222; padding:20px; border-radius:8px; min-width:300px; text-align:center;">
            <h3>🔑 Admin Login</h3>
            <input type="password" id="admin-password" placeholder="Enter password" style="padding:6px; width:90%;" />
            <div style="margin-top:10px;">
                <button onclick="checkAdminPassword()">✅ OK</button>
                <button onclick="closeAdminLogin()">❌ Cancel</button>
            </div>
            <p id="admin-error" style="color:red; display:none; margin-top:8px;">❌ Wrong password!</p>
        </div>
    </div>
    <!-- Админ-панель -->
    <div id="admin-panel" style="display:none; position:fixed; top:0; left:0; width:100vw; height:100vh;
        background:rgba(0,0,0,0.9); color:white; z-index:4000; overflow:auto; padding:20px;">
        
        <button onclick="closeAdminPanel()" 
            style="position:absolute; top:10px; right:10px; font-size:20px;">❌</button>

        <h2>⚙️ Admin Panel</h2>
        
        <!-- Блок автосейва -->
        <div style="margin-top:20px;">
            <h3>💾 AutoSave schedule</h3>
            <label>⏰ First save time: 
                <input type="time" id="autosave-time1" onchange="updateAutosaveTimes()">
            </label><br><br>
            <label>⏰ Second save time: 
                <input type="time" id="autosave-time2" onchange="updateAutosaveTimes()">
            </label>
        </div>

        <!-- Блок перерыва -->
        <div style="margin-top:20px;">
            <h3>🔔 Break overlay</h3>
            <label>
                <input type="checkbox" id="break-toggle" onchange="toggleBreakOverlay()"> Show break overlay
            </label>
        </div>
        
        <!-- Проценты этапов -->
    <div style="margin-top:20px;">
        <h3>📊 Speed stages (%)</h3>

        <b>1 zmiana</b><br>
        Etap 1: <input type="number" id="s1_e1" value="50" style="width:60px;"> %
        Etap 2: <input type="number" id="s1_e2" value="25" style="width:60px;"> %
        Etap 3: <input type="number" id="s1_e3" value="25" style="width:60px;"> %
        <br><br>

        <b>2 zmiana</b><br>
        Etap 1: <input type="number" id="s2_e1" value="50" style="width:60px;"> %
        Etap 2: <input type="number" id="s2_e2" value="25" style="width:60px;"> %
        Etap 3: <input type="number" id="s2_e3" value="25" style="width:60px;"> %

        <br><br>
        <button onclick="saveStagePercents()">💾 Save</button>
    </div>


    </div>
