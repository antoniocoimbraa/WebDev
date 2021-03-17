<template>
	<div id="quiz-container">
		<img id="neec-logo" src="../../public/Logo_NEEC_Blue.png" />
		<h1 id="logo-headline">Web-Dev Quiz</h1>
		<!-- div#correctAnswers -->
		<h1 id="question-number"></h1>
		<h1 id="count-down-timer"></h1>
		<hr class="divider" />
		<input
			id="start-quiz"
			type="button"
			@click.prevent="startQuiz"
			value="Start Quiz"
		/>
		<button id="custom-quiz" @click.prevent="customQuizInput">
			Custom Quiz
		</button>
		<input id="file-input" type="file" accept=".txt" hidden="true" />
		<button id="choose-file" hidden="true">Upload Your Custom Quiz</button>
		<input id="submit" type="submit" hidden="true" />
		<div id="quiz" hidden="true">
			<h1 v-html="loading ? 'Loading...' : currentQuestion.question"></h1>
			<!-- here we use the ternary operator to check the loading property -->
			<form v-if="currentQuestion">
				<button
					v-for="answer in currentQuestion.answers"
					:index="currentQuestion.key"
					:key="answer"
					v-html="answer"
					@click.prevent="handleButtonClick"
				></button>
			</form>
		</div>
		<h1 id="score" hidden="true"></h1>
		<hr class="divider" />
	</div>
</template>


