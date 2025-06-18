FluxLang: Сценарии Использования
Этот документ описывает различные сценарии использования для FluxLang, демонстрируя, как этот язык, исполняемый AI-моделями, может быть применен для концептуального моделирования и обработки данных в сложных системах.

1. Моделирование и Симуляция Потоков Данных (ETL-процессы)
FluxLang идеально подходит для описания декларативных конвейеров обработки данных, имитируя Extract, Transform, Load (ETL) процессы.

Пример: Симуляция обработки логов веб-сервера.

Задачи: Чтение потока концептуальных логов, фильтрация по типу события, парсинг данных, агрегация метрик (например, количество запросов в секунду), "запись" обработанных данных в концептуальное хранилище.

Ключевые возможности FluxLang: stream, map, filter, reduce, concept_read_file, concept_write_file.

// Модуль для парсинга логов
module LogParser {
    export record LogEntry {
        timestamp: String,
        level: String,
        message: String,
        ip: String?
    }

    export func parse_log_line(line: String) {
        // Концептуальная симуляция парсинга строки лога в Record
        // В реальной реализации AI здесь будет использоваться его способность к парсингу текста
        if len(line) == 0 then { none() } else {
            some(LogEntry {
                timestamp: "2025-01-01T12:00:00Z", // Симуляция
                level: "INFO",                   // Симуляция
                message: line,                   // Вся строка как сообщение
                ip: some("192.168.1.1")          // Симуляция
            })
        }
    }
}

import LogParser.parse_log_line

// Симуляция чтения лог-файла
let raw_log_stream = concept_read_file("access.log")
    -> flat_map_future((result) => {
        match result {
            case ok(content) => Future.ready(ok(stream from_list(split(content, "\n")))),
            case error(msg) => Future.ready(error(msg))
        }
    })
    -> flat_map_future((result_stream) => { // Разворачиваем Result<Stream> до Stream
        match result_stream {
            case ok(s) => s,
            case error(msg) => { log("Error reading log file:", msg); stream empty() }
        }
    })


// Конвейер обработки логов
raw_log_stream
    -> map(parse_log_line) // Парсинг каждой строки
    -> filter(is_some)     // Отфильтровываем неудачные попытки парсинга
    -> map((option_log) => option_log?) // Извлекаем LogEntry из Option
    -> filter((entry) => entry.level == "INFO") // Фильтруем по уровню
    -> peek((entry) => log("Processed INFO entry:", entry.message)) // Логируем
    -> collect() // Собираем в список
    -> log("Total INFO entries processed:")

2. Моделирование Сложных Асинхронных Взаимодействий
FluxLang позволяет описывать и симулировать неблокирующие операции и зависимости между ними.

Пример: Моделирование получения данных из нескольких концептуальных API и их объединение.

Задачи: Параллельный запрос данных о пользователе и его заказах, ожидание обоих ответов, объединение данных.

Ключевые возможности FluxLang: async, await, Future, concept_http_get, merge.

// Концептуальная функция для получения данных пользователя
func get_user_data(user_id: Number) {
    async {
        log("Simulating API call for user:", user_id)
        let url = "https://api.example.com/users/" + to_string(user_id)
        let response = (await concept_http_get(url))? // Ожидаем Future<Result<String, String>> и распаковываем
        log("Received user data response for user:", user_id)
        // Концептуальный парсинг JSON
        ok({ "id": user_id, "name": "User_" + to_string(user_id), "email": "user" + to_string(user_id) + "@example.com" })
    }
}

// Концептуальная функция для получения данных о заказах пользователя
func get_user_orders(user_id: Number) {
    async {
        log("Simulating API call for orders of user:", user_id)
        let url = "https://api.example.com/orders?userId=" + to_string(user_id)
        let response = (await concept_http_get(url))? // Ожидаем Future<Result<String, String>> и распаковываем
        log("Received orders data response for user:", user_id)
        // Концептуальный парсинг JSON
        ok([{"order_id": 100 + user_id, "amount": user_id * 10}, {"order_id": 200 + user_id, "amount": user_id * 5}])
    }
}

