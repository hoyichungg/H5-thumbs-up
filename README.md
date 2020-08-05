# CSS3 實現

用 CSS3 實現動畫，顯然，我們想到的是用 animation 。
首先看下 animation 合併寫法。


```javascript
animation: name duration timing-function delay iteration-count direction fill-mode play-state;
```

## Step 1: 固定區域，設置基本樣式

準備 1 張點讚動畫圖片。
看一下 HTML 結構。外層一個結構固定整個顯示動畫區域的位置。這裏在一個寬 100px ,高 200px 的 div 區域。

```HTML
<div class="praise_bubble"> 
	<div class="bubble b1 bl1"></div> 
</div>
```

```css
.praise_bubble{ 
	width:100px; 
	height:200px; 
	position:relative; 
	background-color:#f4f4f4; 
	} 
.bubble{ 
	position: absolute; 
	left:50%; 
	bottom:0; 
	}
```
## Step 2: 運動起來

使用 animation 的幀動畫，定義一個 bubble_y 的幀序列。

```css
.bl1{ 
	animation:bubble_y 4s linear 1 forwards ; 
	} 
	@keyframes bubble_y { 
		0% { 
			margin-bottom:0; 
			} 
		100% { 
			margin-bottom:200px; 
			} 
		}
```

這裏設置運行時間 4s ; 採用線性運動 linear，如果有需求當然也可以使用其他曲線，比如 ease; 每個點讚動畫只運行 1 次; 動畫是只需要向前 forwards。

## Step 3: 增加漸隱

漸隱效果，使用 opacity 即可。這裏我們固定在最後 1/4 開始漸隱。 修改 bubble_y:

```css
@keyframes bubble_y { 
	0% {
		 margin-bottom:0; 
	} 
	75%{ 
		opacity:1; 
		} 
	100% { 
		margin-bottom:200px; opacity:0; 
			} 
	}
```

## Step 4: 增加動畫放大效果

在最開始一小段時間，圖片由小變大。

於是我們新增一個動畫：bubble_big_1。

這裏從 0.3 倍原圖放大到 1 倍。這裏注意運行時間，比如上面設置，從動畫開始到結束總共是 4s，那麼這個放大時間就可以按需設置了，比如 0.5s。

```css
.bl1{ 
	animation:bubble_big 0.5s linear 1 forwards; 
	} 

@keyframes bubble_big_1 {
		 0% { 
			 transform: scale(.3); 
			 } 
		100% {
				transform: scale(1); 
			} 
		}
```			

## Step 5: 設置偏移

我們先定義幀動畫：bubble_1 來執行偏移。圖片開始放大階段，這裏沒有設置偏移，保持中間原點不變。

在運行到 25% * 4 = 1s，即 1s之後，是向左偏移 -8px, 2s 的時候，向右偏移 8px，3s 的時候，向做偏移 15px ,最終向右偏移 15px。

大家可以想到了，這是定義的一個經典的左右擺動軌跡，「向左向右向左向右」 曲線擺動效果

```css
@keyframes bubble_1 {
	0% { 
	} 
	25% { 
		 margin-left:-8px; 
		 } 
	50% {
		margin-left:8px 
		} 
	75% { 
		margin-left:-15px
		} 
	100% { 
		margin-left:15px 
		} 
	}
```
## Step 6: 補齊動畫樣式

這裏預設了一種運行曲線軌跡，左右擺動的樣式，我們在再預設更多種曲線，達到隨機軌跡的目的。
比如 bubble_1 的左右偏移動畫軌跡，我們可以修改偏移值，來達到不同的曲線軌跡。


## Step 7: JS 操作隨機增加節點樣式

提供增加點讚的方法，隨機將點讚的樣式組合，然後渲染到節點上。

```javascript
let praiseBubble = document.getElementById("praise_bubble"); let last = 0; function addPraise() { const b =Math.floor(Math.random() * 6) + 1; const bl =Math.floor(Math.random() * 11) + 1; // bl1~bl11 
let d = document.createElement("div"); d.className = `bubble b${b} bl${bl}`; d.dataset.t = String(Date.now()); praiseBubble.appendChild(d); } setInterval(() => { addPraise(); },300)
```
- 在使用 CSS 來實現點讚的時候，通常還需要注意設置 bubble 的隨機延時，比如：

