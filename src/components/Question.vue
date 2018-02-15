<template>
    <div class="question">
        <h2 class="question-text">{{questionText}}</h2>
        <div class="answer-list">
            <answer v-for="(answer, index) in possibleAnswers" 
                     :index="index" 
                     :answer="answer" 
                     :uuid="questionIndex + '-' + index" 
                     :correctAnswerIndices="correctAnswerIndices"
                     :answerMode="answerMode"
                     :questionIndex="questionIndex"
                     ></answer>    
        </div>
        
    </div>
</template>

<script>
    import eventHub from '../eventHub.js';
    import Answer from './Answer.vue';
    export default {
        name: 'Question',
        components: { Answer },
        props: {
            questionText: String,
            questionIndex: Number,
            correctAnswerIndices: Array,
            possibleAnswers: Array,
            answerMode: Boolean
        },
        data () {
            let model = {
                answerCount: 0
            }; 
            return model
        },
        mounted() {
            // When an answer is checked (in Answer.vue), we emit an event called "answerChecked"
            eventHub.$on('answerChecked', (data) => {
                if(data.questionIndex == this.questionIndex && data.isChecked != null){
                    this.answerCount ++
                } else if (data.questionIndex == this.questionIndex && data.isChecked == false) {
                    this.answerCount --
                }
            })
        },
        watch: {
            answerCount() {
                let payload = {
                    questionIndex: this.questionIndex,
                    answerCount: this.answerCount
                }
                eventHub.$emit('answerCountUpdated', payload);
            }
        }
        // data () {
        // }
        // data: hasAtLeastOneAnswer true/false. 
        // this holds an array of the selected answers. when they change, we emit an event to App, simply returning the questionID and true or false depending on whether or not the array is empty or not
    }
</script>
<style type="text/css">
    
</style>

