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
		
		<div v-for='(card, idx) of cards' :key='card.index' class="card" :style='[cardStyle1(card, idx)]' @click='cardClicked(card)'>
		</div>
		<div class="red-picked-value" :class='{show: redPickedValue>0 && redCanPick}'>+{{redPickedValue}}</div>
		<div class="blue-picked-value" :class="{show: bluePickedValue > 0 && picked}">+{{bluePickedValue}}</div>
		<div class="red-value">{{redValue}}</div>
		<div class="blue-value">{{blueValue}}</div>
		<div class="card-left">{{cardLeftComp}}</div>
		<div class="control-bar">
			<div class="button" @click='pickCard' :class='{disabled: !redCanPick || pickedSum2 !== 14}'>捡牌</div>
			<div class="button" @click='dropCard' :class='{disabled: !redCanDrop || cardCntInRed2 !== 1}'>弃牌</div>
		</div>
		
		<div class="welcome" @click='show.welcome = false; newGame()'>
			<div class="image">
				<div class="card" v-for="(card) of welcomeCards" :style="cardStyle2(card)" :key="card.index"></div>
			</div>
			<div class="mask">
				点击屏幕开始游戏
			</div>
		</div>

		<div class="game-over">
			<div class="go-score">得分 {{redValue}}</div>
			<div class="statistic">
				<div class="row" v-for="item of typeValueMapSorted" :key="item.type">
					<div class="card" :style="cardStyle3({type: item.type, num: 0})"></div>
					<div class="stat-text">{{item.value}} * {{getRedTypeCount(item.type)}}</div>
				</div>
			</div>
			<div class="go-text1">Game Over</div>
			<div class="go-text2">{{redValue > blueValue ? '胜出' : redValue ===  blueValue ? '平局' :'洗洗睡吧'}}</div>
		</div>
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
	
	const fapaiWait = 500
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
			const welcomeCards = createCards()
			welcomeCards.splice(0, 34)
			const {
				size
			} = this
			welcomeCards.forEach(card => {
				card.top = Math.random() * size.height
				card.left = Math.random() * size.width
				card.rotate = Math.random() * 360,
				card.speed = Math.random() * 3 + 1
			})
			return {
				welcomeCards,
				show: {
					welcome: true,
					gameover: false
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
			this.timeId = setInterval(() => {
				const arr = []
				for (const card of this.welcomeCards) {
					card.top += card.speed
					if (card.top > this.heightComp) {
						card.top = -120
					}
					arr.push(card)
				}
				this.welcomeCards = arr
			}, 10)
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
	
		umount() {
			clearInterval(this.timeId)
		},
		
		computed: {
			typeValueMapSorted() {
				const arr = []
				for (const key in typeValueMap) {
					const value = typeValueMap[key]
					arr.push({
						type: key,
						value
					})
				}
				arr.sort((a, b) => {
					return b.value - a.value
				})
				return arr
			},
			redPicked0() {
				const {
					redPicked
				} = this
				return redPicked.filter(card => card.type === 0)
			},
			redPicked1() {
				const {
					redPicked
				} = this
				return redPicked.filter(card => card.type === 1)
			},
			redPicked2() {
				const {
					redPicked
				} = this
				return redPicked.filter(card => card.type === 2)
			},
			redPicked3() {
				const {
					redPicked
				} = this
				return redPicked.filter(card => card.type === 3)
			},
			redPicked4() {
				const {
					redPicked
				} = this
				return redPicked.filter(card => card.type === 4)
			},
			

			cardLeftComp() {
				let rv = this.cards.length - this.cardIndex
				if (rv < 0) {
					rv = 0
				}
				return rv
			},
			cardOnWelcome() {
				const cards =  mess(createCards());
				return cards
			},
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
				//return Math.floor(this.widthComp / 10)
				return this.cardHeight / 102 * 68 
			},
			cardHeight() {
				//Math.floor(102 / 68 * this.cardWidth)
				return this.heightComp / 10
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
					welcome,
					gameover
				} = this.show
				
				const arr = []
				if (welcome) {
					arr.push('show-welcome')
				}
				if (gameover) {
					arr.push('show-gameover')
				}
				return arr
			},
			pickedSum() { 
				const that = this
				const sum = this.red2.reduce((acc, cc) => {
					return acc + that.getCardNumber((cc))
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
					num = this.getCardNumber(picked)
				}
				return num + pickedSum
			},
			bluePickedValue() {
				const {
					blue2,
					picked
				} = this

				let value = 0
				for (const cc of blue2) {
					if (cc) {
						value += typeValueMap[cc.type]
					}
				}
				if (picked) {
					value += typeValueMap[picked.type]
				}
				return value
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
			},
		},
		
		methods: {
			getRedTypeCount(type) {
				if (4 == type) {
					return this.redPicked4.length
				}
				if (3 == type) {
					return this.redPicked3.length
				}
				if (2 == type) {
					return this.redPicked2.length
				}
				if (1 == type) {
					return this.redPicked1.length
				}
				if (0 == type) {
					return this.redPicked0.length
				}
				
				return 0
			},
			getNextCard() {
				const {
					cards,
				} = this
				if (this.cardIndex >= cards.length) {
					return false
				}
				const rv = cards[this.cardIndex]
				this.cardIndex++
				return rv
			},
			getMatched(card, arr) {
				const that = this
				const matched = []
				for (const cc of arr) { // 单张匹配
					if (that.getCardNumber(cc) + that.getCardNumber(card)) {
						matched.push({
							arr: [cc],
							picked: card
						})
					}
				}

				for (let ii=0; ii<arr.length - 1; ii++) { // 双匹配
					for (let jj=ii+1; jj<arr.length; jj++) {
						const c1 = arr[ii]
						const c2 = arr[jj]
						if (that.getCardNumber(c1) + that.getCardNumber(c2) + that.getCardNumber(card)) {
						matched.push({
							arr: [c1, c2],
							picked: card
						})
					}
					}
				}

				return matched
			},
			getBlueDropCard() {
				const {
					blue, 
					red,
					getMatched
				} = this
				const blueSorted = JSON.parse(JSON.stringify(blue))
				blueSorted.sort((a, b) => {
					return typeValueMap[a.type] - typeValueMap[b.type]
				})
				for (const cardInSortedBlue of blueSorted) {
					const matched = getMatched(cardInSortedBlue, red)
					if (matched.length === 0) {
						return cardInSortedBlue
					}
				}
				return blueSorted[0]
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
				
				
				that.picked = false
				
				
				let dropCard = that.red2[0]
				if (!dropCard) {
					dropCard = that.red2[1]
				}
				if (dropCard) {
					that.red = takeOut(that.red, [dropCard])
					if (that.red.length === 0) { // 红方没牌了，游戏结束
						this.show.gameover = true
						return 
					}
					that.ground.push(dropCard)
					that.red2 = []
				}
				
				for (let idx=that.red.length; idx<4; idx++) { // 补齐到4张
					const nextCard = that.getNextCard()
					if(nextCard) {
						that.red.push(nextCard)
						await wait()
					}
				}
				
				// 该电脑出牌了
				const {
					blue,
					ground
				} = this
				
				const cardGroundMap = {} //  根据牌的value 放到 性能优化map中
				for (const cardInGround of ground) {
					const groundCardValue = that.getCardNumber(cardInGround)
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
					const value = that.getCardNumber(cardInBlue)
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
						const value = that.getCardNumber(b1) + that.getCardNumber(b2)
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
						// that.audio[that.getCardNumber(bb)].play()
						await wait()
						// that.audio[14].play()
						await wait()
					}
					// that.audio[that.getCardNumber(pickedBlue.picked)].play()
					that.picked = pickedBlue.picked
					
					await wait(3000) // 等待用户观察一下
					
					for (const bb of that.blue2) { // 去积分
						that.bluePicked.push(bb)
					}
					that.bluePicked.push(that.picked)// 去积分
					that.blue = takeOut(that.blue, that.blue2)
					that.blue2 = []
					that.ground = takeOut(that.ground, [that.picked])
					that.picked = false

					for (let idx=that.blue.length; idx<5; idx++) {
						const nextCard = that.getNextCard()
						if (nextCard) {
							that.blue.push(nextCard)
							await wait();
						}
					}

					const toGround = that.getBlueDropCard()
					that.blue2.push(toGround)
					await wait(2000)
					that.blue = takeOut(that.blue, [toGround])
					that.ground.push(toGround)

				} else { // 没有匹配得弃牌逻辑是，先打一张，再补一张
					const toGround = that.getBlueDropCard()  // 
					that.blue2.push(toGround)
					await wait(2000)
					that.blue = takeOut(that.blue, [toGround])
					that.ground.push(toGround)

					const nextCard = that.getNextCard()
					if (nextCard) {
						that.blue.push(nextCard)
						await wait();
					}
				}

				if (that.blue.length === 0) { // 蓝方没牌了，游戏结束
					that.show.gameover = true
				}
				
				
				that.blue2 = []
				that.picked = false
				
				that.redCanDrop = true
				that.redCanPick = true
				
			},
			async pickCard() {
				const that = this
				const {
					picked,
					red2
				} = that
				
				if (picked) {
					that.redPicked.push(picked)
					for (const card of red2) {
						if(card) {
							that.redPicked.push(card)
						}
					}
					const newRed = []
					for(const cardInRed of that.red) {
						const found = that.red2.find(cc => {
							return cc && cc.index === cardInRed.index
						})
						if (!found) {
							newRed.push(cardInRed)
						}
					}
					that.red = newRed;
					
					const newGround = []
					for (const cardInGround of that.ground) {
						if(cardInGround.index !== picked.index) {
							newGround.push(cardInGround)
						}
					}
					that.ground = newGround
					
					that.picked = false
					that.red2 = []
					
					
					
					for (let idx=this.red.length; idx<5; idx++) {
						const nextCard = that.getNextCard()
						if(nextCard) {
							that.red.push(this.getNextCard())
							await wait();
						}
					}
					
					this.redCanPick = false
				}
			},
			
			getCardNumber(card) {
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
					if (that.getCardNumber(card) + this.pickedSum  === 14) {
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
				that.cards = mess([...createCards(1), ...createCards(2)])
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

			cardBg(card) {
				const {
					cardHeight,
					cardWidth
				} = this
				const rate = 68 / this.cardWidth
				const bgWidth = 884 / rate
				const bgHeight = 507 / rate
				return {
					'background-size': `${bgWidth}px ${bgHeight}px`,
					'background-position': `${bgWidth - cardWidth * card.num}px ${bgHeight - cardHeight * card.type}px`,
				}
			},
			cardStyle3(card) {
				const style = this.cardBg(card)
				style.position = 'relative'
				return style
			},

			cardStyle2(card) {
				const style = this.cardBg(card)
				style.top = `${card.top}px`;
				style.left = `${card.left}px`;
				style.transform = `rotate(${card.rotate}deg)`
				style.transition = 'none'
				return style
			},
			
			cardStyle1(card, idx) {
				const that = this
				const {
					red,
					blue,
					ground,
					cardWidth,
					cardHeight
				} = that
				

				const groundTop = cardHeight * 2
				let top = groundTop

				const rightCardLeft = this.widthComp - 1.2 * cardWidth
				let left = rightCardLeft
				
				let leftOffset = this.cardWidth * .3
				const popDist = cardHeight * .2
				let found = false

				const redBaseTop = this.heightComp / 5 * 4
				const blueBaseTop = cardHeight * .5
				const redIdx = indexOf(card, red)
				if (redIdx>=0) {
					top = redBaseTop
					left = leftOffset + redIdx * cardWidth * 1.1
					found = true
					if (indexOf(card, that.red2) >= 0) {
						top -= popDist
					}
				}
				
				const blueIdx = indexOf(card, blue)
				
				if (blueIdx>=0) {
					top = blueBaseTop
					left = leftOffset + blueIdx * cardWidth * 1.1
					found = true
					if (indexOf(card, that.blue2) >=0 ) {
						top -= popDist
					}
				}
				
				const groundIdx = indexOf(card, ground)
				let cardInGround = false
				if (groundIdx >= 0) {
					found = true
					const columnCount = 6
					const rowIdx = Math.floor(groundIdx / columnCount)
					const colIdx = groundIdx %  columnCount
					
					top = groundTop + rowIdx * cardHeight * 1.1
					left = leftOffset + colIdx * cardWidth * 1.1

					if(that.picked) {
						if (that.picked.index === card.index) {
							top -= popDist
						}
					}
					cardInGround = true
				}
				
				if(indexOf(card, that.redPicked) >= 0) { // 红方已经捡走得分的牌
					top = redBaseTop
					left = rightCardLeft
				}
				
				if(indexOf(card, that.bluePicked) >= 0) {
					top = blueBaseTop
					left = rightCardLeft
					
				}
				const rate = 68 / this.cardWidth
				const bgWidth = 884 / rate
				
				const cardBg = this.cardBg(card)
				if (!found) {
					cardBg['background-position'] = `${bgWidth + cardWidth}px ${cardHeight}px`
				}
				const style = {
					...cardBg,
					top: `${top}px`,
					left: `${left}px`,
					'z-index': 10001 + idx
				}
				
				if (cardInGround && that.redCanPick) {
					const {
						red2
					} = that
					if (red2.length > 0) {
						if (that.getCardNumber(card) + that.pickedSum  !== 14) {
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
		user-select: none;
		--main-height: 100%;
		--main-width: 100%;
		height: var(--main-height);
		width: var(--main-width);
		
		--card-width: calc(var(--main-width) / 10);
		--card-height: calc(var(--card-width) / 0.618);

		--font-size: calc(var(--card-width) * .3);
		--score-left: calc(var(--card-width) * .2);

		background: hsl(120, 100%, 30%);
		z-index: 10000;
		overflow: hidden;
		position: relative;
	}

	.card {
		width: var(--card-width);
		height: var(--card-height);
		background: url('../assets/poker.png');
		display: inline-block;
		margin: 1px;
		border-radius: 4px;
		position: absolute;
		transition: top 1s, left 1s;
		font-size: 80px;
		color: red;
	}
	
	.welcome {
		height: 100%;
		width: 100%;
		background: white;
		position: absolute;
		right: 0;
		top: -200%;
		position: absolute;
		font-size: calc(var(--main-width) * .08);
		cursor: pointer;
		transition: all 1s;
	}

	.welcome>.image {
		position: absolute;
		top: 0;
		background: url('../assets/peaple.png');
		width: 100%;
		height: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: calc(var(--card-width));
		--shadow: calc(var(--card-width) * .05);
		text-shadow: var(--shadow) var(--shadow) calc(var(--shadow) * .3) gray;
		overflow: hidden;
	}

	.welcome>.mask {
		position: absolute;
		top: 0;
		background: rgba(255, 255, 255, .5);
		width: 100%;
		height: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: calc(var(--card-width));
		--shadow: calc(var(--card-width) * .05);
		text-shadow: var(--shadow) var(--shadow) calc(var(--shadow) * .3) gray;
		overflow: hidden;
		backdrop-filter: blur(calc(var(--card-width) * .03));
	}
	
	.show-welcome > .welcome{
		top: 0;
	}
	
	.control-bar {
		position: absolute;
		width: 90%;
		bottom: 20px;
		display: flex;
		justify-content: space-around;
	}
	
	.control-bar > .button {
		border: 1px solid gray;
		--c2: hsl(220, 100%, 66%);
		--c1: hsl(240, 100%, 66%);
		background: linear-gradient(var(--c1), var(--c2), var(--c1));
		color: white;
		border-radius: calc(var(--card-width) * .1);
		margin: 10px;
		min-width: calc(var(--card-width));
		height: calc(var(--card-width) * .5);
		display: flex;
		justify-content: space-around;
		align-items: center;
		cursor: pointer;
	}
	
	.button.disabled {
		--c2: hsl(220, 100%, 76%);
		--c1: hsl(240, 100%, 76%);
	}
	
	.blue-picked-value {
		top: 50px;
		right: 200px;
	}

	.red-picked-value.show, .blue-picked-value.show {
		opacity: 1;
	}
	.red-picked-value, .blue-picked-value {
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
	
	.red-value {
		position: absolute;
		top: calc(var(--main-height) * 4 / 5 + var(--card-height));
		right: var(--score-left);
		font-size: var(--font-size);
	}
	.red-value:before {
		content: '玩家得分：'
	}
	
		
	.blue-value {
		position: absolute;
		top: calc(var(--card-height) * .25);
		right: var(--score-left);
		font-size: var(--font-size);
	}
	.blue-value:before {
		content: '电脑得分：'
	}
	
	.card-left {
		position: absolute;
		right: var(--score-left);
		top: calc(var(--card-height) * 3);
		font-size: var(--font-size);
	}
	.card-left:before {
		content: '剩余：'
	}

	.game-over {
		z-index: 999999;
		background: linear-gradient(rgba(200, 200, 200, .8), rgba(125, 122, 200, .8), rgba(200, 200, 200, .8));
		height: 100%;
		width: 100%;
		position: absolute;
		top: -100%;
		right: 0;
		transition: all 1s;

	}

	.game-over>.go-score {
		margin-top: calc(var(--card-height) * 1);
		font-size: var(--card-width);
		color: white;
		--shadow: calc(var(--card-width) * .05);
		text-shadow: var(--shadow) var(--shadow) 10px black;
	}

	.game-over>.statistic>.row {
		display: flex;
		align-items: center;
		padding-left: calc(var(--card-width) * 1.5);
		margin-bottom: calc(var(--card-height) * .1);
	}
	.game-over>.statistic>.row>.stat-text {
		font-size: calc(var(--card-width) * .7);
		color: white;
		padding-left: calc(var(--card-width) * .3);
		--shadow: calc(var(--card-width) * .05);
		text-shadow: var(--shadow) var(--shadow) 10px black;
	}

	.go-text1 {
		color: hsl(120, 100%, 50%);
		font-size: var(--card-width);
		--shadow: calc(var(--card-width) * .05);
		text-shadow: var(--shadow) var(--shadow) 10px black;
	}

	.go-text2 {
		color: hsl(120, 100%, 50%);
		font-size: var(--card-width);
		--shadow: calc(var(--card-width) * .05);
		text-shadow: var(--shadow) var(--shadow) 10px black;
	}

	.show-gameover>.game-over {
		top: 0;
	}
</style>
