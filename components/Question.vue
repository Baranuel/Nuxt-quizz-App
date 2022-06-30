<template>
  <main>
    <div v-for=" question,index in questions " :key="index">
      <h1>{{ question.question }}</h1>
      <ul v-for="incorrectAnswer,index in question.incorrect_answers" :key="index">
        <li>{{ incorrectAnswer }}</li>
      </ul>
    </div>
  </main>
</template>

<script>

export default {
  name: 'Question',
  data () {
    return {
      questions: [],
      loading: true
    }
  },
  created () {
    this.getQuestions()
  },
  methods: {
    async getQuestions () {
      await fetch('https://opentdb.com/api.php?amount=10')
        .then(response => response.json())
        .then((loadedQuestions) => {
          this.questions = loadedQuestions.results.map(
            (loadedQuestion) => {
              const formattedQuestion = {
                question: loadedQuestion.question
              }
              return formattedQuestion
            }
          )
        })
      console.log(this.questions)
    }
  }
}
</script>

<style>

</style>
