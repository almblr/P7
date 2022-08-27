<script setup>
import { ref, onMounted, watch } from 'vue';
import { usePostStore } from '../stores/index.js';
import ButtonFormComponent from '../components/ButtonFormComponent.vue';

const props = defineProps({
  show: {
    type: Boolean,
  },
  title: {
    type: String,
  },
  post: {
    type: Object,
  },
});

const input = ref(null);
const label = ref(null);
const postData = ref({});
const postStore = usePostStore();
const isFileHere = ref(false);

const deleteImg = () => {
  input.value.value = null;
  label.value.style.backgroundImage = '';
  isFileHere.value = false;
  label.value.innerText = '+';
};

const showUploadedImg = (event) => {
  isFileHere.value = true;
  const image = URL.createObjectURL(event.target.files[0]);
  label.value.style.backgroundImage = `url(${image})`;
  label.value.innerText = '';
};

const createPost = async () => {
  if (postData.value.content || input.value.value !== '') {
    const formData = new FormData();
    if (postData.value.content) {
      formData.append('content', postData.value.content);
    }
    if (input.value.value) {
      formData.append('imageUrl', input.value.files[0]);
    }
    await postStore.createOne(formData);
  } else {
  }
};

// Observe les changements de visibilité de la modal pour reset l'objet post
watch(
  () => props.show,
  async (newShow) => {
    //newShow représente la nouvelle valeur qu'a pris la props show
    if (newShow && props.post) {
      postData.value = { ...props.post }; // ... = affectation par décomposition : déstructure les 2 obj pour éviter que si l'on modife un, l'autre se modifie aussi
    }
  }
);
onMounted(() => {
  if (props.post) {
    postData.value = { ...props.post };
  }
});
</script>

<template>
  <Transition name="modal">
    <div v-if="show" class="modal__layer">
      <div class="modal__container">
        <button class="modal__container__close" @click="$emit('close')">
          X
        </button>
        <div class="modal__body">
          <h1>{{ title }}</h1>
          <div class="modal__textarea">
            <textarea
              class="post_textarea"
              placeholder="Écrivez-ici"
              v-model="postData.content"
              maxlength="560"
            ></textarea>
          </div>
          <div class="image-send">
            <div class="imageInfo">
              <div class="previewImg">
                <label
                  for="image-input"
                  id="custom-label"
                  ref="label"
                  :style="`background-image:url(${postData.imageUrl})`"
                  >+</label
                >
                <input
                  ref="input"
                  id="image-input"
                  type="file"
                  accept="image/png, image/jpg, image/jpeg"
                  @change="showUploadedImg"
                />
                <button
                  @click="deleteImg;"
                  class="delete-img"
                  v-if="isFileHere"
                >
                  X
                </button>
              </div>
              <div>
                <p>
                  <b> Cliquez pour insérer une image. </b>
                  <br />
                  Formats acceptés : png, jpg, jpeg.
                </p>
              </div>
            </div>
            <ButtonFormComponent
              text="Publier"
              @click="
                createPost();
                $emit('close');
              "
            ></ButtonFormComponent>
          </div>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style lang="scss">
.modal__layer {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.267);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.3s ease;
}

.modal__container {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  width: 100%;
  height: 80%;
  max-width: 1000px;
  margin: 0px auto;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
}

.modal__container__close {
  position: absolute;
  text-align: center;
  top: 20px;
  right: 20px;
  color: white;
  background-color: rgba(255, 0, 0, 0.568);
  border: none;
  border-radius: 15px;
  font-size: 15px;
  font-weight: bold;
  width: 25px;
  height: 25px;
  transition: 0.2s;
  &:hover {
    cursor: pointer;
    box-shadow: 3px 5px 10px rgba(0, 0, 0, 0.527);
    background-color: rgba(255, 0, 0, 0.726);
    transition: 0.2s;
  }
}

/*
 * The following styles are auto-applied to elements with
 * transition="modal" when their visibility is toggled
 * by Vue.js.
 *
 * You can easily play with the modal transition by editing
 * these styles.
 */

.modal-enter-from {
  opacity: 0;
}

.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal__container,
.modal-leave-to .modal__container {
  -webkit-transform: scale(1.02);
  transform: scale(1.02);
}

///////////////////////////////////////////////////
///////////////////////////////////////////////////
///////////////////////////////////////////////////

.modal__body {
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background-color: white;
  border-radius: 15px;
  & h1 {
    margin-top: 6%;
    // color: rgba(0, 0, 0, 0.76);
    color: red;
    font-size: 30px;
  }
}
.modal__textarea {
  border: 1px solid #b8b8b8;
  width: 90%;
  margin-top: 5%;
  max-width: 1000px;
  height: 30%;
  & textarea {
    height: 100%;
    width: 100%;
    border: none;
    outline: none;
    padding: 10px 10px 0 10px;
    resize: none;
    &::placeholder {
      position: absolute;
      top: 10px;
      left: 10px;
    }
  }
}
.image-send {
  display: flex;
  flex-wrap: wrap;
  margin: 30px 0 20px 0;
  width: 90%;
  justify-content: center;
  align-items: center;
  .imageInfo {
    display: flex;
    flex: 1;
    min-width: 100%;
    margin-bottom: 50px;
  }
  input[type='file'] {
    display: none;
  }
  #custom-label {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 55px;
    width: 100px;
    height: 100px;
    opacity: 0.8;
    left: 10px;
    color: rgba(255, 255, 255, 0.411);
    border-radius: 13px;
    background-color: rgba(72, 93, 105, 0.507);
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    background-position: center;
    background-size: cover;
    background-repeat: no-repeat;
    transition: 0.1s;
    &:hover {
      opacity: 1;
      cursor: pointer;
      transition: 0.1s;
    }
  }
  .previewImg {
    position: relative;
  }
  p {
    max-width: 400px;
    margin-left: 15px;
    font-size: 14px;
  }
}

.delete-img {
  position: absolute;
  top: 5px;
  right: 5px;
  width: 25px;
  height: 25px;
  background-color: rgba(255, 0, 0, 0.63);
  border: 1px solid rgba(255, 255, 255, 0.103);
  color: white;
  border-radius: 5px;
  &:hover {
    cursor: pointer;
  }
}

@media all and (min-width: 700px) {
  #custom-label {
    width: 130px !important;
    height: 110px !important;
  }
}
</style>