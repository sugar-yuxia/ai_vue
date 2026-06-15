<template> 
    <el-dialog
        :title="isEdit ? '编辑文章' : '文章详情'"
        v-model="dialogVisible"
        width="50%"
        @close="handleClose"
    >
        <el-form :model="formData" :rules="rules" ref="formref" label-width="120px">
            <el-form-item label="文章标题" prop="title">
                <el-input v-model="formData.title" placeholder="请输入文章标题" maxlength="50" show-word-limit clearable></el-input>
            </el-form-item>
            <el-form-item label="所属分类" prop="categoryId">
                <el-select v-model="formData.categoryId" placeholder="请选择分类" clearable>
                    <el-option v-for="item in props.categories" :key="item.value" :label="item.label" :value="item.value"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="文章摘要" prop="summary">
                <el-input type="textarea" v-model="formData.summary" placeholder="请输入文章摘要（可选）" maxlength="1000" show-word-limit clearable :rows="4"></el-input>
            </el-form-item>
            <el-form-item label="标签" prop="tags">
                <el-select v-model="formData.tagsArray" placeholder="请输入文章标签（可选）" multiple filterable allow-create style="width: 100%;">
                    <el-option v-for="tag in commonTags" :key="tag" :label="tag" :value="tag"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="封面图片">
                <div class="cover-uploader">
                    <el-upload
                        class="upload-demo"
                        action="#"
                        :before-upload="beforeUpload"
                        :http-request="handleUploadRequest"
                        :show-file-list="false"
                        accept="image/#"
                    >
                        <div v-if="!imgUrl" class="cover-placeholder">
                            <p>点击上传封面</p>
                        </div>
                        <img v-else :src="imgUrl" class="cover-image" alt="封面图片">
                        </el-upload>
                        <div v-if="imgUrl" class="cover-remove">
                            <el-button type="danger" size="mini" @click="handleRemove">删除封面</el-button>
                        </div>
                </div>
            </el-form-item>
            <el-form-item label="文章内容" prop="content">
                <RichTextEditor 
                    v-model="formData.content" 
                    placeholder="请输入文章内容" 
                    :maxCharCount="5000"
                    @change="handleContentChange"
                    @created="handleEditorCreated"
                    min-height="400px"
    > 
                </RichTextEditor>
            </el-form-item>
        </el-form>
        <div v-if="btnpreview">
            <h3>内容预览</h3>
            <div v-html = "formData.content">
            </div>
        </div>
        <template #footer>
            <el-button  @click="btnpreview = !btnpreview">{{ btnpreview ? '隐藏预览' : '预览效果' }}</el-button>
            <el-button  @click="handleClose">取消</el-button>
            <el-button  type="primary" @click="handleSubmit" :loading="loading">{{ isEdit ? '更新文章' : '创建文章' }}</el-button>
        </template>
    </el-dialog>
</template>

<script setup>
import { ref, reactive, computed,nextTick, watch } from 'vue'
import { ElMessage } from 'element-plus'
import { uploadFile,createArticle } from '@/api/admin'
import { fileBaseUrl } from '@/config'
import RichTextEditor from '@/components/RichTextEditor.vue'

const props = defineProps({
    modelValue: {
        type: Boolean,
        default: false
    },
    categories: {
        type: Array,
        default: () => []
    },
    article: {
        type: Object,
        default: null
    }
})

const emit = defineEmits(['update:modelValue','success'])

const dialogVisible = computed({
    get() {
        return props.modelValue
    },
    set(val) {
        emit('update:modelValue',val)
    }
})

const isEdit = computed(() => !!props.article?.id)

// 监听编辑数据
watch(() => props.article,(newVal) => {
    if(newVal){
        nextTick(() => { 
            Object.assign(formData,newVal)
            //使用现有ID 
            businessId.value = newVal.id
            // 封面Url
            imgUrl.value = fileBaseUrl + newVal.coverImage
        })
    }
})

const handleClose = () => {
    // 重置表单
    formref.value.resetFields()
    // 重置ID
    businessId.value = null
    // 重置标签数组
    formData.tagsArray = []
    // 重置封面图片和数据
    handleRemove()
    emit('update:modelValue',false)
}

// 表单数据
const formData = reactive({
    "title": "",
    "content": "",
    "coverImage": "",
    "categoryId": 1,
    "summary": "",
    "tags": "",
    "id": ""
})


const rules = reactive({
    title: [
        { required: true, message: '请输入文章标题', trigger: 'blur' },
        { max: 50, message: '文章标题最多50个字符', trigger: 'blur' }
    ], 
    categoryId: [
        { required: true, message: '请选择分类', trigger: 'change' }
    ],
    content: [
        { required: true, message: '请输入文章内容', trigger: 'blur' },
        { max: 5000, message: '文章内容最多5000个字符', trigger: 'blur' }
    ]
})

const commonTags = [
  '情绪管理', '焦虑', '抑郁', '压力', '睡眠', 
  '冥想', '正念', '放松', '心理健康', '自我成长',
  '人际关系', '工作压力', '学习方法', '生活技巧'
]

//上传
const imgUrl = ref('')
const beforeUpload = (file) => { 
    //针对上传的文件进行校验
    console.log(file)
    const isImage = file.type.startsWith('image/')
    const isLt5M = file.size / 1024 / 1024 < 5
    if(!isImage) {
        ElMessage.error('请上传图片文件')
        return false
    }
    if(!isLt5M) {
        ElMessage.error('上传文件大小不能超过5M')
        return false
    }

    return true
}

const businessId = ref(null)
const handleUploadRequest = async({ file }) => { 
    // UUID生成
    businessId.value = crypto.randomUUID()

    const fileRes = await uploadFile(file,{
        businessId: businessId.value,
    })
    console.log(fileRes)

    // 拼接完整图片地址
    imgUrl.value = fileBaseUrl + fileRes.filePath
    formData.coverImage = fileRes.filePath
}

const handleRemove = () => { 
    imgUrl.value = ''
    formData.coverImage = ''
}

// 富文本
const handleContentChange = (data) => {
    console.log(data,'富文本内容')
    formData.content = data.html
}

const editorInstance = ref(null)
const handleEditorCreated = (editor) => {
    editorInstance.value = editor
    // 编辑
    if(formData.content && editor){
        nextTick(() => {
            editor.setHtml(formData.content)
        })  
    }
}

const btnpreview = ref(false)

// 提交
const formref = ref()
const loading = ref(false)
const handleSubmit = () => {
    formref.value.validate((valid, fields) => {
        if(valid){
            loading.value = true
        }
        console.log(formData,'提交数据')
        const submitData = {
            ...formData,
            tags: formData.tagsArray.join(',')
        }
        delete submitData.tagArray

        createArticle(submitData).then(res => {
            loading.value = false
            emit('success')
        })
    })
}
</script>

<style scoped lang="scss">
.cover-placeholder{
    width: 200px;
    height: 120px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #8b949c;
    background-color: #f6f8fa;
}
.cover-image{
    width: 200px;
    height: 120px;
    display: block;
}
</style>