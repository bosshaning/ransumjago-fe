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

    <div class="mt-3">
      <p><b>Kategori Ransum</b></p>
      <b-form-select v-model="selectedransum" :options="optionsransum"></b-form-select>
      <p>{{ optionsransum[selectedransum-1].rule }}</p>
    </div>

    <div>
      <p><b>Kondisi Sapi</b></p>
      <div class="mt-3">
        <p>Tipe Pemeliharaan</p>
        <b-form-select v-model="selectedtype" :options="optionstype"></b-form-select>
      </div>
      <div class="mt-3">
        <p>Bobot Sapi (Kg)</p>
        <b-form-select v-model="selectedweight" :options="optionsweight"></b-form-select>
      </div>
      <div class="mt-3">
        <p>Jenis Kelamin Sapi</p>
        <b-form-select v-model="selectedgender" :options="optionsgender"></b-form-select>
      </div>
      <div v-if="selectedtype === 1" class="mt-3">
        <p>PBBH yang Diinginkan (Kg)</p>
        <b-form-select v-model="selectedpbbh" required>
          <option v-for="option in filterPbbh" :key="option.adg" :value="option.id">
        <p>{{ option.adg }}</p>
      </option>
        </b-form-select>
      </div>
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
      <button v-if="selecteditem !== null" class="mt-2" @click.prevent="onAddItems()">+ Bahan Pakan</button>
      <ve-table class="mt-3" :fixed-header="true" :columns="columns" :table-data="tableData" />
    </div>
    <div>
      <button v-if="selecteditem !== null" class="mt-2" @click.prevent="onsubmit()">Kalkulasi</button>
    </div>
    <div class="mt-5">
      <p><b>Hasil Optimasi Ransum</b></p>
      <p>Berat Kering : {{ resCalculate[0] }} kg</p>
      <p>Nutrisi yang Harus Diberikan:</p>
      <ve-table class="mt-3" :columns="columnsNutrient" :table-data="resComposition" />
      <ve-table class="mt-3" :fixed-header="true" :footer-data="footerData"
      rowKeyFieldName="rowKey" :columns="resColumns" :table-data="result" />
      <p>Takaran Pemberian:</p>
      <ve-table class="mt-3" :columns="columnsTakaran" :table-data="result" />
    </div>
  </div>
  </div>
</template>

