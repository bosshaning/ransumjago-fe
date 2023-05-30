<template>
  <div>
    <div v-if="$fetchState.pending">
      Loading
    </div>
    <div v-else>
      <div class="judul">
          <h1>Ransum Jago</h1>
          <h4>Ransum Optimal dengan Harga Minimal</h4>
        </div>
    <b-tabs v-model="tab" content-class="mt-3">
      <b-tab title="Multi Ransum PBBH">
        <div>
          <b-alert class="mt-3" v-model="showDismissibleAlertComponent" variant="danger" dismissible>
            Cek Kembali Bahan dan PBBH yang Diinginkan
          </b-alert>
          <p><b>Kondisi Sapi</b></p>
          <div class="mt-3">
            <p>Nama Sapi</p>
            <b-form-input v-model="namaSapi" type="text"></b-form-input>
          </div>
          <div class="mt-3">
            <p>Tipe Pemeliharaan</p>
            <b-form-select v-model="tipePemeliharaan" :options="optionstype"></b-form-select>
          </div>
          <div class="mt-3">
            <p>Bobot Sapi (Kg)</p>
            <b-form-input v-model="bobotSapi" type="number"></b-form-input>
            <!-- <b-form-select v-model="selectedweight" :options="optionsweight"></b-form-select> -->
          </div>
          <div class="mt-3">
            <p>Jenis Kelamin Sapi</p>
            <b-form-select v-model="jenisKelamin" :options="optionsgender"></b-form-select>
          </div>
          <div v-if="selectedtype === 1" class="mt-3">
            <p>PBBH yang Diinginkan (Kg)</p>
            <b-form-select v-model="pbbh">
              <option :value="null" disabled>-- Pilih Pertumbuhan Bobot Badan Harian --</option>
              <option v-for="option in filterPbbh" :key="option.adg" :value="option.id" >
            <p>{{ option.adg }}</p>
          </option>
            </b-form-select>
          </div>
          <button class="mt-3" @click="onAddSapi()">+ Tambahkan Sapi</button>
          <div class="mt-3">
            <ve-table class="mt-3" :fixed-header="true" :columns="columnsSapi" :table-data="tableSapi" />
            <div v-show="tableSapi.length === 0" class="p-5 text-center border">
                Tidak Ada Data Sapi yang Ditambahkan
            </div>
          </div>
          <div class="mt-3">
          <p><b>Pilih Bahan Pakan yang Tersedia</b></p>
          <b-form-select v-model="selectedcategory" :options="optionscategory"></b-form-select>
          <b-form-select v-model="selecteditem">
            <option :value="null" disabled>-- Pilih Bahan Pakan --</option>
            <option v-for="option in filterItems" :key="option.id" :value="option.id">
          <p>{{ option.nama }}</p>
          </option>
            </b-form-select>
            <button class="mt-2" :disabled="selecteditem === null" @click.prevent="onAddItems()">+ Bahan Pakan</button>
            <button class="mt-2" @click.prevent="onAddItemsCustom()">+ Custom Bahan Pakan</button>
            <ve-table class="mt-3" :fixed-header="true" :columns="columns" :table-data="tableDataMulti" />
            <div v-show="tableDataMulti.length === 0" class="p-5 text-center border">
                Tidak Ada Bahan yang Dimasukkan
            </div>
          </div>
          <div>
            <button class="mt-2" :disabled="tableDataMulti.length === 0" @click.prevent="onsubmit()">Kalkulasi</button>
          </div>
          <VueHtml2pdf
            :show-layout="false"
            :float-layout="false"
            :enable-download="true"
            :preview-modal="false"
            :paginate-elements-by-height="2480"
            filename="Hasil Kalkulasi"
            :pdf-quality="2"
            :manual-pagination="false"
            pdf-format="a4"
            pdf-orientation="portrait"
            :pdf-content-width="1920"
            ref="html2Pdf"
          >
          <section slot="pdf-content">
            <div>
              <div class="mt-5 shadow p-3 mb-5 bg-white rounded" v-for="(result, index) in resMulti" :key="index">
                <p>Nama Sapi : {{ formData.head[index].namaSapi }}</p>
                <p>Kondisi Sapi : {{ optionsgender[formData.head[index].jenisKelamin].text }}, {{ formData.head[index].bobotSapi }} Kg </p>
                <p>Berat Kering : {{ result.wransum }} kg</p>
                <p>Nutrisi yang Harus Diberikan:</p>
                <ve-table class="mt-3" :columns="columnsNutrient" :table-data="arrResComposition[index]" :border-y="true" />
                <div v-show="result.price !== null">
                  <ve-table class="mt-3" :fixed-header="true" :footer-data="arrFooterData[index]" rowKeyFieldName="rowKey" :columns="resColumns" :table-data="arrResult[index]" :border-y="true" />
                  <i>*Perhitungan dapat sedikit bergeser karena faktor pembulatan</i>
                </div>
                <div v-show="result.price === null" class="p-5 text-center border">
                  Tidak ada Komposisi yang memenuhi
                </div>
                <div v-show="result.price !== null">
                  <p class="mt-3">Takaran Pemberian:</p>
                  <ve-table class="mt-3" :columns="columnsTakaran" :table-data="arrResult[index]" :border-y="true" />
                </div>
              </div>
            </div>
          </section>
        </VueHtml2pdf>
        <div v-show="resMulti.length !== 0" class="mt-3">
          <button @click.prevent="generateReport()">Download Hasil</button>
        </div>
        </div>
      </b-tab>

      <b-tab title="Ransum Custom">
        <div class="mt-3">
          <p><b>Komposisi Nutrisi yang Diinginkan</b></p>
          <b-form>
            <label class="mr-sm-2" for="inline-form-custom-select-pref">Protein Kasar (%)</label>
            <b-form-input v-model="formDataCustom.head.pk"></b-form-input>
          </b-form>
          <b-form>
            <label class="mr-sm-2" for="inline-form-custom-select-pref">TDN (%)</label>
            <b-form-input v-model="formDataCustom.head.tdn"></b-form-input>
          </b-form>
          <b-form>
            <label class="mr-sm-2">Kalsium (%)</label>
            <b-form-input v-model="formDataCustom.head.ca"></b-form-input>
          </b-form>
          <b-form>
            <label class="mr-sm-2" for="inline-form-custom-select-pref">Fosfor (%)</label>
            <b-form-input v-model="formDataCustom.head.p"></b-form-input>
          </b-form>
        </div>
        <div class="mt-3">
          <p><b>Pilih Bahan Pakan yang Tersedia</b></p>
          <b-form-select v-model="selectedcategory" :options="optionscategory"></b-form-select>
          <b-form-select v-model="selecteditem">
            <option :value="null" disabled>-- Please select an option --</option>
            <option v-for="option in filterItems" :key="option.id" :value="option.id">
          <p>{{ option.nama }}</p>
        </option>
          </b-form-select>
          <button class="mt-2" :disabled="selecteditem === null" @click.prevent="onAddItems()">+ Bahan Pakan</button>
          <button class="mt-2" @click.prevent="onAddItemsCustom()">+ Custom Bahan Pakan</button>
          <ve-table class="mt-3" :fixed-header="true" :columns="columns" :table-data="tableDataCustom" />
          <div v-show="tableDataCustom.length === 0" class="p-5 text-center border">
              Tidak Ada Bahan yang Dimasukkan
          </div>
        </div>
        <div>
          <button class="mt-2" :disabled="tableDataCustom.length === 0" @click.prevent="onsubmit()">Kalkulasi</button>
        </div>
        <b-alert class="mt-3" v-model="showDismissibleAlertCustom" variant="danger" dismissible>
          Tidak Ada Komposisi Bahan Pakan yang Memenuhi
        </b-alert>
        <VueHtml2pdf
        :show-layout="false"
        :float-layout="false"
        :enable-download="true"
        :preview-modal="false"
        :paginate-elements-by-height="1400"
        filename="myPDF"
        :pdf-quality="2"
        :manual-pagination="false"
        pdf-format="a4"
        pdf-orientation="portrait"
        pdf-content-width="1920"
        ref="html2PdfCustom"
    >
        <section class="ml-3 mr-3" slot="pdf-content">
        <div v-show="resultCustom.length !== 0" class="mt-5">
          <p><b>Hasil Optimasi Ransum</b></p>
          <div>
            <p>Protein Kasar (min) : {{ formDataCustom.head.pk }}</p>
            <p>TDN (min) : {{ formDataCustom.head.tdn }}</p>
            <p>Kalsium (min): {{ formDataCustom.head.ca }}</p>
            <p>Fosfor (min): {{ formDataCustom.head.p }}</p>
            <ve-table class="mt-3" :fixed-header="true" rowKeyFieldName="rowKey" :footer-data="footerDataCustom" :columns="resColumns" :table-data="resultCustom" />
            <i>*Perhitungan dapat sedikit bergeser karena faktor pembulatan</i>
          </div>
        </div>
        </section>
        </VueHtml2pdf>
        <div class="ml-3 mr-3 mt-3">
          <button v-show="resultCustom.length !== 0" @click.prevent="generateReportCustom()">Download Kalkulasi</button>
        </div>
      </b-tab>
    </b-tabs>
  </div>
  </div>
