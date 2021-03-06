### togetherAd
togetherAd 是一个对所有广告和随机展示逻辑进行封装的一个 Lib

### 目前支持的广告
腾讯广点通GDT、百度Mob

### 调用方法
参数以及作用见相关类的注释即可

1. Lib 初始化操作
```
AdConfig.init(instance);
```

2. 配置各个广告的 ID 和 位ID
```
AdConfig.setAdConfigBaidu(new AdConfig.AdConfigBaiduId() {
    @Override
    public String BAIDU_AD_APP_ID() {
        return "XXXXXXX";
    }

    @Override
    public String BAIDU_AD_SPLASH() {
        return "XXXXXXX";
    }

    @Override
    public String BAIDU_AD_INTER() {
        return "XXXXXXX";
    }

    @Override
    public String BAIDU_AD_PLAYER() {
        return "XXXXXXX";
    }

    @Override
    public String BAIDU_AD_HOT() {
        return "XXXXXXX";
    }
});

AdConfig.setAdConfigTencent(new AdConfig.AdConfigGDTId() {
    @Override
    public String GDT_AD_APP_ID() {
        return "XXXXXXX";
    }

    @Override
    public String GDT_AD_SPLASH() {
        return "XXXXXXX";
    }

    @Override
    public String GDT_AD_INTER() {
        return "XXXXXXX";
    }

    @Override
    public String GDT_AD_HOT() {
        return "XXXXXXX";
    }

    @Override
    public String GDT_AD_PLAYER() {
        return "XXXXXXX";
    }
});
```

3. 开屏广告的调用方法
```
AdHelperSplashFull.showAdFull(mContext, configSplash ?: "", mFlTopContainer, object : AdHelperSplashFull.AdListenerSplashFull {
    override fun onStartRequest(channel: String) {
        ClickEvent.adPolyRequest(mContext, "开屏", channel)
    }

    override fun onAdClick(channel: String) {
        ClickEvent.adPolyClick(mContext, "开屏", channel)
    }

    override fun onAdFailed(failedMsg: String?) {
        actionHome(2000)
    }

    override fun onAdDismissed() {
        if (canJumpImmediately) {
            actionHome(0)
        }
        canJumpImmediately = true
    }

    override fun onAdPrepared(channel: String) {
        ClickEvent.adPolyShow(mContext, "开屏", channel)
    }
})
```

### 随机广告配置的规则
假如有 BAIDU，GDT，ADVIEW 这三种广告，实际的配置字符串应该是这样的："baidu:3,gdt:3,adview:4" 

1. 随机广告配置必须符合这样的格式
"xxx:m,yyy:n,zzz:i"

2. AdRandomUtil 这个类只会识别特定的 key （ 例：baidu、gdt、adview ）
"baidu:2,gdt:8" <==>  "baidu:2,gdt:8,abc:3"

3. key 不区分大小写
"BAIDU:2,GDT:8" <==>  "baidu:2,gdt:8"
"Baidu:2,Gdt:8" <==>  "baidu:2,gdt:8"

### 广告切源的逻辑以及实际实现的方式
假如有 BAIDU，GDT，ADVIEW 这三种广告 （ 实际的配置字符串："baidu:3,gdt:3,adview:4" ） 

第一次随机到了 GDT，如果 GDT 请求失败，将 GDT 的 key 使用一个通用的字符串替换，再从其他的广告中再随机 
此时的配置字符串："baidu:3,HIDE:3,adview:4"

第二次随机到了 BAIDU，如果 BAIDU 也请求失败了，将 BAIDU 的 key 使用一个通用的字符串替换，再从其他的广告中再随机  
此时的配置字符串："HIDE:3,HIDE:3,adview:4"
......
直到请求某个广告成功后停止 
如果所有的广告全部失败，此时的配置字符串："HIDE:3,HIDE:3,HIDE:4"

> 目前开屏广告没有切源的逻辑（ 因为开屏时间太久影响用户的体验 ）
