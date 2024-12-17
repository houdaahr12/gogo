<template>
  <div>
    <!-- Form Container -->
    <transition name="fade" @before-enter="beforeEnter" @enter="enter" @leave="leave">
      <div v-if="formVisible" class="modal-container" @click="handleBackgroundClick">
        <!-- Modal Window -->
        <div class="modal-content" @click.stop>
          <!-- Form Header -->
          <h2 class="form-title">Créer une Nouvelle Tâche</h2>

          <!-- Form -->
          <form @submit.prevent="addTask">
            <!-- Task Name -->
            <div class="input-group">
              <label for="taskName">Nom de la tâche *</label>
              <input
                type="text"
                id="taskName"
                v-model="task.name"
                placeholder="Entrez le nom de la tâche"
              />
              <span v-if="!task.name && isSubmitted" class="error">Ce champ est obligatoire.</span>
            </div>

            <!-- Category Selection -->
            <div class="input-group">
              <label>Catégorie *</label>
              <div class="categories">
                <div
                  v-for="category in categories"
                  :key="category.name"
                  :class="['category-item', { selected: task.category === category.name }, { 'disabled-category': categoryDisabled }]"
                  @click="!categoryDisabled && (task.category = category.name)"
                  :style="categoryDisabled ? { pointerEvents: 'none', opacity: 0.5 } : {}"
                  :disabled="categoryDisabled"
                >
                  <i :class="category.icon"></i> {{ category.name }}
                </div>
              </div>
              <span v-if="!task.category && isSubmitted" class="error">Veuillez choisir une catégorie.</span>
            </div>

            <!-- Date -->
            <div class="input-group">
              <label for="date">Date *</label>
              <input type="date" id="date" v-model="task.due_date" :disabled="isDateDisabled"/>
              <span v-if="!task.due_date && isSubmitted && !isDateDisabled" class="error">Veuillez sélectionner une date.</span>
            </div>

            <!-- Time -->
            <div class="input-group">
              <label for="time">Heure (Optionnel)</label>
              <input type="time" id="time" v-model="task.due_time" />
            </div>

            <!-- Importance Level -->
            <div class="input-group">
              <label>Niveau d'importance</label>
              <div class="importance-levels">
                <div
                  v-for="level in importanceLevels"
                  :key="level.value"
                  :class="['importance-level', { selected: task.importance === level.value }]"
                  @click="selectImportance(level.value)"
                >
                  {{ level.label }}
                </div>
              </div>
            </div>

            <!-- Action Buttons -->
            <div class="button-group">
              <button type="button" class="btn btn-cancel" @click="resetForm">Annuler</button>
              <button type="submit" class="btn btn-add">Ajouter</button>
            </div>
          </form>
        </div>

        <!-- Popup Modal -->
        <div v-if="showPopup" class="popup-overlay">
          <div class="popup-content">
            <h3>Bravo ! 🎉</h3>
            <p>Tâche ajoutée avec succès</p>
            <div class="popup-actions">
              <router-link to="/dashboard" class="btn btn-secondary">Revenir au tableau de bord..</router-link>
              <router-link to="/cat" class="btn see">Voir ma tâche</router-link>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      task: {
        name: "",
        category: this.$route.query.categoryName || "",
        importance: "low",
        due_date: "",
        due_time: "",
        status: this.$route.query.status || 'pas commence', 
      },
      categories: [
        { name: "Travail", icon: "fas fa-briefcase" },
        { name: "Etude", icon: "fas fa-book" },
        { name: "Maison", icon: "fas fa-home" },
        { name: "Personnel", icon: "fas fa-user" },
        { name: "Loisirs", icon: "fas fa-gamepad" },
        { name: "Autre", icon: "fas fa-exclamation-circle" },
      ],
      categoryDisabled: false,
      isDateDisabled: false,
      importanceLevels: [
        { value: "low", label: "Moins important" },
        { value: "medium", label: "Important" },
        { value: "high", label: "Urgent" },
      ],
      progressBarWidth: 33,
      progressBarColor: "#9b59b6",
      importanceLabel: "Moins Important",
      formVisible: false,
      isSubmitted: false,
      showPopup: false,
    };
  },
  created() {
    // Vérifier si une date est présente dans l'URL
    const todayDate = this.$route.query.date;
    if (todayDate) {
      this.task.due_date = todayDate; // Pré-remplir la date
      this.isDateDisabled = true; // Désactiver le champ de la date
    }
  },
  mounted() {
    this.formVisible = true;

    // Vérifier la catégorie et la désactiver si présente dans l'URL
    const urlParams = new URLSearchParams(window.location.search);
    const categoryName = urlParams.get('categoryName');
    if (categoryName) {
      this.task.category = categoryName;
      this.categoryDisabled = true; // Désactiver la sélection de catégorie
    }
  },
  methods: {
    addTask() {
      this.isSubmitted = true;

      // Valider les champs obligatoires, sauf la date si elle est pré-remplie
      if (!this.task.name || !this.task.category || (!this.task.due_date && !this.isDateDisabled)) {
        alert("Veuillez remplir tous les champs obligatoires.");
        return;
      }

      // Mapper les niveaux d'importance aux valeurs backend
      const importanceMap = {
        low: "moins important",
        medium: "important",
        high: "urgent",
      };

      const formattedTask = {
        task_name: this.task.name,
        category: this.task.category,
        due_date: this.task.due_date,
        due_time: this.task.due_time || null,
        priority: importanceMap[this.task.importance],
        status: this.task.status,
      };

      // Envoyer les données au backend
      axios
        .post("http://localhost:3000/api/tasks-add", formattedTask)
        .then(() => {
          this.showPopup = true;
        })
        .catch((err) => {
          console.error("Erreur lors de l'ajout de la tâche:", err);
          alert("Une erreur s'est produite lors de l'ajout de la tâche.");
        });
    },
    selectImportance(level) {
      this.task.importance = level;
      const settings = {
        low: { width: 33, color: "#f1c40f", label: "Moins Important" },
        medium: { width: 70, color: "#2ecc71", label: "Important" },
        high: { width: 100, color: "#e74c3c", label: "Urgent" },
      };
      this.progressBarWidth = settings[level].width;
      this.progressBarColor = settings[level].color;
      this.importanceLabel = settings[level].label;
    },
    resetForm() {
      this.task = {
        name: "",
        category: "",
        due_date: "",
        due_time: "",
        importance: "low",
      };
      this.isSubmitted = false;
      this.$router.push('/dashboard');
    },
  },
};
</script>



