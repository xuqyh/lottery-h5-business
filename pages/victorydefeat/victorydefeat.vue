<template>
	<view class="box">
		<cmd-nav-bar title="任选九" :back="true" font-color="#fff" background-color="#515ffb" @rightText="rightText"
			right-text="任选九开奖">
		</cmd-nav-bar>
		<div class="soccer_wrap">
			<!--main-->
			<div class="soccer_main">
				<!--混合过关-->
				<u-empty icon="http://cdn.uviewui.com/uview/empty/data.png" :show="winBurdenMatchList.length<=0"
					mode="data" text="暂无比赛"></u-empty>
				<div class="soccer_hunheguoguan">
					<!--每周比赛-->
					<div class="soccer_weekGame" v-for="(wk,weekIdx) in winBurdenMatchList" :key="weekIdx">
						<!--比赛-->
						<div class="weekGame_games">
							<!--每个比赛-->
							<div class="games" v-for="(game,index) in wk.winBurdenMatchList" :key="index">
								<div class="games_left">
									<span><b>{{game.number}}</b></span>
									<u-tag :borderColor="game.color" :bgColor="game.color"
										:text="game.match">
									</u-tag>
									<span>{{game.deadline|formatDate(that)}} 截止</span>
									<span style="color: #515ffb;" @click="analysis(game.analysis)">分析</span>
								</div>
								<div class="games_right">
									<!--比赛队伍-->
									<span class="right_title">
										<span
											class="title_team">{{game.homeTeam}}<strong>vs</strong>{{game.visitingTeam}}</span>
									</span>
									<!--表格-->
									<table>
										<tr>
											<td class="sheng_top">
												<p>
													<span style="position: relative;"
														v-for="(notLet,notLetIndex) in game.notLetOddsList"
														:class="{active:notLet.active}"
														@tap="selectBtn(game,notLet,weekIdx,index,1)">{{notLet.describe}}
														{{notLet.odds}}
													</span>
													<span v-if="game.notLetOddsList.length<=0">未开售</span>
												</p>
											</td>
											<td>
												<div :class="game.isGallbladder?'content':'content2'" @tap="gallbladderBtn(game)">
													<p>胆</p>
												</div>
											</td>
										</tr>
									</table>
								</div>
							</div>
						</div>
					</div>
				</div>
				<BuyFooterSport :count='count' :total='total' :isShowCount="isShowCount" @clean="clean"
					@confirm="confirm"></BuyFooterSport>
			</div>
		</div>

	</view>
</template>

<script>
	import {
		winBurdenWinEvenLoseList
	} from '@/api/winburden.js'
	import BuyFooterSport from '../common/buy-footer-sport.vue'
	export default {
		name: 'winBurdenWin',
		components: {
			BuyFooterSport
		},
		data() {
			return {
				isShowCount: false,
				winBurdenMatchList: [],
				that: this,
				count: "",
				total: "",
				selectItem: []
			}
		},
		filters: {
			formatDate(data, that) {
				if (null == data) {
					return null;
				}
				return that.globalUtil.dateTimeFormat1(data)
			},
		},
		methods: {
			gallbladderBtn(item){
				let count=0;
				item.notLetOddsList.map(notLet=>{
					if(notLet.active){
						count++;
					}
				})
				if(count>0){
					item.isGallbladder=!item.isGallbladder;
				}
			},
			/** 选中时改变当前单元格的背景颜色
			 * @param {Object} wk 当前比赛数据
			 * @param {Object} item 当前点击的数据
			 * @param {Object} weekIdx 上上级父级list的index
			 * @param {Object} index 上级父级list的index
			 * @param {Object} ,type 类型
			 */
			selectBtn(wk, item, weekIdx, index, type) {
				if (type == 1) {
					//不让球
					this.winBurdenMatchList[weekIdx].winBurdenMatchList[index].notLetOddsList.map(data => {
						if (item.id == data.id) {
							this.$set(data, "active", !item.active)
						}
					})
				}

			//计算选中了几项
				if (item.active) {
					wk.choiceCount++;
				} else {
					wk.choiceCount--;
				}
				//获取选中的数据放入新的数组中
				if (this.selectItem.length > 0) {
					//标识是否在数组中是否存在
					let flag = true
					this.selectItem.map((d, idx) => {
						//判断id 如果相同进行重新赋值 在将flag置为true
						if (d.id == wk.id) {
							flag = false
							//如果都取消了判断选中项是否为零，为零就删除元素
							if (d.choiceCount == 0) {
								//index 当前元素索引；1：需要删除的元素个数
								this.selectItem.splice(idx, 1);
							}
							//重新赋值数据
							d = item;
						}
					})
					//如果都不存在进行添加到数组中
					if (flag) {
						this.selectItem.push(wk)
					}
				} else {
					//第一次数组为空的时候添加到数组中
					this.selectItem.push(wk)
				}
				this.count = this.selectItem.length;
			},
			rightText() {
				uni.navigateTo({
					url: "pages/victorydefeat/victorydefeatResult"
				});
			},
			clean() {
				this.count = "";
				this.fecthSoccerData();
				this.selectItem = []
			},
			analysis(url) {
				if (url == null || url == "" || url == undefined) {
					uni.showToast({
						title: '暂无分析数据',
						icon: 'none'
					});
					return;
				}
				uni.navigateTo({
					url: "/pages/common/analysis?url=" + encodeURIComponent(url)
				});
			},
			confirm() {
				var count;
				var flag = true;
				this.selectItem.map((item) => {
					//判断是否有选择数据
					item.notLetOddsList.map(notLet => {
						if (notLet.active) {
							count = this.selectItem.length;
							//用来标记是否是单关，传到下单也有用来判断是否有那个单关的下注选项
							flag = false
							return;
						}
					})
				})
				//如果不是单关,选中了让球或者非让球就需要校验场数，比赛是最低二场
				if (count < 9 || this.selectItem.length <= 0) {
					uni.showToast({
						title: '至少选择9场比赛',
						icon: 'none'
					});
					return;
				}
				uni.navigateTo({
					url: "pages/victorydefeat/victorydefeatConfirm?obj=" + encodeURIComponent(JSON.stringify(this.selectItem))
				});
			},
			// 获取竞彩足球数据
			fecthSoccerData() {
				uni.showLoading();
				winBurdenWinEvenLoseList().then(res => {
					this.winBurdenMatchList = res.voList
					setTimeout(function() {
						uni.hideLoading();
					}, 50);
				})
			},
		},
		onLoad() {
			this.fecthSoccerData()
		},
		onPullDownRefresh() {
			this.fecthSoccerData();
			setTimeout(function() {
				uni.stopPullDownRefresh()
			}, 500);
		},
	}
