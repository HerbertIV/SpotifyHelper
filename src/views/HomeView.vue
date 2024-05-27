<script setup>
import TheWelcome from '../components/TheWelcome.vue'
import {ref} from "vue";

const auth = ref(false);
const errors = ref(false);
const password = ref(null);

const authorize = () => {
  if (password.value.value !== process.env.PASSWORD_PAGE) {
    auth.value = false;
    errors.value = true;
  } else {
    auth.value = true;
    errors.value = false;
  }
}

</script>

<template>
  <main>
    <TheWelcome v-if="auth" />
    <div v-else>
      <div class="flex items-center">
        <div class="sm:col-span-4" >
          <label for="password" class="block text-sm font-medium leading-6 text-gray-900">Password</label>
          <div class="mt-2">
            <div class="flex rounded-md shadow-sm ring-1 ring-inset ring-gray-300 focus-within:ring-2 focus-within:ring-inset focus-within:ring-indigo-600 sm:max-w-md">
              <input type="password" placeholder="password" ref="password" class="block flex-1 border-0 bg-transparent py-1.5 pl-1 text-gray-900 placeholder:text-gray-400 focus:ring-0 sm:text-sm sm:leading-6">
            </div>
          </div>
        </div>
        <div class="mt-8 ml-2 flex items-center justify-end gap-x-6">
          <button type="submit" @click="authorize" class="rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">Submit</button>
        </div>
      </div>
      <p class="mt-1 text-sm leading-6 text-gray-600" v-if="errors">Wrong password.</p>
    </div>

  </main>
</template>
