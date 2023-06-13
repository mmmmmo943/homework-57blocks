<template>
    <div class="table-container" ref="table_container">
        <div class="day-cell" v-for="(item, index) in table_data" :key="index">
            <div class="title">{{ item.day }}</div>
            <div class="inner-box">
                <div style="width: 100% ;height:100%;" v-for="(ele, index_cell) in item.hour_sections" :key="index_cell"
                    v-if="item.hour_sections" :ref="el => (divRefs[index][index_cell] = el)">
                    <el-popover placement="top-start"  trigger="hover">
                        <div style="background-color: white;">{{item.day + ','+item.hour_sections[index_cell].time }}</div>
                        <template #reference>
                            <div class="inner-cell" 
                                :class="{ 'selected': ele.selected, 'unselected': ele.selected == false }">
                            </div>
                        </template>
                    </el-popover>
                </div>
            </div>
        </div>
        <div ref="frame" style="border: 1px solid black;background: rgba(0, 119, 255, 0.2);position: absolute;z-index: 99;">
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted, defineEmits } from 'vue';
import dayjs from 'dayjs'


let coordinates = reactive([])//表格内方形模块元素的收集数组
//按照星期进行数据预设
const table_data = reactive([
    {
        day: '星期一',
        key: 1,
    }
    ,
    {
        day: '星期二',
        key: 2,
    }
    ,
    {
        day: '星期三',
        key: 3,
    }
    ,
    {
        day: '星期四',
        key: 4,
    }
    ,
    {
        day: '星期五',
        key: 5,
    }
    ,
    {
        day: '星期六',
        key: 6,
    }
    ,
    {
        day: '星期日',
        key: 7,
    }
])

const num_time_convert2 = (num) => {
    //   使用dayjs转化
    let baseTime = dayjs().startOf('day')
    let startTime = baseTime.add(num * 30, 'minute')
    let endTime = startTime.add(30, 'minute')
    return (`${startTime.format('HH:mm')}-${endTime.format('HH:mm')}`)
}
//给列表单位添加时间段属性
table_data.forEach((item) => {
    item.hour_sections = []
    for (let i = 0; i < 48; i++) {
        item.hour_sections.push({
            num: i,//时间段的对应key值
            included: false,//网格是否已包含
            selected: false,//网格是否已选择
            time:num_time_convert2(i)
        })
    }
})
console.log('table_data',table_data)
const divRefs = ref(table_data.map(() => Array(48).fill(null)))//网格方块的单个引用
// 悬停方法
const hover_label = ref('')

// //网格悬停显示方法
// const hover = (ele, item) => {
//     num_time_convert(ele.num)
// }
// //将时间num转化为时间段
// const num_time_convert = (num) => {
//     //   使用dayjs转化
//     let baseTime = dayjs().startOf('day')
//     let startTime = baseTime.add(num * 30, 'minute')
//     let endTime = startTime.add(30, 'minute')
//     hover_label.value = `${startTime.format('HH:mm')}-${endTime.format('HH:mm')}`
// }

