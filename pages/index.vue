<template>
  <div class="container  mx-auto bg-slate-400">
    <h1 class="text-2xl font-bold mb-4 text-center">
      Project Status
    </h1>
  
    <div class="flex justify-between items-center mb-4">
      <div class="ml-2">
        <label for="client-filter" class="mr-2 font-bold text-lg"
          >Client:</label>
        <input
          type="text"
          v-model="filters.client"
          id="client-filter"
          class="px-2 py-1 border border-gray-300 rounded-md"
        />
      </div>
      <div>
        <label for="status-filter" class="mr-2 font-bold text-lg"
          >Status:</label
        >
        <select
          v-model="filters.status"
          id="status-filter"
          class="px-2 py-1 border border-gray-300 rounded-md mr-5"
        >
          <option value="">All</option>
          <option value="In Progress">In Progress</option>
          <option value="Completed">Completed</option>
          <option value="On Hold">On Hold</option>
          <option value="Cancelled">Cancelled</option>
          <option value="Not Started">Not Started</option>
        </select>
      </div>
    </div>
    <table class="min-w-full bg-white border border-gray-300">
      <thead>
        <tr>
          <th class="py-3 px-4 bg-gray-100 border-b text-left cursor-pointer" @click="sortTable('client')">Client</th>
          <th class="py-3 px-4 bg-gray-100 border-b text-left cursor-pointer" @click="sortTable('project')">Project</th>
          <th class="py-3 px-4 bg-gray-100 border-b text-left cursor-pointer" @click="sortTable('startDate')">Start Date</th>
          <th class="py-3 px-4 bg-gray-100 border-b text-center cursor-pointer" @click="sortTable('revenue')">Revenue</th>
          <th class="py-3 px-4 bg-gray-100 border-b text-left cursor-pointer">Status</th>
          <th class="py-3 px-4 bg-gray-100 border-b text-left cursor-pointer" @click="sortTable('percentCompletion')">Completion</th>
          <th class="py-3 px-4 bg-gray-100 border-b text-left cursor-pointer" @click="sortTable('margin')">Margin</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in filteredState" :key="item.project">
          <td class="py-4 px-6 border-b">{{ item.client }}</td>
          <td class="py-4 px-6 border-b">{{ item.project }}</td>
          <td class="py-4 px-6 border-b">{{ item.startDate }}</td>
          <td class="py-4 px-6 border-b text-center">{{ item.revenue }}</td>
          <td class="py-4 px-6 border-b">
            <select
              v-model="item.status"
              @change="updateCompletionColor(item)"
              class="py-2 px-3 border border-gray-300 rounded-md"
            >
              <option value="In Progress">In Progress</option>
              <option value="Completed">Completed</option>
              <option value="On Hold">On Hold</option>
              <option value="Cancelled">Cancelled</option>
              <option value="Not Started">Not Started</option>
            </select>
          </td>
          <td class="py-4 px-6 border-b">
            <div class="flex items-center">
              <input
                v-model.number="item.percentCompletion"
                @input="validateCompletion(item)"
                type="number"
                min="0"
                max="100"
                step="1"
                class="w-16 py-2 px-3 border border-gray-300 rounded-md"
              />
              <div
                class="flex-grow h-2 bg-gray-200 rounded-full overflow-hidden ml-2"
              >
                <div
                  :class="getCompletionColorClass(item.percentCompletion)"
                  :style="getCompletionWidthStyle(item.percentCompletion)"
                  class="h-full rounded-full"
                ></div>
              </div>
              <span class="ml-2">{{ item.percentCompletion }}%</span>
            </div>
          </td>
          <td class="py-4 px-6 border-b">
            <template
              v-if="
                getMarginAlertClass(item.margin) === 'text-red-500 font-bold'
              "
            >
              <span :class="getMarginAlertClass(item.margin)">
                {{ item.margin }}
                <svg
                  class="inline w-4 h-4 ml-1"
                  fill="currentColor"
                  viewBox="0 0 20 20"
                >
                  <path
                    fill-rule="evenodd"
                    d="M10 3.29l-6.365 6.364a2 2 0 0 0 0 2.83L10 16.71l6.364-6.364a2 2 0 0 0 0-2.83L10 3.29zm0 9.42a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"
                  ></path>
                </svg>
              </span>
            </template>
            <template v-else>
              <span :class="getMarginAlertClass(item.margin)">
                {{ item.margin }}
              </span>
            </template>
          </td>
        </tr>
      </tbody>
      <tfoot>
            <tr>
              <th class="py-2 px-4"></th>
              <th class="py-2 px-4"></th>
              <th class="py-2 px-4"></th>
              <th class="py-2 px-4">Total: {{ totalRevenue }}</th>
              <th class="py-2 px-4"></th>
              <th class="py-2 px-4"></th>
              <th class="py-2 px-4"></th>
            </tr>
          </tfoot>
    </table>
  </div>

