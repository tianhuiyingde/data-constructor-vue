<template>

  <div class="home">
    <a-steps v-model="current" type="navigation" :style="stepStyle" >
      <a-step v-for="(item, index) in stepList" :key = "index" :status="item.status" :title="item.title" :description="item.description"/>
    </a-steps>
  </div>

  <div class="components-create" v-if="stepStatus == 0">
    <a-input  class="components-input" maxLength = 10 size="large" placeholder="请输入互动题名称" allow-clear @change="onChange" />
    <a-button type="primary" :size="size" @click="nextStep1">
      创建
    </a-button>
  </div>
  <div v-if="stepStatus == 4">
    <a-spin size="large" />
  </div>
  <div class="components-update" v-if="stepStatus == 1 || stepStatus == 5" :data-sourse="data">
    <div style="color: red;font-size: 20px;margin-top: 10px;">创建成功！请进入以下地址编辑互动题内容并保存。</div>
    <a class="target" target="_blank" :href=editorUrl @click="nextStep4">
      互动题名称：{{inputVal}}</a> 

    </div>

  <div class="components-publish" v-if="stepStatus == 2">
    <span>学段：</span>
    <a-select default-value="1" :size="large"  style="width: 150px;margin-right: 20px;" placeholder="请选择学段" @change="handleChangeSubject" >
      <a-select-option value="1">
        小学
      </a-select-option>
      <a-select-option value="2">
        初中
      </a-select-option>
      <a-select-option value="3" >
        高中
      </a-select-option>
    </a-select>
    <span>学科：</span>
    <a-select default-value="1"  :size="large"  style="width: 150px;margin-right: 50px;" placeholder="请选择学科" @change="handleChangeSchoolSection" >
      <a-select-option value="1">
        语文
      </a-select-option>
      <a-select-option value="2">
        英语
      </a-select-option>
      <a-select-option value="3" >
        数学
      </a-select-option>
    </a-select>

    <a-button type="primary" :size="size" @click="nextStep3">
      发布
    </a-button>
  </div>

  <div class="publish-success" v-if="stepStatus == 3" :data-sourse="data">
    <div style="color: red;font-size: 20px;margin-top: 10px;">互动题发布成功!去查看</div>
    <a class="target" target="_blank" :href=gameUrl >
      互动题名称：{{inputVal}}</a> 

    </div>

  <div id='components-next' v-if="stepStatus == 5">
    <a-button-group>
      <div style="margin-right:10px ">互动题编辑完成，请点击</div>
      <a-button type="primary" @click="nextStep2"> 下一步<a-icon type="right" /> </a-button>
    </a-button-group>
  </div>

</template>

<script lang="ts">
import { defineComponent, onMounted, reactive, toRefs } from 'vue';
import axios from "axios";
// vue3.0: https://www.jianshu.com/p/fb54fcc6a229

export default defineComponent({
  
  name: 'Home',
  
  components: { },

  setup() {

    let data = reactive({
      stepStatus: 0,
      stepStyle: {
        marginBottom: '120px',
        boxShadow: '0px -1px 0 0 #e8e8e8 inset',
      },
      stepList: [
        {status: 'process', title: 'Step 1',description: '创建互动题'},
        {status: 'wait', title: 'Step 2',description: '编辑互动题'},
        {status: 'wait', title: 'Step 3',description: '发布'},
      ],
      editorUrl:'',
      gameUrl:'',
      inputVal:'默认新互动题',
      subjectVal:1,
      schoolsectionVal:1
    });

    // onMounted(() => {

    // });

    const onChange = (e: any) =>{
      data.inputVal=e.target.value;
    }

    const handleChangeSubject = (value: any) =>{
      data.subjectVal=value;
      console.log(`selected ${value}`);
    }
    const handleChangeSchoolSection = (value: any) =>{
      data.schoolsectionVal=value;
      console.log(`selected ${value}`);
    }

    const created = (): void => {
      let iname = {"name" :data.inputVal};
      axios.post(process.env.VUE_APP_SERVER + "/game/create",iname).then((res: any) => {
        let game = res.data;
        console.log("data",data);
        if (game.success) {
          let stepList: Array<any> = data.stepList;
          data.editorUrl = game.data.editorUrl;
          console.log("editorUrl",data.editorUrl);
          data.stepStatus = 1;
          stepList[0].status = 'finish';
          stepList[1].status = 'process';
        }
      })
    };

    
    const published = (): void => {
      let req = {"subject" :data.subjectVal,"schoolsection" :data.schoolsectionVal};
      axios.post(process.env.VUE_APP_SERVER + "/game/publish",req).then((res: any) => {
        let game1 = res.data;
        console.log("data",data);
        if (game1.success) {
          data.gameUrl = game1.data.gameUrl;
          let stepList: Array<any> = data.stepList;
          stepList[2].status = 'finish';
          data.stepStatus = 3;
        }
      })
    };

    const nextStep1 = ():void =>{
      data.stepStatus = 4;
      created();
      
    }
    
    const nextStep2 = ():void =>{
      let stepList: Array<any> = data.stepList;
      stepList[1].status = 'finish';
      stepList[2].status = 'process';
      data.stepStatus = 2;
    }

    const nextStep3 = ():void =>{
      data.stepStatus = 4;
      published();
      
    }
    const nextStep4 = ():void =>{
      data.stepStatus = 5;      
    }

    return {
      ...toRefs(data),
      nextStep1,
      nextStep2,
      nextStep3,
      nextStep4,
      onChange,
      handleChangeSubject,
      handleChangeSchoolSection
    }

  },

 

});
</script>

<style scoped>


  .target {
    font-size: 20px;
    text-decoration:underline;
  }

  .ant-input-affix-wrapper {
    width:15%
  }

  .components-create .ant-input {
    width: 200px;
    margin: 8px 8px 8px 0;
 
  }

  .components-update {
    margin-top: 20px;
    margin: 8px 8px 8px 0;
 
  }

  .components-input  {
    margin-right: 50px ;
 
  }

  .ant-descriptions-item-content {
    color: #f5222d !important;
  }

  #components-next .ant-btn-group {
   margin-right: 8px;
   margin-top: 100px;
  }
</style>
