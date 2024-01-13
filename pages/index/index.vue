<template>
	<view class="container">
		<view class="logo-container">
			<image class="logo" src="/static/logo.jpg"></image>
		</view>
		<view class="title">{{ survey.title }}</view>
		<view v-for="(question, index) in survey.questions" :key="index" class="question">
			<view class="item">
				<view class="question-title">{{ index + 1 }}. {{ question.title }}</view>
				<view v-if="question.type === 'radio'" class="options">
					<radio-group>
						<radio style="margin-right: 30rpx;margin-top: 10rpx;"
							v-for="(option, optionIndex) in question.options" :key="optionIndex" :value="question.values[optionIndex]"
							@click="handleRadioClick(question, question.values[optionIndex])">{{ option }}
						</radio>
					</radio-group>
				</view>
				<view v-else-if="question.type === 'text' || question.type === 'number'">
					<input :type="question.type" :placeholder="question.placeholder" v-model="question.answer"
						class="inputFrame">
				</view>
				<view v-else-if="question.type === 'textarea'">
					<textarea style="border: 1rpx solid #ccc;padding-left: 10rpx;padding-top: 10rpx;" 
					:placeholder="question.placeholder" v-model="question.answer" 
					> </textarea>
				</view>
				<view v-else-if="question.type === 'select'" class="select">
					<picker mode="selector" :range="question.options" @change="handlePickerChange($event, index)">
						<view class="picker">{{ question.answer || '请选择' }}</view>
					</picker>
				</view>
				<view v-else-if="question.type === 'time'" class="time">
					<picker mode="date" @change="handleTimeChange($event, index)">
						<view class="picker">{{ question.answer || '请选择时间' }}</view>
					</picker>
				</view>
				<view v-else-if="question.type === 'image'" class="image-upload">
					<uni-file-picker 
						v-model="imageValue" 
						fileMediatype="image" 
						mode="grid" 
						@select="select" 
						@progress="progress" 
						@success="success" 
						@fail="fail" 
						:limit="1"
					/>
					<!-- <button @click="handleImageUpload(question)">上传图片</button>
					<image :src="question.answer" v-if="question.answer"
						style="height: 200rpx;width: 200rpx;margin-top:30rpx"></image> -->
				</view>
			</view>
		</view>
		<view style="position: relative">
			<button :class="submitClass" @tap="handleSubmit">提交</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imageValue:[],
				submitClass: "submit-button-before-upload",
				survey: {
					title: "无高校证明开具",
					questions: [
						{
							type: "image",
							title: "上传毕业证书",
							answer: ""
						},
						{
							type: "text",
							title: "姓名",
							placeholder: "姓名",
							answer: ""
						},
						{
							type: "radio",
							title: "性别",
							options: ["男", "女"],
							values: [0, 1],
							answer: ''
						},
						{
							type: "text",
							title: "毕业学院（需要与毕业证保持一致）",
							placeholder: "毕业学院",
							answer: ""
						},
						// {
						// 	type: "select",
						// 	title: "毕业学院",
						// 	options: [
						// 		"萃英学院",
						// 		"政治与国际关系学院",
						// 		"艺术学院",
						// 		"管理学院",
						// 		"经济学院",
						// 		"马克思主义学院",
						// 		"法学院",
						// 		"外国语学院",
						// 		"哲学社会学院",
						// 		"历史文化学院",
						// 		"新闻与传播学院",
						// 		"文学院",
						// 		"生态学院",
						// 		"材料与能源学院",
						// 		"动物医学与生物安全学院",
						// 		"地质科学与矿产资源学院",
						// 		"土木工程与力学学院",
						// 		"资源环境学院",
						// 		"大气科学学院",
						// 		"生命科学学院",
						// 		"草地农业科技学院",
						// 		"化学化工学院",
						// 		"信息科学与工程学院",
						// 		"核科学与技术学院",
						// 		"物理科学与技术学院",
						// 		"数学与统计学院",
						// 		"护理学院",
						// 		"口腔医学院",
						// 		"第二临床医学院",
						// 		"第一临床医学院",
						// 		"基础医学院",
						// 		"药学院",
						// 		"公共卫生学院",
						// 		"其他"
						// 	],
						// 	answer: ""
						// },
						{
							type: "text",
							title: "专业名称（需要与毕业证保持一致）",
							placeholder: "专业名称",
							answer: ""
						},
						{
							type: "number",
							title: "请输入毕业年份（格式：2020）",
							placeholder: "毕业年份需要与毕业证保持一致",
							answer: ""
						},
						{
							type: "number",
							title: "请输入您的身份证号码",
							placeholder: "请输入18位合法身份证号码",
							answer: ""
						},
						{
							type: "number",
							title: "联系方式",
							placeholder: "请输入手机号",
							answer: ""
						},
						{
							type: "textarea",
							title: "邮寄地址",
							placeholder: "请输入详细地址",
							answer: ""
						}
						
					]
				}
			};
		},
		methods: {
			onFileChange(e) {
				console.log(e.target.files[0])
			},
			handleRadioClick(question, value) {
				question.answer = value;
			},
			handleCheckboxClick(question, option) {
				const index = question.answer.indexOf(option);
				if (index > -1) {
					question.answer.splice(index, 1);
				} else {
					question.answer.push(option);
				}

				if (question.other && question.answer.includes("其他")) {
					question.checkedOther = true;
				} else {
					question.checkedOther = false;
				}
			},
			handlePickerChange(e, index) {
				const selectedValue = e.detail.value;
				this.survey.questions[index].answer = this.survey.questions[index].options[selectedValue];
			},
			handleTimeChange(e, index) {
				const selectedValue = e.detail.value;
				this.survey.questions[index].answer = e.detail.value;
			},
			// 获取上传状态
			select(e){
				console.log('选择文件：',e.tempFilePaths[0])
				this.survey.questions[0].answer = e.tempFilePaths[0]
			},
			// 获取上传进度
			progress(e){
				console.log('上传进度：',e)
			},
			
			// 上传成功
			success(e){
				console.log('上传成功')
			},
			
			// 上传失败
			fail(e){
				console.log('上传失败：',e)
			},
			handleSubmit() {
				console.log("this.survey.questions", this.survey.questions);
				let emptyQuestions = [];
				let invalidPhoneIndex = -1;
				let invalidIDCardIndex = -1;
				let invalidGradeIndex = -1;
				let imageIndex=-1;
				const phonePattern = /^1[3456789]\d{9}$/;
				const idCardPattern = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
				const GradeIndex = /^\d{4}$/;
				const currentDate = new Date();
				const currentYear = currentDate.getFullYear();
				
				for (let index = 0; index < this.survey.questions.length; index++) {
					let question = this.survey.questions[index]
					
					if (question.type === 'image' && question.title === '上传毕业证书') {
						imageIndex = index;
						if (!this.survey.questions[0].answer) {
							uni.showToast({
								title: `请上传图片`,
								icon: 'none',
								duration: 2000
							});
							return
						}
					}
					if (question.type === 'number' && question.title.indexOf('请输入毕业年份') !== -1 && ( !GradeIndex.test(question.answer) || question.answer >  currentYear) ) {
						invalidGradeIndex = index
					}
					if (!question.answer && question.answer !== 0) {
						emptyQuestions.push(index);
					}
					if (question.type === 'number' && question.title === '请输入您的身份证号码' && !idCardPattern.test(
							question
							.answer)) {
						invalidIDCardIndex = index;
					}
					if (question.type === 'number' && question.title === '联系方式' && !phonePattern.test(question
							.answer)) {
						invalidPhoneIndex = index;
					}
				}

				if (emptyQuestions.length > 0) {
					emptyQuestions.forEach((index) => {
						const questionTitle = this.survey.questions[index].title;
						uni.showToast({
							title: `请填写${questionTitle}`,
							icon: 'none',
							duration: 2000
						});
					});
				} else if (invalidGradeIndex !== -1) {
					const questionTitle = this.survey.questions[invalidGradeIndex].title;
					uni.showToast({
						title: `请填写正确的毕业年份`,
						icon: 'none',
						duration: 2000
					});
				} else if (invalidPhoneIndex !== -1) {
					const questionTitle = this.survey.questions[invalidPhoneIndex].title;
					uni.showToast({
						title: `请填写正确的手机号`,
						icon: 'none',
						duration: 2000
					});
				} else if (invalidIDCardIndex !== -1) {
					const questionTitle = this.survey.questions[invalidIDCardIndex].title;
					uni.showToast({
						title: `请填写正确的身份证号码`,
						icon: 'none',
						duration: 2000
					});
				} else {
					let that = this
					uni.uploadFile({
						url: 'https://collect-and-send-email-admin-lzu.replit.app/upload/',
						filePath: this.survey.questions[imageIndex].answer,  
						name: 'file',
						formData: {
							g_id_no: this.survey.questions[6].answer, // 身份证号码
						},
						success: (res) => {
							console.log('图片上传成功！');
							console.log(JSON.parse(res.data));
							const data = JSON.parse(res.data)
							that.survey.questions[imageIndex].answer = data.file_url
							
							// 最后一步：上传数据到后端
							const graduate_data = that.survey.questions
							console.log("graduate_data", graduate_data)
							uni.request({
								url: 'https://collect-and-send-email-admin-lzu.replit.app/graduate_add/',
								method: 'POST',
								header: {
									'Content-Type': 'application/x-www-form-urlencoded' // 设置请求头
								},
								data: {
									g_cert_pic: graduate_data[0].answer,
									g_name: graduate_data[1].answer,
									g_sex: graduate_data[2].answer,
									g_college: graduate_data[3].answer,
									g_major: graduate_data[4].answer,
									g_year: graduate_data[5].answer,
									g_id_no: graduate_data[6].answer,
									g_phone: graduate_data[7].answer,
									g_mailing_address: graduate_data[8].answer
								},
								success: (res) => {
									uni.showToast({
										title: `信息上传成功`,
										icon: 'none',
										duration: 2000
									});
								},
								fail: (err) => {
									uni.showToast({
										title: `信息上传失败：${err.errMsg}`,
										icon: 'none',
										duration: 2000
									});
								}
							})
							
							
							
						},
						fail: (err) => {
							uni.showToast({
								title: `图片上传失败`,
								icon: 'none',
								duration: 2000
							});
							console.error('图片上传失败！');
							console.error(err);
						}
					});
					console.log(this.survey);
				}
				
				
				
			}
		}
	};
