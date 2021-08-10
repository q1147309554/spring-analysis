<template>
  <div class="feed-back">
    <header-bar text="意见反馈"></header-bar>

    <van-form @submit="onSubmit">
      <div class="cell-email">
        <van-field
            v-model="form_info.email"
            label="邮箱"
            name="email"
            placeholder="请输入您的邮箱"
            input-align="right"
        />
      </div>
      <span class="red-title"> *标题</span>
      <div class="cell-title">
      <van-field
          v-model="form_info.title"
          name="title"
          placeholder="20字以内"
          :rules="[{ pattern, message: '请输入20字以内标题' }]"
      />
      </div>
      <span class="red-title"> *问题和建议（照片选填）</span>
      <div class="cell-title">
        <van-field
            v-model="form_info.message"
            rows="1"
            autosize
            name="message"
            type="textarea"
            placeholder="请输入200字以内"
            :border="boo"
            style="min-height: 80px"
        />
        <van-field name="uploader">
          <template #input>
        <van-uploader
            :after-read="afterRead"
            v-model ="form_info.File_list"
            max-count="1"
        />
          </template>
        </van-field>
      </div>
      <div style="margin: 16px;">
        <van-button round block type="danger" native-type="submit">提交</van-button>
      </div>
    </van-form>


  </div>
</template>

<script>
import HeaderBar from "@/components/base/HeaderBar";
import {Notify} from "vant";
import axios from 'axios'
export default {
  name: "FeedBack",
  components: {HeaderBar},
  data() {
    return {
      key:'',
      form_info: {
        title: '',
        email: '',
        message:'',
        key:'',
        File_list: []
      },
      pattern: /^[ \u4e00-\u9fa5A-Za-z0-9]{1,20}$/,
      patterns: /^[ \u4e00-\u9fa5A-Za-z0-9]{1,200}$/,
      boo:false,
      File_list: [],
      url:''

    }
  },
  methods: {
   async onSubmit(values) {
     console.log('submit',values)
     if(values.uploader[0]){

       const{code, data} = await this.$axios.get('/api/common/upToken')
       if(code===200){
         const axiosInstance = axios.create({withCredentials: false});
         let formData = new FormData();
         formData.append('token',data.upToken)
         formData.append('file',values.uploader[0].file)
         formData.append('key',"advise"+'-'+this.$store.state.userInfo.userId+'-'+new Date().getTime() +'-'+values.uploader[0].file.name)
         axiosInstance({
           method: 'POST',
           url: 'https://upload.qiniup.com/',  //上传地址
           data: formData,
           timeout:30000,      //超时时间，因为图片上传时间有可能比较长
         }).then((res) =>{
           if(res.status == 200){
             // 上传成功， 返回的res中会带上文件的名字 例：1.jpg  传入的key是什么返回的就是什么
             let prcKey = data.domain +'/'+ res.data.key
             let form = {
               "adviceLogo": prcKey,
               "adviceType": 1,
               "advise": values.message,
               "email": values.email,
               "title": values.title,
               "userId": this.$store.state.userInfo.userId
             }
             this.$axios.post('/api/app/userAdvise/insert',form).then(
                 res =>{
                   if(res.code===200){
                       console.log(res.data)
                       Notify({ type: 'success', message: '反馈成功！' });
                       this.$router.go(-1)
                   }
                 }
             )

           }else{
             //上传失败
           }
         }).catch(function(err) {
           //上传失败
         });
       }
     }else {
       let form = {
         "adviceLogo": '',
         "adviceType": 1,
         "advise": values.message,
         "email": values.email,
         "title": values.title,
         "userId": this.$store.state.userInfo.userId
       }
       const{ code,data } = await this.$axios.post('/api/app/userAdvise/insert',form)
       if(code===200){
         console.log(data)
         Notify({ type: 'success', message: '反馈成功！' });
         this.$router.go(-1)
       }
     }

    },
    async afterRead(file) {
      // 此时可以自行将文件上传至服务器
      // console.log(file);
      // console.log(file.file);


    },
  },
}
</script>

<style lang="scss" scoped>
.cell-email,.cell-title{
  margin: 20px auto 20px;
  width: 90%;
  background: white;
  border-radius: 15px;
  border: solid #EEEEEE 1px;
  font-size: 24px;
  color: #050505;
}
.red-title{
  color: #DC2423;
  margin-left: 48px;
  font-size: 28px;
}
</style>