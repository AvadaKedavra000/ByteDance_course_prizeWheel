<template>
  <div>
    <div class="overall">
      <div class="wheel-box">
        <div class="light">
          <ul class="lightList">
            <li
              class="lightItem"
              v-for="(item, index) in lightItem"
              :key="index"
              :style="`transform: rotate(${
                (360 / lightItem.length) * index
              }deg) translateY(-188px) `"
            ></li>
          </ul>
        </div>
        <div class="wheel">
          <!-- .unit-box说明:
          第一个unit skew之后，底边在x轴上，左边由垂直x轴变为向右倾斜
            故整体来看第一个unit盒子在转盘中心的右上方
            为方便起见
            第一个unit盒子的左下角已对准转盘中心，
            用transform: rotate(${-(90 - (360/list.length)/2)}deg),令其角平分线与y轴重合
            也即令第一个unit盒子在转盘中心的正上方
            其他unit盒子也相应旋转-->
          <div
            class="unit-box"
            :style="` transform: rotate(${-90 + 180 / prizeInfo.length}deg)`"
          >
            <!-- .unit说明:
        利用skew可以改变盒子左下角的角度，从而配合rotate实现转盘等分 -->
            <div
              class="unit"
              v-for="(i, index) in prizeInfo"
              :key="index"
              :style="` 
            transform:rotate(${(-index * 360) / prizeInfo.length}deg)  skew(${
                -90 + 360 / prizeInfo.length
              }deg) ;`"
            ></div>
          </div>
          <!-- .prize说明:
        rotate同.unit
        translateY(-7rem)可以使得.prize从屏幕中心"散开" -->
          <div
            class="prize"
            :style="`transform: rotate(${
              (-index * 360) / prizeInfo.length
            }deg) translateY(-7rem);`"
            v-for="(item, index) in prizeInfo"
            :key="index"
          >
            <span class="prizeName">{{ item.prize_name }}</span>
            <div class="prizeImg">
              <img :src="item.prize_img_url" alt />
            </div>
          </div>
        </div>
        <div class="start-btn" @click="go()">抽奖</div>
      </div>
    </div>
    <Modal
      :title="this.prizeInfo[this.winner].prize_winning_info"
      :show="show"
      @submit="submit"
    >
      <img
        style="height: 100px; text-align: center"
        :src="this.prizeInfo[this.winner].prize_img_url"
        alt=""
      />
    </Modal>
  </div>
</template>

<script>
import axios from "axios";
import Modal from "./Modal.vue";
export default {
  name: "wheel",
  components: {
    Modal,
  },
  data() {
    return {
      show: false,
      prizeInfo: [],
      lightItem: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16],
      winner: 2, //指定获奖下标 specified为true时生效
      isGo: false, //抽奖执行状态，防止用户多次点击
      wheelElement: null,
    };
  },
  mounted() {
    //获取奖品信息
    this.getPrizeInfo();
  },
  methods: {
    //从后端获取奖品信息
    getPrizeInfo() {
      axios
        .post("https://qcs8ni.fn.thelarkcloud.com/getPrizeInfo")
        .then((response) => {
          //给this.prizeInfo赋值
          this.prizeInfo = response.data.result;
          console.log(this.prizeInfo); //!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
        })
        .catch(function (error) {
          // 请求失败处理
          console.log(error);
        });
    },
    //开始抽奖
    go() {
      if (!this.isGo) {
        this.isGo = true;
        this.wheelElement = document.querySelector(".wheel"); //获取.wheel
        this.wheelElement.classList.remove("wr"); //先把原动画类移除
        this.winCallback();
      }
    },

    //中奖返回方法
    winCallback() {
      axios
        .post(`https://qcs8ni.fn.thelarkcloud.com/winOnePrize`)
        .then((res) => {
          if (res.data.result) {
            console.log(res.data.result);
            this.winner = res.data.result.prize_index;
            console.log("抽奖成功");

            //计算出要执行的CSS动画
            //将奖项数目赋值给--nums,中奖下标赋值给--winner,以便CSS计算
            let root = document.querySelector(":root");
            root.style.setProperty("--nums", this.prizeInfo.length);
            root.style.setProperty("--winner", this.winner);

            setTimeout(() => {
              /* 此处是为了解决当下次抽中的奖励与这次相同，动画不重新执行的 */
              /* 添加一个定时器，是为了解决动画属性的替换效果，实现动画的重新执行 */
              this.wheelElement.classList.add("wr"); //添加动画类
            }, 0);
            // 因为动画时间为 5s ，所以这里5s后获取结果，其实结果早就定下了，只是何时显示，告诉用户
            setTimeout(() => {
              this.isGo = false;
              console.log(`恭喜你获得了第${this.winner}项奖品`);
              console.log(`${this.prizeInfo[this.winner].prize_winning_info}`);
              this.show = true;
            }, 5000);
          } else {
            console.log("抽奖失败");
            alert("抽奖失败");
          }
        });
    },
    // 确认弹窗回调
    submit() {
      this.show = false;
    },
  },
};
</script>


