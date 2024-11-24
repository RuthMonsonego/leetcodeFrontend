<template>
  <div class="dashboard">
    <Sidebar 
      :questions="questions" 
      @select="selectQuestion" 
      @open-modal="openModal" 
    />
    <Content 
      :selectedQuestion="selectedQuestion" 
      :apiError="apiError" 
      @open-modal="openModal" 
      @delete="deleteQuestion" 
    />
    <Editor 
      :goTemplate="selectedQuestion?.template_for_go || ''" 
      :pythonTemplate="selectedQuestion?.template_for_python || ''"
      :questionId="selectedQuestion?.id"
      :parameters="selectedQuestion?.parameters || []"
      @execute="executeCode"
    />
    <Modal 
      :show="showModal" 
      :type="modalType" 
      :questionData="selectedQuestion"
      @close="closeModal" 
      @submit="submitForm" 
    />
  </div>
</template>

<script>
import Sidebar from '../components/Sidebar.vue'
import Content from '../components/Content.vue'
import Editor from '../components/Editor.vue'
import Modal from '../components/Modal.vue'

const API_BASE_URL = process.env.VUE_APP_API_URL || 'http://localhost:8080';

async function fetchWithConfig(url, options = {}) {
  const config = {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers
    }
  };

  const response = await fetch(`${API_BASE_URL}${url}`, config);
  const data = await response.json();
  
  if (!response.ok) {
    throw new Error(data.error || 'API request failed');
  }
  
  return data;
}

export default {
  name: 'Dashboard',
  components: {
    Sidebar,
    Content,
    Editor,
    Modal
  },
  data() {
    return {
      questions: [],
      selectedQuestion: null,
      showModal: false,
      modalType: '',
      apiError: false,
      isLoading: false
    }
  },
  async mounted() {
    await this.fetchQuestions();
  },
  methods: {
    async fetchQuestions() {
      try {
        this.isLoading = true;
        const questions = await fetchWithConfig('/questions');
        this.questions = questions;
        this.apiError = false;
      } catch (error) {
        await this.handleApiError(error, 'fetch questions');
      } finally {
        this.isLoading = false;
      }
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
      if (this.modalType === 'add') {
        this.selectedQuestion = null;
      }
    },

    async submitForm(formData) {
      try {
        this.isLoading = true;
        const method = this.modalType === 'add' ? 'POST' : 'PUT';
        const url = this.modalType === 'add' 
          ? '/questions' 
          : `/questions/${this.selectedQuestion.id}`;
        
        await fetchWithConfig(url, {
          method,
          body: JSON.stringify(formData)
        });

        await this.fetchQuestions();
        
        if (this.modalType === 'update') {
          const updatedQuestion = this.questions.find(q => q.id === this.selectedQuestion.id);
          this.selectQuestion(updatedQuestion);
        }
        
        this.closeModal();
      } catch (error) {
        await this.handleApiError(error, 'submit form');
      } finally {
        this.isLoading = false;
      }
    },

    async deleteQuestion(id) {
      try {
        this.isLoading = true;
        await fetchWithConfig(`/questions/${id}`, { method: 'DELETE' });
        this.selectedQuestion = null;
        await this.fetchQuestions();
      } catch (error) {
        await this.handleApiError(error, 'delete question');
      } finally {
        this.isLoading = false;
      }
    },

    async executeCode(codeData, callback) {
      if (!this.selectedQuestion) return;
      
      try {
        this.isLoading = true;
        
        const requestBody = {
          user_execution_code: codeData.user_execution_code,
          language: codeData.language,
          question_code: this.selectedQuestion.id,
          arguments: codeData.arguments
        };

        const response = await fetch(`${API_BASE_URL}/questions/${this.selectedQuestion.id}/test`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(requestBody)
        });

        const data = await response.json();
        console.log('Raw API Response:', data);

        if (!response.ok) {
          throw new Error(data.error || data.message || 'Failed to execute code');
        }

        callback(data);
        
      } catch (error) {
        console.error('Execute code error:', error);
        callback({ error: error.message });
      } finally {
        this.isLoading = false;
      }
    },

    async handleApiError(error, action) {
      console.error(`Failed to ${action}:`, error);
      this.apiError = true;
      const errorMessage = error.response?.data?.error || error.message || `Failed to ${action}`;
      alert(errorMessage);
    }
  }
}
</script>

<style scoped>
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 1fr;
  height: 100vh;
  overflow: hidden;
}

.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.loading-spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>