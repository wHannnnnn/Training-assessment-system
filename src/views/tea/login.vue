<template>
	<div class="login">
		<img src="../../assets/img/171.png" class="img" >
		<div class="input_box">
			<form class="form">
				<div class="form__cover"></div>
				<div class="form__loader">
					<div class="spinner active">
						<svg class="spinner__circular" viewBox="25 25 50 50">
							<circle class="spinner__path" cx="50" cy="50" r="20" fill="none" stroke-width="4" stroke-miterlimit="10"></circle>
						</svg>
					</div>
				</div>
				<div class="form__content">
					<h1>用户登录</h1>
					<div class="styled-input">
						<input type="text" class="styled-input__input" name="nickname" v-model='username'>
						<div class="styled-input__placeholder">
							<span class="styled-input__placeholder-text">用户名</span>
						</div>
						<div class="styled-input__circle"></div>
					</div>
					<div class="styled-input">
						 <input type="password" class="styled-input__input"  v-model='password'>
						<div class="styled-input__placeholder">
							<span class="styled-input__placeholder-text">密码</span>
						</div>
						<div class="styled-input__circle"></div>
					</div>
					<div class="styled-input" style="width: 65%">
						 <input type="text" class="styled-input__input" v-model='picLyanzhengma'>
						<div class="styled-input__placeholder">
							<span class="styled-input__placeholder-text">验证码</span>
						</div>
						<div class="styled-input__circle"></div>
						<div class="YZimg">
							 <input type="button"  @click="createCode"  class="verification" v-model="checkCode"/>
						</div>
					</div>
					<div class="unit_but">
						<unit_button :msg='msg'  @click.native='handleLogin()'></unit_button>
					</div>
					
				</div>
				
			</form>
		</div>
	</div>
</template>

