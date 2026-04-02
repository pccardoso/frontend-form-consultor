<template>
  <div class="space-y-8">
    <!-- Header -->
    <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 bg-white p-8 rounded-3xl border border-slate-200/60 shadow-sm">
      <div>
        <h1 class="text-2xl font-black text-brand-navy tracking-tight">Dashboard Analítico</h1>
        <p class="text-sm text-gray-500 font-medium">Visualize as estatísticas e tendências das pesquisas SGA.</p>
      </div>
      
      <div class="flex flex-wrap items-center gap-3 bg-gray-50/50 p-2 rounded-2xl border border-slate-200/40">
        <div class="flex items-center gap-2">
          <label class="text-xs font-bold text-gray-400 uppercase ml-2">Mês/Ano:</label>
          <select 
            v-model="selectedMonth" 
            class="bg-white border border-brand-light rounded-lg px-3 py-1.5 text-sm font-medium text-brand-navy focus:ring-2 focus:ring-brand-green outline-none transition-all"
          >
            <option v-for="month in availableMonths" :key="month.value" :value="month.value">
              {{ month.label }}
            </option>
          </select>
        </div>
        <div class="flex items-center gap-2">
          <button @click="loadData" class="p-2 text-brand-green hover:bg-brand-light rounded-lg transition-colors" title="Atualizar dados">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" :class="{ 'animate-spin': loading }" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="flex flex-col items-center justify-center py-24 bg-white rounded-3xl border border-gray-100 shadow-sm">
      <div class="animate-spin rounded-full h-16 w-16 border-b-2 border-brand-green mb-4"></div>
      <p class="text-gray-500 font-medium text-lg">Processando estatísticas...</p>
    </div>

    <template v-else>
      <!-- Stats Summary -->
      <div class="grid grid-cols-1 md:grid-cols-4 gap-6">
        <div class="bg-white p-6 rounded-3xl border border-slate-200/60 shadow-sm hover:shadow-md transition-all hover:-translate-y-1">
          <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest mb-1">Total no Mês</p>
          <p class="text-3xl font-black text-brand-navy leading-none">{{ monthItems.length }}</p>
          <div class="mt-3 flex items-center text-[10px]" :class="growth >= 0 ? 'text-emerald-600' : 'text-rose-600'">
            <span class="font-black bg-emerald-50 px-1.5 py-0.5 rounded">{{ growth >= 0 ? '+' : '' }}{{ growth }}%</span>
            <span class="text-gray-400 ml-1 font-bold">vs mês anterior</span>
          </div>
        </div>
        <div class="bg-white p-6 rounded-3xl border border-slate-200/60 shadow-sm hover:shadow-md transition-all hover:-translate-y-1">
          <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest mb-1">Média Diária</p>
          <p class="text-3xl font-black text-brand-navy leading-none">{{ dailyAverage }}</p>
          <p class="text-[10px] text-gray-400 mt-3 font-bold italic">Pesquisas por dia útil</p>
        </div>
        <div class="bg-white p-6 rounded-3xl border border-slate-200/60 shadow-sm hover:shadow-md transition-all hover:-translate-y-1">
          <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest mb-1">Consultor Top</p>
          <p class="text-lg font-black text-brand-navy truncate leading-none">{{ topConsultant.name || 'N/A' }}</p>
          <p class="text-xs text-brand-green font-black mt-2">{{ topConsultant.count || 0 }} pesquisas</p>
        </div>
        <div class="bg-white p-6 rounded-3xl border border-slate-200/60 shadow-sm hover:shadow-md transition-all hover:-translate-y-1">
          <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest mb-1">Taxa com Terceiro</p>
          <p class="text-3xl font-black text-brand-navy leading-none">{{ thirdPartyRate }}%</p>
          <div class="w-full bg-gray-100 h-1.5 rounded-full mt-4 overflow-hidden">
            <div class="bg-amber-500 h-full rounded-full shadow-sm" :style="{ width: thirdPartyRate + '%' }"></div>
          </div>
        </div>
      </div>

      <!-- Charts Grid -->
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <!-- Line Chart: Searches per Day -->
        <div class="bg-white p-8 rounded-3xl border border-slate-200/60 shadow-sm flex flex-col h-[450px]">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-brand-navy">Volume de Pesquisas Diárias</h3>
            <span class="text-xs font-medium px-2 py-1 bg-brand-light text-brand-green rounded-lg">Mês Atual</span>
          </div>
          <div class="flex-1 relative min-h-0">
            <Line v-if="lineChartData" :data="lineChartData" :options="lineChartOptions" />
          </div>
        </div>

        <!-- Pie Chart: Consultant Type -->
        <div class="bg-white p-8 rounded-3xl border border-slate-200/60 shadow-sm flex flex-col h-[450px]">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-brand-navy">Distribuição por Tipo de Consultor</h3>
            <div class="flex gap-2">
              <span class="flex items-center gap-1 text-[10px] font-bold text-gray-400 uppercase">
                <span class="w-2 h-2 rounded-full bg-brand-green"></span> Interno
              </span>
              <span class="flex items-center gap-1 text-[10px] font-bold text-gray-400 uppercase">
                <span class="w-2 h-2 rounded-full bg-brand-navy"></span> Externo
              </span>
            </div>
          </div>
          <div class="flex-1 relative min-h-0 flex items-center justify-center">
            <div class="w-full max-w-[300px]">
              <Pie v-if="pieChartData" :data="pieChartData" :options="pieChartOptions" />
            </div>
          </div>
        </div>

        <!-- Pie Chart: Evogard Education -->
        <div class="bg-white p-8 rounded-3xl border border-slate-200/60 shadow-sm flex flex-col h-[450px]">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-brand-navy">Consultou Univ. Evogard?</h3>
            <div class="flex gap-2">
              <span class="flex items-center gap-1 text-[10px] font-bold text-gray-400 uppercase">
                <span class="w-2 h-2 rounded-full bg-brand-green"></span> Sim
              </span>
              <span class="flex items-center gap-1 text-[10px] font-bold text-gray-400 uppercase">
                <span class="w-2 h-2 rounded-full bg-rose-500"></span> Não
              </span>
            </div>
          </div>
          <div class="flex-1 relative min-h-0 flex items-center justify-center">
            <div class="w-full max-w-[300px]">
              <Pie v-if="evogardEduChartData" :data="evogardEduChartData" :options="pieChartOptions" />
            </div>
          </div>
        </div>

        <!-- Pie Chart: Client Resolved -->
        <div class="bg-white p-8 rounded-3xl border border-slate-200/60 shadow-sm flex flex-col h-[450px]">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-brand-navy">Tentou resolver administrativo?</h3>
            <div class="flex gap-2">
              <span class="flex items-center gap-1 text-[10px] font-bold text-gray-400 uppercase">
                <span class="w-2 h-2 rounded-full bg-brand-green"></span> Sim
              </span>
              <span class="flex items-center gap-1 text-[10px] font-bold text-gray-400 uppercase">
                <span class="w-2 h-2 rounded-full bg-rose-500"></span> Não
              </span>
            </div>
          </div>
          <div class="flex-1 relative min-h-0 flex items-center justify-center">
            <div class="w-full max-w-[300px]">
              <Pie v-if="clienteResolveuChartData" :data="clienteResolveuChartData" :options="pieChartOptions" />
            </div>
          </div>
        </div>

        <!-- Bar Chart: Department Rate -->
        <div class="bg-white p-8 rounded-3xl border border-slate-200/60 shadow-sm flex flex-col h-[450px] lg:col-span-2">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-brand-navy">Taxa por Departamento</h3>
            <p class="text-xs text-gray-400">Distribuição percentual de todas as pesquisas</p>
          </div>
          <div class="flex-1 relative min-h-0">
            <Bar v-if="barChartData" :data="barChartData" :options="barChartOptions" />
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import { dataService } from '../services/dataService';
import { 
  Chart as ChartJS, 
  Title, 
  Tooltip, 
  Legend, 
  LineElement, 
  LinearScale, 
  PointElement, 
  CategoryScale,
  ArcElement,
  BarElement
} from 'chart.js';
import { Line, Pie, Bar } from 'vue-chartjs';
import ChartDataLabels from 'chartjs-plugin-datalabels';

