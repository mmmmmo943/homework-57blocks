<script setup>
import { reactive,ref } from 'vue';
import Main from './table.vue'

//上栏时间范围列表
const time_range = [
  {
    title: '00:00 - 12:00',
    hours: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
  },
  {
    title: '12:00 - 24:00',
    hours: [12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]
  }
]


//从main组件传过来的已选中的时间信息
let data_selected=ref([])
const get_selected = (data) => {
  data_selected.value=data
  console.log('data_getSelected', data_selected.value)
}

const main=ref(null)//main组件的引用ref
//清除按钮
const clear=()=>{
  data_selected.value=[]
  main.value.clear_output()
}
//选择周内黄金时间函数
const gold_weekday=()=>{
  data_selected.value=[]
  main.value.weekday_output()
}
//选择周末黄金时间函数
const gold_weekend=()=>{
  data_selected.value=[]
  main.value.weekdend_output()
}
</script>

<template>
  <div class="container">
    <div class="table">
      <div class="top">
        <div style="width:10px;height:2px;background-color:cornflowerblue;margin-right: 10px;"></div>
        <div style="margin-right: 10px;">已选</div>
        <div style="width:10px;height:2px;background-color:gray;margin-right: 10px"></div>
        <div style="margin-right: 10px;">可选</div>
      </div>
      <div class="label-box">星期/时间</div>
      <div class="time-box">
        <div class="time-box-item" v-for="(item, index) in time_range" :key="index">
          <div class="time-box-item-label">{{ item.title }}</div>
          <div class="time-box-item-cells">
            <div class="cell" v-for="(ele, index_inner) in item.hours" :key="index_inner">{{ ele }}</div>
          </div>
        </div>
      </div>
      <div class="main">
        <Main @updateData="get_selected" ref="main"/>
      </div>
    </div>
    <div class="select-box">
      <div class="select-box-top">
        <div style="flex:4;display:flex;justify-content:center;align-items:center">已选择时间段</div>
        <div @click="gold_weekday" style="flex:1;display:flex;justify-content:center;align-items:center;color: cornflowerblue;cursor: pointer;">周内黄金时间</div>
        <div @click="gold_weekend" style="flex:1;display:flex;justify-content:center;align-items:center;color: cornflowerblue;cursor: pointer;">周末黄金时间</div>
        <div @click="clear" style="flex:1;display:flex;justify-content:center;align-items:center;color: cornflowerblue;cursor: pointer;">清空</div>   
      </div>
      <div class="select-box-content">
        <div class="select-cells" v-for="(item,index) in data_selected" :key="index" >
          <div class="select-day">{{item.day}}</div>
          <div class="select-times">{{item.list.join(',')}}</div>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped lang="scss">
.container {
  width: 700px;
  height: auto;
  margin: 0 auto;
  background: white;

  .table {
    width: 100%;
    height: 400px;
    display: grid;
    grid-template-columns: 1fr 7fr;
    grid-template-rows: 0.5fr 1fr 3.5fr;
    grid-template-areas: "top top"
      "label-box time-box"
      "main main";

    .top {
      grid-area: top;
      display: flex;
      justify-content: flex-end;
      color: darkgray;
      align-items: center;
    }

    .label-box {
      grid-area: label-box;
      color: darkgray;
      font-weight: bold;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .time-box {
      grid-area: time-box;
      display: flex;
      color: darkgray;

      .time-box-item {
        width: 50%;
        height: 100%;
        border-right: 0.5px solid rgb(179, 177, 177);

        .time-box-item-label {
          border-bottom: 0.5px solid rgb(179, 177, 177);
          height: 50%;
          display: flex;
          justify-content: center;
          align-items: center;
        }

        .time-box-item-cells {
          display: flex;
          width: 100%;
          height: 50%;

          .cell {
            height: 100%;
            flex: 1;
            border-right: 0.5px solid rgb(179, 177, 177);
            display: flex;
            justify-content: center;
            align-items: center;
          }
        }

      }

    }

    .main {
      grid-area: main;
    }

  }

  .table>* {
    border: 0.5px solid rgb(179, 177, 177);
  }

  .select-box {
    width: 100%;
    height: auto;
    display: flex;
    flex-direction: column;
    color: darkgray;

    .select-box-top {
      display: flex;
      width: 100%;
      height: 30px;
      border-bottom: 1px solid rgb(179, 177, 177);

    }

    .select-box-content {
      width: 100%;
      height: auto;
      display: flex;
      flex-direction: column;

      .select-cells {
        width: 100%;
        min-height: 40px;
        display: flex;
        border-bottom: 0.5px solid rgb(179, 177, 177);
        

        .select-day {
          flex: 1;
          display: flex;
          justify-content: center;
          align-items: center;
          border-right: 0.5px solid rgb(179, 177, 177);
        }

        .select-times {
          flex: 7;
          display: flex;
          // justify-content: center;
          align-items: center;
        }
      }
    }
  }
}</style>
