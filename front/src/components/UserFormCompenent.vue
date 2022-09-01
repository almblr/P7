<template>
  <div class="form">
    <h2 class="form__title">{{ title }}</h2>
    <form class="form__body">
      <input
        name="firstName"
        type="text"
        placeholder="Prénom*"
        class="input small"
        @input="testRegexp(firstName)"
        ref="firstName"
        v-if="isSignupView"
      />
      <input
        name="lastName"
        type="text"
        placeholder="Nom*"
        class="input small"
        @input="testRegexp(lastName)"
        ref="lastName"
        v-if="isSignupView"
      />
      <input
        name="email"
        type="email"
        placeholder="Adresse mail*"
        class="input large"
        @input="testRegexp(email)"
        ref="email"
      />
      <div>
        <input
          id="password"
          name="password"
          :type="inputType"
          placeholder="Mot de passe*"
          class="input large"
          @input="
            testRegexp(password);
            checkPassword();
          "
          ref="password"
        />
        <eye-component :type="inputType" :click="showPassword"></eye-component>
        <p class="form__body__passwordInfo" v-if="isSignupView">
          Doit contenir une majuscule, un chiffre et un caractère spécial (8
          caractères min.)
        </p>
      </div>
      <div v-if="isSignupView">
        <input
          id="passwordConfirm"
          name="password"
          type="password"
          placeholder="Confirmez votre mot de passe*"
          class="input large"
          @input="
            testRegexp(passwordConfirm);
            checkPassword();
          "
          ref="passwordConfirm"
        />
      </div>

      <div class="errorDiv" v-if="isSignupView">
        <p ref="errorMsg" class="form__body__errorMessage"></p>
      </div>

      <button-form-component
        :text="isSignupView ? 'Envoyer' : 'Se connecter'"
        @click.prevent="isSignupView ? signupForm() : loginForm()"
      ></button-form-component>
    </form>
    <footer>
      <text-bottom-form-component
        :question="isSignupView ? 'Déjà inscrit ?' : 'Pas de compte ?'"
        :response="isSignupView ? 'Connecez-vous !' : 'Inscrivez-vous !'"
        :url="isSignupView ? '/' : '/signup'"
      ></text-bottom-form-component>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useUserStore, testRegexp } from '../stores/index.js';
import { useRouter } from 'vue-router';
import router from '../router/index.js';
import ButtonFormComponent from '../components/ButtonFormComponent.vue';
import EyeComponent from '../components/EyeComponent.vue';
import TextBottomFormComponent from '../components/TextBottomFormComponent.vue';

const props = defineProps({
  title: {
    type: String,
  },
});

/* Permet de vérifier si la view actuelle est signup ou login */
const isSignupView = computed(() => {
  if (route.currentRoute.value.name === 'Signup') {
    return true;
  } else {
    return false;
  }
});

const route = useRouter();
const userStore = useUserStore();
const inputType = ref('password');
const firstName = ref(null);
const lastName = ref(null);
const email = ref(null);
const password = ref(null);
const passwordConfirm = ref(null);
const errorMsg = ref(null);

/* Permet de switch le type de l'input afin de voir le mdp */
function showPassword() {
  inputType.value === 'password'
    ? (inputType.value = 'text')
    : (inputType.value = 'password');
}

/* Vérifie si les deux mots de passe correspondent dans Signup View */
const checkPassword = () => {
  if (passwordConfirm.value.value !== password.value.value) {
    errorMsg.value.innerText = 'Les mots de passe ne sont pas identiques.';
    if (passwordConfirm.value.classList.contains('isOk')) {
      passwordConfirm.value.classList.remove('isOk');
    }
    return false;
  } else {
    errorMsg.value.innerText = '';
    return true;
  }
};

/* Fonction de création de compte */
const signupForm = async () => {
  // On vérie d'abord si les regExp sont correctes
  if (
    testRegexp(lastName.value) &&
    testRegexp(firstName.value) &&
    testRegexp(email.value) &&
    testRegexp(password.value) &&
    testRegexp(passwordConfirm.value) &&
    checkPassword()
  ) {
    const data = {
      email: email.value.value,
      password: password.value.value,
      firstName: firstName.value.value,
      lastName: lastName.value.value,
    };
    await userStore.signup(data);
    const result = await userStore.login(
      email.value.value,
      password.value.value
    );
    localStorage.setItem(
      'TokenUser',
      JSON.stringify({
        token: result.token,
        userId: result.userId,
        userRole: result.userRole,
        userName: result.userName,
      })
    );
    router.push('/news');
  } else {
    alert('Des champs sont manquants ou incorrects.');
  }
};

/* Fonction de connexion */
const loginForm = async (e) => {
  e.preventDefault();
  if (testRegexp(email.value) && testRegexp(password.value)) {
    const result = await userStore.login(
      email.value.value,
      password.value.value
    );
    if (Object.values(result).includes('Utilisateur non trouvé.')) {
      // Si le resultat du fetch contient "Utilisateur non trouvé !"
      alert('Vérifiez vos informations de connexion.');
    } else {
      if (localStorage !== null) {
        localStorage.clear();
      }
      localStorage.setItem(
        'TokenUser',
        JSON.stringify({
          token: result.token,
          userId: result.userId,
          userRole: result.userRole,
          userName: result.userName,
        })
      );
      router.push('/news');
    }
  } else {
    alert('Des champs sont manquants ou incorrects.');
  }
};
</script>

<style lang="scss" scoped>
.form {
  @include justify-and-align_center;
  background-color: white;
  flex-wrap: wrap;
  width: 90%;
  max-width: 1000px;
  height: 80%;
  box-shadow: 5px 5px 20px #0000003d;
  border-radius: 16px;
  margin-top: 2%;
  &__title {
    width: 298px;
    height: 50px;
    font-weight: 700;
    font-size: 32px;
    text-align: center;
    margin-top: 3%;
    color: rgba(0, 0, 0, 0.76);
  }
  &__body {
    position: relative;
    display: flex;
    flex-flow: row wrap;
    justify-content: space-between;
    align-items: center;
    width: 90%;
    height: 80%;
    & .input {
      height: 41px;
      background: #ffffff;
      mix-blend-mode: normal;
      border: 1px solid #00000085;
      border-radius: 5px;
      padding-left: 5px;
      input::placeholder {
        height: 30px;
        font-size: 16px;
      }
      &:focus {
        outline: none;
        border: 2px solid #fd2d01;
      }
    }
    & .small {
      width: 49%;
    }
    & .large {
      width: 100%;
    }
    & div {
      position: relative;
      width: 100%;
      height: 41px;
      border-radius: 5px;
      & input {
        width: 100% !important;
        height: 100% !important;
        border: 1px solid #00000085;
      }
    }
    &__passwordInfo {
      display: block;
      margin-top: 1%;
      width: 100%;
      font-size: 12px;
      line-height: 14px;
    }
  }
}

footer {
  display: flex;
  width: max-content;
  justify-content: space-between;
  gap: 10px;
  margin-bottom: 15px;
}

.isOk {
  border: 2px solid green !important;
}

.isNotOk {
  border: 2px solid red !important;
}

.errorDiv {
  height: min-content !important;
  & p {
    text-align: center;
    color: #fd2d01;
    font-weight: bold;
  }
}
</style>