<template>
  <div class="modal fade show" style="display: block; background: rgba(0, 0, 0, 0.5);">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" style="color: black;">{{ isEditing ? 'Edytuj Film' : 'Dodaj Film' }}</h5>
          <button type="button" class="btn-close" @click="closeModal"></button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="onSave">
            <div class="mb-3">
              <label for="title" class="form-label">Tytuł:</label>
              <input
                type="text"
                v-model="$v.localMovie.title.$model"
                name="title"
                id="title"
                maxlength="200"
                @blur="$v.localMovie.title.$touch()"
                class="form-control"
              />
              <div v-if="$v.localMovie.title.$error && $v.localMovie.title.$dirty" class="text-danger">
                <span v-if="$v.localMovie.title.required">Tytuł jest wymagany.</span>
                <span v-if="$v.localMovie.title.maxLength">Tytuł musi mieć mniej niż 200 znaków.</span>
              </div>
            </div>
            <div class="mb-3">
              <label for="director" class="form-label">Reżyser:</label>
              <input
                type="text"
                v-model="$v.localMovie.director.$model"
                name="director"
                id="director"
                @blur="$v.localMovie.director.$touch()"
                class="form-control"
              />
              <div v-if="$v.localMovie.director.$error && $v.localMovie.director.$dirty" class="text-danger">
                <span v-if="$v.localMovie.director.required">Reżyser jest wymagany.</span>
              </div>
            </div>
            <div class="mb-3">
              <label for="year" class="form-label">Rok:</label>
              <input
                type="number"
                v-model="$v.localMovie.year.$model"
                name="year"
                id="year"
                @blur="$v.localMovie.year.$touch()"
                class="form-control"
              />
              <div v-if="$v.localMovie.year.$error && $v.localMovie.year.$dirty" class="text-danger">
                <span v-if="$v.localMovie.year.required">Rok jest wymagany.</span>
                <span v-if="$v.localMovie.year.between">Rok musi być pomiędzy 1900 a 2200.</span>
              </div>
            </div>
            <div class="mb-3">
              <label for="rate" class="form-label">Ocena:</label>
              <input
                type="number"
                v-model="$v.localMovie.rate.$model"
                name="rate"
                id="rate"
                step="0.1"
                @blur="$v.localMovie.rate.$touch()"
                class="form-control"
              />
              <div v-if="$v.localMovie.rate.$error && $v.localMovie.rate.$dirty" class="text-danger">
                <span v-if="$v.localMovie.rate.required">Ocena jest wymagana.</span>
                <span v-if="$v.localMovie.rate.$invalid">Ocena musi być pomiędzy 0.0 a 10.0.</span>
              </div>
            </div>
            <p v-if="errorMessage" class="text-danger">{{ errorMessage }}</p>
            <div class="modal-footer">
              <button type="submit" class="btn" :class="isEditing ? 'btn-warning' : 'btn-success'" :disabled="$v.$invalid">{{ isEditing ? 'Zaktualizuj' : 'Dodaj' }}</button>
              <button type="button" class="btn btn-secondary" @click="closeModal">Anuluj</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import useVuelidate from "@vuelidate/core";
import { ref, watch } from "vue";
import { required, maxLength, between, minValue, maxValue } from "@vuelidate/validators";

export default {
  name: "MovieModalComponent",
  props: {
    isEditing: Boolean,
    movie: Object,
  },
  setup(props, { emit }) {
    const localMovie = ref({ ...props.movie });

    const rules = {
      localMovie: {
        title: { required, maxLength: maxLength(200) },
        director: { required },
        year: { required, between: between(1900, 2200) },
        rate: { required, minValue: minValue(0.0), maxValue: maxValue(10.0) },
      },
    };

    const $v = useVuelidate(rules, { localMovie });

    watch(
      () => props.movie,
      (newValue) => {
        localMovie.value = { ...newValue };
      },
      { deep: true, immediate: true }
    );

    const onSave = () => {
      if ($v.value.$invalid) {
        $v.value.$touch();
        return;
      }
      emit("save-movie", localMovie.value);
    };

    const closeModal = () => {
      emit("close-modal");
    };

    return {
      localMovie,
      $v,
      onSave,
      closeModal,
    };
  },
};
</script>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-content {
  background: white;
  padding: 20px;
  border-radius: 4px;
  width: 400px;
}
.btn-success {
  background-color: #10c228;
  color: white;
}
.btn-success:hover {
  background-color: hsl(120, 2%, 37%);
  color: white;
}
.btn-warning {
  background-color: #ffc107;
  color: black;
}
.btn-warning:hover {
  background-color: #e0a800;
  color: black;
}
</style>