<script>

	export default {
		name: 'login',
		data() {
			return {
				username:'',
				password:'',
				picLyanzhengma:'',
				checkCode:'',
				msg:'登陆'
			}
		},
		methods: {
			// 图片验证码
		    createCode(){
	         //先清空验证码的输入
	         this.code = "";
	         this.checkCode = "";
	         this.picLyanzhengma = "";
	         //验证码的长度  
	            var codeLength = 4; 
	            //随机数 
	          var random = new Array(0,1,2,3,4,5,6,7,8,9,'A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z');  
	          for(var i = 0; i < codeLength; i++) {
            //取得随机数的索引（0~35）
                var index = Math.floor(Math.random()*36);   
            //根据索引取得随机数加到code上
              this.code += random[index];   
                    }
                    //把code值赋给验证码  
	          this.checkCode = this.code; 
		      },
			 async handleLogin() {
  			 if (this.picLyanzhengma.toLowerCase() != this.checkCode.toLowerCase()) {
  			 	this.$message.error('验证码输入错误')
          this.createCode()
  			 	return ;
  			 }

  			var data = await this.$axiosPost(this.$axiosURL.system+'login',{'username':this.username,'password':this.password})
        if (data.code == '401') {
          this.$message.error(data.error_description)
          this.createCode()
        } else if(data.code == '400'){
          this.$message.error('密码错误')
          this.createCode()
        }else if(data.loginUser.sysRole.code == 'STUDENT'){
          this.$message.error('密码错误')
          this.createCode()
        }else{
          if (data.access_token) {
            sessionStorage.setItem('access_token', data.access_token)
            sessionStorage.setItem('loginUser', JSON.stringify(data.loginUser))
            sessionStorage.setItem('role', data.loginUser.sysRole.code)
            this.$store.state.login = true;

            this.$store.state.role = data.loginUser.sysRole.code;
            this.$store.state.access_token = data.access_token;

             var userID = data.loginUser.id
              this.$axiosRes('get',this.$axiosURL.k_neoUser+ 'find/' + userID).then((res)=>{
                sessionStorage.setItem('aId', JSON.stringify(res.id))
                this.$store.state.aId = res.id
              })

            this.$axiosRes('get',this.$axiosURL.b_dictionarys,{}).then((res)=>{
              sessionStorage.setItem('dictionarys', JSON.stringify(res))
              this.$store.state.dictionarys = res;
            }).then(()=>{
              console.log(44,this.$store.state.role );
              if (this.$store.state.role =='QUESTIONS_BANK_TEACHER') {
                this.$router.push({path: '/knowManagement'});
              } else {
               this.$router.push({path: '/teaindex'});
              }
            })
          }
        }
  			

			},
      keyupSubmit:function(){
        document.onkeydown = e =>{
          let body = document.getElementsByName('body')[0]
          if (e.keyCode === 13 ) {
            this.handleLogin()
          }
        }
      }
		},
		created:function(){
			this.createCode()
      this.keyupSubmit()

		},
		mounted:function(){
			var placeholders = document.querySelectorAll('.styled-input__placeholder-text'),
		    inputs = document.querySelectorAll('.styled-input__input');

			placeholders.forEach(function (el, i) {
			    var value = el.innerText,
			        html = '';
			    for (var _iterator = value, _isArray = Array.isArray(_iterator), _i = 0, _iterator = _isArray ? _iterator : _iterator[Symbol.iterator]();;) {
			        var _ref;

			        if (_isArray) {
			            if (_i >= _iterator.length) break;
			            _ref = _iterator[_i++];
			        } else {
			            _i = _iterator.next();
			            if (_i.done) break;
			            _ref = _i.value;
			        }

			        var w = _ref;

			        if (!value) value = '&nbsp;';
			        html += '<span class="letter">' + w + '</span>';
			    }
			    el.innerHTML = html;
			});

			inputs.forEach(function (el) {
			    var parent = el.parentNode;
			    el.addEventListener('focus', function () {
			        parent.classList.add('filled');
			        placeholderAnimationIn(parent, true);
			    }, false);
			    el.addEventListener('blur', function () {
			        if (el.value.length) return;
			        parent.classList.remove('filled');
			        placeholderAnimationIn(parent, false);
			    }, false);
			});

			function placeholderAnimationIn(parent, action) {
			    var act = action ? 'add' : 'remove';
			    var letters = parent.querySelectorAll('.letter');
			    letters = [].slice.call(letters, 0);
			    if (!action) letters = letters.reverse();
			    letters.forEach(function (el, i) {
			        setTimeout(function () {
			            var contains = parent.classList.contains('filled');
			            if (action && !contains || !action && contains) return;
			            el.classList[act]('active');
			        }, 50 * i);
			    });
			}

			setTimeout(function () {
			    document.body.classList.add('on-start');
			}, 100);

			setTimeout(function () {
			    document.body.classList.add('document-loaded');
			}, 1800);
		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style >

.login{
	width: 100%;
	height: 100%;
	background: url('../../assets/img/login_background.jpg') center center no-repeat;
}
.img{
	position: absolute;
	left: 15%;
	top: 29%;
	width: 500px;
}
.input_box{
	position: absolute;
	left: 55%;
	top: 25%;
	/*width: 340px;
	height: 310px;
	padding: 0px 5px 0px 5px;
	background: rgba(0,0,0,0.4 );
	border: rgba(16,117,183,0.5) 2px solid !important;
	border-radius: 8px;*/
	z-index: 100;
}
/*.input_box h3{
	margin-bottom: 10px;
	color: #fff
}

.login i{
	color: #fff;
	margin-top:10px;
}*/

.form {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  position: relative;
  width: 400px;
  height: 400px;
  -ms-flex-negative: 0;
      flex-shrink: 0;
  padding: 20px;
  border-radius: 5px;
}
.form__loader {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  z-index: -4;
  -webkit-transition: all 0.5s ease;
  transition: all 0.5s ease;
}
.form__content {
	width: 300px;
  text-align: center;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
      -ms-flex-direction: column;
          flex-direction: column;
  position: relative;
  opacity: 0;
  -webkit-transform: translateY(10px);
          transform: translateY(10px);
  -webkit-transition: all 0.5s ease 0.7s;
  transition: all 0.5s ease 0.7s;
}
.form__cover {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  z-index: -4;
  border-radius: 7px;
  overflow: hidden;
  -webkit-transition: all 0.3s ease 0.8s;
  transition: all 0.3s ease 0.8s;
  box-shadow: 0 0 0 0 rgba(0, 0, 0, 0);
  border: rgba(16,117,183,0.5) 2px solid !important;
}
.form__cover:after {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  /*background: #4d317a;*/
  background: rgba(0,0,0,0.4 );

  z-index: -4;
  border-radius: 50%;
  -webkit-transition: all 1.5s ease 0.3s;
  transition: all 1.5s ease 0.3s;
  -webkit-transform: scale(0);
          transform: scale(0);
}
.form__cover:before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  background: white;
  z-index: -5;
  border-radius: 50%;
  -webkit-transition: all 0.5s ease;
  transition: all 0.5s ease;
  -webkit-transform: scale(0);
          transform: scale(0);
}
body.on-start .form__cover:before {
  -webkit-transform: scale(0.15);
          transform: scale(0.15);
}
body.document-loaded .form__loader {
  -webkit-transform: scale(0);
          transform: scale(0);
  opacity: 0;
  visibility: hidden;
}
body.document-loaded .form__content {
  opacity: 1;
  -webkit-transform: none;
          transform: none;
}
body.document-loaded .form__cover {
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
}
body.document-loaded .form__cover:after {
  -webkit-transform: scale(2);
          transform: scale(2);
}
body.document-loaded .form__cover:before {
  -webkit-transition: opacity 0.3s ease 0.8s, -webkit-transform 2s ease;
  transition: opacity 0.3s ease 0.8s, -webkit-transform 2s ease;
  transition: transform 2s ease, opacity 0.3s ease 0.8s;
  transition: transform 2s ease, opacity 0.3s ease 0.8s, -webkit-transform 2s ease;
  -webkit-transform: scale(2);
          transform: scale(2);
  opacity: 0;
}
h1 {
  font-size: 40px;
  margin: 15px 0 20px 0;
  letter-spacing: 0.05em;
  /*color: #714cab;*/
  color: #ffffff;
  font-weight: 700;
}
.styled-button {
	left: 60px;
  -webkit-appearance: none;
  -webkit-user-select: none;
  cursor: pointer;
  font-size: 14px;
  width: 60%;
  padding: 20px;
  outline: none;
  background: none;
  position: relative;
  color: #492e72;
  border-radius: 3px;
  margin-bottom: 25px;
  border: none;
  text-transform: uppercase;
  font-weight: 700;
  letter-spacing: 0.1em;
  background: #714cac;
  -webkit-transition: all 0.3s ease;
  transition: all 0.3s ease;
  overflow: hidden;
}
.styled-button__real-text-holder {
  position: relative;
}
.styled-button__real-text {
  color: transparent;
  display: inline-block;
}
.styled-button__text-holder {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-transition: all 0.3s ease;
  transition: all 0.3s ease;
}
.styled-button__moving-block {
  -webkit-transition: all 0.3s ease;
  transition: all 0.3s ease;
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
}
.styled-button__moving-block.back {
  color: white;
  -webkit-transform: translateX(-100%);
          transform: translateX(-100%);
}
.styled-button__moving-block.back .styled-button__text-holder {
  -webkit-transform: translateX(100%);
          transform: translateX(100%);
}
.styled-button:hover,
.styled-button:active {
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
  background: #7a51bb;
}
.styled-button:hover .face,
.styled-button:active .face {
  -webkit-transform: translateX(100%);
          transform: translateX(100%);
}
.styled-button:hover .face .styled-button__text-holder,
.styled-button:active .face .styled-button__text-holder {
  -webkit-transform: translateX(-100%);
          transform: translateX(-100%);
}
.styled-button:hover .back,
.styled-button:active .back {
  -webkit-transform: translateX(0);
          transform: translateX(0);
}
.styled-button:hover .back .styled-button__text-holder,
.styled-button:active .back .styled-button__text-holder {
  -webkit-transform: translateX(0);
          transform: translateX(0);
}
.styled-button:active {
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
}
.styled-input {
  width: 100%;
  position: relative;
  margin-bottom: 25px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 3px;
  -webkit-transition: all 0.3s ease;
  transition: all 0.3s ease;
}
.styled-input__circle {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: -2;
  overflow: hidden;
  border-radius: 3px;
}
.styled-input__circle:after {
  content: '';
  position: absolute;
  left: 16.5px;
  top: 19px;
  height: 14px;
  width: 14px;
  z-index: -2;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.15);
  box-shadow: 0 0 10px rgba(255, 255, 255, 0);
  -webkit-transition: opacity 1s ease, -webkit-transform 0.6s ease;
  transition: opacity 1s ease, -webkit-transform 0.6s ease;
  transition: transform 0.6s ease, opacity 1s ease;
  transition: transform 0.6s ease, opacity 1s ease, -webkit-transform 0.6s ease;
}
.styled-input__input {
  width: 100%;
  -webkit-appearance: none;
  font-size: 14px;
  outline: none;
  background: none;
  padding: 18px 15px;
  color: #ceafff;
  border: none;
  font-weight: 600;
  letter-spacing: 0.035em;
}
.styled-input__placeholder {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  z-index: -1;
  padding-left: 45px;
  color: white;
}
.styled-input__placeholder-text {
  -webkit-perspective: 500px;
          perspective: 500px;
  display: inline-block;
}
.styled-input__placeholder-text .letter {
  display: inline-block;
  vertical-align: middle;
  position: relative;
  -webkit-animation: letterAnimOut 0.25s ease forwards;
          animation: letterAnimOut 0.25s ease forwards;
  text-shadow: 0 0 5px;
}
.styled-input__placeholder-text .letter.active {
  -webkit-animation: letterAnimIn 0.25s ease forwards;
          animation: letterAnimIn 0.25s ease forwards;
}
.styled-input:hover {
  border-color: rgba(255, 255, 255, 0.4);
}
.styled-input.filled {
  border-color: rgba(255, 255, 255, 0.2);
}
.styled-input.filled .styled-input__circle:after {
  -webkit-transform: scale(37);
          transform: scale(37);
  opacity: 0;
}
@-webkit-keyframes letterAnimIn {
  0% {
    -webkit-transform: translate(0, 0);
            transform: translate(0, 0);
  }
  25% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    color: red;
  }
  45% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
    color: red;
  }
  55% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
  }
  56% {
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
    opacity: 0;
    color: #00ff6b;
  }
  76% {
    color: #00ff6b;
    opacity: 1;
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
  }
  100% {
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
    opacity: 1;
  }
}
@keyframes letterAnimIn {
  0% {
    -webkit-transform: translate(0, 0);
            transform: translate(0, 0);
  }
  25% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    color: red;
  }
  45% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
    color: red;
  }
  55% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
  }
  56% {
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
    opacity: 0;
    color: #00ff6b;
  }
  76% {
    color: #00ff6b;
    opacity: 1;
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
  }
  100% {
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
    opacity: 1;
  }
}
@-webkit-keyframes letterAnimOut {
  0% {
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
    opacity: 1;
  }
  25% {
    -webkit-transform: translate(-30px, -40px);
            transform: translate(-30px, -40px);
    opacity: 0;
  }
  45% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
  }
  55% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
    color: red;
  }
  56% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    color: red;
  }
  100% {
    -webkit-transform: translate(0, 0);
            transform: translate(0, 0);
  }
}
@keyframes letterAnimOut {
  0% {
    -webkit-transform: translate(-30px, -27px);
            transform: translate(-30px, -27px);
    opacity: 1;
  }
  25% {
    -webkit-transform: translate(-30px, -40px);
            transform: translate(-30px, -40px);
    opacity: 0;
  }
  45% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
  }
  55% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    opacity: 0;
    color: red;
  }
  56% {
    -webkit-transform: translate(0, 10px);
            transform: translate(0, 10px);
    color: red;
  }
  100% {
    -webkit-transform: translate(0, 0);
            transform: translate(0, 0);
  }
}
.spinner {
  position: relative;
  margin: auto;
  width: 50px;
  height: 50px;
  -webkit-transition: all 0.2s ease 0s;
  transition: all 0.2s ease 0s;
}
.spinner__circular {
  -webkit-animation: rotate 1.5s linear infinite;
          animation: rotate 1.5s linear infinite;
  -webkit-animation-play-state: paused;
          animation-play-state: paused;
  -webkit-transform-origin: center center;
          transform-origin: center center;
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  margin: auto;
}
.spinner__path {
  stroke-dasharray: 1, 200;
  stroke-dashoffset: 0;
  -webkit-animation: dash 1.3s ease forwards 0.5s;
          animation: dash 1.3s ease forwards 0.5s;
  opacity: 0;
  stroke-linecap: round;
  stroke: #7b23ff;
  -webkit-animation-play-state: running;
          animation-play-state: running;
}
@-webkit-keyframes dash {
  0% {
    stroke-dasharray: 1, 200;
    stroke-dashoffset: 0;
    opacity: 0;
  }
  50% {
    stroke-dasharray: 40, 200;
    opacity: 1;
  }
  100% {
    stroke-dasharray: 125, 200;
    opacity: 1;
  }
}
@keyframes dash {
  0% {
    stroke-dasharray: 1, 200;
    stroke-dashoffset: 0;
    opacity: 0;
  }
  50% {
    stroke-dasharray: 40, 200;
    opacity: 1;
  }
  100% {
    stroke-dasharray: 125, 200;
    opacity: 1;
  }
}
.YZimg{
	position: absolute;
    left: 100%;
    top: 15%;
    
}
.verification{
    border-radius: 12px;
    width:100px;
    letter-spacing:5px;
    margin-left: 20px;
    height: 40px;
    transform: translate(-15px,0);
}
.unit_but{
	left: 75px;
}
</style>
