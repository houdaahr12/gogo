<template>
  <div class="page-container">
    <main class="main-content">
      <!-- Calendrier centré -->
      <section :class="['calendar-container', darkMode ? 'dark' : 'light']">
        <div class="calendar-header">
          <button @click="prevMonth" class="nav-button">&lt;</button>
          <span class="month-label">{{ monthNames[currentMonth] }} {{ currentYear }}</span>
          <button @click="nextMonth" class="nav-button">&gt;</button>
          
        </div>

        <div class="calendar-grid">
          <div class="day-name" v-for="day in dayNames" :key="day">{{ day }}</div>
          <div
            class="day"
            v-for="day in daysInMonth"
            :key="day.date"
            :class="{
              'is-today': day.isToday,
              'is-empty': !day.date,
              'has-task': day.date && hasTasks(day.date),
            }"
            @click="day.date && openModal(day.date)"
          >
            {{ day.day }}
            <div v-if="hasTasks(day.date)" class="task-indicator"></div>
          </div>

        </div>

        <!-- Fenêtre modale -->
      <div v-if="modalVisible" class="modal-overlay" @click.self="closeModal">
        <div class="modal-content">
          <h2>Tâches pour le {{ formatSelectedDate(selectedDate) }}</h2>
          <ul v-if="tasks.length > 0">
            <li v-for="(task, index) in tasks" :key="index">
              📌 <strong>{{ task.title }}</strong>
              <br />
              🗓️ Due: {{ formatSelectedDate(task.start) }}
            </li>
          </ul>
          <p v-else>Aucune tâche pour cette date.</p>
          <button @click="goToAddView" class="add-task-button">Ajouter une Tâche</button>
          <button @click="closeModal" class="close-modal-button">Fermer</button>
        </div>
      </div>

      </section>
    </main>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      currentYear: new Date().getFullYear(),
      currentMonth: new Date().getMonth(),
      selectedDate: null,
      tasks: [],
      modalVisible: false, // État pour la fenêtre modale
      darkMode: false,
      dayNames: ["Dim", "Lun", "Mar", "Mer", "Jeu", "Ven", "Sam"],
    };
  },
  computed: {
    monthNames() {
      return [
        "Janvier", "Février", "Mars", "Avril", "Mai", "Juin",
        "Juillet", "Août", "Septembre", "Octobre", "Novembre", "Décembre",
      ];
    },
    daysInMonth() {
      const days = [];
      const firstDay = new Date(this.currentYear, this.currentMonth, 1).getDay();
      const lastDate = new Date(this.currentYear, this.currentMonth + 1, 0).getDate();

      for (let i = 0; i < firstDay; i++) {
        days.push({ day: "", date: null });
      }

      for (let i = 1; i <= lastDate; i++) {
        const date = new Date(this.currentYear, this.currentMonth, i);
        days.push({
          day: i,
          date: this.formatDate(date),
          isToday: this.isToday(date),
        });
      }

      return days;
    },
  },
  methods: {
    isToday(date) {
      const today = new Date();
      return (
        date.getFullYear() === today.getFullYear() &&
        date.getMonth() === today.getMonth() &&
        date.getDate() === today.getDate()
      );
    },
    formatDate(date) {
      if (!(date instanceof Date)) {
        date = new Date(date); // Safely convert string to Date object
      }
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, "0");
      const day = String(date.getDate()).padStart(2, "0");
      return `${year}-${month}-${day}`;
    },

    formatSelectedDate(date) {
  if (!date) return "Date invalide";
  const [fullDate] = date.split("T"); // Only use the full date part
  const [year, month, day] = fullDate.split("-");
  const formattedDate = `${day}/${month}/${year}`;
  return formattedDate;
},



      

    prevMonth() {
      if (this.currentMonth === 0) {
        this.currentMonth = 11;
        this.currentYear--;
      } else {
        this.currentMonth--;
      }
    },
    nextMonth() {
      if (this.currentMonth === 11) {
        this.currentMonth = 0;
        this.currentYear++;
      } else {
        this.currentMonth++;
      }
    },
    async loadTasksForDate(date) {
  try {
    const formattedDate = this.formatDate(new Date(date));
    const response = await axios.get(`http://localhost:3000/api/tasks-for-calendar/${formattedDate}`);

    if (response.status !== 200) {
      throw new Error('Échec de la récupération des tâches');
    }

    // Log the response data to see its structure
    console.log('Tasks for date:', formattedDate, response.data);

    this.tasks = response.data.length > 0 ? response.data : [];
  } catch (error) {
    console.error('Erreur lors de la récupération des tâches:', error);
    this.tasks = [];
  }
},


    openModal(date) {
      if (!date) return;
      this.selectedDate = date;
      this.loadTasksForDate(date);
      this.modalVisible = true;
    },

    closeModal() {
      this.modalVisible = false;
    },
    goToAddView() {
      if (this.selectedDate) {
        this.$router.push({
          name: "AddView",
          query: { date: this.selectedDate },
        });
      } else {
        alert("Veuillez sélectionner une date avant d'ajouter une tâche !");
      }
    },
    toggleDarkMode() {
      this.darkMode = !this.darkMode;
    },
    hasTasks(date) {
      return this.tasks.some(task => task.due_date && task.due_date.startsWith(date));  // Ensure due_date exists
}


  },
};
</script>


