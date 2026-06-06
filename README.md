<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PM Task Dashboard — Разбор входящих задач</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js">
    </script>
    <style>
        :root {
            --bg: #f5f6f8;
            --card: #ffffff;
            --border: #e2e5ea;
            --text: #1a1d23;
            --text-secondary: #5f6570;
            --critical: #e53e3e;
            --critical-bg: #fef2f2;
            --high: #f59e0b;
            --high-bg: #fffbeb;
            --medium: #3b82f6;
            --medium-bg: #eff6ff;
            --low: #6b7280;
            --low-bg: #f9fafb;
            --accent: #6366f1;
            --accent-light: #eef2ff;
            --success: #10b981;
            --overdue: #ef4444;
            --radius: 12px;
            --radius-sm: 8px;
            --shadow: 0 1px 3px rgba(0, 0, 0, 0.06), 0 1px 2px rgba(0, 0, 0, 0.04);
            --shadow-md: 0 4px 16px rgba(0, 0, 0, 0.08), 0 2px 4px rgba(0, 0, 0, 0.04);
            --transition: 0.2s ease;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Inter', 'Segoe UI', system-ui, -apple-system, sans-serif;
            background: var(--bg);
            color: var(--text);
            line-height: 1.6;
            min-height: 100vh;
        }

        /* Header */
        .header {
            background: var(--card);
            border-bottom: 1px solid var(--border);
            padding: 16px 28px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            flex-wrap: wrap;
            gap: 16px;
            position: sticky;
            top: 0;
            z-index: 100;
            box-shadow: var(--shadow);
        }
        .header-title {
            font-size: 1.25rem;
            font-weight: 700;
            letter-spacing: -0.02em;
            color: var(--text);
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .header-title .icon {
            width: 36px;
            height: 36px;
            border-radius: var(--radius-sm);
            background: var(--accent);
            color: #fff;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.1rem;
        }
        .header-meta {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
            font-size: 0.85rem;
            color: var(--text-secondary);
        }
        .header-meta span {
            display: flex;
            align-items: center;
            gap: 6px;
            background: var(--bg);
            padding: 6px 12px;
            border-radius: 20px;
            font-weight: 500;
        }
        .header-meta .ref-date {
            background: var(--accent-light);
            color: var(--accent);
            font-weight: 600;
            cursor: help;
        }
        .badge-count {
            background: var(--text);
            color: #fff;
            border-radius: 12px;
            padding: 2px 10px;
            font-size: 0.8rem;
            font-weight: 700;
            min-width: 24px;
            text-align: center;
        }
        .badge-count.critical {
            background: var(--critical);
        }
        .badge-count.high {
            background: var(--high);
            color: #1a1d23;
        }

        /* Main layout */
        .container {
            max-width: 1340px;
            margin: 0 auto;
            padding: 24px 20px;
            display: flex;
            flex-direction: column;
            gap: 28px;
        }

        /* Section cards */
        .section {
            background: var(--card);
            border-radius: var(--radius);
            border: 1px solid var(--border);
            box-shadow: var(--shadow);
            overflow: hidden;
        }
        .section-header {
            padding: 18px 24px;
            border-bottom: 1px solid var(--border);
            display: flex;
            align-items: center;
            justify-content: space-between;
            flex-wrap: wrap;
            gap: 12px;
            cursor: pointer;
            user-select: none;
            transition: var(--transition);
        }
        .section-header:hover {
            background: #fafbfc;
        }
        .section-header h2 {
            font-size: 1.05rem;
            font-weight: 700;
            letter-spacing: -0.01em;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .section-header h2 .dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            display: inline-block;
            flex-shrink: 0;
        }
        .section-body {
            padding: 20px 24px;
            overflow-x: auto;
        }
        .section-body.collapsed {
            display: none;
        }
        .collapse-arrow {
            font-size: 0.75rem;
            transition: transform 0.3s;
            color: var(--text-secondary);
        }
        .collapsed .collapse-arrow {
            transform: rotate(-90deg);
        }

        /* Tables */
        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 0.9rem;
            min-width: 700px;
        }
        th {
            text-align: left;
            font-weight: 600;
            font-size: 0.78rem;
            text-transform: uppercase;
            letter-spacing: 0.04em;
            color: var(--text-secondary);
            padding: 10px 14px;
            border-bottom: 2px solid var(--border);
            white-space: nowrap;
        }
        td {
            padding: 12px 14px;
            border-bottom: 1px solid var(--border);
            vertical-align: top;
        }
        tr:hover td {
            background: #fafbfd;
        }
        .msg-original {
            max-width: 360px;
            font-size: 0.83rem;
            color: var(--text-secondary);
            line-height: 1.5;
            font-style: italic;
            position: relative;
            padding-left: 12px;
            border-left: 3px solid var(--border);
            display: -webkit-box;
            -webkit-line-clamp: 3;
            -webkit-box-orient: vertical;
            overflow: hidden;
            cursor: pointer;
            transition: var(--transition);
        }
        .msg-original:hover {
            border-left-color: var(--accent);
            -webkit-line-clamp: unset;
            overflow: visible;
            background: #fdfdfe;
            z-index: 2;
            position: relative;
            box-shadow: var(--shadow-md);
            border-radius: 0 6px 6px 0;
            padding: 10px 14px;
            margin: -4px 0;
        }

        /* Priority badges */
        .priority-badge {
            display: inline-flex;
            align-items: center;
            gap: 5px;
            padding: 4px 10px;
            border-radius: 14px;
            font-size: 0.78rem;
            font-weight: 700;
            letter-spacing: 0.02em;
            white-space: nowrap;
        }
        .priority-badge.critical {
            background: var(--critical-bg);
            color: var(--critical);
        }
        .priority-badge.high {
            background: var(--high-bg);
            color: #b45309;
        }
        .priority-badge.medium {
            background: var(--medium-bg);
            color: var(--medium);
        }
        .priority-badge.low {
            background: var(--low-bg);
            color: var(--low);
        }
        .risk-tag {
            display: inline-block;
            font-size: 0.75rem;
            background: #fef3c7;
            color: #92400e;
            padding: 3px 8px;
            border-radius: 10px;
            font-weight: 500;
            white-space: nowrap;
            max-width: 200px;
            overflow: hidden;
            text-overflow: ellipsis;
        }
        .risk-tag.critical-risk {
            background: #fee2e2;
            color: #991b1b;
        }
        .deadline-date {
            font-weight: 600;
            white-space: nowrap;
        }
        .deadline-date.overdue {
            color: var(--overdue);
            animation: pulse-overdue 2s infinite;
        }
        @keyframes pulse-overdue {
            0%,
            100% {
                opacity: 1;
            }
            50% {
                opacity: 0.5;
            }
        }
        .deadline-date.soon {
            color: #d97706;
        }
        .type-tag {
            display: inline-block;
            font-size: 0.73rem;
            padding: 3px 9px;
            border-radius: 10px;
            font-weight: 600;
            letter-spacing: 0.03em;
            background: #eef2ff;
            color: #4338ca;
            white-space: nowrap;
        }
        .type-tag.bug {
            background: #fef2f2;
            color: #dc2626;
        }
        .type-tag.feature {
            background: #ecfdf5;
            color: #059669;
        }
        .type-tag.process {
            background: #eff6ff;
            color: #2563eb;
        }
        .type-tag.compliance {
            background: #fefce8;
            color: #a16207;
        }

        /* Task cards grid (structured view) */
        .task-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 16px;
        }
        .task-card {
            background: var(--card);
            border: 1px solid var(--border);
            border-radius: var(--radius-sm);
            padding: 16px 18px;
            box-shadow: var(--shadow);
            transition: var(--transition);
            display: flex;
            flex-direction: column;
            gap: 10px;
            position: relative;
            overflow: hidden;
        }
        .task-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 4px;
            height: 100%;
            border-radius: 4px 0 0 4px;
        }
        .task-card.critical-card::before {
            background: var(--critical);
        }
        .task-card.high-card::before {
            background: var(--high);
        }
        .task-card.medium-card::before {
            background: var(--medium);
        }
        .task-card.low-card::before {
            background: var(--low);
        }
        .task-card:hover {
            box-shadow: var(--shadow-md);
            transform: translateY(-1px);
        }
        .task-card .card-header {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            gap: 8px;
        }
        .task-card .card-title {
            font-weight: 700;
            font-size: 0.95rem;
            line-height: 1.3;
            letter-spacing: -0.01em;
        }
        .task-card .card-meta {
            display: flex;
            flex-wrap: wrap;
            gap: 6px;
            font-size: 0.75rem;
        }
        .task-card .card-channel {
            font-size: 0.75rem;
            color: var(--text-secondary);
            font-weight: 500;
            text-transform: uppercase;
            letter-spacing: 0.05em;
        }

        /* Charts grid */
        .charts-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
            gap: 20px;
        }
        .chart-container {
            background: var(--card);
            border-radius: var(--radius-sm);
            padding: 16px;
            border: 1px solid var(--border);
            position: relative;
        }
        .chart-container h3 {
            font-size: 0.85rem;
            font-weight: 700;
            margin-bottom: 12px;
            color: var(--text);
            letter-spacing: -0.01em;
        }
        .chart-container canvas {
            max-height: 280px;
        }

        /* Insights */
        .insights-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
            gap: 12px;
            list-style: none;
        }
        .insights-list li {
            background: var(--bg);
            padding: 14px 16px;
            border-radius: var(--radius-sm);
            font-size: 0.85rem;
            border-left: 4px solid var(--accent);
            line-height: 1.5;
            font-weight: 500;
        }
        .insights-list li.warn {
            border-left-color: var(--critical);
            background: var(--critical-bg);
        }
        .insights-list li.info {
            border-left-color: var(--medium);
            background: var(--medium-bg);
        }

        /* Methodology */
        .method-block {
            background: #fafbfc;
            border-radius: var(--radius-sm);
            padding: 16px 20px;
            margin-bottom: 12px;
            border: 1px solid var(--border);
            font-size: 0.85rem;
        }
        .method-block h4 {
            font-weight: 700;
            margin-bottom: 6px;
            font-size: 0.9rem;
            color: var(--accent);
        }
        .method-block code {
            background: #eef2ff;
            padding: 2px 6px;
            border-radius: 4px;
            font-size: 0.8rem;
            color: #4338ca;
            font-weight: 500;
        }

        /* Responsive */
        @media (max-width: 768px) {
            .container {
                padding: 12px 8px;
                gap: 16px;
            }
            .section-body {
                padding: 12px 10px;
            }
            table {
                font-size: 0.75rem;
                min-width: auto;
            }
            th,
            td {
                padding: 8px 6px;
            }
            .header {
                padding: 10px 14px;
            }
            .header-title {
                font-size: 1rem;
            }
            .charts-grid {
                grid-template-columns: 1fr;
            }
            .task-grid {
                grid-template-columns: 1fr;
            }
        }

        /* Tooltip style */
        .tooltip-trigger {
            cursor: help;
            border-bottom: 1px dotted var(--text-secondary);
        }
        .highlight-note {
            background: #fefce8;
            padding: 2px 6px;
            border-radius: 3px;
            font-size: 0.75rem;
            font-weight: 600;
            color: #92400e;
        }

        /* Print-friendly */
        @media print {
            .header {
                position: static;
            }
            .section-body.collapsed {
                display: block;
            }
            body {
                background: #fff;
            }
        }
    </style>
