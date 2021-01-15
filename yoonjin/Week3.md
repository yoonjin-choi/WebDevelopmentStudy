# Week3
## Section 6. Asynchronous Node.js
### 1.Asynchronous(비동기) basics

 + 동기(Syncronous) 와 비동기(Asynchronous)

   	- 동기 : 결과가 주어질 때까지 아무것도 못하고 대기
   	- 비동기 : 결과가 주어지는데 걸리는 시간동안 다른 작업 수행 가능
  + 예시
	-  code
```javascript
	console.log('Starting')
	setTimeout(()=>{
		console.log('2 second timer')
	 },2000) // run some code after a specific time passed ( fuction, millsecond)
console.log('Stopping')
```

	-  result
    > Starting     
    Stopping    
    2 second timer

  - Synchronous model : 2second를 기다린 후 program 재진행
  - Asynchronous non blocking model : 2초간 다른 코드를 실행할 수 있음.
  - 한 사용자의 데이터를 가져오는 data request의 경우, 동시에 다른 일을 수행할 수 있으므로 그러한 경우에 유용.
### 2.Call stack, Callback Queue, Event loop
1. 예시
	+ code
		```javascript
		console.log('Starting')

		setTimeout(()=>{
   			console.log('2 second timer')
		},2000) // run some code afte a specific time passed ( fuction, millsecond)

		setTimeout(()=>{
	
 		console.log('0 second timer')
	 	},0)
	 	
	 	console.log('Stopping')
```
	+ result)   
		
		>Starting     

 		Stopping  
 	    0 second timer   
 		2 second time
 	    <br>
 	    
 	   <img src="./img/asynchronous.PNG" width="700px" height="450px"  ></img>
  2. Call stack
	  + V8과 javaScript engine에 의해 제공되는 간단한 데이터 structure.
	  + 실행되고 있는 모든 함수들을 지속적으로 관리하는 곳
	  + Last In First Out 의 Stack 형태
  
  3. Node APIS
	  + **setTimeout** 은 javscript definition 도 아니고 v8에 속하는 것도 아닌, Nodejs에서 제공하는 c++을 사용하여 구현된 function.(web API에서 제공하는 비동기 함수)
	  + **setTimeout()**  호출 시, node API에 등록되어 (위 얘제에서는) 2초간 대기, 비동기 처리.
	  + 그 동안, Call Stack에서는 다른 함수 수행 가능.    
	  + java script는 single thread 동작 언어. 즉, 한번에 하나의 작업만 수행 가능.
	  + 하지만, 동시성을 보장하는 비동기(asynchronous), nonblocking 작업들은 javascript engine을 구동하는 runtime 환경 즉, Node.js에서 담당.
	 + **setTimeout(func(),0)** 또한 node API에 등록된 후 0초 대기 - Call Stack에 쌓이는 다른 함수보다 늦게 호출 되는 
  4. callback queue
	 + event가 끝이 나면, **setTimeout(func(),0)** 은 Callback Queue 에 추가
  5. Eventloop
	 + Eventloop는 call stack이 empty 상태가 될 때까지 대기
	 + synchrnoous: main함수가 끝이나면 application ends.
	 + Asynchrnous: main이 끝이나도 eventloop는 일을 시작.
	 + Call stack empty를 확인 후, call back Queue 로부터 call stack으로 item을 불러 온 후 실행.


### 3. Making Http Request

+ http request 가 중요한 이유 : 다른 서버와의 소통에 사용 되는 핵심.

+ (예시) location에 대한 정보를 통해 그 지역에 대한 weather 정보 불러오기
	
	1. npm init 
    2. npm install postman-request -. 필요모듈 설치    
    3. Node application에서 HTTP request 요청하기
      <br>
      <br>
  
    ```javascript
    	//weatherstack url : access_key: 암호화키 &qeury=위경도(위치)
		const url ='http://api.weatherstack.com/current?access_key=0394f78f3876a54a71655a23d096c115&query=37.8267,-122.4233'

		request({ url:url },(error,response)=> {
    		const data= JSON.parse(response.body) //Url 통해 가져온 JSON to Object
    		console.log(data.current)
		})
    ```
  
### 4. HTTP Request 맞춤설정하기.
 1. *chrome extension* 다운로드 사용하여 JSON data를 정리된 형태로 보여줌.
 2. json: true 통해 respone body를 JSON으로 parse.(JSON.parse() 따로 사용할 필요 없음.)
  ```javascript
  request({ url:url, json : true },(error,response)=> {
  	console.log(response.body.current)
}
  ```
 3. 필요한 설정 추가(URL) 및 필요한 데이터만 가져오기
 ```javascript
 const url ='http://api.weatherstack.com/current?access_key=0394f78f3876a54a71655a23d096c115&query=37.8267,-122.4233&units=f' //units =f 추가하여 온도 단위 변경
 // url에 units =f 추가하여 온도 단위 변경
 request({ url:url, json : true },(error,response)=> { // parse the response body as JSON
    const temp=response.body.current.temperature
    const feel_temp=response.body.current.feelslike

    console.log(response.body.location.country+"'s weather is "+response.body.current.weather_descriptions[0]+".")
    //United States of America's weather is Clear. (지역, 날씨 정보)
    console.log("It is currently ",temp," degrees out. It feels like ",feel_temp," degrees out.")
    //It is currently  46  degrees out. It feels like  45  degrees out.(온도 및 체감온도)
})
  	
        
      


 ```
