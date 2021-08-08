<template>
  <div class="quiz" :class="state">
    <transition name="t-fade" mode="out-in">
      <div
        class="quiz__bg"
        :style="{ backgroundImage: `url(${bgImage})` }"
        :key="bgImage"
      ></div>
    </transition>
    <div
      class="quiz__content"
      :style="{ backgroundImage: `url(${bgContent})` }"
    >
      <transition name="t-content" mode="out-in">
        <quiz-start
          v-if="state === 'start' || state === 'questions'"
          v-bind="initialData"
          @submit="nextStep"
        />
        <quiz-loading
          v-else-if="state === 'loading'"
          @submit="nextStep"
          v-bind="loadingData"
        />
        <quiz-form
          v-else-if="state === 'form'"
          @submit="sendForm"
          v-bind="formData"
        />
        <quiz-results-content
          v-else-if="state === 'results'"
          v-bind="resultsData"
        />
      </transition>
    </div>
    <transition name="t-fade">
      <quiz-questions
        v-model="answers"
        v-if="state === 'questions'"
        :items="questions"
        @submit="sendQuestions"
      />
    </transition>
    <quiz-results v-if="state === 'results'" v-bind="resultsData" />
    <transition name="t-fade">
      <quiz-form-image v-if="state === 'form'" v-bind="formData" />
    </transition>
    <quiz-info v-bind="initialData" />
  </div>
</template>

<script>
import QuizForm from "./QuizForm.vue";
import QuizFormImage from "./QuizFormImage.vue";
import QuizInfo from "./QuizInfo.vue";
import QuizLoading from "./QuizLoading.vue";
import QuizQuestions from "./QuizQuestions/QuizQuestions.vue";
import QuizResults from "./QuizResults/QuizResults.vue";
import QuizResultsContent from "./QuizResults/QuizResultsContent.vue";
import QuizStart from "./QuizStart.vue";
import { getInitialData, saveAnswers, completeQuiz } from "@/api/quiz-routes";
export default {
  components: {
    QuizStart,
    QuizLoading,
    QuizQuestions,
    QuizResults,
    QuizForm,
    QuizResultsContent,
    QuizInfo,
    QuizFormImage,
  },
  data() {
    return {
      initialData: {},
      formData: {},
      resultsData: {},
      bgImage: "",
      answers: {},
      phone: "",
      stateIdx: 0,
      stateOrder: ["start", "questions", "loading", "form", "results"],
    };
  },
  computed: {
    screens() {
      return this.initialData.screens || [];
    },
    loadingData() {
      return this.screens[this.screens.length - 1];
    },
    questions() {
      return this.screens.slice(0, this.screens.length - 1);
    },
    bgContent() {
      return this.initialData.imageBack2;
    },
    state() {
      return this.stateOrder[this.stateIdx];
    },
    sessionId() {
      return this.initialData.sessionId;
    },
  },
  created() {
    this.fetchInitial();
    this.$eventBus.$on("bg", (newBg) => {
      console.log("new bg", newBg);
      this.bgImage = newBg;
    });
  },
  methods: {
    async sendQuestions() {
      const answers = this.questions.map((q) => {
        const qAnswers = this.answers[q.id];
        return {
          id: q.id,
          value: qAnswers,
        };
      });
      const toSend = {
        sessionId: this.sessionId,
        screens: answers,
      };
      console.log(toSend);
      this.formData = await saveAnswers(toSend);
      this.bgImage = this.formData.imageBack;
      this.nextStep();
    },
    async sendForm(phone) {
      const normPhone = phone.replace(/[()\s-]/g, "");
      this.resultsData = await completeQuiz({
        phone: normPhone,
        sessionId: this.sessionId,
      });
      console.log("results", this.resultsData);
      this.nextStep();
      this.bgImage = this.resultsData.imageBack;
    },
    nextStep() {
      if (this.stateIdx < this.stateOrder.length - 1) {
        this.stateIdx++;
      }
    },
    async fetchInitial() {
      const data = await getInitialData();
      this.initialData = data;
      this.bgImage = data.imageBack;
      console.log(data);
    },
  },
};
</script>

<style lang="postcss">
.quiz {
  @apply w-full min-h-screen relative pt-[60px] pb-[60px] flex flex-col md:pt-[300px] sm:pt-[90px];
  h1,
  h2 {
    @apply text-[50px] font-extrabold leading-none md:text-[40px] sm:text-[24px] sm:leading-tight;
  }
  &__bg {
    @apply z-10 absolute top-0 left-0 right-0 bottom-0 bg-center bg-no-repeat bg-cover;
    &::before {
      content: "";
      @apply absolute top-0 left-0 right-0 bottom-0 opacity-60;
      background: linear-gradient(180deg, #5b698b 0%, #000000 100%);
    }
  }
  &__content {
    @apply flex-1 relative overflow-hidden z-30
            w-[690px] pt-[100px] pl-[140px] pb-[80px] pr-[90px] bg-red rounded-tr-[30px] rounded-br-[30px]
            bg-cover bg-center bg-no-repeat min-h-[700px]
            flex flex-col
            md:pl-[120px] md:pt-[80px] md:pb-[70px]
            md:flex-none md:mt-auto md:w-[640px] md:min-h-[600px] 
            sm:w-full sm:pt-[30px] sm:px-[45px] sm:min-h-[370px] sm:pb-[40px];
    box-shadow: 0px 4px 47px #000000;
    @screen md {
      box-shadow: none;
    }
    > div {
      @apply flex-1;
    }
  }
  &__float-bar {
    @apply z-30 rounded-[30px] bg-white
      pt-[50px] pb-[50px] sm:pt-[25px] sm:pb-[25px];
    box-shadow: 0px 4px 57px rgba(0, 0, 0, 0.25);
  }
  &.results {
    @apply md:pt-[70px] sm:pt-0;
  }
  &.results & {
    &__content {
      @apply pt-[60px] pr-[50px] pl-[60px] sm:mt-0 sm:rounded-tr-none
      sm:pt-[30px] sm:px-[30px];
      h1,
      h2 {
        @apply sm:text-[20px];
      }
    }
  }
}
.t-content-enter,
.t-content-leave-to {
  @apply transform translate-x-full opacity-0;
}
.t-content-enter-active,
.t-content-leave-active {
  @apply transition-all duration-500;
}
</style>