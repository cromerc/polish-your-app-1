<script setup>
import { computed, reactive, ref } from "vue";
/*
 This is an Icon that you can use to represent the stars if you like
 otherwise you could just use a simple ⭐️ emoji, or * character.
*/
import { StarIcon, TrashIcon, PencilIcon } from "@heroicons/vue/24/solid";
import { items } from "./movies.json";

const movies = ref(items);

function updateRating(movieIndex, rating) {
  movies.value[movieIndex].rating = rating;
}

const errors = reactive({
  name: null,
  description: null,
  image: null,
  inTheaters: null,
  genres: null,
});
const form = reactive({
  name: null,
  description: null,
  image: null,
  inTheaters: false,
  genres: null,
});
const validations = reactive({
  name: "required",
  genres: "required",
});

const edit = ref(-1);

const genres = reactive([
  { text: "Drama", value: "Drama" },
  { text: "Crime", value: "Crime" },
  { text: "Action", value: "Action" },
  { text: "Comedy", value: "Comedy" },
]);

const validationRules = (rule) => {
  if (rule === "required") return /^ *$/;

  return null;
};

function validate() {
  let valid = true;
  clearErrors();
  for (const [field, rule] of Object.entries(validations)) {
    const validation = validationRules(rule);
    if (validation) {
      if (validation.test(form[field] || "")) {
        errors[field] = `${field} is ${rule}`;
        valid = false;
      }
    }
  }

  return valid;
}

function addMovie() {
  if (validate()) {
    const movie = {
      id: Number(Date.now()),
      name: form.name,
      description: form.description,
      image: form.image,
      genres: form.genres,
      inTheaters: form.inTheaters,
      rating: null,
    };
    movies.value.push(movie);
    hideForm();
  }
}

function editMovie() {
  if (validate()) {
    movies.value = movies.value.map(movie => {
      if (movie.id === edit.value) {
        const editMovie = {
          id: movie.id,
          name: form.name,
          description: form.description,
          image: form.image,
          genres: form.genres,
          inTheaters: form.inTheaters,
          rating: movie.rating,
        }
        return editMovie;
      }
      return movie;
    });
    hideForm();
  }
}

function deleteMovie(id) {
  movies.value = movies.value.filter((movie) => movie.id !== id);
}

function cleanUpForm() {
  form.name = null;
  form.description = null;
  form.image = null;
  form.genres = null;
  form.inTheaters = false;
  clearErrors();
}

function clearErrors() {
  errors.name = null;
  errors.description = null;
  errors.image = null;
  errors.genres = null;
  errors.inTheaters = null;
}

const showMovieForm = ref(false);

function hideForm() {
  showMovieForm.value = false;
  cleanUpForm();
}

function showForm(movie = {}) {
  showMovieForm.value = true;
  edit.value = -1;
  if (movie.name !== undefined) {
    edit.value = movie.id;
    form.name = movie.name;
    form.description = movie.description;
    form.image = movie.image;
    form.genres = movie.genres;
    form.inTheaters = movie.inTheaters;
  }
}

function removeRatings() {
  movies.value = movies.value.map(movie => {
    movie.rating = 0;
    return movie;
  });
}

const movieTotal = computed(() => {
  return movies.value.length;
});

const averageRating = computed(() => {
  var average = 0;
  movies.value.forEach((movie) => {
    average += movie.rating === null ? 0 : movie.rating;
  });
  average /= movies.value.length !== 0 ? movies.value.length : 1;
  return Math.round(average * 10) / 10;
});
</script>

