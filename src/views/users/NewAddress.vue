<template>
  <div class="import-address bg-gray">
    <div class="bg-white"> </div>
    <div style="">
      <el-tabs v-model="activeName" @tab-click="handleClick" class="new_import w1200">
        <el-tab-pane :label="$t('importAddress.importAddress3')" name="keyImport">
          <div class="tab w1200 mt_30">
            <el-form :model="importKeyForm" :rules="importKeyRules" ref="importKeyForm" status-icon class="import-form w630">
              <el-form-item :label="$t('importAddress.importAddress5')" prop="keys">
                <el-input type="textarea" v-model.trim="importKeyForm.keys" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item :label="$t('newAddress.newAddress6')" prop="pass">
                <el-input v-model="importKeyForm.pass" type="password" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item :label="$t('newAddress.newAddress7')" prop="checkPass">
                <el-input v-model="importKeyForm.checkPass" type="password" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item class="form-bnt">
                <el-button type="success" @click="keyImport('importKeyForm')">{{$t('importAddress.importAddress8')}}
                </el-button>
              </el-form-item>
            </el-form>
          </div>
        </el-tab-pane>
        <el-tab-pane :label="$t('importAddress.importAddress0')" name="newAddress" >
          <div class="new_address">
            <el-form :model="passwordForm" status-icon :rules="passwordRules" ref="passwordForm"
                     class="w630">
              <el-form-item :label="$t('newAddress.newAddress6')" prop="pass">
                <el-input type="password" v-model="passwordForm.pass" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item :label="$t('newAddress.newAddress7')" prop="checkPass">
                <el-input type="password" v-model="passwordForm.checkPass" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item label="" prop="agreement">
                <el-checkbox-group v-model="passwordForm.agreement">
                  <el-checkbox :label="$t('newAddress.newAddress8')" name="agreement"></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
              <el-form-item class="form-bnt">
                 <el-button type="success" @click="submitPasswordForm('passwordForm')" :disabled="!passwordForm.agreement">
                   {{$t('newAddress.newAddress10')}}
                 </el-button>
              </el-form-item>
            </el-form>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
    <Password ref="password" @passwordSubmit="passSubmit">
    </Password>
  </div>
</template>

<script>
 // import nuls from 'nuls-sdk-js'
  import Password from '@/components/PasswordBar'