<style scoped>
body {
  font-family: Arial, sans-serif;
  background-color: #4f195f;
  color: #f5f5f5;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.modal-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(146, 144, 148, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding-top: 60px; /* Ajoute un padding-top pour compenser la sidebar */
}

.modal-content {
  background-color: #3d0f50;
  padding: 40px;
  border-radius: 10px;
  width: 90%;
  max-width: 900px;
  box-shadow: 0 4px 10px rgba(107, 19, 115, 0.717);
  color: #ffffff;
  overflow-y: auto;
  z-index: 1001; /* S'assure que le modal est au-dessus */
}

h2.form-title {
  text-align: center;
  font-size: 28px;
  margin-bottom: 20px;
}

.input-group {
  margin-bottom: 15px;
}

.input-group label {
  font-size: 16px;
  margin-bottom: 5px;
  display: block;
}

.input-group input {
  width: 100%;
  padding: 10px;
  font-size: 14px;
  border-radius: 5px;
  border: 1px solid #ddd;
  background: #c1bfbf;
  color: #000000;
}

.categories,
.importance-levels {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
}

.category-item,
.importance-level {
  flex: 1 1 auto;
  padding: 10px;
  text-align: center;
  border-radius: 5px;
  background-color: #5a2b72;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

.category-item:hover,
.importance-level:hover {
  background-color: #7f2f96;
}

.category-item.selected,
.importance-level.selected {
  background-color: #712092;
}

.progress-bar {
  background-color: #ddd;
  height: 10px;
  width: 100%;
  border-radius: 5px;
  margin: 10px 0;
}

.progress {
  height: 100%;
  border-radius: 5px;
  background-color: #5a2b72; /* Changez la couleur de la barre ici */
  width: 100%; /* Remplissage complet de la barre sans texte */
}

.button-group {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  margin-top: 20px;
}

.btn {
  flex: 1;
  padding: 10px;
  text-align: center;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.btn-cancel {
  background-color: #a71302;
  color: white;
  transition: transform 0.2s ease;
}

.btn-cancel:hover {
  background-color: #501610;
  transform: translateY(-3px) scale(1.05);
}

.btn-add {
  background-color: #681e85;
  color: white;
  transition: transform 0.2s ease;
}

.btn-add:hover {
  background-color: #8122a7;
  transform: translateY(-3px) scale(1.05);
}


/* Responsivity for small screens */
@media (max-width: 768px) {
  .modal-content {
    padding: 20px;
  }

  h2.form-title {
    font-size: 24px;
  }

  /* Categories: 3 items per row */
  .categories {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }

  .category-item {
    flex: 1 1 100%;
  }

  /* Importance Levels: each on a new line */
  .importance-levels {
    display: flex;
    flex-direction: line;
    gap: 5px;
  }

  .importance-level {
    flex: 1 1 100%; 
    padding: 15px; 
    font-size: 14px; 
  }

  .button-group {
    flex-wrap: wrap;
    gap: 10px;
  }

  .btn {
    flex: 1 1 48%;
  }
}

@media (max-width: 480px) {
  .modal-content {
    padding: 10px;
  }

  h2.form-title {
    font-size: 20px;
    margin-bottom: 15px;
  }

  .input-group input {
    font-size: 14px;
    padding: 8px;
  }

  .categories {
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;
  }

  .category-item {
    flex: 1 1 100%; 
  }

  .importance-levels {
    flex-direction: column;
    gap: 8px;
  }

  .importance-level {
    flex: 1 1 100%;
    padding: 10px; 
    font-size: 14px;
  }

  .button-group {
    flex-direction: column;
    gap: 10px;
  }

  .btn {
    padding: 8px 12px;
    font-size: 14px;
  }
}
</style>