// Основной процесс
async {
    let user_id = 42

    // Концептуально запускаем два запроса параллельно
    let future_user = get_user_data(user_id)
    let future_orders = get_user_orders(user_id)

    // Ожидаем результаты обоих запросов
    let user_result = await future_user // Future<Result<Map, String>>
    let orders_result = await future_orders // Future<Result<List, String>>

    // Обрабатываем результаты
    match (user_result, orders_result) {
        case (ok(user_data), ok(orders_data)) => {
            let combined_data = merge(user_data, {"orders": orders_data})
            log("Successfully combined data:", combined_data)
            ok(combined_data)
        },
        case (error(user_msg), _) => {
            log("Failed to fetch user data:", user_msg)
            error("User data fetch failed")
        },
        case (_, error(orders_msg)) => {
            log("Failed to fetch orders data:", orders_msg)
            error("Orders data fetch failed")
        }
    }
} -> log("Async Operation Result:")

3. Интеграция с Базой Знаний AI (AI Oracle)
FluxLang предоставляет прямой способ для кода запрашивать информацию из внутренней базы знаний исполняющей AI-модели.

Пример: Динамическое принятие решений на основе "реальных" данных, доступных AI.

Задачи: Спросить AI о текущей погоде в городе, получить структурированный ответ, использовать его в логике.

Ключевые возможности FluxLang: ask, Future, match.

record WeatherInfo {
    city: String,
    temperature: Number,
    conditions: String
}

async {
    let city = "Tashkent"
    let question = `What is the current weather in ${city}? Provide temperature in Celsius and general conditions. Format as JSON with fields "city", "temperature", "conditions".`
    let future_weather_data_raw = ask(question, {"format": "json"})

    let weather_data_raw = (await future_weather_data_raw)? // Result<String, String>

    // Концептуальный парсинг JSON
    let weather_map_result = parse_json_to_map(weather_data_raw)
    let weather_map = weather_map_result?

    // Создание Record из полученных данных
    let weather = WeatherInfo {
        city: get_map_item_or_none(weather_map, "city")?,
        temperature: get_map_item_or_none(weather_map, "temperature")?,
        conditions: get_map_item_or_none(weather_map, "conditions")?
    }

    log("Weather in", weather.city, ":", weather.temperature, "°C,", weather.conditions)

    if weather.temperature < 10 then {
        log("It's cold!")
    } else if weather.temperature > 25 then {
        log("It's warm!")
    } else {
        log("The weather is mild.")
    }
} -> log("Weather check completed.")

(Примечание: parse_json_to_map — это концептуальная функция, которая предполагает, что AI может выполнить симуляцию парсинга JSON из строки в Map.)

4. Моделирование Графов Знаний (Knowledge Graphs)
FluxLang позволяет определять, заполнять и запрашивать концептуальные графы знаний, управляемые AI-моделью.

Пример: Построение простого графа отношений между сущностями и выполнение запросов.

Задачи: Определение городов и стран, отношений "находится в", запрос всех городов в определенной стране.

Ключевые возможности FluxLang: entity, relation, concept_add_entity, concept_add_relation, query_kg.

// Определяем типы сущностей
entity City { name: String, population: Number }
entity Country { name: String, capital: String }

// Определяем отношение
relation is_located_in(City, Country)

