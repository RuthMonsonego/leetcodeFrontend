<template>
  <div v-if="show" class="modal-overlay">
    <div class="modal-content">
      <div class="modal-header">
        <h2>{{ type === 'add' ? 'Add New Question' : 'Update Question' }}</h2>
        <button class="close-button" @click="$emit('close')">×</button>
      </div>

      <div class="modal-body">
        <div class="form-group">
          <label>Function Name:</label>
          <input 
            v-model="formData.title" 
            placeholder="Enter question title"
            required
          />
          <span class="error" v-if="errors.title">{{ errors.title }}</span>
        </div>

        <div class="form-group">
          <label>Description:</label>
          <textarea 
            v-model="formData.description" 
            placeholder="Enter question description"
            required
          ></textarea>
          <span class="error" v-if="errors.description">{{ errors.description }}</span>
        </div>

        <div class="form-group">
          <label>Go Template:</label>
          <textarea 
            v-model="formData.template_for_go" 
            placeholder="Enter Go code template"
            required
          ></textarea>
          <span class="error" v-if="errors.template_for_go">{{ errors.template_for_go }}</span>
        </div>

        <div class="form-group">
          <label>Python Template:</label>
          <textarea 
            v-model="formData.template_for_python" 
            placeholder="Enter Python code template"
            required
          ></textarea>
          <span class="error" v-if="errors.template_for_python">{{ errors.template_for_python }}</span>
        </div>

        <div class="parameters-section">
          <div class="parameters-header">
            <h3>Parameters</h3>
            <button class="add-param-button" @click="addParameter">+ Add Parameter</button>
          </div>
          
          <div v-for="(param, index) in formData.parameters" :key="index" class="parameter-item">
            <div class="parameter-inputs">
              <div class="form-group">
                <label>Name:</label>
                <input 
                  v-model="param.name"
                  placeholder="Parameter name"
                  required
                />
              </div>
              
              <div class="form-group">
                <label>Type:</label>
                <select v-model="param.type" required>
                  <option value="int">Integer</option>
                  <option value="double">Double</option>
                  <option value="float">Float</option>
                  <option value="string">String</option>
                  <option value="char">Char</option>
                  <option value="bool">Boolean</option>
                  <option value="int[]">Integer Array</option>
                  <option value="double[]">Double Array</option>
                  <option value="float[]">Float Array</option>
                  <option value="string[]">String Array</option>
                  <option value="char[]">Char Array</option>
                  <option value="bool[]">Boolean Array</option>
                </select>
              </div>
              
              <div class="form-group">
                <label>Position:</label>
                <input 
                  type="number"
                  v-model.number="param.position"
                  min="0"
                  required
                />
              </div>
            </div>
            
            <button class="remove-param-button" @click="removeParameter(index)">×</button>
          </div>
        </div>
      </div>

      <div class="modal-footer">
        <button class="cancel" @click="$emit('close')">Cancel</button>
        <button class="submit" @click="submit">
          {{ type === 'add' ? 'Add' : 'Update' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    show: {
      type: Boolean,
      required: true
    },
    type: {
      type: String,
      required: true
    },
    questionData: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      formData: {
        title: '',
        description: '',
        template_for_go: '',
        template_for_python: '',
        parameters: []
      },
      errors: {
        title: '',
        description: '',
        template_for_go: '',
        template_for_python: '',
        parameters: ''
      }
    }
  },
  watch: {
    questionData: {
      handler(newVal) {
        if (newVal && this.type === 'update') {
          this.formData = { ...newVal };
        } else {
          this.resetForm();
        }
      },
      immediate: true
    }
  },
  methods: {
    addParameter() {
      this.formData.parameters.push({
        name: '',
        type: 'int',
        position: this.formData.parameters.length
      });
    },

    removeParameter(index) {
      this.formData.parameters.splice(index, 1);
      this.formData.parameters.forEach((param, idx) => {
        param.position = idx;
      });
    },

    validateForm() {
      let isValid = true;
      this.errors = {
        title: '',
        description: '',
        template_for_go: '',
        template_for_python: '',
        parameters: ''
      };

      if (!this.formData.title.trim()) {
        this.errors.title = 'Title is required';
        isValid = false;
      }

      if (!this.formData.description.trim()) {
        this.errors.description = 'Description is required';
        isValid = false;
      }

      if (!this.formData.template_for_go.trim()) {
        this.errors.template_for_go = 'Go template is required';
        isValid = false;
      }

      if (!this.formData.template_for_python.trim()) {
        this.errors.template_for_python = 'Python template is required';
        isValid = false;
      }

      if (this.formData.parameters.length > 0) {
        const invalidParams = this.formData.parameters.some(param => 
          !param.name || !param.type || param.position < 0
        );
        
        if (invalidParams) {
          this.errors.parameters = 'All parameter fields are required';
          isValid = false;
        }

        const paramNames = this.formData.parameters.map(p => p.name);
        if (new Set(paramNames).size !== paramNames.length) {
          this.errors.parameters = 'Parameter names must be unique';
          isValid = false;
        }
      }

      return isValid;
    },

    submit() {
      if (this.validateForm()) {
        this.$emit('submit', { ...this.formData });
      }
    },

    resetForm() {
      this.formData = {
        title: '',
        description: '',
        template_for_go: '',
        template_for_python: '',
        parameters: []
      };
      this.errors = {
        title: '',
        description: '',
        template_for_go: '',
        template_for_python: '',
        parameters: ''
      };
    }
  }
}
</script>

<style scoped>
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
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 8px;
  width: 90%;
  max-width: 600px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  padding: 1rem;
  border-bottom: 1px solid #ddd;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.close-button {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
}

.modal-body {
  padding: 1rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-group textarea {
  height: 100px;
  font-family: monospace;
}

.parameters-section {
  margin-top: 1.5rem;
  border-top: 1px solid #ddd;
  padding-top: 1rem;
}

.parameters-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.add-param-button {
  background: #28a745;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
}

.parameter-item {
  display: flex;
  gap: 1rem;
  align-items: flex-start;
  margin-bottom: 1rem;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 4px;
}

.parameter-inputs {
  flex: 1;
  display: grid;
  grid-template-columns: 2fr 1fr 1fr;
  gap: 1rem;
}

.remove-param-button {
  background: #dc3545;
  color: white;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 1.2rem;
}

select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: white;
}

.error {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

.modal-footer {
  padding: 1rem;
  border-top: 1px solid #ddd;
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
}

.modal-footer button {
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
}

.cancel {
  background: #6c757d;
  color: white;
  border: none;
}

.submit {
  background: #007bff;
  color: white;
  border: none;
}
</style>