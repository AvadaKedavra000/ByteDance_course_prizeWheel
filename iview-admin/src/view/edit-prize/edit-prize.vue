<template>
  <div>
    <Card style="font-size: 30px">
      <p>当前中奖概率和={{ totalProbability }}</p>
      <Alert v-if="totalProbability == 1" show-icon type="success"
        >中奖概率之和已为1</Alert
      >
      <Alert show-icon type="error" v-else>
        请将信息补充完整并将中奖概率(中奖品+未中奖)之和调整为1,否则抽奖程序无法正常运行!
      </Alert>
    </Card>
    <Card style="font-size:20px;margin-top:10px">
      <p >项目地址:</p>
      <a @click="linkDownload('https://prize-wheel.web.cloudendpoint.cn')">
      https://prize-wheel.web.cloudendpoint.cn
      </a>
      </Card>

    <div
      style="
        display: flex;
        justify-content: space-between;
        margin-top: 30px;
        margin-bottom: 10px;
      "
    >
      <!-- 左侧按钮 -->
      <Button
        :disabled="this.isMoving"
        type="primary"
        size="large"
        @click="addOneRow"
        >添加一行</Button
      >
      <!-- 右侧按钮 -->
      <div>
        <Button
          v-if="!this.isMoving"
          type="primary"
          size="large"
          @click="reorder"
          >调整奖品顺序</Button
        >
        <Button
          v-if="this.isMoving"
          type="success"
          size="large"
          style="margin-right: 10px"
          @click="saveReorder"
          >保存调整</Button
        >
        <Button
          v-if="this.isMoving"
          type="error"
          size="large"
          @click="cancelReorder"
          >取消调整</Button
        >
      </div>
    </div>
    <Table :columns="columns" size="small" :data="tableData" :loading="loading">
    </Table>
  </div>
</template>