async {
    // Добавляем сущности
    let city_tashkent = concept_add_entity("City", {"name": "Tashkent", "population": 3000000})?
    let city_london = concept_add_entity("City", {"name": "London", "population": 9000000})?
    let country_uz = concept_add_entity("Country", {"name": "Uzbekistan", "capital": "Tashkent"})?
    let country_uk = concept_add_entity("Country", {"name": "United Kingdom", "capital": "London"})?

    // Добавляем отношения (здесь используются концептуальные ID, возвращаемые concept_add_entity)
    concept_add_relation("is_located_in", city_tashkent.id, country_uz.id)?
    concept_add_relation("is_located_in", city_london.id, country_uk.id)?

    log("Conceptual Knowledge Graph populated.")

    // Запрашиваем все города в Узбекистане
    let query_pattern = {
        "entity_type": "City",
        "relations": [
            {"relation_name": "is_located_in", "target_entity": {"entity_type": "Country", "name": "Uzbekistan"}}
        ]
    }

    let result_cities = (await query_kg(query_pattern))?
    log("Cities in Uzbekistan:", result_cities)

    // Запрашиваем столицу Великобритании
    let capital_query_pattern = {
        "entity_type": "Country",
        "name": "United Kingdom",
        "return_fields": ["capital"] // Запрашиваем конкретное поле
    }
    let uk_capital = (await query_kg(capital_query_pattern))?
    log("Capital of UK:", uk_capital)
} -> log("Knowledge Graph interaction completed.")

5. Моделирование ML-Конвейеров
FluxLang позволяет концептуально описывать процессы обучения и инференса (предсказания) машинного обучения, используя внутренние возможности AI-модели.

Пример: Симуляция обучения простой модели классификации и ее использования для предсказания.

Задачи: "Обучение" модели на концептуальных данных, "предсказание" для новых данных.

Ключевые возможности FluxLang: ml.train, ml.predict, Model, Prediction.

record FeatureSet {
    feature1: Number,
    feature2: Number,
    label: String
}
record PredictionResult {
    input_id: Number,
    predicted_label: String,
    confidence: Number
}

async {
    // Симуляция обучающих данных
    let training_data = stream from_list([
        FeatureSet { feature1: 0.1, feature2: 0.2, label: "A" },
        FeatureSet { feature1: 0.8, feature2: 0.9, label: "B" },
        FeatureSet { feature1: 0.2, feature2: 0.3, label: "A" },
        FeatureSet { feature1: 0.7, feature2: 0.6, label: "B" }
    ])

    // Концептуальное обучение модели
    let model_config = { "model_type": "binary_classifier", "iterations": 100 }
    let trained_model = (await ml.train(training_data, model_config))?
    log("Conceptual model trained.")

    // Симуляция новых данных для предсказания
    let inference_data = stream from_list([
        FeatureSet { feature1: 0.15, feature2: 0.25, label: "unknown", input_id: 1 },
        FeatureSet { feature1: 0.9, feature2: 0.8, label: "unknown", input_id: 2 }
    ])

    // Концептуальное предсказание
    let predictions_stream = (await ml.predict(trained_model, inference_data))?
    let predictions = predictions_stream -> collect()
    log("Conceptual predictions generated:", predictions)

    predictions
        -> map((pred) => PredictionResult {
            input_id: pred.input_id,
            predicted_label: pred.label,
            confidence: pred.confidence
        })
        -> log("Formatted Predictions:")
} -> log("ML Pipeline simulation completed.")

6. Концептуальное Тестирование и Валидация
FluxLang можно использовать для описания сценариев тестирования и валидации логики.

Пример: Проверка, что функция возвращает ожидаемый результат.

Задачи: Определение тестовых случаев, выполнение функции с тестовыми данными, сравнение фактического и ожидаемого результатов.

Ключевые возможности FluxLang: func, if/then/else, log, сравнения.

func multiply_by_two(x: Number) { x * 2 }

func run_test(test_name: String, func_to_test, input_value, expected_output) {
    let actual_output = func_to_test(input_value)
    if actual_output == expected_output then {
        log("Test PASSED:", test_name)
        ok(void())
    } else {
        log("Test FAILED:", test_name, "Input:", input_value, "Expected:", expected_output, "Actual:", actual_output)
        error("Test Failed: " + test_name)
    }
}

run_test("Multiply by two positive", multiply_by_two, 5, 10) -> log()
run_test("Multiply by two negative", multiply_by_two, -3, -6) -> log()
run_test("Multiply by two zero", multiply_by_two, 0, 0) -> log()
run_test("Multiply by two decimal", multiply_by_two, 2.5, 5.0) -> log()
run_test("Multiply by two with incorrect expectation", multiply_by_two, 4, 9) -> log() // Ожидаем, что этот тест провалится

