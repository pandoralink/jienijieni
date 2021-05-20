<template>
<div>
    <div class="NoteHead">
        <button class="pl-btn" v-if="!DeleteFlag" @click="NoteDelete">确认删除</button>
        <button class="pl-btn" @click="NoteAdd">增添</button>
        <button class="pl-btn" @click="DeleteFlag = !DeleteFlag">{{ DeleteFlag?'删除':'取消删除'}}</button>
    </div>
    <!-- <div class="loader" v-if="!isGet">Loading…</div>
    <div style="display: flex; width: 100%;" v-if="isError">
        <span style="font-weight: 1000; font-size: larger; margin: auto;">加载失败，请刷新</span>
    </div> -->
    <div style="display: flex">
        <pl-loder :isget="isGet" :ismsg="isError"></pl-loder>
    </div>
    <div
        class="NoteContain"
        :class="{ 'cancel' : isCancel }"
        v-for="(i,index) of NoteData"
        :key="index"
    >
        <!-- <todo-check-box
            v-model="NoteDeleteList[index]"
            v-show="isCancel"
            @endPost="NoteInsert(i,$event)"
        ></todo-check-box> -->
        <todo-check-box
            :radioState="NoteDeleteList[index]"
            @radioInput="NoteDeleteList[index] = $event"
            v-show="isCancel"
            @endPost="NoteInsert(i,$event)"
        ></todo-check-box>
        <div :class="{'pl-note-empty' : NoteDeleteList[index]}">
            <div class="line"></div>
        </div>
        <!-- <el-checkbox
            class="pl-note-checkbox"
            style="margin-right: 10px"
            v-model="NoteDeleteList[index]"
            v-show="isCancel"
            @click="test"
        ></el-checkbox> -->
        <!-- <el-checkbox
            class="pl-note-checkbox"
            style="margin-right: 10px"
            v-model="i.taskState"
            v-show="isCancel"
        >
        </el-checkbox> -->
        <div
            class="NoteCol"
            @touchstart.stop.prevent="gtouchstart($event)"
            @touchmove="gtouchmove($event)"
            @touchend="gtouchend($event,i)"
        >
            <h1
                class="title"
                @click="DialogShow(i)"
                :class="{ 'cancel' : isCancel }"
            >{{ i.head?i.head:'空标签'}}
            </h1>
            <h2 class="time">{{ i.NoteTime?i.NoteTime:'标签时间' }}</h2>
        </div>
    </div>
    <el-dialog
        title="便签内容"
        :visible.sync="DialogFlag"
        width="50%">
        <textarea type="text" v-model="NoteObject.head" placeholder="标签头"></textarea>
        <!-- <textarea ref="input" placeholder="输入内容" v-model="NoteObject.data"></textarea> -->
        <el-button slot="footer" @click="DialogFlag = !DialogFlag">关闭</el-button>
        <el-button slot="footer" @click="NoteInsert(NoteObject)">确认</el-button>
    </el-dialog>
</div>
</template>

<script>
import axios from 'axios'
import plLoder from '../loder/index.vue'
import todoCheckBox from './note/todoCheckBox.vue'

export default {
    name: 'note',
    data() {
        return {
            DeleteFlag: true,
            DialogFlag: false,
            NoteData: [],
            NoteDeleteList: [],
            NoteObject: {},
            isGet: false,
            isError: false,
            timeOutEvent: 0,
            isCancel: false,
        }
    },
    components: {
        plLoder,
        todoCheckBox
    },
    mounted() {
        this.getData();
    },
    methods: {
        NoteAdd() {
            let notedata = {
                head: '',
                NoteTime: '',
                data: '',
                taskState: false
            };
            this.NoteData.unshift(notedata);
            this.NoteDeleteList.unshift(false);
        },
        NoteInsert(InsertData,state) {
            if(state) InsertData.taskState = state; // state 存在更新值才更新
            InsertData.NoteTime = new Date().toLocaleString();
            axios.post('/note/add', InsertData)
            .then(res => {
                if(res.data == 'update') console.log('update');
                else InsertData.id = parseInt(res.data);
            })
            .catch(function (error) {
                console.log(error);
            });
            InsertData.NoteTime = InsertData.NoteTime.slice(0,-3);
            this.DialogFlag = false;
        },
        DialogShow(NoteItem) {
            this.NoteObject = NoteItem;
            this.DialogFlag = true;
        },
        NoteDelete() {
            let list = this.NoteData.filter((item,index) => {
                if(this.NoteDeleteList[index] == true) return item;
            })
            console.log(list);
            axios.post('/note/delete',list).catch(err => { console.log(err); });
            this.NoteData = this.NoteData.filter((item,index) => {
                if(this.NoteDeleteList[index] == false) return item; 
            })
            this.NoteDeleteList = new Array(this.NoteData.length).fill(false);
            this.DeleteFlag = !this.DeleteFlag;
        },
        getData(){
            axios.get('/note')
            .then((res) => {
                console.log(res.data);
                this.isGet = true;
                this.NoteData = res.data.reverse();
                this.NoteData.forEach((item,index) => {
                    this.NoteData[index].NoteTime = item.NoteTime.slice(0,-3);
                })
                this.NoteDeleteList = new Array(this.NoteData.length).fill(false);
                this.NoteDeleteList.forEach((item,index) => {
                    this.NoteDeleteList[index] = this.NoteData[index].taskState == 1;
                })
            })
            .catch((error) => {
                this.isError = true;
                console.log(error);
            });
        },
        gtouchstart(e) {
            this.timeOutEvent = setTimeout(() => {
                this.timeOutEvent = 0;
                // alert("长按事件触发发");
                this.isCancel = !this.isCancel;
            },500);//这里设置定时器，定义长按500毫秒触发长按事件，时间可以自己改，个人感觉500毫秒非常合适   
            return false;
        },
        gtouchend(e,i){
        //手释放，如果在500毫秒内就释放，则取消长按事件，此时可以执行onclick应该执行的事件
            // console.log(e);
            let timeOutEvent = this.timeOutEvent;   
            clearTimeout(timeOutEvent);//清除定时器
            if(timeOutEvent!=0){   
                //这里写要执行的内容（尤如onclick事件）   
                this.DialogShow(i);
                // alert("你这是点击，不是长按");   
            }
            return false;
        },   
        gtouchmove(e){   
        //如果手指有移动，则取消所有事件，此时说明用户只是要移动而不是长按
            console.log(e);
            let timeOutEvent = this.timeOutEvent;   
            clearTimeout(timeOutEvent);//清除定时器   
            timeOutEvent = 0;       
        }
    }
}
</script>

<style>
.NoteHead {
    display: flex; 
    flex-direction: row-reverse
}
.NoteButton {
    margin-right: 1vh;
    border: 0.5vh solid black; 
    border-radius: 2vh;
}
.NoteContain {
    display: flex; 
    align-items: center;
}
.NoteCol {
    display: flex; 
    justify-content: space-between; 
    align-items: center; 
    width: 100%;
}
</style>
