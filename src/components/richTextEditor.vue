<template>
  <div
    >
    <quill-editor
      ref="newEditor"
      :options="newOption"
      style="height: 200px; margin-bottom: 54px"
      v-model="editorContent"
      @change="editorChange">
    </quill-editor>
    <form action="" method="post" enctype="multipart/form-data" id="uploadFormMulti">
      <input style="display: none" :id="uniqueId" type="file" name="avator" multiple accept="image/jpg,image/jpeg,image/png,image/gif" @change="uploadImg('uploadFormMulti')"><!--style="display: none"-->
    </form>
  </div>
</template>
<script>
  import { quillEditor } from 'vue-quill-editor'
  import Quill from 'quill'
  import { ImageImport } from './modules/ImageImport.js'
  import { ImageResize } from './modules/ImageResize.js'
  Quill.register('modules/imageImport', ImageImport)
  Quill.register('modules/imageResize', ImageResize)

  export default {
    props: ['text', 'editorId'],
    components: {quillEditor},
    data(){
      return {
        editorContent: '',
        uniqueId: '',
        imgPercent: 0,
        imageLoading: false,
        newOption: {
          placeholder: '请填写简介',
          history: {
            delay: 100,
            maxStack: 100,
            userOnly: false
          },
          modules: {
            toolbar: [
              [{ 'size': ['small' ,'large', 'huge'] }],
              ['bold', 'italic', 'underline'],
              ['blockquote'],
              [{ 'color': [] }, { 'background': [] }],          // dropdown with defaults from theme
              [{ 'font': [] }],
              [{ 'list': 'ordered'}, { 'list': 'bullet' }],
              ['link', 'image', 'video']
            ],
            imageImport: true,
            imageResize: {
              displaySize: true
            },
          },
          strict: false,//should be false for ignore unexpected update
        },
        addImgRange: '',//全局参数，每次添加图片时记录当前索引和长度
      }
    },
    computed: {
    },
    methods: {
      editorChange({ editor, html, text }) {
        var vm = this
        vm.$emit('editorChange', html)
      },
      uploadImg: async function(id) {
        var vm = this
        vm.imageLoading = true
        var formData = new FormData($('#' + id)[0])
        console.log('uploadImg')
        try {
          const url = await vm.uploadImgReq(formData)
          console.log(url)
          if (url != null && url.length > 0) {

            var value = url
            value = value.indexOf('http') != -1 ? value : 'http:' + value
            let index = vm.addImgRange != null?vm.addImgRange.index:0
            vm.$refs.newEditor.quill.insertEmbed(index , 'image', value, Quill.sources.USER)
            var img = new Image()
            img.src = value
            vm.$refs.newEditor.quill.formatText(index, index + 1, 'width', 400 + 'px');
          } else {

          }
          document.getElementById(vm.uniqueId).value=''


        } catch ({message: msg}) {
          document.getElementById(vm.uniqueId).value=''

        }
        vm.imageLoading = false
      },
      uploadImgReq(formData) {
        return new Promise((resolve, reject) => {
          // 这里实现你自己的图片上传
          if (true) {
            resolve("http://img4.duitang.com/uploads/item/201603/01/20160301110042_xeTuF.thumb.700_0.jpeg")
          } else {
            reject({message: '图片上传失败'})
          }
        })
      },
    },
    created: function () {
      var vm = this
      vm.imgPercent = 0
      vm.editorContent = vm.text
        vm.uniqueId = vm.editorId?vm.editorId:'inputImg'
    },
    watch:{
      text: function () {
        var vm = this
        vm.editorContent = vm.text
      }
    },
    mounted() {
      var vm =this
      // you can use current editor object to do something(quill methods)

      var imgHandler = async function(image) {
        vm.addImgRange = vm.$refs.newEditor.quill.getSelection()
        if (image) {
          var fileInput = document.getElementById(vm.uniqueId) //隐藏的file文本ID
          fileInput.click() //加一个触发事件
        }
      }

      vm.$refs.newEditor.quill.getModule("toolbar").addHandler("image", imgHandler)
    }

  }
</script>
<style scoped>

</style>
