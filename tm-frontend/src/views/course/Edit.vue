<script setup lang="ts">
import {SaveCourseAPI, getCourseAPI} from '../../request/course/api'
import {Chapter} from '../../request/course/type'
import { onMounted, ref, reactive } from 'vue'
import { ElTable } from 'element-plus'
import { useRoute } from 'vue-router'
import router from '../../router'

document.title = "编辑课程"
const route = useRoute()
const courseid = Number(route.params.id)
const addChapterFormVisible = ref(false)
const editChapterFormVisible = ref(false)
const diaglogwidth = '400px'
const formLabelWidth = '100px'
let tableData:Chapter[] = reactive([])
const course = reactive({
  id:0,
  title: '',
  director_name: '',
  director_id: 0,
  desc: '',
})

const getCourse = async () => {
  let res = await getCourseAPI({course_id:courseid})
  console.log(res)
  course.id = res.course.id
  course.title=res.course.title
  course.director_id=res.course.director_id
  course.director_name=res.course.director_name
  course.desc=res.course.desc
  tableData.push(...res.chapters)
  document.title = course.title
}

onMounted(getCourse)

const currentRow = ref()
const singleTableRef = ref<InstanceType<typeof ElTable>>()
// interface Chapter {
//     id:number
//     title: string
//     author_id: number
//     author_name: string
//     period: string
//     url:string
// }

let newChapter = reactive<Chapter>({
  id: 0,
  title: '',
  author_id: 0,
  author_name: '',
  period: '',
  url:''
})

const handleCurrentChange = (val: Chapter | undefined) => {
  currentRow.value = val
}
// const temp:Chapter[] = [
//   {
//     id:1,
//     title: 'Chapter1',
//     author_id: 1,
//     author_name: '黎伟',
//     period: '7天',
//     url:'https:zishu.co'
//   },
//   {
//     id:2,
//     title: 'Chapter2',
//     author_id: 1,
//     author_name: '黎伟',
//     period: '7天',
//     url:'https:zishu.co'
//   }
// ]
// tableData.push(...temp)

const add_chapter = async() => {
  addChapterFormVisible.value = true
}



const add_confirm = async() => {
  let temp = {
    id: newChapter.id,
    title: newChapter.title,
    author_id: newChapter.author_id,
    author_name: newChapter.author_name,
    period: newChapter.period,
    url:newChapter.url
  }
  if (!currentRow.value) {
    tableData.push(temp)
  } else {
    let rowindex = tableData.indexOf(currentRow.value);
    tableData.splice(rowindex + 1, 0, temp)
  }
  addChapterFormVisible.value = false
  newChapter.title = ''
  newChapter.author_name = ''
  newChapter.period = ''
  newChapter.url = ''

}

// 请写出 edit_chapter 函数的处理逻辑
const edit_chapter = async() => {
  editChapterFormVisible.value = true
}

const edit_confirm = async() => {
  editChapterFormVisible.value = false

}

// 请写出 move_up 的函数
const move_up = async() => {
  if (!currentRow) return
  let rowindex = tableData.indexOf(currentRow.value)
  if (rowindex === 0) return
  let temp = tableData[rowindex]
  tableData.splice(rowindex, 1)
  tableData.splice(rowindex - 1, 0, temp)
}

// 请写出 move_down 的函数
const move_down = async() => {
  if (!currentRow) return
  let rowindex = tableData.indexOf(currentRow.value)
  if (rowindex === tableData.length - 1) return
  let temp = tableData[rowindex]
  tableData.splice(rowindex, 1)
  tableData.splice(rowindex + 1, 0, temp)
}


// 请写出 save_course 的函数
const save_course = async() => {
  // 保存tableData的数据
  let data={
    "courseid":course.id,
    "title": course.title, 
    "desc":course.desc, 
    "chapters":JSON.stringify(tableData)
  }
  let res = await SaveCourseAPI(data)
  console.log(res)
  if (res.code == '200'){
    router.push('/course/'+courseid)
  }
}

</script>

<template>
  <el-input v-model="course.title" placeholder="请输入课程名称" />
  <br><br>
  <el-input
    v-model="course.desc"
    maxlength="100"
    placeholder="请输入课程简介"
    show-word-limit
    type="textarea"
  />
  <br><br>
  <div>
    <el-button v-if="course.title.length>1 && course.desc.length>1" type="primary" @click="add_chapter">新增</el-button>
    <el-button v-if="currentRow" type="primary" @click="edit_chapter">编辑</el-button>
    <el-button v-if="currentRow" type="primary" @click="move_up">上移</el-button>
    <el-button v-if="currentRow" type="primary" @click="move_down">下移</el-button>
  </div>
  <el-table 
  ref="singleTableRef"
  :data="tableData" 
  stripe 
  highlight-current-row
  style="width: 100%"
  @current-change="handleCurrentChange"
  v-if="tableData.length>0"
  >
    <el-table-column label="章节序号" type="index" width="100" />
    <el-table-column label="章节名称">
      <template #default="scope">
        <el-link type="primary" :href="scope.row.url" target="_blank">{{ scope.row.title }}</el-link>
      </template>
    </el-table-column>
    <el-table-column prop="author_name" label="作者" />
    <el-table-column prop="period" label="学习周期" />
  </el-table>
  <el-button v-if="tableData.length>0" type="primary" @click="save_course">保存课程</el-button>


  <el-dialog v-model="addChapterFormVisible" :width="diaglogwidth" :center="true" title="新增章节">
    <el-form :model="newChapter" ref="ruleFormRef" status-icon>
      <el-form-item label="章节名称" prop="title" :label-width="formLabelWidth">
        <el-input v-model="newChapter.title" autocomplete="off" />
      </el-form-item>
      <el-form-item label="作者" prop="author_name" :label-width="formLabelWidth">
        <el-input v-model="newChapter.author_name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="学习周期" prop="period" :label-width="formLabelWidth">
        <el-input v-model="newChapter.period" autocomplete="off" />
      </el-form-item>
      <el-form-item label="学习地址" prop="url" :label-width="formLabelWidth">
        <el-input v-model="newChapter.url" autocomplete="off" />
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="add_confirm()">确定</el-button>
      </span>
    </template>
  </el-dialog>

  <el-dialog v-model="editChapterFormVisible" :width="diaglogwidth" :center="true" title="编辑章节">
    <el-form :model="currentRow" ref="ruleFormRef" status-icon>
      <el-form-item label="章节名称" prop="title" :label-width="formLabelWidth">
        <el-input v-model="currentRow.title" autocomplete="off" />
      </el-form-item>
      <el-form-item label="作者" prop="author_name" :label-width="formLabelWidth">
        <el-input v-model="currentRow.author_name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="学习周期" prop="period" :label-width="formLabelWidth">
        <el-input v-model="currentRow.period" autocomplete="off" />
      </el-form-item>
      <el-form-item label="学习地址" prop="url" :label-width="formLabelWidth">
        <el-input v-model="currentRow.url" autocomplete="off" />
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="edit_confirm()">确定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<style lang="scss" scoped>
</style>