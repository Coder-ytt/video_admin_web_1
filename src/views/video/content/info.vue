<template>
<div class="app-container">
  <h1 style="text-align: center">添加作品</h1>
  <el-steps :active="1" finish-status="success">
    <el-step title="填写作品基本信息"></el-step>
    <el-step title="添加章节视频"></el-step>
    <el-step title="完成"></el-step>
  </el-steps>

  <el-form label-width="120px" style="margin-top: 10px">
    <!--标题-->
    <el-form-item label="作品标题">
      <el-input placeholder="请填写作品标题" v-model="contentVO.title"/>
    </el-form-item>
    <!--分类-->
    <el-form-item label="作品分类">
      <el-select placeholder="一级分类" v-model="contentVO.categoryParentId" @change="oneCategoryChanged">
        <el-option v-for="oneCayegory in oneCategoryList"
                   :key="oneCayegory.id"
                   :label="oneCayegory.title"
                   :value="oneCayegory.id" />
      </el-select>
      &nbsp;
      <el-select placeholder="二级分类" v-model="contentVO.categoryId">
        <el-option v-for="twoCayegory in twoCategoryList"
                   :key="twoCayegory.id"
                   :label="twoCayegory.title"
                   :value="twoCayegory.id" />
      </el-select>
    </el-form-item>
    <!--作者-->
    <el-form-item label="作者">
      <el-select placeholder="选择作者" v-model="contentVO.authorId">
        <el-option v-for="author in authorList"
                   :key="author.id"
                   :label="author.name"
                   :value="author.id"
        />
      </el-select>
    </el-form-item>
    <!--总视频数-->
    <el-form-item label="总视频数">
      <el-input :min="0"  placeholder="总视频数" v-model="contentVO.contentNum"/>
    </el-form-item>
    <!--简介-->
    <el-form-item label="作品简介">
      <tinymce ref="editor" v-model="contentVO.description"/>
    </el-form-item>
    <!--封面-->
    <el-form-item label="作品封面">

      <el-upload
        :show-file-list="false"
        :action="BASE_API+'/service_upload/file/ossUploadFile'"
        :on-success="uploadSuccess"
        :limit="1"
        name="file"
      >
        <img :src="contentVO.cover" style="width: 300px; height: 150px"/>
      </el-upload>


    </el-form-item>
    <!--价格-->
    <el-form-item label="价格">
      <el-input :min="0"  placeholder="价格" v-model="contentVO.price"/>
    </el-form-item>
    <el-button style="margin-top: 12px;" type="primary" @click="saveAndNext">保存并下一步</el-button>
  </el-form>


</div>
</template>

<script>
  import content from '@/api/video/content'
  import category from '@/api/video/category'
  import tinymce from '@/components/Tinymce/tinymce'
  export default {
    components:{
      'tinymce':tinymce
    },
    data(){
      return {
        contentVO:{
          title:'',//标题
          categoryId:'',//二级分类ID
          categoryParentId:'',//一级分类ID
          authorId:'',//作者
          contentNum:'',//视频总数
          description:'',//简介
          cover:require('@/assets/cover.jpg'), //封面
          price:0 //价格
        },
        contentId:'', //作品的id
        authorList:[],//作者列表
        oneCategoryList:[],//一级分类 列表
        twoCategoryList:[],//二级分类 列表
        BASE_API:process.env.VUE_APP_BASE_API
      };
    },
    created() {

      //判断路由里面有没有id 如果有 修改 做数据回显
      if(this.$route.params&&this.$route.params.id){
        this.contentId = this.$route.params.id;
        //根据id查询当前的作品 回显
        this.getInfo();
      }else{
        //加载所有的作者
        this.getAuthorList()
        //加载所有的分类
        this.getCategoryList()
      }
    },
    methods: {

      //根据id获取作品信息
      getInfo(){
        content.getContentWithInfoId(this.contentId).then(res=>{
           this.contentVO = res.data.contentVO;

            //1.获取一级分类
            category.getCategoryData().then(res=>{
              this.oneCategoryList = res.data.list;

              //遍历每一个一级分类，判断当前的contentVO 是属于哪一个一级分类
              for(var i = 0;i<this.oneCategoryList.length;i++){
                let oneCategory = this.oneCategoryList[i];
                if(oneCategory.id == this.contentVO.categoryParentId){
                  this.twoCategoryList = oneCategory.children;
                }
              }

            });

           //加载作者列表
          this.getAuthorList();

        });
      },
      getAuthorList(){
        content.getAuthorList().then(res=>{
          this.authorList = res.data.list;
        });
      },
      getCategoryList(){
        //1.获取一级分类
        category.getCategoryData().then(res=>{
          this.oneCategoryList = res.data.list;
        });
      },
      //当一级分类改变时调用的方法 参数：当前一级分类选择的id
      oneCategoryChanged(value){
        for(var i = 0;i<this.oneCategoryList.length;i++){
          var category = this.oneCategoryList[i];
          if(value ===category.id){
            this.twoCategoryList = category.children;
            //清空已经选择的二级分类
            this.contentVO.categoryId='';
          }
        }
      },
      //上传封面成功回调
      uploadSuccess(res,file){
        this.contentVO.cover = res.data.url;
      },
      saveAndNext(){
        //判断当前是添加还是更新
        if(this.contentId == ''){
          content.addContentInfo(this.contentVO).then(res=>{
            this.$message({
              type:'success',
              message:'添加成功'
            });
            //路由跳转
            this.$router.push({path:'/content/chapter/'+res.data.contentId})
          })
        }
        else{
          //更新操作
          content.updateContentInfo(this.contentVO).then(res=>{
            this.$message({
              type:'success',
              message:'更新信息成功'
            });
            //路由跳转 跳转到章节
            this.$router.push({path:'/content/chapter/'+this.contentId})
          })
        }
      }
    }
  }
</script>

<style scoped>

</style>
