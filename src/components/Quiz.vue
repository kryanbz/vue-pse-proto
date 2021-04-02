<template>
<div>
  <b-container >
    <b-container class="bv-example-row">
      <b-row>
        <b-col>     
          <img id="logo-crown" src="@/assets/arder.png" alt="UB Logo" />
<br>
          <b-dropdown id="dropdown-1" text="Categories" class="m-md-2">
          <b-dropdown-item v-on:click="isShown = true">Algebra</b-dropdown-item>
          <b-dropdown-item disabled>Geometry</b-dropdown-item>
          <b-dropdown-item disabled>Fractions/Decimals</b-dropdown-item>
          <b-dropdown-item disabled>Computation</b-dropdown-item>
          <b-dropdown-item disabled>Concept</b-dropdown-item>
          </b-dropdown>



        </b-col>
        <b-col> 
          <br>
          <h1 id="logo-headline">PSE Quiz App  </h1> 
        </b-col>
        <b-col>





        </b-col>
      </b-row>
    </b-container> 
  </b-container>



    <b-container >
       <b-row>
    <b-col >   
      
      
      
       <div id="quiz-1" v-if="isShown">
    
  <button  @click=fetchQuestions(0)>Algebra Quiz 1</button>
</div>
<div id="quiz-1" v-if="isShown">
  <button  @click=fetchQuestions(1)>Algebra Quiz 2</button>
</div>
</b-col>

    <b-col >
    <div id="quiz-container">

     <!-- <h1 v-html="loading ? ' ' : currentQuestion.question"></h1> -->
  
    
    <img v-if="clickNext" v-bind:src="loading ? ' ' : currentQuestion.question">
    </div> 
	
  

<div id="quiz-container">

     <!-- <h1 v-html="loading ? ' ' : currentQuestion.question"></h1> -->
  
    
    <img v-if="clickShow" v-bind:src="loading ? ' ' : previousQuestion.question">
    <img v-if="clickShow" v-bind:src="loading ? ' ' : previousQuestion.solution">
    <youtube v-if="clickShow" v-bind:video-id="loading ? ' ' : previousQuestion.videoId"></youtube>
    </div> 








 </b-col>



    <b-col >    <hr class="divider" />
    <div>
     
      <form v-if="currentQuestion && clickNext">
        <button
          v-for="answer in currentQuestion.answers"
          :index="currentQuestion.key"
          :key="answer"
          v-html="answer"
          @click.prevent="handleButtonClick"
          
        ></button>

      </form>
      <hr class="divider" />
      <div v-if="showButtons">
        <center> <b-button v-on:click="clickNext = true, clickShow = false, showButtons = false" >Next</b-button></center>

        
        <center> <b-button v-on:click="clickShow = true, showButtons = true" >Explanation</b-button></center>
      </div>
 <div id="quiz-container">
    


    <!-- New Section for User Statistics -->
    <div class="correctAnswers">
      You have
      <strong>{{ correctAnswers }} correct {{ pluralizeAnswer }}!</strong>
    </div>
    <div class="correctAnswers">
      Currently at question {{ index + 1 }} of {{ questions.length }}
    </div>


  </div>


    </div>
    </b-col>
  </b-row> 
	
</b-container>
</div>
  
</template>