7. Моделирование Пользовательских Интерфейсов и Событий (Концептуально)
Хотя FluxLang не является языком для UI, его ADT и потоки могут моделировать события UI.

Пример: Обработка последовательности концептуальных событий веб-страницы.

Задачи: Определение Enum для событий, фильтрация по типу события, агрегация информации о кликах.

Ключевые возможности FluxLang: enum, match, stream, filter, reduce.

// Enum WebEvent уже определен в спецификации
// enum WebEvent { PageLoad, Click({ x: Number, y: Number }), KeyPress(String) }

let web_events = stream from_list([
    WebEvent.PageLoad,
    WebEvent.Click({ x: 10, y: 20 }),
    WebEvent.KeyPress("A"),
    WebEvent.Click({ x: 50, y: 60 }),
    WebEvent.KeyPress("Enter")
])

// Отфильтровываем только события кликов
web_events
    -> filter((event) => match event { case WebEvent.Click(_) => true, case _ => false })
    -> map((click_event) => match click_event { // Извлекаем данные клика
        case WebEvent.Click({ x, y }) => {x: x, y: y},
        case _ => {x: 0, y: 0} // Никогда не произойдет благодаря фильтру
    })
    -> reduce({total_clicks: 0, last_x: 0, last_y: 0}, (acc, click_data) => {
        { total_clicks: acc.total_clicks + 1, last_x: click_data.x, last_y: click_data.y }
    })
    -> log("Aggregated Click Data:")

8. Моделирование Распределенных Систем и Консенсуса (Концептуально)
FluxLang может быть использован для высокоуровневого моделирования взаимодействий в распределенных системах.

Пример: Симуляция процесса голосования или консенсуса.

Задачи: Моделирование участников, их голосов, сбор голосов и определение консенсуса.

Ключевые возможности FluxLang: async/await, List, reduce, Map.

record Vote {
    participant_id: String,
    decision: Boolean
}

// Симуляция голосования участника
func cast_vote_async(participant_id: String) {
    async {
        log("Participant", participant_id, "is casting vote...")
        // Концептуальная задержка или "сетевой" вызов
        let is_agree = if parse_int(participant_id)? % 2 == 0 then { true } else { false } // Симуляция решения
        log("Participant", participant_id, "voted:", is_agree)
        ok(Vote { participant_id: participant_id, decision: is_agree })
    }
}

async {
    let participants = ["P1", "P2", "P3", "P4", "P5"]

    // Каждый участник голосует асинхронно
    let future_votes = participants
        -> map_future(cast_vote_async) // Создаем Future для каждого голоса

    // Ждем, пока все голоса будут поданы
    let all_votes_results = await_all_futures_with_results(future_votes)? // Концептуальная функция для await всех Future<Result>
    log("All votes cast and collected:", all_votes_results)

    // Фильтруем успешные голоса
    let successful_votes = all_votes_results
        -> filter(is_ok)
        -> map((result) => result?)

    // Агрегируем результаты
    let vote_counts = successful_votes
        -> reduce({"true_count": 0, "false_count": 0}, (acc, vote) => {
            if vote.decision then {
                { "true_count": acc.true_count + 1, "false_count": acc.false_count }
            } else {
                { "true_count": acc.true_count, "false_count": acc.false_count + 1 }
            }
        })

    log("Vote Counts:", vote_counts)

    if vote_counts.true_count > vote_counts.false_count then {
        log("Consensus Reached: Majority AGREED!")
    } else if vote_counts.false_count > vote_counts.true_count then {
        log("Consensus Reached: Majority DISAGREED!")
    } else {
        log("No Clear Consensus: Votes are tied.")
    }
} -> log("Consensus simulation completed.")

Эти сценарии показывают, что FluxLang, несмотря на свою концептуальную природу, является мощным и выразительным инструментом для моделирования широкого круга систем и задач, особенно тех, где требуется взаимодействие с внутренними возможностями AI-модели.
