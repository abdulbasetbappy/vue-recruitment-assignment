<template>
  <div class="wrapper">
    <h2>Submit a Complaint</h2>
    <!-- Form to add a new complaint -->
    <form class="complain-form" @submit.prevent="saveComplain">
      <!-- Title input field -->
      <div class="form-group">
        <input
          v-model="title"
          type="text"
          placeholder="Enter title"
          :class="{ 'error-input': errors.title }"
        />
        <p v-if="errors.title" class="error-message">{{ errors.title }}</p>
      </div>
      <!-- Body textarea field -->
      <div class="form-group">
        <textarea
          v-model="body"
          placeholder="Enter your complaint"
          :class="{ 'error-input': errors.body }"
        ></textarea>
        <p v-if="errors.body" class="error-message">{{ errors.body }}</p>
      </div>
      <!-- Submit button -->
      <button :disabled="isSaving" type="submit">
        {{ isSaving ? "Saving..." : "Submit Complaint" }}
      </button>
    </form>

    <h2>Complaints List</h2>
    <!-- Simple loader while loading -->
    <div v-if="isLoading === 'pending'" class="loader">Loading...</div>
    <!-- Display complaints list -->
    <template v-else-if="complains.length">
      <div class="complain-list-wrapper">
        <div
          v-for="complain in complains"
          :key="complain.Id"
          class="complain-item"
        >
          <h3>{{ complain.Title }}</h3>
          <p>{{ complain.Body }}</p>
        </div>
      </div>
    </template>
    <!-- No complaints available message -->
    <div v-else>
      <p>No complaints available.</p>
    </div>

    <!-- Toast Notification -->
    <div v-if="toast.message" :class="['toast', toast.type]">
      {{ toast.message }}
    </div>
  </div>
</template>

<script setup>
const baseUrl = "https://sugarytestapi.azurewebsites.net/";
const listPath = "TestApi/GetComplains";
const savePath = "TestApi/SaveComplain";

const title = ref("");
const body = ref("");
const errors = ref({ title: null, body: null });
const toast = ref({ message: "", type: "" });
const submitted = ref(false);
const isSaving = ref(false);

// Fetch complaints using useFetch
const {
  data: complains,
  refresh: refreshComplains,
  status: isLoading,
} = useFetch(`${baseUrl}${listPath}`, {
  key: "fetchComplains",
  method: "GET",
  onRequestError: () => showToast("Failed to fetch complaints.", "error"),
});

// Validate form inputs
const validateForm = () => {
  if (submitted.value) {
    errors.value.title = title.value.trim() ? "" : "Title is required.";
    errors.value.body = body.value.trim() ? "" : "Complaint body is required.";
  }
  return !errors.value.title && !errors.value.body;
};

// Watch fields for validation errors after submission
watch([title, body], () => {
  if (submitted.value) validateForm();
});

// Save a new complaint using useLazyFetch
const saveComplain = async () => {
  // Mark form as submitted
  submitted.value = true;
  if (!validateForm()) return;
  // Show "Saving..." status
  isSaving.value = true;

  const { execute } = useLazyFetch(`${baseUrl}${savePath}`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: {
      Title: title.value,
      Body: body.value,
    },
    onResponse: ({ response }) => {
      if (response.ok) {
        showToast("Complaint saved successfully!", "success");
        title.value = "";
        body.value = "";
        submitted.value = false; // Reset form submission state
        refreshComplains();
      } else {
        showToast("Failed to save complaint.", "error");
      }
    },
    onResponseError: () => {
      showToast("Failed to save complaint.", "error");
    },
    immediate: false,
  });
  // Execute the POST Request
  await execute();
  // Hide "Saving..." status
  isSaving.value = false;
};

// Show toast notification
const showToast = (message, type) => {
  toast.value = { message, type };
  setTimeout(() => {
    toast.value.message = "";
  }, 3000);
};
</script>

