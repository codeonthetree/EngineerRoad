Arthas是一个诊断工具，使用简单，功能实用，可以帮助定位问题，在指定的方法上加入切面来实现

常见问题：
我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
遇到问题，看代码逻辑看不出来，QA环境下，无法复现，线上环境无法 debug，只能在预发上通过加日志再重新发布吗？极其耗时
查看一个非接口方法的RT，或者是几个内部方法的RT总和

使用：我们公司的机器都自带arthas 
- su app
- sh as.sh

常用命令：
jad 可以反编译指定的类，遇上不知道当前这个class是什么版本的时候使用

jad com.youzan.ump.voucher.core.impl.service.voucher.BuyerVoucherReadServiceImpl

trace 查看接口的响应时间，包括每个下一级方法的响应时间，可以持续跟踪，查看下一级方法被调用

trace com.youzan.ump.voucher.core.biz.service.voucher.impl.BuyerVoucherReadInnerServiceImpl generateActivityEntityIdMap 'params[0]==63077'

查看getAssets接口RT
- trace com.youzan.ump.asset.service.BillPreferenceServiceImpl getAssets 'params[0].kdtId==63077'
- trace com.youzan.ump.voucher.core.biz.service.voucher.impl.VoucherVerifyInnerServiceImpl listVoucherBeforeOrder 'params[0].kdtId==63077'

- trace com.youzan.ump.voucher.core.biz.service.voucher.impl.VoucherVerifyInnerServiceImpl listVoucherBeforeOrder 'params[0].kdtId==63077'

- trace com.youzan.ump.voucher.core.biz.service.voucher.impl.VoucherVerifyInnerServiceImpl listVoucherBeforeOrder 'params[0].kdtId==63077'

trace com.youzan.ump.voucher.core.biz.service.voucher.impl.VoucherVerifyInnerServiceImpl listInnerVoucherBeforeOrder 'params[0].kdtId==63077'

trace com.youzan.ump.voucher.core.biz.service.voucher.impl.VoucherVerifyInnerServiceImpl prepareVoucherVerifyContexts 'params[0].kdtId==63077'

trace com.youzan.ump.voucher.core.core.service.voucher.UserVoucherCoreService listUsableVoucher 'params[0]==63077'

stack 与trace相反，可以查看当前方法被调用的调用栈

stack com.youzan.ump.voucher.core.core.service.voucher.UserVoucherCoreService listUsableVoucher 'params[0]==63077'

watch 可以查看方法的出参，入参，提高效率，可以省去在预发上部署代码，打日志
第一个参数：类名
第二个参数：方法名
第三个参数："{params,returnObj,throwExp}"
第四个表达式 支持条件，'params[0]=63077'
命令参数 -x 2 -e
watch com.youzan.ump.voucher.core.biz.service.voucher.impl.BuyerVoucherReadInnerServiceImpl generateActivityEntityIdMap "{params,returnObj,throwExp}" 'params[0]==63077' -x 2

修改为json格式
options

其他：
jvm，thread，dashboard