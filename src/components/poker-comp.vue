<template>
	<div class="poker-main" :class='[mainPanelClass]' ref="mainPanelRef">
		<audio src="./static/audio/_1.m4a" ref="audioRef1"/>
		<audio src="./static/audio/_2.m4a" ref="audioRef2"/>
		<audio src="./static/audio/_3.m4a" ref="audioRef3"/>
		<audio src="./static/audio/_4.m4a" ref="audioRef4"/>
		<audio src="./static/audio/_5.m4a" ref="audioRef5"/>
		<audio src="./static/audio/_6.m4a" ref="audioRef6"/>
		<audio src="./static/audio/_7.m4a" ref="audioRef7"/>
		<audio src="./static/audio/_8.m4a" ref="audioRef8"/>
		<audio src="./static/audio/_9.m4a" ref="audioRef9"/>
		<audio src="./static/audio/_10.m4a" ref="audioRef10"/>
		<audio src="./static/audio/_11.m4a" ref="audioRef11"/>
		<audio src="./static/audio/_12.m4a" ref="audioRef12"/>
		<audio src="./static/audio/_13.m4a" ref="audioRef13"/>
		<audio src="./static/audio/_14.m4a" ref="audioRef14"/>
		<audio src="./static/audio/_plus.m4a" ref="audioRef15"/>
		
		<div v-for='(card, idx) of cards' :key='card.index' class="card" :style='[cardStyle(card, idx)]' @click='cardClicked(card)'></div>
		<div class="red-picked-value" :class='{show: redPickedValue>0 && redCanPick}'>{{redPickedValue}}</div>
		<div class="red-value">{{redValue}}</div>
		<div class="blue-value">{{blueValue}}</div>
		<div class="card-left">{{cards.length - cardIndex}}</div>
		<div class="control-bar">
			<div class="button" @click='pickCard' :class='{disabled: !redCanPick || pickedSum2 !== 14}'>捡牌</div>
			<div class="button" @click='dropCard' :class='{disabled: !redCanDrop || cardCntInRed2 !== 1}'>弃牌</div>
		</div>
		
		<div class="welcome" @click='show.welcome = false; newGame()'> 点击屏幕开始游戏</div>
	</div>
</template>

