<template>
<div style="padding-top: 30px;">
  <!-- <div class="campaignInfo">
    <div>Campaign: {{campaignName}}</div>
  </div> -->
  <div class="main-wrapper shadow-1" v-if="!isLoggedIn" style="max-width: 400px;">
    <div class="tab-content">
      <div class="tab-content-title" style="text-align: center;">LOG IN</div>
      <div class="log-in-wrapper">
        <form @submit="logIn">
          <mini-input type="text" label="Username" v-model="logInId" id="logInId" maxlength="100" color="#4285F4" fontColor="#444444" :error="LogInIdError" @validate="validateLogInId()"></mini-input>
          <!-- <input type="text" placeholder="Username" v-model="logInId" class="mini-input shadow-1" /> -->
          <mini-input type="password" label="Password" v-model="logInPassword" id="logInPassword" maxlength="100" color="#4285F4" fontColor="#444444" :error="LogInPasswordError" @validate="validateLogInPassword()">></mini-input>
          <!-- <input type="password" placeholder="Passowrd" v-model="logInPassword" class="mini-input shadow-1"/> -->
           <button class="primary" type="submit" v-if="!loggingIn" style="padding: 8px 20px;">Log In</button>
            <div class="form-loader" v-if="loggingIn"></div>
        </form>
      </div>
        
    </div>
  </div>
  <div class="main-wrapper shadow-1" v-if="isLoggedIn">
    <div class="tab-container">
      <div @click="currentTab=1" :class="{active:currentTab == 1}">{{translateWord('USER')}}</div>
      <div @click="currentTab=2" :class="{active:currentTab == 2}">{{translateWord('COUPON')}}</div>
    </div>
    <div class="tab-content-wrapper">
      <transition-group name="tab-transition">
      <div class="tab-content" v-if="currentTab==1" :key="1">
        <div class="tab-content-title">{{translateWord('getUserInfo')}}</div>
        <form @submit="getUser">
          <mini-input type="text" :label="wording.twitterID[lang]" v-model="userId" id="userId" maxlength="100" color="#4285F4" fontColor="#444444" :error="userIdError"></mini-input>
          <button class="primary" type="submit" v-if="!userSubmitting" style="padding: 8px 20px;margin-left: 20px;">{{translateWord('SUBMIT')}}</button>
          <div class="form-loader" v-if="userSubmitting"></div>
          <div class="user-info-wrapper">
            <div class="queryMsg" :class={error:userQueryError} v-html="userQueryMsg"></div>
            <div v-for="(val, key) in userInfo" :key="key">
              <user-data :lang="lang" :data="val" @reissue="reissueCoupon(val)" ref="userUser"></user-data>
            </div>
          </div>
        </form>
        
      </div>
      <div class="tab-content" v-if="currentTab==2" :key="2">
        <div class="tab-content-title">{{translateWord('getCouponInfo')}}</div>
        <form @submit="getCoupon">
          <mini-input type="text" :label="wording.couponCode[lang]" v-model="couponCode" id="couponCode" maxlength="100" color="#4285F4" fontColor="#444444" :error="couponCodeError"></mini-input>
          <button class="primary" type="submit" v-if="!couponSubmitting" style="padding: 8px 20px;margin-left: 20px;">{{translateWord('SUBMIT')}}</button>
          <div class="form-loader" v-if="couponSubmitting"></div>
        </form>
        <div class="coupon-info-wrapper">
          <div class="queryMsg" :class={error:couponQueryError} v-html="couponQueryMsg"></div>
          <div class="coupon-info-property-wrapper">
            <div class="coupon-info shadow-1" v-if="couponInfo.group">
              <div class="coupon-info-property">{{translateWord('claimed')}}: {{translateWord(couponInfo.claimed)}}</div>
              <!-- <div class="coupon-info-property">{{translateWord('group')}}: {{translateWord(couponInfo.group)}}</div> -->
              <div class="coupon-info-property">{{translateWord('owner')}}: {{translateWord(couponInfo.owner)}}</div>
              <div class="coupon-info-property">{{translateWord('issued')}}: {{translateWord(couponInfo.redeemed)}} </div>
              <div class="coupon-info-property">{{translateWord('issuedOn')}}: {{couponInfo.formattedDate}}</div>
              <div v-if="couponUserInfo.length > 0">
                <div class="coupon-info-property" style="margin-bottom: -2px; padding-bottom: 0px;">{{translateWord('ownerInfo')}}:</div>
                <div class="user-info-wrapper">
                  <div v-for="(val, key) in couponUserInfo" :key="key">
                    <user-data :lang="lang" :data="val" @reissue="reissueCoupon(val)" :white="true"></user-data>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition-group>
    </div>
  </div>