</script>

<style scoped lang="scss">
	/deep/.u-tag{
		width: 45px;
		height: 20px;
		justify-content: center;
		align-items: center;
	}
	/deep/.u-tag__text--medium {
	    font-size: 11px;
	    line-height: 11px;
	}
	.content,.content2{
		width: 30px;
		height: 30px;
		background-color: #515ffb;
		border-radius: 50%;
		p{
			width: 30px;
			height: 30px;
			color: #fff;
			text-align: center;
			line-height: 30px;
			font-size: 14px;
		}
	}
	.content2{
		background-color: #D3D3D3 !important;
	}
	page {
		background-color: #f7f9fa;
	}

	/deep/.cmd-nav-bar-right-text {
		font-size: 16px !important;
	}

	/deep/.u-empty {
		margin-top: 50% !important;
	}

	.active {
		background-color: #515ffb !important;
		color: #fff !important;
	}

	.box {
		padding-bottom: 0px;
	}

	.soccer_wrap {
		.soccer_main {
			background-color: #f7f9fa;
			width: 100%;
			padding-bottom: 10px;

			/*混合过关*/
			.soccer_hunheguoguan {
				width: 100%;
				padding-bottom: 10px;

				.soccer_weekGame {
					width: 100%;

					/*比赛*/
					.weekGame_games {
						.games {
							box-sizing: border-box;
							display: flex;
							align-items: center;
							justify-content: space-evenly;
							height: 30.66667vmin;
							margin: 0 2.66667vmin;
							border-bottom: 1px solid #e6e6e6;

							/*box-sizing: border-box;*/
							.games_left {
								width: 15vmin;
								height: 100%;
								font-size: 2.93333vmin;
								font-weight: 400;
								margin-right: 2.4vmin;
								color: #333333;
								display: flex;
								flex-direction: column;
								align-items: center;
								justify-content: center;

								span {
									text-overflow: ellipsis;
									display: block;
									margin-bottom: 1.8vmin;
									white-space: nowrap;
								}
							}

							.games_right {
								/*height: 100%;*/
								box-sizing: border-box;

								.right_title {
									.title_team {
										/*margin:0 auto;*/
										font-size: 3.2vmin;
										height: 35px;
										font-weight: 700;
										color: #5d5d5d;
										width: 100%;
										display: flex;
										justify-content: center;
										align-items: center;

										strong {
											margin: 0 .66667vmin;
											font-weight: 400;
										}

										em {
											font-style: normal;
											font-size: 2.4vmin;
											font-weight: 400;
										}
									}
								}

								table {
									width: 100%;
									border: 1px solid #e6e6e6;
									color: #333333;

									tr {
										.sheng_top {
											height: 9.6vmin;
											box-sizing: border-box;
											border: 1px solid #e6e6e6;
											font-size: 3.46667vmin;
											text-align: center;
											justify-content: center;

											p {
												display: flex;
												align-items: center;
												height: 100%;
												box-sizing: border-box;
												justify-content: center;

												span {

													width: 20.5vmin;
													display: flex;
													align-items: center;
													justify-content: center;
													height: 100%;
													border-right: 1px solid #e6e6e6;
												}

												span:last-child {
													border: none;
												}
											}
										}
									}
								}
							}
						}
					}
					.games:last-child {
						border: none;
					}
				}
				.soccer_weekGame:last-child {
					margin-bottom: 13.33333vmin;
				}
			}
		}
	}
</style>