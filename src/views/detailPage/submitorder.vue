<template>
    <div class="submitorder">
        <!-- 头部 -->
        <yd-navbar :title="title">
            <a slot="left" @click="goBack">
                <yd-navbar-back-icon></yd-navbar-back-icon>
            </a>
        </yd-navbar>
        <!-- 地址 -->
        <div class="center" v-if="tAddress.length===0? true : false" @click="address">
            <img class="img1" src="../../assets/img/add.png" alt="">
            <span class="add">填写收货地址</span>
        </div>
        <div class="center1" v-if="tAddress.length!=0" @click="pushPay">
            <div class="sub-header">
                <div class="header-l">收货人:{{tAddress.name}}</div>
                <div class="header-r"> {{tAddress.phonenumber}}</div>
            </div>
            <div class="sub-header">
                <div class="header-l" style="font-size:12px">收货地址：{{tAddress.provcityarea || ""}}
                    {{tAddress.detailaddress || ""}}
                </div>
            </div>
            <p>
                (请仔细检查您的收货地址）
            </p>
        </div>
        <!-- 商品 -->
        <div class="shop" v-for="item in carList" :key="item.id">
            <div class="shop-header">
                <div style="width: 80px;height: 80px;">
                    <img v-lazy="item.image" style="border-radius: 6px" width="80" height="80"/>
                </div>
                <div class="header-r">
                    <p class="q">{{item.name}}</p>
                    <p class="a">{{item.formname}}</p>
                    <p class="s">¥ {{((item.price * item.count)-(item.pointcount*item.pointmaxmoney)).toFixed(2) + " (运费:"+item.freight.toFixed(2)+")"}}</p>
                </div>
            </div>
           <!-- <div class="he">
                <yd-spinner unit="1" v-model="item.count"></yd-spinner>
            </div>-->
        </div>
        <!-- 支付按钮 -->
        <div class="footer">
            <div class="footer-l">付款: <span class="money">
                ￥{{orderMoney.toFixed(2)}}
                <span v-if="orderPoint!=0">+ {{orderPoint}}积分</span>
                <span v-if="oFreight!=0">(含运费:{{oFreight.toFixed(2)}})</span>
            </span>
            </div>
            <div class="footer-r" v-if="orderMoney!=0" @click="selectPay">选择支付</div>
            <div class="footer-r" v-if="orderMoney==0" @click="integralPay">积分支付</div>
        </div>
        <!-- 支付弹框 -->
        <yd-popup v-model="show1" position="bottom" height="50%">
            <div class="line">
                <div class="line-header">
                    <h1 class="line-title">选择付款方式</h1>
                    <img class="imgti" src="../../assets/img/X.png" alt="" @click="show1=false">
                </div>
                <div class="line-header" v-if="!isWx" style=" justify-content:space-between;" @click="change('zfb')">
                    <img src="../../assets/img/zhifubao.png" alt="" width="24" height="24">
                    <div class="img-r">
                        <h1 class="pay">支付宝支付</h1>
                        <h2 :class="{c:true, d:payType=='zfb'}"></h2>
                    </div>
                </div>
                <div class="line-header" style="justify-content:space-between;" @click="change('wx')">
                    <img src="../../assets/img/weixin122.png" alt="" width="24" height="24">
                    <div class="img-r">
                        <h1 class="pay">微信支付</h1>
                        <h2 :class="{c:true, d:payType=='wx'}"></h2>
                    </div>
                </div>
<!--                <div class="line-header" style="justify-content:space-between;" @click="change('hb')">-->
<!--                    <img src="../../assets/img/ysf.png" alt="" width="24" height="24">-->
<!--                    <div class="img-r">-->
<!--                        <h1 class="pay">混编支付</h1>-->
<!--                        <h2 :class="{c:true, d:payType=='hb'}"></h2>-->
<!--                    </div>-->
<!--                </div>-->
                <div class="footer-r" @click="pay">支付</div>
            </div>
        </yd-popup>

        <yd-keyboard v-model="showhb" :callback="hbPay" title="商城专用安全键盘" disorder ref="keyboardDemo2"></yd-keyboard>


        <!-- <transition name="slide-fade">
            <div class="ceng" v-if="show1">
                <div class="line">
                    <div class="line-header">
                        <h1 class="line-title">选择付款方式</h1>
                        <img class="imgti" src="../../assets/img/X.png" alt="" @click="show1=false">
                    </div>
                    <div class="line-header" style=" justify-content:space-between;" @click="change('zfb')">
                        <img src="../../assets/img/zhifubao.png" alt="" width="24" height="24">
                        <div class="img-r">
                            <h1 class="pay">支付宝支付</h1>
                            <h2 :class="{c:true, d:payType=='zfb'}"></h2>
                        </div>
                    </div>
                    <div class="line-header" style="justify-content:space-between;" @click="change('wx')">
                        <img src="../../assets/img/weixin122.png" alt="" width="24" height="24">
                        <div class="img-r">
                            <h1 class="pay">微信支付</h1>
                            <h2 :class="{c:true, d:payType=='wx'}"></h2>
                        </div>
                    </div>
                </div>
            </div>
     </transition> -->
    </div>
</template>
<script>
    export default {
        data() {
            return {
                title: "提交订单",
                id: this.$route.params.id,
                flag: false,
                show1: false,
                showhb: false,
                isWx: false,
                payType: 'zfb',
                number: 1,
                carList: [], //购物车选中的商品
                add: [],
                tAddress: this.$store.state.tAddress.length === 0 ? [] : this.$store.state.tAddress,
                oFreight: 0, //运费
                orderMoney:0,//需要支付金额
                orderPoint:0,//需要积分
                oPoontbanalce:0,//积分余额
                pAddressID: this.$store.state.tAddress.length <= 0 ? 0 : this.$store.state.tAddress[0].id, //地址id
                pDetails: "", //.net的特色集合
                isPay: false, //是否可以支付
            }
        },
        computed: {},
        beforeRouteEnter(to, from, next) {
            next(vm => {
                vm.$parent.$data.showfooter = false;
                vm.$parent.$data.showmenu = false;
            });
        },
        beforeRouteLeave(to, from, next) {
            this.$parent.$data.showfooter = true;
            this.$parent.$data.showmenu = true;
            next()
        },
        computed: {
            totalPrice() {
                let total = 0;
                for (let key in this.carList) {
                    total += this.carList[key].price * this.carList[key].count
                }
                return total
            }
        },
        created() {
            //判断是否是微信浏览器
            var ua = window.navigator.userAgent.toLowerCase();
            if (ua.match(/MicroMessenger/i) == 'micromessenger') {
                this.isWx = true;
                this.payType = "wx";
            }
            //加载购物车选中的数据
            this.loadCarList();
        },
        methods: {
            //下单
            pay() {
                //获取浏览器标识
                //判断是否选择了地址
                if (!this.isPay || this.pAddressID == 0) {
                    this.$dialog.toast({
                        mes: "请查检是否填写收货地址",
                        timeout: 1000
                    });
                    return;
                }
                this.$dialog.loading.open('正在调起支付,请稍等');
                let param = {};
                //判断是直接购买还是购物车
                if (localStorage.getItem("xdType") == "gwcxd") {
                    //购物车购买
                    param = {
                        "TransType": "GWCXD",
                        "pDetails": this.pDetails,
                        "pAddressID": this.pAddressID,
                        "body": "啊啊啊啊",
                        "ip": ""
                    };
                } else if (localStorage.getItem("xdType") == "zjxd") {
                    //直接购买
                    param = {
                        "TransType": "ZJXD",
                        "pGoodsFormID": this.carList[0].goodsFormsId, //商品规格id
                        "pSecondclass": this.carList[0].secondclass, //二级分类
                        "pPointcount": this.carList[0].pPointcount, //积分抵扣数量
                        "pCount": this.carList[0].count, //pCount
                        "body": "啊啊啊啊",
                        "pAddressID": this.pAddressID,
                        "ip": ""
                    };
                }
                //判断是否是微信浏览器
                if (this.isWx) {
                    if (this.payType === "hb") {
                        //混编支付
                        //移除loading
                        this.$dialog.loading.close();
                        this.showhb = true;
                    } else {
                        //公众号支付
                        this.payType = "wx";
                        this.show1 = false;
                        this.publicCodePay(param);
                    }
                } else {
                    //this.$dialog.loading.open('正在调起支付,请稍等');
                    if (this.payType === "wx") {
                        //H5支付
                        this.$axios.post("/pay/h5", param).then((result) => {
                            //移除loading
                            this.$dialog.loading.close();
                            if (result != null && result.state == "00") {
                                location.href = result.payUrl + "&redirect_url=" + encodeURIComponent("http://mall.zybkeji.com/#/myorder");
                            } else {
                                this.$dialog.toast({
                                    mes: result.message,
                                    timeout: 1500
                                });
                            }
                        });
                    } else if (this.payType === "zfb") {
                        this.$axios.post("/pay/zfb", param).then((result) => {
                            //移除loading
                            this.$dialog.loading.close();
                            document.write(result)
                        });
                    } else if (this.payType === "hb") {
                        //混编支付
                        //移除loading
                        this.$dialog.loading.close();
                        this.showhb = true;
                    }
                }
            },
            hbPay() {
                this.$dialog.loading.open('正在调起混编支付');
                setTimeout(() => {
                    //移除loading
                    this.$dialog.loading.close();
                    this.$dialog.toast({
                        mes: "啊,我失败了!",
                        timeout: 1000
                    });
                }, 1000);
            },
            //加载数据
            loadCarList() {
                this.carList = JSON.parse(localStorage.getItem("lCarList"));
                for (let i = 0; i < this.carList.length; i++) {
                    if (localStorage.getItem("xdType") == "gwcxd") {
                        this.pDetails += this.carList[i].id + "," + this.carList[i].count +","+this.carList[i].pointcount + (i >= this.carList.length - 1 ? "" : "|");
                    } else if (localStorage.getItem("xdType") == "zjxd") {
                        this.pDetails += this.carList[i].goodsFormsId + "," + this.carList[i].count +","+this.carList[i].pointcount + (i >= this.carList.length - 1 ? "" : "|");
                    }
                }
                //如果地址不为空
                //if(this.pAddressID!=0){
                //计算运费
                let param = {
                    "TransType": "JSYF",
                    "pDetails": this.pDetails,
                    "pAddressID": this.pAddressID
                };
                this.$axios.post("/send/getData", param).then((result) => {
                    if (result != null) {
                        if (result.state == "00") {
                            this.tAddress = result.addressList.length === 0 ? [] : result.addressList[0];
                            this.pAddressID = result.addressList.length == 0 ? 0 : result.addressList[0].id;
                            this.isPay = true;
                            this.oPoontbanalce=result.oPoontbanalce;
                            this.orderMoney=result.orderMoney;
                            this.orderPoint=result.orderPoint;
                            this.oFreight = result.oFreight;
                        } else {
                            this.$dialog.toast({
                                mes: result.message,
                                timeout: 1500
                            });
                        }
                    }
                });
            },
            goBack() {
                if(this.$route.params.backUrl=="reservepage"){
                    this.$router.go(-1);
                }else {
                    this.$router.push({
                        name: "car"
                    });
                }
            },
            address() {
                this.$router.push({
                    name: 'addressmanage'
                });
            },
            //积分支付
            integralPay(){
                if (!this.isPay || this.pAddressID == 0) {
                    this.$dialog.toast({
                        mes: "请查检是否填写收货地址",
                        timeout: 1000
                    });
                    return;
                }
                if(this.oPoontbanalce<this.orderPoint){
                    this.$dialog.toast({
                        mes: "积分余额不足",
                        timeout: 1500
                    });
                    return;
                }
                this.$dialog.loading.open('正在调起支付,请稍等');
                let param = {};
                if (localStorage.getItem("xdType") == "gwcxd") {
                    //购物车购买
                    param = {
                        "TransType": "GWCXD",
                        "pDetails": this.pDetails,
                        "pAddressID": this.pAddressID,
                        "body": "啊啊啊啊",
                        "ip": ""
                    };
                } else if (localStorage.getItem("xdType") == "zjxd") {
                    //直接购买
                    param = {
                        "TransType": "ZJXD",
                        "pGoodsFormID": this.carList[0].goodsFormsId, //商品规格id
                        "pSecondclass": this.carList[0].secondclass, //二级分类
                        "pCount": this.carList[0].count, //pCount
                        "body": "啊啊啊啊",
                        "pAddressID": this.pAddressID,
                        "pPointcount":this.carList[0].pointcount,
                        "ip": ""
                    };
                }
                //积分支付
                this.$axios.post("/pay/integral", param).then((result) => {
                    //移除loading
                    this.$dialog.loading.close();
                    if (result != null && result.state == "00") {
                        this.$dialog.toast({
                            mes: "支付成功,1秒后自动去订单页面",
                            timeout: 1500
                        });
                        setTimeout(function () {
                            window.location.href = "/#/myorder";
                        },1000);
                    } else {
                        this.$dialog.toast({
                            mes: result.message,
                            timeout: 1500
                        });
                    }
                });

            },
            selectPay() {
                this.show1 = true;
            },
            change(i) {
                //this.show1 = false;
                this.payType = i;
            },
            addshop(index) {
                this.carList[index].count++
            },
            reduce(index) {
                this.carList[index].count === 1 ? 1 : this.carList[index].count--
            },
            pushPay() {
                this.$router.push({
                    name: "addressmanage"
                })
            },
            //公众号支付方法
            async publicCodePay(params) {
                //
                let data = await this.$axios.post("/pay/jsapi", params);
                if (data.state != "00") {
                    this.$dialog.toast({
                        mes: data.msg,
                        timeout: 1500
                    });
                    //移除loading
                    this.$dialog.loading.close();
                    return;
                } else {
                    //移除loading
                    this.$dialog.loading.close();
                }
                let onBridgeReady = () => {
                    WeixinJSBridge.invoke(
                        'getBrandWCPayRequest', {
                            "appId": data.appid, //公众号名称，由商户传入
                            "timeStamp": data.timeStamp, //时间戳，自1970年以来的秒数
                            "nonceStr": data.nonceStr, //随机串
                            "package": data.packageValue,
                            "signType": "MD5", //微信签名方式：
                            "paySign": data.paySign //微信签名
                        }, (res) => {
                            //支付成功
                            if (res.err_msg == "get_brand_wcpay_request:ok") {
                                /*this.$dialog.toast({
                                    mes: "支付成功!",
                                    icon: 'success',
                                    timeout: 1500
                                });*/
                                //延迟跳转页面
                                window.location.href = "/#/myorder";
                                // 使用以上方式判断前端返回,微信团队郑重提示：
                                //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
                            }
                        });
                };
                if (typeof WeixinJSBridge == "undefined") {
                    if (document.addEventListener) {
                        document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
                    } else if (document.attachEvent) {
                        document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
                        document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
                    }
                } else {
                    onBridgeReady();
                }
            }
        },
        components: {
            //     'remote-js'
            // :
            //     {
            //         render(createElement)
            //         {
            //             return createElement('script', {attrs: {type: 'text/javascript', src: this.src}});
            //         }
            //     ,
            //         props: {
            //             src: {
            //                 type: String, required
            //             :
            //                 true
            //             }
            //         ,
            //         }
            //     ,
            //     }
            // ,
        }
    }
</script>
<style scoped>
    .slide-fade-enter-active {
        transition: all .8s ease;
    }

    .slide-fade-leave-active {
        transition: all .3s cubic-bezier(1.0, 0.8, 0.8, 1.0);
    }

    .slide-fade-enter,
    .slide-fade-leave-to {
        transform: translateY(100px);
        opacity: 0;
    }

    .sub-header {
        height: auto;
    }

    .header-l {
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
    }

    .submitorder {
        padding-bottom: 50px;
    }

    .pay {
        font-size: 14px;
        font-weight: 500;
        color: #333333;
    }

    .c {
        width: 20px;
        height: 20px;
    }

    .d {
        background: url('../../assets/img/zhifu.jpg');
        background-size: 100% 100%;
        background-repeat: no-repeat;
    }

    .center {
        width: 100%;
        height: 100px;
        background-color: #fff;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .center1 {
        width: 100%;
        padding: 1px 20px;
        background-color: #fff;
    }

    .header-r,
    .header-l {
        font-size: 14px;
        color: #333333;
    }

    .sub-header {
        height: 46px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .add {
        padding-left: 10px;
        font-family: PingFangSC-Regular;
        font-size: 18px;
        color: #666666;
    }

    .shop {
        margin: 10px;
        background-color: #fff;
        padding: 10px;
    }

    .shop-header {
        display: flex;
        align-items: center;
    }

    .header-r {
        padding-left: 20px;
    }

    .q {
        font-size: 13px;
        color: #000000;
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 2;
        overflow: hidden;
    }

    .s {
        font-size: 13px;
        color: #E5532A;
    }

    .f {
        font-size: 10px;
        height: 30px;
        line-height: 30px;
        color: #999999;
    }

    .he {
        width: 100%;
        margin-top: 10px;
        height: 40px;
        border-top: 1px solid #ccc;
        display: flex;
        align-items: center;
        justify-content: flex-end;
    }

    .r {
        width: 100px;
        height: 28px;
        display: flex;
        align-items: center;
        justify-content: space-around;
        border: 1px solid #cccccc;
    }

    .add1,
    .reduce {
        width: 28px;
        height: 28px;
        text-align: center;
        line-height: 28px;
        font-size: 20px;
    }

    .text {
        width: 55px;
        height: 28px;
        text-align: center;
        line-height: 28px;
    }

    .add1 {
        border-left: 1px solid #ccc
    }

    .reduce {
        color: #cccccc;
        border-right: 1px solid #ccc
    }

    .footer {
        width: 100%;
        height: 44px;
        background-color: #fff;
        display: flex;
        align-items: center;
        position: fixed;
        bottom: 0;
    }

    .footer-l {
        flex: 4;
        padding-left: 10px;
        height: 44px;
        line-height: 44px;
        font-size: 16px;
        color: #333333;
    }

    .footer-r {
        height: 44px;
        flex: 1;
        font-size: 13px;
        color: #FFFFFF;
        text-align: center;
        line-height: 44px;
        background: #F5A623;
        letter-spacing: 0;
        width: 80%;
        margin: 20px auto;
    }

    .money {
        font-size: 12px;
        color: #e5532a;
    }

    .ceng {
        width: 100%;
        height: 100%;
        z-index: 1500;
        background-color: rgba(0, 0, 0, 0.5);
        position: fixed;
        left: 0;
        bottom: 0;
        right: 0;
        top: 0;
    }

    .line {
        width: 100%;
        height: 50%;
        /* position: absolute;
        bottom: 0;
        left: 0; */
        background-color: #fff;
    }

    .line-header {
        height: 50px;
        padding: 3px 14px;
        display: flex;
        align-items: center;
        border-bottom: 1px solid #cccccc
    }

    .line-title {
        flex: 5;
        font-size: 16px;
        font-weight: 500;
        color: #333333;
        height: 40px;
        text-align: center;
        line-height: 40px
    }

    .img-r {
        width: 30%;
        height: 50px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
</style>