### 5. Handling Error
1. Network connecting Error (Low level error)
- network에 연결되어 있지 않다면 error이 value를 가지고 , response는 가지지 않음.
- 반대로 reponse가 값을 가진다면, error은 가지지 않음.
- code
	```javascript
	request({ url:geocode_url, json:true },( error,response)=> {
    //error code
	if (error){ 
        console.log('Unable to connect to geocoding service.')
    }
	//response
    else{
    }
})
```
2. URL Error
- URL 이 필요로 하는 query를 모두 입력받지 못했을 경우,
  response의 body값을 받지만 error-> url 직접 입력해본 후 error 처리
-(ex)
`````javascript
request({ url:geocode_url, json:true },(error,response)=> {
    if (error){ // error in low level OS
        console.log('Unable to connect to geocoding service.')
    } else if(response.body.message){ //잘못된 url
        console.log('Unable to find location')
    }
    else{
    }
	})
`````
### 6. Callback Function
- 콜백함수는 함수(A)의 인자로 또 다른 함수(B)를 전달하여 , A함수 내부에서 B함수를 실행시키는 것.  <br>

 ```javascript
const names=['Jean','Jen','Jessy']
const shortNames = names.filter( (name)=> name.length <=4  )
```
	- ex1)  예시에서 filter함수 내에 익명의 함수를 넣어 names의 각 요소중 길이가 4보다 작거나 같은 요소를 return 하도록 함.

 ```javascript
setTimeout(()=>{
 	console.log('Two seconds are up') 
	//setTimeout()안의 function 또한 callback
 },2000)
```
	- ex2) setTimeout functon 내에 'Two seconds are up' 출력함수를 넣어
	setTimeout 함수 실행시 호출되도록 함.
	- setTimeout : call back function 이자 asychronous
- call back 함수는 비동기식 개발의 핵심이다
- call back 함수는 어떻게 사용하는가?
```javascript
const geocode = (address, callback) => {
 	setTimeout(() => {
 		const data = {
 		latitude: 0,
 		longitude: 0
 		}
 		callback(data) // callback은 1개의 argument
 	}, 2000)
}
geocode('Philadelphia', (data) => { // call back 함수는 1개의 인자 필요
 console.log(data) 
})
```
	=> **geocode** 함수는 address 와 callback function을 인자로 가지고,
	 callback function은 data로 정의된 하나의 인자를 필요로 한다.
 ### 7. Callback Chaining
 (ex)
```javascript
	geocode( address ,(error, data )=>{ 
		
		//forecast function은 geocode로 부터 data.lat과 data.long을 인자로,
        forecast(data.lat,data.long , (error, foreCastdata) => { 
			
		//forecast function내부에서 geocode의 data.location 호출가능.
        console.log(data.location)
        console.log(foreCastdata)
        })
    })
```
 즉, 
  Step 1) geocode 실행 통해 data(좌표) get .
  Step 2) forecast 통해 날씨 정보(foreCastdata) get.

### 8. CallBack Destructuring
 #### 1. Object property shorthand syntax
  ```javascript
	const name = 'Andrew'
	const userAge = 25

	//original
	const user ={
    	name : name,
    	age : userAge,
    	location : 'Korea'
	}
```
- object 생성 시, 일반적으로는 위와 같이 사용해옴.
- object property 와 variable 의 이름이 같은 경우,  shorthand syntax로 다음과 같이 사용가능.
	
 ```javascript
	const name = 'Andrew'
	const userAge = 25

	const user ={
    	name,
    	userAge, //property 명 = variable 명 = userAge
    	location : 'Korea'
	}
```

#### 2. Object Destructuring
```javascript
	const product = {
    	label : 'Red notebook',
    	price : 3,
    	stock : 201,
    	slaePrice : undefined
	}
```
1. const { 사용하고자 하는 properties : label, stock } -> label, stock variable이 생성
```javascript
	const {label, stock} = product
	console.log( label, stock ) // print ) Red notebook, 3
```

2. Rename 가능 
```javascript
	const { label:productLabel, stock } =product
	console.log(productLabel) // Red notebook
	console.log(stock) //3
```
3.  object에 포함되지 않은 property(*rating*) 또한 variable로 생성,
 - object에 포함되지 않은 property에 대해서는 default값(여기서 5)또한 설정 가능.
```javascript
	const {label:productLabel, stock, rating =5} =product
	console.log(rating) //5
```

 #### 3. Default Function Parameter ( Section 8 에서 좀 더 상세히 다룸.)
```javascript
const greet = (name = 'User')=>{ 
    console.log('Hello, '+name+'!')
}
greet('Andrew') //print : Hello, Andrew!
greet() //print: Hello, User!
```
 - name 이 인자로 주어지면 , name = 인자값,
 - 그렇지 않을 경우, name = 'User'.
 - weather -app example 
 
  `geocode(String(address),(error, {location,lat,long} )=>{ `
 
  > 위 코드는 {location,lat,long} 에 값이 들어오지 않은 경우, 충돌 발생하므로

 	`geocode(String(address),(error, {location,lat,long}={ } )=>{ `
  > 이와 같이 해주어야, default값이 설정되며 오류 해결.