<style>
/*============= Import Google Fonts ===========*/
@import url("https://fonts.googleapis.com/css2?family=Fira+Code:wght@300..700&display=swap");
/*============= Variables ===========*/
:root {
  --bg-color: #2c3333;
  --primery-color: #395b64;
  --secondary-color: #a5c9ca;
  --text-color: #e7f6f2d0;
  --font-family: "Fira Code", monospace;
  --error-color: rgba(243, 94, 94, 0.973);
}
/*============= Reset Browser ===========*/
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
/*============= Global Styles ===========*/
html,
body {
  background-color: var(--bg-color);
  color: var(--text-color);
  font-family: var(--font-family);
  height: 100%;
}
/*============= Global Input/Button/H2 Styles ===========*/
input,
textarea {
  outline: none;
  background: transparent;
  border: 1px solid var(--secondary-color);
  border-bottom: 4px solid var(--secondary-color);
  border-radius: 8px;
  font-family: var(--font-family);
  font-size: 16px;
  padding: 10px;
  caret-color: var(--secondary-color);
  color: var(--text-color);
}

button {
  border: 1px solid var(--secondary-color);
  font-family: var(--font-family);
  border-radius: 8px;
  padding: 10px;
  font-size: medium;
  font-weight: bold;
  color: var(--bg-color);
  cursor: pointer;
  background: var(--secondary-color);
  width: 80%;
  transition: all 0.3s ease-in-out;
}
button:hover {
  border-color: var(--secondary-color);
  color: var(--secondary-color);
  background-color: transparent;
}

h2 {
  margin: 20px 0px;
  font-size: 28px;
}
/*============= Wrapper Styles ===========*/
.wrapper {
  max-width: 800px;
  margin: 10px auto;
  @media screen and (max-width: 768px) {
    padding: 0 12px;
    margin: 5px auto;
  }
  @media screen and (max-width: 1024px) {
    padding: 0 20px;
    margin: 5px auto;
  }
}
/*============= Form Styles ===========*/
.complain-form {
  display: flex;
  justify-items: center;
  align-items: center;
  flex-direction: column;
}
.form-group > input {
  font-weight: 600;
  font-size: 18px;
}
.form-group > textarea {
  height: 130px;
  resize: none;
}
.form-group {
  display: flex;
  justify-items: center;
  align-items: left;
  flex-direction: column;
  margin-bottom: 15px;
  width: 80%;
}
.form-group input.error-input,
.form-group textarea.error-input {
  border-color: var(--error-color);
}
.error-message {
  color: var(--error-color);
  font-size: 14px;
  margin-top: 5px;
}
.complain-form button:disabled {
  background-color: #cccccc3b;
  cursor: not-allowed;
}
/* Loader */
.loader {
  text-align: center;
  font-size: 16px;
}

/*============= Complaint Item ===========*/
.complain-list-wrapper{
  padding-bottom: 20px;
}
.complain-item {
  margin-bottom: 15px;
  padding: 12px;
  border: 1px solid #395b64;
  border-radius: 8px;
  border-left: 6px solid #395b64;
  background-color: #395b6409;
}
.complain-item p {
  margin-top: 10px;
  color: #a5c9ca;
}
/*============= Toast Notification ===========*/
.toast {
  position: fixed;
  top: 50px;
  right: 50px;
  padding: 12px 20px;
  border-radius: 5px;
  color: #fff;
  font-size: 16px;
  z-index: 100;
  animation: slideIn 0.5s forwards, slideOut 0.5s 2.5s forwards;
}
@keyframes slideIn {
  from {
    transform: translateY(-100%);
  }
  to {
    transform: translateY(0);
  }
}
@keyframes slideOut {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(-100%);
  }
}
.toast.success {
  background-color: var(--primery-color);
}

.toast.error {
  background-color: var(--error-color);
}

/* ============== Scrollbar CSS ==================  */
::-webkit-scrollbar {
  width: 5px;
  height: 5px;
}
::-webkit-scrollbar-button {
  width: 2px;
  height: 2px;
}
::-webkit-scrollbar-thumb {
  background-color: #395b64;
  border-radius: 50px;
}
::-webkit-scrollbar-track {
  background-color: transparent;
  border-radius: 50px;
}
::-webkit-scrollbar-corner {
  background: transparent;
}
</style>
