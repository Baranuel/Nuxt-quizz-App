<template>
  <main class="flex justify-center items-center text-white-100">

    <div v-if="$fetchState.pending" class="absolute flex flex-col h-screen w-screen justify-center items-center">
      <div class="loader"></div>
      <h1 class="neonGreen text-2xl">Loading...</h1>
    </div>
    <div v-else class="w-5/12 game">
      <p class="neonGreen  text-lg">Question {{ questionCount }}/{{ availableQuestions.length}} </p>
      <div class="w-full bg-grey-100 my-2 h-1">
        <div :style="{ 'width' : lineWidth + '%' }" class="  selected h-1 transition-all duration-500 ease-in-out hover:bg-red-900"></div>
      </div>
      <div class="flex justify-between mb-5" >
      <p class=" text-xl text-white">Score: {{score}}</p>
      <div>
        <h1 class=" text-xl text-white">{{time}} sec</h1>
      </div>
      </div>
        <h1 v-show="visible" class="text-6xl lightGrey mb-5 "> {{ currentQuestion.question}} </h1>
      <ul class="grid grid-cols-2 neonGreen gap-2 my-15 font-sans font-medium  drop-shadow-xl" >
        <li  v-for="answer, index in currentQuestion.answers" id="answer" :key="index"  class=" my-2  pr-2 py-4 text-xl cursor-pointer" @click="checkAnswer">{{answer}}</li>
      </ul>
    </div>
  </main>
</template>

<script>
export default {
  name: 'game',
  data () {
    return {
      start: false,
      div: '',
      questionCount: 1,
      index: 0,
      apiQuestions: [],
      allQuestions: [],
      lineWidth: 100,
      availableQuestions: [],
      currentQuestion: {},
      allAnswers: [],
      isSelected: false,
      selectedAnswer: {},
      visible: true,
      score: 0,
      time: 10
    }
  },
  async fetch () {
    this.allQuestions = await fetch('https://opentdb.com/api.php?amount=100')
      .then(data => data.json())
      .then((response) => {
        this.apiQuestions = response.results.map(
          (apiQuestion) => {
            const formatedQuestion = {
              question: apiQuestion.question,
              correct_answer: apiQuestion.correct_answer,
              incorrect_answers: apiQuestion.incorrect_answers
            }
            formatedQuestion.answers = [...apiQuestion.incorrect_answers, apiQuestion.correct_answer]
            return formatedQuestion
          }
        )
        for (let i = 0; i < 10; i++) {
          this.availableQuestions.push(this.apiQuestions[i])
        }
        this.startGame()
      })
  },
  fetchDelay: 2000,
  methods: {
    startGame () {
      console.log(this.div)
      this.start = true
      this.lineWidth = (this.questionCount / 10) * (100)
      this.getCurrentQuestion(this.index)
      this.timeOut()
    },
    nextQuestion () {
      const allAnswers = document.querySelectorAll('#answer')
      allAnswers.forEach((answer) => {
        answer.classList.remove('disabled', 'wrongAnswer', 'selected', 'correct', 'noClick')
      })
      this.isSelected = false
      this.questionCount++
      this.lineWidth = (this.questionCount / 10) * (100)
      this.time = 10
      this.index++
      this.getCurrentQuestion(this.index)
    },
    getCurrentQuestion (index) {
      console.log(this.selectedAnswer)
      for (let i = 0; i < this.availableQuestions.length; i++) {
        this.currentQuestion = this.availableQuestions[index]
      }
      for (let i = this.currentQuestion.answers.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [this.currentQuestion.answers[i], this.currentQuestion.answers[j]] = [this.currentQuestion.answers[j], this.currentQuestion.answers[i]]
      }
    },
    checkAnswer (e) {
      this.selectedAnswer = e.target
      this.isSelected = true
      this.selectedAnswer.classList.add('selected', 'noClick')
      this.allAnswers = document.querySelectorAll('#answer')
      for (let i = 0; i < this.allAnswers.length; i++) {
        if (this.allAnswers[i] !== this.selectedAnswer) {
          this.allAnswers[i].classList.add('disabled')
        }
      }
      console.log(this.currentQuestion)
      if (this.selectedAnswer.innerHTML === this.currentQuestion.correct_answer) {
        this.selectedAnswer.classList.add('correct')
      } else {
        this.selectedAnswer.classList.add('wrongAnswer')
      }
      const correctAnswer = this.currentQuestion.correct_answer
      this.showCorrect(this.selectedAnswer.innerHTML.replace(/\s/g, ''), correctAnswer)
      setTimeout(this.nextQuestion, 2000)
    },
    showCorrect (myAnswer, correctAnswer) {
      if (myAnswer === null) {
        myAnswer = ''
      }
      if (myAnswer !== correctAnswer) {
        this.allAnswers.forEach((answer) => {
          if (answer.innerHTML === correctAnswer) {
            answer.classList.add('correct')
          }
        })
        this.score -= 50
      } else {
        this.score += 50
      }
    },
    timeOut () {
      setInterval(() => {
        this.time--
        this.allAnswers = document.querySelectorAll('#answer')
        if (this.time === 0) {
          for (let i = 0; i < this.allAnswers.length; i++) {
            if (this.allAnswers[i].innerHTML === this.currentQuestion.correct_answer) {
              this.allAnswers[i].classList.add('correct')
            } else { this.allAnswers[i].classList.add('disabled') }
          }
          if (this.isSelected !== true) {
            this.score -= 50
            setTimeout(this.nextQuestion, 2000)
          }
        }
      }, 1000)
    }
  }

}
</script>

