<template>
	<div class="editorContainer">
		<el-form label-width="100px" label-position="right" :modal="formContent">
			<div class="clearfix"></div>
			<el-form-item label='封面图片'>
				<div v-if="picList.length!=0" style="width:100%">
					<div style='float:left;margin-right:10px' v-for="item in picList" :key='item.name' class="picBox">
						<img  class="pic" :src='item.url'/>
						<p><i @click="delPic(item)" class="el-icon-delete"></i></p>
					</div>
				</div>
				<input ref="updateCover" @change="updateCover()" id="updateCover" type="file"/>
			</el-form-item>
			<el-form-item label='视频文件'>
				<el-upload :action="actionUrl" :multiline="false"  :auto-upload='true'  :limit="1" :on-success='getVideo' :on-remove="delVideo" :file-list="videoList">
					<el-button size="small" type="primary">点击上传</el-button>
				</el-upload>
			</el-form-item>				
			<el-form-item label="正文">
				<input type='file' @change="update" ref='contentUpload' id='avatar-uploader'/>
				<quill-editor v-model="formContent.Body" ref="myQuillEditor" :options="editorOption" class='editor' >
					<img id='contentPic' v-if="imgUrl!=''" :src='imgUrl'/>
				</quill-editor>	
			</el-form-item>
		</el-form>
	</div>
</template>

<script>
import { quillEditor, Quill } from 'vue-quill-editor'
Quill.register('modules/ImageExtend', ImageExtend)
export default {
	data(){
		return {
			
		}
	},
	computed:{
		
	},
	created(){
		this.actionUrl = portsUrl+'/Article/AddImg';
		this.editorOption = {
			modules: {
				ImageExtend: {  // 如果不作设置，即{}  则依然开启复制粘贴功能且以base64插入 
					name: 'img',  // 图片参数名
					size: 1,  // 可选参数 图片大小，单位为M，1M = 1024kb
					action: this.actionUrl,  // 服务器地址, 如果action为空，则采用base64插入图片
					// response 为一个函数用来获取服务器返回的具体图片地址
					// 例如服务器返回{code: 200; data:{ url: 'baidu.com'}}
					// 则 return res.data.url
					response: (res) => {
						return res.data
					},
					headers: (xhr) => {
					// xhr.setRequestHeader('myHeader','myValue')
					},  // 可选参数 设置请求头部
				},
				 toolbar: {
					container: container,  // 工具栏
					handlers: {
						'image': function (value) {  //劫持quill自身的文件上传，用原生替换
						if (value) {
								document.querySelector('#avatar-uploader').click()
							} else {
								this.quill.format('image', false);
							}
						}
					}
				}
			}
		}

	},
	methods:{
		updateCover() {
			if(this.picList.length>=3){
				this.$message({type: 'warning',message: '封面图限制三张'});      
				return false;
			}
			let inputDOM = this.$refs.updateCover;
			// 通过DOM取文件数据
			this.file = inputDOM.files[0];
			var formdata = new FormData();
			formdata.append('file',document.getElementById("updateCover").files[0]);
			axios({
				url: this.actionUrl,
				method: 'post',
				data: formdata,
				headers: {'Content-Type': 'application/x-www-form-urlencoded'}
			}).then((re)=>{
				this.picList.push({
					name:this.picList.length,
					url:re.data.data[0]
				});
			}).catch(function (err) {
				console.log(err);
			})

		},
		update(){
            let inputDOM = this.$refs.contentUpload;
			this.file = inputDOM.files[0];
			var formdata = new FormData();
			formdata.append('file',document.getElementById("avatar-uploader").files[0]);
			axios({
				url: this.actionUrl,
				method: 'post',
				data: formdata,
				headers: {'Content-Type': 'application/x-www-form-urlencoded'}
			}).then((re)=>{
				this.imgUrl = re.data.data[0]
				let quill = this.$refs.myQuillEditor.quill;
				const range = quill.getSelection();
				if(range){
					quill.insertEmbed(range.index, 'image',this.imgUrl); 
				}  
				quill.setSelection(quill.getSelection().index+1)
			}).catch((err)=>{
				console.log(err)
			})
		},
	
	}
}
</script>

