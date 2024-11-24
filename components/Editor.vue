<template>
  <div class="editor-container">
    <div class="editor-header">
      <div class="language-selector">
        <button 
          :class="{ active: selectedLanguage === 'go' }" 
          @click="selectLanguage('go')"
        >
          Go
        </button>
        <button 
          :class="{ active: selectedLanguage === 'python' }" 
          @click="selectLanguage('python')"
        >
          Python
        </button>
      </div>
      <button 
        class="run-button" 
        @click="handleRunCode" 
        :disabled="!questionId || isRunning"
      >
        {{ isRunning ? 'Running...' : 'Run Code' }}
      </button>
    </div>
    
    <div class="editor-main">
      <textarea 
        v-model="userCode" 
        :placeholder="currentTemplate"
        :disabled="!questionId || isRunning"
        class="code-editor"
        @keydown.tab.prevent="handleTab"
      ></textarea>
    </div>

    <div v-if="output" class="output-section">
      <div class="output-header">
        <h3>Output:</h3>
        <button class="clear-button" @click="clearOutput">Clear</button>
      </div>
      <pre class="output-content" :class="{ error: isError }">{{ output }}</pre>
    </div>

    <div class="parameters-section" v-if="parameters.length">
      <h3>Input Parameters:</h3>
      <div v-for="(param, index) in parameters" :key="index" class="parameter-input">
        <label>{{ param.name }} ({{ param.type }}):</label>
        <input 
          v-if="param.type.endsWith('[]')"
          v-model="parameterValues[index]"
          type="text"
          :placeholder="'Enter array values [1,2,3]'"
          @input="handleInputChange(index, $event.target.value)"
        />
        <input 
          v-else
          v-model="parameterValues[index]"
          :type="getInputType(param.type)"
          :placeholder="`Enter ${param.type} value`"
          @input="handleInputChange(index, $event.target.value)"
        />
        <span class="error-message" v-if="parameterErrors[index]">
          {{ parameterErrors[index] }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    goTemplate: {
      type: String,
      default: ''
    },
    pythonTemplate: {
      type: String,
      default: ''
    },
    questionId: {
      type: Number,
      default: null
    },
    parameters: {
      type: Array,
      default: () => []
    }
  },

  data() {
    return {
      userCode: '',
      selectedLanguage: 'go',
      output: '',
      isRunning: false,
      isError: false,
      parameterValues: [],
      parameterErrors: []
    }
  },

  computed: {
    currentTemplate() {
      return this.selectedLanguage === 'go' ? this.goTemplate : this.pythonTemplate;
    }
  },

  methods: {
    selectLanguage(lang) {
      this.selectedLanguage = lang;
      this.userCode = this.currentTemplate;
      this.clearOutput();
    },

    getInputType(paramType) {
      const baseType = paramType.replace('[]', '');
      switch(baseType) {
        case 'int':
        case 'float':
        case 'double':
          return 'number';
        case 'bool':
          return 'checkbox';
        case 'char':
          return 'text';
        default:
          return 'text';
      }
    },

    parseParameterValue(value, type) {
      if (value === '' || value === null || value === undefined) {
        return type.endsWith('[]') ? [] : null;
      }

      // Handle array types
      if (type.endsWith('[]')) {
        if (!value.startsWith('[') || !value.endsWith(']')) {
          throw new Error(`Array value must be in format [x,y,z] for type ${type}`);
        }
        
        const baseType = type.slice(0, -2);
        const arrayStr = value.slice(1, -1).trim();
        if (!arrayStr) return [];
        
        const arrayValues = arrayStr.split(',').map(v => v.trim());
        return arrayValues.map(v => this.parseSingleValue(v, baseType));
      }

      return this.parseSingleValue(value, type);
    },

    parseSingleValue(value, type) {
      switch(type) {
        case 'int':
          const intVal = parseInt(value);
          if (isNaN(intVal)) throw new Error('Invalid integer value');
          return intVal;
        case 'float':
        case 'double':
          const floatVal = parseFloat(value);
          if (isNaN(floatVal)) throw new Error('Invalid floating point value');
          return floatVal;
        case 'bool':
          if (typeof value === 'boolean') return value;
          const strVal = String(value).toLowerCase();
          if (!['true', 'false', '0', '1'].includes(strVal)) {
            throw new Error('Invalid boolean value');
          }
          return ['true', '1'].includes(strVal);
        case 'char':
          if (String(value).length !== 1) {
            throw new Error('Char must be a single character');
          }
          return String(value)[0];
        case 'string':
          return String(value);
        default:
          return value;
      }
    },

    handleInputChange(index, value) {
      this.parameterValues[index] = value;
      this.validateParameter(index);
    },

    validateParameter(index) {
      try {
        const param = this.parameters[index];
        const value = this.parameterValues[index];
        this.parseParameterValue(value, param.type);
        this.parameterErrors[index] = '';
      } catch (error) {
        this.parameterErrors[index] = error.message;
      }
    },

    validateAllParameters() {
      let isValid = true;
      this.parameters.forEach((param, index) => {
        try {
          const value = this.parameterValues[index];
          this.parseParameterValue(value, param.type);
          this.parameterErrors[index] = '';
        } catch (error) {
          this.parameterErrors[index] = error.message;
          isValid = false;
        }
      });
      return isValid;
    },

    async handleRunCode() {
      if (!this.questionId || this.isRunning) return;
      
      if (!this.validateAllParameters()) {
        this.output = 'Please fix parameter errors before running';
        this.isError = true;
        return;
      }
      
      try {
        this.isRunning = true;
        this.clearOutput();

        const args = this.parameters.map((param, index) => {
          const rawValue = this.parameterValues[index];
          return this.parseParameterValue(rawValue, param.type);
        });
        
        const requestData = {
          user_execution_code: this.userCode,
          language: this.selectedLanguage,
          arguments: args,
          question_code: this.questionId
        };
        
        this.$emit('execute', requestData, (response) => {
          console.log('Execution response:', response);
          if (response && response.output) {
            this.output = response.output;
            this.isError = false;
          } else {
            this.output = 'No output received';
            this.isError = true;
          }
        });

      } catch (error) {
        console.error('Run code error:', error);
        this.output = `Error: ${error.message}`;
        this.isError = true;
      } finally {
        this.isRunning = false;
      }
    },

    clearOutput() {
      this.output = '';
      this.isError = false;
    },

    handleTab(e) {
      const start = e.target.selectionStart;
      const end = e.target.selectionEnd;
      
      this.userCode = this.userCode.substring(0, start) + '    ' + this.userCode.substring(end);
      
      this.$nextTick(() => {
        e.target.selectionStart = e.target.selectionEnd = start + 4;
      });
    }
  },

  watch: {
    currentTemplate: {
      handler(newVal) {
        if (newVal && !this.userCode) {
          this.userCode = newVal;
        }
      },
      immediate: true
    },
    questionId() {
      this.clearOutput();
    },
    parameters: {
      handler(newParams) {
        this.parameterValues = new Array(newParams.length).fill('');
        this.parameterErrors = new Array(newParams.length).fill('');
      },
      immediate: true
    }
  }
}
</script>

