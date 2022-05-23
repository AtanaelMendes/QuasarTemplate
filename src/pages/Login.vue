<template>
	<q-layout view="hHh Lpr lFf">
		<q-page-container class="background">
			<q-page>
				<div class="row justify-center items-center" style="height: 100vh">
					<div class="col-xs-10 col-sm-5 col-md-4 col-lg-3"></div>
				</div>
			</q-page>
		</q-page-container>

		<q-dialog minimized v-model="passwordRecoveryModalOpened" @hide="clearResetPasswordForm()">
			<div class="col-xs-10 col-sm-8 col-md-6 col-lg-4 q-pa-lg" style="width: 400px">

				<span>Digite seu email para receber um link de acesso para criar uma nova senha.</span>

				<div v-if="passwordRecoveryModalOpened">
					<form @keyup.enter="login">

						<custom-input-text type="email" placeholder="Digite seu email" label="Email" :model="resetPasswordForm.email" />

					</form>
				</div>

				<div align="end">
					<q-btn @click="closePasswordRecoveryModal" color="primary" flat label="Cancelar" class="q-mr-sm"/>
					<q-btn @click="submitPasswordRecoveryForm" color="primary" label="recuperar"/>
				</div>

			</div>
		</q-dialog>

	</q-layout>
</template>


<script>
	import { required, email, minLength } from 'vuelidate/lib/validators'
	import { Loading } from 'quasar'
	import AuthService from "../assets/js/services/AuthService";
	import PasswordCredential from "../assets/js/auth/PasswordCredential";

	export default {
		name: 'login',
		components: {
		},
		data () {
			return {
				authService: new AuthService(),
				form: {
					email: {
						value: null,
						errorMessage: null
					},
					password: {
						value: null,
						errorMessage: null
					},
				},
				resetPasswordForm: {
					email: {
						value: null,
						errorMessage: null
					},
				},
				passwordRecoveryModalOpened: false,

			}
		},

		validations: {
			form: {
				email: { value: { required, email } },
				password: { value: {required, minLength: minLength(8) } }
			},
			resetPasswordForm: {
				email: { value: { required, email } }
			}
		},
		created() {
			let token = localStorage.getItem('auth.token');

			if (token != null) {
				this.$router.push('/admin')
			}
		},
		methods: {
			openPasswordRecoveryModal: function(){
				this.passwordRecoveryModalOpened = true;
			},
			closePasswordRecoveryModal: function(){
				this.passwordRecoveryModalOpened = false
			},
			clearResetPasswordForm : function(){
				this.resetPasswordForm.email.value = null;
				this.resetPasswordForm.email.errorMessage = null;
			},
			submitPasswordRecoveryForm: function () {
				this.$v.resetPasswordForm.$touch();

				if (this.$v.resetPasswordForm.$error ) {

					if(!this.$v.resetPasswordForm.email.value.required){
						this.resetPasswordForm.email.errorMessage = "Digite um email"
					}else if(!this.$v.resetPasswordForm.email.value.email){
						this.resetPasswordForm.email.errorMessage = "Este email é inválido."
					}
					return;
				}
			},
			login: function () {
				this.$v.form.$touch();

				if (this.$v.form.$error) {

					if(!this.$v.form.email.value.required){
						this.form.email.errorMessage = "Digite um email"
					}else if(!this.$v.form.email.value.email){
						this.form.email.errorMessage = "Este email é inválido."
					}

					if(!this.$v.form.password.value.required){
						this.form.password.errorMessage = "Digite uma senha."
					}else if(!this.$v.form.password.value.minLength){
						this.form.password.errorMessage = "A senha deve ter no mínimo 8 caracteres."
					}

					return
				}

				Loading.show();

				this.authService.login(new PasswordCredential(this.form.email.value, this.form.password.value)).then(()=> {
					this.$router.push( '/admin' );
					Loading.hide();
				}).catch(error => {
					this.showErrorDialog(error.message);
					Loading.hide();
				});

			},
			showErrorDialog: function (message) {
				this.$q.dialog({
					title: 'Ops!',
					message: message,
					ok: true,
				})
			}
		}
	}
</script>
<style>
	/*.background {*/
	/*	background-image: url('/statics/images/login-background.jpg');*/
	/*	background-position: center center;*/
	/*	background-repeat: no-repeat;*/
	/*	background-attachment: fixed;*/
	/*	background-size: cover;*/
	/*}*/
</style>
