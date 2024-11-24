<template>
  <div class="sidebar">
    <div class="header">
      <h2>LeetCode Questions</h2>
      <button class="add-button" @click="$emit('open-modal', 'add')">
        + Add Question
      </button>
    </div>
    <div class="questions-list">
      <div 
        v-for="question in questions" 
        :key="question.id" 
        class="question-item"
        :class="{ active: selectedQuestionId === question.id }"
        @click="selectQuestion(question)"
      >
        {{ question.title }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    questions: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      selectedQuestionId: null
    }
  },
  methods: {
    selectQuestion(question) {
      this.selectedQuestionId = question.id;
      this.$emit('select', question);
    }
  }
}
</script>

<style scoped>
.sidebar {
  border-right: 1px solid #ddd;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.header {
  padding: 1rem;
  border-bottom: 1px solid #ddd;
}

.header h2 {
  margin: 0 0 1rem 0;
}

.add-button {
  width: 100%;
  padding: 0.5rem;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.questions-list {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
}

.question-item {
  padding: 0.5rem;
  margin-bottom: 0.5rem;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.question-item:hover {
  background: #f8f9fa;
}

.question-item.active {
  background: #007bff;
  color: white;
}
</style>