<style scoped>
.editor-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  border-left: 1px solid #ddd;
}

.editor-header {
  display: flex;
  justify-content: space-between;
  padding: 1rem;
  border-bottom: 1px solid #ddd;
}

.language-selector button {
  margin-right: 0.5rem;
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
}

.language-selector button.active {
  background: #007bff;
  color: white;
  border-color: #0056b3;
}

.run-button {
  padding: 0.5rem 1rem;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.run-button:disabled {
  background: #6c757d;
  cursor: not-allowed;
  opacity: 0.7;
}

.editor-main {
  flex: 1;
  padding: 1rem;
}

.code-editor {
  width: 100%;
  height: 100%;
  font-family: 'Fira Code', monospace;
  font-size: 14px;
  line-height: 1.5;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  resize: none;
  tab-size: 4;
}

.code-editor:disabled {
  background: #f8f9fa;
  cursor: not-allowed;
}

.output-section {
  border-top: 1px solid #ddd;
  padding: 1rem;
  max-height: 30%;
  overflow-y: auto;
}

.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.clear-button {
  padding: 0.25rem 0.5rem;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.clear-button:hover {
  background: #c82333;
}

.output-content {
  margin: 0;
  white-space: pre-wrap;
  font-family: 'Fira Code', monospace;
  font-size: 14px;
  background: #f8f9fa;
  padding: 1rem;
  border-radius: 4px;
}

.output-content.error {
  color: #dc3545;
  background: #f8d7da;
}

.parameters-section {
  padding: 1rem;
  border-top: 1px solid #ddd;
}

.parameter-input {
  margin-bottom: 1rem;
}

.parameter-input label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
}

.parameter-input input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error-message {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.25rem;
  display: block;
}

@media (max-width: 768px) {
  .editor-container {
    height: 50vh;
  }
}
</style>