<script>
export default {
	name: "Quiz",
	// data() function stores state variables
	data() {
		return {
			questions: [],
			loading: true,
			playing: true,
			index: 0, // initialize index at 0
			score: 0,
			timer: 30,
		};
	},
	computed: {
		// this computed property returns the current question at the current index
		currentQuestion() {
			if (this.questions !== []) {
				// "this" usually refers to the Vue component instance properties, e.g. "this.questions" points to the
				return this.questions[this.index]; // questions array in our data() function
			}
			return null;
		},
	},
	// All methods of a Vue component are written on the methods property of the compenent instance
	// Custom methods of the Vue component
	methods: {
		//this method is used to fetch the questions, manipulate them, and store them in the questions array
		async fetchQuestions() {
			this.loading = true;

			/* // fetch the questions
		let response = await fetch("https://opentdb.com/api.php?amount=10&category=9");

		// convert questions to json
		let jsonResponse = await response.json(); */

			let jsonResponse = require("../../public/Quiz.json");

			let index = 0; // index is used to identify single answer

			// manipulate questions
			let data = jsonResponse.results.map((question) => {
				// put answers on question into single array
				question.answers = [
					question.correct_answer,
					...question.incorrect_answers,
				];

				// shuffle question.answers array
				for (let i = question.answers.length - 1; i > 0; i--) {
					const j = Math.floor(Math.random() * (i + 1));
					[question.answers[i], question.answers[j]] = [
						question.answers[j],
						question.answers[i],
					];
				}

				// add rightAnswer and key property to each question
				question.rightAnswer = null;
				question.key = index;
				index++;
				return question;
			});

			/////////
			let arr = [];
			for (let i = 0; i < data.length; i++) {
				arr.push(i);
			}
			var currentIndex = arr.length,
				temporaryValue,
				randomIndex;

			// While there remain elements to shuffle...
			while (0 !== currentIndex) {
				// Pick a remaining element...
				randomIndex = Math.floor(Math.random() * currentIndex);
				currentIndex -= 1;

				// And swap it with the current element.
				temporaryValue = arr[currentIndex];
				arr[currentIndex] = arr[randomIndex];
				arr[randomIndex] = temporaryValue;
			}
			/////////
			var newdata = [];

			for (let i = 0; i < arr.length; i++) {
				newdata.push(data[arr[i]]);
			}
			//console.log("previous data: " + data);
			//console.log("new data: " + newdata);

			// put data on questions property
			this.questions = newdata;
			//console.log(this.questions);

			// shuffle questions
			/* for (let i = this.questions.length - 1; i > 0; i--) {
				const k = Math.floor(Math.random() * (i + 1));
				[this.questions[i], this.questions[k]] = [
					this.questions[k],
					this.questions[i],
				];
			} */
			// this.questions.sort(() => Math.random() - 0.5);
			// console.log(Math.random() - 0.5);
			// console.log(this.questions);

			this.loading = false;
			document.getElementById("question-number").innerHTML =
				"Question number " +
				(this.index + 1) +
				"/" +
				this.questions.length;
		},
		handleButtonClick: function (event) {
			// find index to identify question object in data
			let index = this.index; //event.target.getAttribute("index");

			// innerHTML is polluted with decoded HTML entities e.g ' from &#039;
			let pollutedUserAnswer = event.target.innerHTML;

			// clear from pollution with '
			let userAnswer = pollutedUserAnswer.replace(/'/, "&#039;");

			// set userAnswer on question object in data
			this.questions[index].userAnswer = userAnswer;

			// set class "clicked" on button with userAnswer -> for CSS Styles
			// disable other sibling buttons
			event.target.classList.add("clicked");
			let allButtons = document.querySelectorAll(`[index="${index}"]`);

			for (let i = 0; i < allButtons.length; i++) {
				if (allButtons[i] === event.target) continue;

				allButtons[i].setAttribute("disabled", "");
			}

			// Invoke checkAnswer to check Answer
			this.checkAnswer(event, index);
		},
		checkAnswer: function (event, index) {
			let question = this.questions[index];
			//console.log("Questions here: " + JSON.stringify(this.questions) + "\nIndex here: " + this.index);

			if (question.userAnswer) {
				if (this.index < this.questions.length - 1) {
					setTimeout(
						function () {
							this.index += 1;
							document.getElementById(
								"question-number"
							).innerHTML =
								"Question number " +
								(this.index + 1) +
								"/" +
								this.questions.length;
						}.bind(this),
						1000
					);
				}
			}
			//console.log(question.userAnswer);
			//console.log("index: " + this.index + "\n" + question.correct_answer);
			if (question.userAnswer === question.correct_answer) {
				// Set class on Button if user answered right, to celebrate right answer with animation joyfulButton
				event.target.classList.add("rightAnswer");

				// Set rightAnswer on question to true, computed property can track a streak out of 10 questions
				this.questions[index].rightAnswer = true;

				this.score++;
			} else {
				// Mark users answer as wrong answer
				event.target.classList.add("wrongAnswer");
				this.questions[index].rightAnswer = false;

				// Show right Answer
				let correctAnswer = this.questions[index].correct_answer;
				let allButtons = document.querySelectorAll(
					`[index="${index}"]`
				);
				allButtons.forEach(function (button) {
					if (button.innerHTML === correctAnswer) {
						button.classList.add("showRightAnswer");
					}
				});
			}
			if (this.questions.length - 1 === this.index) {
				setTimeout(() => {
					this.playing = false;
					document.getElementById("quiz").hidden = true;
					document.getElementById("score").hidden = false;
					document.getElementById("score").innerHTML =
						"You got " +
						this.score +
						" out of " +
						this.questions.length +
						" questions right!";
				}, 1000);
			}
		},
		startQuiz() {
			document.getElementById("start-quiz").hidden = true;
			document.getElementById("custom-quiz").hidden = true;
			document.getElementById("quiz").hidden = false;
			this.countDownTimer();
		},
		countDownTimer() {
			document.getElementById("count-down-timer").innerHTML =
				"Time remaining: " + this.timer;
			let interval = setInterval(() => {
				this.timer--;
				if (this.timer === 0 || !this.playing) {
					document.getElementById("quiz").hidden = true;
					document.getElementById("score").hidden = false;
					document.getElementById("score").innerHTML =
						"You got " +
						this.score +
						" out of " +
						this.questions.length +
						" questions right!";
					clearInterval(interval);
				}
				document.getElementById("count-down-timer").innerHTML =
					"Time remaining: " + this.timer;
			}, 1000);
		},
		customQuizInput() {
			document.getElementById("start-quiz").hidden = true;
			document.getElementById("custom-quiz").hidden = true;
			const fileInput = document.getElementById("file-input");
			const inputButton = document.getElementById("choose-file");
			const submitButton = document.getElementById("submit");

			inputButton.hidden = false;
			submitButton.hidden = false;

			inputButton.addEventListener("click", fileInput.click());
			fileInput.addEventListener("change", function () {
				if (fileInput.value) {
					fileInput.innerHTML = fileInput.value.str
						.split(/(\\|\/)/g)
						.pop();
				} else if (fileInput.value.split.pop() != ".txt") {
					fileInput.innerHTML = "Invalid File Type";
				}
			});
		},
	},
	// We want the Component to fetch and store the data, when the Component mounts
	mounted() {
		this.fetchQuestions();
	},
};
</script>

<style scoped>
form {
	display: flex;
	flex-direction: row;
	flex-wrap: wrap;
	justify-content: center;
}

button {
	font-size: 1.1rem;
	font-weight: bold;
	box-sizing: border-box;
	padding: 1rem;
	margin: 0.3rem;
	width: 47%;
	background-color: rgba(100, 100, 100, 0.3);
	border: none;
	border-radius: 0.4rem;
	box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
	cursor: pointer;
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

#quiz-container {
	text-align: center;
	margin: 1rem auto;
	padding: 1rem;
	max-width: 750px;
}

#logo-headline {
	font-size: 3rem;
	padding: 0.5rem;
	color: #000000;
	text-align: center;
}

#neec-logo {
	display: block;
	width: 40%;
	margin: 0 auto;
}

@media only screen and (max-width: 500px) {
	#neec-logo {
		width: 30%;
	}

	#logo-headline {
		font-size: 1.8rem;
	}
}

/* Styles in Quiz.vue for UX on user answer */
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

h1 {
	font-size: 1.3rem;
	padding: 0.7rem;
	text-shadow: 1px 3px 3px rgba(0, 0, 0, 0.3);
}

.divider {
	margin: 0.5rem 0;
	border: 3.7px solid rgba(0, 0, 0, 0.7);
	border-radius: 3px;
	box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.3);
}

input {
	font-size: 1.1rem;
	font-weight: bold;
	box-sizing: border-box;
	padding: 1rem;
	margin: 1rem;
	width: 47%;
	background-color: rgba(100, 100, 100, 0.3);
	border: none;
	border-radius: 0.4rem;
	box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
	cursor: pointer;
}
</style>