<script>
import '~/css/styles.css'
export default {
  name: 'IndexPage',
  data() {
    return {
      selectedtype: 1,
      selectedransum: 1,
      selectedcategory: 0,
      selectedweight: 100,
      selectedgender: 0,
      selectedpbbh: null,
      selecteditem: null,
      weight: null,
      resCalculate: [],
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
        {value: 3, text: "Sumber Protein"}
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
      columns: [
        { field: "nama", key: "a", title: "Nama Bahan", align: "center", renderBodyCell: null, fixed: "left" },
        { field: "category", key: "b", title: "Kategori", align: "left", renderBodyCell: ({ row, column, rowIndex }, h) => {
          if (row.category === 1) {
            return "Hijauan"
          } else if (row.category === 2) {
            return "Sumber Karbohidrat"
          } else if (row.category === 3) {
            return "Sumber Protein"
          }
         }},
        { field: "cp", key: "c", title: "Protein Kasar (%)", align: "center", renderBodyCell: null},
        { field: "tdn", key: "tdn", title: "TDN (%)", align: "center", renderBodyCell: null},
        { field: "ca", key: "ca", title: "Kalsium (%)", align: "center", renderBodyCell: null},
        { field: "p", key: "p", title: "Fosfor (%)", align: "center", renderBodyCell: null},
        { field: "minpercentage", key: "f", title: "Kandungan Minimal (%)", align: "center", renderBodyCell: ({ row, column, rowIndex }, h) => {
          const text = row[column.field]
          return h('input', {
          attrs: {
            type: 'number',
            value: text
          },
          on: {
            change: (event) => {
              row.minpercentage = parseInt(event.target.value)
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
              row.maxpercentage = parseInt(event.target.value)
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
              row.harga = parseInt(event.target.value)
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
        return ((row.cp*row.resPercentage).toFixed(2) + '%')
        }},
        { field: "totalTDN", key: "totalTDN", title: "Total TDN", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.tdn*row.resPercentage).toFixed(2) + '%')
        }},
        { field: "totalCa", key: "totalCa", title: "Total Ca", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.ca*row.resPercentage).toFixed(2) + '%')
        }},
        { field: "totalP", key: "totalP", title: "Total P", align: "center",renderBodyCell: ({ row, column, rowIndex }, h) => {
        return ((row.p*row.resPercentage).toFixed(2) + '%')
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
      { field: "takaran", key: "takaran", title: "Takaran (Kg)", align: "center", renderBodyCell: null, fixed: "left" }
      ],
      tableData: [],
      dataItems: {},
      dataType: {},
      dataIt: {},
      formData: {
        head: [],
        item: []
      },
      checkComponent: true,
      result: [],
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
      // eslint-disable-next-line vue/no-side-effects-in-computed-properties
      this.optionsPbbh = [];
      for (let i = 0; i < this.dataType.length; i++) {
          if (this.dataType[i].gender === this.selectedgender && this.dataType[i].weight === this.selectedweight) {
          // eslint-disable-next-line vue/no-side-effects-in-computed-properties
          this.optionsPbbh.push(this.dataType[i])
        }
      }
      return (this.optionsPbbh)
    },
  },
  methods: {
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
    onAddItems(){
      const item = this.dataItems.find(x=>x.id===this.selecteditem)
      const checkitem = this.tableData.find(x=>x.id===this.selecteditem)
      if (checkitem === undefined) {
        this.tableData.push(item);
      }
    },
    onClickDelete(rowIndex){
      this.tableData.splice(rowIndex, 1);
    },
    checkComponentRansum(){
      this.checkComponent = true
      const hijauan = this.tableData.find(x=>x.category===1)
        const skarbo = this.tableData.find(x=>x.category===2)
        const sprotein = this.tableData.find(x=>x.category===3)
      if (this.selectedransum === 1) {
        if (hijauan === null || skarbo === null || sprotein === null) {
          this.checkComponent = false
        }
      }
      if (this.selectedransum === 2) {
        if (skarbo === null || sprotein === null) {
          this.checkComponent = false
        }
      }
    },
    onsubmit(){
      this.result = []
      this.checkComponentRansum()
      if (this.checkComponent) {
        const item = this.dataType.find(x=>x.id===this.selectedpbbh)
        this.formData.head = item
        this.formData.item = this.tableData
        let sumPK = 0
        let sumTDN = 0
        let sumCa = 0
        let sumP = 0
        this.$axios
        .post('/calculate', this.formData)
        .then(res => {
          this.resCalculate = res.data
          this.result = this.tableData
          this.result.forEach(object => {
            object.resPercentage = 0;
            object.takaran = 0;
          });
          for (let i = 0; i < this.result.length; i++) {
            this.result[i].resPercentage = res.data[3][i];
            this.result[i].takaran = (this.result[i].resPercentage * res.data[0]).toFixed(2)
            sumPK += this.result[i].cp * this.result[i].resPercentage
            sumTDN += this.result[i].tdn * this.result[i].resPercentage
            sumCa += this.result[i].ca * this.result[i].resPercentage
            sumP += this.result[i].p * this.result[i].resPercentage
          }
          this.footerData[0].totalPK = sumPK.toFixed(2) + '%'
          this.footerData[0].totalTDN = sumTDN.toFixed(2) + '%'
          this.footerData[0].totalCa = sumCa.toFixed(2) + '%'
          this.footerData[0].totalP = sumP.toFixed(2) + '%'
          this.footerData[0].hargatotal = res.data[2].toFixed()
          for (let index = 0; index < this.resComposition.length; index++) {
            this.resComposition[index].value = -res.data[1][index].toFixed(1)
          }
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
