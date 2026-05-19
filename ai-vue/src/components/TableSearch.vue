<template>
    <el-form ref="ruleFormRef" :model="formData">
        <el-row :gutter="24">
            <template v-for="item in formItemAttrs" :key="item.prop">
                <el-col v-bind="item.col">
                    <el-form-item :label="item.label" :prop="item.prop">
                        <component :is="isComp(item.comp)" :placeholder="item.placeholder" v-model="formData[item.prop]">
                            <template v-if="item.comp === 'select'">
                                <el-option label="全部" value=""></el-option>
                                <el-option 
                                    v-for="opt in item.options"
                                    :key="opt.value"
                                    :label="opt.label" :value="opt.value">
                                </el-option>    
                            </template>
                        </component>
                    </el-form-item>
                </el-col> 
            </template>
        </el-row>
        <el-row>
            <el-button type="primary" @click="handleSearch">查询</el-button>
            <el-button @click="handleReset(ruleFormRef)">重置</el-button>
        </el-row>
        
        
    </el-form>
</template>

<script setup>
import { reactive,ref,computed } from 'vue'

const emit = defineEmits(['search'])

const props = defineProps({
    formItem:{
        type:Array,
        default:()=> []
    }
})

const formItemAttrs = computed(() => {
    return props.formItem.map(item => ({
        ...item,
        col: { xs: 24, sm: 12, md: 8, lg: 6, xl: 6 }
    }))
})

// 表单数据
const ruleFormRef = ref()
const formData = reactive({})
const isComp = (comp) => {
    return {
        input:'elInput',
        select:'elSelect'
    }[comp]
}

const handleSearch = () => {
    // console.log(formData)
    emit('search',formData)
}

const handleReset = (formEL) => {
    // 先重置表单，再触发查询(此时触发的查询其实就是查询全部，即把所有列表都显示出来，不然点了重置后就没有数据了)
    if(!formEL) return
    formEL.resetFields()
    handleSearch()
}
</script>