<style>
:root{--neonGreen : hsl(150,100%,66%);
      --lightGrey : hsl(193,38%,86%)
}
.game{

}
.show{
  opacity:1
}
.neonGreen{
  color:var(--neonGreen)
}
.neonBlueBg{
  background:var(--neonBlue)
}
.lightGrey{
  color:var(--lightGrey)
}
.onHover{
color:var(neonGreen)
}
.onHover:hover{
  color:white;
  background-color: var(neonGreen)
}
.disabled{
  opacity: 0.25;
  pointer-events: none
}
.noClick{
  pointer-events: none
}
.selected{
  color:white;
  background-color: hsl(150,100%,66%)
}
.correct{
   color:white;
  background-color: hsl(150,100%,66%)
}
.wrongAnswer{
  background-color:red;
  z-index: 100;
  color:white
}
#answer{
  transition:background-color .1s ease;
  margin:.25rem;
  padding-left:.5rem
}
#answer:hover{

  border-left:2px solid rgb(42, 125, 84);
  z-index:-5;
}
ul{
  font-family:"Roboto"
}

main{
  background-color:hsl(218,23%,16%);
  height:100vh
}
fill{
  animation: fill 1s ease
}
.loader,
.loader:before,
.loader:after {
  background: hsl(150,100%,66%);
  -webkit-animation: load1 1s infinite ease-in-out;
  animation: load1 1s infinite ease-in-out;
  width: 1em;
  height: 4em;
}
.loader {
  color: hsl(150,100%,66%);
  text-indent: -9999em;
  margin: 88px auto;
  position: relative;
  font-size: 11px;
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
  -webkit-animation-delay: -0.16s;
  animation-delay: -0.16s;
}
.loader:before,
.loader:after {
  position: absolute;
  top: 0;
  content: '';
}
.loader:before {
  left: -1.5em;
  -webkit-animation-delay: -0.32s;
  animation-delay: -0.32s;
}
.loader:after {
  left: 1.5em;
}
@-webkit-keyframes load1 {
  0%,
  80%,
  100% {
    box-shadow: 0 0;
    height: 4em;
  }
  40% {
    box-shadow: 0 -2em;
    height: 5em;
  }
}
@keyframes load1 {
  0%,
  80%,
  100% {
    box-shadow: 0 0;
    height: 4em;
  }
  40% {
    box-shadow: 0 -2em;
    height: 5em;
  }
}

@keyframes fill {

  from {transform:rotate(0deg) }
  to {transform:rotate(360deg) }
}
</style>
