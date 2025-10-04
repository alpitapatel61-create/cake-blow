// Global variables
let candleCount = 0;
const maxCandles = 20;
let activeCandles = [];
const cake = document.getElementById('cake');
const countDisplay = document.querySelector('.candle-count');
let audioContext, analyser, dataArray;
let isBlowingDetected = false;

// Add candle on cake click
function addCandle(event) {
    if (candleCount >= maxCandles) {
        alert('Maximum 20 candles added! Now blow them out to celebrate. 🎉');
        return;
    }

    candleCount++;
    updateCountDisplay();

    // Create candle
    const candle = document.createElement('div');
    candle.classList.add('candle');

    // Position based on click (relative to cake)
    const rect = cake.getBoundingClientRect();
    const x = ((event.clientX - rect.left) / rect.width) * 100;
    const y = ((event.clientY - rect.top) / rect.height) * 100;
    candle.style.left = `${Math.max(5, Math.min(95, x))}%`; // Clamp to edges
    candle.style.top = `${Math.max(10, Math.min(80, y))}%`; // Avoid edges

    // Add flame
    const flame = document.createElement('div');
    flame.classList.add('flame');
    candle.appendChild(flame);

    cake.appendChild(candle);
    activeCandles.push(candle);
}

// Update display
function updateCountDisplay() {
    countDisplay.textContent = `Candles: ${candleCount} / ${maxCandles}`;
}

// Blow detection function
function blowCandles() {
    if (!analyser) return;

    analyser.getByteFrequencyData(dataArray);
    const volume = dataArray.reduce((a, b) => a + b) / dataArray.length;

    if (volume > 30) { // Threshold for blowing (adjust if needed)
        activeCandles.forEach((candle, index) => {
            const flame = candle.querySelector('.flame');
            if (flame) {
                flame.style.opacity = '0';
                flame.style.animation = 'none';
                setTimeout(() => {
                    candle.style.opacity = '0';
                    setTimeout(() => candle.remove(), 300);
                }, 200);
            }
        });
        activeCandles = []; // Clear all at once for simplicity (or extinguish one by one if preferred)

        if (candleCount === maxCandles && activeCandles.length === 0) {
            showHappyBirthday();
        }
    }
}

// Animation loop for mic
function animate() {
    blowCandles();
    requestAnimationFrame(animate);
}

// Show Happy Birthday message
function showHappyBirthday() {
    const message = document.createElement('div');
    message.id = 'birthday-message';
    message.innerHTML = `
        <h2>Happy Birthday! 🎉🎂</h2>
        <p>All candles blown out! Time to celebrate! 🥳</p>
    `;
    message.style.display = 'block';
    message.style.animation = 'celebrate 1s ease-out';
    document.body.appendChild(message);

    // Auto-remove after 5 seconds
    setTimeout(() => {
        message.style.animation = 'none';
        setTimeout(() => message.remove(), 500);
    }, 5000);
}

// Initialize microphone
async function initMic() {
    try {
        const stream = await navigator.mediaDevices.getUser Media({ audio: true });
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
        const source = audioContext.createMediaStreamSource(stream);
        analyser = audioContext.createAnalyser();
        analyser.fftSize = 256;
        const bufferLength = analyser.frequencyBinCount;
        dataArray = new Uint8Array(bufferLength);
        source.connect(analyser);
        animate();
    } catch (err) {
        console.error('Microphone access denied:', err);
        alert('Please allow microphone access to blow out the candles!');
    }
}

// Event listeners
cake.addEventListener('click', addCandle);

// Start mic on load (or user gesture for better compatibility)
window.addEventListener('load', () => {
    // Resume audio context on user interaction if suspended
    document.addEventListener('click', () => {
        if (audioContext && audioContext.state === 'suspended') {
            audioContext.resume();
        }
        if (!audioContext) initMic();
    }, { once: true });
});
LICENSE). This means it can be used, modified, and distributed freely, as long as it is not used for commercial purposes.
