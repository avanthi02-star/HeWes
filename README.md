<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>AI Health Analyzer</title>

<style>
body {
    font-family: Arial;
    background: #eef2f7;
    text-align: center;
}

.container {
    width: 400px;
    margin: auto;
    margin-top: 40px;
    background: white;
    padding: 20px;
    border-radius: 10px;
}

button {
    padding: 10px;
    background: teal;
    color: white;
    border: none;
    margin-top: 10px;
    cursor: pointer;
}

input, select, textarea {
    width: 90%;
    padding: 8px;
    margin-top: 10px;
}

.progress {
    height: 10px;
    background: lightgray;
    border-radius: 5px;
    margin-bottom: 15px;
}

.progress-bar {
    height: 10px;
    background: teal;
    width: 0%;
    border-radius: 5px;
}
</style>
</head>

<body>

<div class="container">
    <div class="progress"><div class="progress-bar" id="progress"></div></div>

    <h2 id="title">Step 1</h2>
    <div id="content"></div>

    <p style="font-size:12px;color:red;">
        ⚠ This is not a medical diagnosis. Consult a doctor.
    </p>
</div>

<script>
let step = 1;
let data = {};

function updateProgress() {
    document.getElementById("progress").style.width = (step * 16) + "%";
}

// STEP RENDER
function render() {
    updateProgress();
    let c = document.getElementById("content");

    if (step === 1) {
        document.getElementById("title").innerText = "Basic Info";
        c.innerHTML = `
            Age: <input id="age"><br>
            Gender:
            <select id="gender">
                <option>Male</option>
                <option>Female</option>
            </select>
            <button onclick="next1()">Next</button>
        `;
    }

    else if (step === 2) {
        document.getElementById("title").innerText = "Medical History";
        c.innerHTML = `
            Existing disease?
            <select id="disease">
                <option>No</option>
                <option>Yes</option>
            </select><br>
            <input id="diseaseDetail" placeholder="If yes, specify">
            <button onclick="next2()">Next</button>
        `;
    }

    else if (step === 3) {
        document.getElementById("title").innerText = "Symptoms";

        c.innerHTML = `
            <textarea id="symptoms" placeholder="Enter symptoms"></textarea><br>
            <button onclick="startVoice()">🎤 Speak</button><br>
            <button onclick="next3()">Next</button>
        `;
    }

    else if (step === 4) {
        document.getElementById("title").innerText = "Details";

        c.innerHTML = `
            Location:
            <input id="location" placeholder="left/right/full"><br>
            Severity:
            <select id="severity">
                <option>Mild</option>
                <option>Moderate</option>
                <option>Severe</option>
            </select><br>
            Duration (days):
            <input id="duration"><br>
            Frequency:
            <input id="frequency"><br>
            <button onclick="next4()">Analyze</button>
        `;
    }

    else if (step === 5) {
        analyze();
    }

    else if (step === 6) {
        showTreatment();
    }
}

// STEP FUNCTIONS
function next1() {
    data.age = age.value;
    data.gender = gender.value;
    step++; render();
}

function next2() {
    data.disease = disease.value;
    data.diseaseDetail = diseaseDetail.value;
    step++; render();
}

function next3() {
    data.symptoms = symptoms.value.toLowerCase();
    step++; render();
}

function next4() {
    data.location = location.value;
    data.severity = severity.value;
    data.duration = duration.value;
    data.frequency = frequency.value;
    step++; render();
}

// VOICE INPUT
function startVoice() {
    let recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recognition.lang = "en-IN";

    recognition.onresult = function(event) {
        document.getElementById("symptoms").value = event.results[0][0].transcript;
    };

    recognition.start();
}

// AI ANALYSIS (Rule-Based)
function analyze() {
    let result = "General Condition";
    let cause = "Unknown";
    let risk = "Low";

    if (data.symptoms.includes("headache")) {
        if (data.location.includes("left") || data.location.includes("right")) {
            result = "Migraine";
            cause = "Stress / Hormonal / Sleep issues";
        } else {
            result = "Tension Headache";
            cause = "Stress / posture";
        }
    }

    if (data.symptoms.includes("fever")) {
        result = "Possible Infection";
        cause = "Viral / Bacterial";
    }

    if (data.severity === "Severe") risk = "High";
    else if (data.severity === "Moderate") risk = "Medium";

    document.getElementById("title").innerText = "Analysis";

    document.getElementById("content").innerHTML = `
        <b>Condition:</b> ${result}<br>
        <b>Cause:</b> ${cause}<br>
        <b>Risk:</b> ${risk}<br><br>

        Choose Treatment:
        <select id="treatment">
            <option>Allopathy</option>
            <option>AYUSH</option>
        </select><br>

        <div id="ayushDiv" style="display:none;">
            <select id="ayushType">
                <option>Ayurveda</option>
                <option>Siddha</option>
                <option>Unani</option>
                <option>Yoga</option>
                <option>Homeopathy</option>
            </select>
        </div>

        <button onclick="nextTreatment()">Get Solution</button>
    `;

    document.getElementById("treatment").onchange = function() {
        ayushDiv.style.display = this.value === "AYUSH" ? "block" : "none";
    };
}

// TREATMENT
function nextTreatment() {
    data.treatment = treatment.value;
    data.ayush = ayushType ? ayushType.value : "";
    step++; render();
}

function showTreatment() {
    let sol = "";

    if (data.treatment === "Allopathy") {
        sol = "Take rest, hydration, and basic OTC medicines like paracetamol.";
    } else {
        if (data.ayush === "Ayurveda") {
            sol = "Use Tulsi, ginger tea, and balanced diet.";
        }
        else if (data.ayush === "Siddha") {
            sol = "Use Siddha herbal formulations.";
        }
        else if (data.ayush === "Unani") {
            sol = "Cooling herbs and natural therapy.";
        }
        else if (data.ayush === "Yoga") {
            sol = "Practice pranayama and relaxation yoga.";
        }
        else {
            sol = "Use mild homeopathic remedies.";
        }
    }

    document.getElementById("title").innerText = "Final Result";

    document.getElementById("content").innerHTML = `
        <h3>Summary</h3>
        Symptoms: ${data.symptoms}<br>
        Severity: ${data.severity}<br><br>

        <h3>Advice</h3>
        ${sol}<br><br>

        <button onclick="location.reload()">Restart</button>
    `;
}

// START
render();
</script>

</body>
</html>
