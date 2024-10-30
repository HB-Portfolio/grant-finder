<script setup>
import { useSupabaseClient } from '#imports'
import { ref, onMounted, computed } from 'vue'

const supabase = useSupabaseClient()
const grants = ref([])
const searchQuery = ref('')
const selectedFilter = ref('')

// Define filter options for dropdown
const filterOptions = [
  { label: 'All', value: '' },
  { label: 'International Students', value: 'International' },
  { label: 'Alberta Students', value: 'Alberta' },
  { label: 'Canadian Students', value: 'Canadian' },
  { label: 'Full-Time Students', value: 'Full-Time' },
  { label: 'Athletes', value: 'Athlete' },
  { label: 'Academic', value: 'Academic' },
]

// Fetch grants from Supabase
async function getGrants() {
  const { data, error } = await supabase.from('grants').select('*')
  if (error) {
    console.error('Error fetching grants:', error)
  } else {
    grants.value = data
  }
}

// Filter grants based on search and dropdown selection
const filteredGrants = computed(() =>
  grants.value.filter((grant, index, self) => {
    // Filter out duplicates
    const isDuplicate = self.findIndex(g => g.award_name === grant.award_name && g.award_type === grant.award_type) !== index
    if (isDuplicate) return false

    // Apply search filter and dropdown filter
    const matchesText = searchQuery.value ? grant.award_name.toLowerCase().includes(searchQuery.value.toLowerCase()) : true
    const matchesFilter = selectedFilter.value ? grant.available_to?.includes(selectedFilter.value) : true
    return matchesText && matchesFilter
  })
)

onMounted(() => {
  getGrants()
})
</script>

<template>
  <div class="p-8 bg-gray-100 min-h-screen">
    <h1 class="text-4xl font-bold mb-8 text-center">Grants Finder</h1>

    <!-- Filters Section -->
    <div class="flex flex-col md:flex-row justify-center items-center gap-4 mb-8">
      <!-- Search Filter -->
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search grants by name..."
        class="px-4 py-2 w-full md:w-1/3 border border-gray-300 rounded-md"
      />

      <!-- Dropdown Filter -->
      <select v-model="selectedFilter" class="px-4 py-2 border border-gray-300 rounded-md">
        <option v-for="option in filterOptions" :key="option.value" :value="option.value">
          {{ option.label }}
        </option>
      </select>
    </div>

    <!-- Grants List -->
    <div class="grid gap-6 max-w-3xl mx-auto">
      <div
        v-for="grant in filteredGrants"
        :key="grant.id"
        class="bg-white p-6 rounded-lg shadow-md border border-gray-200"
      >
        <h2 class="text-2xl font-semibold mb-2 text-gray-800">{{ grant.award_name }}</h2>

        <p v-if="grant.award_type && grant.award_type !== 'N/A'" class="text-gray-700"><strong>Type:</strong> {{ grant.award_type }}</p>
        <p v-if="grant.value && grant.value !== 'N/A'" class="text-gray-700"><strong>Value:</strong> ${{ grant.value }}</p>
        <p v-if="grant.spots_available && grant.spots_available !== 'N/A'" class="text-gray-700"><strong>Spots Available:</strong> {{ grant.spots_available }}</p>
        <p v-if="grant.criteria" class="text-gray-600 mt-3"><strong>Criteria:</strong> {{ grant.criteria }}</p>
        <p v-if="grant.available_to" class="text-gray-600 mt-1"><strong>Available To:</strong> {{ grant.available_to }}</p>
        <p v-if="grant.yr_eligibility" class="text-gray-600 mt-1"><strong>Year Eligibility:</strong> {{ grant.yr_eligibility }}</p>
        <p v-if="grant.gpa_requirement" class="text-gray-600 mt-1"><strong>GPA Requirement:</strong> {{ grant.gpa_requirement }}</p>
        <p v-if="grant.school" class="text-gray-600 mt-1"><strong>School:</strong> {{ grant.school }}</p>
        <p v-if="grant.program" class="text-gray-600 mt-1"><strong>Program:</strong> {{ grant.program }}</p>
        <p v-if="grant.preference" class="text-gray-600 mt-1"><strong>Preference:</strong> {{ grant.preference }}</p>
        <p v-if="grant.deadline" class="text-gray-600 mt-1"><strong>Deadline:</strong> {{ grant.deadline }}</p>
        <p v-if="grant.resident_status" class="text-gray-600 mt-1"><strong>Resident Status:</strong> {{ grant.resident_status }}</p>
      </div>
    </div>
  </div>
</template>
