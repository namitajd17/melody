# melody
web harmonium
// ===============================
// WEB HARMONIUM - JAVASCRIPT
// ===============================

// Wait until the HTML page is loaded
document.addEventListener("DOMContentLoaded", () => {

    // Audio setup
    const AudioContextClass =
        window.AudioContext || window.webkitAudioContext;

    const audioContext = new AudioContextClass();

    // Master volume
    const masterGain = audioContext.createGain();
    masterGain.connect(audioContext.destination);
    masterGain.gain.value = 0.5;

    // Store currently playing notes
    const activeNotes = {};

    // Frequencies
    const frequencies = {
        C: 261.63,   // Sa
        D: 293.66,   // Re
        E: 329.63,   // Ga
        F: 349.23,   // Ma
        G: 392.00,   // Pa
        A: 440.00,   // Dha
        B: 493.88,   // Ni
        C2: 523.25   // Higher Sa
    };

    // Laptop keyboard mapping
    const keyMapping = {
        a: "C",
        s: "D",
        d: "E",
        f: "F",
        g: "G",
        h: "A",
        j: "B",
        k: "C2"
    };

    // Get HTML keys
    const keys = document.querySelectorAll(".key");

    // ===============================
    // PLAY NOTE
    // ===============================

    function playNote(note, element) {

        // Prevent duplicate notes
        if (activeNotes[note]) return;

        // Resume audio if browser suspended it
        if (audioContext.state === "suspended") {
            audioContext.resume();
        }

        // Create oscillator
        const oscillator = audioContext.createOscillator();

        // Create gain for smooth sound
        const gain = audioContext.createGain();

        // Harmonium-like waveform
        oscillator.type = "sawtooth";

        // Set frequency
        oscillator.frequency.value = frequencies[note];

        // Smooth attack
        gain.gain.setValueAtTime(
            0,
            audioContext.currentTime
        );

        gain.gain.linearRampToValueAtTime(
            0.25,
            audioContext.currentTime + 0.05
        );

        // Connect sound
        oscillator.connect(gain);
        gain.connect(masterGain);

        // Start sound
        oscillator.start();

        // Save active note
        activeNotes[note] = {
            oscillator: oscillator,
            gain: gain,
            element: element
        };

        // Visual effect
        if (element) {
            element.classList.add("active");
        }
    }

    // ===============================
    // STOP NOTE
    // ===============================

    function stopNote(note) {

        const active = activeNotes[note];

        if (!active) return;

        const now = audioContext.currentTime;

        // Smooth release
        active.gain.gain.cancelScheduledValues(now);

        active.gain.gain.setValueAtTime(
            active.gain.gain.value,
            now
        );

        active.gain.gain.linearRampToValueAtTime(
            0.001,
            now + 0.1
        );

        // Stop oscillator
        active.oscillator.stop(now + 0.12);

        // Remove visual effect
        if (active.element) {
            active.element.classList.remove("active");
        }

        // Remove note from active notes
        delete activeNotes[note];
    }

    // ===============================
    // LAPTOP KEYBOARD
    // ===============================

    document.addEventListener("keydown", (event) => {

        // Ignore repeated key presses
        if (event.repeat) return;

        const pressedKey =
            event.key.toLowerCase();

        const note =
            keyMapping[pressedKey];

        if (!note) return;

        // Find matching HTML key
        const element =
            document.querySelector(
                `[data-note="${note}"]`
            );

        playNote(note, element);
    });


    document.addEventListener("keyup", (event) => {

        const releasedKey =
            event.key.toLowerCase();

        const note =
            keyMapping[releasedKey];

        if (!note) return;

        stopNote(note);
    });

    // ===============================
    // MOUSE SUPPORT
    // ===============================

    keys.forEach((key) => {

        const note =
            key.dataset.note;

        key.addEventListener("mousedown", () => {
            playNote(note, key);
        });

        key.addEventListener("mouseup", () => {
            stopNote(note);
        });

        key.addEventListener("mouseleave", () => {
            stopNote(note);
        });

    });

    // ===============================
    // TOUCH SUPPORT
    // ===============================

    keys.forEach((key) => {

        const note =
            key.dataset.note;

        key.addEventListener(
            "touchstart",
            (event) => {

                event.preventDefault();

                playNote(note, key);
            }
        );

        key.addEventListener(
            "touchend",
            (event) => {

                event.preventDefault();

                stopNote(note);
            }
        );

    });

    // ===============================
    // VOLUME CONTROL
    // ===============================

    const volumeSlider =
        document.getElementById("volume");

    if (volumeSlider) {

        volumeSlider.addEventListener(
            "input",
            () => {

                const volume =
                    volumeSlider.value / 100;

                masterGain.gain.setTargetAtTime(
                    volume,
                    audioContext.currentTime,
                    0.01
                );
            }
        );
    }

    // ===============================
    // STOP ALL NOTES
    // ===============================

    const stopAllButton =
        document.getElementById("stopAll");

    if (stopAllButton) {

        stopAllButton.addEventListener(
            "click",
            () => {

                Object.keys(activeNotes)
                    .forEach((note) => {
                        stopNote(note);
                    });

            }
        );
    }

    // Stop notes if user leaves window
    window.addEventListener("blur", () => {

        Object.keys(activeNotes)
            .forEach((note) => {
                stopNote(note);
            });

    });

});
