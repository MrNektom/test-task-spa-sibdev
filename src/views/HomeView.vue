<script setup>
import logo from "@/assets/logo.png";
import { reactive, ref } from '@vue/reactivity';
import { Search, Grid, List } from "@element-plus/icons-vue"
import VideoCard from "@/components/VideoCard.vue";
import Heart from "@/components/icons/Heart.vue";
import { ElMessageBox } from "element-plus";
const data = reactive({
  query:"",
  count:12,
  order:null,
  nextPageToken:"",
  list:[],
  showMode:"Grid",
  saved:JSON.parse(localStorage.getItem("saved") || "[]"),
  dialogVisible:false,
  dialogIsEdit:false,
  dialogQueryCount:12,
  dialogQuery:"",
  dialogQueryName:"",
  dialogQuerySort:"",
  editingIndex:null
})

const update = async () => {
  if(data.query){
  const list = await execute()
  data.nextPageToken = list.result.nextPageToken
  data.list.length = 0
  list.result.items.forEach(item => {
    data.list.push(item)
    // console.log(item.snippet.title)
  });
  } else {
    data.list.length = 0
  }
}
  // Make sure the client is loaded and sign-in is complete before calling this method.
  function execute(nextPageToken=null) {
    return gapi.client.youtube.search.list({
      "part":[
        "id",
        "snippet"
      ],
      "q": data.query,
      "maxResults":data.count,
      pageToken:nextPageToken,
      order:data.order
    })
  }

  window.addEventListener("scroll",async ()=>{
    if ((window.scrollY + document.documentElement.clientHeight) - document.documentElement.scrollHeight == 0) {
      const add = await execute(data.nextPageToken)
      console.log(add)
      data.nextPageToken = add.result.nextPageToken
      add.result.items.forEach(item => {
        data.list.push(item)
      })
    }
  })

  const addSaved = () => {
    data.dialogQuery = data.query
    data.dialogQueryCount = 12
    data.dialogVisible = true
  }
  const saveQuery = () => {
    data.saved.push({
      query:data.dialogQuery,
      name:data.dialogQueryName,
      sort:data.dialogQuerySort,
      count:data.dialogQueryCount
    })
    localStorage.setItem("saved",JSON.stringify(data.saved))
    data.dialogVisible = false
  }

  const openSaved = function(index,e){
    let el = e.target.closest(".el-tabs").querySelector("#tab-0.el-tabs__item")
    el.click()
    data.query = data.saved[index].query
    data.count = data.saved[index].count
    data.order = data.saved[index].sort
  }

  const editSaved = index => {
    data.dialogIsEdit = true
    data.dialogQuery = data.saved[index].query
    data.dialogQueryName = data.saved[index].name
    data.dialogQuerySort = data.saved[index].sort
    data.dialogQueryCount = data.saved[index].count
    data.editingIndex = index
    data.dialogVisible = true
  }

  function saveEdited() {
    data.saved[data.editingIndex].query = data.dialogQuery
    data.saved[data.editingIndex].name = data.dialogQueryName
    data.saved[data.editingIndex].sort = data.dialogQuerySort
    data.saved[data.editingIndex].count = data.dialogQueryCount
    localStorage.setItem("saved",JSON.stringify(data.saved))
    data.dialogVisible = false
    data.dialogIsEdit = false
    data.editingIndex = null
  }

  function confirmDeleteSaved(index){
    ElMessageBox.confirm("Вы действительно хотите удалить закладку?",{
      confirmButtonText:"Да",
      cancelButtonText:"Нет"
    })
    .then(() => {
      data.saved.splice(index,1)
      localStorage.setItem("saved", JSON.stringify(data.saved))
    })
    .catch(()=>{})
  }

  const logout = () => {
    localStorage.removeItem('token')
    location = '/'
  }
</script>



<script>
// script
  let api_key = "AIzaSyCMR0XiP14UY-Pqa2ZCeQwAsZLlYD_YuPs"
  function loadClient() {
    gapi.client.setApiKey(api_key);
    return gapi.client.load("https://www.googleapis.com/discovery/v1/apis/youtube/v3/rest")
        .then(function() { console.log("GAPI client loaded for API"); },
              function(err) { console.error("Error loading GAPI client for API", err); });
  }

  gapi.load("client:auth2", function() {
    gapi.auth2.init({client_id: "120869204599-g82tcoa4vf38a9558665t2o4i8h5677h.apps.googleusercontent.com"})
    .then(function() {
      loadClient()
    })
  });
