<script setup>

  import { ref, watch } from 'vue'

  const newSubject = ref('');
  const newTask = ref('');
  const cards = ref([]);
  const showModal = ref(false);
  const usedColors = ref([]);
  const pickedCard = ref(null);




  watch(newTask, (newValue) => {
    if (newValue.length > 150) {
      newTask.value = newValue.slice(0,150)
      alert('Too many characters!')
    }
  })

  watch(newSubject, (newValue) => {
    if (newValue.length > 25) {
      newSubject.value = newValue.slice(0,25)
      alert('Too many characters!')
    }
  })

function getRandomColor() {
  const minDifference = 30; // Minimum hue difference between colors
  let newColor;
  let attempts = 0;
  const maxAttempts = 10;

  do {
    const hue = Math.floor(Math.random() * 360);
    const saturation = Math.floor(Math.random() * (100 - 70) + 70);
    const lightness = Math.floor(Math.random() * (80 - 60) + 60);
    newColor = `hsl(${hue}, ${saturation}%, ${lightness}%)`;
    attempts++;

    // Check if color is sufficiently different from used colors
    const isDifferent = usedColors.value.every(usedColor => {
      const usedHue = parseInt(usedColor.match(/hsl\((\d+)/)[1]);
      const newHue = hue;
      const hueDiff = Math.min(Math.abs(newHue - usedHue), 360 - Math.abs(newHue - usedHue));
      return hueDiff >= minDifference;
    });

    if (isDifferent || attempts >= maxAttempts) {
      usedColors.value.push(newColor);
      // Keep only the last 10 colors in history
      if (usedColors.value.length > 10) {
        usedColors.value.shift();
      }
      return newColor;
    }
  } while (attempts < maxAttempts);

  return newColor;
}

const pickRandomCard = () => {
  if (cards.value.length > 0) {
    // Generate a random index
    const randomIndex = Math.floor(Math.random() * cards.value.length);

    // Store the picked card
    pickedCard.value = cards.value[randomIndex];

    // Remove the card from the cards array
    cards.value.splice(randomIndex, 1);
  }
};

  const addTask = () => {
    if(newSubject.value.length === 0) {
      return alert('Add a Subject!')
    } else if (newTask.value.length === 0) {
      return alert ('Not Enough Characters!')
    } else if (cards.value.length >= 14) {
      return alert ('too many cards')
      showModal.value = false;
    }

    cards.value.push({
      subject: newSubject.value,
      text: newTask.value,
      date: new Date().toLocaleDateString(),
      id: Math.floor(Math.random() * 1000),
      color: getRandomColor(),
      completed: false,
    })
    newTask.value = '';
    showModal.value = false;
    newSubject.value = '';

  }

  const removeTask = (id) => {
    cards.value = cards.value.filter(card => card.id !== id)
  }

 
  const closeButton = () => {
    showModal.value = false
    newSubject.value = ''
    newTask.value = ''

    if(pickedCard.value) {
      cards.value.push(pickedCard.value);
      pickedCard.value = null;
    }
  }


</script>

<template>
<body>
  <header class="header">
      <h4 data-text="TODO-LIST">TODO-LIST</h4>
    <div class="header-buttons">
      <button class="addTask" @click="showModal = true">+</button>
      <button class="randomCard" @click="pickRandomCard">Pick a task!</button>
    </div>
  </header>
  <main>   
    <div v-if="showModal" class="overlay">
      <div v-if="showModal" class="modal">
        <!-- <input v-model="newSubject" type="text" placeholder="Enter Subject Here"> -->
        <textarea class="input" v-model="newSubject" type="text" placeholder="Enter Subject Here"></textarea>
        <textarea class="text" v-model="newTask"  cols="25" rows="5" placeholder="What's on your mind?..."></textarea>
        <div class="button-container">
          <button @click="addTask">Add To List</button>
          <button class="close" @click="closeButton" >Close</button>
        </div>
      </div>
    </div>
    <div class="cards-container">
      <div class="card" v-for="card in cards" :key="card.id" :class="{ 'completed-card' : card.completed}">
        <p class="main-subject" :style="{backgroundColor : card.color}">{{ card.subject }}</p>
        <p class="main-text">{{ card.text }}</p>
        <div class="card-footer">
          <div class="footer-buttons">
            <input  type="checkbox" class="completed" v-model="card.completed" >
            <button v-if="card.completed" class="delete" @click="removeTask(card.id)">✖</button>
          </div>  
          <p class="date">{{ card.date }}</p>
        </div>
        
        <span class="slam" v-if="card.completed">COMPLETED!</span>
      </div>
      
    </div>  
    <div v-if="pickedCard" class="picked-card">
    <div class="card" >
      <p class="main-subject" :style="{ backgroundColor: pickedCard.color }">{{ pickedCard.subject }}</p>
      <p class="main-text">{{ pickedCard.text }}</p>
      <p class="date">{{ pickedCard.date }}</p>
    </div>
    <button class="close" @click="closeButton">Put it back</button>
  </div>
  </main>
</body>
</template>

<style>


@keyframes rattle
{
  0% { margin-top: 0; margin-left: 0; }
  10% { margin-top: -5px; margin-left: 0; }
  20% { margin-top: 0; margin-left: -5px; }
  30% { margin-top: 5px; margin-left: 0; }
  40% { margin-top: 0; margin-left: 5px; }
  50% { margin-top: -2px; margin-left: 0; }
  60% { margin-top: 0; margin-left: -2px; }
  70% { margin-top: 2px; margin-left: 0; }
  80% { margin-top: 0; margin-left: 2px; }
  90% { margin-top: -1px; margin-left: 0; }
  100% { margin-top: 0; margin-left: 0; }
}

.completed {
  -webkit-appearance:none;
  appearance:none;
  width:25px;
  height:25px;
  border: 2px solid #333;
  border-radius: 5px;
  background-color: white;
  display:flex;
  justify-content: center;
  align-items: center;
  
  
}

.completed:checked {
  background-color: #4caf4fa2;
  border-color: #408642;
}

/* Add a checkmark using pseudo-element */
.completed:checked::before {
  content: '✓';
  font-size: 16px;
  color: white;
  font-weight: bold;
}

/* Add hover effect */
.completed:hover {
  border-color: #4CAF50;
}

.delete, .completed {
  z-index:4;
}

.delete {
  width: 25px;
  height: 25px;
  cursor: pointer;
  top: 1%;
  right: 1%;
  margin: 0;
  background-color: rgba(255, 73, 73, 0.76);
  padding-bottom: 2px;
  opacity: 0;  /* Start hidden */
  transition: opacity 0.7s ease;  /* Add transition */
  z-index: 4;
}

.button-container {
  width:100%;
  height:100px;
  margin:10px;
  gap:10px;
  padding:0;
  display:flex;
  flex-direction:column;
  align-items:center;
}

.cards-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    height: auto;
    width: auto;
    overflow: hidden;
    justify-content: center; 
    margin: 0 auto;
    padding:0;
}

