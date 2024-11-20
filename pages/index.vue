<template>
  <div class="dashboard">
    <div class="sidebar">
      <ul>
        <li v-for="question in questions" :key="question.id" @click="selectQuestion(question)">
          {{ question.title }}
        </li>
      </ul>
      <button @click="openModal('add')">Add</button>
    </div>
    <div class="content">
      <div v-if="selectedQuestion">
        <h2>{{ selectedQuestion.title }}</h2>
        <p>{{ selectedQuestion.description }}</p>
        <button @click="openModal('update')">Update</button>
        <button @click="deleteQuestion(selectedQuestion.id)">Delete</button>
      </div>
    </div>
    <div class="editor">
      <textarea v-model="userCode" placeholder="Enter your code here"></textarea>
      <button @click="runCode">Run</button>
    </div>
    <nuxt-ui-modal v-if="showModal" @close="closeModal">
      <div v-if="modalType === 'add' || modalType === 'update'">
        <input v-model="formData.title" placeholder="Title" />
        <input v-model="formData.description" placeholder="Description" />
        <button @click="submitForm">{{ modalType === 'add' ? 'Add' : 'Update' }}</button>
      </div>
    </nuxt-ui-modal>
  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [],
      selectedQuestion: null,
      userCode: '',
      showModal: false,
      modalType: '',
      formData: {
        title: '',
        description: ''
      }
    }
  },
  async mounted() {
    await this.fetchQuestions();
  },
  methods: {
    async fetchQuestions() {
      const response = await fetch('http://localhost:8080/questions');
      this.questions = await response.json();
    },
    selectQuestion(question) {
      this.selectedQuestion = question;
    },
    openModal(type) {
      this.modalType = type;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    async submitForm() {
      const method = this.modalType === 'add' ? 'POST' : 'PUT';
      const url = this.modalType === 'add' ? 'http://localhost:8080/questions' : `http://localhost:8080/questions/${this.selectedQuestion.id}`;
      await fetch(url, {
        method,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(this.formData)
      });
      this.closeModal();
      await this.fetchQuestions();
    },
    async deleteQuestion(id) {
      await fetch(`http://localhost:8080/questions/${id}`, { method: 'DELETE' });
      await this.fetchQuestions();
    },
    runCode() {
      // Implement code execution logic
    }
  }
}
</script>

<style>
.dashboard {
  display: flex;
  height: 100vh;
  font-family: Arial, sans-serif;
}

.sidebar {
  background-color: #34495e;
  color: white;
  width: 20%;
  padding: 20px;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
}

.sidebar ul {
  list-style-type: none;
  padding: 0;
}

.sidebar li {
  margin: 10px 0;
  cursor: pointer;
  transition: background-color 0.3s;
}

.sidebar li:hover {
  background-color: #2c3e50;
}

.content {
  background-color: #ecf0f1;
  width: 40%;
  padding: 20px;
  overflow-y: auto;
}

.editor {
  background-color: #bdc3c7;
  width: 40%;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

textarea {
  width: 100%;
  height: 200px;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  resize: none;
}

button {
  background-color: #3498db;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #2980b9;
}
</style>
