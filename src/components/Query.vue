<template>
  <div>
  <el-row :gutter="20" style="margin: 0">
    <el-col :span="6">
      <div class="grid-content bg-purple">
        <el-select v-model="destination" filterable placeholder="请选择">
          <el-option
              v-for="item in destinationItems"
              :key="item.stationCode"
              :label="item.stationName"
              :value="item.stationCode">
          </el-option>
        </el-select>
      </div>
    </el-col>
    <el-col :span="6">
      <div class="grid-content bg-purple">
        <el-date-picker
            v-model="edDate"
            type="date"
            @change="edDateChange"
            value-format="yyyy-MM-dd"
            placeholder="选择日期">
        </el-date-picker>
      </div>
    </el-col>
    <el-col :span="6">
      <div class="grid-content bg-purple">
        <el-select v-model="commodity" filterable placeholder="请选择" @change=commodityChange>
          <el-option
              v-for="item in commodityItems"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              >
          </el-option>
        </el-select>
      </div>
    </el-col>
    <el-col :span="6">
      <div class="grid-content bg-purple">
        <el-select v-model="speed" filterable placeholder="请选择">
          <el-option
              v-for="item in speedItems"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </div>
    </el-col>
  </el-row>
  <label label="Overall shipment information:" >Overall shipment information:</label>
  <el-row :gutter="20" style="margin: 0">
    <el-col :span="6" style="text-align: left;">
        Total no. of pieces: 
        <el-input type="number" v-model="piece" placeholder="Total no. of pieces"></el-input>
    </el-col>
    <el-col :span="6" style="text-align: left;">
        Total weight (kg): 
        <el-input type="number" v-model="weight" placeholder="Total weight (kg)"></el-input>
    </el-col>
    <el-col :span="6" style="text-align: left;">
        Total volume (mc):
        <el-input type="number" v-model="volume" placeholder="Total volume (mc)"></el-input>
    </el-col>
  </el-row>
  <el-checkbox v-model="dryIce">Dry ice</el-checkbox>
  <el-button type="primary" @click="query">查询</el-button>

  <br>
  <el-table
        :data="tableData"
        style="width: 100%;margin-bottom: 20px;"
        default-expand-all
        row-key="id"
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
      <el-table-column
          prop="carrier"
          label="Carrier"
          sortable
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
</div>

</template>

<script>
import {stations} from '@/components/stations'
import axios from 'axios'
import { Loading } from 'element-ui';
import {res} from '@/components/res'

export default {
  name: 'MainQuery',
  props: {
  },
  data() {
    return {
      destinationItems: [
        {value : "PVG" , label : "上海"}
      ],
      commodityItems: [
        {value : "General Cargo, not restr" , label : "General Cargo, not restr", speedItems: [{value : "td.Flash" , label : "TD PRO"},]},
        {value : "Chemicals, not restr, temp-sens" , label : "Chemicals, not restr, temp-sens", speedItems: [{value : "td.Pro" , label : "TD FLASH"}]}
      ],
      productItems: [
        {value : "courier" , label : "courier"},
        {value : "courier" , label : "courier"},
        {value : "courier" , label : "courier"}
      ],
      speedItems : [],
      destination : 'FRA',
      edDate : new Date(),
      commodity : '',
      speed : '',
      piece : 0,
      weight : 0,
      volume : 0,
      dryIce : false,
      tableData: res.data
    };
  },
  created() {
    this.destinationItems = []
    console.log("stations: " + stations)
    stations.stationsResponseData.stationDetails.forEach(item => {
      this.destinationItems.push({"stationCode": item["stationCode"], "stationName": item["stationCode"]+"("+item["stationName"]+")"})
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
      var date = new Date(a[0], parseInt(a[1])-1, a[2], 0, 0, 0)
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
            if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
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
      let loadingInstance = Loading.service({ fullscreen: true });
      try{
        const params = {
          "destination": this.destination,
          "edDate": this.dateFormat(this.edDate, 'yyyy-MM-dd'),
          "commodity": this.commodity,
          "speed": this.speed,
          "piece": parseFloat(this.piece),
          "weight": parseFloat(this.weight),
          "volume": parseFloat(this.volume),
          "dryIce": this.dryIce
        }
        console.log(params)
        axios.post('/api/query', params)
          .then(res => {
            console.log(res)
            loadingInstance.close();
            
            this.tableData = res.data.data
            console.log(this.tableData)
          }).catch(err=> {
            console.log(err)
          })
        } catch (err) {
          console.log("err: " + err)
          loadingInstance.close();
        }
      
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