</div>
</template>

<script>
import axios from 'axios';
import MiniInput from '@/components/inputs/MiniInput';
import UserData from '@/components/data/UserData';

export default {
  components: {
    MiniInput,
    UserData
  },
  data() {
    return {
      isLoggedIn: false,
      logInId: '',
      LogInIdError: '',
      logInPassword: '',
      LogInPasswordError: '',
      loggingIn: false,
      params: [],
      wording: {
        'USER': {
          'en': 'USER',
          'ja': 'ユーザー'
        },
        'COUPON': {
          'en': 'COUPON',
          'ja': 'クーポン'
        },
        'getUserInfo': {
          'en': 'Get User Info',
          'ja': 'ユーザー情報を確認'
        },
        'getCouponInfo': {
          'en': 'Get Coupon Info',
          'ja': 'クーポン情報を確認'
        },
        'SUBMIT': {
          'en': 'SUBMIT',
          'ja': '確認'
        },
        'twitterID': {
          'en': 'Twitter ID',
          'ja': 'Twitter ID'
        },
        'twitterName': {
          'en': 'Twitter Name',
          'ja': 'Twitter 名'
        },
        'registeredOn': {
          'en': 'Registered On',
          'ja': '登録日時'
        },
        'couponCode': {
          'en': 'Coupon Code',
          'ja': 'クーポンコード'
        },
        'source': {
          'en': 'Source',
          'ja': '店舗'
        },
        'state': {
          'en': 'State',
          'ja': '状態'
        },
        'reissueCoupon': {
          'en': 'Reissue Coupon',
          'ja': 'クーポン再発行'
        },
        'claimed': {
          'en': 'Claimed',
          'ja': 'クーポン交換'
        },
        'group': {
          'en': 'Group',
          'ja': '店舗'
        },
        'owner': {
          'en': 'Owner',
          'ja': '保有しているユーザー'
        },
        'issued': {
          'en': 'Issued',
          'ja': '発行済み'
        },
        'issuedOn': {
          'en': 'Issued On',
          'ja': '発行日時'
        },
        'ownerInfo': {
          'en': 'Owner Info',
          'ja': '所有者情報'
        },
        'win': {
          'en': 'win',
          'ja': '当選'
        },
        'lose': {
          'en': 'lose',
          'ja': '落選者'
        },
        'FamilyMart': {
          'en': 'Family Mart',
          'ja': 'ファミリーマート'
        },
        'CircleK': {
          'en': 'Circle K',
          'ja': 'CKS'
        },
        'exchanged': {
          'en': 'True',
          'ja': '交換済み'
        },
        'notExchanged': {
          'en': 'False',
          'ja': '未交換'
        },
      },
      lang: 'ja',
      campaignName: 'BodyMainte',
      functionsDomain: 'https://us-central1-familymarto2o.cloudfunctions.net/twitter',
      apiDomain: 'https://api.mobileads.com',
      currentTab: 1,
      campaignId:'634501954374a87c6b6f4dde00493ded',
      adUserId: '4831',
      rmaId: '3',
      generalUrl: 'https://track.richmediaads.com/a/analytic.htm?rmaId={{rmaId}}&domainId=0&pageLoadId={{cb}}&userId={{adUserId}}&pubUserId=0&campaignId={{campaignId}}&callback=trackSuccess&type={{type}}&value={{value}}&uniqueId={{userId}}&customId={{source}}',
      timestamp: Date.now(),
      userId: '',
      userSubmitting: false,
      userIdError: '',
      userQueryMsg: '',
      userQueryError: false,
      userInfo: [],
      couponCode: '',
      couponCodeError: '',
      couponSubmitting: false,
      couponQueryMsg: '',
      couponQueryError: false,
      couponInfo: {},
      couponUserInfo: []
    }
  },
  methods: {
      canReissue(val) {
        return val.state == 'win' && !val.reissued && !val.reissuing
      },
      validateLogInId() {
        if (!this.logInId) {
          this.LogInIdError = 'Username is empty';
          return false;
        }
        else {
          this.LogInIdError = '';
          return true;
        }
      },
      validateLogInPassword() {
        if (!this.logInPassword) {
          this.logInPasswordError = 'Password is empty';
          return false;
        }
        else {
          this.logInPasswordError = '';
          return true;
        }
      },
      logIn(event) {
        event.preventDefault();
        if (this.logInId && this.logInPassword) {
          this.LogInIdError = '';
          this.LogInPasswordError = '';
          this.loggingIn = true;
          axios.post('https://us-central1-mtrainier-78bc8.cloudfunctions.net/twitter/adminLogIn', {
            username: this.logInId,
            password: this.logInPassword
          }).then((response) => {
            this.loggingIn = false;
            console.log(response);
            if (response.data.authorized == true) {
              this.isLoggedIn = true;
            }
            else {
              this.isLoggedIn = false;
              if (response.data.errors.username) {
                this.LogInIdError = response.data.errors.username;
              }
              if (response.data.errors.password) {
                this.LogInPasswordError = response.data.errors.password;
              }
            }
          }).catch((error) => {
            console.error(error);
            this.loggingIn = false;
          })
        }
        else {
          if (!this.logInId) {
            this.LogInIdError = 'Username is empty.';
          }
          if (!this.logInPassword) {
            this.LogInPasswordError = 'Password is empty';
          }
        }
      },
      getParams() {
        var query_string = {};
        var query = window.location.search.substring(1);
        var vars = query.split("&");
        for (var i=0;i<vars.length;i++) {
            var pair = vars[i].split("=");
            // If first entry with this name
            if (typeof query_string[pair[0]] === "undefined") {
                query_string[pair[0]] = pair[1];
            // If second entry with this name
            } else if (typeof query_string[pair[0]] === "string") {
                var arr = [ query_string[pair[0]], pair[1] ];
                query_string[pair[0]] = arr;
            // If third or later entry with this name
            } else {
                query_string[pair[0]].push(pair[1]);
            }
        } 
        return query_string;
    },
    formatDate(time) {
      var months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      var date = new Date(time);
      var day = (date.getDate()).toString();
      var month = months[date.getMonth()].toString();
      var year = (date.getFullYear()).toString();
      var hours = (date.getHours()).toString();
      var minutes = (date.getMinutes()).toString();
      minutes = minutes.length < 2 ? '0' + minutes : minutes;
      return `${day}-${month}-${year} ${hours}:${minutes}`;
    },
    trackReissue(userId, oldCoupon, newCoupon, source) {
      var trackingUrl = this.generalUrl.replace('{{rmaId}}', this.rmaId).replace('{{campaignId}}', this.campaignId).replace('{{adUserId}}', this.adUserId).replace('{{cb}}', this.timestamp.toString());
      var type = 'coupon_reissue';
      var value = `${userId}_${oldCoupon}_${newCoupon}`;
      var url = trackingUrl.replace('{{type}}', type).replace('{{value}}', value).replace('{{userId}}', userId).replace('{{source}}', source);
      return axios.get(url);
    },
    getUserError() {
      this.userSubmitting = false;
      this.userQueryMsg = 'Error getting user info';
      this.userQueryError = true;
    },
    translateWord(word) {
      if (this.wording[word]) {
        return this.wording[word][this.lang];
      }
      else {
        return word;
      }
    },
    processUserData(data, isTwitterId) {
      var newData = Object.assign(data, {});
      newData.reissuing = false;
      newData.reissued = false;
      newData.reissueResult = '';
      if (data.state == 'win') {
        var couponLink = this.generateCouponLink(data.id, data.source);
        newData.couponLink = `<a href="${couponLink}" target="_blank"> ${data.couponCode}</a>`;
      }
      else {
        newData.couponLink = '';
      }
      newData.formattedDate = this.formatDate(data.dateCreated);
      if (data.dateMarked) {
        newData.formatMarkedDate = this.formatDate(data.dateMarked);
      }
      if (isTwitterId) {
        newData.isTwitterId = true;
        newData.twitterName = 'loading...';
        axios.post(this.functionsDomain + '/getTwitterName', {
          id: data.id
        }).then((res) => {
          newData.twitterName = res.data.screen_name;
        }).catch((err) => {
          console.error(err);
        });
      }
      return newData;
    },
    getUser2(userId) {
      return new Promise ((resolve, reject) => {
        axios.get(this.apiDomain + '/coupons/mtRainier/check_user', {
          params: {
            id: userId
          }
        }).then((response) => {
          this.userSubmitting = false;
          if (response.data.user.length > 0) {
            for (var r = 0; r < response.data.user.length; r++) {
              response.data.user[r] = this.processUserData(response.data.user[r], false);
            }
            this.userInfo = response.data.user;
            if (this.lang == 'ja') {
              this.userQueryMsg = 'ユーザーが存在しています';
            }
            else{
              this.userQueryMsg = 'User Found!';
            }
            this.userQueryError = false;
            resolve(this.userInfo);
          }
          else {
            this.userInfo = [];
            if (this.lang == 'ja') {
              this.userQueryMsg = 'ユーザーが見つかりません';
            }
            else {
              this.userQueryMsg = 'User Not Found';
            }
            
            this.userQueryError = true;
            resolve([]);
          }
        }).catch((error) => {
          this.getUserError();
          console.error(error);
          reject(error);
        });
      });
    },
    getUser(event) {
      event.preventDefault();
      if (this.userId) {
        this.userIdError = '';
        this.userSubmitting = true;
        if (this.userId.indexOf('@') > 0) {
          // email
          console.log('email');
          this.getUser2(this.userId);
        }
        else {
          //twitter
          console.log('twitter');
          var isTwitterId = /^\d+$/.test(this.userId);
          if (!isTwitterId) {
            if (this.userId.indexOf('@') > -1) {
              this.userId = this.userId.replace('@', '');
            }
            axios.post(this.functionsDomain + '/getTwitterId', {
              id: this.userId
            })
            .then((res) => {
              this.getUser2(res.data.id).then((response) => {
                if (response.length > 0) {
                  if (this.lang == 'ja') {
                    this.userQueryMsg = `ユーザーが存在しています <br>Twitter 名: ${res.data.screen_name}`;
                  }
                  else {
                    this.userQueryMsg = `User Found! <br>Twitter Name: ${res.data.screen_name}`;
                  }
                }
              });
            }).catch((err) => {
              console.error(error);
              this.getUserError();
            });
          }
          else {
            this.getUser2(this.userId).then((response) => {
              if (response.length > 0) {
                axios.post(this.functionsDomain + '/getTwitterName', {
                  id: response[0].id
                }).then((res) => {
                  if (this.lang == 'ja') {
                    this.userQueryMsg = `ユーザーが存在しています <br>Twitter 名: ${res.data.screen_name}`;
                  }
                  else {
                    this.userQueryMsg = `User Found! <br>Twitter Name: ${res.data.screen_name}`;
                  }
                })
              }
            }).catch((error) => {
              this.getUserError();
            });
          }
        }

      }
      else {
        this.userIdError = 'Input is empty.'
      }
    },
    generateCouponLink(userId, source) {
      return 'https://s3.amazonaws.com/rmarepo/o2o/MtRainier/coupon.html?userId=' + userId + '&source=MtRainier';
    },
    reissueCoupon(data) {
      data.reissuing = true;
      var userId = data.id;
      var source = data.source;
      var group = 'A'
      axios.post(this.apiDomain + `/coupons/mtRainier/reissue?id=${userId}&source=${source}&group=${group}`).then((response) => {
        if (response.data.status) {
          data.reissuing = false;
          data.reissued = true;
          data.reissueResult = 'Coupon Reissued!';
          // this.trackReissue(userId, data.couponCode, response.data.newCouponCode, source);
          var couponLink = this.generateCouponLink(data.id, data.source);
          data.couponLink = `<a href="${couponLink}" target="_blank"> ${response.data.newCouponCode}</a>`;
          data.couponCode = response.data.newCouponCode;
        }
        else {
          data.reissuing = false;
        }
      }).catch((error) => {
        console.error(error);
        data.reissuing = false;
      })
    },
    getCoupon(event) {
      event.preventDefault();
      if (this.couponCode) {
        this.couponCodeError = '';
        axios.get(this.apiDomain + '/coupons/mtRainier/check_coupon', {
          params: {
            couponCode: this.couponCode
          }
        }).then((response) => {
          if (response.data.coupon) {
            this.couponQueryError = false;
            if (this.lang == 'ja') {
              this.couponQueryMsg = 'クーポンが存在しています';
            }
            else {
              this.couponQueryMsg = 'Coupon Found!';
            }
            if (!response.data.coupon.claimed) {
              response.data.coupon.claimed = false;
            }
            response.data.coupon.claimed = response.data.coupon.claimed ? 'exchanged' : 'notExchanged';
            response.data.coupon.redeemed = response.data.coupon.redeemed ? 'Yes' : 'No';
/*            if (response.data.coupon.group == 'A') {
              response.data.coupon.group = 'FamilyMart';
            }
            if (response.data.coupon.group == 'B') {
              response.data.coupon.group = 'CircleK'
            }
            */
            if (response.data.coupon.date) {
              response.data.coupon.formattedDate = this.formatDate(response.data.coupon.date);
            }
            
            this.couponInfo = Object.assign({}, response.data.coupon);
            if (response.data.user.length > 0) {
              for (var u = 0; u < response.data.user.length; u++) {
                response.data.user[u] = this.processUserData(response.data.user[u], true);
              }
              this.couponUserInfo = response.data.user;
            }
            else {
              this.couponUserInfo = [];
            }
          }
          else {
            this.couponInfo = {};
            this.couponQueryError = true;
            if (this.lang == 'ja') {
              this.couponQueryMsg = 'クーポンは見つかりません';
            }
            else {
              this.couponQueryMsg = 'Coupon Not Found';
            }
            
          }
        }).catch((error) => {
          console.error(error);
        })
      }
      else {
        this.couponCodeError = 'Input is empty.'
      }
    }
  },
  mounted() {
    this.params = this.getParams();
    if (this.params.lang) {
      this.lang = this.params.lang == 'en' ? 'en' : 'ja' ;
    }
  }
}
</script>

