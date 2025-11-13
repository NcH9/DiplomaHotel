<template>
    <div class="grid1">
        <h2 class="flex_center">{{ $t('auth.login') }}</h2>
        
        <el-form
            :model="form"
            :rules="rules"
            ref="loginForm"
            label-position="top"
            class="login"
            @submit.native.prevent="validateForm"
        >
            <el-form-item :label="$t('auth.email')">
                <el-input
                    v-model.lazy="form.email"
                    type="email"
                    @blur="validateEmail"
                ></el-input>
                <p class="error" v-if="error.email">{{ error.email }}</p>
            </el-form-item>

            <el-form-item :label="$t('auth.password')">
                <el-input
                    v-model.lazy="form.password"
                    type="password"
                    @blur="validatePassword"
                ></el-input>
                <p class="error" v-if="error.password">{{ error.password }}</p>
            </el-form-item>

            <el-form-item>
                <el-button type="primary" @click="validateForm">{{ $t('auth.action_login') }}</el-button>
            </el-form-item>

            <p class="error" v-if="errorMsg">{{ errorMsg }}</p>
            <p class="success" v-if="successMessage">{{ successMessage }}</p>
        </el-form>
    </div>
</template>

<script>
import { ref } from 'vue';
import axiosInstance from '@/api/axios.js';
import formValidation from '@/mixins/validator.js';
import router from '@/router/index.js';
export default {
    name: 'Login',
    mixins: [formValidation],
    setup() {
        const form = ref ({
            email: '',
            password: '',
        });
        const errorMsg = ref('');
        const successMessage = ref('');
        const { error, validatePassword, validateEmail, resetError } = formValidation();

        function validateForm() {
            resetError();
            validateEmail(form.value.email);
            validatePassword(form.value.password);
            console.log(error.value)
            if (!error.value.email && !error.value.password){
                loginUser();
            }
        }

        const loginUser = async () => {
            document.getElementById('loginbtn').disabled = true;
            try {
                errorMsg.value = '';
                successMessage.value = "";

                const response = await axiosInstance.post("/login", form.value);
                successMessage.value = "Login successful!";
                localStorage.setItem('authToken', response.data.token);

                router.push({ name: 'home' }).then(() => {
                    window.location.reload();
                });
            } catch (error) {
                if (error.response && error.response.status === 422) {
                    errorMsg.value = error.response.data.message;
                } else {
                    console.error("An unexpected error occurred:", error);
                }
            } finally {
                if (successMessage.value !== "Login successful!") {
                    document.getElementById('loginbtn').disabled = false;
                }
            }
        }

        return {
            form, errorMsg, successMessage, error,
            loginUser, validateForm, validatePassword, validateEmail, resetError
        }
    },
    
}
</script>