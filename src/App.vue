<template>
  <div id="app" class="min-h-screen bg-gray-100">
    <header
      v-if="currentStudent"
      class="bg-white shadow p-4 flex justify-between items-center"
    >
      <div>
        <h1 class="text-xl font-font">{{ schoolName }}</h1>
        <p class="text-gray-600 font-font font-bold">{{ className }}</p>
      </div>
      <div>
        <p class="text-gray-700 font-font">{{ currentStudent }}</p>
      </div>
    </header>

    <main class="max-w-4xl mx-auto my-6 bg-white p-6">
      <div v-if="!currentStudent">
        <h1 class="text-xl mb-4 font-font text-center font-bold tracking-wider">Identifique-se</h1>
        <div class="mb-4">
          <label class="font-font block text-black font-bold tracking-wide">Nome da Escola:</label>
          <input
            v-model="schoolName"
            placeholder="Digite o nome da escola"
            class="w-full p-2 border border-gray-300 rounded mb-4"
          />
        </div>

        <div class="mb-4">
          <label class=" font-font block text-black font-bold tracking-wide">Turma:</label>
          <input
            v-model="className"
            placeholder="Digite a turma"
            class="w-full p-2 border border-gray-300 rounded mb-4"
          />
        </div>

        <div class="mb-4">
          <label class="font-font block text-black font-bold tracking-wide">Nome do Aluno:</label>
          <input
            v-model="studentName"
            placeholder="Digite o nome do aluno"
            class="w-full p-2 border border-gray-300 rounded mb-4"
          />
        </div>

        <button
          @click="startQuiz"
          class="flex items-center rounded-md bg-[#1134d5] py-2 px-4 border border-transparent text-sm text-white transition-all shadow-sm hover:shadow-lg focus:bg-slate-700 focus:shadow-none active:bg-slate-700 hover:bg-blue-700 active:shadow-none disabled:pointer-events-none disabled:opacity-50 disabled:shadow-none font-font font-bold tracking-wider"
        >
          Iniciar Prova
        </button>
      </div>

      <div v-else>
        <!-- Colocando o texto primeiro -->
        <div>
          <h2 class="text-lg font-bold mb-4 font-font tracking-wider">
            {{ currentQuestion.subject }} - Questão
            {{ currentQuestionIndex + 1 }}/{{ questions.length }}
          </h2>
          <h3 class="mb-4 font-font tracking-wide">{{ currentQuestion.text }}</h3>
          <!-- imagem depois do texto-->
          <img
            v-if="currentQuestion.image"
            :src="currentQuestion.image"
            alt="Imagem da questão"
            class="max-w-xs h-auto mb-4 rounded"
          />
          <!-- Texto adicional após a imagem -->
          <h3 class="mb-4 font-font tracking-wide">{{ currentQuestion.extraText }}</h3>

          <div class="grid grid-cols-2 gap-4 font-font tracking-wide">
            <div
              v-for="(option, index) in currentQuestion.options"
              :key="option.id"
            >
              <button
                :class="[
                  'w-full text-left p-2 rounded border',
                  studentAnswers[currentQuestion.id]?.confirmed &&
                  studentAnswers[currentQuestion.id].selected === option.id
                    ? 'bg-blue-100 border-blue-500'
                    : studentAnswers[currentQuestion.id]?.confirmed
                    ? 'bg-gray-200 border-gray-400'
                    : selectedOption === option.id
                    ? 'bg-blue-100 border-blue-500'
                    : 'bg-white border-gray-300',
                ]"
                :disabled="studentAnswers[currentQuestion.id]?.confirmed"
                @click="selectAnswer(option)"
              >
                {{ option.id }}. {{ option.text }}
              </button>
            </div>
          </div>

          <div class="mt-6 flex justify-center font-font">
            <button
              @click="confirmAnswer"
              :disabled="
                studentAnswers[currentQuestion.id]?.confirmed || !selectedOption
              "
              :class="[
                'w-full max-w-sm py-2 px-4 font-semibold rounded',
                studentAnswers[currentQuestion.id]?.confirmed
                  ? 'bg-blue-300 text-white cursor-not-allowed'
                  : 'bg-blue-500 hover:bg-blue-600 text-white',
              ]"
            >
              <!-- Ícone de check quando a questão estiver respondida -->
              <span
                v-if="studentAnswers[currentQuestion.id]?.confirmed"
                class="mr-2"
              >
                <i class="fas fa-check-circle"></i>
              </span>
              {{
                studentAnswers[currentQuestion.id]?.confirmed
                  ? "Questão Respondida"
                  : "Confirmar Resposta"
              }}
            </button>
          </div>
        </div>

        <div class="flex justify-between mt-6 font-font">
          <button
            @click="previousQuestion"
            :disabled="currentQuestionIndex === 0"
            class="flex items-center space-x-2 bg-gray-300 text-gray-700 py-2 px-4 rounded hover:bg-gray-400 disabled:opacity-50"
          >
            <i class="fas fa-arrow-left"></i>
            <span>Questão Anterior</span>
          </button>

          <button
            @click="nextQuestion"
            :disabled="currentQuestionIndex === questions.length - 1"
            class="flex items-center space-x-2 bg-gray-300 text-gray-700 py-2 px-4 rounded hover:bg-gray-400 disabled:opacity-50"
          >
            <span>Próxima Questão</span>
            <i class="fas fa-arrow-right"></i>
          </button>
        </div>

        <div class="flex justify-between mt-6">
          <button
            v-if="allQuestionsAnswered"
            @click="finishQuiz"
            class="bg-green-500 text-white font-semibold py-2 px-4 rounded hover:bg-green-600"
          >
            Finalizar Prova
          </button>

          <button
            @click="logout"
            class="bg-red-500 text-white font-semibold py-2 px-4 rounded hover:bg-red-600"
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
      schoolName: "",
      className: "",
      studentName: "",
      currentStudent: null,
      currentQuestionIndex: 0,
      selectedOption: null,
      questions: [
        {
          id: 1,
          subject: "Ciências",
          text: "A Acústica é a ciência que estuda o som e suas características, como intensidade, altura ou timbre. A imagem representa uma dessas características.",
          extraText:
            "Sobre a situação apresentada, pode-se concluir que ela ocorre porque:",
          options: [
            { id: "A", text: "As vibrações da guitarra são lentas." },
            { id: "B", text: "A intensidade do som produzido é forte." },
            { id: "C", text: "O som produzido pela guitarra está alto." },
            { id: "D", text: "O timbre produzido pela guitarra é intenso." },
          ],
          correctAnswer: "B",
          image: "/img-question.png",
        },
        {
          id: 2,
          subject: "Matemática",
          text: "João tem 3 caixas com 12 lápis em cada uma...",
          extraText: "Quantos lápis João tem ao todo?",
          options: [
            { id: "A", text: "36" },
            { id: "B", text: "24" },
            { id: "C", text: "30" },
            { id: "D", text: "40" },
          ],
          correctAnswer: "A",
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
      if (
        this.schoolName.trim() === "" ||
        this.className.trim() === "" ||
        this.studentName.trim() === ""
      ) {
        alert("Por favor, preencha todos os campos.");
        return;
      }
      this.currentStudent = this.studentName.trim();
      localStorage.setItem("currentStudent", this.currentStudent);
      localStorage.setItem("schoolName", this.schoolName);
      localStorage.setItem("className", this.className);
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
    selectAnswer(option) {
      this.selectedOption = option.id;
    },
    confirmAnswer() {
      if (this.selectedOption) {
        const isCorrect =
          this.selectedOption === this.currentQuestion.correctAnswer;
        this.studentAnswers[this.currentQuestion.id] = {
          selected: this.selectedOption,
          confirmed: true,
          isCorrect,
        };
        this.saveStudentAnswers();
        this.selectedOption = null;
      }
    },
    finishQuiz() {
      alert("Prova finalizada! Suas respostas foram salvas.");
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
      const jsonBlob = new Blob([JSON.stringify(dataToDownload, null, 2)], {
        type: "application/json",
      });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(jsonBlob);
      link.download = fileName;
      link.click();
    },
    logout() {
      this.currentStudent = null;
      this.schoolName = "";
      this.className = "";
      this.studentName = "";
      this.studentAnswers = {};
      localStorage.removeItem("currentStudent");
      localStorage.removeItem("schoolName");
      localStorage.removeItem("className");
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
  mounted() {
    const student = localStorage.getItem("currentStudent");
    if (student) {
      this.currentStudent = student;
      this.schoolName = localStorage.getItem("schoolName");
      this.className = localStorage.getItem("className");
      this.loadStudentAnswers(student);
    }
  },
};
</script>