<style>
  .tab-transition-enter-active, .tab-transition-leave-active {
    transition: all 0.3s;
  }

  .tab-transition-leave-active {
    position: absolute;
  }

  .tab-transition-enter, .tab-transition-leave-to {
    transform: translateY(30px);
    opacity: 0;
  }

  .mini-input {
    display: block;
    width: 100%;
    max-width: 250px;
    padding: 8px;
    border-radius: 2px;
    border: none;
    font-size: 14px;
    margin: auto;
    margin-bottom: 20px;
  }

.mini-input:focus {
    -webkit-box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
  box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
}
  .campaignInfo {
    margin: 25px 0px;
  }

  .main-wrapper {
    max-width: 600px;
    margin: auto;
  }

  .tab-container {
    display: flex;
    justify-content: space-around;
  }

  .tab-container div {
    padding: 10px;
    width: 50%;
    text-align: center;
    font-weight: 600;
    color: #888;
    position: relative;
    transition: all 0.3s;
  }

  .tab-container div::after {
    content: "";
    width: 96%;
    height: 4px;
    background: #e82f22;
    position: absolute;
    bottom: 0px;
    left: 2%;
    z-index: 2;
    transition: all 0.3s;
    transform: scaleX(0);
  }

  .tab-container div:hover, .tab-container div.active{
    cursor: pointer;
    color: #e82f22;
  }

  .tab-container div:hover::after, .tab-container div.active::after {
    transform: scaleX(1);
  }

  .tab-content-wrapper {
    border-top: 1px solid #aaa;
  }

  .tab-content {
    padding: 20px 10px;
  }

  .tab-content-title {
    margin-left: 20px;
    font-size: 18px;
    font-weight: 600;
    color:#666;
    margin-bottom: 20px;
  }

  .log-in-wrapper {
    max-width: 300px;
    text-align: center;
    margin: auto;
  }
 
  .queryMsg {
    font-weight: 600;
    color: #4285F4;
    margin-left: 5px;
    margin-bottom: 10px;
  }

  .queryMsg.error {
    color: red;
  }

  .user-info-wrapper, .coupon-info-wrapper {
    margin: 15px;
  }

  .user-info, .coupon-info {
    margin: 5px 5px 15px;
    padding: 4px 10px;
    background-color: #fafafa;
  }

  .user-info-property, .coupon-info-property {
    padding: 2px;
    transition: all 0.3s;
  }
/*.user-info-property:nth-child(odd), .coupon-info-property:nth-child(odd) {
  background-color: #eee;
}

.user-info-property:nth-child(even), .coupon-info-property:nth-child(even) {
  background-color: #ddd;
}*/
  .coupon-info .user-info {
    background-color: white;
  }

  .reissued {
    background-color: #7fff00;
  }
</style>