//拖拽框创建与定位
const table_container = ref(null)//网格table的引用
const startPoint = ref()//拖框的起始点
const endPoint = ref()//拖框的结束点
//框相对坐标换算
let frame_location
const frame_location_compute = (clientX, clientY) => {
    const rect = table_container.value.getBoundingClientRect()
    const { left, top } = rect
    let x = clientX - left - 88
    let y = clientY - top
    if (x < 0) {
        x = 0
    }
    if (y < 0) {
        y = 0
    }
    return { x, y }
}
//点击鼠标
const downTrigger = ref(false)
const mouseDown = () => {
    document.addEventListener('mousedown', e => {

        downTrigger.value = true
        const { clientX, clientY } = e
        startPoint.value = frame_location_compute(clientX, clientY)
        endPoint.value = startPoint.value
        console.log('start', startPoint.value)
    })
}
//点击时拖拽鼠标
const mouseMove = () => {
    document.addEventListener('mousemove', e => {
        if (downTrigger.value == false) return

        frame.value.style.display = 'block'
        const { clientX, clientY } = e
        endPoint.value = frame_location_compute(clientX, clientY)
        area_creation(startPoint.value, endPoint.value)
        console.log('end', endPoint.value)
    })
}
//创建拖框区域
const area_creation = (startPoint, endPoint) => {
    console.log('frame', frame.value)
    frame.value.style.left = (Math.min(startPoint.x, endPoint.x) + 88) + 'px'
    frame.value.style.top = Math.min(startPoint.y, endPoint.y) + 'px'
    frame.value.style.width = Math.abs(startPoint.x - endPoint.x) + 'px'
    frame.value.style.height = Math.abs(startPoint.y - endPoint.y) + 'px'
}
//松开鼠标
const mouseUp = () => {
    document.addEventListener('mouseup', e => {
        downTrigger.value = false
        selected_area()
        frame.value.style.display = 'none'
        processing_selected()
    })
}
let finished_data = reactive([
    { day: '星期一' }, { day: '星期二' }, { day: '星期三' }, { day: '星期四' }, { day: '星期五' }, { day: '星期六' }, { day: '星期日' }
])
// 辅助函数，将数字转化为时间
const numToTime = (num) => {
    return dayjs().startOf('day').add(num * 30, 'minute').format('HH:mm');
}
// 辅助函数，将数组中的连续数字分组
const groupContinuousNumbers = (nums) => {
    if (nums.length === 0) return [];
    nums.sort((a, b) => a - b);

    let result = [[nums[0]]];

    for (let i = 1; i < nums.length; i++) {
        if (nums[i] - nums[i - 1] === 1) {
            result[result.length - 1].push(nums[i]);
        } else {
            result.push([nums[i]]);
        }
    }

    return result.map(r => r.length === 1 ? r[0] : r);
}
//处理已选中的数据
const processing_selected = () => {
    finished_data.forEach((item) => {
        let num_box = []
        selected_items.forEach((ele) => {
            if (item.day == ele.day) {
                num_box.push(ele.hourSection.num)
            }
        })
        num_box = groupContinuousNumbers(num_box)
        item.list = num_box.map(r => {
            if (Array.isArray(r)) {
                let start = numToTime(r[0]);
                let end = numToTime(r[r.length - 1] + 1);  // 结束时间应为最后一个数字对应的时间段的后一段
                return `${start}-${end}`;
            } else {
                let start = numToTime(r);
                let end = numToTime(r + 1);  // 结束时间应为该数字对应的时间段的后一段
                return `${start}-${end}`;
            }
        })
    })
    console.log('finished_data', finished_data)
    output_data = finished_data.filter(item => item.list.length != 0);
    update()
}
//对子组件抛给父组件的数据进行更新
let output_data = []
const emit = defineEmits(['updateData'])
const update = () => {
    emit('updateData', output_data)
}

// 周内黄金时段选取
const weekday_output = () => {
    
    coordinates.forEach((item) => {
        if ((item.day == '星期一' || item.day == '星期二' || item.day == '星期三' || item.day == '星期四' || item.day == '星期五')&&(item.hourSection.num>17&&item.hourSection.num<42)&&item.hourSection.selected==false) {
            
     
                item.hourSection.selected = true
            item.hourSection.included = true
            selected_items.push(item)
            
            
        }
    })
    processing_selected()
}

//周末黄金时段选取
const weekdend_output = () => {
    coordinates.forEach((item) => {
        if ((item.day == '星期六' || item.day == '星期日')&&(item.hourSection.num>17&&item.hourSection.num<42)&&item.hourSection.selected==false) {
            item.hourSection.selected = true
            item.hourSection.included = true
            selected_items.push(item)
        }
    })
    processing_selected()
}

//清空方法
const clear_output = () => {
    coordinates.forEach((item) => {
        item.hourSection.selected = false
        item.hourSection.included = false
    })
    selected_items.splice(0)
}
const frame = ref(null)