<script>
import { errorReq } from "@/api/data";
import { getTableData } from "@/api/data";
import axios from "axios";
export default {
  name: "edit_prize_page",
  mounted() {
    this.getData();
  },
  methods: {
    // 新窗口打开外链接
    linkDownload(url) {
      window.open(url, "_blank");
    },
    //点击调整奖品顺序后
    reorder() {
      this.editIndex = -1;
      this.isMoving = true;
      // 不能用this.originalTableData=this.tableData 否则它们会指向同一处地址，导致它们同步变化
      this.originalTableData = JSON.parse(JSON.stringify(this.tableData));
    },
    saveReorder() {
      //保存调整奖品顺序
      // console.log(this.tableData);
      axios
        .post(`https://qcs8ni.fn.thelarkcloud.com/reorder`, {
          reorderTableData: this.tableData,
        })
        .then((res) => {
          if (res.data.result) {
            this.$Message.success("保存调整成功");
          } else {
            this.$Message.error("请求失败");
          }
        });
      this.isMoving = false;
    },
    cancelReorder() {
      // console.log(this.tableData);
      //取消调整奖品顺序
      for (let i = 0; i < this.tableData.length; i++) {
        this.$set(this.tableData, i, this.originalTableData[i]);
      }
      this.isMoving = false;
    },
    getData() {
      if (this.loading) return;
      this.loading = true;
      getTableData()
        .then((res) => {
          this.tableData = res.data.result;
          console.log(res.data.result);
          this.loading = false;
        })
        .catch((err) => {
          this.$Message.error("请求失败");
          console.log(err);
        });
    },
    addOneRow() {
      axios
        .post(`https://qcs8ni.fn.thelarkcloud.com/addOrUpdateOnePrize`, {
          //现在是添加一空行，不用传出_id
        })
        .then((res) => {
          if (res.data.result) {
            this.getData();
            this.$Message.success("添加成功");
          } else {
            this.$Message.error("请求失败");
          }
        });
    },
  },
  data() {
    return {
      isMoving: false, //是否正在调整表格行顺序
      editIndex: -1,
      edit_prize_id: "",
      edit_prize_name: "",
      edit_prize_amout: null,
      edit_prize_winning_info: "",
      edit_prize_img_url: "",
      edit_prize_probability: null,
      loading: false,
      size1: 10, // 设置初始每页可以显示的数据条数
      tableData: [],
      originalTableData: [], //原始表格数据，用于表格数据调整
      columns: [
        {
          title: "奖品顺序(在转盘上逆时针排列)",
          key: "prize_index",
          align: "center",
        },
        {
          title: "奖品id",
          key: "_id",
          align: "center",
        },

        {
          title: "奖品名称",
          key: "prize_name",
          align: "center",
          render: (h, { row, index }) => {
            let edit;
            if (this.editIndex === index) {
              this.edit_prize_name = row.prize_name;
              edit = [
                h("Input", {
                  props: {
                    value: row.prize_name,
                  },
                  on: {
                    input: (val) => {
                      this.edit_prize_name = val;
                    },
                  },
                }),
              ];
            } else {
              edit = row.prize_name;
            }
            return h("div", [edit]);
          },
        },
        {
          title: "奖品数量",
          key: "prize_amout",
          align: "center",
          render: (h, { row, index }) => {
            let edit;
            if (this.editIndex === index) {
              this.edit_prize_amout = row.prize_amout;
              edit = [
                h("Input", {
                  props: {
                    value: row.prize_amout,
                  },
                  on: {
                    input: (val) => {
                      this.edit_prize_amout = val;
                    },
                  },
                }),
              ];
            } else {
              edit = row.prize_amout;
            }
            return h("div", [edit]);
          },
        },
        {
          title: "中奖信息",
          key: "prize_winning_info",
          align: "center",
          render: (h, { row, index }) => {
            let edit;
            if (this.editIndex === index) {
              this.edit_prize_winning_info = row.prize_winning_info;
              edit = [
                h("Input", {
                  props: {
                    value: row.prize_winning_info,
                  },
                  on: {
                    input: (val) => {
                      this.edit_prize_winning_info = val;
                    },
                  },
                }),
              ];
            } else {
              edit = row.prize_winning_info;
            }
            return h("div", [edit]);
          },
        },
        {
          title: "奖品图片地址",
          key: "prize_img_url",
          align: "center",
          render: (h, { row, index }) => {
            let edit;
            if (this.editIndex === index) {
              this.edit_prize_img_url = row.prize_img_url;
              edit = [
                h("Input", {
                  props: {
                    value: row.prize_img_url,
                  },
                  on: {
                    input: (val) => {
                      this.edit_prize_img_url = val;
                    },
                  },
                }),
              ];
            } else {
              edit = row.prize_img_url;
            }
            return h("div", [edit]);
          },
        },
        {
          title: "中奖概率",
          key: "prize_probability",
          align: "center",
          render: (h, { row, index }) => {
            let edit;
            if (this.editIndex === index) {
              this.edit_prize_probability = row.prize_probability;
              edit = [
                h("Input", {
                  props: {
                    value: row.prize_probability,
                  },
                  on: {
                    input: (val) => {
                      this.edit_prize_probability = val;
                    },
                  },
                }),
              ];
            } else {
              edit = row.prize_probability;
            }
            return h("div", [edit]);
          },
        },
        {
          title: "操作",
          align: "center",
          render: (h, { row, index }) => {
            if (this.isMoving) {
              //若正在调整，操作区显示向上和向下按钮
              return [
                h(
                  "Button", //"向上"按钮
                  {
                    props: {
                      type: "primary",
                    },
                    on: {
                      click: () => {
                        if (index === 0) {
                          this.$Message.error("已是列表中第一行！");
                        } else {
                          //先交换一次prize_index,再整体交换
                          this.tableData[index - 1].prize_index += 1;
                          this.tableData[index].prize_index -= 1;

                          let temp = this.tableData[index - 1];
                          this.$set(
                            this.tableData,
                            index - 1,
                            this.tableData[index]
                          );
                          this.$set(this.tableData, index, temp);
                        }
                      },
                    },
                  },
                  "向上"
                ),
                h(
                  "Button", //"向下"按钮
                  {
                    props: {
                      type: "primary",
                    },
                    style: {
                      marginLeft: "6px",
                    },
                    on: {
                      click: () => {
                        if (index === this.tableData.length - 1) {
                          this.$Message.error("已是列表中最后一行！");
                        } else {
                          //先交换一次prize_index,再整体交换
                          this.tableData[index + 1].prize_index -= 1;
                          this.tableData[index].prize_index += 1;

                          let temp = this.tableData[index + 1];
                          this.$set(
                            this.tableData,
                            index + 1,
                            this.tableData[index]
                          );
                          this.$set(this.tableData, index, temp);
                        }
                      },
                    },
                  },
                  "向下"
                ),
              ];
            } else {
              //若点击表格中某行的“编辑”按钮，this.editIndex变为当前index，
              //这一单元格渲染出"保存"和"取消"两个按钮，同时该行信息单元变为输入框以供修改
              if (this.editIndex === index) {
                return [
                  h(
                    "Button",
                    {
                      props: {
                        type: "success",
                      },
                      on: {
                        //点击"保存按钮"
                        click: () => {
                          let reg1 = /^[0-9]+?$/; //库存数量需为非负整数
                          if (
                            !reg1.test(this.edit_prize_amout) &&
                            !(this.edit_prize_amout === null) &&
                            !(this.edit_prize_amout === "") &&
                            !(this.edit_prize_amout === undefined)
                          ) {
                            this.$Message.error("奖品数量非法");
                            return;
                          }
                          let reg2 = /^(?:0\.\d+|[01](?:\.0)?)$/; //中奖概率需为≤1且≥0的小数
                          if (
                            !reg2.test(this.edit_prize_probability) &&
                            !(this.edit_prize_probability === null) &&
                            !(this.edit_prize_probability === "") &&
                            !(this.edit_prize_probability === undefined)
                          ) {
                            this.$Message.error("中奖概率非法");
                            return;
                          }
                          row.prize_name = this.tableData[index].prize_name =
                            this.edit_prize_name;
                          row.prize_amout = this.tableData[index].prize_amout =
                            Number(this.edit_prize_amout);
                          row.prize_winning_info = this.tableData[
                            index
                          ].prize_winning_info = this.edit_prize_winning_info;
                          row.prize_img_url = this.tableData[
                            index
                          ].prize_img_url = this.edit_prize_img_url;

                          row.prize_probability = this.tableData[
                            index
                          ].prize_probability = Number(
                            this.edit_prize_probability
                          );

                          // console.log('保存后',this.tableData[index].prize_probability);

                          this.editIndex = -1;
                          //进行封装!!!!!!!
                          axios
                            .post(
                              `https://qcs8ni.fn.thelarkcloud.com/addOrUpdateOnePrize`,
                              {
                                _id: row._id, //现在是更改某一行，直接获取其Id
                                prize_name: this.edit_prize_name,
                                prize_amout: Number(this.edit_prize_amout),
                                prize_winning_info:
                                  this.edit_prize_winning_info,
                                prize_img_url: this.edit_prize_img_url,
                                prize_probability: Number(
                                  this.edit_prize_probability
                                ),
                              }
                            )
                            .then((res) => {
                              if (res.data.result) {
                                this.$Message.success("修改成功");
                              } else {
                                this.$Message.error("请求失败");
                              }
                            });
                        },
                      },
                    },
                    "保存"
                  ),
                  h(
                    "Button",
                    {
                      props: {
                        type: "error",
                      },
                      style: {
                        marginLeft: "6px",
                      },
                      on: {
                        //点击"取消"按钮
                        click: () => {
                          this.editIndex = -1;
                        },
                      },
                    },
                    "取消"
                  ),
                ];
              } else {
                //否则，没选中的行渲染"编辑"按钮
                return [
                  h(
                    "Button",
                    {
                      props: {
                        type: "primary",
                      },
                      on: {
                        //点击"编辑"按钮
                        click: () => {
                          this.editIndex = index;
                          this.edit_prize_name = row.prize_name;
                          this.edit_prize_amout = row.prize_amout;
                          this.edit_prize_winning_info = row.prize_winning_info;
                          this.edit_prize_img_url = row.prize_img_url;
                          this.edit_prize_probability = row.prize_probability;
                        },
                      },
                    },
                    "编辑"
                  ),
                  h(
                    "Button",
                    {
                      props: {
                        type: "error",
                      },
                      style: {
                        marginLeft: "6px",
                      },
                      on: {
                        //点击"删除"按钮
                        click: () => {
                          axios
                            .post(
                              `https://qcs8ni.fn.thelarkcloud.com/deleteOnePrize`,
                              {
                                _id: row._id, //现在是删除某一行，只需传出其Id
                              }
                            )
                            .then((res) => {
                              if (res.data.result) {
                                this.tableData.splice(index, 1);
                                this.$Message.success("删除成功");
                              } else {
                                this.$Message.error("请求失败");
                              }
                            });
                        },
                      },
                    },
                    "删除"
                  ),
                ];
              }
            }
          },
        },
      ],
    };
  },
  computed: {
    totalProbability() {
      let res = 0;
      for (const item of this.tableData) {
        // console.log(item.prize_probability);
        res += item.prize_probability;
      }
      res = res.toFixed(2);
      return res;
    },
  },
};
</script>

<style>
</style>