<script>
export default {
  name: "Quiz",
  data() {
    return {
      questions: [],
      loading: true,
      index: 0,
      isShown: false,
      clickNext: true,
      clickShow: false,
      showButtons: false,
      
      
    qids: [
      { id: 0, text: "606649d0ef3fe6309c1830e4" },
      { id: 1, text: "606649d0ef3fe6309c1830e4" }
    ]

    

    };
  },
  
  computed: {
    currentQuestion() {
      if (this.questions !== []) {
        return this.questions[this.index];
        
      }
      return null;
    },

    previousQuestion() {
      if (this.questions !== []) {
        return this.questions[this.index - 1];
        
      }
      return null;
    },

    score() {
      if (this.questions !== []) {
        // Here, we want to collect data in an object about the users statistics - later be emitted on an event when users finishes quiz
        return {
          allQuestions: this.questions.length,
          answeredQuestions: this.questions.reduce((count, currentQuestion) => {
            if (currentQuestion.userAnswer) {
              // userAnswer is set when user has answered a question, no matter if right or wrong
              count++;
            }
            return count;
          }, 0),
          correctlyAnsweredQuestions: this.questions.reduce(
            (count, currentQuestion) => {
              if (currentQuestion.rightAnswer) {
                // rightAnswer is true, if user answered correctly
                count++;
              }
              return count;
            },
            0
          ),
        };
      } else {
        return {
          allQuestions: 0,
          answeredQuestions: 0,
          correctlyAnsweredQuestions: 0,
        };
      }
    },
    correctAnswers() {
      if (this.questions && this.questions.length > 0) {
        let streakCounter = 0;
        this.questions.forEach(function(question) {
          if (!question.rightAnswer) {
            return;
          } else if (question.rightAnswer === true) {
            streakCounter++;
          }
        });
        return streakCounter;
      } else {
        return "--";
      }
    },
    pluralizeAnswer() {
      // For grammatical correctness
      return this.correctAnswers === 1 ? "Answer" : "Answers";
    },
    quizCompleted() {
      if (this.questions.length === 0) {
        return false;
      }
      /* Check if all questions have been answered */
      let questionsAnswered = 0;
      this.questions.forEach(function(question) {
        question.rightAnswer !== null ? questionsAnswered++ : null;
      });
      return questionsAnswered === this.questions.length;
    },
  },
  watch: {
    quizCompleted(completed) {
      /*
       * Watcher on quizCompleted fires event "quiz-completed"
       * up to parent App.vue component when completed parameter
       * returned by quizCompleted computed property true
       */
      completed &&
        setTimeout(() => {
          this.$emit("quiz-completed", this.score);
        }, 2000); // wait 3 seconds until button animation is over
    },
  },
  methods: {
    async fetchQuestions(qid) {
        let  qids = [
      { id: 0, text: "606649d0ef3fe6309c1830e4" },
      { id: 1, text: "606649d0ef3fe6309c1830e4" }
    ];
    
      this.qid = this.qids[qid];
      

      let x = (qids[qid].text);
      let url = "http://localhost:3333/questions/" + x;
      console.log(url)
      this.loading = true;
      let response = await fetch(
        url
      );
      let jsonResponse = await response.json();
      let index = 0; // index is used to identify single answer
      let data = jsonResponse.results.map((question) => {
        // put answers on question into single array
        question.answers = [
          question.correct_answer,
          ...question.incorrect_answers,
        ];
        /* Shuffle question.answers array */
        for (let i = question.answers.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [question.answers[i], question.answers[j]] = [
            question.answers[j],
            question.answers[i],
          ];
        }
        // mention in Step 1
        question.rightAnswer = null;
        question.key = index;
        index++;
        return question;
      });
      this.questions = data;
      this.loading = false;
    },
    handleButtonClick: function(event) {
      /* Find index to identiy question object in data */
      let index = event.target.getAttribute("index");
      let pollutedUserAnswer = event.target.innerHTML; // innerHTML is polluted with decoded HTML entities e.g ' from &#039;
      /* Clear from pollution with ' */
      let userAnswer = pollutedUserAnswer.replace(/'/, "&#039;");

      /* Set userAnswer on question object in data */
      this.questions[index].userAnswer = userAnswer;

      /* Set class "clicked" on button with userAnswer -> for CSS Styles; Disable other sibling buttons */
      event.target.classList.add("clicked");
      let allButtons = document.querySelectorAll(`[index="${index}"]`);

      for (let i = 0; i < allButtons.length; i++) {
        if (allButtons[i] === event.target) continue;

        allButtons[i].setAttribute("disabled", "");
      }

      /* Invoke checkAnswer to check Answer */
      this.checkAnswer(event, index);
      
    },
    checkAnswer: function(event, index) {
    
      let question = this.questions[index];

      if (question.userAnswer) {
        console.log(this.clickNext)
        console.log(this.previousQuestion)
        if (this.index < this.questions.length - 1) {
          setTimeout(
            function() {
              this.clickNext = false;
              this.clickShow = false;
              this.showButtons = true;
              this.index += 1;
            }.bind(this),
            2000
          );
          
        }
        if (question.userAnswer === question.correct_answer) {
          /* Set class on Button if user answered right, to celebrate right answer with animation joyfulButton */
          event.target.classList.add("rightAnswer");
          /* Set rightAnswer on question to true, computed property can track a streak out of 10 questions */
          this.questions[index].rightAnswer = true;
        } else {
          /* Mark users answer as wrong answer */
          event.target.classList.add("wrongAnswer");
          this.questions[index].rightAnswer = false;
          /* Show right Answer */
          let correctAnswer = this.questions[index].correct_answer;
          let allButtons = document.querySelectorAll(`[index="${index}"]`);
          allButtons.forEach(function(button) {
            if (button.innerHTML === correctAnswer) {
              button.classList.add("showRightAnswer");
            }
          });
        }
      }
      
      

    },
   
     


  },
 // mounted() {
   // this.fetchQuestions();
 // },



};
</script>

<style scoped>
#quiz-container {
  margin: 1rem auto;
  padding: 1rem;
  max-width: 750px;
  
}