</template>

<script>
import './style.css'
export default {
  data() {
    return {
      state: [
        {
          client: "Client A",
          project: "Project W",
          startDate: "2023-01-01",
          revenue: 4000,
          status: "Completed",
          percentCompletion: 100,
          margin: -500,
        },
        {
          client: "Client B",
          project: "Project X",
          startDate: "2023-20-01",
          revenue: 8000,
          status: "In Progress",
          percentCompletion: 60,
          margin: 800,
        },
        {
          client: "Client C",
          project: "Project Y",
          startDate: "2023-06-01",
          revenue: 3200,
          status: "Cancelled",
          percentCompletion: 0,
          margin: 500,
        },
        {
          client: "Client D",
          project: "Project Z",
          startDate: "2023-08-01",
          revenue: 1500,
          status: "On Hold",
          percentCompletion: 25,
          margin: 600,
        },
        // Add more items as needed
      ],
      sortColumn: '',
      sortDirection: '',
      filters: {
        client: "",
        status: "",
      },
    };
  },
  // Filters the state based on the selected client and status
  computed: {
    
    filteredState() {
      const { client, status } = this.filters;
      return this.state.filter((item) => {
        return (
          (client === "" ||
            item.client.toLowerCase().includes(client.toLowerCase())) &&
          (status === "" || item.status === status)
        );
      });
      
    },
    // Calculates the total revenue
    totalRevenue() {
    return this.filteredState.reduce((total, item) => total + item.revenue, 0);
  }
  },
  
  methods: {
     // Returns the CSS class for the completion color based on the percent completion
    getCompletionColorClass(percentCompletion) {
      if (percentCompletion <= 25) {
        return "bg-red-500";
      } else if (percentCompletion <= 50) {
        return "bg-yellow-500";
      } else if (percentCompletion <= 75) {
        return "bg-blue-500";
      } else {
        return "bg-green-500";
      }
    },
    //Returns the inline style for the completion width based on the percent completion
    getCompletionWidthStyle(percentCompletion) {
      return `width: ${percentCompletion}%`;
    },
    // Returns the CSS class for the margin alert based on the margin value
    getMarginAlertClass(margin) {
      if (margin < 0) {
        return "text-red-500 font-bold";
      } else {
        return "text-gray-700";
      }
    },
     // Updates the percent completion based on the selected status
    updateCompletionColor(item) {
      const status = item.status;
      if (status === "Completed") {
        item.percentCompletion = 100;
      } else if (status === "In Progress") {
        item.percentCompletion = 50;
      } else if (status === "On Hold") {
        item.percentCompletion = 25;
      } else if (status === "Cancelled" || status === "Not Started") {
        item.percentCompletion = 0;
      }
      this.saveStateToLocalStorage();
    },
    // Validates and updates the percent completion
    validateCompletion(item) {
      if (item.percentCompletion < 0) {
        item.percentCompletion = 0;
      } else if (item.percentCompletion > 100) {
        item.percentCompletion = 100;
      }
      this.saveStateToLocalStorage();
    },
     // Saves the state to localStorage
    saveStateToLocalStorage() {
      localStorage.setItem('state', JSON.stringify(this.state));
    },
    // Loads the state from localStorage
    loadStateFromLocalStorage() {
      const savedState = localStorage.getItem('state');
      if (savedState) {
        this.state = JSON.parse(savedState);
      }
    },

    // Sorts the table based on the selected column

    sortTable(column) {
      if (this.sortColumn === column) {
        this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortColumn = column;
        this.sortDirection = 'asc';
      }

      this.sortState();
    },

    // Sorts the state based on the selected column and direction
    sortState() {
      this.state.sort((a, b) => {
        const aValue = a[this.sortColumn];
        const bValue = b[this.sortColumn];

        if (aValue < bValue) {
          return this.sortDirection === 'asc' ? -1 : 1;
        } else if (aValue > bValue) {
          return this.sortDirection === 'asc' ? 1 : -1;
        } else {
          return 0;
        }
      });
    },
  },

  
  mounted() {
    this.loadStateFromLocalStorage();
  }, 
};
</script>

<style>


</style>
