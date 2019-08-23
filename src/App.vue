<template>
	<div id="app">
		<div class="ingput-wrap">
			<input v-model="text" class="input-content">
			<div @click="getData" class="btn">查</div>
			<div @click="upLoad" class="btn">录</div>
			<div @click="upLoad" class="btn">录</div>
		</div>
		
		<div class="show-wrap" v-if="word">
			<div class="show">
				<div class="tip">单词:</div>
				<div>{{word.name}}</div>
			</div>


			<template v-if="word.translate.length">
				<div class="show">
					<div class="tip">单词翻译:</div>
					<div style="flex: 1;">
						<div v-for="item in word.translate">{{ item }}</div>
					</div>
				</div>
			</template>


			<template v-if="word.state.length > 0">
				<div class="show">
					<div class="tip">时态:</div>
					<div>
						<div v-for="item in word.state">{{ item[0] }}, &nbsp;&nbsp;{{ item[1] }}</div>
					</div>
				</div>
			</template>

			<template v-if="word.webPhrase.length > 0">
				<div class="show">
					<div class="tip">网络释义:</div>
					<div>
						<div v-for="item in word.webPhrase">{{ item[0] }}, &nbsp;&nbsp;{{ item[1] }}</div>
					</div>
				</div>
			</template>


			<template v-if="word.wordGroup.length > 0">
				<div class="show">
					<div class="tip">词组:</div>
					<div>
						<div v-for="item in word.wordGroup">{{ item[0] }}, &nbsp;&nbsp;{{ item[1] }}</div>
					</div>
				</div>
			</template>


			<template v-if="word.sentence.length > 0">
				<div class="show">
					<div class="tip">例句:</div>
					<div style="flex: 1;">
						<div v-for="item in word.sentence">{{ item[0] }}, &nbsp;&nbsp;{{ item[1] }}</div>
					</div>
				</div>
			</template>





			<template v-if="word.difference.length > 0">
				<div class="show" v-for="(item, index) in word.difference">
					<div class="tip">词语辨析{{index + 1}}:</div>
					<div>
						<div> {{ item.title }} , {{ item.describe[0]  }} </div>
						<div v-for="item2 in item.explain">{{ item2 }}</div>
					</div>
				</div>
			</template>


			<template v-if="word.synonym.length >0">
				<div class="show">
					<div class="tip">同义词:</div>
					<div>
						<div v-for="item in word.synonym">{{ item[0] }}, &nbsp;&nbsp;{{ item[1] }}</div>
					</div>
				</div>
			</template>

			<template v-if="Object.keys(word.conjugate).length > 0">
				<div class="show">
					<div class="tip">同根词:</div>
					<div>
						<div v-for="item in Object.keys(word.conjugate).filter((s)=>s!=='world_root')">
							<div style="margin-bottom: 1px;margin-top: 3px;">{{item}}</div>
							<div v-for="item2 in word.conjugate[item]">{{ item2[0] }}, {{ item2[1]}} </div>
						</div>
					</div>
				</div>
			</template>

		</div>

	</div>
</template>

