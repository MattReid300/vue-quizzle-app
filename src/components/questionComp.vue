<template>
  <div>
    <div class="config-screen" v-if="!initialised">
      <h1>Quizzle</h1>
      <h2>Category</h2>
      <div class="categories">
        <label for="categories">Category: </label>
        <select
          name="categories"
          id="categories-select"
          v-model="chosenCategory"
        >
          <option
            class="category-button"
            :class="{ 'is-selected': chosenCategory == category.id }"
            v-for="category in categories"
            :key="category.id"
            :value="category.id"
          >
            {{ category.name }}
          </option>
        </select>
        <br />
        <br />
      </div>
      <h2>Types</h2>
      <div class="types">
        <input
          type="checkbox"
          id="true-false"
          value="TRUE-FALSE"
          v-model="boolean"
        />
        <label for="true-false">True/False Questions</label><br />
        <input
          type="checkbox"
          id="mutiple-choice"
          value="MULTIPLE-CHOICE"
          v-model="multipleChoice"
        />
        <label for="multiple-choice">Multiple Choice Questions</label><br />
      </div>
      <h2>Length</h2>
      <div class="length">
        <div
          class="length-button"
          :class="{ 'is-selected': chosenLength == length }"
          v-for="length in lengths"
          :key="length.value"
        >
          <input
            type="radio"
            name="choose-length"
            v-model="chosenLength"
            :value="length"
          />
          <label>{{ length }} questions</label>
        </div>
      </div>
      <button class="start" @click="startQuiz()">Start Quiz</button>
    </div>
    <div class="quiz-screen" v-if="initialised">
      <!-- <div v-if="questions.length == 0">LOADING I GUESS</div> -->
      <div
        class="quiz"
        v-if="currentQuestionIndex < chosenLength"
        :key="currentQuestionIndex"
      >
        <h2>Questions</h2>
        <h3>{{ decipher(questions[currentQuestionIndex]?.text) }}</h3>
        <div class="answers-container">
          <div
            class="answer answer--option"
            v-for="(answer, index) in questions[currentQuestionIndex]?.answers"
            @click="selectAnswer(index)"
            :class="{
              'is-selected': chosenAnswers[currentQuestionIndex] == index,
            }"
            :key="index"
          >
            {{ answer.text }}
          </div>
        </div>
      </div>
      <div v-else :key="currentQuestionIndex" class="quiz-completed">
        <h2>Game Over</h2>
        <h2>You scored {{ calcScore() }} / {{ chosenLength }}</h2>
        <div
          class="quiz-answer"
          v-for="(question, index) in questions"
          :class="{
            'is-selected': chosenAnswers[currentQuestionIndex] == index,
          }"
          :key="index"
        >
          <h3 class="question">{{ decipher(question.text) }}</h3>
          <p class="answer answer--correct">
            Correct answer:
            {{ question.answers.find((a) => a.correct == true).text }}
          </p>
        </div>
        <button @click="reset()">Play again</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      // fuck the difficulty, Ima add number of questions instead (5, 10 or 20), maybe even style of questions (true false or multiple choice)
      initialised: false,
      categories: [],
      chosenCategory: null,
      lengths: [5, 10, 20],
      chosenLength: 0,
      questions: [],
      boolean: true,
      multipleChoice: true,
      chosenAnswers: [],
      currentQuestionIndex: 0,
    };
  },
  mounted() {
    this.setup();
  },
  methods: {
    setup() {
      const url = "https://opentdb.com/api_category.php";
      axios
        .get(url)
        .then((response) => {
          this.categories = response.data.trivia_categories;
        })
        .catch((error) => {
          console.log(error);
          alert("Categories failed to load");
        });
    },
    setLength(length) {
      this.chosenLength = length;
    },
    setType() {
      if (this.boolean && !this.multipleChoice) {
        return `boolean`;
      } else if (!this.boolean && this.multipleChoice) {
        return `multiple`;
      }
    },
    startQuiz() {
      // alert(this.chosenCategory + this.chosenLength);
      const url = this.createUrl();
      this.initialised = true;
      axios
        .get(url)
        .then((response) => {
          this.generateQuestions(response.data.results);
          // console.log(this.questions);
        })
        .catch((error) => {
          console.log(error);
          alert("Categories failed to loadad");
        });
    },
    createUrl() {
      let categoryParam =
        this.chosenCategory == null ? "" : `&category=${this.chosenCategory}`;

      let lengthParam =
        this.chosenLength == null ? "" : `amount=${this.chosenLength}`;

      let specific = this.setType();
      //let specific = [];
      //specific.push(Object.values(this.typeObject).includes(true));
      let typeParam =
        this.boolean && this.multipleChoice ? "" : `&type=${specific}`;
      return `https://opentdb.com/api.php?${lengthParam}${categoryParam}${typeParam}&difficulty=easy`;
    },
    generateQuestions(response) {
      if (response.length > 0) {
        response.forEach((questionData) => {
          const newQuestion = {}; // Question object
          const answers = []; // Answers array

          [
            ...questionData.incorrect_answers, //Establish params from api
            questionData.correct_answer,
          ].forEach((answer) => {
            let answerObject = { text: "", correct: false }; // Two properties to each answer

            [answerObject.text, answerObject.correct] = [
              answer,
              answer == questionData.correct_answer, // Assign correct answer
            ];
            answers.push(answerObject);
          });

          newQuestion.text = questionData.question;
          newQuestion.answers = answers;
          this.questions.push(newQuestion); // Push newQuestion object to questions array in data
        });
      } else {
        alert("something went wrong");
      }
    },
    //add index param to method
    selectAnswer(index) {
      //let variable = (this.chosenAnswers[this.currentQuestionIndex] = index);

      this.chosenAnswers[this.currentQuestionIndex] = index;
      this.currentQuestionIndex++;
    },
    calcScore() {
      let total = 0;

      this.chosenAnswers.forEach((answer, index) => {
        if (
          typeof this.questions[index].answers[this.chosenAnswers[index]] !==
            "undefined" &&
          this.questions[index].answers[this.chosenAnswers[index]].correct
        ) {
          total += 1;
        }
      });

      return total;
    },
    answerCorrect(question, answerIndex) {
      return question.answers[answerIndex].correct;
      // question.answers.find(a => a.correct == true);
    },
    reset() {
      this.currentQuestionIndex = 0;
      this.chosenAnswers = [];
      this.chosenCategory = null;
      this.questions = [];
      this.initialised = false;
    },
    decipher(html) {
      const textArea = document.createElement("textarea");
      textArea.innerHTML = html;
      return textArea.value;
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../styles.scss";
</style>