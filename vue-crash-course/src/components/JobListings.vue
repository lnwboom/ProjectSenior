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
});

const totalPages = computed(() => {
    return Math.ceil(state.jobs.length / state.jobsPerPage);
});

const paginatedJobs = computed(() => {
    const start = (state.currentPage - 1) * state.jobsPerPage;
    const end = start + state.jobsPerPage;
    return state.jobs.slice(start, end);
});

const goToPage = (page) => {
    state.currentPage = page;
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