<script>
	const cheerio = require("cheerio");
	const axios = require("axios");
	import qs from 'qs'
	import md5 from 'js-md5'

	export default {
		name: 'app',
		data() {
			return {
				text: '',
				word: ''
			}
		},
		methods: {
			mySort(param) {
				let arr = Object.keys(param).sort();
				let newParam = {};
				arr.forEach((item) => {
					newParam[item] = param[item];
				})
				return newParam;
			},

			// 把爬取的数据发给后台
			upLoad() {
				let word = {a: 1} 
				let bodydata = md5(JSON.stringify(word))
				let authPwd = '4G:' + md5(bodydata + '4fbfc9b4d58838f1757b68c8eff5b5564ba2c7fb')
				console.log(authPwd)

				axios({
					methods: 'post',
					url: 'http://dreamtest.strongwind.cn:7150/v2/eng/translate',
					headers: {
						'Content-Type': 'application/json',
						Authentication: authPwd,
						Accept: 'application/json'
					},
					data: word
				})
			},

			// 爬去数据函数
			async getData() {

				let word = {
					name: '',
					translate: [],
					wordGroup: [],
					synonym: [],
					conjugate: {},
					difference: [],
					sentence: [],
					webPhrase: [],
					state: [],
					state_origin: ''
				}

				let wordKey = this.text
				word.name = wordKey
				await axios.get(`http://dict.youdao.com/w/${wordKey}`).then((res) => {

						var $ = cheerio.load(res.data);
						let list_1 = $('.trans-container.tab-content.hide')

						let word_list = []

						// tab1
						let wrap = $('#wordGroup').find('.wordGroup')
						if (wrap.length > 0) {
							wrap.each(function(index, element) {
								element.children.forEach((item2) => {
									let str = $(item2).text().replace(/\n/g, '').trim()
									if (str) {
										word_list.push(str)
									}
								})
								word.wordGroup.push(word_list)
								word_list = []
							})
						}


						// tab2
						let wrap1_li = $('#synonyms').find('li')
						if (wrap1_li.length > 0) {
							wrap1_li.each(function(index, element) {
								word.synonym.push([$(element).text()])
							})
						}


						let wrap1_p = $('#synonyms').find('.wordGroup')

						if (wrap1_p.length > 0) {
							wrap1_p.each(function(index, element) {
								let value = $(element).text().trim().replace(/\n/g, '')
								let list = value.split(',')
								let newList = []
								list.forEach(item8 => {
									let val3 = item8.trim()
									if (val3) {
										newList.push(val3)
									}
								})
								word.synonym[index].push(newList.join(','))
							})
						}


						// tab3
						const wrap3 = $('#relWordTab')

						if (wrap3.length > 0) {
							let temp = []
							let current = ''
							let keys = 'world_root'
							wrap3['0'].children.forEach((item, index) => {
								let str = ''
								let temp1 = []



								if (item.name === 'p') {
									item.children.forEach((item2) => {
										str = $(item2).text().replace(/((\n)|(\s))/g, '')
										str && temp1.push(str)
									})
								} else {
									str = $(item).text().replace(/((\n)|(\s))/g, '')
									str && (keys = str)
								}
								if (temp1.length === 0) return
								if (word.conjugate[keys]) {
									word.conjugate[keys].push(temp1)
								} else {
									word.conjugate[keys] = [temp1]
								}


								str && temp.push(str)

							})
						}


						// tab4
						let wrap4 = $('#discriminate').find('.wt-container')
						if (wrap4.length > 0) {
							wrap4.each(function(index, element) {
								let dom = $(element)
								let obj = {
									title: [],
									describe: [],
									explain: []
								}

								let title = dom.find('.title').text().replace(/\n|\s/g, '')
								obj.title.push(title)

								dom.find('.collapse-content')['0'].children.forEach((item5) => {
									if (item5.name === 'p' && !(item5.attribs.class)) {

										obj.describe.push($(item5).text())
									} else if (item5.name === 'div') {
										let explain_item = []
										$(item5).find('p')['0'].children.forEach((item6) => {
											const str = $(item6).text().replace(/\n|\s/g, '')
											str && (explain_item.push(str))
										})
										obj.explain.push(explain_item)
									}
								})


								word.difference.push(obj)
							})
						}


						let wrap5 = $('#bilingual')

						let ol = $(wrap5).find('.ol')
						let li_List = $(ol).find('li')
						let demo = []

						li_List.each(function(index, element) {
							let temp = []
							element.children.forEach((sentence) => {
								if (sentence.attribs && sentence.attribs.class === 'example-via') return
								let s = $(sentence).text().replace(/\n|\t/g, '').trim()
								s && temp.push(s)
							})
							word.sentence[index] = temp
						})



						//翻译			
						let wrap6 = $('#phrsListTab').find('.trans-container')
						let oUl = $(wrap6).find('ul')['0']
						if (oUl) {
							let translate = []
							oUl.children.forEach((item) => {
								const s = $(item).text().replace(/\n/g, '').trim()
								s && translate.push(s)
							})
							word.translate = translate
						}

						// 时态
						let oP_t = $(wrap6).find('p')['0']

						if (oP_t) {
							let text = $(oP_t).text()
							word.state_origin = text.replace(/(\n)|(\s)/g, '')
							let keys = text.match(/([\u4e00-\u9fa5])+/g)
							let values = text.match(/[a-zA-Z]+/g)
							let len = keys.length
							let list = []
							for (let ki = 0; ki < len; ++ki) {
								list[ki] = [values[ki], keys[ki]]
							}
							word.state = list
						}




						// 网络短语
						let webPhrase = []
						let wrap7 = $('#webPhrase').find('.wordGroup')
						if (wrap7.length > 0) {
							wrap7.each(function(index, element) {
								let webPhraseList = []
								element.children.forEach((item7) => {
									let str7

									if (item7.name === 'span') {
										str7 = $(item7).text()
									} else {
										str7 = $(item7).text().replace(/(\n|\s)/g, '')
									}
									str7 && webPhraseList.push(str7)
								})
								webPhrase[index] = webPhraseList
							})
							word.webPhrase = webPhrase
						}
						console.log(word)
						this.word = word
					})
					.catch((err) => {
						alert('出错了')
					})

			}
		}
	}
</script>

<style>
	#app {
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		font-size: 14px;
		color: #2c3e50;

	}

	html,
	body {
		width: 100%;
		height: 100%;
		margin: 0;
		padding: 0;

	}

	.ingput-wrap {
		display: flex;
		flex-direction: row;
		justify-content: center;
	}

	.input-content {
		border: 1px solid black;
		border-radius: 20px;
		outline: none;
		padding-left: 10px;
		width: 300px;
		margin-right: 20px;
	}

	.input-content:focus {
		border: 1px solid orangered;
		color: red;

	}

	.btn {
		cursor: pointer;
		background-color: orange;
		border-radius: 50%;
		color: white;
		width: 30px;
		height: 30px;
		text-align: center;
		line-height: 30px;
		user-select: none;
	}

	.btn:active {
		background-color: red;
	}

	.show {
		display: flex;
		margin-bottom: 10px;
	}

	.tip {
		width: 75px;
		text-align: center;
		word-break: keep-all;
	}
</style>
