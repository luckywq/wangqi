<template>
  <div id="login" @mousemove="moveLogo($event)" @keydown="enter($event)">
    <div class="bg_line"></div>
    <img src="../../assets/image/login_logo.png" />
    <div class="login-main">
        <label class="main-subtitle">分布式水处理系统</label>
        <div class="main-divider"></div>
        <div class="main-post">
          <label>账号</label>
          <input placeholder="请输入账号"  v-model="username" autocomplete="off"/>
          <label>密码</label>
          <input type="password" name="pwd" style="display:none">
          <input type="password" name="pwd" placeholder="请输入密码" v-model="password" autocomplete="off"/>
          <button @click="login($event)">登录</button>
        </div>
        <div class="main-tools">
          <el-checkbox v-model="remember">记住我</el-checkbox>
          <span @click="forgot">忘记密码</span>
        </div>
    </div>
    <label class="copyright">
      北京智信远景软件技术有限公司 Copyright©2015.All Rights Reserved
    </label>

    <el-dialog
      title="密码找回"
      :visible.sync="findDialog.visible"
      width="450px">
      <label>账号</label><el-input v-model="findDialog.username" placeholder="请输入账号"></el-input>
      <span slot="footer" class="dialog-footer">
        <el-button @click="findDialog.visible = false">取 消</el-button>
        <el-button type="primary" @click="verifyUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
const reg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+((\.[a-zA-Z0-9_-]{2,3}){1,2})$/;
export default {
	name: 'Login',
	data() {
		return {
			username: '',
			password: '',
			remember: true,
			findDialog: {
				username: '',
				visible: false
			}
		};
	},
	methods: {
		async login() {
			if (this.username !== '' && this.password !== '') {
				if (reg.test(this.username)) {
					const button = document.getElementsByTagName('button')[0];
					button.innerHTML = '登陆中';
					button.style.cssText =
						'font-size:14px;letter-spacing:2px;cursor:not-allowed';
					let user = {};
					user.username = this.username;
					user.password = this.password;
					const res = await this.$http.post('/public/login', user);
					sessionStorage.setItem('actk', res.result.actk);
					let userInfo = {};
					Object.assign(userInfo, res.result);
					sessionStorage.setItem('userInfo', JSON.stringify(userInfo));
					if (this.remember) {
						localStorage.setItem('rememberUser', true);
						localStorage.setItem('authorize', JSON.stringify(user));
					} else {
						localStorage.setItem('rememberUser', false);
						localStorage.removeItem('authorize');
					}
					this.$router.push({ path: '/screen' });
				} else {
					this.$message({
						message: '请输入正确的账号',
						type: 'error'
					});
				}
			} else {
				this.$message({
					message: '请输入账号密码',
					type: 'error'
				});
			}
		},
		enter(ev) {
			//回车事件
			if (ev.keyCode === 13) {
				this.login();
			}
		},
		moveLogo(ev) {
			//logo和背景线条视觉差
			const logo = document.getElementsByTagName('img')[0];
			const bgLine = document.getElementsByClassName('bg_line')[0];
			let offsetX = ev.clientX - window.innerWidth / 2;
			let offsetY = ev.clientY - window.innerHeight / 2;
			const logoCenterTop = window.innerHeight * 0.25;
			const logoCenterLeft = window.innerWidth * 0.5 - 162.5;
			const lineCenterLeft = window.innerWidth * 0.5 - 325;
			const lineCenterTop = window.innerHeight * 0.03;
			let logMoveY, lineMoveY, logMoveX, lineMoveX;
			logMoveY = logoCenterTop + offsetY * 0.03;
			lineMoveY = lineCenterTop - offsetY * 0.03;
			logMoveX = logoCenterLeft + offsetX * 0.05;
			lineMoveX = lineCenterLeft - offsetX * 0.05;
			logo.style.top = logMoveY + 'px';
			logo.style.left = logMoveX + 'px';
			bgLine.style.top = lineMoveY + 'px';
			bgLine.style.left = lineMoveX + 'px';
		},
		forgot() {  //密码找回
			this.findDialog.visible = true;
			this.findDialog.username = '';
		},
		async verifyUser() {
			if (this.findDialog.username !== '' && reg.test(this.findDialog.username)) {
				const res = await this.$http.post('/public/forget', { 'username': this.findDialog.username, 'system': 'water' });
				this.$message({
					message: res.message,
					type: 'success'
				});
			} else {
				this.$message({
					message: '请输入正确的账号',
					type: 'error'
				});
			}
		}
	},
	mounted() {
		if (localStorage.getItem('rememberUser') === 'true') {
			//根据localStorage中rememberUser判断记住我的状态，如果为true则自动填写之前保存的账号密码
			this.remember = true;
			if (localStorage.getItem('authorize')) {
				let user = JSON.parse(localStorage.getItem('authorize'));
				this.username = user.username;
				this.password = user.password;
			}
		} else {
			this.remember = false;
		}
		if (sessionStorage.getItem('userInfo')) {
			sessionStorage.removeItem('userInfo');
		}

		if (sessionStorage.getItem('menus')) {
			sessionStorage.removeItem('menus');
		}

		if (sessionStorage.getItem('actk')) {
			sessionStorage.removeItem('actk');
		}
	}
};
</script>

<style>
@import "Login.css";
</style>
