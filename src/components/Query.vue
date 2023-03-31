<template>
  <div class = "query">
    <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <el-form-item label="Destination:">
        <el-select v-model="destination" filterable placeholder="请选择">
          <el-option
              v-for="item in destinationItems"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="Earliest delivery date:">
        <el-date-picker
            v-model="edDate"
            type="date"
            placeholder="选择日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="Commodity:">
        <el-select v-model="commodity" filterable placeholder="请选择">
          <el-option
              v-for="item in commodityItems"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="Speed:">
        <el-select v-model="speed" filterable placeholder="请选择">
          <el-option
              v-for="item in speedItems"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="Overall shipment information:">
        <el-input v-model="piece" placeholder="Total no. of pieces"></el-input>
        <el-input v-model="weight" placeholder="Total weight (kg)"></el-input>
        <el-input v-model="volume" placeholder="Total volume (mc)"></el-input>
      </el-form-item>
      <el-checkbox v-model="dryIce">Dry ice</el-checkbox>
      <el-button type="primary">查询</el-button>
    </el-form>
    <el-table
        :data="tableData"
        style="width: 100%;margin-bottom: 20px;"
        row-key="id"
        border
        default-expand-all
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
      <el-table-column
          prop="date"
          label="Carrier"
          sortable
          width="180">
      </el-table-column>
      <el-table-column
          prop="name"
          label="Departure(LAT)"
          sortable
          width="180">
      </el-table-column>
      <el-table-column
          prop="address"
          label="Stops"
          sortable>
      </el-table-column>
      <el-table-column
          prop="address"
          label="Arrival(TOA)"
          sortable>
      </el-table-column>
      <el-table-column
          prop="address"
          label="price"
          sortable>
      </el-table-column>
    </el-table>
  </div>

</template>

<script>
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
        {value : "General Cargo, not restr" , label : "General Cargo, not restr"}
      ],
      productItems: [
        {value : "courier" , label : "courier"},
        {value : "courier" , label : "courier"},
        {value : "courier" , label : "courier"}
      ],
      speedItems : [
        {value : "td.Flash" , label : "td.Flash"},
        {value : "td.Pro" , label : "td.Pro"}
      ],
      destination : '',
      edDate : '',
      commodity : '',
      speed : '',
      piece : 0,
      weight : 0,
      volume : 0,
      dryIce : false
    };
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
