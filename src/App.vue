<template>
    <main id="App"  class="component" :class="'all-answered-'+allAnswered" :totalAnswers="totalAnswers">
        <header class="primary-header">
            <p v-if="!answerMode">For each cost item below, select the best classification to describe the cost. More than one classification may apply to the same cost item.</p>
            <p v-if="answerMode">
                Out of {{totalAnswers}}, you got {{yourAnswers.correct}} right, {{yourAnswers.incorrect}} wrong and you missed {{yourAnswers.missed}}
            </p>
        </header>
        <section class="questions">
            <header class="question-header">
                <div class="answer-heading-list">
                    <div class="answer-header-label" v-for="answer in answers">{{answer}}</div>                    
                </div>

            </header>
            <div class="question-list">
                <question v-for="(question, index) in questions" 
                      :questionText="question.questionText" 
                      :possibleAnswers="answers" 
                      :correctAnswerIndices="question.correctAnswerIndices" 
                      :questionIndex="index"
                      :answerMode="answerMode"></question>
            </div>
            <footer class="question-footer">
                <button class="button" v-if="!answerMode" @click="enterCheckAnswerMode()">Check Answers</button>    
                <button class="button" v-if="answerMode" @click="resetQuiz()">Reset and try again</button>    
            </footer>
       </section>
    </main>
</template>

<script>
import eventHub from './eventHub.js';
import Question from './components/Question.vue';
import data from './content/_data.js';
export default {
    name: 'app', 
    components: { Question  },
    data () { 
        let quiz = data;
            quiz.answerMode = false,
            quiz.answeredQuestions = {}, 
            quiz.allAnswered = false, 
            quiz.yourAnswers = {
                correct: 0,
                missed: 0,
                incorrect: 0
            }
        return quiz
    }, 
    computed: {
        totalAnswers() {
            let totalAnswerSum = 0;
            for (var i = this.questions.length - 1; i >= 0; i--) {
                totalAnswerSum = totalAnswerSum + this.questions[i].correctAnswerIndices.length;
                // console.log("LOL WE LOOPING FUCK YOU CUNT")
                // console.log(this.questions[i].correctAnswerIndices.length)
            }
            return totalAnswerSum
        }
    },
    methods: { 
        enterCheckAnswerMode () {
            if (this.allAnswered == true) {
                this.answerMode = true;    
            }
            else {alert("Please answer all the questions first!")}
        },
        resetQuiz() {
            this.answerMode = false;
            this.yourAnswers.correct = 0;
            this.yourAnswers.missed = 0;
            this.yourAnswers.incorrect = 0
        }
     },
    mounted () {
        eventHub.$on('answerCountUpdated', (data) => {
            if (data.answerCount > 0) {
                this.answeredQuestions[data.questionIndex] = true;
            } else if (data.answerCount == 0) {
                this.answeredQuestions[data.questionIndex] = false;
            }
            let qtyAnswers = Object.keys(this.answeredQuestions).length,
                qtyQuestions = this.questions.length;
            if (qtyAnswers == qtyQuestions) {
                console.log("We might be nearing the finish line, one last check")
                for (var o in this.answeredQuestions) {
                    if (this.answeredQuestions[o]) {
                        this.allAnswered = true;
                    }
                    else {this.allAnswered = false}
                } 
            }
        }),
        eventHub.$on('correctAnswer', () => {
            this.yourAnswers.correct ++ 
        }),
        eventHub.$on('incorrectAnswer', () => {
            this.yourAnswers.incorrect ++ 
        }),
        eventHub.$on('missedAnswer', () => {
            this.yourAnswers.missed ++ 
        })

    }
}
</script>

<style type="text/css">
    .all-answered-false button {
        cursor: not-allowed;
        opacity: 0.5;
    }
    .all-answered-true button {
        cursor: pointer;
    }
</style>