</script>
<template>
  <div class="home">
    <el-image class="logo" :src="logo"/>
    <el-tabs>
      <!-- <el-image src="/src/assets/logo.png" fit="cover"/> -->
      <el-tab-pane label="Поиск" class="search-pane">
        <span class="home__title">Поиск видео</span>
        <el-input v-model="data.query" placeholder="Что хотите посмотреть?">
          <template #prepend>
            <el-button @click="addSaved" type="primary" :icon="Heart"></el-button>
          </template>
          <template #append>
            <el-button @click="update">Поиск</el-button>
          </template>
        </el-input>
        <div class="filters-bar">
          <el-select v-model="data.order">
            <el-option label="Релевантность" value="relevance"></el-option>
            <el-option label="Дата публикации" value="date"></el-option>
            <el-option label="Рейтинг" value="rating"></el-option>
            <el-option label="Название" value="title"></el-option>
            <el-option label="Количество просмотров" value="viewCount"></el-option>
          </el-select>
          <div></div>
          <el-radio-group v-model="data.showMode">
            <el-radio-button :icon="List" label="List"></el-radio-button>
            <el-radio-button :icon="Grid" label="Grid"></el-radio-button>
          </el-radio-group>
        </div>
        <div :class="{list_mode:data.showMode=='List'}" class="video-cards">
          <video-card v-for="video in data.list" :key="video.id" :video="video"></video-card>
        </div>
      </el-tab-pane>
      <el-tab-pane label="Избранное" class="saved-pane">
        <h1>Избранное</h1>
        <div v-for="(mark, index) in data.saved" :key="index" class="saved-item">
          <span @click="openSaved(index, $event)" v-text="mark.name"></span>
          <el-button @click="editSaved(index)" plain type="primary" class="saved-item__btn">Изменить</el-button>
          <el-button @click="confirmDeleteSaved(index)" plain type="danger" class="saved-item__btn">Удалить</el-button>
        </div>
      </el-tab-pane>
    </el-tabs>

    <el-dialog v-model="data.dialogVisible" :title="data.dialogIsEdit?'Изменить':'Сохранить'">
      <el-form label-position="top">
        <el-form-item label="Запрос">
          <el-input v-model="data.dialogQuery" />
        </el-form-item>
        <el-form-item label="Название">
          <el-input v-model="data.dialogQueryName" />
        </el-form-item>
        <el-form-item label="Тип сортировки">
          <el-select v-model="data.dialogQuerySort" >
            <el-option label="Релевантность" value="relevance"></el-option>
            <el-option label="Дата публикации" value="date"></el-option>
            <el-option label="Рейтинг" value="rating"></el-option>
            <el-option label="Название" value="title"></el-option>
            <el-option label="Количество просмотров" value="viewCount"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Количество">
          <el-slider :max="50" v-model="data.dialogQueryCount" />
        </el-form-item>
      </el-form>
      <el-button @click="data.dialogVisible = false">Отменить</el-button>
      <el-button @click="data.dialogIsEdit?saveEdited():saveQuery()" type="primary">Сохранить</el-button>
    </el-dialog>
    <el-button @click="logout" class="logout-btn" type="text">Выйти</el-button>
  </div>
</template>

<style>
  
  #app {
    width: 100%;
    padding: 0;
  }
  .home {
    position: relative;
  }
  .logo {
    position: absolute;
    top: 0;
    left: 0px;
    width:60px;
  }
  .el-tabs__nav-scroll {
    padding: 20px 100px 0 100px;
  }
  .el-tabs__item {
    font-size: 18px;
    line-height: 20px;
    padding-bottom: 20px;
  }
  .logout-btn {
    position: absolute;
    right: 30px;
    top: 12px;
    font-size: 18px;

  }
  .search-pane, .saved-pane {
    padding: 0 50px;
  }
  .video-cards {
    display: grid;
    grid-template-columns: auto auto auto auto;
    padding: 10px 0;
    gap: 7px;
  }

  .video-cards.list_mode {
    grid-template-columns: 1fr;
  }

  .home__title {
    display: block;
    text-align: center;
    font-size: 28px;
    font-weight: normal;
  }
  .el-dialog {
    border-radius: var(--el-border-radius-base);
  }
  .saved-item {
    display: grid;
    grid-template-columns: 1fr auto auto;
    padding: 10px 15px;
    font-size: 18px;
  }

  .saved-item:not(:hover) .saved-item__btn {
    display: none;
  }

  .saved-item:hover {
    background-color: #f1f1f1;
  }
  .filters-bar {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: auto;
    margin-top: 10px;
  }
</style>