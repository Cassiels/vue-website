<template>
	<div class="user-info">
<!-- 		<p class="desc">敬请期待...</p>
		<span class="loginout" @click="loginout">退出登录</span> -->
		<div class="info-content">
			<div class="user-basic-info" v-if='personalCenter && personalCenter'>
				<div class="user-basic-l">
					<div class="user-avatar">
						<div class="image-avatar" style="background-image: url('http://daiwei.org/vue/bg/avatar1.jpg');background-size:cover;background-position:center"></div>
						<span class="change-avatar"></span>
					</div>
					<div class="user-basic">
						<h3 class="user-nickname">{{personalCenter.nickname == '' ? personalCenter.username : ''}}</h3>
						<p class="user-disc">{{personalCenter.desc !== '' ? personalCenter.desc : '暂无描述'}}</p>
					</div>
				</div>
				<div class="user-basic-r">
					<div class="icon-setinfo" title="累计听歌数量">
						<i class="icon-music"></i>
						<span class="count">{{personalCenter.musicCount || 0}}</span>
					</div>
					<div class="icon-setinfo need-click" title="个人设置">
						<i class="icon-setting"></i>
						<span class="count">个人设置</span>
					</div>
					<div class="icon-setinfo need-click" @click="loginout" title="退出登录">
						<i class="icon-setting"></i>
						<span class="count">退出登录</span>
					</div>
				</div>
			</div>
			<div class="mode-info">
				
			</div>
			<div class="user-detail-info">
				<!-- 用户的具体信息。音乐收藏。评价 点赞 互相评论的功能 -->
				<ul class="detail-title">
					<li class="title-list" @click="selectSettingIndex" data-index="0" :class="selectIndex === 0 ? 'active': ''">音乐收藏</li>
					<li class="title-list" @click="selectSettingIndex" data-index="1" :class="selectIndex === 1 ? 'active': ''">我的建议</li>
				</ul>
				<div class="detail-content">
					<div class="content-info list_content_info music-list" :class="selectIndex === 0 ? 'active': ''">
						<div class="music_list_title border-1px">
							<span class="music_index"></span>
							<span class="music_name">歌曲</span>
							<span class="music_singer">歌手</span>
							<span class="music_zhuanji">专辑</span>
							<span class="music_duration">时长</span>
						</div>
						<musiclist v-if="musicList" :musiclist = "musicList" showdelicon="false" @init = "initMusic"></musiclist>
					</div>
					<div class="content-info suggest-info" :class="selectIndex === 1 ? 'active': ''">
						<li v-if="suggestList" v-for="item in suggestList">你在 {{item.suggestTime.split(' ')[0]}}日 给作者留言 # {{item.suggestContent}} #</li>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>
<script>
	import {Storage} from 'common/js/Storage.js'
	import musiclist from 'components/common/musiclist/musiclist.vue'
	import store from 'store'
	import musicApi from 'components/music/music.js'
	import {fecthPromise, todoUserInfo} from 'common/api/user.js'
	export default {
		data () {
			return {
				selectIndex: 0,
				personalCenter: [],
				suggestList: []
			}
		},
		components: {
			musiclist
		},
		methods: {
			loginout () {
				Storage.deleteCookie('c_user_info')
				this.$router.push('/home')
			},
			selectSettingIndex (e) {
				this.selectIndex = parseInt(e.target.getAttribute('data-index'))
			},
			initMusic () {
				// 获取本地音乐
				// musicApi.getAlbum(this.params.id)
				musicApi.getLocalMusic.call(this)
			},
			initData () {
				todoUserInfo().then((res) => {
					const fecthUrl = 'http://www.daiwei.org/vue/server/user.php?inAjax=1&do=personalCenter'
					fecthPromise(fecthUrl, {
						userid: store.getters.getUserInfo.id || 0
					}).then((res) => {
						this.personalCenter = res.data.data
						this.suggestList = res.data.suggestList
						console.log(this.personalCenter)
					}, (err) => {
						this.$msg(err)
					})
				}, (err) => {
					this.$msg(err)
				})
			}
		},
		computed: {
			musicList () {
				return store.getters.getMusicCollectList
			}
		},
		mounted () {
			this.$nextTick(() => {
				this.initMusic()
				this.initData()
			})
		}
	}