.card {
  background-color:white;
  color:black;
  width:240px;
  height:325px;
  position:relative;
  border-radius:5px;
  border-bottom-left-radius:5px ;
  border-bottom-right-radius:5px ;
  display:flex;
  justify-content:space-between;
  align-items: center;
  flex-direction:column;
}

 .picked-card {
  display:flex;
  justify-content: center;
  align-items:center;
  position:absolute;
  flex-direction:column;
  top:0;
  width:100vw;
  height:100vh;
  background-color:rgba(0, 0, 0, 0.699);
  z-index:5;
}

.picked-card .card {
  width:calc(400px / 1.3);
  height:425px;
}

.picked-card .close {
  position:absolute;
  bottom:22.3%;
  width:310px;
  height:50px;
  cursor:pointer;
  background-color:rgb(0, 0, 0);
  color :rgb(255, 255, 255);
  margin:0;
}


.overlay {
  position:absolute;
  background-color:rgba(0, 0, 0, 0.623);
  width:100%;
  height:100%;
  display:flex;
  justify-content: center;
  align-items: center;
  z-index:999;
  top:0;
  left:0;
}

main {
  padding:40px 0px;
}

body {
  margin:0;
  padding:0;
  background-color:#333;
}

h4 {
  position: relative;
  display: inline-block;
  font-size: 4.5vw; /* Adjust as needed */
  font-weight: bold;
  text-transform: uppercase;
  animation: glitch 1s linear infinite;
  color:white;
}

