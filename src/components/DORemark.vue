<template>
  <Card>
    <template #title>
      <div class="p-text-center">
        <h1>{{ msg }}</h1>
      </div>
    </template>
    <template #content>
      <div class="p-grid p-jc-center">
        <div class="p-col-8">
          <div class="box">
            <div class="p-fluid">
              <div class="p-field p-grid">
                <label for="buyer" class="p-col-fixed" style="width:100px">
                  Buyer
                </label>
                <div class="p-col-12 p-md-10">
                  <Dropdown
                    v-model="selectedBuyer.value"
                    :options="buyers"
                    optionLabel="buyer"
                    optionValue="code"
                    placeholder="Pilih buyer"
                    :class="selectedBuyer.invalidClass"
                  />
                  <small
                    id="buyer"
                    class="p-error"
                    v-if="selectedBuyer.invalid"
                  >
                    {{ selectedBuyer.errorMessage }}
                  </small>
                </div>
              </div>
              <div class="p-field p-grid">
                <label for="doc-number" class="p-col-fixed" style="width:100px">
                  Doc Ke -
                </label>
                <div class="p-col-12 p-md-10">
                  <InputNumber
                    v-model="docNumber.value"
                    :class="docNumber.invalidClass"
                    placeholder="Masukkan nomor urutan dokumen"
                  />
                  <small id="buyer" class="p-error" v-if="docNumber.invalid">
                    {{ docNumber.errorMessage }}
                  </small>
                </div>
              </div>
              <div class="p-field p-grid">
                <label for="excel-file" class="p-col-fixed" style="width:100px">
                  Upload Excel File
                </label>
                <div class="p-col-12 p-md-10">
                  <FileUpload
                    mode="basic"
                    name="demo[]"
                    :customUpload="true"
                    @uploader="myUploader"
                    :auto="true"
                    v-if="!reload"
                  />
                </div>
              </div>
              <div class="p-field p-grid">
                <ProgressSpinner v-if="isLoading" />
                <label
                  for="do-remark"
                  class="p-col-fixed"
                  style="width:100px"
                  v-else
                >
                  Output DO Remark
                </label>
                <div class="p-col-12 p-md-10" v-if="!isLoading">
                  <Textarea
                    v-model="value"
                    :autoResize="true"
                    rows="5"
                    cols="30"
                  />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>
  </Card>
</template>

<script>
import axios from 'axios';

import Card from 'primevue/card';
import FileUpload from 'primevue/fileupload';
import Dropdown from 'primevue/dropdown';
import InputNumber from 'primevue/inputnumber';
import Textarea from 'primevue/textarea';
import ProgressSpinner from 'primevue/progressspinner';

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  components: {
    Card,
    FileUpload,
    Dropdown,
    InputNumber,
    Textarea,
    ProgressSpinner
  },
  data() {
    return {
      array: [],
      selectedBuyer: {
        value: null,
        required: true,
        invalid: false,
        invalidClass: null,
        errorMessage: 'Pilih salah satu buyer.'
      },
      buyers: [
        { buyer: 'HeeSung', code: 'HS' },
        { buyer: 'LG', code: 'LG' }
      ],
      docNumber: {
        value: null,
        required: true,
        invalid: false,
        invalidClass: null,
        errorMessage: 'Nomor urutan dokumen tidak boleh kosong.'
      },
      reload: false,
      isLoading: false,
      invalidInput: false
    };
  },
  computed: {
    value() {
      return this.array.join('\n');
    },
    selectedBuyerVal() {
      return this.selectedBuyer.value;
    },
    docNumberVal() {
      return this.docNumber.value;
    }
  },
  watch: {
    selectedBuyerVal() {
      if (this.selectedBuyer.value) {
        this.selectedBuyer.invalid = false;
        this.selectedBuyer.invalidClass = '';
      }
    },
    docNumberVal() {
      if (this.docNumber.value) {
        this.docNumber.invalid = false;
        this.docNumber.invalidClass = '';
      }
    }
  },
  methods: {
    async myUploader(event) {
      if (!this.selectedBuyer.value) {
        this.selectedBuyer.invalid = true;
        this.selectedBuyer.invalidClass = 'p-invalid';
        this.invalidInput = true;
      } else {
        this.invalidInput = false;
      }

      if (!this.docNumber.value) {
        this.docNumber.invalid = true;
        this.docNumber.invalidClass = 'p-invalid';
        this.invalidInput = true;
      } else {
        this.invalidInput = false;
      }

      if (this.invalidInput) {
        this.setReload();
        return;
      }

      this.isLoading = true;
      this.array = [];

      const formData = this.createData(event);

      axios.post('api/upload', )

      axios
        .post('api/upload', formData, {})
        .then((res) => {
          res.data.data.forEach((element) => {
            let plt;
            let seq = element.seq;

            const br = element.br;
            const _br = this.splitStr(br, ', ');

            if (!br.includes('Ctn') && _br.length === 1) {
              const arr = this.splitStr(br, '*');
              if (this.selectedBuyer.value === 'HS') {
                plt = this.palletStr(arr, 1);
              } else {
                plt = this.palletStr(arr, 0);
              }
            } else {
              plt = element.br;
            }

            let str = `DOC KE ${this.docNumber.value}, BL : ${element.bl}, INV : ${element.inv}, ${plt}`;

            if (this.selectedBuyer.value === 'HS' && seq) {
              if (seq.toString().length === 1) {
                seq = '0' + seq;
              }
              str = str + ` (${seq})`;
            }

            this.array.push(str);
            this.setReload();
            this.isLoading = false;
          });
        })
        .catch((error) => {
          console.log(error);
        });
    },
    createData(event) {
      const formData = new FormData();

      formData.append('buyer', this.selectedBuyer);
      formData.append('docKe', this.docNumber);

      for (const i of Object.keys(event.files)) {
        formData.append('excel', event.files[i]);
      }

      return formData;
    },
    splitStr(str, separator) {
      const array = str.split(separator);
      return array;
    },
    setReload() {
      this.reload = true;
      setTimeout(() => {
        this.reload = false;
      }, 1);
    },
    palletStr(arr, index) {
      return `${arr[index]} PLT`;
    }
  }
};
</script>
