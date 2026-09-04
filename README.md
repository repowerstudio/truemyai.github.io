<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Image Generator Chat</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
            position: relative;
            overflow: hidden;
            background: linear-gradient(180deg, 
                #0c1445 0%,
                #1a237e 20%,
                #283593 35%,
                #4a148c 50%,
                #6a1b9a 60%,
                #8e24aa 70%,
                #d81b60 80%,
                #ff6f00 90%,
                #ffab00 100%
            );
        }

        .stars {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 60%;
            pointer-events: none;
            z-index: 0;
        }

        .star {
            position: absolute;
            background: white;
            border-radius: 50%;
            animation: twinkle 3s infinite;
        }

        @keyframes twinkle {
            0%, 100% {
                opacity: 0.3;
                transform: scale(1);
            }
            50% {
                opacity: 1;
                transform: scale(1.3);
            }
        }

        .chat-container {
            width: 100%;
            max-width: 900px;
            height: 90vh;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            position: relative;
            z-index: 1;
            backdrop-filter: blur(10px);
        }

        .chat-header {
            background: linear-gradient(135deg, 
                rgba(26, 35, 126, 0.9) 0%,
                rgba(74, 20, 140, 0.8) 30%,
                rgba(216, 27, 96, 0.7) 70%,
                rgba(255, 111, 0, 0.6) 100%
            );
            color: white;
            padding: 20px;
            text-align: center;
            position: relative;
            border-bottom: 1px solid rgba(255, 255, 255, 0.2);
        }

        .chat-header h1 {
            font-size: 24px;
            margin-bottom: 5px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .chat-header p {
            font-size: 14px;
            opacity: 0.9;
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
        }

        .quota-container {
            position: absolute;
            top: 10px;
            left: 10px;
            background: rgba(0, 0, 0, 0.3);
            padding: 8px 12px;
            border-radius: 20px;
            display: flex;
            align-items: center;
            gap: 8px;
            font-size: 14px;
            flex-wrap: wrap;
        }

        .quota-icon {
            font-size: 18px;
        }

        .quota-count {
            font-weight: bold;
            color: #ffd700;
        }

        .quota-used {
            font-weight: bold;
            color: #ff6b6b;
            margin-left: 5px;
        }

        .quota-bar {
            width: 100px;
            height: 6px;
            background: rgba(255, 255, 255, 0.3);
            border-radius: 3px;
            overflow: hidden;
        }

        .quota-bar-fill {
            height: 100%;
            background: linear-gradient(90deg, #4caf50, #ffd700);
            border-radius: 3px;
            transition: width 0.5s ease;
        }

        .settings-bar {
            background: linear-gradient(90deg, 
                rgba(26, 35, 126, 0.1),
                rgba(74, 20, 140, 0.1),
                rgba(216, 27, 96, 0.1)
            );
            padding: 10px 20px;
            border-bottom: 1px solid #e0e0e0;
            display: flex;
            align-items: center;
            gap: 15px;
            flex-wrap: wrap;
        }

        .settings-group {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .settings-label {
            font-size: 14px;
            color: #1a237e;
            font-weight: 600;
        }

        .settings-select {
            padding: 8px 12px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 14px;
            outline: none;
            cursor: pointer;
            background: white;
            transition: all 0.3s ease;
        }

        .settings-select:focus {
            border-color: #667eea;
            box-shadow: 0 0 10px rgba(102, 126, 234, 0.3);
        }

        .chat-messages {
            flex: 1;
            overflow-y: auto;
            padding: 20px;
            background: linear-gradient(180deg,
                rgba(255, 255, 255, 0.9) 0%,
                rgba(255, 248, 225, 0.9) 50%,
                rgba(255, 224, 178, 0.9) 100%
            );
        }

        .message {
            margin-bottom: 20px;
            display: flex;
            flex-direction: column;
        }

        .message.user {
            align-items: flex-end;
        }

        .message.ai {
            align-items: flex-start;
        }

        .message-content {
            max-width: 70%;
            padding: 12px 16px;
            border-radius: 18px;
            word-wrap: break-word;
            position: relative;
        }

        .user .message-content {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border-bottom-right-radius: 4px;
            box-shadow: 0 2px 5px rgba(102, 126, 234, 0.3);
        }

        .ai .message-content {
            background: white;
            color: #333;
            border: 1px solid #e0e0e0;
            border-bottom-left-radius: 4px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }

        .translated-prompt {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
            font-style: italic;
        }

        .ai-image {
            max-width: 300px;
            border-radius: 12px;
            margin-top: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
            display: block;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
        }

        .ai-image:hover {
            transform: scale(1.05);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
        }

        .image-info {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
        }

        .loading-spinner {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid #667eea;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-right: 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .progress-container {
            width: 100%;
            margin-top: 10px;
            background: #f0f0f0;
            border-radius: 10px;
            overflow: hidden;
            display: none;
        }

        .progress-bar {
            width: 0%;
            height: 8px;
            background: linear-gradient(90deg, #667eea, #764ba2, #d81b60);
            border-radius: 10px;
            transition: width 0.5s ease;
            animation: shimmer 2s infinite;
        }

        @keyframes shimmer {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        .progress-text {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
            text-align: center;
        }

        .chat-input-container {
            padding: 20px;
            background: linear-gradient(90deg,
                rgba(255, 255, 255, 0.95),
                rgba(255, 248, 225, 0.95)
            );
            border-top: 1px solid #e0e0e0;
        }

        .chat-input-wrapper {
            display: flex;
            gap: 10px;
        }

        .chat-input {
            flex: 1;
            padding: 12px 16px;
            border: 2px solid #e0e0e0;
            border-radius: 25px;
            font-size: 16px;
            outline: none;
            transition: all 0.3s ease;
            background: white;
        }

        .chat-input:focus {
            border-color: #667eea;
            box-shadow: 0 0 15px rgba(102, 126, 234, 0.2);
        }

        .send-button {
            padding: 12px 24px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.3s ease;
            white-space: nowrap;
            box-shadow: 0 4px 10px rgba(102, 126, 234, 0.3);
        }

        .send-button:hover:not(:disabled) {
            transform: scale(1.05);
            box-shadow: 0 6px 20px rgba(102, 126, 234, 0.5);
        }

        .send-button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .clear-button {
            position: absolute;
            top: 10px;
            right: 10px;
            background: rgba(255, 255, 255, 0.2);
            color: white;
            border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 5px 10px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 12px;
            transition: all 0.3s ease;
        }

        .clear-button:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: scale(1.05);
        }

        .image-modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.9);
            justify-content: center;
            align-items: center;
            z-index: 1000;
            cursor: pointer;
        }

        .image-modal.active {
            display: flex;
        }

        .image-modal img {
            max-width: 90%;
            max-height: 90%;
            object-fit: contain;
        }

        .quota-warning {
            color: #ff6b6b;
            animation: pulse 1s infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
    </style>
</head>
<body>
    <div class="stars" id="stars"></div>
    
    <div class="chat-container">
        <div class="chat-header">
            <button class="clear-button" onclick="clearChat()">Очистить</button>
            <h1>🎨 AI Генератор Изображений</h1>
            <p>Опишите изображение, которое хотите создать</p>
        </div>
        
        <div class="settings-bar">
            <div class="settings-group">
                <label class="settings-label">Разрешение:</label>
                <select class="settings-select" id="resolutionSelect">
                    <option value="512x512">512 × 512 (Быстро)</option>
                    <option value="768x768">768 × 768 (Средне)</option>
                    <option value="1024x1024" selected>1024 × 1024 (Высокое)</option>
                    <option value="1280x720">1280 × 720 (HD 16:9)</option>
                    <option value="1920x1080">1920 × 1080 (Full HD)</option>
                </select>
            </div>
            
            <div class="settings-group">
                <label class="settings-label">Стиль:</label>
                <select class="settings-select" id="styleSelect">
                    <option value="">Обычный</option>
                    <option value="photorealistic">Фотореалистичный</option>
                    <option value="anime">Аниме</option>
                    <option value="watercolor">Акварель</option>
                    <option value="oil-painting">Масляная живопись</option>
                    <option value="cyberpunk">Киберпанк</option>
                    <option value="fantasy">Фэнтези</option>
                </select>
            </div>
        </div>
        
        <div class="chat-messages" id="chatMessages">
            <div class="message ai">
                <div class="message-content">
                    Привет! Я AI генератор изображений. Опишите, какое изображение вы хотите создать, и я сгенерирую его для вас!
                </div>
            </div>
        </div>
        
        <div class="chat-input-container">
            <div class="chat-input-wrapper">
                <input 
                    type="text" 
                    class="chat-input" 
                    id="chatInput" 
                    placeholder="Опишите изображение на русском..."
                    onkeypress="handleKeyPress(event)"
                >
                <button class="send-button" id="sendButton" onclick="sendMessage()">
                    Отправить
                </button>
            </div>
        </div>
    </div>

    <div class="image-modal" id="imageModal" onclick="closeModal()">
        <img id="modalImage" src="" alt="Увеличенное изображение">
    </div>

    <script>
        // Создание звезд на фоне
        function createStars() {
            const starsContainer = document.getElementById('stars');
            const starCount = 100;
            
            for (let i = 0; i < starCount; i++) {
                const star = document.createElement('div');
                star.className = 'star';
                
                const x = Math.random() * 100;
                const y = Math.random() * 100;
                const size = Math.random() * 3 + 1;
                const delay = Math.random() * 3;
                
                star.style.left = x + '%';
                star.style.top = y + '%';
                star.style.width = size + 'px';
                star.style.height = size + 'px';
                star.style.animationDelay = delay + 's';
                
                starsContainer.appendChild(star);
            }
        }
        
        createStars();
        
        const API_URL = "https://truemyai.repowerstudio.workers.dev";
        const API_KEY = "cbc5a9ed9cd88913941f8d99241d62ec";
        
        let remainingQuota = 0;
        let totalQuota = 100000;
        let usedQuota = 0;
        
        const chatMessages = document.getElementById('chatMessages');
        const chatInput = document.getElementById('chatInput');
        const sendButton = document.getElementById('sendButton');
        const resolutionSelect = document.getElementById('resolutionSelect');
        const styleSelect = document.getElementById('styleSelect');
        const quotaCount = document.getElementById('quotaCount');
        const quotaUsed = document.getElementById('quotaUsed');
        const quotaBarFill = document.getElementById('quotaBarFill');
        const quotaContainer = document.getElementById('quotaContainer');

        // Функции для работы с cookies
        function setCookie(name, value, days = 30) {
            const expires = new Date();
            expires.setTime(expires.getTime() + days * 24 * 60 * 60 * 1000);
            document.cookie = `${name}=${encodeURIComponent(value)};expires=${expires.toUTCString()};path=/`;
        }

        function getCookie(name) {
            const nameEQ = name + "=";
            const ca = document.cookie.split(';');
            for (let i = 0; i < ca.length; i++) {
                let c = ca[i];
                while (c.charAt(0) === ' ') c = c.substring(1, c.length);
                if (c.indexOf(nameEQ) === 0) return decodeURIComponent(c.substring(nameEQ.length, c.length));
            }
            return null;
        }

        function deleteCookie(name) {
            document.cookie = `${name}=;expires=Thu, 01 Jan 1970 00:00:00 UTC;path=/`;
        }

        // Сохранение чата в cookies
        function saveChatToCookies() {
            const messages = [];
            const messageElements = chatMessages.querySelectorAll('.message');
            
            messageElements.forEach(msgEl => {
                const isUser = msgEl.classList.contains('user');
                const isAI = msgEl.classList.contains('ai');
                const content = msgEl.querySelector('.message-content');
                
                if (content) {
                    const text = content.textContent;
                    const image = content.querySelector('.ai-image');
                    
                    if (image) {
                        messages.push({
                            type: 'image',
                            text: text,
                            imageSrc: image.src,
                            isUser: false
                        });
                    } else {
                        messages.push({
                            type: 'text',
                            text: text,
                            isUser: isUser
                        });
                    }
                }
            });
            
            // Сохраняем только последние 50 сообщений
            const recentMessages = messages.slice(-50);
            setCookie('chatMessages', JSON.stringify(recentMessages), 7);
        }

        // Загрузка чата из cookies
        function loadChatFromCookies() {
            const saved = getCookie('chatMessages');
            
            if (saved) {
                try {
                    const messages = JSON.parse(saved);
                    
                    // Очищаем чат
                    chatMessages.innerHTML = '';
                    
                    // Добавляем сообщения
                    messages.forEach(msg => {
                        if (msg.type === 'image' && msg.imageSrc) {
                            const messageDiv = document.createElement('div');
                            messageDiv.className = 'message ai';
                            
                            const messageContent = document.createElement('div');
                            messageContent.className = 'message-content';
                            messageContent.textContent = msg.text;
                            
                            const image = document.createElement('img');
                            image.src = msg.imageSrc;
                            image.className = 'ai-image';
                            image.alt = 'Сгенерированное изображение';
                            image.onclick = function() {
                                openModal(msg.imageSrc);
                            };
                            
                            messageContent.appendChild(document.createElement('br'));
                            messageContent.appendChild(image);
                            messageDiv.appendChild(messageContent);
                            
                            chatMessages.appendChild(messageDiv);
                        } else {
                            const messageDiv = document.createElement('div');
                            messageDiv.className = `message ${msg.isUser ? 'user' : 'ai'}`;
                            
                            const messageContent = document.createElement('div');
                            messageContent.className = 'message-content';
                            messageContent.textContent = msg.text;
                            
                            messageDiv.appendChild(messageContent);
                            chatMessages.appendChild(messageDiv);
                        }
                    });
                    
                    scrollToBottom();
                    return true;
                } catch (error) {
                    console.error('Ошибка загрузки чата:', error);
                    return false;
                }
            }
            return false;
        }

        // Функция для получения квоты с сервера
        async function fetchQuota() {
            try {
                const response = await fetch(`${API_URL}/quota`, {
                    method: "GET",
                    headers: {
                        "Authorization": `Bearer ${API_KEY}`,
                    },
                });

                if (response.ok) {
                    const data = await response.json();
                    remainingQuota = data.remaining;
                    totalQuota = data.total;
                    usedQuota = data.used || (totalQuota - remainingQuota);
                    updateQuotaDisplay();
                    
                    // Сохраняем квоту в cookies
                    setCookie('remainingQuota', remainingQuota, 1);
                    setCookie('usedQuota', usedQuota, 1);
                    
                    if (data.resetTime) {
                        const resetDate = new Date(data.resetTime);
                        console.log('Квота сбросится:', resetDate.toLocaleString());
                    }
                }
            } catch (error) {
                console.error('Ошибка получения квоты:', error);
                
                // Пробуем загрузить из cookies
                const savedRemaining = getCookie('remainingQuota');
                const savedUsed = getCookie('usedQuota');
                
                if (savedRemaining) {
                    remainingQuota = parseInt(savedRemaining);
                    updateQuotaDisplay();
                } else {
                    quotaCount.textContent = 'Ошибка';
                }
                
                if (savedUsed) {
                    usedQuota = parseInt(savedUsed);
                    quotaUsed.textContent = usedQuota.toLocaleString();
                }
            }
        }

        // Обновление отображения квоты
        function updateQuotaDisplay() {
            quotaCount.textContent = remainingQuota.toLocaleString();
            quotaUsed.textContent = usedQuota.toLocaleString();
            
            const percentage = (remainingQuota / totalQuota) * 100;
            quotaBarFill.style.width = percentage + '%';
            
            if (remainingQuota < 1000) {
                quotaBarFill.style.background = '#ff6b6b';
                quotaContainer.classList.add('quota-warning');
            } else if (remainingQuota < 10000) {
                quotaBarFill.style.background = '#ffd700';
                quotaContainer.classList.remove('quota-warning');
            } else {
                quotaBarFill.style.background = 'linear-gradient(90deg, #4caf50, #ffd700)';
                quotaContainer.classList.remove('quota-warning');
            }
        }

        // Функция для перевода текста (только локальный словарь)
        async function translateToEnglish(text) {
            const translations = {
                // Существительные
                'лес': 'forest', 'леса': 'forest', 'лесу': 'forest', 'лесом': 'forest', 'лесе': 'forest',
                'закат': 'sunset', 'заката': 'sunset', 'закату': 'sunset', 'закатом': 'sunset', 'закате': 'sunset',
                'рассвет': 'sunrise', 'рассвета': 'sunrise', 'рассвету': 'sunrise', 'рассветом': 'sunrise',
                'город': 'city', 'города': 'city', 'городу': 'city', 'городом': 'city', 'городе': 'city',
                'небо': 'sky', 'неба': 'sky', 'небу': 'sky', 'небом': 'sky', 'небе': 'sky',
                'облака': 'clouds', 'облаков': 'clouds', 'облакам': 'clouds', 'облаках': 'clouds',
                'облако': 'cloud', 'тучи': 'clouds', 'туча': 'cloud',
                'гора': 'mountain', 'горы': 'mountains', 'горе': 'mountain', 'гору': 'mountain', 'горой': 'mountain', 'горах': 'mountains',
                'река': 'river', 'реки': 'river', 'реке': 'river', 'реку': 'river', 'рекой': 'river', 'рекою': 'river',
                'море': 'sea', 'моря': 'sea', 'морю': 'sea', 'морем': 'sea', 'море': 'sea',
                'океан': 'ocean', 'океана': 'ocean', 'океану': 'ocean', 'океаном': 'ocean', 'океане': 'ocean',
                'пляж': 'beach', 'пляжа': 'beach', 'пляжу': 'beach', 'пляжем': 'beach', 'пляже': 'beach',
                'дерево': 'tree', 'деревья': 'trees', 'деревьев': 'trees', 'деревьям': 'trees', 'деревьях': 'trees',
                'цветок': 'flower', 'цветы': 'flowers', 'цветов': 'flowers', 'цветам': 'flowers', 'цветах': 'flowers',
                'животное': 'animal', 'животные': 'animals', 'животных': 'animals', 'животным': 'animals',
                'птица': 'bird', 'птицы': 'birds', 'птиц': 'birds', 'птицам': 'birds', 'птицах': 'birds',
                'рыба': 'fish', 'рыбы': 'fish', 'рыб': 'fish', 'рыбам': 'fish',
                'кошка': 'cat', 'кошки': 'cat', 'кошку': 'cat', 'кошкой': 'cat', 'кошке': 'cat',
                'кот': 'cat', 'кота': 'cat', 'коту': 'cat', 'котом': 'cat', 'коте': 'cat',
                'собака': 'dog', 'собаки': 'dog', 'собаку': 'dog', 'собакой': 'dog', 'собаке': 'dog',
                'дом': 'house', 'дома': 'house', 'дому': 'house', 'домом': 'house', 'доме': 'house',
                'здание': 'building', 'здания': 'building', 'зданию': 'building', 'зданием': 'building', 'здании': 'building',
                'улица': 'street', 'улицы': 'street', 'улице': 'street', 'улицу': 'street', 'улицей': 'street',
                'дорога': 'road', 'дороги': 'road', 'дороге': 'road', 'дорогу': 'road', 'дорогой': 'road',
                'мост': 'bridge', 'моста': 'bridge', 'мосту': 'bridge', 'мостом': 'bridge', 'мосте': 'bridge',
                'парк': 'park', 'парка': 'park', 'парку': 'park', 'парком': 'park', 'парке': 'park',
                'сад': 'garden', 'сада': 'garden', 'саду': 'garden', 'садом': 'garden', 'саде': 'garden',
                'зима': 'winter', 'зимы': 'winter', 'зиме': 'winter', 'зиму': 'winter', 'зимой': 'winter',
                'весна': 'spring', 'весны': 'spring', 'весне': 'spring', 'весну': 'spring', 'весной': 'spring',
                'лето': 'summer', 'лета': 'summer', 'лету': 'summer', 'летом': 'summer', 'лете': 'summer',
                'осень': 'autumn', 'осени': 'autumn', 'осенью': 'autumn', 'осень': 'autumn',
                'дождь': 'rain', 'дождя': 'rain', 'дождю': 'rain', 'дождем': 'rain', 'дожде': 'rain',
                'снег': 'snow', 'снега': 'snow', 'снегу': 'snow', 'снегом': 'snow', 'снеге': 'snow',
                'ветер': 'wind', 'ветра': 'wind', 'ветру': 'wind', 'ветром': 'wind', 'ветре': 'wind',
                'солнце': 'sun', 'солнца': 'sun', 'солнцу': 'sun', 'солнцем': 'sun', 'солнце': 'sun',
                'луна': 'moon', 'луны': 'moon', 'луне': 'moon', 'луну': 'moon', 'луной': 'moon',
                'звезда': 'star', 'звезды': 'stars', 'звезд': 'stars', 'звездам': 'stars', 'звездах': 'stars',
                'космос': 'space', 'космоса': 'space', 'космосу': 'space', 'космосом': 'space', 'космосе': 'space',
                'планета': 'planet', 'планеты': 'planet', 'планете': 'planet', 'планету': 'planet', 'планетой': 'planet',
                'земля': 'earth', 'земли': 'earth', 'земле': 'earth', 'землю': 'earth', 'землей': 'earth',
                'вода': 'water', 'воды': 'water', 'воде': 'water', 'воду': 'water', 'водой': 'water',
                'огонь': 'fire', 'огня': 'fire', 'огню': 'fire', 'огнем': 'fire', 'огне': 'fire',
                'воздух': 'air', 'воздуха': 'air', 'воздуху': 'air', 'воздухом': 'air', 'воздухе': 'air',
                'туман': 'fog', 'тумана': 'fog', 'туману': 'fog', 'туманом': 'fog', 'тумане': 'fog',
                'радуга': 'rainbow', 'радуги': 'rainbow', 'радуге': 'rainbow', 'радугу': 'rainbow',
                'водопад': 'waterfall', 'водопада': 'waterfall', 'водопаду': 'waterfall', 'водопадом': 'waterfall',
                'озеро': 'lake', 'озера': 'lake', 'озеру': 'lake', 'озером': 'lake', 'озере': 'lake',
                'поле': 'field', 'поля': 'field', 'полю': 'field', 'полем': 'field', 'поле': 'field',
                'пустыня': 'desert', 'пустыни': 'desert', 'пустыне': 'desert', 'пустыню': 'desert',
                'остров': 'island', 'острова': 'island', 'острову': 'island', 'островом': 'island', 'острове': 'island',
                
                // Прилагательные
                'красивый': 'beautiful', 'красивая': 'beautiful', 'красивое': 'beautiful', 'красивые': 'beautiful',
                'большой': 'big', 'большая': 'big', 'большое': 'big', 'большие': 'big',
                'маленький': 'small', 'маленькая': 'small', 'маленькое': 'small', 'маленькие': 'small',
                'яркий': 'bright', 'яркая': 'bright', 'яркое': 'bright', 'яркие': 'bright',
                'темный': 'dark', 'темная': 'dark', 'темное': 'dark', 'темные': 'dark',
                'светлый': 'light', 'светлая': 'light', 'светлое': 'light', 'светлые': 'light',
                'цветной': 'colorful', 'цветная': 'colorful', 'цветное': 'colorful', 'цветные': 'colorful',
                'футуристический': 'futuristic', 'футуристическая': 'futuristic', 'футуристическое': 'futuristic',
                'старый': 'old', 'старая': 'old', 'старое': 'old', 'старые': 'old',
                'новый': 'new', 'новая': 'new', 'новое': 'new', 'новые': 'new',
                'современный': 'modern', 'современная': 'modern', 'современное': 'modern', 'современные': 'modern',
                'древний': 'ancient', 'древняя': 'ancient', 'древнее': 'ancient', 'древние': 'ancient',
                'волшебный': 'magical', 'волшебная': 'magical', 'волшебное': 'magical', 'волшебные': 'magical',
                'сказочный': 'fairy tale', 'сказочная': 'fairy tale', 'сказочное': 'fairy tale', 'сказочные': 'fairy tale',
                'реалистичный': 'realistic', 'реалистичная': 'realistic', 'реалистичное': 'realistic',
                'абстрактный': 'abstract', 'абстрактная': 'abstract', 'абстрактное': 'abstract',
                'ночной': 'night', 'ночная': 'night', 'ночное': 'night', 'ночные': 'night',
                'дневной': 'daytime', 'дневная': 'daytime', 'дневное': 'daytime',
                'утренний': 'morning', 'утренняя': 'morning', 'утреннее': 'morning',
                'вечерний': 'evening', 'вечерняя': 'evening', 'вечернее': 'evening',
                'золотой': 'golden', 'золотая': 'golden', 'золотое': 'golden', 'золотые': 'golden',
                'серебряный': 'silver', 'серебряная': 'silver', 'серебряное': 'silver',
                'хрустальный': 'crystal', 'хрустальная': 'crystal', 'хрустальное': 'crystal',
                'ледяной': 'icy', 'ледяная': 'icy', 'ледяное': 'icy', 'ледяные': 'icy',
                'огненный': 'fiery', 'огненная': 'fiery', 'огненное': 'fiery',
                
                // Глаголы
                'стоит': 'standing', 'стоят': 'standing',
                'лежит': 'lying', 'лежат': 'lying',
                'плывет': 'floating', 'плывут': 'floating',
                'летит': 'flying', 'летят': 'flying',
                'бежит': 'running', 'бегут': 'running',
                'идет': 'walking', 'идут': 'walking',
                'течет': 'flowing', 'текут': 'flowing',
                'светит': 'shining', 'светят': 'shining',
                'сияет': 'shining', 'сияют': 'shining',
                'горит': 'burning', 'горят': 'burning',
                'сверкает': 'sparkling', 'сверкают': 'sparkling',
                'цветет': 'blooming', 'цветут': 'blooming',
                'падает': 'falling', 'падают': 'falling',
                'поднимается': 'rising', 'поднимаются': 'rising',
                
                // Предлоги и союзы
                'с': 'with', 'со': 'with', 'и': 'and', 'в': 'in', 'во': 'in', 'на': 'on',
                'под': 'under', 'над': 'above', 'около': 'near', 'возле': 'near', 'у': 'by',
                'без': 'without', 'для': 'for', 'из': 'from', 'к': 'to', 'ко': 'to',
                'по': 'by', 'о': 'about', 'об': 'about', 'про': 'about', 'за': 'behind',
                'перед': 'in front of', 'между': 'between', 'через': 'through',
                
                // Дополнительные слова
                'очень': 'very', 'самый': 'most', 'много': 'many', 'мало': 'few',
                'немного': 'a little', 'совсем': 'completely', 'почти': 'almost',
                'уже': 'already', 'еще': 'still', 'тоже': 'also', 'также': 'also',
                'только': 'only', 'просто': 'just', 'даже': 'even', 'именно': 'exactly',
                'всегда': 'always', 'никогда': 'never', 'иногда': 'sometimes',
                'сегодня': 'today', 'завтра': 'tomorrow', 'вчера': 'yesterday',
                'сейчас': 'now', 'потом': 'later', 'раньше': 'earlier',
                'здесь': 'here', 'там': 'there', 'везде': 'everywhere',
                'далеко': 'far', 'близко': 'close', 'рядом': 'nearby'
            };
            
            // Разбиваем текст на слова
            let words = text.toLowerCase().split(/\s+/);
            let translatedWords = [];
            
            // Переводим каждое слово
            for (let word of words) {
                // Убираем знаки препинания для поиска
                const cleanWord = word.replace(/[.,!?;:()\[\]{}]/g, '');
                
                // Ищем перевод
                if (translations[cleanWord]) {
                    translatedWords.push(translations[cleanWord]);
                } else {
                    // Если слово не найдено, оставляем как есть
                    translatedWords.push(word);
                }
            }
            
            // Собираем обратно в предложение
            let translated = translatedWords.join(' ');
            
            return translated;
        }

        async function sendMessage() {
            const prompt = chatInput.value.trim();
            
            if (!prompt) {
                return;
            }

            if (remainingQuota < 0) {
                addMessage('ai', '❌ Дневной лимит запросов исчерпан! Попробуйте завтра.');
                return;
            }

            addMessage('user', prompt);
            saveChatToCookies();
            
            chatInput.value = '';
            
            const loadingMessage = addLoadingMessage();
            sendButton.disabled = true;
            chatInput.disabled = true;

            try {
                const translatedPrompt = await translateToEnglish(prompt);
                
                if (translatedPrompt !== prompt) {
                    addTranslationMessage(translatedPrompt);
                }
                
                const resolution = resolutionSelect.value;
                const style = styleSelect.value;
                
                let fullPrompt = translatedPrompt;
                if (style) {
                    const stylePrompts = {
                        'photorealistic': 'photorealistic, highly detailed, 8k resolution',
                        'anime': 'anime style, manga art, vibrant colors',
                        'watercolor': 'watercolor painting, soft colors, artistic',
                        'oil-painting': 'oil painting, classical art, textured brushstrokes',
                        'cyberpunk': 'cyberpunk style, neon lights, futuristic',
                        'fantasy': 'fantasy art, magical, epic scene'
                    };
                    fullPrompt = `${translatedPrompt}, ${stylePrompts[style]}`;
                }

                updateProgress(loadingMessage, 20);
                
                const response = await fetch(API_URL, {
                    method: "POST",
                    headers: {
                        "Authorization": `Bearer ${API_KEY}`,
                        "Content-Type": "application/json",
                    },
                    body: JSON.stringify({ 
                        prompt: fullPrompt,
                        resolution: resolution 
                    }),
                });

                updateProgress(loadingMessage, 60);

                if (!response.ok) {
                    const errorData = await response.json();
                    throw new Error(errorData.error || `HTTP ${response.status}`);
                }

                const blob = await response.blob();
                
                updateProgress(loadingMessage, 90);
                
                if (blob.size === 0) {
                    throw new Error('Получен пустой ответ');
                }

                
                loadingMessage.remove();
                
                const imageUrl = URL.createObjectURL(blob);
                
                addImageMessage(imageUrl, resolution);
                saveChatToCookies();
                
                // Обновляем квоту с сервера
                setTimeout(fetchQuota, 1000);
                
            } catch (error) {
                console.error('Ошибка:', error);
                loadingMessage.remove();
                addMessage('ai', '❌ Ошибка: ' + error.message);
                saveChatToCookies();
            } finally {
                sendButton.disabled = false;
                chatInput.disabled = false;
                chatInput.focus();
            }
        }

        function addMessage(type, text) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${type}`;
            
            const messageContent = document.createElement('div');
            messageContent.className = 'message-content';
            messageContent.textContent = text;
            
            messageDiv.appendChild(messageContent);
            chatMessages.appendChild(messageDiv);
            
            scrollToBottom();
            return messageDiv;
        }

        function addTranslationMessage(translatedText) {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message ai';
            
            const messageContent = document.createElement('div');
            messageContent.className = 'message-content';
            messageContent.textContent = 'Перевод: ' + translatedText;
            messageContent.style.fontSize = '12px';
            messageContent.style.opacity = '0.8';
            
            messageDiv.appendChild(messageContent);
            chatMessages.appendChild(messageDiv);
            
            scrollToBottom();
            return messageDiv;
        }

        function addImageMessage(imageUrl, resolution) {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message ai';
            
            const messageContent = document.createElement('div');
            messageContent.className = 'message-content';
            messageContent.textContent = 'Вот ваше изображение:';
            
            const image = document.createElement('img');
            image.src = imageUrl;
            image.className = 'ai-image';
            image.alt = 'Сгенерированное изображение';
            image.onclick = function() {
                openModal(imageUrl);
            };
            
            const info = document.createElement('div');
            info.className = 'image-info';
            info.textContent = `Разрешение: ${resolution}`;
            
            messageContent.appendChild(document.createElement('br'));
            messageContent.appendChild(image);
            messageContent.appendChild(info);
            messageDiv.appendChild(messageContent);
            
            chatMessages.appendChild(messageDiv);
            scrollToBottom();
            return messageDiv;
        }

        function addLoadingMessage() {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message ai';
            
            const messageContent = document.createElement('div');
            messageContent.className = 'message-content';
            
            const spinner = document.createElement('span');
            spinner.className = 'loading-spinner';
            
            const text = document.createTextNode('Генерирую изображение...');
            
            const progressContainer = document.createElement('div');
            progressContainer.className = 'progress-container';
            progressContainer.style.display = 'block';
            
            const progressBar = document.createElement('div');
            progressBar.className = 'progress-bar';
            
            const progressText = document.createElement('div');
            progressText.className = 'progress-text';
            progressText.textContent = '0%';
            
            progressContainer.appendChild(progressBar);
            progressContainer.appendChild(progressText);
            
            messageContent.appendChild(spinner);
            messageContent.appendChild(text);
            messageContent.appendChild(progressContainer);
            
            messageDiv.appendChild(messageContent);
            chatMessages.appendChild(messageDiv);
            
            scrollToBottom();
            return messageDiv;
        }

        function updateProgress(loadingMessage, percent) {
            const progressBar = loadingMessage.querySelector('.progress-bar');
            const progressText = loadingMessage.querySelector('.progress-text');
            
            if (progressBar && progressText) {
                progressBar.style.width = percent + '%';
                progressText.textContent = percent + '%';
            }
        }

        function scrollToBottom() {
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        function handleKeyPress(event) {
            if (event.key === 'Enter' && !event.shiftKey) {
                event.preventDefault();
                sendMessage();
            }
        }

        function clearChat() {
            chatMessages.innerHTML = '';
            
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message ai';
            
            const messageContent = document.createElement('div');
            messageContent.className = 'message-content';
            messageContent.textContent = 'Привет! Я AI генератор изображений. Опишите, какое изображение вы хотите создать, и я сгенерирую его для вас!';
            
            messageDiv.appendChild(messageContent);
            chatMessages.appendChild(messageDiv);
            
            // Очищаем cookies
            deleteCookie('chatMessages');
        }

        function openModal(imageUrl) {
            const modal = document.getElementById('imageModal');
            const modalImage = document.getElementById('modalImage');
            
            modalImage.src = imageUrl;
            modal.classList.add('active');
        }

        function closeModal() {
            const modal = document.getElementById('imageModal');
            modal.classList.remove('active');
        }

        document.addEventListener('keydown', function(event) {
            if (event.key === 'Escape') {
                closeModal();
            }
        });

        // Инициализация
        loadChatFromCookies();
        //fetchQuota();
        chatInput.focus();
        
        // Обновляем квоту каждые 5 минут
        //setInterval(fetchQuota, 300000);
        
        // Сохраняем чат при закрытии страницы
        window.addEventListener('beforeunload', saveChatToCookies);
    </script>
</body>
</html>