```css
.bl2{ 
	animation:bubble_2 $bubble_time linear .4s 1 forwards,bubble_big_2 $bubble_scale linear .4s 1 forwards,bubble_y $bubble_time linear .4s 1 forwards; 
	}
```
這裏如果是隨機到 bl2，那麼延時 0.4s 再運行，bl3 延時 0.6s ……

如果是批量更新到節點上，不設置延時的話，那就會扎堆出現。隨機「 bl 」樣式，就隨機了延時，然後批量出現，都會自動錯峰顯示。當然，我們還需要增加當前用戶手動點讚的動畫，這個不需要延時。

另外，有可能同時別人下發了點讚 40 個，業務需求通常是希望這 40 個點讚氣泡都能依次出現，製造持續的點讚氛圍，否則下發量大又會扎堆顯示了。

那麼我們還需要分批打散點讚數量，比如一次點讚的時間（$bubble_time）是 4s, 那麼 4s 內，希望同時出現多少個點讚呢？比如是 10個，那麼 40 個點讚，需要分批 4 次渲染。



# Canvas 繪圖實現

## Step 1:初始化

頁面元素上新建 canvas 標籤，初始化 canvas。

canvas 上可以設置 width 和 height 屬性，也可以在 style 屬性裏面設置 width 和 height。

- canvas 上 style 的 width 和 height 是 canvas 在瀏覽器中被渲染的高度和寬度，即在頁面中的實際寬高。
- canvas 標籤的 width 和 height 是畫布實際寬度和高度。

```html
<canvas id="thumsCanvas" width="200" height="400" style="width:100px;height:200px"></canvas>

```
頁面上一個寬 200，高 400 的 canvas 畫布，然後整個 canvas 顯示在 頁面 寬 100，高 200 的區域內。canvas 畫布的內容被等比縮小一倍顯示在頁面。

定義一個點讚類，ThumbsUpAni，構造函數就是讀取 canvas,保存寬高值。

```javascript
class ThumbsUpAni{ 
	constructor(){ 
		const canvas = document.getElementById('thumsCanvas'); 
		this.context = canvas.getContext('2d')!; 
		this.width = canvas.width; 
		this.height = canvas.height; 
		} }
```
## Step 2：提前加載圖片資源

將需要隨機渲染的點讚圖片，先預加載，獲得圖片的寬高，如果有下載失敗的，則不顯示該隨機圖片即可。

```javascript
loadImages(){ 
	const images = [ 
		'jfs/t1/93992/8/9049/4680/5e0aea04Ec9dd2be8/608efd890fd61486.png', 
		'jfs/t1/108305/14/2849/4908/5e0aea04Efb54912c/bfa59f27e654e29c.png', 
		'jfs/t1/98805/29/8975/5106/5e0aea05Ed970e2b4/98803f8ad07147b9.png', 
		'jfs/t1/94291/26/9105/4344/5e0aea05Ed64b9187/5165fdf5621d5bbf.png', 
		'jfs/t1/102753/34/8504/5522/5e0aea05E0b9ef0b4/74a73178e31bd021.png', 
		'jfs/t1/102954/26/9241/5069/5e0aea05E7dde8bda/720fcec8bc5be9d4.png' 
		]; 
		
		const promiseAll = [] as Array<Promise<any>>; 
		images.forEach((src) => { 
			const p = new Promise(function (resolve) { 
				const img = new Image; 
				img.onerror = img.onload = resolve.bind(null, img); 
				img.src = 'https://img12.360buyimg.com/img/' + src; 
				}); promiseAll.push(p); 
			});
			Promise.all(promiseAll).then((imgsList) => { 
				this.imgsList = imgsList.filter((d) => { 
					if (d && d.width > 0) return true; 
					return false; 
					}); 
					if (this.imgsList.length == 0) { 
						logger.error('imgsList load all error'); 
						return; 
						} 
					}) 
				}
```
## Step 3：創建渲染對象

實時渲染圖片，使其變成一個連貫的動畫，很重要的是：生成曲線軌跡。這個曲線軌跡需要是平滑的均勻曲線。 假如生成的曲線軌跡不平滑的話，那看到的效果就會太突兀，比如上一個是 10 px，下一個就是 -10px，那顯然，動畫就是忽左忽右左右閃爍了。

