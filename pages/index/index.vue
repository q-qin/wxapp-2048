<template>
	<view class="content">
		<scroll-view scroll-y class='container'>
			<view class="action_cavas" @touchstart="tapStart" @touchmove="tapMove" @touchend="tapEnd">
				<view class="score">
					<view class="title">
						2048
					</view>
					<view class="right">
						<view class="scoredetail ">
							<view class="scoredesc">历史最高</view>
							<view class="scorenumber">{{maxscore}}</view>
						</view>
						<view class="others">
							<view class="menu" @tap="menu">菜单</view>
							<view class="newgame" @tap="newGame">新游戏</view>
						</view>
					</view>
				</view>

				<view class="bc_cavas">
					<view class="bc" v-for="(row,index) in numbers" :key="index">
						<view v-for="(item,c_index) in row" :class="'bc_ bc_'+item" :key="c_index"> {{item}} </view>
					</view>
				</view>
			</view>
			<view class="intro">👉 规则：⬆⬇⬅➡滑动手指使得某个数字累加到2048~</view>
		</scroll-view>
		<view class="mask" v-show="menuShow">
			<view class="bg-white">
				<button class="btn" @tap="menuShow = false">继续游戏</button>
				<button class="btn" open-type="getUserInfo" @getuserinfo="getuserinfo">世界排名</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				score: 0,
				maxscore: 0,
				startx: 0,
				starty: 0,
				endx: 0,
				endy: 0,
				direction: '',
				numbers: [
					[2, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0],
					[0, 0, 0, 0]
				],
				menuShow: false,
			}
		},
		onLoad() {
			// 页面初始化 options为页面跳转所带来的参数
			var maxscore = wx.getStorageSync('maxscore')
			if (!maxscore) maxscore = 0
			this.maxscore = maxscore;
		},
		onShareAppMessage() {
			let peoples = (Math.random() * (99 - 60 + 1) | 0) + 60;
			return {
				title: `我已战胜全国${peoples}%的玩家，你行吗？`,
				path: "/index"
			}
		},
		methods: {
			menu(){
				this.menuShow = true;
			},
			getuserinfo(e){
				if (e.detail.userInfo) {
					console.log('getuserinfo=======>', e.detail.userInfo)
					uni.showModal({
						showCancel:false,
						content:'开发中，敬请期待~',
						success: (res) => {
							if(res.confirm){
								this.menuShow = false;
							}
						}
					})
				}else{
					uni.showToast({
						title:'需要授权才能获得完整权限',
						icon:'none'
					})
				}
			},
			// 记录得分
			storeScore() {
				if (this.maxscore < this.score) {
					this.maxscore = this.score;
					wx.setStorageSync('maxscore', this.maxscore);
				}
			},
			tapStart(event) {
				this.startx = event.touches[0].pageX;
				this.starty = event.touches[0].pageY;
			},
			tapMove(event) {
				this.endx = event.touches[0].pageX;
				this.endy = event.touches[0].pageY;
			},
			tapEnd(event) {
				var heng = (this.endx) ? (this.endx - this.startx) : 0;
				var shu = (this.endy) ? (this.endy - this.starty) : 0;
				if (Math.abs(heng) > 5 || Math.abs(shu) > 5) {
					var direction = (Math.abs(heng) > Math.abs(shu)) ? this.computeDir(1, heng) : this.computeDir(0, shu);
					this.startx = 0;
					this.starty = 0;
					this.endx = 0;
					this.endy = 0;
					this.mergeAll(direction) && this.randInsert();
				}
			},
			computeDir(heng, num) {
				if (heng) return (num > 0) ? 'right' : 'left';
				return (num > 0) ? 'bottom' : 'top';
			},
			mergeAll(dir) {
				this.checkGame();
				switch (dir) {
					case 'left':
						return this.mergeleft();
					case 'right':
						return this.mergeright();
					case 'top':
						return this.mergetop();
					case 'bottom':
						return this.mergebottom();
					default:
				}
			},
			newGame(){
				// 游戏结束
				uni.showModal({
					content: '开始新的游戏？',
					success: (res) => {
						if (res.confirm) {
							this.score = 0;
							this.numbers = [
								[2, 0, 0, 0],
								[0, 0, 0, 0],
								[0, 0, 0, 0],
								[0, 0, 0, 0]
							];
						}
					}
				})
			},
			checkGame() {
				var arr = this.numbers
				for (var i = 0; i < 4; i++) {
					for (var j = 0; j < 4; j++) {
						if (arr[i][j] == 0) return;
					}
				}
				for (var i = 0; i < 3; i++) {
					for (var j = 0; j < 3; j++) {
						if (arr[i][j] == arr[i + 1][j] || arr[i][j] == arr[i][j + 1]) return;
					}
				}

				for (var j = 0; j < 3; j++) {
					if (arr[3][j] == arr[3][j + 1]) return;
					if (arr[j][3] == arr[j + 1][3]) return;
				}
				// 游戏结束
				uni.showModal({
					content: '游戏结束，是否重新开始？',
					success: (res) => {
						if (res.confirm) {
							this.score = 0;
							this.numbers = [
								[2, 0, 0, 0],
								[0, 0, 0, 0],
								[0, 0, 0, 0],
								[0, 0, 0, 0]
							];
						}
					}
				})
			},
			//左划
			mergeleft() {
				var change = false;
				var arr = this.numbers;

				for (var i = 0; i < 4; i++) {
					//merge first
					for (var j = 0; j < 3; j++) {
						if (arr[i][j] == 0) continue;
						for (var k = 1; k < 4 - j; k++) {
							if (arr[i][j] != 0 && arr[i][j + k] != 0) {
								if (arr[i][j] != arr[i][j + k]) break; //不相同则直接跳过
								arr[i][j] = arr[i][j] * 2;
								arr[i][j + k] = 0;
								change = true;
								if (this.score < arr[i][j]) {
									this.score = arr[i][j];
								}
								break;
							}
						}
					}
					//movemove
					for (var j = 0; j < 3; j++) {
						if (arr[i][j] == 0) {
							for (var k = 1; k < 4 - j; k++) {
								if (arr[i][j + k] != 0) {
									arr[i][j] = arr[i][j + k];
									arr[i][j + k] = 0;
									change = true;
									break;
								}
							}
						}
					}
				}
				this.numbers = arr;
				this.storeScore()
				return change;
			},
			// 右滑
			mergeright() {
				var change = false
				var arr = this.numbers;

				for (var i = 0; i < 4; i++) {
					//merge first
					for (var j = 3; j > 0; j--) {
						if (arr[i][j] == 0) continue;
						for (var k = 1; k <= j; k++) {
							if (arr[i][j] != 0 && arr[i][j - k] != 0) {
								if (arr[i][j] != arr[i][j - k]) break;
								arr[i][j] = arr[i][j] * 2;
								arr[i][j - k] = 0;
								change = true;
								if (this.score < arr[i][j]) {
									this.score = arr[i][j];
								}
								break;
							}
						}
					}
					//movemove
					for (var j = 3; j > 0; j--) {
						if (arr[i][j] == 0) {
							for (var k = 1; k <= j; k++) {
								if (arr[i][j - k] != 0) {
									arr[i][j] = arr[i][j - k];
									arr[i][j - k] = 0;
									change = true;
									break;
								}
							}
						}
					}
				}
				this.numbers = arr
				this.storeScore();
				return change
			},
			// 下划
			mergebottom() {
				var change = false
				var arr = this.numbers;

				for (var i = 0; i < 4; i++) {
					//merge first
					for (var j = 3; j > 0; j--) {
						if (arr[j][i] == 0) continue;
						for (var k = 1; k <= j; k++) {
							if (arr[j][i] != 0 && arr[j - k][i] != 0) {
								if (arr[j][i] != arr[j - k][i]) break;
								arr[j][i] = arr[j][i] * 2;
								arr[j - k][i] = 0;
								change = true
								if (this.score < arr[j][i]) {
									this.score = arr[j][i];
								}
								break;
							}
						}
					}
					//movemove
					for (var j = 3; j > 0; j--) {
						if (arr[j][i] == 0) {
							for (var k = 1; k <= j; k++) {
								if (arr[j - k][i] != 0) {
									arr[j][i] = arr[j - k][i];
									arr[j - k][i] = 0;
									change = true
									break;
								}
							}
						}
					}
				}
				this.numbers = arr;
				this.storeScore()
				return change
			},
			// 上滑
			mergetop() {
				var change = false
				var arr = this.numbers;

				for (var i = 0; i < 4; i++) {
					//merge first
					for (var j = 0; j < 3; j++) {
						if (arr[j][i] == 0) continue;
						for (var k = 1; k < 4 - j; k++) {
							if (arr[j][i] != 0 && arr[j + k][i] != 0) {
								if (arr[j][i] != arr[j + k][i]) break;
								arr[j][i] = arr[j][i] * 2;
								arr[j + k][i] = 0;
								change = true
								if (this.score < arr[j][i]) {
									this.score = arr[j][i];
								}
								break;
							}
						}
					}
					//movemove
					for (var j = 0; j < 3; j++) {
						if (arr[j][i] == 0) {
							for (var k = 1; k < 4 - j; k++) {
								if (arr[j + k][i] != 0) {
									arr[j][i] = arr[j + k][i];
									arr[j + k][i] = 0;
									change = true
									break;
								}
							}
						}
					}
				}
				this.numbers = arr;
				this.storeScore()
				return change
			},
			// 随机插入
			randInsert() {
				var arr = this.numbers
				//随机2或4
				var num = Math.random() < 0.8 ? 2 : 4;
				console.log('num===>',num)
				//计算随机位置
				var zeros = [];
				for (var i = 0; i < 4; i++) {
					for (var j = 0; j < 4; j++) {
						if (arr[i][j] == 0) {
							zeros.push([i, j]);
						}
					}
				}
				var position = zeros[Math.floor(Math.random() * zeros.length)];
				arr[position[0]][position[1]] = num
				this.numbers = arr;
				console.log(arr)
			},
		},
	}
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.score {
		display: flex;
	}

	.title {
		width: 250rpx;
		height: 250rpx;
		line-height: 250rpx;
		background: #eec22e;
		margin: 80rpx 20rpx 40rpx 40rpx;
		text-align: center;
		font-size: 75rpx;
		color: white;
		border-radius: 10rpx;
	}
	.right{
		flex:1;
		height: 250rpx;
		margin: 80rpx 40rpx 0 0;
	}

	.scoredetail {
		flex: 1;
		height: 150rpx;
		background: #ccc0b2;
		text-align: center;
		border-radius: 10rpx;
	}

	.scoredetail:last-child {
		margin-right: 40rpx;
	}

	.scoredesc {
		color: #eee4da;
		font-size: 40rpx;
		line-height: 80rpx;
	}

	.scorenumber {
		line-height: 70rpx;
		font-size: 50rpx;
		color: #FFFFFF;
	}
	.others{
		flex: 1;
		display: flex;
		justify-content: center;
		.menu,.newgame{
			background: #f59563;
			color: #FFFFFF;
			margin-right: 60rpx;
			height: 50rpx;
			line-height: 50rpx;
			margin-top: 50rpx;
			padding: 0 30rpx;
			display: inline-block;
			border-radius: 8rpx;
		}
		.newgame{
			margin-right: 0;
		}
	}

	.bc_ {
		height: 152rpx;
		width: 152rpx;
		margin: 6rpx 6rpx;
		line-height: 152rpx;
		text-align: center;
		font-size: 80rpx;
		color: #fff7eb;
		border-radius: 10rpx;
	}

	.bc_0 {
		color: #ccc0b2;
		background: #ccc0b2;
	}

	.bc_2 {
		color: #7c736a;
		background: #eee4da;
	}

	.bc_4 {
		color: #7c736a;
		background: #ece0c8;
	}

	.bc_8 {
		color: #fff7eb;
		background: #f2b179;
	}

	.bc_16 {
		color: #fff7eb;
		background: #f59563;
	}

	.bc_32 {
		color: #fff7eb;
		background: #f57c5f;
	}

	.bc_64 {
		color: #fff7eb;
		background: #f65d3b;
	}

	.bc_128 {
		color: #fff7eb;
		background: #edce71;
	}

	.bc_256 {
		color: #fff7eb;
		background: #edcc61;
	}

	.bc_512 {
		color: #fff7eb;
		background: #ecc850;
	}

	.bc_1024 {
		color: #fff7eb;
		background: #edc53f;
	}

	.bc_2048 {
		color: #fff7eb;
		background: #eec22e;
	}

	.bc {
		display: flex;
	}

	.bc_cavas {
		display: flex;
		height: 670rpx;
		background-color: #b8af9e;
		justify-content: center;
		align-content: center;
		flex-wrap: wrap;
		margin: 10rpx 40rpx;
		border-radius: 10rpx;
	}

	.action_cavas {
		width: 100%;
		height: 100%;
	}

	.intro {
		display: block;
		margin: 20rpx 60rpx 0;
		font-size: small;
		color: #666;
		justify-content: flex-end;
	}
	.mask{
		position: fixed;
		overflow: hidden;
		left: 0;
		right: 0;
		top: 0;
		bottom: 0;
		background: rgba(0,0,0,.5);
		.bg-white{
			background: #FFFFFF;
			padding: 50rpx;
			border-radius: 20rpx;
			width: 500rpx;
			position: absolute;
			top: 50%;
			left: 0;
			right: 0;
			margin: auto;
			transform:translateY(-50%);
			button::after{
				border: none;
			}
			.btn{
				background: #F5F5F5;
				width: 500rpx;
				line-height: 100rpx;
				margin-top: 20rpx;
				text-align: center;
				border-radius: 10rpx;
				color: #666;
				border: none;
				&:active{
					background: #F1F1F1;
				}
			}
		}
	}
</style>
