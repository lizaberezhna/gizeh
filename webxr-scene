import * as THREE from 'three';
import { ARButton } from 'ARButton';

// ========== ІНІЦІАЛІЗАЦІЯ СЦЕНИ ==========

const scene = new THREE.Scene();
scene.background = null; // Прозорий фон для AR
scene.environment = null;

// ========== НАЛАШТУВАННЯ КАМЕРИ ==========
// Камера створюється автоматично WebXR, але ми задаємо параметри
const camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.01, 100);

// ========== НАЛАШТУВАННЯ РЕНДЕРЕРА ==========
const renderer = new THREE.WebGLRenderer({ 
    antialias: true, 
    alpha: true 
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
renderer.xr.enabled = true; // Активація WebXR

// ========== ОСВІТЛЕННЯ ==========
// Головне джерело світла (сонце)
const directionalLight = new THREE.DirectionalLight(0xFFF5E0, 1.2);
directionalLight.position.set(5, 10, 3);
directionalLight.castShadow = true;
directionalLight.receiveShadow = true;
scene.add(directionalLight);

// Додаткове освітлення знизу
const fillLight = new THREE.PointLight(0xCCAA88, 0.5);
fillLight.position.set(0, -1, 0);
scene.add(fillLight);

// М'яке загальне освітлення
const ambientLight = new THREE.AmbientLight(0x404060, 0.6);
scene.add(ambientLight);

// Контрове світло ззаду
const backLight = new THREE.PointLight(0xFFAA66, 0.4);
backLight.position.set(-2, 3, -4);
scene.add(backLight);

// ========== ДОПОМІЖНІ ФУНКЦІЇ ==========
function updateStatus(message, isOk = true) {
    const statusDiv = document.getElementById('status');
    if (statusDiv) {
        statusDiv.innerHTML = message;
        statusDiv.style.color = isOk ? '#0f0' : '#ff0';
        if (!isOk) {
            setTimeout(() => {
                statusDiv.style.color = '#0f0';
            }, 3000);
        }
    }
    console.log('[WebXR]', message);
}

// ========== СТВОРЕННЯ ПІРАМІД ==========

/**
 * Створення піраміди з цегляною текстурою та золотою верхівкою
 * @param {number} radius - радіус основи (метри)
 * @param {number} height - висота (метри)
 * @param {number} x - позиція по X
 * @param {number} z - позиція по Z
 * @param {string} color - колір/текстура
 * @param {number} goldDelay - затримка анімації верхівки
 * @returns {THREE.Group} - група з пірамідою
 */
function createPyramid(radius, height, x, z, color, goldDelay = 0) {
    const group = new THREE.Group();
    
    // Геометрія піраміди (конус з 4 гранями для вигляду піраміди)
    const geometry = new THREE.ConeGeometry(radius, height, 32);
    
    // Матеріал з цегляною текстурою
    // Створюємо canvas текстуру "цегли" програмно (без зовнішніх файлів)
    const canvas = document.createElement('canvas');
    canvas.width = 512;
    canvas.height = 512;
    const ctx = canvas.getContext('2d');
    
    // Малюємо цегляну текстуру
    ctx.fillStyle = '#B5651D';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Малюємо цеглинки
    ctx.strokeStyle = '#8B4513';
    ctx.lineWidth = 4;
    const brickW = 64;
    const brickH = 32;
    
    for (let y = 0; y < canvas.height; y += brickH) {
        const offset = (Math.floor(y / brickH) % 2) * brickW / 2;
        for (let x = offset; x < canvas.width; x += brickW) {
            ctx.strokeRect(x + 2, y + 2, brickW - 4, brickH - 4);
            ctx.fillStyle = '#C47E3A';
            ctx.fillRect(x + 2, y + 2, brickW - 4, brickH - 4);
            ctx.fillStyle = '#B5651D';
        }
    }
    
    const brickTexture = new THREE.CanvasTexture(canvas);
    brickTexture.wrapS = THREE.RepeatWrapping;
    brickTexture.wrapT = THREE.RepeatWrapping;
    brickTexture.repeat.set(radius / 2, height / 3);
    
    const material = new THREE.MeshStandardMaterial({
        map: brickTexture,
        roughness: 0.7,
        metalness: 0.1,
        color: 0xFFFFFF
    });
    
    // Основне тіло піраміди
    const pyramid = new THREE.Mesh(geometry, material);
    pyramid.castShadow = true;
    pyramid.receiveShadow = true;
    pyramid.position.y = height / 2;
    group.add(pyramid);
    
    // Золота верхівка (пірамідіон)
    const goldGeometry = new THREE.ConeGeometry(radius * 0.07, height * 0.08, 16);
    const goldMaterial = new THREE.MeshStandardMaterial({
        color: 0xFFD700,
        metalness: 0.95,
        roughness: 0.2,
        emissive: 0x442200,
        emissiveIntensity: 0.3
    });
    
    const goldTop = new THREE.Mesh(goldGeometry, goldMaterial);
    goldTop.castShadow = true;
    goldTop.position.y = height;
    group.add(goldTop);
    
    // Анімація золотої верхівки (пульсація)
    const startTime = performance.now();
    function animateGold() {
        const elapsed = (performance.now() - startTime) / 1000;
        const intensity = 0.3 + Math.sin(elapsed * 3 + goldDelay) * 0.15;
        goldMaterial.emissiveIntensity = intensity;
        
        // Зміна кольору в залежності від часу
        const hue = 0.12 + Math.sin(elapsed * 1.5 + goldDelay) * 0.03; // Золотий діапазон
        goldMaterial.color.setHSL(hue, 1.0, 0.55);
        
        requestAnimationFrame(animateGold);
    }
    animateGold();
    
    // Позиціонування
    group.position.set(x, 0, z);
    
    return group;
}

/**
 * Створення піщаної основи (землі)
 */
function createGround() {
    const groundGeometry = new THREE.PlaneGeometry(12, 10);
    
    // Створюємо піщану текстуру
    const canvas = document.createElement('canvas');
    canvas.width = 512;
    canvas.height = 512;
    const ctx = canvas.getContext('2d');
    
    ctx.fillStyle = '#D2B48C';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Додаємо текстуру піску (точки)
    for (let i = 0; i < 2000; i++) {
        ctx.fillStyle = `rgba(160, 100, 60, ${Math.random() * 0.3})`;
        ctx.fillRect(
            Math.random() * canvas.width,
            Math.random() * canvas.height,
            2, 2
        );
    }
    
    const groundTexture = new THREE.CanvasTexture(canvas);
    groundTexture.wrapS = THREE.RepeatWrapping;
    groundTexture.wrapT = THREE.RepeatWrapping;
    groundTexture.repeat.set(4, 4);
    
    const groundMaterial = new THREE.MeshStandardMaterial({
        map: groundTexture,
        roughness: 0.9,
        metalness: 0.05,
        color: 0xFFFFFF
    });
    
    const ground = new THREE.Mesh(groundGeometry, groundMaterial);
    ground.rotation.x = -Math.PI / 2;
    ground.position.y = -0.05;
    ground.receiveShadow = true;
    
    return ground;
}

// ========== СТВОРЕННЯ ПІРАМІД З ПРАВИЛЬНИМИ ПРОПОРЦІЯМИ ==========

// Пропорції збережені відносно реальних розмірів
// Масштаб: 1 одиниця = 1 метр (але зменшено для AR)
const scale = 0.008; // Коефіцієнт зменшення (1:125)

// Піраміда Хеопса (найбільша) - центр
// Реальні розміри: висота 145м, основа 230м
const khufu = createPyramid(230 * scale, 145 * scale, 0, 0, '#B5651D', 0);
scene.add(khufu);

// Піраміда Хефрена (середня) - зліва
// Реальні розміри: висота 135м, основа 210м
const khafre = createPyramid(210 * scale, 135 * scale, -2.2, -1.5, '#B5651D', 1);
scene.add(khafre);

// Піраміда Мікеріна (мала) - справа
// Реальні розміри: висота 65м, основа 100м
const menkaure = createPyramid(100 * scale, 65 * scale, 2.0, -1.2, '#B5651D', 2);
scene.add(menkaure);

// Додавання піщаної основи
const ground = createGround();
scene.add(ground);

// ========== ДЕКОРАТИВНІ ЕЛЕМЕНТИ ==========

// Сфінкс (спрощена модель)
function createSphinx() {
    const group = new THREE.Group();
    
    // Тіло (лев)
    const bodyGeo = new THREE.BoxGeometry(0.4, 0.2, 0.8);
    const bodyMat = new THREE.MeshStandardMaterial({ color: 0xCD853F, roughness: 0.6 });
    const body = new THREE.Mesh(bodyGeo, bodyMat);
    body.position.y = 0.1;
    body.castShadow = true;
    group.add(body);
    
    // Голова (людини)
    const headGeo = new THREE.BoxGeometry(0.25, 0.25, 0.25);
    const headMat = new THREE.MeshStandardMaterial({ color: 0xDEB887, roughness: 0.4 });
    const head = new THREE.Mesh(headGeo, headMat);
    head.position.y = 0.3;
    head.castShadow = true;
    group.add(head);
    
    // Головний убір (Немес)
    const nemesGeo = new THREE.BoxGeometry(0.3, 0.08, 0.3);
    const nemesMat = new THREE.MeshStandardMaterial({ color: 0xFFD700, metalness: 0.5 });
    const nemes = new THREE.Mesh(nemesGeo, nemesMat);
    nemes.position.y = 0.45;
    group.add(nemes);
    
    group.position.set(-1.2, -0.15, 2.5);
    group.scale.set(0.8, 0.8, 0.8);
    
    return group;
}

const sphinx = createSphinx();
scene.add(sphinx);

// Камера з анімацією "наїзджання" (перспектива)
// Додаємо об'єкт-маркер для анімації камери
const cameraTarget = new THREE.Object3D();
cameraTarget.position.set(0, 0.5, 1.5);
scene.add(cameraTarget);

// ========== АНІМАЦІЯ ==========

let time = 0;

function animate() {
    time += 0.016; // Приблизно 60 FPS
    
    // Анімація "наїзджання" камери до центральної піраміди
    // Камера рухається по синусоїді вперед-назад
    if (renderer.xr.isPresenting) {
        // Тільки в AR режимі
        const t = Math.sin(time * 0.3) * 0.5;
        // Невелике наближення/віддалення
        // WebXR камера керується системою, тому ми рухаємо ціль
    }
    
    // Обертання сфінкса (повільне)
    if (sphinx) {
        sphinx.rotation.y = Math.sin(time * 0.2) * 0.1;
    }
    
    // Ефект "сонячного світла" - зміна інтенсивності
    const sunIntensity = 1.0 + Math.sin(time * 0.5) * 0.1;
    directionalLight.intensity = sunIntensity;
    
    // Оновлення текстур
    if (khufu.children[0]?.material?.map) {
        khufu.children[0].material.map.needsUpdate = true;
    }
}

// ========== СТВОРЕННЯ AR BUTTON ==========

const arButton = ARButton.createButton(renderer, {
    requiredFeatures: ['hit-test'],
    optionalFeatures: ['dom-overlay'],
    domOverlay: { root: document.body }
});
arButton.classList.add('ar-button');
document.body.appendChild(arButton);

// Додаємо стиль для кнопки
const style = document.createElement('style');
style.textContent = `
    .ar-button {
        position: fixed !important;
        bottom: 30px !important;
        left: 50% !important;
        transform: translateX(-50%) !important;
        padding: 12px 24px !important;
        font-size: 16px !important;
        font-weight: bold !important;
        background: #FFD700 !important;
        color: #000 !important;
        border: none !important;
        border-radius: 40px !important;
        cursor: pointer !important;
        z-index: 200 !important;
        font-family: monospace !important;
        box-shadow: 0 4px 15px rgba(0,0,0,0.3) !important;
    }
    .ar-button:hover {
        background: #FFC107 !important;
        transform: translateX(-50%) scale(1.02) !important;
    }
`;
document.head.appendChild(style);

// ========== ЗАПУСК АНІМАЦІЇ ==========

// Оновлення статусу
updateStatus('✅ WebXR готовий | Натисніть кнопку AR');

// Сповіщення про зміну стану сесії
renderer.xr.addEventListener('sessionstart', () => {
    updateStatus('🎉 AR-сесія активна! Шукайте піраміди навколо');
    const instruction = document.getElementById('instruction');
    if (instruction) {
        instruction.innerHTML = '🏛️ Піраміди з\'явилися! Обійдіть їх навколо';
        instruction.style.background = 'rgba(0,100,0,0.8)';
    }
});

renderer.xr.addEventListener('sessionend', () => {
    updateStatus('⏸️ AR-сесію завершено');
    const instruction = document.getElementById('instruction');
    if (instruction) {
        instruction.innerHTML = '📱 Натисніть кнопку AR для запуску';
        instruction.style.background = 'rgba(0,0,0,0.7)';
    }
});

// Цикл анімації WebXR
renderer.setAnimationLoop((timestamp, frame) => {
    if (frame) {
        // Отримуємо позицію камери для hit-test (опціонально)
        const session = renderer.xr.getSession();
        if (session && frame) {
            // Можна додати hit-test для розміщення на реальній поверхні
        }
    }
    
    animate();
    renderer.render(scene, camera);
});

// ========== ОБРОБКА ЗМІНИ РОЗМІРУ ВІКНА ==========
window.addEventListener('resize', onWindowResize, false);
function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
}

// ========== ДОПОМІЖНІ ФУНКЦІЇ ДЛЯ НАЛАГОДЖЕННЯ ==========
console.log('🏛️ WebXR Піраміди Гізи завантажено');
console.log('📐 Масштаб: 1:' + (1 / scale));
console.log('🏗️ Піраміди створено: Хеопс, Хефрен, Мікерін');

// Експорт для налагодження (опціонально)
window.debugScene = scene;
window.debugCamera = camera;