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
				isSubmitting: false,  // 添加一个标志变量来控制提交状态
				survey: {
					title: "无高校学生登记表证明",
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
							answer: "231"
						},
						{
							type: "radio",
							title: "性别",
							options: ["男", "女"],
							values: ["0", "1"],
							answer: ''
						},
						{
							type: "text",
							title: "毕业学院（需要与毕业证保持一致）",
							placeholder: "毕业学院",
							answer: "213"
						},
						{
							type: "text",
							title: "专业名称（需要与毕业证保持一致）",
							placeholder: "专业名称",
							answer: "123"
						},
						{
							type: "number",
							title: "请输入毕业年份（格式：2020）",
							placeholder: "毕业年份需要与毕业证保持一致",
							answer: "2020"
						},
						{
							type: "number",
							title: "请输入您的身份证号码",
							placeholder: "请输入18位合法身份证号码",
							answer: "371521202233331123"
						},
						{
							type: "number",
							title: "联系方式",
							placeholder: "请输入手机号",
							answer: "15022221140"
						},
						{
							type: "textarea",
							title: "邮寄地址",
							placeholder: "请输入详细地址",
							answer: "23213"
						},
						{
							type: "text",
							title: "邮箱",
							placeholder: "请输入邮箱（后续将通过邮箱发送证明邮寄通知）",
							answer: "123@qq.com"
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
			
			// JS 等待函数
			delay(time) {
			    return new Promise(resolve => setTimeout(resolve, time));
			},
			
			// 验证输入
			validateInput() {
				const validationResults = {
					emptyQuestions: [],
					invalidPhoneIndex: -1,
					invalidIDCardIndex: -1,
					invalidGradeIndex: -1,
					invalidEmailIndex: -1,
					imageIndex: -1,
				};
		
				const phonePattern = /^1[3456789]\d{9}$/;
				const idCardPattern = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
				const GradePattern = /^\d{4}$/;
				const emailPattern = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
				const currentDate = new Date();
				const currentYear = currentDate.getFullYear();
		
				for (let index = 0; index < this.survey.questions.length; index++) {
					const question = this.survey.questions[index];
					if (question.type === 'image' && question.title === '上传毕业证书') {
						validationResults.imageIndex = index;
						if (!question.answer) {
							uni.showToast({
								title: `请上传图片`,
								icon: 'none',
								duration: 2000
							});
							return validationResults;
						}
					}
		
					if (question.type === 'number' && question.title.includes('请输入毕业年份') && (!GradePattern.test(question.answer) || question.answer > currentYear)) {
						validationResults.invalidGradeIndex = index;
					}
		
					if (!question.answer && question.answer !== 0) {
						validationResults.emptyQuestions.push(index);
					}
		
					if (question.type === 'number' && question.title === '请输入您的身份证号码' && !idCardPattern.test(question.answer)) {
						validationResults.invalidIDCardIndex = index;
					}
		
					if (question.type === 'number' && question.title === '联系方式' && !phonePattern.test(question.answer)) {
						validationResults.invalidPhoneIndex = index;
					}
		
					if (question.type === 'text' && question.title === '邮箱' && !emailPattern.test(question.answer)) {
						validationResults.invalidEmailIndex = index;
					}
				}
		
				return validationResults;
			},
		
			handleSubmit() {
				if (this.isSubmitting) {
					uni.showToast({
						title: '正在提交，请稍候...',
						icon: 'none',
						duration: 2000
					});
					return;
				}
				this.isSubmitting = true;
		
				uni.showLoading({
					title: '请求提交中...'
				});

				const { emptyQuestions, invalidPhoneIndex, invalidIDCardIndex, invalidGradeIndex, invalidEmailIndex, imageIndex } = this.validateInput();
				
				let isReurn = false
				if (emptyQuestions.length > 0) {
					emptyQuestions.forEach((index) => {
						const questionTitle = this.survey.questions[index].title;
						uni.showToast({
							title: `请填写${questionTitle}`,
							icon: 'none',
							duration: 2000
						});
					});
					isReurn = true
				} else if (invalidGradeIndex !== -1) {
					const questionTitle = this.survey.questions[invalidGradeIndex].title;
					uni.showToast({
						title: `请填写正确的毕业年份`,
						icon: 'none',
						duration: 2000
					});
					isReurn = true
				} else if (invalidPhoneIndex !== -1) {
					const questionTitle = this.survey.questions[invalidPhoneIndex].title;
					uni.showToast({
						title: `请填写正确的手机号`,
						icon: 'none',
						duration: 2000
					});
					isReurn = true
				} else if (invalidIDCardIndex !== -1) {
					const questionTitle = this.survey.questions[invalidIDCardIndex].title;
					uni.showToast({
						title: `请填写正确的身份证号码`,
						icon: 'none',
						duration: 2000
					});
					isReurn = true
				} else if (invalidEmailIndex !== -1) {
					const questionTitle = this.survey.questions[invalidEmailIndex].title;
					uni.showToast({
						title: `请填写正确的${questionTitle}`,
						icon: 'none',
						duration: 2000
					});
					isReurn = true
				}
				
				if (isReurn) {
					this.isSubmitting = false; // 重置提交标志
					uni.hideLoading();
					return;
				}
		
				// 提交逻辑
				let that = this;
				uni.uploadFile({
					url: 'http://127.0.0.1:8000/upload/',
					filePath: this.survey.questions[imageIndex].answer,
					name: 'file',
					formData: {
						g_id_no: this.survey.questions[6].answer, // 身份证号码
					},
					success: (res) => {
						console.log('图片上传成功！');
						const data = JSON.parse(res.data);
						that.survey.questions[imageIndex].answer = data.file_url;
						
						// 上传数据到后端
						const graduate_data = that.survey.questions;
						console.log("graduate_data", graduate_data);
						uni.request({
							url: 'http://127.0.0.1:8000/graduate_add/',
							method: 'POST',
							header: {
								'Content-Type': 'application/x-www-form-urlencoded'
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
								g_mailing_address: graduate_data[8].answer,
								g_email: graduate_data[9].answer
							},
							success: () => {
								uni.navigateTo({
									url: '../index/success'
								});
							},
							fail: (err) => {
								uni.showToast({
									title: `信息上传失败：${err.errMsg}`,
									icon: 'none',
									duration: 2000
								});
							},
							complete: () => {
								// 无论成功失败，提交完成后都需要重置提交标志
								that.isSubmitting = false;
								uni.hideLoading();
							}
						});
					},
					fail: (err) => {
						uni.showToast({
							title: `图片上传失败`,
							icon: 'none',
							duration: 2000
						});
						console.error('图片上传失败！');
						console.error(err);
						that.isSubmitting = false; // 重置提交标志
						uni.hideLoading();
					}
				});
								
				
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
		font-size: 56rpx;
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