</template>

<script>
import '~/css/styles.css'
// const config = {
//   headers:{
//     Authorization: `Bearer oYiBJDPNz0P8U6KHlqhSwVkx`
//   }
// };
export default {
  name: 'IndexPage',
  data() {
    return {
      tab: 0,
      isPbbh: true,
      showDismissibleAlert: false,
      showDismissibleAlertComponent: false,
      showDismissibleAlertCustom: false,
      namaSapi: '',
      tipePemeliharaan: 1,
      bobotSapi: 100,
      jenisKelamin: 0,
      pbbh: null,
      selectedtype: 1,
      selectedransum: 1,
      selectedcategory: 0,
      selectedweight: 100,
      selectedgender: 0,
      selectedpbbh: null,
      selecteditem: null,
      weight: null,
      consumption: null,
      isCalculate: false,
      resCalculate: [],
      resMulti: [],
      arrResult: [],
      arrFooterData: [],
      optionstype: [
        {value: 1, text: "Penggemukan"},
        {value: 2, text: "Breeding"}
      ],
      optionsransum: [
        {value: 1, text: "Ransum Komplit", rule: "Bahan pakan yang dimasukkan harus terdiri dari hijauan, sumber karbohidrat, dan sumber protein"},
        {value: 2, text: "Konsentrat", rule: "Bahan pakan yang dimasukkan harus terdiri dari sumber karbohidrat dan sumber protein"}
      ],
      optionscategory: [
        {value: 0, text: "Semua"},
        {value: 1, text: "Hijauan"},
        {value: 2, text: "Sumber Karbohidrat"},
        {value: 3, text: "Sumber Protein"},
      ],
      optionsweight: [
        {value: 100, text: "100"},
        {value: 150, text: "150"},
        {value: 200, text: "200"},
        {value: 250, text: "250"},
        {value: 300, text: "300"},
        {value: 350, text: "350"},
        {value: 400, text: "400"},
        {value: 450, text: "450"},
        {value: 500, text: ">500"},
      ],
      optionsgender: [
        {value: 0, text: "Jantan"},
        {value: 1, text: "Betina"},
      ],
      optionsItem: [],
      resComposition:[
        {
          nama: 'Protein Kasar (%)',
          value: 0
        },
        {
          nama: 'TDN (%)',
          value: 0
        },
        {
          nama: 'Kalsium (%)',
          value: 0
        },
        {
          nama: 'Fosfor (%)',
          value: 0
        }
      ],
      columnsSapi:[
      { field: "namaSapi", key: "namaSapi", title: "Nama Sapi", align: "center", fixed: "left", renderBodyCell: null},
      { field: "tipePemeliharaan", key: "tipePemeliharaan", title: "Tipe Pemeliharaan", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.tipePemeliharaan === 1) {
            return "Penggemukan"
          } else if (row.tipePemeliharaan === 2) {
            return "Breeding"
          } else if (row.tipePemeliharaan === 3) {
            return "Sapi Dara"
          } else {
            return "Custom"
          }
         }},
      { field: "bobotSapi", key: "bobotSapi", title: "Bobot Sapi", align: "center", renderBodyCell: null},
      { field: "jenisKelamin", key: "jenisKelamin", title: "Jenis Kelamin", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.jenisKelamin === 0) {
            return "Jantan"
          } else if (row.jenisKelamin === 1) {
            return "Betina"
          }
         }},
      { field: "pbbh", key: "pbbh", title: "PBBH", align: "center", renderBodyCell: null},
      { field: "", key: "e", title: "Action", fixed:"right", center: "left", renderBodyCell: ({ row, column, rowIndex }, h) => {
        return h(
          'div',
          {
            domProps: {
              classList: 'flex v-center flex--wrap flex-gap-14'
            }
          },
          [
            h(
              'button',
              {
                domProps: {
                  classList: 'btn--red pv-4 ph-8'
                },
                on: {
                  click: () => this.onClickDeleteSapi(rowIndex)
                }
              },
              [
                h('span', {
                  class: 'fas fa-trash'
                })
              ]
            )
          ]
        )
        }},
      ],
      columns: [
        { field: "nama", key: "a", title: "Nama Bahan", align: "center", fixed: "left", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 4) {
            const text = row[column.field]
            return h('input', {
              attrs: {
                type: 'text',
                value: text
              },
              on: {
                change: (event) => {
                  row.nama = event.target.value
                }
              }
            },
            )
          } else {
            return (row.nama)
          }
         } },
        { field: "category", key: "b", title: "Kategori", align: "left", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 1) {
            return "Hijauan"
          } else if (row.category === 2) {
            return "Sumber Karbohidrat"
          } else if (row.category === 3) {
            return "Sumber Protein"
          } else {
            return "Custom"
          }
         }},
        {
          field: "bk", key: "bk", title: "Bahan Kering", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 4) {
            const text = row[column.field]
            return h('input', {
              attrs: {
                type: 'number',
                value: text
              },
              on: {
                change: (event) => {
                  row.bk = parseFloat(event.target.value)
                }
              }
            },
            )
          } else {
            return (row.bk)
          }
         }
        },
        { field: "cp", key: "c", title: "Protein Kasar (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 4) {
            const text = row[column.field]
            return h('input', {
              attrs: {
                type: 'number',
                value: text
              },
              on: {
                change: (event) => {
                  row.cp = parseFloat(event.target.value)
                }
              }
            },
            )
          } else {
            return (row.cp)
          }
         } },
        { field: "tdn", key: "tdn", title: "TDN (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 4) {
            const text = row[column.field]
            return h('input', {
              attrs: {
                type: 'number',
                value: text
              },
              on: {
                change: (event) => {
                  row.tdn = parseInt(event.target.value)
                }
              }
            },
            )
          } else {
            return (row.tdn)
          }
         }},
        { field: "ca", key: "ca", title: "Kalsium (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 4) {
            const text = row[column.field]
            return h('input', {
              attrs: {
                type: 'number',
                value: text
              },
              on: {
                change: (event) => {
                  row.ca = parseFloat(event.target.value)
                }
              }
            },
            )
          } else {
            return (row.ca)
          }
         } },
        { field: "p", key: "p", title: "Fosfor (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 4) {
            const text = row[column.field]
            return h('input', {
              attrs: {
                type: 'num',
                value: text
              },
              on: {
                change: (event) => {
                  row.p = parseFloat(event.target.value)
                }
              }
            },
            )
          } else {
            return (row.p)
          }
         } },
        { field: "minpercentage", key: "f", title: "Kandungan Minimal (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          const text = row[column.field]
          return h('input', {
          attrs: {
            type: 'number',
            value: text
          },
          on: {
            change: (event) => {
              row.minpercentage = parseFloat(event.target.value)
            }
          }
        },
        )
         }},
        { field: "maxpercentage", key: "g", title: "Kandungan Maksimal (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          const text = row[column.field]
          return h('input', {
            attrs: {
              type: 'number',
              value: text
            },
            on: {
              change: (event) => {
                row.maxpercentage = parseFloat(event.target.value)
              }
            }
          },
          )
        }},
        { field: "harga", key: "harga", title: "Harga", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          const text = row[column.field]
          return h('input', {
          attrs: {
            type: 'number',
            value: text
          },
          on: {
            change: (event) => {
              row.harga = parseFloat(event.target.value)
            }
          }
        },
        )}
        },
        { field: "", key: "e", title: "Action", center: "left", renderBodyCell: ({ row, column, rowIndex }, h) => {
        return h(
          'div',
          {
            domProps: {
              classList: 'flex v-center flex--wrap flex-gap-14'
            }
          },
          [
            h(
              'button',
              {
                domProps: {
                  classList: 'btn--red pv-4 ph-8'
                },
                on: {
                  click: () => this.onClickDelete(rowIndex)
                }
              },
              [
                h('span', {
                  class: 'fas fa-trash'
                })
              ]
            )
          ]
        )
        }},
      ],
      resColumns:[
      { field: "nama", key: "nama", title: "Nama Bahan", align: "center", renderBodyCell: null, fixed: "left" },
        { field: "resPercentage", key: "resPercentage", title: "Persentase Bahan", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
          return ((row.resPercentage*100).toFixed(2) + '%')
         }},
        { field: "totalPK", key: "totalPK", title: "Total PK", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.cp*row.resPercentage*row.bk).toFixed(2) + '%')
        }},
        { field: "totalTDN", key: "totalTDN", title: "Total TDN", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.tdn*row.resPercentage*row.bk).toFixed(2) + '%')
        }},
        { field: "totalCa", key: "totalCa", title: "Total Ca", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.ca*row.resPercentage*row.bk).toFixed(2) + '%')
        }},
        { field: "totalP", key: "totalP", title: "Total P", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.p*row.resPercentage*row.bk).toFixed(2) + '%')
        }},
        { field: "hargatotal", key: "hargatotal", title: "Harga", align: "center", fixed: 'right', renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.harga*row.resPercentage).toFixed())
        }}
      ],
      columnsNutrient:[
      { field: "nama", key: "nama", title: "Nama Bahan", align: "center", renderBodyCell: null, fixed: "left" },
      { field: "value", key: "value", title: "Komposisi Bahan (%)", align: "center", renderBodyCell: null, fixed: "left" }
      ],
      columnsTakaran:[
      { field: "nama", key: "nama", title: "Nama Bahan", align: "center", renderBodyCell: null, fixed: "left" },
      { field: "takaran", key: "takaran", title: "Takaran Bahan Kering (Kg)", align: "center", renderBodyCell: null, fixed: "left" },
      { field: "takaranBasah", key: "takaranBasah", title: "Takaran Bahan Basah (Kg)", align: "center", renderBodyCell: null, fixed: "left" }
      ],

      tableDataMulti: [],
      tableSapi:[],
      arrResComposition: [],
      tableDataCustom: [],
      dataItems: {},
      dataType: {},
      dataIt: {},
      formData: {
        head: [],
        item: [],
      },
      formDataCustom: {
        head : {
          pk: 0,
          tdn: 0,
          ca: 0,
          p: 0
        },
        item: []
      },
      checkComponent: true,
      result: [],
      resultCustom: [],
      footerData : [
        {
          rowKey: 0,
          nama : 'Total',
          resPercentage: '100%',
          totalPK: 0,
          totalTDN: 0,
          totalCa: 0,
          totalP: 0,
          hargatotal:0
        }
      ],
      footerDataCustom : [
        {
          rowKey: 0,
          nama : 'Total',
          resPercentage: '100%',
          totalPK: 0,
          totalTDN: 0,
          totalCa: 0,
          totalP: 0,
          hargatotal:0
        }
      ]
    };
  },
  async fetch(){
    const res = await this.$axios.get('/type')
    const resItem = await this.$axios.get('/item')
    this.dataType =  res.data
    this.dataItems = resItem.data
  },
  fetchOnServer: false,
  computed: {
    filterItems() {
      // eslint-disable-next-line vue/no-side-effects-in-computed-properties
      this.optionsItem = [];
      if (this.selectedcategory===0) {
        return (this.dataItems);
      }
      else{
        for (const element of this.dataItems) {
        if (element.category === this.selectedcategory) {
          // eslint-disable-next-line vue/no-side-effects-in-computed-properties
          this.optionsItem.push(element)
        }
      }
      return (this.optionsItem)
      }
    },
    filterPbbh() {
      const hund = parseInt(Math.floor(this.selectedweight / 50))
      const num = this.selectedweight - 50*hund
      let inputweight = 0
      if (num === 0) {
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        inputweight = 50*hund
      }
      else {
        inputweight = 50*(hund+1)
      }
      console.log(inputweight);
      // eslint-disable-next-line vue/no-side-effects-in-computed-properties
      this.optionsPbbh = [];
      for (let i = 0; i < this.dataType.length; i++) {
          if (this.dataType[i].gender === this.selectedgender && this.dataType[i].weight === inputweight) {
          // eslint-disable-next-line vue/no-side-effects-in-computed-properties
          this.optionsPbbh.push(this.dataType[i])
        }
      }
      return (this.optionsPbbh)
    },
  },
  methods: {
    generateReport () {
            this.$refs.html2Pdf.generatePdf()
        },
    generateReportCustom () {
        this.$refs.html2PdfCustom.generatePdf()
    },
    async loadDataType() {
      await this.$axios.get('/type')
      .then(res=>{
        this.dataTypeOfMaintenance = res.data
      })
    },
    async loadDataItems() {
      await this.$axios.get('/item')
      .then(res=>{
        this.dataItems = res.data
      })
    },
    onAddSapi(){
      this.tableSapi.push(
        {
          namaSapi: this.namaSapi,
          tipePemeliharaan: this.tipePemeliharaan,
          bobotSapi: this.bobotSapi,
          jenisKelamin: this.jenisKelamin,
          pbbh: this.pbbh,
        }
      )
    },
    onAddItems(){
      const item = this.dataItems.find(x=>x.id===this.selecteditem)
      if (this.tab === 1) {
        const checkitem = this.tableDataCustom.find(x=>x.id===this.selecteditem)
        if (checkitem === undefined) {
          this.tableDataCustom.push(item);
        }
      } else if (this.tab === 0) {
        const checkitem = this.tableDataMulti.find(x=>x.id===this.selecteditem)
        if (checkitem === undefined) {
          this.tableDataMulti.push(item);
        }
      }
    },
    onAddItemsCustom(){
      if (this.tab === 1) {
        this.tableDataCustom.push(
        {
        category: 4,
        nama: "Custom",
        bk: 0,
        cp: 0,
        me: 0,
        tdn: 0,
        ca: 0,
        p: 0,
        minpercentage: 0,
        maxpercentage: 100,
        harga: 0
        },
        )
      } else if (this.tab === 0) {
        this.tableDataMulti.push(
        {
        category: 4,
        nama: "Custom",
        bk: 0,
        cp: 0,
        me: 0,
        tdn: 0,
        ca: 0,
        p: 0,
        minpercentage: 0,
        maxpercentage: 100,
        harga: 0
        },
        )
      }
    },
    onClickDelete(rowIndex){
      if (this.tab === 1) {
        this.tableDataCustom.splice(rowIndex, 1);
      } else if (this.tab === 0) {
        this.tableDataMulti.splice(rowIndex, 1);
      }
    },
    onClickDeleteSapi(rowIndex){
      this.tableSapi.splice(rowIndex, 1);
    },
    onsubmit(){
      if (this.tab === 1) {
        this.formDataCustom.head.pk = parseFloat(this.formDataCustom.head.pk)
        this.formDataCustom.head.tdn = parseFloat(this.formDataCustom.head.tdn)
        this.formDataCustom.head.ca = parseFloat(this.formDataCustom.head.ca)
        this.formDataCustom.head.p = parseFloat(this.formDataCustom.head.p)
        this.formDataCustom.item = this.tableDataCustom
        let sumPK = 0
        let sumTDN = 0
        let sumCa = 0
        let sumP = 0
        this.showDismissibleAlertCustom= false
        this.resultCustom = []
        this.$axios
        .post('/calculatecustom', this.formDataCustom)
        .then(res=>{
          if (res.data.price === null) {
            this.showDismissibleAlertCustom = true
          }
          else {
            this.resultCustom = this.tableDataCustom
            this.resultCustom.forEach(object => {
              object.resPercentage = 0;
            });
            for (let i = 0; i < this.resultCustom.length; i++) {
              this.resultCustom[i].resPercentage = res.data.percentage[i];
              sumPK += this.resultCustom[i].cp * this.resultCustom[i].resPercentage
              sumTDN += this.resultCustom[i].tdn * this.resultCustom[i].resPercentage
              sumCa += this.resultCustom[i].ca * this.resultCustom[i].resPercentage
              sumP += this.resultCustom[i].p * this.resultCustom[i].resPercentage
            }
            this.footerDataCustom[0].totalPK = sumPK.toFixed(2) + '%'
            this.footerDataCustom[0].totalTDN = sumTDN.toFixed(2) + '%'
            this.footerDataCustom[0].totalCa = sumCa.toFixed(2) + '%'
            this.footerDataCustom[0].totalP = sumP.toFixed(2) + '%'
            this.footerDataCustom[0].hargatotal = res.data.price.toFixed()
            }
        })
        .catch(error => {
          // eslint-disable-next-line no-console
          console.log(error)
        })
      } else if (this.tab === 0) {
        this.resMulti = []
        this.arrResComposition = []
        this.arrResult = []
        this.arrFooterData = []
        // eslint-disable-next-line prefer-const
        let arr = []
        this.tableSapi.forEach(sapi => {
          const hund = parseInt(Math.floor(sapi.bobotSapi / 50))
          const num = sapi.bobotSapi - 50*hund
          let inputweight = 0
          if (num === 0) {
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            inputweight = 50*hund
          }
          else {
            inputweight = 50*(hund+1)
          }
          if (sapi.tipePemeliharaan === 1) {
            // eslint-disable-next-line prefer-const
            let dataKonsumsi = {...this.dataType.find(x=>x.weight===inputweight && x.adg === sapi.pbbh && x.gender === sapi.jenisKelamin && x.type === "fattening")}
            dataKonsumsi.namaSapi = sapi.namaSapi
            dataKonsumsi.jenisKelamin = sapi.jenisKelamin
            dataKonsumsi.bobotSapi = sapi.bobotSapi
            arr.push(dataKonsumsi)
          } else if (sapi.tipePemeliharaan === 2) {
            const dataKonsumsi = {...this.dataType.find(x=>x.weight===inputweight && x.gender === sapi.jenisKelamin && x.type === "breeding")}
            arr.push(dataKonsumsi)
          }
        });
        this.formData.head = arr
        this.formData.item = this.tableDataMulti
        this.$axios
        .post('/calculatemulti', this.formData)
        .then(res=>{
          this.resMulti = res.data
          this.resMulti.forEach(ress => {
            const label = ['Protein Kasar (%)', 'TDN (%)', 'Kalsium (%)', 'Fosfor (%)']
            // eslint-disable-next-line prefer-const
            let composition = []
            for (let index = 0; index < 4; index++) {
              composition.push({nama: label[index], value: -ress.nutrition[index].toFixed(1)});
            }
            this.arrResComposition.push(composition)
            if (ress.price !== null) {
              let sumPK = 0
              let sumTDN = 0
              let sumCa = 0
              let sumP = 0
              this.result = []
              this.result = JSON.parse(JSON.stringify(this.tableDataMulti));
              this.result.forEach(object => {
                object.resPercentage = 0;
                object.takaran = 0;
                object.takaranBasah = 0;
              });
              for (let i = 0; i < this.result.length; i++) {
                this.result[i].resPercentage = ress.percentage[i];
                this.result[i].takaran = (this.result[i].resPercentage * ress.wransum).toFixed(2)
                this.result[i].takaranBasah = (this.result[i].resPercentage * ress.wransum/this.result[i].bk).toFixed(2)
                sumPK += this.result[i].cp * this.result[i].resPercentage
                sumTDN += this.result[i].tdn * this.result[i].resPercentage
                sumCa += this.result[i].ca * this.result[i].resPercentage
                sumP += this.result[i].p * this.result[i].resPercentage
              }
              this.arrResult.push(this.result)

              // eslint-disable-next-line prefer-const
              let fd = JSON.parse(JSON.stringify(this.footerData));
              fd[0].totalPK = sumPK.toFixed(2) + '%'
              fd[0].totalTDN = sumTDN.toFixed(2) + '%'
              fd[0].totalCa = sumCa.toFixed(2) + '%'
              fd[0].totalP = sumP.toFixed(2) + '%'
              fd[0].hargatotal = ress.price.toFixed()
              this.arrFooterData.push(fd)
            }
            else {
              this.arrResult.push([])
              this.footerData.push([])
            }
          });
        })
        .catch(error => {
          // eslint-disable-next-line no-console
          console.log(error)
        })
      }
    },
  }
}
</script>