<script>
	const typeValueMap = {
		0: 2,
		1: 4,
		2: 3,
		3: 1,
		4: 5,
	}
	
	const fapaiWait = 800
	const createCards = (part=Math.random()) => {
		const arr = []
		
		arr.push({ // 小王
			type: 4,
			num: 0,
			part
		})
		arr.push({ // 大王
			type: 4,
			num: 1,
			part
		})
		
		for (let type=0; type<4; type++) {
			for (let num=0; num<13; num++) {
				arr.push({
					type,
					num,
					part
				})
			}
		}
		
		return arr;
	}
	
	const mess = arr => {
		let i = arr.length - 1
		while (i>0) {
			const j = Math.floor(Math.random() * i)
			let tmp = arr[i]
			arr[i] = arr[j]
			arr[j] = tmp
			i--
		}
		for (let idx=0; idx<arr.length; idx++) {
			arr[idx].index = idx
		}
		return arr
	}
	
	
	const indexOf = (card, arr) => {
		for (let idx=0; idx<arr.length; idx++) {
			const cardInArr = arr[idx]
			if (cardInArr) {
				if (cardInArr.type === card.type && cardInArr.num === card.num && cardInArr.part === card.part) {
					return idx
				}
			}
			
		}
		return -1
	}
	
	const wait = (num=fapaiWait) => {
		return new Promise(res => {
			setTimeout(res, num)
		})
	}
	
	const takeOut = (arr1, arr2) => {
		const arr = []
		for (const e1 of arr1) {
			const found = arr2.find(e2 => {
				return e1.index === e2.index
			})
			if (!found) {
				arr.push(e1)
			}
		}
		return arr
	}
	
	export default {
		props: {
			size: Object
		},
		data() {
			return {
				show: {
					welcome: true
				},
				cards: [],   // 剩余可用的牌
				cardIndex: 0, // 当前拿到第几张牌了
				
				blue: [],     // 电脑手里的牌\
				blue2: [],   // 蓝方当前选择
				bluePicked: [], // 蓝方已经捡起并得分的牌
				
				ground: [],    // 场上可以捡的牌
				picked: false, // 在场上捡的那个牌
				
				red: [],      // 我方手里的牌
				red2: [], // 记录我方当前选择的牌
				red2Idx: 0, // 如果添加需要添加到哪里
				redPicked: [], // 红方已经捡走的14
				
				redCanPick: false,  // 我方可以操作,
				redCanDrop: false,
			}
		},
		mounted() {
			const {
				$refs: refs
			} = this
			refs.mainPanelRef.style.setProperty('--main-height', `${this.heightComp}px`)
			refs.mainPanelRef.style.setProperty('--main-width', `${this.widthComp}px`)
			refs.mainPanelRef.style.setProperty('--card-width', `${this.cardWidth}px`)
			refs.mainPanelRef.style.setProperty('--card-height', `${this.cardHeight}px`)

			this.audio = [
				refs.audioRef1,
				refs.audioRef2,
				refs.audioRef3,
				refs.audioRef4,
				refs.audioRef5,
				refs.audioRef6,
				refs.audioRef7,
				refs.audioRef8,
				refs.audioRef9,
				refs.audioRef10,
				refs.audioRef11,
				refs.audioRef12,
				refs.audioRef13,
				refs.audioRef14,
				refs.audioRef15,
			] 
			// https://img.tukuppt.com/newpreview_music/09/03/80/5c8ae2d24455276952.mp3
		},
		
		computed: {
			widthComp() {
				const {
					size
				} = this
				return size.width
			},
			heightComp() {
				const {
					size
				} = this
				return size.height
			},
			cardWidth() {
				return Math.floor(this.widthComp / 10)
			},
			cardHeight() {
				return Math.floor(this.cardWidth / 0.618)
			},
			groundSort() {
				const arr = JSON.parse(JSON.stringify(this.ground))
				arr.sort((a, b) => {
					return b.index - a.index
				})
				return arr;
			},
			cardCntInRed2() {
				let cnt = 0;
				for (const card of this.red2) {
					if (card) {
						cnt++
					}
				}
				return cnt
			},
			mainPanelClass() {
				const {
					welcome
				} = this.show
				
				const arr = []
				if (welcome) {
					arr.push('show-welcome')
				}
				return arr
			},
			pickedSum() { 
				const that = this
				const sum = this.red2.reduce((acc, cc) => {
					return acc + that.getCardValue((cc))
				}, 0)
				return sum
			},
			pickedSum2() { // 我们目前点选的牌的分
				const {
					pickedSum,
					picked
				} = this
				let num = 0;
				if (picked) {
					num = this.getCardValue(picked)
				}
				return num + pickedSum
			},
			redPickedValue() {
				const{
					red2,
					picked
				} = this
				
				let value = 0
				for (const cc of red2) {
					if (cc) {
						value += typeValueMap[cc.type]
					}
				}
				if (picked) {
					value += typeValueMap[picked.type]
				}
				return value
			},
			redValue() {
				const that = this
				const {
					redPicked
				} = that
				const acc = redPicked.reduce((acc, card) => {
					if (card) {
						return acc + typeValueMap[card.type]
					}
					return acc
				}, 0)
				return acc
			},
			blueValue() {
				const that = this
				const {
					bluePicked
				} = that
				const acc = bluePicked.reduce((acc, card) => {
					if (card) {
						return acc + typeValueMap[card.type]
					}
					return acc
				}, 0)
				return acc
			}
		},
		
		watch: {
			pickedSum2(value) {
				if (this.redCanPick) {  // 红方点选的时候，蓝方的时候不清理
					if (value !== 14) {
						this.picked = false
					}
				}
			}
		},
		
		methods: {
			getNextCard() {
				const {
					cards,
				} = this
				const rv = cards[this.cardIndex]
				this.cardIndex++
				return rv
			},
			async dropCard() {
				const that = this
				if (!that.redCanDrop) {
					return false
				}
				if (that.cardCntInRed2 !== 1) {
					// that.du.play();
					return false
				}
				that.redCanDrop = false
				that.redCanPick = false
				
				
				
				
				
				let dropCard = that.red2[0]
				if (!dropCard) {
					dropCard = that.red2[1]
				}
				if (dropCard) {
					that.red = takeOut(that.red, [dropCard])
					that.ground.push(dropCard)
					that.red2 = []
				}
				
				for (let idx=that.red.length; idx<4; idx++) { // 补齐到4张
					that.red.push(that.getNextCard())
					await wait()
				}
				
				// 该电脑出牌了
				const {
					blue,
					ground
				} = this
				
				const cardGroundMap = {} //  根据牌的value 放到 性能优化map中
				for (const cardInGround of ground) {
					const groundCardValue = that.getCardValue(cardInGround)
					cardGroundMap[groundCardValue] = cardGroundMap[groundCardValue] || []
					cardGroundMap[groundCardValue].push(cardInGround)
				}
				
				const matched = [] // 所有匹配的方案
				
				const getHitObj = (value, map) => {
					const hitArr = map[14 - value]
					if (hitArr && hitArr.length > 0) {
						hitArr.sort((c1, c2) => {
							return  typeValueMap[c2.type] - typeValueMap[c1.type]
						})
						return hitArr[0]
					}
					return false
				}
				
				for (const cardInBlue of blue) { // 单张匹配
					const value = that.getCardValue(cardInBlue)
					const picked = getHitObj(value, cardGroundMap)
					if (picked) {
						matched.push({
							blue: [cardInBlue],
							picked
						})
					}
				}
				
				for (let idx1=0; idx1<blue.length - 1; idx1++) { // 两张加一张的匹配
					for (let idx2=idx1+1; idx2<blue.length; idx2++) {
						const b1 = blue[idx1]
						const b2 = blue[idx2]
						const value = that.getCardValue(b1) + that.getCardValue(b2)
						const picked = getHitObj(value, cardGroundMap)
						if (picked) {
							matched.push({
								blue: [b1, b2],
								picked
							})
						}
					}
				}
				
				if (matched.length > 0) { // 存在匹配的额方案
					const mappedValue = matched.map(item => {
						const {
							blue,
							picked
						} = item
						const value = blue.reduce((acc, cc) => {
							return acc + typeValueMap[cc.type]
						}, 0) + typeValueMap[picked.type]
						return {
							blue,
							picked,
							value
						};
					})
					
					mappedValue.sort((m1, m2) => {
						return m2.value - m1.value
					})
					
					console.log(mappedValue, '所有方案')
					
					const pickedBlue = mappedValue[0] //积分最高的那个方案
					
					for (const bb of pickedBlue.blue) {
						that.blue2.push(bb)
						// that.audio[that.getCardValue(bb)].play()
						await wait()
						// that.audio[14].play()
						await wait()
					}
					// that.audio[that.getCardValue(pickedBlue.picked)].play()
					that.picked = pickedBlue.picked
					
					await wait(2000) // 等待用户观察一下
					
					for (const bb of that.blue2) { // 去积分
						that.bluePicked.push(bb)
					}
					that.bluePicked.push(that.picked)// 去积分
					that.blue = takeOut(that.blue, that.blue2)
					that.blue2 = []
					that.ground = takeOut(that.ground, [that.picked])
				}
				
				
				
				

				
				
				for (let idx=that.blue.length; idx<5; idx++) {
					that.blue.push(that.getNextCard())
					await wait();
				}
				
				const toGround = that.blue[Math.floor(Math.random()*that.blue.length)] // 随机丢弃一张
				that.blue2.push(toGround)
				await wait()
				that.blue = takeOut(that.blue, [toGround])
				
				that.ground.push(toGround)
				that.blue2 = []
				that.picked = false
				
				that.redCanDrop = true
				that.redCanPick = true
				
			},
			async pickCard() {
				const {
					picked,
					red2
				} = this
				if (picked) {
					this.redPicked.push(picked)
					for (const card of red2) {
						if(card) {
							this.redPicked.push(card)
						}
					}
					const newRed = []
					for(const cardInRed of this.red) {
						const found = this.red2.find(cc => {
							return cc && cc.index === cardInRed.index
						})
						if (!found) {
							newRed.push(cardInRed)
						}
					}
					this.red = newRed;
					
					const newGround = []
					for (const cardInGround of this.ground) {
						if(cardInGround.index !== picked.index) {
							newGround.push(cardInGround)
						}
					}
					this.ground = newGround
					
					this.picked = false
					this.red2 = []
					
					
					
					for (let idx=this.red.length; idx<5; idx++) {
						await wait();
						this.red.push(this.getNextCard())
					}
					
					this.redCanPick = false
				}
			},
			
			getCardValue(card) {
				let num = card.num + 1
				if (card.type === 4) {
					num = 5
				}
				return num
			},
			
			cardClicked(card) {
				const that = this
				if (!that.redCanPick && !that.redCanDrop) { // 用户当前不能操作
					return false
				}
				const groundIdx = indexOf(card, this.ground)
				if (!(indexOf(card, that.red) >=0 || groundIdx >= 0)) { // 只能点我方的牌, 和场上的牌
					return false
				}
				
				if (groundIdx >= 0) {
					if (that.getCardValue(card) + this.pickedSum  === 14) {
						that.picked = card
					}
				} else {
					const redIdx = indexOf(card, that.red2)
					if (that.redCanPick) {
						if(redIdx >= 0) { // 已经存在
							const arr = []
							for (let idx=0; idx<that.red2.length; idx++) {
								
								if (idx != redIdx && that.red2[idx]) {
									arr.push(that.red2[idx])
								}
							}
							that.red2 = arr
							that.red2Idx = 1
						} else {
							const arr = JSON.parse(JSON.stringify(that.red2))
							arr[that.red2Idx] = card
							that.red2 = arr
							that.red2Idx++;
							that.red2Idx = that.red2Idx % 2
						}
					} else {
						this.red2 = [card]
					}
					
				}
			},
			newGame() {
				const that = this
				that.cards = [...createCards(1), ...createCards(2)]
				mess([])
				that.red = []
				that.blue = []
				that.ground = []
				that.cardIndex = 0
				
				
				setTimeout(() => {
					const fapaile = async () => {
						
						for (let idx=0; idx<4; idx++) {
							await wait(fapaiWait)
							that.red.push(that.getNextCard())
							await wait(fapaiWait)
							that.blue.push(that.getNextCard())
						}
						
						for (let idx=0; idx<4; idx++) {
							await wait(fapaiWait)
							that.ground.push(that.getNextCard())
						}
						that.redCanPick = true
						that.redCanDrop = true
					}
					fapaile();
					
					
				}, 100)
			},
			
			cardStyle(card, idx) {
				const that = this
				const {
					red,
					blue,
					ground,
					cardWidth: cardWidthBase
				} = that
				
				const cardWidth = cardWidthBase * 1.1

				let top = this.heightComp / 3
				let left = this.widthComp / 10 * 8.5
				
				let leftOffset = 10
				
				let found = false
				const redIdx = indexOf(card, red)
				if (redIdx>=0) {
					top = this.heightComp / 4 * 3
					left = leftOffset + redIdx * cardWidth
					found = true
					if (indexOf(card, that.red2) >= 0) {
						top = this.heightComp / 4 * 3 - this.cardHeight * .2
					}
				}
				
				const blueIdx = indexOf(card, blue)
				if (blueIdx>=0) {
					top = 10
					left = leftOffset + blueIdx * cardWidth 
					found = true
					if (indexOf(card, that.blue2) >=0 ) {
						top = 60
					}
				}
				
				const groundIdx = indexOf(card, ground)
				let cardInGround = false
				if (groundIdx >= 0) {
					found = true
					const columnCount = 4
					const rowIdx = Math.floor(groundIdx / columnCount)
					const colIdx = groundIdx %  columnCount
					
					top = 400 + rowIdx * cardWidth
					left = leftOffset + colIdx * cardWidth
					if(that.picked) {
						if (that.picked.index === card.index) {
							top -= 50
						}
					}
					cardInGround = true
				}
				
				if(indexOf(card, that.redPicked) >= 0) { // 红方已经捡走得分的牌
					top = 2100
					left = 900
				}
				
				if(indexOf(card, that.bluePicked) >= 0) {
					top = -300
					left = - 800
					
				}
				
				
				const style = { 
					'background-position': found ? `${118 * 69 / this.cardWidth + card.num * that.cardWidth}px ${514 - card.type * that.cardHeight}px` : '980px 200px',
					top: `${top}px`,
					left: `${left}px`,
					'z-index': 10001 + idx
				}
				
				if (cardInGround && that.redCanPick) {
					const {
						red2
					} = that
					if (red2.length > 0) {
						if (that.getCardValue(card) + that.pickedSum  !== 14) {
							style.filter = 'brightness(.5)'
						}
					}
				}
				
				if(redIdx >= 0) {
					if (!that.redCanDrop && !that.redCanPick) {
						style.filter = 'brightness(.5)'
					}
				}
				
				if(that.picked) {
					if (that.picked.index === card.index) {
						style.transform = 'scale(1.1)'
						style['z-index'] = 99999999
					}
				}
				
				return style
			}
		}
	}