理想的軌跡是上一個位置是 10px,接下來是 9px，然後一直平滑到 -10px，這樣的坐標點就是連貫的，看起來動畫就是平滑運行。

### 隨機平滑 X 軸偏移

如果要做到平滑曲線，其實可以使用我們再熟悉不過的正弦( Math.sin )函數來實現均勻曲線。

Math.sin(0) 到 Math.sin(9) 的曲線圖走勢圖，它是一個平滑的從正數到負數，然後再從負數到正數的曲線圖，完全符合我們的需求，於是我們再需要生成一個隨機比率值，讓擺動幅度隨機起來。

```javascript
const angle = getRandom(2, 10); 
let ratio = getRandom(10,30)*((getRandom(0, 1) ? 1 : -1)); 
const getTranslateX = (diffTime) => { 
	if (diffTime < this.scaleTime) {// 放大期間，不進行搖擺偏移 
		return basicX; 
		} else { 
			return basicX + ratio*Math.sin(angle*(diffTime - this.scaleTime)); 
			} 
			};
```
scaleTime 是從開始放大到最終大小，用多長時間，這裏我們設置 0.1，即總共運行時間前面的 10% 的時間，點讚圖片逐步放大。

diffTime，是只從開始動畫運行到當前時間過了多長時間了，為百分比。實際值是從 0 --》 1 逐步增大。 diffTime - scaleTime = 0 ~ 0.9, diffTime 為 0.4 的時候，說明是已經運行了 40% 的時間。

因為 Math.sin(0) 到 Math.sin(0.9) 曲線幾乎是一個直線，所以不太符合擺動效果，從 Math.sin(0) 到 Math.sin(1.8) 開始有細微的變化，所以我們這裏設置的 angle 最小值為 2。

這裏設置角度係數 angle 最大為 10 ，從底部到頂部運行兩個波峰。

當然如果運行距離再長一些，我們可以增大 angle 值，比如變成 3 個波峰(如果時間短，出現三個波峰，就會運行過快，有閃爍現象)。

### Y 軸偏移

開始 diffTime 為 0 ，所以運行偏移從 this.height --> image.height / 2。即從最底部，運行到頂部留下，實際上我們在頂部會淡化隱藏。

```javascript
const getTranslateY = (diffTime) => { 
	return image.height / 2 + (this.height - image.height / 2) * (1-diffTime); 
	};
```

### 放大縮小

當運行時間 diffTime 小於設置的 scaleTime 的時候，按比例隨着時間增大，scale 變大。超過設置的時間閾值，則返回最終大小。

```javascript
const basicScale = [0.6, 0.9, 1.2][getRandom(0, 2)]; 
const getScale = (diffTime) => { 
	if (diffTime < this.scaleTime) { 
		return +((diffTime/ this.scaleTime).toFixed(2)) * basicScale; 
		} else { 
			return basicScale; 
			} };
```

### 淡出

同放大邏輯一致，只不過淡出是在運行快到最後的位置開始生效。

```javascript
const fadeOutStage = getRandom(14, 18) / 100; 
const getAlpha = (diffTime) => { 
	let left = 1 - +diffTime; 
	if (left > fadeOutStage) { 
		return 1; 
		} else { 
			return 1 - +((fadeOutStage - left) / fadeOutStage).toFixed(2); 
			} 
		};
```

### 實時繪製

創建完繪製對象之後，就可以實時繪製了，根據上述獲取到的「偏移值」，「放大」和「淡出」值，然後實時繪製點讚圖片的位置即可。

每個執行周期，都需要重新繪製 canvas 上的所有的動畫圖片位置，最終形成所有的點讚圖片都在運動的效果。

```javascript
createRender(){ 
	return (diffTime) => { 
		 if(diffTime>=1) return true;
		  context.save();
		  const scale = getScale(diffTime); 
		  const translateX = getTranslateX(diffTime); 
		  const translateY = getTranslateY(diffTime); 
		  context.translate(translateX, translateY); 
		  context.scale(scale, scale); context.globalAlpha = getAlpha(diffTime); 
		  // const rotate = getRotate(); 
		  // context.rotate(rotate * Math.PI / 180); 
		  context.drawImage( 
			  image, 
			  -image.width / 2, 
			  -image.height / 2, 
			  image.width, 
			  image.height 
			  ); 
			  context.restore(); }; 
			  }
```
這裏繪製的圖片是原圖的 width 和 height。前面我們設置了 basiceScale，如果圖片更大，我們可以把 scale 再變小即可。

