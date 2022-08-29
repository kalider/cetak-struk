<script setup>
import { ref } from 'vue';
import html2pdf from 'html2pdf.js'

const image = ref(null)
const images = ref([])

const imageChangeHandler = async () => {
  for (const file of image.value.files) {
    let imageExist = images.value.find(img => img.name == file.name)

    if (typeof imageExist != "undefined") continue

    let imageUri = await new Promise(resolve => {
      var fileReader = new FileReader();
      fileReader.onload = function () {
        resolve(fileReader.result);
      };
      fileReader.readAsDataURL(file);
    })

    file.url = imageUri
    images.value.push(file)
  }
}

const removeImageHandler = image => {
  images.value = images.value.filter(img => img.name != image.name)
}

const printHandler = () => {
  const printElement = document.createElement('table')
  printElement.setAttribute('width', '100%')
  // printElement.setAttribute('style', 'border: 1px solid black; border-collapse: collapse;')

  let htmlTables = [];
  let row = 1
  for (const i in images.value) {
    let indexRow = htmlTables.findIndex(item => item.row == row)
    if (indexRow >= 0) {
      htmlTables[indexRow].items.push(images.value[i])
    }
    else htmlTables.push({
      row: row,
      items: [images.value[i]]
    })
    if ((parseInt(i) + 1) % 3 == 0) row += 1;
  }

  htmlTables = htmlTables.map((row, i) => `<tr class="${((i + 1) % 2 == 0 && (i + 1) != htmlTables.length) ? 'afterClass' : ''}">${row.items.map(item => `<td width="25%" align="center" valign="middle"><img style="max-height: 530px; border: 1px solid #dedede; margin-top:1em" src="${item.url}"></td>`)}</tr>`)
  console.log(htmlTables)
  // return
  printElement.innerHTML = htmlTables.join("\n")
  html2pdf().set({
    margin: 0,
    filename: Date.now() + '-cetak-struk.pdf',
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2 },
    jsPDF: { unit: 'in', format: 'a4', orientation: 'portrait' },
    pagebreak: { after: '.afterClass' }
  }).from(printElement).save()
}
</script>

<template>
  <header>
    <nav class="lg:w-1/2 lg:mx-auto py-3 px-3 lg:px-0">
      <div class="flex justify-between">
        <div class="text-2xl flex items-center">Cetak struk</div>
        <div class="flex">
          <label class="flex">
            <a class="px-5 py-3 bg-lime-700 text-white rounded cursor-pointer">Masukan Gambar</a>
            <input class="hidden" type="file" ref="image" multiple accept="image/*" @change="imageChangeHandler" />
          </label>
          <button v-show="images.length > 0" class="ml-4 rounded px-5 py-3 bg-sky-600 text-white"
            @click="printHandler()">Print</button>
        </div>
      </div>
    </nav>
  </header>

  <div class="lg:w-1/2 lg:mx-auto mx-3 my-5">

    <div v-if="images.length < 1" class="bg-orange-100 p-5 rounded-lg">Belum ada gambar. Silahkan masukan gambar!
    </div>
    <div id="list-images" class="grid grid-cols-3 gap-3">
      <div v-for="image in images" class="relative">
        <span @click="removeImageHandler(image)"
          class="rounded-tr-xl rounded-bl-md cursor-pointer bg-slate-800 text-white absolute h-8 w-8 right-0 -top-0 flex justify-center items-center">
          <div class="rotate-45">+</div>
        </span>
        <img class="border rounded-xl w-full" :src="image.url">
        {{ image.name }}
      </div>
    </div>

  </div>
</template>
