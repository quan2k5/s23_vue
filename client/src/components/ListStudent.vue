<template>
  <div class="container">
    <main class="main-content">
      <header class="header">
        <h3>Student Management</h3>
        <button class="btn-add" @click="showStudentForm">
          Add New Student
        </button>
      </header>
      <div class="search-bar">
        <input
          v-model="searchTerm"
          class="search-input"
          placeholder="Search by student name"
        />
        <i class="icon-refresh" @click="resetSearch">⟳</i>
      </div>
      <table class="student-table">
        <thead>
          <tr>
            <th>#</th>
            <th>Student Name</th>
            <th>Email</th>
            <th>Address</th>
            <th colspan="2">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(student, idx) in searchResults" :key="student.id">
            <td>{{ idx + 1 }}</td>
            <td>{{ student.name }}</td>
            <td>{{ student.email }}</td>
            <td>{{ student.address }}</td>
            <td><button @click="editStudentDetails(student)" class="btn-edit">Edit</button></td>
            <td><button @click="promptDeleteStudent(student)" class="btn-delete">Delete</button></td>
          </tr>
        </tbody>
      </table>
    </main>

    <!-- Form Modal for Adding/Editing Student -->
    <div v-if="isFormVisible" class="modal-overlay">
      <form @submit.prevent="handleSubmit" class="form-modal">
        <div class="modal-header">
          <h4>{{ isEditMode ? 'Edit Student' : 'Add Student' }}</h4>
          <span class="close-icon" @click="closeForm">✖</span>
        </div>
        <div class="form-group">
          <label for="name">Student Name</label>
          <input v-model="studentForm.name" id="name" type="text" class="form-input" />
          <small v-if="formErrors.name" class="error-message">{{ formErrors.name }}</small>
        </div>
        <div class="form-group">
          <label for="email">Email</label>
          <input v-model="studentForm.email" id="email" type="email" class="form-input" />
          <small v-if="formErrors.email" class="error-message">{{ formErrors.email }}</small>
        </div>
        <div class="form-group">
          <label for="address">Address</label>
          <input v-model="studentForm.address" id="address" type="text" class="form-input" />
          <small v-if="formErrors.address" class="error-message">{{ formErrors.address }}</small>
        </div>
        <button type="submit" class="btn-submit">{{ isEditMode ? 'Save Changes' : 'Add Student' }}</button>
      </form>
    </div>
    <div v-if="isDeleteModalVisible" class="modal-overlay">
      <div class="confirmation-modal">
        <h4>Warning</h4>
        <p>Are you sure you want to delete this student?</p>
        <button @click="closeDeleteModal" class="btn-cancel">Cancel</button>
        <button @click="confirmDelete" class="btn-confirm">Confirm</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

const studentList = ref([]);
const searchTerm = ref("");
const isFormVisible = ref(false);
const isDeleteModalVisible = ref(false);
const isEditMode = ref(false);
const studentForm = ref({ name: "", email: "", address: "" });
const selectedStudent = ref(null);
const formErrors = ref({});

const loadStudents = async () => {
  try {
    const response = await axios.get("http://localhost:3000/students");
    studentList.value = response.data;
  } catch (error) {
    console.error(error);
  }
};

const searchResults = computed(() => {
  if (!searchTerm.value) return studentList.value;
  return studentList.value.filter((student) =>
    student.name.toLowerCase().includes(searchTerm.value.toLowerCase())
  );
});

const validateForm = () => {
  formErrors.value = {};
  if (!studentForm.value.name) formErrors.value.name = "Student name is required.";
  if (!studentForm.value.email || !studentForm.value.email.includes("@"))
    formErrors.value.email = "Invalid email format.";
  if (!studentForm.value.address) formErrors.value.address = "Address is required.";
  return Object.keys(formErrors.value).length === 0;
};

const addNewStudent = async () => {
  try {
    const response = await axios.post("http://localhost:3000/students", studentForm.value);
    studentList.value.push(response.data);
    closeForm();
  } catch (error) {
    console.error(error);
  }
};

const updateStudentInfo = async () => {
  try {
    await axios.put(`http://localhost:3000/students/${studentForm.value.id}`, studentForm.value);
    const idx = studentList.value.findIndex((student) => student.id === studentForm.value.id);
    if (idx !== -1) studentList.value[idx] = { ...studentForm.value };
    closeForm();
  } catch (error) {
    console.error(error);
  }
};

const handleSubmit = () => {
  if (validateForm()) {
    if (isEditMode.value) {
      updateStudentInfo();
    } else {
      addNewStudent();
    }
  }
};

const showStudentForm = () => {
  isFormVisible.value = true;
  isEditMode.value = false;
  studentForm.value = { name: "", email: "", address: "" };
};

const closeForm = () => {
  isFormVisible.value = false;
  formErrors.value = {};
};

const editStudentDetails = (student) => {
  studentForm.value = { ...student };
  isEditMode.value = true;
  isFormVisible.value = true;
};

const promptDeleteStudent = (student) => {
  selectedStudent.value = student;
  isDeleteModalVisible.value = true;
};

const confirmDelete = async () => {
  try {
    await axios.delete(`http://localhost:3000/students/${selectedStudent.value.id}`);
    studentList.value = studentList.value.filter((s) => s.id !== selectedStudent.value.id);
    closeDeleteModal();
  } catch (error) {
    console.error(error);
  }
};

const closeDeleteModal = () => {
  isDeleteModalVisible.value = false;
  selectedStudent.value = null;
};

const resetSearch = () => {
  searchTerm.value = "";
};

onMounted(() => {
  loadStudents();
});
</script>

<style>
.container {
  width: 80%;
  margin: auto;
  margin-top: 20px;
  height: 100vh;
}

.main-content {
  padding: 10px;
}

.header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.btn-add {
  background-color: blue;
  color: white;
  padding: 8px 15px;
}

.search-bar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 15px;
}

.search-input {
  width: 350px;
  padding: 5px;
  margin-right: 10px;
}

.student-table {
  width: 100%;
  border-collapse: collapse;
}

.student-table th,
.student-table td {
  border: 1px solid #ddd;
  padding: 8px;
}

.btn-edit {
  color: green;
}

.btn-delete {
  color: red;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.form-modal {
  background-color: white;
  padding: 20px;
  width: 400px;
}

.close-icon {
  cursor: pointer;
}

.error-message {
  color: red;
}

.confirmation-modal {
  background-color: white;
  padding: 20px;
  width: 300px;
  text-align: center;
}

.btn-cancel {
  margin-right: 10px;
}
</style>
