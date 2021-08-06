<template>
  <div>

    <Table :columns="columns" size="small" :data="tableData" :loading="loading">
    </Table>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "winning_record_page",
  mounted() {
    this.getData();
  },
  methods: {
    UTC_to_BeiJing(utc_datetime) {
      //"2017-03-31T08:02:06Z"
      // 转为正常的时间格式 年-月-日 时:分:秒
      let T_pos = utc_datetime.indexOf("T");
      let Z_pos = utc_datetime.indexOf("Z");
      let year_month_day = utc_datetime.substr(0, T_pos);
      let hour_minute_second = utc_datetime.substr(
        T_pos + 1,
        Z_pos - T_pos - 1
      );
      let new_datetime = year_month_day + " " + hour_minute_second; // 2017-03-31 08:02:06

      // 处理成为时间戳
      let timestamp = new Date(Date.parse(new_datetime));
      timestamp = timestamp.getTime();
      // 增加8个小时，北京时间比utc时间多八个时区
        timestamp = timestamp + 8 * 60 * 60*1000;
      return this.getExactTime(timestamp);
    },
    //时间戳转为时间
    getExactTime(time) {
        var date = new Date(time);
        // var date = new Date(time* 1000);
        var year = date.getFullYear() + '-';
        var month = (date.getMonth()+1 < 10 ? '0' + (date.getMonth()+1) : date.getMonth()+1) + '-';
        var dates = date.getDate() + ' ';
        var hour = date.getHours() + ':';
        var min = date.getMinutes() + ':';
        var second = date.getSeconds();
        return year + month + dates + hour + min + second ;
 },
    getData() {
      if (this.loading) return;
      this.loading = true;
      let that = this;
      axios
        .post("https://qcs8ni.fn.thelarkcloud.com/getWinningRecord")
        .then(function (response) {
          that.tableData = response.data.result;
          that.loading = false;
          console.log(that.tableData); //!!!!!!!!!!!!!!!!!!!!!!!!!!!
        })
        .catch(function (error) {
          // 请求失败处理
          console.log(error);
        });
    },
  },
  data() {
    return {
      loading: false,
      size1: 10, // 设置初始每页可以显示的数据条数
      tableData: [],
      columns: [
        {
          title: "奖品名称",
          key: "prize_name",
          align: "center",
        },
        {
          title: "中奖时间",
          key: "createdAt",
          align: "center",
          render: (h, { row, index }) => {
            return h("div", `${this.UTC_to_BeiJing(row.createdAt)}`);
          },
        },
      ],
    };
  },
};
</script>

<style>
</style>