```javascript
const basicScale = [0.6, 0.9, 1.2][getRandom(0, 2)];
```

### 實時繪製掃描器

開啟實時繪製掃描器，將創建的渲染對象放入 renderList 數組，數組不為空，說明 canvas 上還有動畫，就需要不停的去執行 scan，直到 canvas 上沒有動畫結束為止。

```javascript

scan() { 
	this.context.clearRect(0, 0, this.width, this.height); 
	this.context.fillStyle = "#f4f4f4"; 
	this.context.fillRect(0,0,200,400); 
	let index = 0; 
	let length = this.renderList.length; 
	if (length > 0) { 
		requestAnimationFrame(this.scan.bind(this)); 
		} 
		while (index < length) { 
			const render = this.renderList[index]; 
			if (!render || !render.render || render.render.call(null, (Date.now() - render.timestamp) / render.duration)) { 
				// 結束了，刪除該動畫 
				this.renderList.splice(index, 1); length--; 
				} else { 
					// 當前動畫未執行完成，continue 
					index++; 
					}
				}
			}
```
這裏就是根據執行的時間來對比，判斷動畫執行到的位置了:

```javascript
diffTime = (Date.now() - render.timestamp) / render.duration
```
如果開始的時間戳是 10000，當前是100100，則說明已經運行了 100 毫秒了，如果動畫本來需要執行 1000 毫秒，那麼 diffTime = 0.1，代表動畫已經運行了 10%。

### 增加動畫

每點讚一次或者每接收到別人點讚一次，則調用一次 start 方法來生成渲染實例，放進渲染實例數組。如果當前掃描器未開啟，則需要啟動掃描器，這裏使用了 scanning 變量，防止開啟多個掃描器。

```javascript
start() { 
	const render = this.createRender(); 
	const duration = getRandom(1500, 3000); 
	this.renderList.push({ render, duration, timestamp: Date.now(), 
	}); 
	if (!this.scanning) { 
		this.scanning = true; 
		requestFrame(this.scan.bind(this)); 
		} 
		return this; 
	}
```

### 保持不扎堆

當接收到大量的點讚數據，且連續多次點讚（直播間人氣很旺的時候）。那麼點讚數據的渲染就需要特別注意了，否則頁面就是一坨一坨的點讚動畫。且銜接不緊密。

```javascript

thumbsUp(num: number) { 
	if (num <= this.praiseLast) return; 
	this.thumbsStart = this.praiseLast; 
	this.praiseLast = num; 
	if (this.thumbsStart + 500 < num) 
		this.thumbsStart = num - 500; 
	const diff = this.praiseLast - this.thumbsStart; 
	let time = 100; 
	let isFirst = true; 
	if (this.thumbsInter != 0) { 
		return; 
		} 
		this.thumbsInter = setInterval(() => { 
			if (this.thumbsStart >= this.praiseLast) { 
				clearInterval(this.thumbsInter); 
				this.thumbsInter = 0; 
				return; 
				} 
				this.thumbsStart++; 
				this.thumbsUpAni.start(); 
				if (isFirst) { 
					isFirst = false; 
					time = Math.round(5000 / diff); 
					} 
				}, time); 
			},
```
這裏開啟定時器，記錄定時器裏面處理的 thumbsStart 的值，如果有新增點讚，且定時器還在運行，直接更新最後的 praiseLast 值，定時器會依次將點讚請求全部處理完。

定時器的延時時間 time 根據開啟定時器的時候，需要渲染多少點讚動畫來決定的，比如需要渲染 100 個點讚動畫，我們將 100 個點讚動畫分佈在 5s 內渲染完。

- 對於熱門直播，會同時渲染的動畫很多，不會扎堆顯示，且動畫完全能銜接上，不停的冒泡點讚動畫。
- 對於冷門直播，有多餘一個的點讚請求，我們能打散到 5s 內顯示，也不會扎堆顯示。



















