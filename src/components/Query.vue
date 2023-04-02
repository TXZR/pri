<template>
  <div>
    <el-form ref="form" :model="form" :label-position="'right'" label-width="200px">
      <el-form-item label="Destination:" prop="destination" :rules="[
        { required: true, message: 'Destination不能为空'},
      ]">
        <el-select v-model="form.destination" filterable placeholder="请选择" >
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
      <div style="width: 100%; text-align: center;">
        <label label="Overall shipment information:">Overall shipment information:</label>
      </div>
      
      <el-form-item label="Total no. of pieces:" prop="piece" :rules="[
        { required: true, message: 'pieces不能为空'},
        { type: 'number', message: 'pieces必须为数字值'}
      ]">
        <el-input type="number" min="1" v-model.number="form.piece"></el-input>
      </el-form-item>
      <el-form-item label="Total weight (kg):" prop="weight" :rules="[
        { required: true, message: 'weight不能为空'},
        { type: 'number', message: 'weight必须为数字值'},
      ]">
        <el-input type="number" v-model.number="form.weight"></el-input>
      </el-form-item>
      <el-form-item label="Total volume (mc):" prop="volume" :rules="[
        { required: true, message: 'volume不能为空'},
        { type: 'number', message: 'volume必须为数字值'}
      ]">
        <el-input type="number" v-model.number="form.volume"></el-input>
      </el-form-item>
      <el-form-item>
        <el-checkbox v-model="form.dryIce" border>Dry ice</el-checkbox>
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
        stripe
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
          label="destination"
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
          label="price"
          sortable>
      </el-table-column>
      <el-table-column
          prop="surcharge"
          label="surcharge"
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
      form: {
        destination: '',
        edDate: new Date(),
        commodity: '',
        speed: '',
        piece: null,
        weight: null,
        volume: null,
        dryIce: false,
      },
      destinationItems: [],
      commodityItems: [
        {
          value: "General Cargo, not restr",
          label: "General Cargo, not restr",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        },
        {
          value: "Chemicals, not restr, temp-sens",
          label: "Chemicals, not restr, temp-sens",
          speedItems: [{value: "td.Pro", label: "TD PRO"}, {value: "td.Flash", label: "TD FLASH"}]
        }
      ],
      speedItems: [],
      tableData: []//res.data
    };
  },
  created() {
    this.destinationItems = []
    console.log("stations: " + stations)
    stations.stationsResponseData.stationDetails.forEach(item => {
      this.destinationItems.push({
        "stationCode": item["stationCode"],
        "stationName": item["stationCode"] + "(" + item["stationName"] + ")"
      })
    })
    // this.tableData.forEach(item => item.hasChildren = true)
    console.log(res)
  },
  methods: {
    edDateChange(val) {
      if (new Date() > this.getDate(val)) {
        this.edDate = new Date()
      }
    },
    getDate(strDate) {
      var st = strDate;
      var a = st.split("-");
      var date = new Date(a[0], parseInt(a[1]) - 1, a[2], 0, 0, 0)
      return date;
    },
    dateFormat(date, fmt) { // author: meizz
      var o = {
        "M+": date.getMonth() + 1, // 月份
        "d+": date.getDate(), // 日
        "h+": date.getHours(), // 小时
        "m+": date.getMinutes(), // 分
        "s+": date.getSeconds(), // 秒
        "q+": Math.floor((date.getMonth() + 3) / 3), // 季度
        "S": date.getMilliseconds() // 毫秒
      };
      if (/(y+)/.test(fmt))
        fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
      for (var k in o)
        if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
      return fmt;
    },
    commodityChange(val) {
      for (let i = 0; i < this.commodityItems.length; i++) {
        if (this.commodityItems[i].value == val) {
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
          console.log(axios, loadingInstance)
          try {
            const params = {
              "destination": this.form.destination,
              "edDate": this.dateFormat(this.form.edDate, 'yyyy-MM-dd'),
              "commodity": this.form.commodity,
              "speed": this.form.speed,
              "piece": parseInt(this.form.piece),
              "weight": parseFloat(this.form.weight),
              "volume": parseFloat(this.form.volume),
              "dryIce": this.form.dryIce
            }
            console.log(params)
            axios.post('/api/query', params)
                .then(res => {
                  console.log(res)
                  loadingInstance.close();

                  this.tableData = res.data.data
                  console.log(this.tableData)
                }).catch(err => {
              console.log(err)
            })
          } catch (err) {
            console.log("err: " + err)
            loadingInstance.close();
          }
        } else {
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
.el-table .cell{
  white-space: pre-line !important;
}
</style>
