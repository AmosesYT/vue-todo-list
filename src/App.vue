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
    if (newValue.length > 16) {
      newSubject.value = newValue.slice(0,16)
      alert('Too many characters')
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
    <div class="header-name">
      <h4>TODO-LIST :)</h4>
    </div>
    <div class="header-buttons">
      <button class="addTask" @click="showModal = true">+</button>
      <button class="randomCard" @click="pickRandomCard">Pick a task!</button>
    </div>
  </header>
  <main>   
    <div v-if="showModal" class="overlay">
      <div v-if="showModal" class="modal">
        <input v-model="newSubject" type="text" placeholder="Enter Subject Here">
        <textarea v-model="newTask"  cols="25" rows="5" placeholder="What's on your mind?"></textarea>
        <button @click="addTask">Add To List</button>
        <button class="close" @click="closeButton" >Close</button>
      </div>
    </div>
    <div class="cards-container">
      <div class="card" v-for="card in cards" :key="card.id" :class="{ 'completed-card' : card.completed}">
        <p class="main-subject" :style="{backgroundColor : card.color}">{{ card.subject }}</p>
        <p class="main-text">{{ card.text }}</p>
        <p class="date">{{ card.date }}</p>
        <input  type="checkbox" class="completed" v-model="card.completed" >
        <button v-if="card.completed" class="delete" @click="removeTask(card.id)">x</button>
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
  -webkit-appearance:  none;
  appearance:none;
  position:absolute;
  bottom:2%;
  left:2%;
  width:25px;
  height:25px;
  margin:0;
  padding:0;
  box-sizing:border-box;
  border: 2px solid #333;
  border-radius: 5px;
  background-color: white;
}

.completed:checked {
  background-color: #4caf4fa2;
  border-color: #408642;
}

/* Add a checkmark using pseudo-element */
.completed:checked::before {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
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
  position: absolute;
  width: 15px;
  height: 15px;
  cursor: pointer;
  top: 1%;
  right: 1%;
  margin: 0;
  background-color: rgba(255, 73, 73, 0.76);
  display: flex;
  justify-content: center;
  align-items: center;
  padding-bottom: 2px;
  opacity: 0;  /* Start hidden */
  transition: opacity 0.7s ease;  /* Add transition */
  z-index: 4;
}

.cards-container {
  display:flex;
  flex-direction:row;
  flex-wrap:wrap;
  gap:20px;
  height:auto;
  width:100%;
  overflow:hidden;
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
  width:100%;
  height:100%;
  display:flex;
  flex-direction:column;
  padding:45px;
}

body {
  background-color: rgb(77, 77, 77);
}

h4 {
  font-size:4vw;
  color:rgb(255, 255, 255);
}

.header {
  display:flex;
  flex-direction:row;
  justify-content:space-between;
  padding:3vw;
  padding-top:1vw;
  position:relative;
}

.header-name {
  display:flex;
}

.header-buttons {
  display:flex;
  justify-content:center;
  align-items:center;
}

.addTask {
  border:none;
  padding: 10px;
  width:50px;
  height:50px;
  cursor:pointer;
  background-color:rgb(0, 0, 0);
  border-radius:100%;
  color:white;
  font-size: 30px;
  top:10%;
  right:10%;
}

.randomCard {
  border:none;
  padding: 10px;
  width:125px;
  height:50px;
  cursor:pointer;
  background-color:rgb(0, 0, 0);
  color:white;
  top:10%;
  right:15%;
}

.card {
  background-color:white;
  color:black;
  width:calc(325px / 1.3);
  height:325px;
  position:relative;
  border-radius:5px;
  display:flex;
  flex-wrap:wrap;
  justify-content:center;
  align-items: center;
  flex-direction:column;
  overflow:hidden;
  
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
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
  background-color: rgba(0, 0, 0, 0.8);
  opacity: 1;
  transition: opacity 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
}

.completed-card .slam {
  animation: 
    slam 0.5s ease-in, /* Match duration with background transition */
    rattle 0.2s ease-in-out 0.5s; /* Adjust delay to start after slam */
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



.main-subject {
  font-weight:bold;
  font-size:24px !important;
  position:absolute;
  top:0;
  width:100%;
  height:65px;
  padding:10px 0px;
  margin-bottom:0;
  border-radius:5px;
  border-bottom-left-radius: 0%;
  border-bottom-right-radius: 0%;
  display:flex;
  justify-content:center;
  align-items:center;
  overflow: hidden;
  box-sizing: border-box;
  line-height:1;
  text-overflow:ellipsis;
  white-space:nowrap;
}

.main-text {
  width: 85%;
  word-wrap: break-word;
  margin: 0;
  max-height: 150px;  /* Fixed height instead of percentage */
  line-height: 1.5;
  padding:5px;
  position: relative;    /* Adjust based on your subject header height */
  box-sizing: border-box;
  font-size: 16px;  /* Fixed font size */
  overflow: hidden;  /* Hide overflow instead of scrolling */
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  text-align:center;
}
.date {
  position:absolute;
  bottom:5px;
  right:5px;
  margin:0;
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

input {
width:325px;
outline:none;
border:none;
background-color:rgb(156, 156, 156);
text-align:center;
font-weight:bold;
font-size:32px;
caret-color: rgba(255, 255, 255, 0.664);
border-bottom:solid rgb(85, 85, 85) 1px;
border-bottom-width: 35px;
padding:10px 20px;
border-radius:5px;
border-bottom-left-radius: 0%;
border-bottom-right-radius: 0%;
overflow:hidden;
}


input::placeholder {
color:rgba(0, 0, 0, 0.555);
}

textarea {
width:325px;
aspect-ratio: 1 /1 ;
outline:none;
border:none;
resize:none;
font-size:22px;
font-weight:bold;
font-family:curisve;
display:flex;
caret-color:white;
background-color:rgb(85, 85, 85);
padding:10px 20px;
border-radius:5px;
border-top-left-radius: 0%;
border-top-right-radius: 0%;
line-height:1.7;
}

textarea::placeholder {
position:absolute;
top:25%;
left:25%;
transform: translate(-10%,-25%);
color:white;
pointer-events:none;
}

.modal {
  width:360px;
  background-color:white;
  border-radius:10px;
  padding:15px 35px;
  display:flex;
  flex-direction:column;
  justify-content: center;
  align-items: center;

}

.modal button {
  font-size:20px;
  width:112%;
  height:50px;
  background-color:rgb(0, 0, 0);
  border:none;
  color:white;
  cursor:pointer;
  margin-top:15px;
}

.modal .close  {
  background-color:rgb(0, 0, 0);
  margin-top:7px;
}


</style>