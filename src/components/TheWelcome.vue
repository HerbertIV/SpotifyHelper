<script setup>

import {ref, watch} from "vue";
import axios from "axios";
import Papa from 'papaparse';

const file = ref();
const data = ref([]);
const download = ref(false);
const loading = ref(false);
const upload = async (e) => {
  download.value = false;
  loading.value = true;
  let singleFile = file.value.files[0];
  await Papa.parse(singleFile, {
    complete: async function (results) {
      let headers = [];
      let max = process.env.MAX_LIMIT <= results.data.length ? process.env.MAX_LIMIT : results.data.length;

      for (let i = 0; i < max; i++) {
        let element = results.data[i];
        let row = [];
        for (let j = 0; j < element.length; j++) {
          if (i === 0) headers.push(element[j]); else row[headers[j]] = element[j];
        }
        if (row && i > 0) {
          if (typeof row['Link'] !== undefined && row['Link']) {
            const result = await fetchRapid(row['Link'].replace(/.*\/(\w+)\??.*$/g, '$1'));
            let monthlyListeners = row['Monthly Listeners'] === '' ? 0 : parseInt(row['Monthly Listeners']);
            row['Zmiana ML'] = result.data.monthly_listeners - monthlyListeners;
            data.value.push({
              ...row,
              ...{'Monthly Listeners': result.data.monthly_listeners}
            });
          }
        }
      }
      data.value.sort(
        (a, b) => a['Monthly Listeners'] > b['Monthly Listeners'] ?
          -1 :
          (a['Monthly Listeners'] < b['Monthly Listeners'] ? 1 : 0)
      );
      updatePosition(data);
      loading.value = false;
      download.value = true;
    }
  });
}

const updatePosition = (data) => {
  let updateData = [];
  for (let i = 0; i < data.value.length; i++) {
    let row = data.value[i];
    let position = row['Pozycja'];
    let newPosition = i+1;
    row['Zmiana pozycji'] = position - newPosition;
    updateData.push({
      ...row,
      ...{'Pozycja': newPosition}
    })
  }

  data.value = updateData;
}

const downloadFile = () => {
  let now = new Date(Date.now());
  let fileDate = now.getDate() + '' + now.getMonth() + '' + now.getFullYear();
  const csv = new Blob([Papa.unparse(data.value)], { type: 'text/csv' });
  const url = window.URL.createObjectURL(csv);
  const link = document.createElement('a');
  link.href = url;
  link.setAttribute('download', 'TOP_CURRENTLY_' + fileDate);
  document.body.appendChild(link);
  link.click();
}

const fetchRapid = async (artistId) => {
  if (process.env.MODE === 'production') {
    const options = {
      method: 'GET',
      url: process.env.RAPID_API_URL,
      params: {
        spotify_artist_id: artistId
      },
      headers: {
        'X-RapidAPI-Key': process.env.RAPID_API_KEY,
        'X-RapidAPI-Host': process.env.RAPID_API_HOST
      }
    };

    try {
      const response = await axios.request(options);
      return {
        data: response.data
      };
    } catch (error) {
      console.error(error);
    }
  }

  return {
    data: {
      message: "Data Retrieved.",
      monthly_listeners: Math.floor(Math.random() * 30000),
      result: "success",
      spotify_artist_id: "4lS1f07YGBEKA9ai638UbU"
    }
  };
}
</script>

<template>
  <div class="space-y-12">
    <div class="border-b border-gray-900/10 pb-12">
      <div class="mt-10 grid grid-cols-1 gap-x-6 gap-y-8 sm:grid-cols-6">
        <div class="col-span-full">
          <label for="photo" class="block text-sm font-medium leading-6 text-gray-900">Upload</label>
          <div class="mt-2 flex items-center gap-x-3">
              <input type="file" @change="upload" ref="file">
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="mt-6 flex items-center justify-end gap-x-6">
    <div role="status" v-if="loading">
      <svg aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
      </svg>
      <span class="sr-only">Loading...</span>
    </div>
    <a
        class="rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
        v-if="download"
        @click.prevent="downloadFile()" >
      Pobierz
    </a>
  </div>
</template>