// 确定选中区域
let selected_items = reactive([])
const selected_area = () => {
    let start = { x: Math.min(startPoint.value.x, endPoint.value.x), y: Math.min(startPoint.value.y, endPoint.value.y) }//拖框左上角点
    let end = { x: Math.max(startPoint.value.x, endPoint.value.x), y: Math.max(startPoint.value.y, endPoint.value.y) }//拖框右上角点
    coordinates.forEach((item) => {
        let itemStart = item.points[0];  // 网格元素的左上角点
        let itemEnd = item.points[3];  // 网格元素的右下角点
        //判断拖框左上角点和右下角点与网格元素的包含关系，寻找左上角点和右下角点所在的网格并选中
        if (((itemStart.x < start.x && itemEnd.x > start.x && itemStart.y < start.y && itemEnd.y > start.y) ||
            (itemStart.x < end.x && itemEnd.x > end.x && itemStart.y < end.y && itemEnd.y > end.y)) &&
            (((itemStart.x < start.x && itemEnd.x > start.x) && (itemStart.x < end.x && itemEnd.x > end.x)) ||
                ((itemStart.y < start.y && itemEnd.y > start.y) && (itemStart.y < end.y && itemEnd.y > end.y)))) {
            if (item.hourSection.selected == true) {
                item.hourSection.selected = false
                item.hourSection.included = false

                console.log('ele11111', item)
            } else {
                item.hourSection.selected = true
                if (item.hourSection.included !== true) {
                    selected_items.push(item);
                }
                console.log('ele22222', item)
            }
        }
        //将整个时间表格元素的四个角进行坐标化，凡是有一个角包含在拖框内，则选中
        item.points.some((e) => {
            if (((start.x < e.x && e.x < end.x) && (start.y < e.y && e.y < end.y))
            ) {
                item.hourSection.selected = true;
                console.log('!!!')
                if (item.hourSection.included !== true) {
                    selected_items.push(item);
                }
                return true
            }
            else {
                return false
            }
        })
        //判断没有角点被框选时的情况
        if ((itemStart.x < start.x && itemEnd.x > end.x && itemStart.y > start.y && itemEnd.y < end.y) || (itemStart.y < start.y && itemEnd.y > end.y && itemStart.x > start.x && itemEnd.x < end.x)) {
            item.hourSection.selected = true;
            if (item.hourSection.included !== true) {
                selected_items.push(item);
            }
        }
    });
    console.log('selected_items', selected_items)
    //收集选中的元素，剔除取消的元素
    selected_items.forEach((ele, index) => {
        if (ele.hourSection.selected == false) {
            selected_items.splice(index, 1)
        } else {
            ele.hourSection.included = true
        }

    })
}

onMounted(() => {
    frame_location = table_container.value.getBoundingClientRect()
    const { left, top } = frame_location
    //动态地拿到网格元素的引用，并给每个网格赋予属性
    coordinates = divRefs.value.flatMap((divRow, i) =>
        divRow.map((div, j) => {
            const rect = div.getBoundingClientRect();
            const currentSection = table_data[i].hour_sections[j];
            return {
                day: table_data[i].day,
                hourSection: currentSection,
                points: [
                    { x: Math.round(rect.left - left - 88), y: Math.round(rect.top - top) },
                    { x: Math.round(rect.right - left - 88), y: Math.round(rect.top - top) },
                    { x: Math.round(rect.left - left - 88), y: Math.round(rect.bottom - top) },
                    { x: Math.round(rect.right - left - 88), y: Math.round(rect.bottom - top) },
                ],
                element: div,
                
            };
        })
    );
    console.log(coordinates);
    //注册事件
    mouseDown()
    mouseMove()
    mouseUp()
})
//将子组件的方法暴露给父组件以方便通过父组件直接控制子组件的数据
defineExpose({ clear_output,weekdend_output,weekday_output })


</script>

<style scoped lang="scss">
.table-container {
    width: 100%;
    height: 100%;
    background-color: white;
    display: flex;
    flex-direction: column;
    position: relative;

    .day-cell {
        width: 100%;
        height: 40px;
        border-bottom: 1px solid darkgray;
        display: flex;

        .title {
            flex: 1;
            border-right: 0.5px solid darkgray;
            color: darkgrey;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .inner-box {
            height: 100%;
            flex: 7;
            display: flex;

            .inner-cell {
                flex: 1;
                height: 100%;
                border-right: 0.5px solid darkgray;
            }

            .selected {
                background-color: rgb(100, 149, 237);
            }

            .unselected:hover {
                background-color: gainsboro;
            }


        }

    }
}
</style>