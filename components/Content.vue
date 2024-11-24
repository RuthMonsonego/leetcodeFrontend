<template>
  <div class="content">
    <div v-if="selectedQuestion" class="question-details">
      <div class="header">
        <h2>{{ selectedQuestion.title }}</h2>
        <div class="actions">
          <button class="edit" @click="$emit('open-modal', 'update')">
            Edit
          </button>
          <button class="delete" @click="confirmDelete">
            Delete
          </button>
        </div>
      </div>
      <div class="description">
        {{ selectedQuestion.description }}
      </div>
      <div v-if="selectedQuestion.parameters.length" class="parameters">
        <h3>Parameters:</h3>
        <ul>
          <li v-for="(param, index) in selectedQuestion.parameters" :key="index">
            {{ param.name }} ({{ param.type }})
          </li>
        </ul>
      </div>
    </div>
    <div v-else class="empty-state">
      <p v-if="!apiError">Select a question to view details</p>
      <p v-else class="error">Failed to load questions</p>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    selectedQuestion: {
      type: Object,
      default: null
    },
    apiError: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    confirmDelete() {
      if (confirm('Are you sure you want to delete this question?')) {
        this.$emit('delete', this.selectedQuestion.id);
      }
    }
  }
}
</script>

<style scoped>
.content {
  padding: 1rem;
  height: 100%;
  overflow-y: auto;
}

.question-details .header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.actions {
  display: flex;
  gap: 0.5rem;
}

.actions button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.edit {
  background: #007bff;
  color: white;
}

.delete {
  background: #dc3545;
  color: white;
}

.description {
  white-space: pre-wrap;
  margin-bottom: 1rem;
}

.parameters {
  background: #f8f9fa;
  padding: 1rem;
  border-radius: 4px;
}

.parameters h3 {
  margin-top: 0;
}

.empty-state {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: #6c757d;
}

.error {
  color: #dc3545;
}
</style>