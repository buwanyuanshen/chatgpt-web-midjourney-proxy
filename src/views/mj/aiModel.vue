<script setup lang="ts">
import {NSelect, NInput,NSlider, NButton, useMessage,NTag,NPopselect ,NAvatar, NText} from "naive-ui"
import type { SelectRenderLabel, SelectRenderTag } from 'naive-ui'
import { ref ,computed,watch, onMounted,h} from "vue";
import {gptConfigStore, homeStore,useChatStore} from '@/store'
import { mlog,chatSetting } from "@/api";
import { t } from "@/locales";

import AiModelServer from "./aiModelServer.vue";
 

const emit = defineEmits(['close']);
const chatStore = useChatStore();
const uuid = chatStore.active;
//mlog('uuid', uuid );
const chatSet = new chatSetting( uuid==null?1002:uuid);

const nGptStore = ref(  chatSet.getGptConfig() );

const config = ref({
model:[
]
,maxToken:4096
}); 
const st= ref({openMore:false,isShow:false ,server:'' });
const voiceList= computed(()=>{
    let rz=[];
    for(let o of "alloy,echo,fable,onyx,nova,shimmer".split(/[ ,]+/ig))rz.push({label:o,value:o}) 
    return rz;
});
const modellist = computed(() => { //
    let rz =[ ];
    for(let o of config.value.model){
        rz.push({label:o,value:o})
    }
    if(gptConfigStore.myData.userModel){
        let arr = gptConfigStore.myData.userModel.split(/[ ,]+/ig);
        //  let uniqueArray  = arr.filter((value, index, self) => {
        //     return self.indexOf(value) === index;
        // });
        for(let o of arr ){
            o && rz.push({label:o,value:o})
        }
    }
    //服务端的 CUSTOM_MODELS 设置
    if( homeStore.myData.session.cmodels ){
        let delModel:string[] = [];
        let addModel:string[]=[];
        let isDelAll= false
        homeStore.myData.session.cmodels.split(/[ ,]+/ig).map( (v:string)=>{
            if(v.indexOf('-')==0){
                delModel.push(v.substring(1))
                if( v=='-all') isDelAll=true;
            }else{
                addModel.push(v);
            }
        });
        mlog('cmodels',delModel,addModel);
        if( isDelAll  )rz=[];
        rz= rz.filter(v=> delModel.indexOf(v.value)==-1 );
        addModel.map(o=>rz.push({label:o,value:o}) )
        if (rz.length==0){
            rz.push({label:'gpt-3.5-turbo',value:'gpt-3.5-turbo'}) 
        }
    }

    let uniqueArray: { label: string, value: string }[] = Array.from(
        new Map(rz.map(item => [JSON.stringify(item), item]))
        .values()
    );
    return uniqueArray ;
});
const ms= useMessage();
// const save = ()=>{ 
//     gptConfigStore.setMyData( nGptStore.value );
//     ms.success( t('common.saveSuccess')); //'保存成功'
//     emit('close');
// }
const saveChat=(type:string)=>{
     chatSet.save(  nGptStore.value );
     gptConfigStore.setMyData( nGptStore.value );
     homeStore.setMyData({act:'saveChat'}); 
     if(type!='hide')ms.success( t('common.saveSuccess'));
     emit('close');
}
 
watch(() => nGptStore.value.model, (n) => {
    nGptStore.value.gpts = undefined;
    let max = 4096;
    let model = n.toLowerCase();
    if (model.includes('4k')) {
        max = 4096;
    } else if (model.includes('8k')) {
        max = 8192;
    } else if (model.includes('16k')) {
        max = 16384;
    } else if (model.includes('32k')) {
        max = 32768;
    } else if (model.includes('vision')) {
        max = 4096;
    } else if (model === 'gpt-4') {
        max = 8192;
    } else if (model.includes('claude-3')) {
        max = 2000000;
    } else if (model.includes('claude-2')) {
        max = 1000000;
		} else if (model.includes('gemini-1.5-pro')) {
        max = 20000000;
		} else if (model.includes('gemini-1.5-flash')) {
        max = 10000000;
		} else if (model.includes('gemini-1.0-pro')) {
        max = 32000;
		} else if (model.includes('gemini-pro')) {
        max = 32000;
    } else if (model.includes('gemini-2.0')) {
        max = 1000000;
    }else {
        max = 4096; // Default value if none of the specific conditions are met
    }

    config.value.maxToken=max;
    if(nGptStore.value.max_tokens> config.value.maxToken ) nGptStore.value.max_tokens= config.value.maxToken;
})

const reSet=()=>{
    gptConfigStore.setInit();
    nGptStore.value= gptConfigStore.myData;
}

onMounted(() => {
    //gptConfigStore.myData= chatSet.getGptConfig();
});

const serverSuccess=(s:any)=>{
    mlog('serverSuccess ', s  )
    nGptStore.value.model= s.model
}
//
//const f= ref({model:gptConfigStore.myData.model});
</script>
<template>
<section class="mb-4 flex justify-between items-center"  >
    <div class=" flex space-x-2 justify-between items-center">
     <div class="flex justify-start items-center">
        <span class="text-red-500">*</span>  
        {{ $t('mjset.model') }}
        
     </div>
     
    </div>
    <div  class="!w-[70%] flex justify-end items-center " >
       <div> 
        <n-select v-model:value="nGptStore.model" :options="modellist" size="small"  filterable   />
       </div>
       <div class=" pl-2" > 
        <!-- <NButton type="primary" @click="saveChat('no')" size="small" >{{ $t('mj.server_load') }}</NButton> -->
        <AiModelServer @success="serverSuccess"/>
        <!-- <n-popselect
                v-model:value="st.server"
                :options="serverOptions"
                :render-label="renderLabel"
                size="medium"
                scrollable
            >
            <NTag  type="primary" round size="small" :bordered="false" class="!cursor-pointer">
            {{ $t('mj.server_load') }}
            
            </NTag>
        </n-popselect> -->
            

       </div>
    </div>