<style scoped>
.page-container {
  color: white;
  font-family: 'Poppins', sans-serif;
  display: flex;
  justify-content: center;

}

.calendar-container {
  color: #333;
  border-radius: 25px;
  padding: 25px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  width: 45vw;
  margin-top: 25px;
  

}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.month-label {
  font-size: 2rem;
  font-weight: bold;
  color: #3c096c;
}

.nav-button {
  background: none;
  border: 3px solid #3c096c;
  color: #3c096c;
  border-radius: 50%;
  padding: 15px 25px;
  font-size: 1.8rem;
  font-weight: bold;
  transition: all 0.3s ease, box-shadow 0.2s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-button:hover {
  background: #5a189a;
  color: #fff;
  border-color: transparent;
  transform: translateY(-3px);
  box-shadow: 0 8px 15px #6a11cb66;
}

.nav-button:active {
  transform: translateY(0);
  box-shadow: 0 4px 8px rgba(106, 17, 203, 0.2);
}


.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 15px;
  width: 100%;
  box-sizing: border-box;
}

.day {
  padding: 25px;
  text-align: center;
  border-radius: 15px;
  background: rgba(106, 17, 203, 0.15);
  cursor: pointer;
  position: relative;
  transition: transform 0.2s, background 0.3s;
  font-size: 1.8rem;
}

.day:hover {
  transform: scale(1.1);
  background: rgba(106, 17, 203, 0.35);
}

.has-task .task-indicator {
  position: absolute;
  bottom: 5px; 
  right: 5px;  
  width: 12px; 
  height: 12px; 
  background: #ff4081; 
  border-radius: 50%; 
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3); 
  animation: pulse 1.5s infinite; 
}

@keyframes pulse {
  0% {
    transform: scale(0.8);
    opacity: 0.7;
  }
  50% {
    transform: scale(1);
    opacity: 1;
  }
  100% {
    transform: scale(0.8);
    opacity: 0.7;
  }
}

.is-today {
  border: 3px solid #6a11cb66; 
  font-weight: bold;
  background-color: #6a11cb66; 
  color: white; 
}



.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: rgb(139, 71, 165);
  padding: 40px;
  border-radius: 15px;
  width: 600px;
  box-shadow: 0 10px 20px rgba(185, 179, 221, 0.9);
}

.add-task-button,
.close-modal-button {
  background: #462a65;
  color: white;
  padding: 15px 30px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
  margin-top: 20px;
  font-size: 1.2rem;
}

.add-task-button:hover,
.close-modal-button:hover {
  background: #542f5f;
}




</style>
