<template>
    <div>
        <el-form ref="form" :model="form" label-width="100px">
            <el-form-item label="id" :hidden="true">
                <el-input v-model="form.memberId"></el-input>
            </el-form-item>
            <el-form-item label="账号（卡号）">
                <el-input ref="id_account" v-model="form.account" @keyup.enter.native="getMember"></el-input>
            </el-form-item>
            <el-form-item label="手机">
                <el-input v-model="form.phone" @keyup.enter.native="getMember"></el-input>
            </el-form-item>
            <el-form-item label="姓名" >
                <el-input v-model="form.name" :disabled="true"></el-input>
            </el-form-item>
            <el-form-item label="商户" >
                <el-input v-model="form.merchantName" :disabled="true"></el-input>
            </el-form-item>
            <el-form-item label="余额" >
                <el-input v-model="form.currMoney" :disabled="true"></el-input>
            </el-form-item>
            <el-form-item label="积分" >
                <el-input v-model="form.currIntegral" :disabled="true"></el-input>
            </el-form-item>
            <el-form-item label="付款方式">
                <!--1-余额 2-微信 3-支付宝 4-现金 5-刷卡 6-积分-->
                <el-select v-model="form.payway" placeholder="请选择付款方式">
                    <el-option label="余额" value="1"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="剩余积分">
                {{form.currIntegral}} - {{form.money}} * 10 = {{getIntegral}}  (<span style="color: #F56C6C">10积分 = ¥1.00</span>)
            </el-form-item>
            <el-form-item label="兑换金额">
                <el-input ref="id_money" v-model.number="form.money" @keyup.enter.native="pwdFocus"></el-input>
            </el-form-item>
            <el-form-item label="密码">
                <el-input ref="id_pwd" type="password" v-model="form.pwd" @keyup.enter.native="onSubmit"></el-input>
            </el-form-item>
            <el-form-item label="操作员" >
                <el-input v-model="form.operator" :disabled="true"></el-input>
            </el-form-item>
            <el-form-item label="备注">
                <el-input ref="id_content" v-model="form.content" type="textarea" ></el-input>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSubmit">兑换</el-button>
                <el-button @click="onCancel">取消</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                form: {
                    memberId: '',
                    account: '',
                    phone: '',
                    name: '',
                    merchantName: '',
                    money: '',
                    currMoney: '',
                    currIntegral: '',
                    payway: '1',
                    content: '积分兑换',
                    operator:''
                },
                // formEmpty: JSON.parse(JSON.stringify(this.form)),
                formEmpty: {
                    memberId: '',
                    account: '',
                    phone: '',
                    name: '',
                    merchantName: '',
                    money: '',
                    currMoney: '',
                    currIntegral: '',
                    payway: '1',
                    content: '积分兑换',
                    operator:''
                },

            }
        },
        created(){
            let user = localStorage.getItem('user') ;
            let userData =JSON.parse(user);
            this.form.operator = userData.nickname;
            this.formEmpty.operator = userData.nickname;
        },
        computed:{
            getIntegral(){
                return this.form.currIntegral - this.form.money * 10
            }
        },
        methods: {
            onSubmit() {
                if(this.form.money){

                }else{
                    return ;
                }
                //剩余积分
                let shengyu = this.form.currIntegral - this.form.money * 10;
                if(isNaN(shengyu)){
                    this.$alert("金额错误！", "失败！", {confirmButtonText: '确定',
                        callback: action => {
                            this.onCancel()
                            this.accountFocus()
                        }
                    });
                    return;
                }
                if(shengyu<0){
                    this.$alert("积分不足！", "失败！", {confirmButtonText: '确定',
                        callback: action => {
                            this.onCancel()
                            this.accountFocus()
                        }
                    });
                    return;
                }
                this.$http.post('busi.integralToMoney', this.form, this).then(respose => {
                    let data = respose.data
                    if("ok" == data.code){
                        //成功
                        this.$alert(data.msg, "成功！", {confirmButtonText: '确定',
                            callback: action => {
                                this.onCancel()
                                this.accountFocus()
                            }
                        });
                    }else{
                        //失败，进行提示
                        this.$alert(data.msg+data.data, "错误！", {confirmButtonText: '确定',
                            callback: action => {
                                this.onCancel()
                                this.accountFocus()
                            }});
                    }
                    //将焦点回到账号

                })

            },
            getMember() {

                let params = {}
                if(this.form.account){
                    params.account = this.form.account
                }else if(this.form.phone){
                    params.phone = this.form.phone
                }else {
                    return ;
                }
                // this.contentFocus();
                this.$http.post('member.get', params, this).then(respose => {
                    let data = respose.data
                    if("ok" == data.code){
                        //成功
                        if(data.data){
                            this.form.name = data.data.nickname
                            this.form.memberId = data.data.id
                            this.form.merchantName = data.data.merchantName
                            this.form.currMoney = data.data.money
                            this.form.currIntegral = data.data.integral
                            this.moneyFocus()
                        }else{
                            this.onCancel();
                            this.$alert("未查询到会员！", "错误！", {confirmButtonText: '确定',
                                callback: action => {
                                    // this.accountFocus()
                                }
                            });
                        }
                        //将焦点放到金额输入框

                    }else{
                        this.onCancel();
                        //失败，进行提示
                        this.$alert(data.msg+data.data, "错误！", {confirmButtonText: '确定',
                            callback: action => {
                                // this.accountFocus()
                            }
                        });
                    }

                })
            },
            onCancel(){
                this.form = JSON.parse(JSON.stringify(this.formEmpty))
            },
            moneyFocus(){
                this.$refs.id_money.focus()
            },
            accountFocus(){
                this.$refs.id_account.focus()
            },
            pwdFocus(){
                this.$refs.id_pwd.focus()
            },
        }
    }
</script>