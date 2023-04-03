<template>
  <div>
    <el-form ref="form" :model="form" :label-position="'right'" label-width="200px">
      <el-form-item label="Destination:" prop="destination" :rules="[
        { required: true, message: 'Destination不能为空'},
      ]">
        <el-select v-model="form.destination" filterable placeholder="请选择">
          <el-option
              v-for="item in destinationItems"
              :key="item.stationCode"
              :label="item.stationName"
              :value="item.stationCode">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="Earliest delivery date:" prop="edDate" :rules="[
        { required: true, message: 'Date不能为空'},
      ]">
      <el-date-picker
            v-model="form.edDate"
            type="date"
            @change="edDateChange"
            value-format="yyyy-MM-dd"
            placeholder="选择日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="Commodity:" prop="commodity" :rules="[
        { required: true, message: 'Commodity不能为空'},
      ]">
        <el-select v-model="form.commodity" filterable placeholder="请选择" @change=commodityChange>
          <el-option
              v-for="item in commodityItems"
              :key="item.value"
              :label="item.label"
              :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="Speed:" prop="speed" :rules="[
        { required: true, message: 'Speed不能为空'},
      ]">
        <el-select v-model="form.speed" filterable placeholder="请选择">
          <el-option
              v-for="item in speedItems"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <div style="width: 20%; text-align: center;">
        <label label="Overall shipment information:">Overall shipment information:</label>
      </div>

      <el-form-item label="Total no. of pieces:" prop="piece"  :rules="[
        { required: true, message: 'pieces不能为空'},
        { type: 'number', message: 'pieces必须为数字值'}
      ]">
        <el-input type="number" min="1" style="width:18%" oninput="value=value.replace(/[^\d]/g,'')" v-model.number="form.piece"></el-input>
      </el-form-item>
      <el-form-item label="Total weight (kg):" prop="weight" :rules="[
        { required: true, message: 'weight不能为空'},
        { type: 'number', message: 'weight必须为数字值'},
      ]">
        <el-input type="number" v-model.number="form.weight" style="width:18%" oninput="if(isNaN(value)) { value = parseFloat(value) } if(value.indexOf('.')>0){value=value.slice(0,value.indexOf('.')+2)}"></el-input>
      </el-form-item>
      <el-form-item label="Total volume (mc):" prop="volume" :rules="[
        { required: true, message: 'volume不能为空'},
        { type: 'number', message: 'volume必须为数字值'}
      ]">
        <el-input type="number" v-model.number="form.volume" style="width:18%" oninput="if(isNaN(value)) { value = parseFloat(value) } if(value.indexOf('.')>0){value=value.slice(0,value.indexOf('.')+3)}"></el-input>
      </el-form-item>
      <el-form-item>
        <el-checkbox v-model="form.dryIce" border>Dry ice</el-checkbox>
      </el-form-item>
      <el-form-item v-show="tflag" label="Temperature:">
        <el-radio v-model="form.temperature" label="1" border>-20°C ~ -10°C</el-radio>
        <el-radio v-model="form.temperature" label="2" border>2°C ~ 8°C</el-radio>
        <el-radio v-model="form.temperature" label="3" border>15°C ~ 25°C</el-radio>
      </el-form-item>
    </el-form>

    <div style="width: 100%; text-align: center;">
      <el-button type="primary" @click="query">查询</el-button>
    </div>

    <br>
    <el-table
        :data="tableData"
        style="width: 100%;margin-bottom: 20px;"
        row-key="id"
        border
        highlight-current-row
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
      <el-table-column
          prop="carrier"
          label="Carrier"
          sortable
          :formatter="carrierFormat"
          width="180">
      </el-table-column>
      <el-table-column
          prop="origin"
          label="Origin"
          sortable>
      </el-table-column>
      <el-table-column
          prop="stops"
          label="Stops"
          sortable>
      </el-table-column>
      <el-table-column
          prop="destination"
          label="Destination"
          sortable>
      </el-table-column>
      <el-table-column
          prop="departure"
          label="Departure(LAT)"
          sortable
          width="180">
      </el-table-column>
      <el-table-column
          prop="arrival"
          label="Arrival(TOA)"
          sortable>
      </el-table-column>
      <el-table-column
          prop="price"
          label="Price"
          sortable>
      </el-table-column>
      <el-table-column
          prop="surcharge"
          label="Surcharge"
          sortable>
      </el-table-column>
    </el-table>
    <br>
  </div>

