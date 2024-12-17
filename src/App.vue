<template>
  <div id="app">
    <div v-if="!currentStudent">
      <h1>Identifique-se</h1>
      <input v-model="studentName" placeholder="Digite seu nome" />
      <button @click="startQuiz">Iniciar Quiz</button>
    </div>
    <div v-else>
      <h1>Quiz: Responda às questões</h1>
      <h2>Aluno: {{ currentStudent }}</h2>
      <div v-for="(question, index) in questions" :key="question.id" class="question">
        <h2>{{ index + 1 }}. {{ question.text }}</h2>
        <div v-for="option in question.options" :key="option.id">
          <button 
            :class="{ selected: studentAnswers[question.id]?.selected === option.id }"
            :disabled="studentAnswers[question.id] !== undefined"
            @click="checkAnswer(question, option)">
            {{ option.text }}
          </button>
        </div>
      </div>
      <button v-if="allQuestionsAnswered" @click="finishQuiz">Finalizar Prova</button>
      <button @click="logout">Trocar de Aluno</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      studentName: '',
      currentStudent: null, // Identifica o aluno atual
      questions: [
        {
          id: 1,
          text: 'Qual é a capital do Brasil?',
          options: [
            { id: 1, text: 'São Paulo' },
            { id: 2, text: 'Brasília' },
            { id: 3, text: 'Rio de Janeiro' },
          ],
          correctAnswer: 2,
        },
        {
          id: 2,
          text: 'Qual é a capital da França?',
          options: [
            { id: 1, text: 'Berlim' },
            { id: 2, text: 'Madri' },
            { id: 3, text: 'Paris' },
          ],
          correctAnswer: 3,
        },
      ],
      studentAnswers: {}, // Respostas do aluno atual
    };
  },
  computed: {
    allQuestionsAnswered() {
      return Object.keys(this.studentAnswers).length === this.questions.length;
    },
  },
  mounted() {
    this.loadCurrentStudent();
  },
  methods: {
    loadCurrentStudent() {
      const savedStudent = localStorage.getItem('currentStudent');
      if (savedStudent) {
        this.currentStudent = savedStudent;
        this.loadStudentAnswers(savedStudent);
      }
    },
    startQuiz() {
      if (this.studentName.trim() === '') {
        alert('Por favor, digite seu nome.');
        return;
      }
      this.currentStudent = this.studentName.trim();
      localStorage.setItem('currentStudent', this.currentStudent);
      this.loadStudentAnswers(this.currentStudent);
    },
    loadStudentAnswers(student) {
      const savedAnswers = localStorage.getItem(`answers_${student}`);
      if (savedAnswers) {
        this.studentAnswers = JSON.parse(savedAnswers);
      } else {
        this.studentAnswers = {};
      }
    },
    saveStudentAnswers() {
      localStorage.setItem(
        `answers_${this.currentStudent}`,
        JSON.stringify(this.studentAnswers)
      );
    },
    checkAnswer(question, option) {
      const isCorrect = option.id === question.correctAnswer;
      this.studentAnswers[question.id] = { selected: option.id, isCorrect };
      this.saveStudentAnswers();
    },
    finishQuiz() {
      alert('Prova finalizada! Suas respostas foram salvas.');
      this.downloadAnswersAsJson();
    },
    logout() {
      this.currentStudent = null;
      this.studentName = '';
      this.studentAnswers = {};
      localStorage.removeItem('currentStudent');
    },
    downloadAnswersAsJson() {
      const fileName = `${this.currentStudent}_respostas.json`;
      const dataToDownload = {
        studentName: this.currentStudent, // Inclui o nome do aluno
        answers: this.studentAnswers,     // Inclui as respostas
      };
      const jsonBlob = new Blob([JSON.stringify(dataToDownload, null, 2)], { type: 'application/json' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(jsonBlob);
      link.download = fileName;
      link.click();
    }
  },
};
</script>


<style>
#app {
  text-align: center;
  padding: 20px;
  font-family: Arial, sans-serif;
}

input {
  margin: 10px;
  padding: 10px;
  font-size: 16px;
  width: 250px;
}

button {
  margin: 10px;
  padding: 10px;
  font-size: 18px;
  cursor: pointer;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

h1, h2 {
  color: #333;
}

.question {
  margin: 20px 0;
}

.selected {
  background-color: #4caf50;
  color: white;
}
</style>