</script>

<style>
	.poker-main {
		--main-height: 100%;
		--main-width: 100%;
		height: var(--main-height);
		width: var(--main-width);
		
		--card-width: calc(var(--main-width) / 10);
		--card-height: calc(var(--card-width) / 0.618);

		background: hsl(120, 100%, 30%);
		z-index: 10000;
		overflow: hidden;
		position: relative;
	}

	.card {

		background-image: url('./poker.jpg');
		width: var(--card-width);
		height: var(--card-height);
		background-size: calc(1000px * var(--card-width) / 69) calc(514px * var(--card-width) / 69);
		display: inline-block;
		margin: 1px;
		border-radius: 4px;
		position: absolute;
		transition: top 1s, left 1s;
		font-size: 80px;
		color: red;
	}
	
	.welcome {
		height: 0;
		width: 100%;
		background: white;
		position: absolute;
		background: linear-gradient(black, lightgray, black);
		display: flex;
		align-items: center;
		overflow: hidden;
		justify-content: center;
		opacity: 0;
		font-size: calc(var(--main-width) * .08);
		cursor: pointer;
	}
	
	.show-welcome > .welcome{
		top: 0;
		height: 100%;
		opacity: 1;
	}
	
	.control-bar {
		position: absolute;
		width: 90%;
		bottom: 20px;
		display: flex;
		justify-content: flex-start;
	}
	
	.control-bar > .button {
		padding: 20px;
		border: 1px solid gray;
		--c2: hsl(220, 100%, 66%);
		--c1: hsl(240, 100%, 66%);
		background: linear-gradient(var(--c1), var(--c2), var(--c1));
		color: white;
		border-radius: 20px;
		margin: 10px;
		min-width: 100px;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-right: 40px;
	}
	
	.button.disabled {
		--c2: hsl(220, 100%, 76%);
		--c1: hsl(240, 100%, 76%);
	}
	
	.red-picked-value.show {
		opacity: 1;
	}
	.red-picked-value {
		position: absolute;
		bottom: 50px;
		right: 200px;
		font-size: calc(var(--main-width) * .05);
		color: white;
		font-weight: bolder;
		text-shadow: 5px 5px 5px gray;
		z-index: 30000;
		opacity: 0;
		transform: all .3s;
	}
	
	.red-picked-value:before {
		content: '+'
	}
	
	.red-value {
		position: absolute;
		bottom: 50px;
		right: 50px;
		font-size: calc(var(--main-width) * .05);
	}
	
		
	.blue-value {
		position: absolute;
		top: 10px;
		right: 50px;
		font-size: calc(var(--main-width) * .05);
	}
	
	.card-left {
		position: absolute;
		right: 10px;
		top: 600px;
		font-size: calc(var(--main-width) * .05);
	}
	.card-left:before {
		content: '剩余：'
	}
</style>