</template>

<script>
import {stations} from '@/components/stations'
import axios from 'axios'
import {Loading} from 'element-ui';
import {res} from '@/components/res'

export default {
  name: 'MainQuery',
  props: {},
  data() {
    return {
      tflag : false,
      form: {
        destination: '',
        edDate: new Date(),
        commodity: '',
        speed: '',
        piece: null,
        weight: null,
        volume: null,
        dryIce: false,
        temperature: null
      },
      destinationItems: [],
      commodityItems: [
        {
          value: "General Cargo, not restr|1",
          label: "普货",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        },
        {
          value: "General Cargo, not restr|2",
          label: "危险品",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        },
        {
          value: "Chemicals, not restr, temp-sens|1",
          label: "普货温控货",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        },
        {
          value: "Chemicals, not restr, temp-sens|2",
          label: "危险品温控货",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        },
        {
          value: "li",
          label: "锂电池",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        }
      ],
      speedItems: [],
      destinationMap: {},
      tableData: []//res.data
    };
  },
  created() {
    this.destinationItems = []
    console.log("stations: " + stations)
    stations.stationsResponseData.stationDetails.forEach(item => {
      if (item["stationName"] !== "Shanghai") {
        this.destinationItems.push({
          "stationCode": item["stationCode"],
          "stationName": item["stationCode"] + "(" + item["stationName"] + ")"
        })
      }
      this.destinationMap[item["stationCode"]] = item["stationCode"] + "(" + item["stationName"] + ")"
    })
    // this.tableData.forEach(item => item.hasChildren = true)
    console.log(res)
  },
  methods: {
    edDateChange(val) {
      if (new Date() > this.getDate(val)) {
        this.form.edDate = new Date()
      }
    },
    getDate(strDate) {
      var st = strDate;
      var a = st.split("-");
      var date = new Date(a[0], parseInt(a[1]) - 1, a[2], 0, 0, 0)
      return date;
    },
    commodityChange(val) {
      if(val === 'li') {
        this.$message.error('锂电池暂不支持运输！');
        this.form.commodity = null
      }
      if(val.indexOf('Chemicals, not restr, temp-sens') > -1) {
        this.form.temperature = '3';
        this.tflag = true;
      } else {
        this.tflag = false;
        this.form.temperature = null;
      }
      for (let i = 0; i < this.commodityItems.length; i++) {
        if (this.commodityItems[i].value === val) {
          // this.speedItems = this.commodityItems[i].speedItems
          this.speedItems = this.commodityItems[i].speedItems
          this.speed = this.speedItems[0]
        }
      }
    },
    query() {
      this.$refs["form"].validate((valid) => {
        console.log("valid: " + valid)
        if (valid) {
          let loadingInstance = Loading.service({fullscreen: true});
          // console.log(axios, loadingInstance)
          try {
            const params = {
              "destination": this.form.destination,
              "edDate": this.form.edDate,
              "commodity": this.form.commodity,
              "speed": this.form.speed,
              "piece": parseInt(this.form.piece),
              "weight": parseFloat(this.form.weight),
              "volume": parseFloat(this.form.volume),
              "dryIce": this.form.dryIce,
              "temperature" : this.form.temperature
            }
            console.log(params)
            axios.post('/api/query', params)
                .then(res => {
                  console.log(res)
                  loadingInstance.close();
                  this.tableData = res.data.data
                  this.tableData.forEach(item => {
                    item.origin = this.destinationMap[item.origin]
                    item.destination = this.destinationMap[item.destination]
                    item.children.forEach(it => {
                      it.origin = this.destinationMap[it.origin]
                      it.destination = this.destinationMap[it.destination]
                    })
                  })
                  console.log(this.tableData)
                }).catch(err => {
              loadingInstance.close();
              console.log(err)
            })
          } catch (err) {
            console.log("err: " + err)
            loadingInstance.close();
          }
        } else {
          this.$message.error('参数有误');
          return false;
        }
      });
    },
    carrierFormat(row) {
      return row["carrier"].join("\n")
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.el-table .cell {
  white-space: pre-line !important;
}
</style>
