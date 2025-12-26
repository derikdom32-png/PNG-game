# PNG-game
PNG multiplayer game
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>PNG Multiplayer Game</title>
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#ff6f61">
<style>
/* Gradient Background */
body {
    font-family: Arial, sans-serif;
    text-align: center;
    background: linear-gradient(135deg, #ff9a9e, #fad0c4, #a18cd1, #fbc2eb);
    background-size: 400% 400%;
    animation: gradientBG 15s ease infinite;
    padding: 50px;
    color: #333;
}

@keyframes gradientBG {
    0%{background-position:0% 50%}
    50%{background-position:100% 50%}
    100%{background-position:0% 50%}
}

h1 { font-size: 2.2em; margin-bottom: 20px; color: #fff; text-shadow: 2px 2px 5px rgba(0,0,0,0.3);}
input, button { padding: 10px; font-size:16px; border-radius:8px; border:none; margin:5px; outline:none;}
button { cursor:pointer; background:#ff6f61; color:white; font-weight:bold; transition: transform 0.2s, box-shadow 0.3s;}
button:hover { transform: scale(1.1); box-shadow: 0 0 15px #fff;}
#gameArea{margin-top:20px; display:none;}
#history{margin-top:20px; text-align:left; max-width:400px; margin-left:auto; margin-right:auto;}
.correct{color:limegreen; font-weight:bold; font-size:1.1em;}
.loadingD{font-weight:bold; font-size:24px; font-family:monospace; letter-spacing:5px; color:gold; margin-top:10px; display:none; animation:bounceD 1s infinite;}
@keyframes bounceD{0%{transform:translateY(0) scale(1);}25%{transform:translateY(-10px) scale(1.2);}50%{transform:translateY(0) scale(1);}75%{transform:translateY(-5px) scale(1.1);}100%{transform:translateY(0) scale(1);}}
</style>
</head>
<body>

<h1>PNG Multiplayer 🎮</h1>

<p>Player 1: Enter a 4-digit number (1-9, no repeats)</p>
<input type="password" id="secretNumber" placeholder="Enter secret number">
<button onclick="startGame()">Start Game</button>

<div id="gameArea">
    <p>Player 2: Enter your guess 🔢</p>
    <input type="text" id="guessInput" maxlength="4" placeholder="4-digit number">
    <button onclick="makeGuess()">Guess</button>

    <div class="loadingD" id="loading">D D D D D</div>
    <div id="history"></div>
</div>

<!-- Sound effects -->
<audio id="correctSound" src="correct.mp3"></audio>
<audio id="wrongSound" src="wrong.mp3"></audio>

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
// Firebase configuration (replace with your real config)
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT.firebaseio.com",
    projectId: "YOUR_PROJECT",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "SENDER_ID",
    appId: "APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

let historyDiv = document.getElementById("history");
let loading = document.getElementById("loading");

function startGame() {
    let value = document.getElementById("secretNumber").value;
    if(!/^[1-9]{4}$/.test(value) || new Set(value).size !== 4){
        alert("❌ Enter 4 digits, no 0, no repeats!");
        return;
    }
    // Save secret to Firebase
    db.ref('pngGame/secret').set(value);
    document.getElementById("gameArea").style.display = "block";
    document.getElementById("secretNumber").disabled = true;
    alert("Game started! 🕹️ Player 2 can now guess.");
}

// Listen for new guesses
db.ref('pngGame/guesses').on('child_added', snapshot => {
    let guess = snapshot.val();
    checkGuessFirebase(guess);
});

function makeGuess() {
    let guess = document.getElementById("guessInput").value;
    if(!/^[1-9]{4}$/.test(guess) || new Set(guess).size !== 4){
        alert("❌ Enter 4 digits, no 0, no repeats!");
        return;
    }
    // Push guess to Firebase
    db.ref('pngGame/guesses').push(guess);
    document.getElementById("guessInput").value = "";
}

function checkGuessFirebase(guess){
    // Show suspense
    loading.style.display = "block";
    document.getElementById("guessInput").disabled = true;

    setTimeout(() => {
        loading.style.display = "none";
        document.getElementById("guessInput").disabled = false;

        db.ref('pngGame/secret').once('value').then(snap=>{
            let secret = snap.val();
            let numberCorrect=0;
            let positionCorrect=0;

            for(let i=0;i<4;i++){
                if(secret.includes(guess[i])) numberCorrect++;
                if(secret[i]===guess[i]) positionCorrect++;
            }

            // Play sound
            if(positionCorrect===4){document.getElementById("correctSound").play();}
            else{document.getElementById("wrongSound").play();}

            let emoji = positionCorrect===4?"🎉":"🤔";
            let result=`Guess: ${guess} → Number: ${numberCorrect}, Position: ${positionCorrect} ${emoji}`;
            let p=document.createElement("p");

            if(positionCorrect===4){
                p.innerHTML=`<span class="correct">${result} ✅ Correct!</span>`;
                historyDiv.prepend(p);

                // Reset new secret
                setTimeout(()=>{
                    let newSecret = prompt("🎮 Player 1: Enter new 4-digit number (no 0, no repeats):");
                    if(newSecret){
                        db.ref('pngGame/secret').set(newSecret);
                        db.ref('pngGame/guesses').remove();
                        historyDiv.innerHTML = "";
                    }
                },500);
            } else{
                p.innerText = result;
                historyDiv.prepend(p);
            }
        });
    },2000); // suspense
}

// Register service worker for offline support
if('serviceWorker' in navigator){
    navigator.serviceWorker.register('sw.js')
    .then(()=> console.log('Service Worker registered'))
    .catch(err => console.log('SW registration failed:', err));
}
</script>
</body>
</html>