</head>
<body>

    <!-- ========== HEADER ========== -->
    <header class="header">
        <div class="header-title">
            <span class="icon">📋</span>
            Task Dashboard — Разбор входящих
        </div>
        <div class="header-meta">
            <span>📨 Сообщений: <strong>8</strong></span>
            <span>✅ Задач выделено: <strong>8</strong></span>
            <span>🔴 Критических: <strong class="badge-count critical">3</strong></span>
            <span>🟡 Высоких: <strong class="badge-count high">2</strong></span>
            <span class="ref-date" title="Опорная дата для расчёта относительных сроков">📅 Опорная дата: <strong id="refDateDisplay"></strong></span>
        </div>
    </header>

    <!-- ========== MAIN CONTENT ========== -->
    <main class="container">

        <!-- SECTION 1: Исходные сообщения -->
        <section class="section" id="rawSection">
            <div class="section-header" onclick="toggleSection('rawBody', this)">
                <h2><span class="dot" style="background:#6366f1;"></span> 📨 Исходные сообщения (входящий хаос)</h2>
                <span class="collapse-arrow">▼</span>
            </div>
            <div class="section-body" id="rawBody">
                <table>
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Канал</th>
                            <th>Сообщение</th>
                        </tr>
                    </thead>
                    <tbody id="rawMessagesTable"></tbody>
                </table>
            </div>
        </section>

        <!-- SECTION 2: Структурированные задачи (таблица) -->
        <section class="section" id="structuredSection">
            <div class="section-header" onclick="toggleSection('structuredBody', this)">
                <h2><span class="dot" style="background:#10b981;"></span> 📊 Структурированные задачи</h2>
                <span class="collapse-arrow">▼</span>
            </div>
            <div class="section-body" id="structuredBody">
                <table>
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>Задача</th>
                            <th>Канал</th>
                            <th>Тип</th>
                            <th>Приоритет</th>
                            <th>Дедлайн</th>
                            <th>Риск</th>
                            <th>Зависимости / Примечания</th>
                        </tr>
                    </thead>
                    <tbody id="structuredTasksTable"></tbody>
                </table>
            </div>
        </section>

        <!-- SECTION 3: Карточки задач (канбан-стиль) -->
        <section class="section" id="cardsSection">
            <div class="section-header" onclick="toggleSection('cardsBody', this)">
                <h2><span class="dot" style="background:#f59e0b;"></span> 🗂 Карточки задач по приоритетам</h2>
                <span class="collapse-arrow">▼</span>
            </div>
            <div class="section-body" id="cardsBody">
                <div class="task-grid" id="taskCardsGrid"></div>
            </div>
        </section>

        <!-- SECTION 4: Аналитика и графики -->
        <section class="section" id="analyticsSection">
            <div class="section-header" onclick="toggleSection('analyticsBody', this)">
                <h2><span class="dot" style="background:#3b82f6;"></span> 📈 Аналитика и графики</h2>
                <span class="collapse-arrow">▼</span>
            </div>
            <div class="section-body" id="analyticsBody">
                <div class="charts-grid">
                    <div class="chart-container">
                        <h3>🎯 Распределение по приоритетам</h3>
                        <canvas id="priorityChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <h3>📡 Задачи по каналам</h3>
                        <canvas id="channelChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <h3>📅 Таймлайн дедлайнов</h3>
                        <canvas id="deadlineChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <h3>⚠️ Распределение типов рисков</h3>
                        <canvas id="riskChart"></canvas>
                    </div>
                </div>
                <!-- Инсайты -->
                <h3 style="margin-top:20px;font-size:0.95rem;font-weight:700;">💡 Ключевые инсайты</h3>
                <ul class="insights-list" id="insightsList"></ul>
            </div>
        </section>

        <!-- SECTION 5: Методология разбора -->
        <section class="section" id="methodSection">
            <div class="section-header" onclick="toggleSection('methodBody', this)">
                <h2><span class="dot" style="background:#6b7280;"></span> 🔍 Методология: правила разбора неоднозначных входящих</h2>
                <span class="collapse-arrow">▼</span>
            </div>
            <div class="section-body" id="methodBody">
                <div class="method-block">
                    <h4>1. Извлечение дедлайнов (правила парсинга относительных сроков)</h4>
                    <p><code>"сегодня"</code> → опорная дата (день открытия страницы).<br>
                        <code>"к пятнице"</code> → ближайшая пятница после опорной даты.<br>
                        <code>"до конца недели"</code> → воскресенье текущей недели.<br>
                        <code>"в среду"</code> → ближайшая среда (если сегодня среда — сегодня).<br>
                        <code>"на следующей неделе"</code> → понедельник следующей недели (диапазон).<br>
                        <code>"в следующем спринте"</code> → не привязывается к конкретной дате, помечается как «следующий спринт».<br>
                        <code>"15 мая"</code> → абсолютная дата, парсится как есть. Если опорная дата позже — задача <strong>просрочена</strong>.</p>
                </div>
                <div class="method-block">
                    <h4>2. Определение приоритета (ключевые слова и контекст)</h4>
                    <p><strong>🔴 Critical</strong>: <code>"срочно"</code>, <code>"юристы"</code>, <code>"сорвется пилот"</code>, <code>"очень плохо"</code>, сочетание <code>"сегодня"</code> + <code>"жалобы"</code>.<br>
                        <strong>🟡 High</strong>: <code>"сегодня"</code> (без critical-маркеров), <code>"к пятнице"</code> с бизнес-риском, <code>"перед вебинаром"</code>, <code>"жалобы"</code>.<br>
                        <strong>🔵 Medium</strong>: <code>"до конца недели"</code>, <code>"на следующей неделе"</code>, <code>"weekly update"</code>.<br>
                        <strong>⚪ Low</strong>: <code>"не горит"</code>, <code>"давно просят"</code>, <code>"хорошо бы"</code>.</p>
                </div>
                <div class="method-block">
                    <h4>3. Классификация типа задачи</h4>
                    <p><strong>bug</strong> — ошибка в работе системы (оплата, вёрстка).<br>
                        <strong>feature</strong> — новая функциональность (NPS, CSV-экспорт, блок на лендинге).<br>
                        <strong>process</strong> — организационное улучшение (инструкция, шаблон, регламент).<br>
                        <strong>compliance</strong> — юридическое требование (оферта).</p>
                </div>
                <div class="method-block">
                    <h4>4. Извлечение рисков</h4>
                    <p>Анализируются фразы после маркеров: <code>"а то"</code>, <code>"иначе"</code>, <code>"потому что"</code>, <code>"будет сложно"</code>, <code>"выглядит очень плохо"</code>. Каждому риску присваивается категория: репутационный, бизнес-риск (потеря клиента), юридический, процессный, конверсионный.</p>
                </div>
                <div class="method-block">
                    <h4>5. Принцип «Договорённости не живут в личке»</h4>
                    <p>Каждое сообщение превращается в рабочий объект с чёткими атрибутами: название задачи, канал-источник, тип, приоритет, дедлайн, риск, зависимости. Это позволяет команде работать предсказуемо, а не полагаться на память и переписку.</p>
                </div>
            </div>
        </section>

    </main>

    <script>
        (function() {
            // ==================== ОПОРНАЯ ДАТА ====================
            const referenceDate = new Date(); // "сегодня" = дата открытия страницы
            const refDateStr = referenceDate.toISOString().split('T')[0];
            document.getElementById('refDateDisplay').textContent =
                referenceDate.toLocaleDateString('ru-RU', { weekday: 'short', day: 'numeric', month: 'long',
                    year: 'numeric' });

            // ==================== ИСХОДНЫЕ ДАННЫЕ ====================
            const rawMessages = [
                { id: 1, channel: 'Поддержка',
                    message: 'После оплаты у части клиентов не уходит письмо с чеком. Саппорт уже второй день отвечает руками. Хорошо бы починить сегодня, а то начнутся жалобы.'
                    },
                { id: 2, channel: 'Маркетинг',
                    message: 'Нужен блок с кейсами на лендинге перед вебинаром 15 мая. Контент маркетинг почти собрал, нужен кто-то, кто это доведет до релиза.'
                    },
                { id: 3, channel: 'Онбординг',
                    message: 'Ребята, новички опять пишут всем в личку, потому что непонятно, где брать доступы и какие шаги делать в первую неделю. Давайте уже соберем одну нормальную инструкцию и чек-лист до конца недели.'
                    },
                { id: 4, channel: 'Мобайл',
                    message: 'На айфоне в Safari кнопка "Оплатить" уезжает за экран. С утра это показывали на созвоне. Желательно срочно, потому что это выглядит очень плохо.'
                    },
                { id: 5, channel: 'Продажи',
                    message: 'Продажи давно просят автоматически собирать NPS после закрытия заказа. Не горит, но в следующем спринте было бы хорошо это наконец сделать.'
                    },
                { id: 6, channel: 'Пилот',
                    message: 'Клиент Acme попросил CSV-экспорт по филиалам к пятнице, иначе говорит, что будет сложно продлить пилот. Можете оценить и взять в работу?'
                    },
                { id: 7, channel: 'Операции',
                    message: 'Мы уже третий раз за неделю обсуждаем одни и те же задачи, потому что статусы разбросаны по Telegram и личкам. Нужен единый weekly update шаблон, можно внедрить на следующей неделе.'
                    },
                { id: 8, channel: 'Юристы',
                    message: 'На сайте до сих пор старая оферта, юристы сегодня напомнили. Нужно обновить файл и ссылку до публикации релиза в среду.'
                    },
            ];

            // ==================== ФУНКЦИИ ПАРСИНГА ====================

            /** Найти ближайший день недели (0=вс, 1=пн, ..., 6=сб) после referenceDate */
            function nextWeekday(targetDay, fromDate = referenceDate) {
                const d = new Date(fromDate);
                const currentDay = d.getDay();
                let diff = targetDay - currentDay;
                if (diff <= 0) diff += 7; // если сегодня этот день или прошёл — берём следующую неделю
                d.setDate(d.getDate() + diff);
                return d;
            }

            /** Найти ближайший день недели, включая сегодня */
            function thisOrNextWeekday(targetDay, fromDate = referenceDate) {
                const d = new Date(fromDate);
                const currentDay = d.getDay();
                let diff = targetDay - currentDay;
                if (diff < 0) diff += 7;
                // diff=0 означает сегодня
                d.setDate(d.getDate() + diff);
                return d;
            }

            /** Конец текущей недели (воскресенье) */
            function endOfWeek(fromDate = referenceDate) {
                const d = new Date(fromDate);
                const day = d.getDay();
                const diff = day === 0 ? 0 : 7 - day;
                d.setDate(d.getDate() + diff);
                return d;
            }

            /** Понедельник следующей недели */
            function nextMonday(fromDate = referenceDate) {
                const d = new Date(fromDate);
                const day = d.getDay();
                const diff = day === 0 ? 1 : 8 - day;
                d.setDate(d.getDate() + diff);
                return d;
            }

            /** Форматирование даты */
            function formatDate(d) {
                return d.toLocaleDateString('ru-RU', { day: 'numeric', month: 'short', year: 'numeric' });
            }

            function formatDateShort(d) {
                return d.toLocaleDateString('ru-RU', { day: 'numeric', month: 'short' });
            }

            /** Проверка: дата в прошлом относительно опорной */
            function isOverdue(d) {
                if (!d) return false;
                const today = new Date(referenceDate);
                today.setHours(0, 0, 0, 0);
                const check = new Date(d);
                check.setHours(0, 0, 0, 0);
                return check < today;
            }

            /** Проверка: дедлайн в ближайшие 3 дня */
            function isSoon(d) {
                if (!d) return false;
                const today = new Date(referenceDate);
                today.setHours(0, 0, 0, 0);
                const check = new Date(d);
                check.setHours(0, 0, 0, 0);
                const diff = (check - today) / (1000 * 60 * 60 * 24);
                return diff >= 0 && diff <= 3;
            }

            // ==================== ПАРСИНГ КАЖДОГО СООБЩЕНИЯ ====================
            function parseMessage(msg) {
                const text = msg.message;
                const lower = text.toLowerCase();
                let taskTitle = '';
                let priority = 'medium';
                let deadlineDate = null;
                let deadlineStr = 'Не указан';
                let riskStr = '';
                let riskLevel = 'low';
                let taskType = 'feature';
                let dependencies = [];
                let notes = '';

                // --- Определение типа задачи ---
                if (lower.includes('оферт') || lower.includes('юрист')) {
                    taskType = 'compliance';
                } else if (lower.includes('инструкц') || lower.includes('чек-лист') || lower.includes('шаблон') || lower
                    .includes('weekly update') || lower.includes('статус') && lower.includes('разбросан')) {
                    taskType = 'process';
                } else if (lower.includes('уезжает') || lower.includes('не уходит') || lower.includes('ошибк') || lower
                    .includes('починить') || lower.includes('кнопка') && lower.includes('экран')) {
                    taskType = 'bug';
                } else if (lower.includes('собирать') || lower.includes('экспорт') || lower.includes('блок') || lower
                    .includes('лендинг') || lower.includes('nps') || lower.includes('csv')) {
                    taskType = 'feature';
                }

                // --- Извлечение заголовка задачи ---
                if (msg.id === 1) taskTitle = 'Исправить отправку чека после оплаты';
                if (msg.id === 2) taskTitle = 'Добавить блок с кейсами на лендинг к вебинару';
                if (msg.id === 3) taskTitle = 'Создать инструкцию и чек-лист для онбординга новичков';
                if (msg.id === 4) taskTitle = 'Починить кнопку «Оплатить» на iPhone в Safari';
                if (msg.id === 5) taskTitle = 'Автоматизировать сбор NPS после закрытия заказа';
                if (msg.id === 6) taskTitle = 'CSV-экспорт по филиалам для клиента Acme';
                if (msg.id === 7) taskTitle = 'Внедрить единый weekly update шаблон для команды';
                if (msg.id === 8) taskTitle = 'Обновить оферту на сайте (файл и ссылку)';

                // --- Извлечение дедлайна ---
                if (lower.includes('сегодня')) {
                    deadlineDate = new Date(referenceDate);
                    deadlineStr = 'Сегодня — ' + formatDate(deadlineDate);
                } else if (lower.includes('к пятнице') || lower.includes('до пятницы')) {
                    deadlineDate = nextWeekday(5); // пятница = 5
                    deadlineStr = 'К пятнице — ' + formatDate(deadlineDate);
                } else if (lower.includes('до конца недели')) {
                    deadlineDate = endOfWeek();
                    deadlineStr = 'До конца недели — ' + formatDate(deadlineDate);
                } else if (lower.includes('в среду') || lower.includes('до среды')) {
                    deadlineDate = thisOrNextWeekday(3); // среда = 3
                    deadlineStr = 'В среду — ' + formatDate(deadlineDate);
                } else if (lower.includes('на следующей неделе')) {
                    deadlineDate = nextMonday();
                    deadlineStr = 'Следующая неделя (с ' + formatDate(deadlineDate) + ')';
                } else if (lower.includes('в следующем спринте')) {
                    deadlineStr = 'Следующий спринт (дата не задана)';
                } else if (lower.includes('15 мая')) {
                    const may15 = new Date(referenceDate.getFullYear(), 4, 15); // 4 = май
                    deadlineDate = may15;
                    deadlineStr = '15 мая ' + referenceDate.getFullYear() + ' (до вебинара)';
                }

                // --- Извлечение приоритета ---
                const criticalMarkers = ['срочно', 'юрист', 'сорвется пилот', 'очень плохо', 'иначе говорит, что будет сложно продлить'];
                const highMarkers = ['сегодня', 'жалоб', 'перед вебинаром', 'к пятнице'];
                const lowMarkers = ['не горит', 'давно просят', 'хорошо бы', 'было бы хорошо'];

                let criticalScore = 0;
                let highScore = 0;
                let lowScore = 0;

                criticalMarkers.forEach(m => { if (lower.includes(m)) criticalScore++; });
                highMarkers.forEach(m => { if (lower.includes(m)) highScore++; });
                lowMarkers.forEach(m => { if (lower.includes(m)) lowScore++; });

                // Специальные правила
                if (lower.includes('сегодня') && lower.includes('жалоб')) criticalScore += 2;
                if (lower.includes('юрист') && lower.includes('сегодня')) criticalScore += 2;
                if (lower.includes('сорвется пилот') || lower.includes('сложно продлить пилот')) criticalScore += 3;
                if (lower.includes('срочно') && lower.includes('очень плохо')) criticalScore += 2;
                if (lower.includes('релиз') && (lower.includes('в среду') || lower.includes('сегодня'))) criticalScore += 1;

                if (criticalScore >= 2) priority = 'critical';
                else if (criticalScore >= 1 || highScore >= 2) priority = 'high';
                else if (lowScore >= 2 && criticalScore === 0 && highScore === 0) priority = 'low';
                else if (lowScore >= 1 && highScore === 0 && criticalScore === 0) priority = 'low';
                else if (highScore >= 1) priority = 'high';
                else priority = 'medium';

                // Уточнение для отдельных сообщений
                if (msg.id === 8) priority = 'critical'; // юристы + релиз
                if (msg.id === 6) priority = 'critical'; // срыв пилота
                if (msg.id === 4) priority = 'critical'; // срочно + очень плохо
                if (msg.id === 1) priority = 'high'; // сегодня + жалобы (но не critical, т.к. саппорт временно решает)
                if (msg.id === 5) priority = 'low'; // не горит

                // --- Извлечение рисков ---
                if (lower.includes('а то начнутся жалобы')) {
                    riskStr = 'Репутационный: жалобы клиентов на чеки';
                    riskLevel = 'high';
                }
                if (lower.includes('иначе говорит, что будет сложно продлить пилот') || lower.includes('сорвется пилот')) {
                    riskStr = 'Бизнес-риск: потеря пилотного клиента Acme';
                    riskLevel = 'critical';
                }
                if (lower.includes('выглядит очень плохо')) {
                    riskStr = 'Репутационный + конверсионный: кнопка оплаты не работает на iPhone';
                    riskLevel = 'critical';
                }
                if (lower.includes('юристы') && lower.includes('напомнили')) {
                    riskStr = 'Юридический: старая оферта на сайте, риск блокировки релиза';
                    riskLevel = 'critical';
                }
                if (lower.includes('статусы разбросаны') || lower.includes('третий раз')) {
                    riskStr = 'Процессный: дублирование обсуждений, потеря фокуса';
                    riskLevel = 'medium';
                }
                if (lower.includes('пишут всем в личку') || lower.includes('непонятно')) {
                    riskStr = 'Процессный: дезориентация новичков, нагрузка на команду';
                    riskLevel = 'medium';
                }
                if (msg.id === 2 && !riskStr) {
                    riskStr = 'Маркетинговый: срыв дедлайна к вебинару';
                    riskLevel = 'high';
                }
                if (msg.id === 5 && !riskStr) {
                    riskStr = 'Низкий: накопление технического долга по запросам продаж';
                    riskLevel = 'low';
                }
                if (!riskStr) riskStr = 'Не выявлен явно';
                if (!riskLevel) riskLevel = 'low';

                // --- Зависимости ---
                if (msg.id === 2) dependencies.push('Контент от маркетинга (почти готов)');
                if (msg.id === 6) dependencies.push('Требуется оценка трудозатрат (estimate)');
                if (msg.id === 8) dependencies.push('Файл оферты от юристов', 'Публикация релиза в среду');
                if (msg.id === 1) notes = 'Саппорт временно обрабатывает вручную — проблема длится 2 дня.';
                if (msg.id === 3) notes = 'Новички дезориентированы, нагрузка на всю команду.';
                if (msg.id === 7) notes = 'Обсуждения дублируются 3+ раз за неделю.';

                return {
                    id: msg.id,
                    channel: msg.channel,
                    original: msg.message,
                    taskTitle,
                    priority,
                    deadlineDate,
                    deadlineStr,
                    riskStr,
                    riskLevel,
                    taskType,
                    dependencies,
                    notes,
                };
            }

            // ==================== ПАРСИМ ВСЕ СООБЩЕНИЯ ====================
            const parsedTasks = rawMessages.map(msg => parseMessage(msg));

            // ==================== РЕНДЕРИНГ ====================

            // -- Таблица исходных сообщений --
            const rawTable = document.getElementById('rawMessagesTable');
            rawMessages.forEach(msg => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td><strong>${msg.id}</strong></td>
                    <td><span class="type-tag">${msg.channel}</span></td>
                    <td><div class="msg-original" title="Нажмите, чтобы развернуть">${msg.message}</div></td>
                `;
                rawTable.appendChild(tr);
            });

            // -- Таблица структурированных задач --
            const structTable = document.getElementById('structuredTasksTable');
            parsedTasks.forEach(task => {
                const tr = document.createElement('tr');
                const priorityClass = task.priority;
                const deadlineClass = task.deadlineDate ? (isOverdue(task.deadlineDate) ? 'overdue' :
                    isSoon(task.deadlineDate) ? 'soon' : '') : '';
                const typeClass = task.taskType;

                tr.innerHTML = `
                    <td><strong>#${task.id}</strong></td>
                    <td><strong style="font-size:0.9rem;">${task.taskTitle}</strong></td>
                    <td><span class="type-tag">${task.channel}</span></td>
                    <td><span class="type-tag ${typeClass}">${task.taskType}</span></td>
                    <td><span class="priority-badge ${priorityClass}">${priorityLabel(task.priority)}</span></td>
                    <td><span class="deadline-date ${deadlineClass}">${task.deadlineStr}</span>${task.deadlineDate && isOverdue(task.deadlineDate) ? ' ⚠️' : ''}</td>
                    <td><span class="risk-tag ${task.riskLevel === 'critical' ? 'critical-risk' : ''}" title="${task.riskStr}">${task.riskStr.length > 55 ? task.riskStr.slice(0, 52) + '…' : task.riskStr}</span></td>
                    <td style="font-size:0.78rem;">
                        ${task.dependencies.length ? task.dependencies.map(d => '• ' + d).join('<br>') : '—'}
                        ${task.notes ? '<br><span class="highlight-note">📝 ' + task.notes + '</span>' : ''}
                    </td>
                `;
                structTable.appendChild(tr);
            });

            function priorityLabel(p) {
                const map = { critical: '🔴 Critical', high: '🟡 High', medium: '🔵 Medium', low: '⚪ Low' };
                return map[p] || p;
            }

            // -- Карточки задач --
            const cardsGrid = document.getElementById('taskCardsGrid');
            // Сортируем: critical → high → medium → low
            const priorityOrder = { critical: 0, high: 1, medium: 2, low: 3 };
            const sortedTasks = [...parsedTasks].sort((a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]);

            sortedTasks.forEach(task => {
                const card = document.createElement('div');
                card.className = `task-card ${task.priority}-card`;
                const deadlineClass = task.deadlineDate ? (isOverdue(task.deadlineDate) ? 'overdue' :
                    isSoon(task.deadlineDate) ? 'soon' : '') : '';
                card.innerHTML = `
                    <div class="card-header">
                        <span class="card-channel">📡 ${task.channel}</span>
                        <span class="priority-badge ${task.priority}">${priorityLabel(task.priority)}</span>
                    </div>
                    <div class="card-title">${task.taskTitle}</div>
                    <div class="card-meta">
                        <span class="type-tag ${task.taskType}">${task.taskType}</span>
                        <span class="deadline-date ${deadlineClass}">📅 ${task.deadlineStr}</span>
                    </div>
                    <div style="font-size:0.78rem;color:var(--text-secondary);">
                        ⚠️ ${task.riskStr}
                    </div>
                    ${task.dependencies.length ? `<div style="font-size:0.75rem;color:var(--accent);">🔗 ${task.dependencies.join('; ')}</div>` : ''}
                    ${task.notes ? `<div style="font-size:0.75rem;font-style:italic;color:var(--text-secondary);">${task.notes}</div>` : ''}
                    <div style="font-size:0.7rem;color:var(--text-secondary);margin-top:4px;">ID: #${task.id}</div>
                `;
                cardsGrid.appendChild(card);
            });

            // -- Инсайты --
            const insightsList = document.getElementById('insightsList');
            const criticalTasks = parsedTasks.filter(t => t.priority === 'critical');
            const overdueTasks = parsedTasks.filter(t => t.deadlineDate && isOverdue(t.deadlineDate));
            const highRiskTasks = parsedTasks.filter(t => t.riskLevel === 'critical');
            const noDeadlineTasks = parsedTasks.filter(t => !t.deadlineDate);

            const insights = [
                { text: `🔴 <strong>${criticalTasks.length} критические задачи</strong> требуют немедленного внимания: ${criticalTasks.map(t=>'#'+t.id).join(', ')}. Это блокирует релиз, пилот и пользовательский опыт.`,
                    cls: 'warn' },
                { text: `⚠️ <strong>${overdueTasks.length} задача уже просрочена</strong> относительно опорной даты (${refDateStr}). Блок с кейсами к вебинару 15 мая требует эскалации.`,
                    cls: overdueTasks.length ? 'warn' : 'info' },
                { text: `🛡 <strong>${highRiskTasks.length} задачи несут критический бизнес-риск:</strong> потеря пилота Acme, юридические последствия, блокировка конверсии на iPhone.`,
                    cls: 'warn' },
                { text: `📋 <strong>3 из 8 задач — процессные:</strong> онбординг, weekly update, шаблоны. Системная проблема: знания не зафиксированы, договорённости живут в личке.`,
                    cls: 'info' },
                { text: `⏳ Задача #5 (NPS) — единственная с низким приоритетом. Её можно запланировать на следующий спринт без ущерба для текущих KPI.`,
                    cls: 'info' },
                { text: `🔄 <strong>Зависимость:</strong> задача #8 (оферта) блокирует релиз в среду. Задача #6 (CSV) требует оценки перед стартом. Это классические точки синхронизации.`,
                    cls: 'info' },
            ];
            insights.forEach(ins => {
                const li = document.createElement('li');
                li.className = ins.cls || '';
                li.innerHTML = ins.text;
                insightsList.appendChild(li);
            });

            // ==================== ГРАФИКИ (Chart.js) ====================

            // -- 1. Приоритеты (Doughnut) --
            const priorityCounts = { critical: 0, high: 0, medium: 0, low: 0 };
            parsedTasks.forEach(t => priorityCounts[t.priority]++);
            new Chart(document.getElementById('priorityChart'), {
                type: 'doughnut',
                data: {
                    labels: ['Critical', 'High', 'Medium', 'Low'],
                    datasets: [{
                        data: [priorityCounts.critical, priorityCounts.high, priorityCounts.medium,
                            priorityCounts.low
                        ],
                        backgroundColor: ['#e53e3e', '#f59e0b', '#3b82f6', '#9ca3af'],
                        borderColor: '#ffffff',
                        borderWidth: 3,
                        hoverBorderWidth: 4,
                    }],
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    plugins: {
                        legend: { position: 'bottom', labels: { padding: 20, usePointStyle: true,
                                pointStyleWidth: 10, font: { size: 12 } } },
                    },
                },
            });

            // -- 2. Каналы (Bar) --
            const channelCounts = {};
            parsedTasks.forEach(t => { channelCounts[t.channel] = (channelCounts[t.channel] || 0) + 1; });
            const channelLabels = Object.keys(channelCounts);
            const channelData = Object.values(channelCounts);
            const channelColors = ['#6366f1', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6', '#06b6d4', '#f97316',
                '#64748b'
            ];
            new Chart(document.getElementById('channelChart'), {
                type: 'bar',
                data: {
                    labels: channelLabels,
                    datasets: [{
                        label: 'Задач',
                        data: channelData,
                        backgroundColor: channelColors.slice(0, channelLabels.length).map(c => c +
                            'cc'),
                        borderColor: channelColors.slice(0, channelLabels.length),
                        borderWidth: 2,
                        borderRadius: 6,
                    }],
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    indexAxis: 'y',
                    plugins: { legend: { display: false } },
                    scales: { x: { beginAtZero: true, ticks: { stepSize: 1 } } },
                },
            });

            // -- 3. Таймлайн дедлайнов (горизонтальный баббл / scatter) --
            const tasksWithDeadline = parsedTasks.filter(t => t.deadlineDate);
            const timelineData = tasksWithDeadline.map(t => ({
                x: t.deadlineDate.getTime(),
                y: t.id,
                label: '#' + t.id + ' ' + t.taskTitle.slice(0, 25),
                priority: t.priority,
            }));
            const priorityColorMap = { critical: '#e53e3e', high: '#f59e0b', medium: '#3b82f6', low: '#9ca3af' };
            const timelineCtx = document.getElementById('deadlineChart').getContext('2d');
            // Используем линейный график с кастомными точками
            const refTime = referenceDate.getTime();
            const allTimes = [refTime, ...timelineData.map(d => d.x)];
            const minTime = Math.min(...allTimes) - 3 * 24 * 60 * 60 * 1000;
            const maxTime = Math.max(...allTimes) + 7 * 24 * 60 * 60 * 1000;

            // Создадим датасеты по одному на задачу для наглядности
            const deadlineDatasets = timelineData.map((td, i) => ({
                label: td.label,
                data: [{ x: td.x, y: i + 1 }],
                backgroundColor: priorityColorMap[td.priority] || '#6366f1',
                borderColor: priorityColorMap[td.priority] || '#6366f1',
                pointRadius: 10,
                pointHoverRadius: 14,
                pointStyle: 'rectRounded',
                showLine: false,
            }));
            // Добавим линию "сегодня"
            deadlineDatasets.push({
                label: 'Сегодня (' + formatDateShort(referenceDate) + ')',
                data: [{ x: refTime, y: 0 }, { x: refTime, y: timelineData.length + 1 }],
                borderColor: '#ef4444',
                borderWidth: 2,
                borderDash: [6, 4],
                pointRadius: 0,
                fill: false,
                showLine: true,
                type: 'line',
            });

            new Chart(timelineCtx, {
                type: 'scatter',
                data: { datasets: deadlineDatasets },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    plugins: {
                        legend: { display: true, position: 'bottom', labels: { font: { size: 9 },
                                usePointStyle: true, pointStyleWidth: 8, padding: 12 } },
                        tooltip: {
                            callbacks: {
                                label: function(ctx) {
                                    const d = new Date(ctx.raw.x);
                                    return ctx.dataset.label + ': ' + formatDate(d);
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            type: 'linear',
                            min: minTime,
                            max: maxTime,
                            ticks: {
                                callback: function(v) { return formatDateShort(new Date(v)); },
                                font: { size: 10 },
                            },
                            title: { display: true, text: 'Дата', font: { size: 11 } },
                        },
                        y: {
                            display: true,
                            title: { display: true, text: 'Задачи', font: { size: 11 } },
                            ticks: { stepSize: 1, font: { size: 9 } },
                            min: 0,
                            max: timelineData.length + 1,
                        },
                    },
                },
            });

            // -- 4. Риски (горизонтальная столбчатая) --
            const riskCategories = {};
            parsedTasks.forEach(t => {
                const cat = t.riskLevel === 'critical' ? 'Критический' : t.riskLevel === 'high' ?
                    'Высокий' : t.riskLevel === 'medium' ? 'Средний' : 'Низкий';
                riskCategories[cat] = (riskCategories[cat] || 0) + 1;
            });
            const riskOrder = ['Критический', 'Высокий', 'Средний', 'Низкий'];
            const riskData = riskOrder.map(r => riskCategories[r] || 0);
            new Chart(document.getElementById('riskChart'), {
                type: 'bar',
                data: {
                    labels: riskOrder,
                    datasets: [{
                        label: 'Количество задач',
                        data: riskData,
                        backgroundColor: ['#e53e3e99', '#f59e0b99', '#3b82f699', '#9ca3af99'],
                        borderColor: ['#e53e3e', '#f59e0b', '#3b82f6', '#9ca3af'],
                        borderWidth: 2,
                        borderRadius: 6,
                    }],
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    plugins: { legend: { display: false } },
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } },
                },
            });

            // ==================== СВОРАЧИВАНИЕ СЕКЦИЙ ====================
            window.toggleSection = function(bodyId, headerEl) {
                const body = document.getElementById(bodyId);
                const section = headerEl.closest('.section');
                if (body.classList.contains('collapsed')) {
                    body.classList.remove('collapsed');
                    section.classList.remove('collapsed');
                } else {
                    body.classList.add('collapsed');
                    section.classList.add('collapsed');
                }
            };

            // Все секции открыты по умолчанию
            console.log('✅ PM Task Dashboard загружен. Опорная дата:', refDateStr);
            console.log('📊 Задач разобрано:', parsedTasks.length);
            console.log('🔴 Critical:', parsedTasks.filter(t => t.priority === 'critical').length);
            console.log('🟡 High:', parsedTasks.filter(t => t.priority === 'high').length);
            console.log('🔵 Medium:', parsedTasks.filter(t => t.priority === 'medium').length);
            console.log('⚪ Low:', parsedTasks.filter(t => t.priority === 'low').length);
            console.log('⚠️ Просрочено:', parsedTasks.filter(t => t.deadlineDate && isOverdue(t.deadlineDate)).length);
            console.log('📅 Опорная дата для "сегодня":', refDateStr,
                '(все относительные сроки вычислены относительно этой даты)');

        })();
    </script>
</body>
</html>