<template>
  <div class="app">
    <div v-if="showMovieForm" class="modal-wrapper">
      <div class="modal-wrapper-inner">
        <form @submit.prevent="addMovie">
          <div class="movie-form-input-wrapper">
            <label for="name">Name</label>
            <input type="text" name="name" v-model="form.name" class="movie-form-input" />
            <span class="movie-form-error">{{ errors.name }}</span>
          </div>
          <div class="movie-form-input-wrapper">
            <label for="description">Description</label>
            <textarea type="text" name="description" v-model="form.description" class="movie-form-textarea" />
            <span class="movie-form-error">{{ errors.description }}</span>
          </div>
          <div class="movie-form-input-wrapper">
            <label for="image">Image</label>
            <input type="text" name="image" v-model="form.image" class="movie-form-input" />
            <span class="movie-form-error">{{ errors.image }}</span>
          </div>
          <div class="movie-form-input-wrapper">
            <label for="genre">Genres</label>
            <select name="genre" v-model="form.genres" class="movie-form-input" multiple>
              <option v-for="option in genres" :key="option.value" :value="option.value">
                {{ option.text }}
              </option>
            </select>
            <span class="movie-form-error">
              {{ errors.genres }}
            </span>
          </div>
          <div class="movie-form-input-wrapper">
            <label for="genre" class="movie-form-checkbox-label">
              <input type="checkbox" v-model="form.inTheaters" :true-value="true" :false-value="false"
                class="movie-form-checkbox" />
              <span>In theaters</span>
            </label>
            <span class="movie-form-error">
              {{ errors.inTheaters }}
            </span>
          </div>
          <div class="movie-form-actions-wrapper">
            <button type="button" class="button" @click="hideForm">
              Cancel
            </button>

            <button v-if="edit !== -1" type="submit" class="button-primary" @click="editMovie()">Save</button>
            <button v-else type="submit" class="button-primary">Create</button>
          </div>
        </form>
      </div>
    </div>
    <div class="movie-actions-list-wrapper">
      <div class="movies-info">
        Total movies: {{ movieTotal }} / Average rating: {{ averageRating }}
      </div>
      <div class="flex-spacer"></div>
      <div class="movie-actions-list-actions">
        <button class="movie-actions-list-action-button" :class="{
            'button-primary': !showMovieForm,
            'button-disabled': showMovieForm,
          }" @click="removeRatings" :disabled="showMovieForm">
          Remove Ratings
        </button>
        <button class="movie-actions-list-action-button" :class="{
            'button-primary': !showMovieForm,
            'button-disabled': showMovieForm,
          }" @click="showForm" :disabled="showMovieForm">
          Add Movie
        </button>
      </div>
    </div>
    <div class="movie-list">
      <div class="movie-item" v-for="(movie, movieIndex) in movies" :key="movie.id">
        <div class="movie-item-image-wrapper">
          <div class="movie-item-star-wrapper">
            <StarIcon id="rating" class="movie-item-star-rating-icon"
              :class="[movie.rating ? 'text-yellow-500' : 'text-gray-500']" />
            <div class="movie-item-star-content-wrapper">
              <span v-if="movie.rating" id="rating-stars" class="movie-item-star-content-rating-rated">
                {{ movie.rating }}
              </span>
              <span v-else class="movie-item-star-content-rating-not-rated">
                -
              </span>
            </div>
          </div>
          <img :src="movie.image" class="movie-item-image" alt="" />
        </div>

        <div class="movie-item-content-wrapper">
          <div class="movie-item-title-wrapper">
            <h3 class="movie-item-title">{{ movie.name }}</h3>
            <div class="movie-item-genres-wrapper">
              <span v-for="genre in movie.genres" :key="`${movie.id}-${genre}`" class="movie-item-genre-tag">{{ genre
              }}</span>
            </div>
          </div>
          <div class="movie-item-description-wrapper">
            <p class="movie-item-description">{{ movie.description }}</p>
          </div>
          <div class="movie-item-rating-wrapper">
            <span class="movie-item-rating-text">
              Rating: ({{ movie.rating }}/5)
            </span>

            <div class="movie-item-star-icon-wrapper">
              <button v-for="star in 5" :key="star" class="movie-item-star-icon-button" :class="[
                  star <= movie.rating ? 'text-yellow-500' : 'text-gray-500',
                ]" :disabled="star === movie.rating" @click="updateRating(movieIndex, star)">
                <StarIcon class="movie-item-star-icon" />
              </button>
            </div>
          </div>
          <div class="movie-item-button-wrapper">
            <button class="button-circle-primary movie-item-button-separation">
              <PencilIcon class="movie-item-button-icon" @click="showForm(movie)" />
            </button>
            <button class="button-circle-primary movie-item-button-separation">
              <TrashIcon class="movie-item-button-icon" @click="deleteMovie(movie.id)" />
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