<style lang="scss" scoped>
$wheelSize: 24rem; //转盘尺寸
$buttonSize: 6rem; //抽奖按钮尺寸
$time: 5s; //转动多少秒后停下的时间

.wheel-box {
  display: flex;
  justify-content: center;
  align-items: center;
  width: $wheelSize;
  height: $wheelSize;
  user-select: none; //不可选择文字

  .light {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;

    .lightItem {
      position: absolute;
      display: block;
      width: 10px;
      height: 10px;
      background: #fff;
      top: 50%;
      left: 50%;
      margin: -4px 0 0 -4px;
      border-radius: 4px;
      z-index: 999;
      animation: twinkling 1.2s infinite ease-in-out;
    }
    .lightItem:nth-child(2n + 1) {
      animation-delay: 0.6s;
    }
    @keyframes twinkling {
      0% {
        background: #fefdfc;
        border: 1px solid #fefdfc;
      }
      25% {
        background: #fefdfc;
        border: none;
        box-shadow: 0px 0px 0 2px #3624b8;
      }
      75% {
        background: #31a3dd;
        border: 1px solid #31a3dd;
      }
      100% {
        background: #31a3dd;
        border: none;
        box-shadow: 0px 0px 0 2px #3b589c;
      }
    }
  }
  /* 抽奖按钮 */
  .start-btn {
    // display: inline-block;

    //position: relative;
    //z-index: 2;
    cursor: pointer;

    width: $buttonSize;
    height: $buttonSize;
    border-radius: 50%;
    background-color: #3923dd;

    color: white;
    font-size: 30px;
    font-weight: bold;
    box-sizing: border-box;
    position: relative;
    z-index: 2;

    display: flex;
    justify-content: center;
    align-items: center;
    //奖盘指针
    &::before {
      content: "";
      //绘制三角形
      width: 0;
      height: 0;
      border-right: 3rem solid transparent;
      border-bottom: 4rem solid #3923dd;
      border-left: 3rem solid transparent;
      //定位
      position: absolute;
      top: -1.5rem;
      z-index: -1;
    }
  }
  /* 转盘 */
  .wheel {
    overflow: hidden;
    border-radius: 50%;
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;

    display: flex;
    justify-content: center;
    align-items: center;

    border: 10px solid #cae0f7;
    box-sizing: border-box;
    /* 每个奖项的样式 */
    .prize {
      position: absolute;
      .prizeName {
        font-weight: bold;
        font-size: 18px;
        color: #3b3b3b;
      }
      .prizeImg {
        margin: 0.5rem auto;
        width: 3rem;
        height: 3rem;
        line-height: 3rem;
        // border: 2px dashed #4b56be;
        overflow: hidden;
        color: white;
        display: flex;
        justify-content: center;
        img {
          height: 100%;
        }
      }
    }
  }
  .unit-box {
    overflow: hidden;
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;

    border-radius: 50%;

    /* 转盘扇形单元 */
    .unit {
      box-sizing: border-box;
      width: 100%;
      height: 100%;
      opacity: 1;

      /* 
      第一个unit盒子为水平垂直居中，由于利用skew可以改变盒子左下角的角度，从而配合rotate实现转盘等分， 
      故而用 transform-origin: bottom left 将旋转原点设置为左下角。 
      同时，将该unit盒子的左下角的旋转原点挪动到屏幕中心，用position:absolute;top: -50%;left: 50%;实现 
      */
      transform-origin: bottom left;
      position: absolute;
      top: -50%;
      left: 50%;
    }
    /* 转盘善行单元色 */
    .unit:nth-child(2n) {
      background: #e5efff;
    }
    .unit:nth-child(2n + 1) {
      background: #4e88c7;
      box-shadow: 0 0 5px blue;
    }
  }

  /* 下面的css样式为每个奖品的旋转动画，这里写了对应8个奖品的动画，如果想要更多的话，可以添加 */
  /* 例如： .wr8  @keyframes play8 */
  .wr {
    animation-duration: $time;
    animation-timing-function: ease;
    animation-fill-mode: both;
    animation-iteration-count: 1;
    animation-name: play;
  }
  @keyframes play {
    to {
      transform: rotate(
        calc(5 * 360deg + 360deg / var(--nums) * var(--winner))
      );
    }
  }
}
</style>