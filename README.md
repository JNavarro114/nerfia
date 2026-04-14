<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Theater Reservation</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-900 text-white flex flex-col items-center p-6">

<h1 class="text-3xl font-bold mb-6">🎭 Teatro Don Bosco</h1>

<!-- AUTH -->
<div id="auth" class="mb-6">
  <input id="email" type="email" placeholder="Email" class="p-2 text-black">
  <input id="password" type="password" placeholder="Password" class="p-2 text-black">
  <button onclick="signUp()" class="bg-green-500 px-4 py-2">Sign Up</button>
  <button onclick="login()" class="bg-blue-500 px-4 py-2">Login</button>
</div>

<!-- SEATS -->
<div id="app" class="hidden">
  <h2 class="text-xl mb-4">Select your seat</h2>
  <div id="seats" class="grid grid-cols-10 gap-2"></div>

  <button onclick="confirmBooking()" class="mt-6 bg-purple-600 px-6 py-2">
    Confirm & Pay ($10)
  </button>

  <div id="ticket" class="mt-6"></div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword } 
from "https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js";
import { getFirestore, doc, setDoc, getDoc } 
from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

/* 🔥 REPLACE WITH YOUR FIREBASE CONFIG */
const firebaseConfig = {
  apiKey: "YOUR_KEY",
  authDomain: "YOUR_DOMAIN",
  projectId: "YOUR_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

let currentUser = null;
let selectedSeat = null;

const seatsDiv = document.getElementById("seats");

/* 🎟️ Generate Seats */
function renderSeats() {
  seatsDiv.innerHTML = "";
  for (let i = 1; i <= 50; i++) {
    const seat = document.createElement("div");
    seat.innerText = i;
    seat.className = "p-3 text-center cursor-pointer bg-gray-400";

    seat.onclick = () => {
      selectedSeat = i;
      renderSeats();
    };

    if (selectedSeat === i) {
      seat.className = "p-3 bg-green-500";
    }

    seatsDiv.appendChild(seat);
  }
}

renderSeats();

/* 🔐 AUTH */
window.signUp = async () => {
  const email = document.getElementById("email").value;
  const pass = document.getElementById("password").value;
  await createUserWithEmailAndPassword(auth, email, pass);
  alert("User created!");
};

window.login = async () => {
  const email = document.getElementById("email").value;
  const pass = document.getElementById("password").value;

  const res = await signInWithEmailAndPassword(auth, email, pass);
  currentUser = res.user;

  document.getElementById("auth").classList.add("hidden");
  document.getElementById("app").classList.remove("hidden");
};

/* 💳 Booking */
window.confirmBooking = async () => {
  if (!selectedSeat) {
    alert("Select a seat!");
    return;
  }

  const seatRef = doc(db, "seats", String(selectedSeat));
  const seatSnap = await getDoc(seatRef);

  if (seatSnap.exists()) {
    alert("Seat already taken!");
    return;
  }

  // Simulate payment
  alert("Payment successful!");

  await setDoc(seatRef, {
    user: currentUser.email,
    seat: selectedSeat
  });

  document.getElementById("ticket").innerHTML = `
    🎟️ <h3 class="text-xl">Your Ticket</h3>
    Seat: ${selectedSeat} <br>
    User: ${currentUser.email}
  `;

  selectedSeat = null;
  renderSeats();
};
</script>

</body>
</html>
