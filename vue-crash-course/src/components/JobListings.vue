<script setup>
import { reactive, ref, defineProps, onMounted, computed } from 'vue';
import JobListing from './JobListing.vue';
import { RouterLink } from 'vue-router';
import axios from 'axios';
import PulseLoader from 'vue-spinner/src/PulseLoader.vue';

const props = defineProps({
    limit: Number,
    showButton: {
        type: Boolean,
        default: false
    }
});

const state = reactive({
    jobs: [],
    isLoading: true,
    currentPage: 1,
    jobsPerPage: 6,  // Number of jobs to display per page
    searchQuery: '', // For search functionality
    selectedType: '', // For filtering by job type
    sortOrder: 'asc', // Default sort order
});

// Function to parse the salary into a numeric value (average of the range)
const parseSalary = (salary) => {
    if (salary.includes('/hr')) {
        // For hourly rates, convert to a rough annual salary
        const hourlyRate = parseFloat(salary.replace(/[^0-9.]/g, ''));
        return hourlyRate * 2000; // Assuming 2000 working hours in a year
    } else {
        const range = salary.match(/\$([0-9]+)K - \$([0-9]+)K/);
        if (range) {
            const low = parseFloat(range[1]) * 1000;
            const high = parseFloat(range[2]) * 1000;
            return (low + high) / 2; // Return the average salary
        }
    }
    return 0; // Default value if parsing fails
};

const filteredJobs = computed(() => {
    return state.jobs.filter(job => {
        const matchesTitle = job.title.toLowerCase().includes(state.searchQuery.toLowerCase());
        const matchesType = state.selectedType ? job.type === state.selectedType : true;
        return matchesTitle && matchesType;
    });
});

const sortedJobs = computed(() => {
    return filteredJobs.value.slice().sort((a, b) => {
        const salaryA = parseSalary(a.salary);
        const salaryB = parseSalary(b.salary);
        return state.sortOrder === 'asc' ? salaryA - salaryB : salaryB - salaryA;
    });
});

const paginatedJobs = computed(() => {
    const start = (state.currentPage - 1) * state.jobsPerPage;
    const end = start + state.jobsPerPage;
    return sortedJobs.value.slice(start, end);
});

const totalPages = computed(() => {
    return Math.ceil(sortedJobs.value.length / state.jobsPerPage);
});

const goToPage = (page) => {
    state.currentPage = page;
};

const toggleSortOrder = () => {
    state.sortOrder = state.sortOrder === 'asc' ? 'desc' : 'asc';
};

onMounted(async () => {
    try {
        const response = await axios.get('/api/jobs');
        state.jobs = response.data;
    } catch (error) {
        console.error('Error fetching jobs', error);
    } finally {
        state.isLoading = false;
    }
});
</script>

<template>
    <section class="bg-blue-50 px-4 py-10">
        <div class="container-xl lg:container m-auto">
            <h2 class="text-3xl font-bold text-green-500 bb-6 text-center">
                Browse Jobs
            </h2>
            
            <!-- Search, Filter, and Sort -->
            <div class="mb-4 flex justify-between items-center">
                <input 
                    v-model="state.searchQuery" 
                    type="text" 
                    placeholder="Search jobs by title..." 
                    class="px-4 py-2 rounded-lg border border-gray-300 w-full md:w-1/3"
                />

                <select 
                    v-model="state.selectedType" 
                    class="ml-4 px-4 py-2 rounded-lg border border-gray-300 w-full md:w-1/4"
                >
                    <option value="">All Types</option>
                    <option value="Full-Time">Full-Time</option>
                    <option value="Part-Time">Part-Time</option>
                    <option value="Intern">Intern</option>
                </select>

                <button @click="toggleSortOrder" class="ml-4 px-4 py-2 rounded-lg bg-green-500 text-white">
                    Sort by Salary {{ state.sortOrder === 'asc' ? '↑' : '↓' }}
                </button>
            </div>

            <!-- Show loading spinner while loading is true -->
            <div v-if="state.isLoading" class="text-center text-gray-500 py-6">
                <PulseLoader />
            </div>

            <!-- Show job listing when done loading -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <JobListing v-for="job in paginatedJobs" :key="job.id" :job="job" />
            </div>
        </div>
    </section>

    <!-- Pagination controls -->
    <div class="flex justify-center mt-6">
        <button
            v-for="page in totalPages"
            :key="page"
            @click="goToPage(page)"
            :class="['px-4 py-2 mx-1 rounded-lg', { 'bg-green-500 text-white': state.currentPage === page, 'bg-white text-green-500': state.currentPage !== page }]"
        >
            {{ page }}
        </button>
    </div>

    <section v-if="props.showButton" class="m-auto max-w-lg my-10 px-6">
        <RouterLink to="/jobs" class="block bg-black text-white text-center py-4 px-6 rounded-xl hover:bg-gray-700">View All Jobs</RouterLink>
    </section>
</template>