</section>
<section class="mb-4 flex justify-between items-center"  >
    <n-input   :placeholder="$t('mjchat.modlePlaceholder')" v-model:value="gptConfigStore.myData.userModel">
      <template #prefix>
        {{ $t('mjchat.myModle') }}
      </template>
    </n-input>
 </section>
 <section class=" flex justify-between items-center"  >
     <div> {{ $t('mjchat.historyCnt') }}
     </div>
     <div class=" flex justify-end items-center w-[80%] max-w-[240px]">
        <div class=" w-[200px]"><n-slider v-model:value="nGptStore.talkCount" :step="1" :max="100000" /></div>
        <div  class="w-[50px] text-right">{{ nGptStore.talkCount }}</div>
    </div>
</section>
<div class="mb-4 text-[12px] text-gray-300 dark:text-gray-300/20">{{ $t('mjchat.historyToken') }}</div>

 <section class=" flex justify-between items-center"  >
     <div> {{ $t('mjchat.historyTCnt') }} 
     </div>
     <div class=" flex justify-end items-center w-[80%] max-w-[240px]">
        <div class=" w-[200px]"><n-slider v-model:value="nGptStore.max_tokens" :step="1" :max="config.maxToken" :min="1" /></div>
        <div  class="w-[50px] text-right">{{ nGptStore.max_tokens }}</div>
    </div>
</section>
<div class="mb-4 text-[12px] text-gray-300 dark:text-gray-300/20">{{ $t('mjchat.historyTCntInfo') }}  </div>

 <section class="mb-4"  >
    <div>{{ $t('mjchat.role') }}</div>
    <div>
     <n-input  type="textarea"  :placeholder=" $t('mjchat.rolePlaceholder') "   v-model:value="nGptStore.systemMessage" :autosize="{ minRows: 3 }"
    />
    </div>
 </section>

<template v-if="st.openMore">
    <section class=" flex justify-between items-center "  >
        <div>{{ $t('mj.temperature') }}</div>
        <div class=" flex justify-end items-center w-[80%] max-w-[240px]">
            <div class=" w-[200px]"><n-slider v-model:value="nGptStore.temperature" :step="0.01" :max="1" /></div>
            <div  class="w-[40px] text-right">{{ nGptStore.temperature }}</div>
        </div>
    </section>
    <div class="mb-4 text-[12px] text-gray-300 dark:text-gray-300/20"> {{ $t('mj.temperatureInfo') }}</div>


    <section class=" flex justify-between items-center "  >
        <div> {{ $t('mj.top_p') }}</div>
        <div class=" flex justify-end items-center w-[80%] max-w-[240px]">
            <div class=" w-[200px]"><n-slider v-model:value="nGptStore.top_p" :step="0.01" :max="1" /></div>
            <div  class="w-[40px] text-right">{{ nGptStore.top_p }}</div>
        </div>
    </section>
    <div class="mb-4 text-[12px] text-gray-300 dark:text-gray-300/20">{{ $t('mj.top_pInfo') }}</div>

    <section class=" flex justify-between items-center "  >
        <div> {{ $t('mj.presence_penalty') }}</div>
        <div class=" flex justify-end items-center w-[80%] max-w-[240px]">
            <div class=" w-[200px]"><n-slider v-model:value="nGptStore.presence_penalty" :step="0.01" :max="1" /></div>
            <div  class="w-[40px] text-right">{{ nGptStore.presence_penalty }}</div>
        </div>
    </section>
    <div class="mb-4 text-[12px] text-gray-300 dark:text-gray-300/20">{{ $t('mj.presence_penaltyInfo') }} </div>


    <section class=" flex justify-between items-center "  >
        <div>{{ $t('mj.frequency_penalty') }}</div>
        <div class=" flex justify-end items-center w-[80%] max-w-[240px]">
            <div class=" w-[200px]"><n-slider v-model:value="nGptStore.frequency_penalty" :step="0.01" :max="1" /></div>
            <div  class="w-[40px] text-right">{{ nGptStore.frequency_penalty }}</div>
        </div>
    </section>
    <div class="mb-4 text-[12px] text-gray-300 dark:text-gray-300/20">{{ $t('mj.frequency_penaltyInfo') }}</div>

    <section class="mb-4 flex justify-between items-center"  >
        <div >{{ $t('mj.tts_voice') }}</div>
        <n-select v-model:value="nGptStore.tts_voice" :options="voiceList" size="small"  class="!w-[50%]"   />
    </section>


</template>
<div v-else class="text-right cursor-pointer mb-4" @click="st.openMore=true">
    <NTag  type="primary" round size="small" :bordered="false" class="!cursor-pointer">More...</NTag>
</div>

 <section class=" text-right flex justify-end space-x-2"  >
    <NButton   @click="reSet()">{{ $t('mj.setBtBack') }}</NButton>
    <!-- <NButton type="primary" @click="saveChat">{{ $t('mj.setBtSaveChat') }}</NButton>
    <NButton type="primary" @click="save">{{ $t('mj.setBtSaveSys') }}</NButton> -->
    <NButton type="primary" @click="saveChat('no')">{{ $t('common.save') }}</NButton>
 </section>

 <!-- <NModal  v-model:show="st.isShow"  preset="card"  :title="$t('mjchat.modelChange')" class="!max-w-[820px]" @close="st.isShow=false" >
    Model内容
 </NModal> -->
</template>