// import BackBar from '@/components/BackBar'
  import {copys, chainID, chainIdNumber, addressInfo, defaultAddressInfo} from '@/api/util'
  import {RUN_PATTERN, LOCALHOST_API_URL, PARAMETER} from '@/config.js'
  import axios from 'axios'

  export default {
    data() {
      let checkKey = (rule, value, callback) => {
        if (value === '') {
          callback(new Error(this.$t('importAddress.importAddress9')));
        } else {
          callback();
        }
      };
      let validatePass = (rule, value, callback) => {
        let patrn = /^(?![\d]+$)(?![a-zA-Z]+$)(?![^\da-zA-Z]+$).{8,20}$/;
        if (value === '') {
          //callback(new Error(this.$t('newAddress.newAddress22')));
            callback();
        } else if (!patrn.exec(value)) {
          callback(new Error(this.$t('newAddress.newAddress23')));
        } else {
          if (this.passwordForm.checkPass !== '') {
            this.$refs.passwordForm.validateField('checkPass');
          }
          callback();
        }
      };
      let validatePassTwo = (rule, value, callback) => {
        if (value === '') {
         //callback(new Error(this.$t('newAddress.newAddress24')));
           callback();
        } else if (value !== this.passwordForm.pass) {
          callback(new Error(this.$t('newAddress.newAddress25')));
        } else {
          callback();
        }
      };
      let validateCheckPass = (rule, value, callback) => {
        if (value === '') {
         // callback(new Error(this.$t('importAddress.importAddress12')));
           callback();
        } else if (value !== this.importKeyForm.pass) {
          callback(new Error(this.$t('importAddress.importAddress13')));
        } else {
          callback();
        }
      };
      return {
        activeName: 'keyImport',//tab选中
        isFirst: true,//第一步
        isBackups: false,//备份账户
        keyDialog: false, //key弹框
        ifAddressInfo: localStorage.hasOwnProperty(chainIdNumber),//判断是否账户地址
        passwordForm: {
          pass: '',
          checkPass: '',
          agreement: true,
        },
        importKeyForm: {
          key: '',
          pass: '',
          checkPass: ''
        },
        passwordRules: {
          pass: [
            {validator: validatePass, trigger: 'blur'}
          ],
          checkPass: [
            {validator: validatePassTwo, trigger: 'blur'}
          ],
          agreement: [
            {required: true, message: this.$t('newAddress.newAddress29'), trigger: 'change'}
          ]
        },
        importKeyRules: {
          pass: [
            {validator: validatePass, trigger: ['blur', 'change']}
          ],
          checkPass: [
            {validator: validateCheckPass, trigger: ['blur', 'change']}
          ],
          key: [
            {validator: checkKey, trigger: ['blur', 'change']}
          ]
        },
        newAddressInfo: {}, //新建的地址信息
        backType: 0,//备份类型 0：keystore备份 1：明文私钥备份
        RUN_PATTERN: RUN_PATTERN,//运行模式
      };
    },
    created() {
      let backAddressInfo = this.$route.query.backAddressInfo;
      if (backAddressInfo) {
        this.isFirst = false;
        this.isBackups = true;
        this.newAddressInfo.address = backAddressInfo.address;
        this.newAddressInfo.aesPri = backAddressInfo.aesPri;
      }
    },
    mounted() {
    },
    components: {
      Password
    },
    methods: {

      /**
       * @disc: tab选择
       * @params: tab
       * @date: 2019-08-31 16:18
       * @author: Wave
       */
      handleClick(tab) {
         if (tab.name === 'keyImport') {
          this.newAddressInfo = {};
          this.$refs['importKeyForm'].resetFields();
        } else {
          this.importAddressInfo = {};
          this.$refs['passwordForm'].resetFields();
        }
      },
      /**
       * 私钥导入
       * @param formName
       */
      keyImport(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.importWallet()
          } else {
            return false;
          }
        });
      },
      /**
       * 导入私钥方法
       */
      importWallet() {
        PARAMETER.method = 'importAccountByPriKey';
        PARAMETER.params = [chainID(), this.importKeyForm.keys, this.importKeyForm.pass, true];
        axios.post(LOCALHOST_API_URL, PARAMETER)
          .then((response) => {
            if (response.data.hasOwnProperty('result')) {
              let newImportAddressInfo = defaultAddressInfo;
              newImportAddressInfo.address = response.data.result.address;
               localStorage.setItem(chainIdNumber(), newImportAddressInfo.address);
              this.toUrl('address')
            }else{
                this.$message({message: this.$t('importAddress.importAddress18')+response.data.error.message, type: 'error', duration: 1000});
            }
          }).catch((err) => {
          console.log(err)
        });
      },
      /**
       * 创建地址
       * @param formName
       */
      submitPasswordForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            PARAMETER.method = 'createAccount';
            PARAMETER.params = [chainID(), this.passwordForm.pass];
            axios.post(LOCALHOST_API_URL, PARAMETER)
              .then((response) => {
                if (response.data.hasOwnProperty('result')) {
                  let newAddressInfo = defaultAddressInfo;
                  newAddressInfo.address = response.data.result.address;
                  newAddressInfo.pri = response.data.result.prikey;
                  this.newAddressInfo = newAddressInfo;
                   localStorage.setItem(chainIdNumber(), newAddressInfo.address);
                  this.$router.push({
                      name: "backupsAddress",
                      query: {'backAddressInfo': newAddressInfo}
                  })
                }else{
                this.$message({message: this.$t('newAddress.newAddress30')+response.data.error.message, type: 'error', duration: 1000});
                }
              }).catch((err) => {
              console.log(err)
            })
          } else {
            return false;
          }
        });
      },

      /**
       * 获取账户列表
       */
      async getAddressList() {
        let accountList = await addressInfo();
        if (accountList.length === 1) {
          localStorage.setItem(chainIdNumber(), this.newAddressInfo.address)
        }
      },

      /**
       * 备份keystore
       **/
      backKeystore() {
        this.backType = 0;
        this.$refs.password.showPassword(true);
      },

      /**
       * 备份私钥
       **/
      backKey() {
        this.backType = 1;
        this.$refs.password.showPassword(true)
      },

      /**
       *  获取密码框的密码
       * @param password
       **/
      passSubmit(password) {
        if (this.backType === 1) {
          PARAMETER.method = 'exportPriKeyByAddress';
          PARAMETER.params = [chainID(), this.newAddressInfo.address, password];
          axios.post(LOCALHOST_API_URL, PARAMETER)
            .then((response) => {
              if (response.data.hasOwnProperty('result')) {
                this.newAddressInfo.pri = response.data.result.privateKey;
                this.keyDialog = true;
              }else{
                this.$message({message: this.$t('newAddress.newAddress31') + response.data.error.message, type: 'error', duration: 1000});
              }
            }).catch((err) => {
            console.log(err);
          })
        }
      },

      /**
       * 进入钱包
       */
      goWallet() {
        this.toUrl('home');
      },

      /**
       * 连接跳转
       * @param name
       */
      toUrl(name) {
        //console.log(name)
        this.$router.push({
          name: name
        })
      },

      /**
       * 复制方法
       * @param sting
       **/
      copy(sting) {
        copys(sting);
        this.$message({message: this.$t('public.copySuccess'), type: 'success', duration: 1000});
        this.keyDialog = false;
      },
    }
  }
</script>

<style lang="less">
  @import "./../../assets/css/style";

  .import-address {
    .bg-white {
      height: 130px;
    }
    .new_import {
      margin: -90px auto 100px;
      .el-tabs__header {
        margin: 0;
        .el-tabs__nav-wrap {
          text-align: center;
          &:after {
            height: 1px;
          }
        }
        .el-tabs__nav-scroll {
          .el-tabs__nav {
            float: none;
            .el-tabs__active-bar {
              height: 0;
            }
            .el-tabs__item {
              text-align: center;
              padding: 0 25px;
              margin: 10px 20px 20px;
              border-radius: 4px;
              &:hover {
                background: linear-gradient(to right, #67C23A, #67C23A);
                color: #FFFFFF;
              }
            }
            .is-active {
              background: linear-gradient(to right, #67C23A, #67C23A);
              color: #FFFFFF;
            }
          }
        }
      }
      .el-tabs__content {
        background-color: #FFFFFF;
        .upload_keystore {
          padding: 100px 0 100px 0;
          border: 1px solid #E4E7ED;
        }

        .form-bnt {
          text-align: center;
          .el-button--success {
            width: 190px;
          }
        }

        .tab {
          border: 1px solid #E4E7ED;
          .import-form {
            margin: 60px auto 100px;
          }
        }
        .new_address {
          border: 1px solid #E4E7ED;
          .w630 {
            margin: 60px auto 100px;
          }
        }
      }
    }
  }

</style>