@keyframes glitch {
  2%, 64% {
    transform: translate(2px, 0) skew(0deg);
  }
  4%, 60% {
    transform: translate(-2px, 0) skew(0deg);
  }
  62% {
    transform: translate(0, 0) skew(5deg);
  }
}

h4::before,
h4::after {
  content: attr(data-text);
  position: absolute;
  left: 2px;
  width: 100%;
  font-weight:bold;
}

h4::before {
  animation: glitchTop 1s linear infinite;
  clip-path: polygon(0 0, 100% 0, 100% 33%, 0 33%);
}

@keyframes glitchTop {
  2%, 64% {
    transform: translate(2px, -2px);
  }
  4%, 60% {
    transform: translate(-2px, 2px);
  }
  62% {
    transform: translate(13px, -1px) skew(-13deg);
  }
}

h4::after {
  bottom: 0;
  animation: glitchBottom 1.5s linear infinite;
  clip-path: polygon(0 67%, 100% 67%, 100% 100%, 0 100%);
}

@keyframes glitchBottom {
  2%, 64% {
    transform: translate(-2px, 0);
  }
  4%, 60% {
    transform: translate(-2px, 0);
  }
  62% {
    transform: translate(-22px, 5px) skew(21deg);
  }
}


.header {
  display:flex;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  padding:3vw;
  padding-top:1vw;
  position:relative;
}

.header-name {
  position: relative;
  display: inline-block;
  font-size: 2rem; /* Adjust as needed */
  font-weight: bold;
  text-transform: uppercase;
  animation: glitch 1s linear infinite;
}

@keyframes glitch {
  2%, 64% {
    transform: translate(2px, 0) skew(0deg);
  }
  4%, 60% {
    transform: translate(-2px, 0) skew(0deg);
  }
  62% {
    transform: translate(0, 0) skew(5deg);
  }
}

.header-name::before,
.header-name::after {
  content: attr(data-text);
  position: absolute;
  left: 0;
  width: 100%;
}

.header-name::before {
  top: 0;
  color: #ff0000; /* Red glitch layer */
  animation: glitchTop 1s linear infinite;
  clip-path: polygon(0 0, 100% 0, 100% 33%, 0 33%);
}

@keyframes glitchTop {
  2%, 64% {
    transform: translate(2px, -2px);
  }
  4%, 60% {
    transform: translate(-2px, 2px);
  }
  62% {
    transform: translate(13px, -1px) skew(-13deg);
  }
}

.header-name::after {
  bottom: 0;
  color: #0000ff; /* Blue glitch layer */
  animation: glitchBottom 1.5s linear infinite;
  clip-path: polygon(0 67%, 100% 67%, 100% 100%, 0 100%);
}

@keyframes glitchBottom {
  2%, 64% {
    transform: translate(-2px, 0);
  }
  4%, 60% {
    transform: translate(-2px, 0);
  }
  62% {
    transform: translate(-22px, 5px) skew(21deg);
  }
}


.header-buttons {
  display:flex;
  justify-content:center;
  align-items:center;
}

.addTask {
  display:flex;
  justify-content:center;
  align-items:center;
  width:60px;
  height:60px;
  cursor:pointer;
  background-color:rgb(94, 94, 94);
  border-radius:50%;
  color:white;
  font-size: 30px;
  top:10%;
  right:10%;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
}

.addTask:hover {
  background-color:rgb(143, 143, 143);
  transition:background-color .3s ease-in-out;
}

.randomCard {
  width:125px;
  height:50px;
  cursor:pointer;
  background-color:rgb(0, 0, 0);
  color:white;
  top:10%;
  right:15%;
  display:none;
}


.card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
  background-color: rgba(0, 0, 0, 0); /* Initial state */
  opacity: 0; /* Initial state */
  transition: opacity 0.5s cubic-bezier(0.4, 0, 0.2, 1); /* Match duration with .slam */
  pointer-events: none;
}

.completed-card::before {
  
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
  background-color: rgba(0, 0, 0, 0.747);
  opacity: 1;
  transition: opacity 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
  border-radius:4px;
}