ChartJS.register(
  Title, 
  Tooltip, 
  Legend, 
  LineElement, 
  LinearScale, 
  PointElement, 
  CategoryScale,
  ArcElement,
  BarElement,
  ChartDataLabels
);

export default {
  name: 'Analytics',
  components: { Line, Pie, Bar },
  data() {
    const now = new Date();
    const currentMonth = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}`;
    
    return {
      items: [],
      loading: false,
      selectedMonth: currentMonth,
      availableMonths: [],
      lineChartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { display: false },
          tooltip: {
            backgroundColor: '#001F3F',
            titleFont: { size: 12, weight: 'bold' },
            bodyFont: { size: 12 },
            padding: 12,
            cornerRadius: 12,
            displayColors: false
          },
          datalabels: {
            align: 'top',
            anchor: 'end',
            offset: 4,
            color: '#001F3F',
            font: { weight: 'bold', size: 10 },
            formatter: (value) => value > 0 ? value : ''
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            grid: { color: '#f3f4f6', drawBorder: false },
            ticks: { font: { size: 10, weight: 'bold' }, color: '#9ca3af' }
          },
          x: {
            grid: { display: false },
            ticks: { font: { size: 10, weight: 'bold' }, color: '#9ca3af' }
          }
        },
        elements: {
          line: { tension: 0.4, borderWidth: 3, borderColor: '#00D166' },
          point: { radius: 4, backgroundColor: '#00D166', hoverRadius: 6 }
        }
      },
      pieChartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { display: false },
          tooltip: {
            backgroundColor: '#001F3F',
            padding: 12,
            cornerRadius: 12
          },
          datalabels: {
            color: '#fff',
            font: { weight: 'bold', size: 12 },
            formatter: (value, ctx) => {
              const sum = ctx.dataset.data.reduce((a, b) => a + b, 0);
              const percentage = ((value * 100) / sum).toFixed(0) + '%';
              return value > 0 ? percentage : '';
            }
          }
        }
      },
      barChartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { display: false },
          tooltip: {
            backgroundColor: '#001F3F',
            padding: 12,
            cornerRadius: 12
          },
          datalabels: {
            align: 'end',
            anchor: 'end',
            color: '#001F3F',
            font: { weight: 'bold', size: 10 },
            formatter: (value) => value > 0 ? value + '%' : ''
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            grid: { color: '#f3f4f6', drawBorder: false },
            ticks: { font: { size: 10, weight: 'bold' }, color: '#9ca3af' }
          },
          x: {
            grid: { display: false },
            ticks: { font: { size: 10, weight: 'bold' }, color: '#9ca3af' }
          }
        }
      }
    }
  },
  computed: {
    monthItems() {
      return this.items.filter(item => item.created_at.startsWith(this.selectedMonth));
    },
    dailyAverage() {
      if (this.monthItems.length === 0) return 0;
      const daysWithData = new Set(this.monthItems.map(i => i.created_at.split('T')[0])).size;
      return (this.monthItems.length / (daysWithData || 1)).toFixed(1);
    },
    topConsultant() {
      if (this.monthItems.length === 0) return { name: '', count: 0 };
      const counts = {};
      this.monthItems.forEach(i => {
        const name = i.consultor_associado_sga;
        counts[name] = (counts[name] || 0) + 1;
      });
      const top = Object.entries(counts).sort((a, b) => b[1] - a[1])[0];
      return top ? { name: top[0], count: top[1] } : { name: '', count: 0 };
    },
    thirdPartyRate() {
      if (this.monthItems.length === 0) return 0;
      const withThird = this.monthItems.filter(i => !!i.nome_terceiro).length;
      return ((withThird / this.monthItems.length) * 100).toFixed(0);
    },
    growth() {
      const [year, month] = this.selectedMonth.split('-').map(Number);
      const prevDate = new Date(year, month - 2, 1);
      const prevMonthStr = `${prevDate.getFullYear()}-${String(prevDate.getMonth() + 1).padStart(2, '0')}`;
      
      const prevMonthCount = this.items.filter(i => i.created_at.startsWith(prevMonthStr)).length;
      const currentMonthCount = this.monthItems.length;
      
      if (prevMonthCount === 0) return currentMonthCount > 0 ? 100 : 0;
      return (((currentMonthCount - prevMonthCount) / prevMonthCount) * 100).toFixed(0);
    },
    lineChartData() {
      if (this.monthItems.length === 0) return null;
      
      const [year, month] = this.selectedMonth.split('-').map(Number);
      const daysInMonth = new Date(year, month, 0).getDate();
      const labels = Array.from({ length: daysInMonth }, (_, i) => i + 1);
      
      const counts = Array(daysInMonth).fill(0);
      this.monthItems.forEach(item => {
        const day = new Date(item.created_at).getDate();
        counts[day - 1]++;
      });
      
      return {
        labels,
        datasets: [{
          label: 'Pesquisas',
          data: counts,
          fill: true,
          backgroundColor: 'rgba(0, 209, 102, 0.1)',
          borderColor: '#00D166',
          tension: 0.4
        }]
      };
    },
    pieChartData() {
      if (this.monthItems.length === 0) return null;
      
      const internal = this.monthItems.filter(i => i.type_consultant?.toLowerCase() === 'interno').length;
      const external = this.monthItems.filter(i => i.type_consultant?.toLowerCase() === 'externo').length;
      
      return {
        labels: ['Interno', 'Externo'],
        datasets: [{
          data: [internal, external],
          backgroundColor: ['#00D166', '#001F3F'],
          borderWidth: 0,
          hoverOffset: 10
        }]
      };
    },
    evogardEduChartData() {
      if (this.monthItems.length === 0) return null;
      const sim = this.monthItems.filter(i => i.pes_evogard_edu === 'Sim').length;
      const nao = this.monthItems.filter(i => i.pes_evogard_edu === 'Não').length;
      return {
        labels: ['Sim', 'Não'],
        datasets: [{
          data: [sim, nao],
          backgroundColor: ['#00D166', '#F43F5E'],
          borderWidth: 0,
          hoverOffset: 10
        }]
      };
    },
    clienteResolveuChartData() {
      if (this.monthItems.length === 0) return null;
      const sim = this.monthItems.filter(i => i.pes_cliente_resolveu === 'Sim').length;
      const nao = this.monthItems.filter(i => i.pes_cliente_resolveu === 'Não').length;
      return {
        labels: ['Sim', 'Não'],
        datasets: [{
          data: [sim, nao],
          backgroundColor: ['#00D166', '#F43F5E'],
          borderWidth: 0,
          hoverOffset: 10
        }]
      };
    },
    barChartData() {
      if (this.monthItems.length === 0) return null;
      
      const depts = {};
      this.monthItems.forEach(i => {
        const dept = i.department || 'N/A';
        depts[dept] = (depts[dept] || 0) + 1;
      });
      
      const labels = Object.keys(depts);
      const data = Object.values(depts);
      
      return {
        labels,
        datasets: [{
          label: 'Pesquisas',
          data,
          backgroundColor: '#00D166',
          borderRadius: 8,
          barThickness: 40
        }]
      };
    }
  },
  async created() {
    this.generateAvailableMonths();
    await this.loadData();
  },
  methods: {
    async loadData() {
      this.loading = true;
      try {
        this.items = await dataService.getAll();
      } catch (error) {
        console.error('Erro ao carregar dados:', error);
      } finally {
        this.loading = false;
      }
    },
    generateAvailableMonths() {
      const months = [
        'Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho',
        'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'
      ];
      const now = new Date();
      const currentYear = now.getFullYear();
      const currentMonth = now.getMonth();
      
      const result = [];
      // Generate last 12 months
      for (let i = 0; i < 12; i++) {
        const d = new Date(currentYear, currentMonth - i, 1);
        const y = d.getFullYear();
        const m = d.getMonth();
        result.push({
          label: `${months[m]}/${y}`,
          value: `${y}-${String(m + 1).padStart(2, '0')}`
        });
      }
      this.availableMonths = result;
    }
  }
}
</script>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  width: 4px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #e5e7eb;
  border-radius: 10px;
}
</style>
