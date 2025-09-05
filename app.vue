<script setup>
const client = useSupabaseClient();

/**
 * @param {string} table
 * @returns {Promise<any[]>}
 */
async function getTableData(table) {
  const { data, error } = await client.from(table).select("*");
  if (error) {
    console.error(error);
    return [];
  }
  return data;
}

/** @type {import('vue').Ref<any[] | null>} */
const dataReference = ref(null);
const formState = reactive({
  name: "",
  email: "",
  message: "",
});

async function fetchAndSetData() {
  const retrievedData = await getTableData("records");
  if (retrievedData && retrievedData.length > 0) {
    dataReference.value = retrievedData;
  } else {
    dataReference.value = [];
  }
}

onMounted(fetchAndSetData);

async function submitForm() {
  console.log("Form State: ", formState);
  
  const { error } = await client.from("records").insert({
    name: formState.name,
    email: formState.email,
    message: formState.message
  });

  if (error) {
    console.error("Error inserting data:", error);
  } else {
    formState.name = "";
    formState.email = "";
    formState.message = "";
    await fetchAndSetData();
  }
}

async function deleteRecord(id) {
  const { error } = await client.from("records").delete().eq("id", id);
  if (error) {
    console.error("Error deleting data:", error);
  } else {
    await fetchAndSetData();
  }
}
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen p-4 bg-gray-100">
    <div class="bg-white p-8 rounded-xl shadow-lg w-full max-w-2xl">
      <img src="/full_logo.png" alt="Clear Lakes Dental" class="w-48 mx-auto mb-6" />
      <h1 class="text-3xl font-bold text-center text-gray-800 mb-2">Supabase Demo</h1>
      <p class="text-center text-gray-500 mb-8">
        Enter your information below to submit a record to the Supabase database.
      </p>

      <!-- Form Section -->
      <form @submit.prevent="submitForm" class="flex flex-col space-y-4">
        <div>
          <label for="name" class="block text-sm font-medium text-gray-700">Name</label>
          <input
            id="name"
            type="text"
            v-model="formState.name"
            placeholder="Enter your name"
            required
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-2"
          />
        </div>
        <div>
          <label for="email" class="block text-sm font-medium text-gray-700">Email</label>
          <input
            id="email"
            type="email"
            v-model="formState.email"
            placeholder="Enter your email"
            required
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-2"
          />
        </div>
        <div>
          <label for="message" class="block text-sm font-medium text-gray-700">Message</label>
          <textarea
            id="message"
            v-model="formState.message"
            placeholder="Enter a message"
            required
            rows="3"
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-2"
          ></textarea>
        </div>
        <button
          type="submit"
          class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition-colors duration-200"
        >
          Submit
        </button>
      </form>

      <!-- Display Section -->
      <div class="mt-10">
        <h2 class="text-2xl font-bold text-center text-gray-800 mb-4">Saved Records</h2>
        <ul class="space-y-4">
          <li v-if="!dataReference">
            <p class="text-gray-500 text-center">Loading records...</p>
          </li>
          <li v-else-if="dataReference.length === 0">
            <p class="text-gray-500 text-center">No records found. Add one above!</p>
          </li>
          <li
            v-for="item in dataReference"
            :key="item.id"
            class="bg-gray-50 p-4 rounded-md shadow-sm border border-gray-200 flex justify-between items-center"
          >
            <div>
              <div class="font-bold text-gray-800">{{ item.name }}</div>
              <div class="text-sm text-gray-500">{{ item.email }}</div>
              <div class="mt-2 text-gray-700">{{ item.message }}</div>
            </div>
            <button
              @click="deleteRecord(item.id)"
              class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg transition-colors duration-200"
            >
              Delete
            </button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
