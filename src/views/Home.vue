<template>

  <div class="home">
    <a-steps type="navigation" :style="stepStyle">
      <a-step v-for="(item, index) in stepList" :key = "index" :status="item.status" :title="item.title" :description="item.description"/>
    </a-steps>
  </div>

  <div class="components-create" v-if="stepStatus == 0">
    <a-input class="components-input" maxLength = 10 size="large" placeholder="请输入互动题名称" allow-clear @change="onChange" />
    <a-button type="primary" :size="size" @click="nextStep(1)">
      创建
    </a-button>
  </div>

  <div class="components-update" v-if="stepStatus == 1" data-sourse="game">
    <div style="color: red;font-size: 20px;margin-top: 10px;">请进入以下地址编辑互动题内容并保存。</div>
    <a class="target" target="_blank" href="https://webeditor.aixuexi.com/#/editor?parentId=0&id=1538&name=thy%E6%B5%8B%E8%AF%95">
      https://webeditor.aixuexi.com/#/editor?parentId=0&id=1538&name=thy%E6%B5%8B%E8%AF%95</a> 
      <router-link :to="'/doc?ebookId=' + item.id">
        {{ game }}
      </router-link>
    </div>

  <div class="components-publish" v-if="stepStatus == 2">
    <a-select :size="large"  style="width: 150px;margin-right: 20px;" placeholder="请选择学段" @change="handleChange" >
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

    <a-select :size="large"  style="width: 150px;margin-right: 50px;" placeholder="请选择学科" @change="handleChange" >
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

    <a-button type="primary" :size="size">
      发布
    </a-button>
  </div>

  <div id='components-next' style ='opacity: 0'>
    <a-button-group>
      <a-button type="primary" > 下一步<a-icon type="right" /> </a-button>
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
        marginBottom: '60px',
        boxShadow: '0px -1px 0 0 #e8e8e8 inset',
      },
      stepList: [
        {status: 'process', title: 'Step 1',description: '创建互动题'},
        {status: 'wait', title: 'Step 2',description: '编辑互动题'},
        {status: 'wait', title: 'Step 3',description: '发布'},
      ]
    });

    onMounted(() => {
      tipsHandler();
    });

    const created = (): void => {
      let iname = {"name" :"t33336553"};
      let categorys;
      axios.post(process.env.VUE_APP_SERVER + "/game/create",iname).then((res: any) => {
        let data = res.data;
        if (data.success) {
          categorys = data.content;
          console.log("原始数据：", categorys);
        }
      })
    };

    const tipsHandler = (): void => {
      let stepList: Array<any> = data.stepList;
      let stepStatus: number = data.stepStatus;
      let index: number;
      for( index = 0; index < stepList.length; index ++) {
        let item = stepList[index] as any;
        if(index < stepStatus) {
          item.status = 'finish';
        } else if (stepStatus == index) {
          item.status = 'process';
        } else if(index > stepStatus) {
          item.status = 'wait';
        }
      }
    };

    const nextStep = (ids: any) => {
      created();
      console.log(ids, '--ids--');
      data.stepStatus = ids;
      tipsHandler();
    }
    
    return {
      ...toRefs(data),
      nextStep
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
   margin-top: 200px;
  }
</style>
