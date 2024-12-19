<template>
  <div id="app" class="min-h-screen bg-gray-100">
    <header v-if="currentStudent" class="bg-white shadow p-4 flex justify-between items-center">
      <div>
        <h1 class="text-xl font-font">{{ schoolName }}</h1>
        <p class="text-gray-600 font-font">{{ className }}</p>
      </div>
      <div>
        <p class="text-gray-700 font-font">{{ currentStudent }}</p>
      </div>
    </header>

    <main class="max-w-4xl mx-auto my-6 bg-white p-6">
      <div v-if="!currentStudent">
        <h2 class="text-xl mb-4 font-font">Identifique-se</h2>
        <div class="mb-4">
          <label class="block text-gray-700">Nome da Escola</label>
          <input
            v-model="schoolName"
            placeholder="Digite o nome da escola"
            class="w-full p-2 border border-gray-300 rounded mb-4"
          />
        </div>

        <div class="mb-4">
          <label class="block text-gray-700">Turma</label>
          <input
            v-model="className"
            placeholder="Digite a turma"
            class="w-full p-2 border border-gray-300 rounded mb-4"
          />
        </div>

        <div class="mb-4">
          <label class="block text-gray-700">Nome do Aluno</label>
          <input
            v-model="studentName"
            placeholder="Digite o nome do aluno"
            class="w-full p-2 border border-gray-300 rounded mb-4"
          />
        </div>

        <button
          @click="startQuiz"
          class="w-full bg-[#1134d5] text-white font-semibold py-2 px-4 rounded hover:bg-blue-600"
        >
          Iniciar Prova
        </button>
      </div>

      <div v-else>
        <div>
          <!-- Colocando o texto primeiro -->
          <h2 class="text-lg font-bold mb-4 font-font">{{ currentQuestion.subject }} - Questão {{ currentQuestionIndex + 1 }}/{{ questions.length }}</h2>
          <h3 class="mb-4 font-font">{{ currentQuestion.text }}</h3>
          
          <!-- Imagem logo depois do texto -->
          <img
            v-if="currentQuestion.image"
            :src="currentQuestion.image"
            alt="Imagem da questão"
            class="max-w-xs h-auto mb-4 rounded"
          />

          <!-- Texto adicional após a imagem -->
          <h3 class="mb-4 font-font">{{ currentQuestion.extraText }}</h3>

          <div class="grid grid-cols-2 gap-4">
            <div v-for="(option, index) in currentQuestion.options" :key="option.id">
              <button
                :class="[ 
                  'w-full text-left p-2 rounded border', 
                  studentAnswers[currentQuestion.id]?.selected === option.id
                    ? 'bg-blue-100 border-blue-500'
                    : 'bg-white border-gray-300' 
                ]"
                :disabled="studentAnswers[currentQuestion.id] !== undefined"
                @click="checkAnswer(currentQuestion, option)"
              >
                {{ option.id }}. {{ option.text }}
              </button>
            </div>
          </div>
        </div>

        <div class="flex justify-between mt-6">
          <button
            @click="previousQuestion"
            :disabled="currentQuestionIndex === 0"
            class="bg-gray-300 text-gray-700 py-2 px-4 rounded hover:bg-gray-400 disabled:opacity-50"
          >
            Questão Anterior
          </button>
          <button
            @click="nextQuestion"
            :disabled="currentQuestionIndex === questions.length - 1"
            class="bg-gray-300 text-gray-700 py-2 px-4 rounded hover:bg-gray-400 disabled:opacity-50"
          >
            Próxima Questão
          </button>
        </div>

        <div class="mt-6">
          <button
            v-if="allQuestionsAnswered"
            @click="finishQuiz"
            class="w-full bg-green-500 text-white font-semibold py-2 px-4 rounded hover:bg-green-600"
          >
            Finalizar Prova
          </button>
          <button
            @click="logout"
            class="w-full mt-2 bg-red-500 text-white font-semibold py-2 px-4 rounded hover:bg-red-600"
          >
            Trocar de Aluno
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      schoolName: '',
      className: '',
      studentName: '',
      currentStudent: null,
      currentQuestionIndex: 0,
      questions: [
        {
          id: 1,
          subject: 'Ciências',
          text: 'A Acústica é a ciência que estuda o som e suas características, como intensidade, altura ou timbre. A imagem representa uma dessas características.',
          extraText: 'Sobre a situação apresentada, pode-se concluir que ela ocorre porque:',
          options: [
            { id: 'A', text: 'As vibrações da guitarra são lentas.' },
            { id: 'B', text: 'A intensidade do som produzido é forte.' },
            { id: 'C', text: 'O som produzido pela guitarra está alto.' },
            { id: 'D', text: 'O timbre produzido pela guitarra é intenso.' },
          ],
          correctAnswer: 'B',
          comments: 'BNCC: EF03CI01 - A questão avalia a habilidade de interpretar as propriedades do som.',
          image: '/img-question.png',  // Caminho para a imagem
        },
        {
          id: 2,
          subject: 'Matemática',
          text: 'João tem 3 caixas com 12 lápis em cada uma. Quantos lápis João tem ao todo?',
          extraText: 'Quantos lápis João tem ao todo?',
          options: [
            { id: 'A', text: '36' },
            { id: 'B', text: '24' },
            { id: 'C', text: '30' },
            { id: 'D', text: '40' },
          ],
          correctAnswer: 'A',
          comments: 'BNCC: EF02MA03 - A questão avalia a habilidade de resolver problemas de multiplicação simples.',
     
        },
      ],
      studentAnswers: {},
    };
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
    allQuestionsAnswered() {
      return Object.keys(this.studentAnswers).length === this.questions.length;
    },
  },
  methods: {
    startQuiz() {
      if (this.schoolName.trim() === '' || this.className.trim() === '' || this.studentName.trim() === '') {
        alert('Por favor, preencha todos os campos.');
        return;
      }
      this.currentStudent = this.studentName.trim();
      localStorage.setItem('currentStudent', this.currentStudent);
      localStorage.setItem('schoolName', this.schoolName);
      localStorage.setItem('className', this.className);
      this.loadStudentAnswers(this.currentStudent);
    },
    loadStudentAnswers(student) {
      const savedAnswers = localStorage.getItem(`answers_${student}`);
      this.studentAnswers = savedAnswers ? JSON.parse(savedAnswers) : {};
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
    downloadAnswersAsJson() {
      const fileName = `${this.currentStudent}_respostas.json`;
      const dataToDownload = {
        schoolName: this.schoolName,
        className: this.className,
        studentName: this.currentStudent,
        answers: this.studentAnswers,
      };
      const jsonBlob = new Blob([JSON.stringify(dataToDownload, null, 2)], { type: 'application/json' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(jsonBlob);
      link.download = fileName;
      link.click();
    },
    logout() {
      this.currentStudent = null;
      this.schoolName = '';
      this.className = '';
      this.studentName = '';
      this.studentAnswers = {};
      localStorage.removeItem('currentStudent');
      localStorage.removeItem('schoolName');
      localStorage.removeItem('className');
    },
    nextQuestion() {
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
      }
    },
    previousQuestion() {
      if (this.currentQuestionIndex > 0) {
        this.currentQuestionIndex--;
      }
    },
  },
};
</script>