.completed-card .slam {
  animation: 
  slam 0.5s ease-in, /* Match duration with background transition */
  rattle 0.2s ease-in-out 0.5s; /* Adjust delay to start after slam */
  transition: ease-in-out 0.3s;
  color: #be0000;
  display: block;
  font-family: "Source Sans Pro", sans-serif;
  font-size: 2em;
  font-weight: 900;
  position: absolute;
  text-align: center;
  text-transform: uppercase;
  transform: rotate(-15deg);
  z-index:3;
  top: 40%;
}

@keyframes slam {
  0% {
    transform: scale(10, 10) rotate(-15deg);
    opacity: 0;
  }
  
  40% {
    opacity: 0;
  }
  
  100% {
    transform: scale(1, 1) rotate(-15deg);
    opacity: 1;
  }
}

/* Add this new style to make the delete button visible when card is completed */
.completed-card .delete {
  opacity: 1;
}


.card-footer {
  width:100%;
  height:10%;
  display:flex;
  justify-content:space-between;
  padding: 0 5px;
}

.footer-buttons {
  display:flex;
  width:100%;
  height:100%;
  gap:5px;
}

.date {
  font-family:monospace,sans-serif;
  font-size:18px;
}

.main-subject {
  font-weight:bold;
  font-size:1.5em;
  letter-spacing:.5px;
  font-family:'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
  padding:5px;
  width:100%;
  height:auto;
  border-radius:5px;
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  word-wrap:break-word;
  white-space:normal;
  overflow-wrap:break-word;
  text-align:center;
}

.main-text {
  width: 100%;
  height:100%;
  overflow-wrap:break-word;
  line-height: 1.8;
  box-sizing: border-box;
  font-size:1em;
  overflow: hidden;  /* Hide overflow instead of scrolling */
  text-overflow: ellipsis;
  text-align:center;
  align-content: center;
  padding:0px 10px 0px 10px;
  
}

.container2 {
  display:flex;
  flex-direction:column;
}


button {
background-color:black;
color:white;
outline:none;
border:none;
width:85px;
height:35px;
border-radius:5px;
margin:5px;
}

.input {
  width:100%;
  height:20%;
  font-size:1.7em;
  outline:none;
  border:none;
  resize:none;
  font-family:'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
  background-color:rgb(156, 156, 156);
  font-weight:bold;
  caret-color: rgba(255, 255, 255, 0.664);
  border-radius:5px;
  border-bottom-left-radius: 0%;
  border-bottom-right-radius: 0%;
  overflow-wrap: break-word;
  white-space:normal;
  text-overflow:ellipsis;
  text-align:center;
  box-sizing:border-box;
  padding:0px 15px 0px 15px;
  overflow:hidden;
  line-height:1.3;
  align-content:center;
}


.input::placeholder {
  color:rgba(0, 0, 0, 0.555);
}

.text {
width:100%;
height:100%;
padding:0px 20px;
outline:none;
border:none;
resize:none;
font-size:1.3em;
font-weight:bold;
text-align:center;
caret-color:white;
background-color:rgb(85, 85, 85);
border-radius:5px;
border-top-left-radius: 0%;
border-top-right-radius: 0%;
line-height:1.7;
align-content:center;
}

.text::placeholder {
  color:rgba(255, 255, 255, 0.445);
}

.modal {
  width:360px;
  height:65%;
  background-color:white;
  border-radius:10px;
  padding:15px 20px 0px 20px;
  display:flex;
  flex-direction:column;
  justify-content: center;
  align-items: center;
}

.modal button {
  font-size:20px;
  width:100%;
  height:50px;
  background-color:rgb(0, 0, 0);
  border:none;
  color:white;
  cursor:pointer; 
  margin:0;
}

@media (max-width:499px) {
  .card {
    width:calc(225px / 1.3) ;
    height:285px ;
  }
  .completed-card .slam {
    font-size: 1.5em;
  }
  .modal {
    width:300px;
    height:65%;
  }
  textarea {
    width:100%;
    height:100%;
  }
  input {
    width:100%;
    
  }
  .modal button {
   width:100%;
  }
  .button-container {
    flex-direction:row;
    justify-content:space-evenly;
  }
  .main-subject {
    font-size:1.2em;
  }
  .main-text {
    font-size:0.9em;
  }
  .button-container {
    height:60px;
  }
}


</style>