#logo-headline {
  font-size: 2rem;
  padding: 0.5rem;
  color: #d65212;
  text-align: center;
  font-family: 'Spartan', sans-serif;
}

#logo-crown {
  display: block;
  width: 35%;
  float: left;
  margin: .2rem auto;
  padding: .2rem;
}

@media only screen and (max-width: 500px) {
  #logo-crown {
    width: 20%;
  }

  #logo-headline {
    font-size: .6rem;
    text-align: right;
  }
}

h1 {
  font-size: .5rem;
  padding: 0.7rem;
}

.divider {
  margin: 0.5rem 0;
  border: 3px solid rgba(125 45 145, 0.7);
  border-radius: 2px;
  box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.3);
}

form {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
}

button {
  font-size: 1.1rem;
  box-sizing: border-box;
  padding: 1rem;
  margin: 0.3rem;
  width: 47%;
  background-color: rgba(100, 100, 100, 0.3);
  border: none;
  border-radius: 0.4rem;
  box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
}

button:hover:enabled {
  transform: scale(1.02);
  box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.14), 0 1px 7px 0 rgba(0, 0, 0, 0.12),
    0 3px 1px -1px rgba(0, 0, 0, 0.2);
}

button:focus {
  outline: none;
}

button:active:enabled {
  transform: scale(1.05);
}

@keyframes flashButton {
  0% {
    opacity: 1;
    transform: scale(1.01);
  }
  50% {
    opacity: 0.7;
    transform: scale(1.02);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

button.clicked {
  pointer-events: none;
}

button.rightAnswer {
  animation: flashButton;
  animation-duration: 700ms;
  animation-delay: 200ms;
  animation-iteration-count: 3;
  animation-timing-function: ease-in-out;
  color: black;
  background: linear-gradient(
    210deg,
    rgba(0, 178, 72, 0.25),
    rgba(0, 178, 72, 0.5)
  );
}

button.wrongAnswer {
  color: black;
  background: linear-gradient(
    210deg,
    rgba(245, 0, 87, 0.25),
    rgba(245, 0, 87, 0.5)
  );
}

button.showRightAnswer {
  animation: flashButton;
  animation-duration: 700ms;
  animation-delay: 200ms;
  animation-iteration-count: 2;
  animation-timing-function: ease-in-out;
  color: black;
  background: linear-gradient(
    210deg,
    rgba(0, 178, 72, 0.25),
    rgba(0, 178, 72, 0.5)
  );
}

.correctAnswers {
  text-align: center;
}

#quiz-1 {
  margin: 0 auto;
}
</style>
