<template>
  <div>
    <!-- Titre avec flèches alignées au centre -->
    <div class="chart-header">
      <button @click="goToPreviousWeek" class="arrow-button">
        <i class="fas fa-arrow-left"></i>
      </button>
      <span class="chart-title">Tâches complétées - Semaine {{ selectedWeek }} ({{ currentYear }})</span>
      <button @click="goToNextWeek" class="arrow-button">
        <i class="fas fa-arrow-right"></i>
      </button>
    </div>

    <!-- Graphique -->
    <div id="weeklyTasksChart" style="width: 100%; height: 400px;"></div>
    <p v-if="!chartData || chartData.length === 0">
      Aucune donnée à afficher pour cette semaine.
    </p>
  </div>
</template>

<script>
import * as echarts from "echarts";

// Fonction pour obtenir la semaine courante
function getCurrentWeekNumber() {
  const currentDate = new Date();
  const startOfYear = new Date(currentDate.getFullYear(), 0, 1);
  const pastDaysOfYear = (currentDate - startOfYear + 86400000) / 86400000;
  return Math.ceil(pastDaysOfYear / 7);
}

export default {
  name: "LineChart",
  props: {
    chartData: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      chart: null, // Instance ECharts
      selectedWeek: null, // Semaine actuellement sélectionnée
      currentYear: null, // Année actuelle
      maxWeeks: 52, // Nombre total de semaines dans une année
    };
  },
  mounted() {
    // Calculer la semaine et l'année courantes
    this.currentYear = new Date().getFullYear();
    this.selectedWeek = getCurrentWeekNumber();

    // Initialiser le graphique
    this.createChart();

    // Mettre à jour les données pour la semaine courante
    this.updateWeekData();
  },
  methods: {
    // Aller à la semaine précédente
    goToPreviousWeek() {
      if (this.selectedWeek > 1) {
        this.selectedWeek -= 1;
      } else if (this.selectedWeek === 1) {
        this.selectedWeek = this.maxWeeks;
        this.currentYear -= 1;
      }
      this.updateWeekData();
    },

    // Aller à la semaine suivante
    goToNextWeek() {
      if (this.selectedWeek < this.maxWeeks) {
        this.selectedWeek += 1;
      } else if (this.selectedWeek === this.maxWeeks) {
        this.selectedWeek = 1;
        this.currentYear += 1;
      }
      this.updateWeekData();
    },

    // Crée le graphique
    createChart() {
      const chartDom = document.getElementById("weeklyTasksChart");
      this.chart = echarts.init(chartDom);

      // Charger les options avec des données initiales vides
      const options = this.generateChartOptions([], this.selectedWeek, this.currentYear);
      this.chart.setOption(options);
    },

    // Met à jour les données pour la semaine sélectionnée
    updateWeekData() {
      // Filtrer les données pour la semaine et l'année sélectionnées
      const filteredData = this.chartData.filter(
        (entry) =>
          entry.week_number === this.selectedWeek && entry.year === this.currentYear
      );

      // Générer les options du graphique
      const options = this.generateChartOptions(filteredData, this.selectedWeek, this.currentYear);

      // Mettre à jour le graphique
      if (this.chart) {
        this.chart.setOption(options);
      }
    },

    // Génère les options pour ECharts
    generateChartOptions(data, weekNumber, year) {
      const days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
      const weekData = {
        Monday: 0,
        Tuesday: 0,
        Wednesday: 0,
        Thursday: 0,
        Friday: 0,
        Saturday: 0,
        Sunday: 0,
      };

      // Remplir weekData avec les données disponibles
      data.forEach((entry) => {
        if (entry.day_of_week && entry.completed_tasks !== undefined) {
          weekData[entry.day_of_week] = entry.completed_tasks;
        }
      });

      return {
        xAxis: {
          type: "category",
          data: days,
        },
        yAxis: {
          type: "value",
        },
        series: [
          {
            name: `Semaine ${weekNumber} (${year})`,
            data: Object.values(weekData),
            type: "line",
            color: "#5a189a",
            smooth: true,
          },
        ],
        tooltip: {
          trigger: "axis",
        },
        title: {
          text: "",
        },
      };
    },
  },
  watch: {
    // Si les données initiales changent, recharger la semaine courante
    chartData: {
      handler() {
        this.updateWeekData();
      },
      immediate: true,
      deep: true,
    },
  },
};
</script>

<style scoped>
/* Ajouter un style pour les flèches */
.chart-header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.chart-title {
  text-align: center;
  font-weight: 900;
  font-size: 28px;
  color: rgb(82, 26, 122);
  margin-bottom: 20px;
}

.arrow-button {
  background-color: transparent;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 10px;
}

.arrow-button i {
  color:  rgb(82, 26, 122); 
 
}

.arrow-button:hover {
  background-color: #f0f0f0;
  border-radius: 50%;
}
</style>