</script>
<style lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/global.styl'
@import '~common/stylus/custom_input.styl'
$avatar_w = 80px
$c_max_w = 1040px
// .user-info
// 	width: 100%
// 	height: 100%
// 	display: flex
// 	align-items: center
// 	justify-content:center
// 	flex-direction: column
// 	p
// 		text-align:center
// 		font-weight: 400
// 		font-size: 24px
// 		color: $text_color
// 		letter-spacing: 10px

.user-info
	position: fixed
	top: 0
	left: 0
	right:0
	bottom: 0
	.loginout
		font-size: 14px
		margin-top: 2px
		color: #fff
		text-decoration: underline
		cursor:pointer
	.info-content
		overflow:auto
		position: absolute
		top: 80px
		left: 30px
		right: 30px
		padding: 0 20px
		bottom:50px
		.user-basic-info
			display:flex
			align-items: flex-start
			.user-basic-l
				display:flex
				align-items: flex-start
				flex: 1 1 auto
				.user-avatar
					flex: 0 0 $avatar_w
					width: $avatar_w
					height:$avatar_w
					border-radius: 50%
					overflow: hidden
					position: relative
					border: 5px solid $mask_color_opacity
					.image-avatar
						width: 100%
						height: 100%
					.change-avatar
						position: absolute
						top: 0
						left:0
						right:0
						bottom:0
						opacity: 0
						visibility: hidden
						transition: all 0.3s ease
						display: flex
						align-items:center
						justify-content:center
						background: $mask_color_opacity
						&:before
							content: '更换图片'
							color: $text_color
							position:relative
							font-size:12px
								
					&:hover
						.change-avatar
							opacity: 1
							visibility: visible
				.user-basic
					flex: 1 1 auto
					overflow: hidden
					padding: 0 0 0 10px
					box-sizing:border-box
					color: $text_color
					.user-nickname
						// color: $text_color
						font-size: 16px
						margin:0 auto
					.user-disc
						font-size: 12px
						// color: $text_color
						line-height: 1.5
						margin:0 auto
			.user-basic-r
				flex: 0 0 300px
				display: flex
				padding: 5px
				box-sizing:border-box
				justify-content: flex-end
				.icon-setinfo
					color: $text_color
					font-size: 14px
					box-sizing:border-box
					padding:0 10px
					border-right: 1px solid $border_bottom_color
					&:last-child
						border:none
						padding-right:0
					&.need-click
						&:hover
							text-decoration:underline
							cursor:pointer
					p
						font-size: 12px
			@media screen and (max-width: 525px)
				&
					display:block
					.user-basic-r
						margin: 10px 0
						justify-content:flex-start		
					
		@media screen and (max-width: 768px)
			&
				left: 0
				right: 0
				padding: 15px
				box-sizing: border-box
		@media screen and (min-width: $c_max_w)
			&
				width: $c_max_w
				left: 50%
				right: unset
				box-sizing: border-box
				transform: translate(-50%, 0)		
		.user-detail-info
			width: 100%
			.detail-title
				width:100%
				line-height:36px
				height:36px
				font-size:0
				border-bottom: 1px solid $border_bottom_color_deep
				padding:0
				.title-list
					display:inline-block
					font-size: 14px
					margin: 0 10px 0 0
					color: $text_color
					box-sizing:border-box 
					&.active
						color: $text_color_active
						border-bottom: 1px solid $border_color		
			.detail-content
				position:relative
				height: 400px
				.content-info
					color: $text_color
					display:none
					&.suggest-info
						padding: 15px 10px
					.music-list
						height:calc(100% - 60px)
						box-sizing:border-box
					&.active
						display:block
		.mode-info
			height:200px
</style>