</script>

<style>
	.container {
		padding: 25rpx;
		border: rgb(0, 133, 200) 20rpx solid;

	}

	.logo-container {
		display: flex;
		justify-content: center;
		height: 250rpx;

	}

	.logo {
		height: 200rpx;
		width: 200rpx;
	}

	.item {
		height: 100%;
		width: 95%;
		padding-left: 20rpx;
		padding-top: 25rpx;
		/* border: #ffc7c8 2rpx solid; */
	}

	.title {
		font-size: 70rpx;
		font-weight: bold;
		text-align: center;
		margin-bottom: 40rpx;
	}


	.question {
		margin-bottom: 35rpx;
	}

	.question-title {
		font-size: 35rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
	}

	.options {
		margin: 10rpx;
	}

	.input {
		margin-top: 10rpx;
		height: 30rpx;
		width: 80vw;
	}

	.inputFrame {
		border: 1rpx solid #ccc;
		height: 60rpx;
		line-height: 60rpx;
		padding-left: 10rpx;
		margin-top: 10rpx;
		width: 98%;
		border-radius: 5rpx;
	}

	.select {
		margin-top: 10rpx;

	}

	.time {
		margin-top: 10rpx;
	}

	.picker {
		border: 1rpx solid #ccc;
		height: 80rpx;
		line-height: 80rpx;
		padding-left: 10rpx;
		margin-top: 10rpx;
	}

	.submit-button-before-upload {
		width: 100%;
		background-color: #007aff;
		color: #fff;
		font-size: 30rpx;
		text-align: center;
		padding: 15rpx 0;
		border-radius: 5rpx;
		margin-top: 70rpx;
	}

	.submit-button-after-upload {
		width: 100%;
		background-color: #007aff;
		color: #fff;
		font-size: 30rpx;
		text-align: center;
		padding: 15rpx 0;
		border-radius: 5rpx;
		margin-top: 300rpx;
	}

</style>
